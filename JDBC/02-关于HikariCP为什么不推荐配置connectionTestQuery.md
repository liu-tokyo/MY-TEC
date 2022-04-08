# 关于HikariCP为什么不推荐配置connectionTestQuery

## 目录

- [关于HikariCP为什么不推荐配置connectionTestQuery](#关于hikaricp为什么不推荐配置connectiontestquery)
  - [目录](#目录)
  - [HikariCP推荐jdbc4不配置connectionTestQuery](#hikaricp推荐jdbc4不配置connectiontestquery)
  - [hikari jdbc4连接检测方式分析](#hikari-jdbc4连接检测方式分析)
    - [针对源码分析](#针对源码分析)
      - [**com.zaxxer.hikari.pool.PoolBase**](#comzaxxerhikaripoolpoolbase)
      - [**com.mysql.cj.jdbc.ConnectionImpl**](#commysqlcjjdbcconnectionimpl)
      - [**com.mysql.cj.NativeSession**](#commysqlcjnativesession)
    - [源码分析结论](#源码分析结论)
  - [参考：](#参考)

---

在[《在SpringBoot 1.5.3上使用gradle引入hikariCP》](https://www.cnblogs.com/lyhero11/p/12097593.html)一文里写了怎么在SpringBoot1.5上用Hikari连接池，转眼来到SpringBoot2.1，不用麻烦了，因为这货已经干掉了tomcat-jdbcPool成为被springboot认可的默认的连接池了！

但之前有个没理解的事情，前文中properties文件有个注释掉的配置：

```erlang
# 配置了这个query则每次拿连接的时候用这个query测试，否则就使用java.sql.Connection的isValid测试  推荐jdbc4不配置。
#spring.datasource.hikari.connection-test-query=SELECT 1 FROM DUAL   
```



## HikariCP推荐jdbc4不配置connectionTestQuery

官网上是这么讲的：

*If your driver supports JDBC4 we strongly recommend not setting this property. This is for "legacy" drivers that do not support the JDBC4 `Connection.isValid()` API. This is the query that will be executed just before a connection is given to you from the pool to validate that the connection to the database is still alive. Again, try running the pool without this property, HikariCP will log an error if your driver is not JDBC4 compliant to let you know. Default: none*
“如果用JDBC4驱动的话强烈建议不要配connectionTestQuery，这玩意是给旧版的不支持Connection.isValid() API的驱动准备的。在每次从池里拿连接的时候用这个connectionTestQuery配置的SQL语句执行一下，来检验连接的有效性。再一次声明，试着不用这个配置项来运行hikari，如果你的驱动不支持JDBC4我们会log一个error来提示你的。”

## hikari jdbc4连接检测方式分析

之前用的**tomcat jdbc pool**或者更早之前过的**dbcp**，在连接有效性保障上都是有两种策略，**testOnBorrow**以及**testWhileIdle**，**test**的手段就是用**sql**比如**select 1 from dual**来执行一下。当时开发人员的共识是为了保证性能，不配置**testOnBorrow**，而采用**testWhileIdle**这种方式，由连接池内部的一个异步线程去定时的调用上面的**test sql**来排除失效的连接。（技术是相通的，**apache HttpClient**的**pool**也是类似思路来排查失效连接）

所以Hikari既然官方号称不要使用这个**test sql**，而且在**testOnBorrow**时检测连接有效性，那么我们可以推断两个事情：

1. 连接池也就是客户端侧应用层不来做检测连接这个事情了，这个事交给底下一层的网络通信层也就是驱动程序来做。
2. 假如能够保证检测这个动作足够高效而轻量，那么使用**testOnBorrow**也未尝不可，且相比异步定时检测的方式时效性上更有保证。
   下面我们带着上面的两个猜测来走读一下相关的Hikari源代码。

### 针对源码分析

#### **com.zaxxer.hikari.pool.PoolBase**

```java
   boolean isConnectionAlive(final Connection connection)
   {
    
            if (isUseJdbc4Validation) {
               return connection.isValid(validationSeconds); //如果支持JDBC4就用connection.isValid接口

   }
```

接下来，来到了驱动程序：

#### **com.mysql.cj.jdbc.ConnectionImpl**

```java
    @Override
    public boolean isValid(int timeout) throws SQLException {
           synchronized (getConnectionMutex()) {
            if (isClosed()) {
                return false;
            }

            try {
                try {
                    pingInternal(false, timeout * 1000); //ping一下检测连接有效性
                } catch (Throwable t) {
                    try {
                        abortInternal();
                    } catch (Throwable ignoreThrown) {
                        // we're dead now anyway
                    }

                    return false;
                }

            } catch (Throwable t) {
                return false;
            }
            return true;
        }
    }
  
    @Override
    public void pingInternal(boolean checkForClosedConnection, int timeoutMillis) throws SQLException {
        this.session.ping(checkForClosedConnection, timeoutMillis);
    }
```

看起来是pingInternal(false, timeout * 1000)这一步，进行了网络通信去检测了连接。进去看看：

#### **com.mysql.cj.NativeSession**

```java
    public void ping(boolean checkForClosedConnection, int timeoutMillis) {
        if (checkForClosedConnection) {
            checkClosed();
        }

        long pingMillisLifetime = getPropertySet().getIntegerProperty(PropertyKey.selfDestructOnPingSecondsLifetime).getValue();
        int pingMaxOperations = getPropertySet().getIntegerProperty(PropertyKey.selfDestructOnPingMaxOperations).getValue();

        if ((pingMillisLifetime > 0 && (System.currentTimeMillis() - this.connectionCreationTimeMillis) > pingMillisLifetime)
                || (pingMaxOperations > 0 && pingMaxOperations <= getCommandCount())) {

            invokeNormalCloseListeners();

            throw ExceptionFactory.createException(Messages.getString("Connection.exceededConnectionLifetime"),
                    MysqlErrorNumbers.SQL_STATE_COMMUNICATION_LINK_FAILURE, 0, false, null, this.exceptionInterceptor);
        }
        //前边一堆判断大概是否超过了最大ping的次数pingMaxOperations，以及连接是否超过了LifeTime
        //如果都ok，那么就执行下面的command, sendCommand(this.commandBuilder.buildComPing(null)是构造了一个message
        sendCommand(this.commandBuilder.buildComPing(null), false, timeoutMillis); // it isn't safe to use a shared packet here 
    }

    public final NativePacketPayload sendCommand(NativePacketPayload queryPacket, boolean skipCheck, int timeoutMillis) {
        return (NativePacketPayload) this.protocol.sendCommand(queryPacket, skipCheck, timeoutMillis);
    }
```

看起来是向数据库发了一个**message**来测试连接。

```java
public NativePacketPayload buildComPing(NativePacketPayload sharedPacket) {
        NativePacketPayload packet = sharedPacket != null ? sharedPacket : new NativePacketPayload(1);
        packet.writeInteger(IntegerDataType.INT1, NativeConstants.COM_PING);
        return packet;
    }
static final int COM_PING = 14;
```

哦，原来是向数据库发了一个网络包，这个包里边是一个整数14
我们来看下是怎么发的，从**sendCommand**方法开始，经过一层一层的**send**方法，最后找到如下方法：

```java
package com.mysql.cj.protocol.a;

import java.io.BufferedOutputStream;
import java.io.IOException;

import com.mysql.cj.protocol.MessageSender;

public class SimplePacketSender implements MessageSender<NativePacketPayload> {
    public void send(byte[] packet, int packetLen, byte packetSequence) throws IOException {
        PacketSplitter packetSplitter = new PacketSplitter(packetLen);
        while (packetSplitter.nextPacket()) {
            this.outputStream.write(NativeUtils.encodeMysqlThreeByteInteger(packetSplitter.getPacketLen()));
            this.outputStream.write(packetSequence++);
            this.outputStream.write(packet, packetSplitter.getOffset(), packetSplitter.getPacketLen());
        }
        this.outputStream.flush();
    }
}
```

搞清楚了，原来是用**java bio**的输出流向数据库服务器写的**message**。

### 源码分析结论

至此，我们搞清楚了Hikari的连接检测机制：

如果驱动程序支持JDBC4的标准，那么就放弃使用执行SQL语句这种重量级的需要数据库引擎参与的检测方式，用这种方式无论是**testOnBorrow**策略还是异步定时检测策略，都是要用到数据库引擎的算力的。

将连接检测的工作交给JDBC4驱动程序的**connectin.isAlive**接口，这个接口十分轻量级，采用的是向数据库发送一个心跳包的方式来试探连接是否存活，发送的时候使用的**java bio**包。（这也说明了我们的JDBC是同步阻塞的模型）

最后，由于检测方式十分轻量，**HikariCP**索性采用了时效性更好的连接检测策略**testOnBorrow**

## 参考：

https://github.com/brettwooldridge/HikariCP/ HikariCP官网
https://www.cnblogs.com/littleatp/p/14088588.html 《MySQL 连接为什么挂死了》 作者真大神，赞一个，关于mysql连接串的配置最好要加上socket超时和connect超时的提醒很及时！jdbc:mysql://10.0.71.13:33052/appdb?socketTimeout=60000&connectTimeout=30000&serverTimezone=UTC
https://blog.csdn.net/weixin_30899789/article/details/113214524 也是说的这个事，数据库地址配置上要加上socketTimeout和connectTimeout