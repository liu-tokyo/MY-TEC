# JMX详解及JConsole使用

## JMX

JMX（Java Management Extensions）是一个应用程序植入管理功能的框架，是一套标准的代理和服务，服务是JDK官方提供的Java程序性能监控程序。支持远程访问，支持扩展，即自定义监控的性能参数。提供网络、API、客户端三个层次的调用。实际上，Java平台使用JMX作为管理和监控的标准接口，任何程序只要按JMX规范访问这个接口，就可以获取所有的管理和监控信息。常用的运维监控如Zabbix、Nagios等工具对JVM本身的监控都是通过JMX获取的信息。


## 应用场景

中间件软件WebLogic的管理页面就是基于JMX开发的，而JBoss则是整个系统都基于JMX架构，对于一些参数的修改，有几种方式如下：

写死在代码里，需要改动的时候去修改，然后重新编译发布。

写在配置文件里，例如java的properties，需要改变时，只需要修改配置文件，但必须重启系统才能生效。

写一段代码，把配置值缓存起来，系统在获取的时候，先看看配置文件改动没，如有改动，则从配置里获取最新值，否则从缓存里读取，例如读取Apollo配置中心数据。

用JMX把需要配置的属性集中在一个类里，然后写一个MBean，再进行相关配置，并且JMX提供了JConsole工具页，方便对参数值进行修改。


## JMX架构

```
  ┌─────────┐  ┌─────────┐
    │jconsole │  │   Web   │
    └─────────┘  └─────────┘
         │            │
┌ ─ ─ ─ ─│─ ─ ─ ─ ─ ─ ┼ ─ ─ ─ ─
 JVM     ▼            ▼        │
│   ┌─────────┐  ┌─────────┐
  ┌─┤Connector├──┤ Adaptor ├─┐ │
│ │ └─────────┘  └─────────┘ │
  │       MBeanServer        │ │
│ │ ┌──────┐┌──────┐┌──────┐ │
  └─┤MBean1├┤MBean2├┤MBean3├─┘ │
│   └──────┘└──────┘└──────┘
 ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
```

JMX的结构一共分为三层：基础层（主要是MBean）、适配层（Adaptor）、接口层(Connector)

### MBean

JMX把所有被管理的资源都成为MBean（ManagedBean），这些MBean全部由MBeanServer管理，如果要访问MBean，可以通过MBeanServer对外提供的访问接口，例如RMI或HTTP。

使用JMX不需要安装任何额外组件，也不需要第三方库，因为MBeanServer已经内置在JavaSErver标准库中，JavaSE还提供了一个JConsole程序，用于RMI连接MBeanServer，这样就可以管理整个进程。

除了JVM会把自身的各种资源以MBean注册到JMX中，我们自己的配置、监控信息也可以作为MBean注册到JMX，这样，管理程序就可以直接控制我们暴露的MBean。

MBean分为如下四种：

| 类型           | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| standard MBean | 这种类型的MBean最简单，它能管理的资源（包括属性，方法，时间）必须定义在接口中，然后MBean必须实现这个接口。它的命名也必须遵循一定的规范，例如我们的MBean为Hello，则接口必须为HelloMBean。 |
| dynamic MBean  | 必须实现javax.management.DynamicMBean接口，所有的属性，方法都在运行时定义 |
| open MBean     | 此MBean的规范还不完善，正在改进中                            |
| model MBean    | 与标准和动态MBean相比，你可以不用写MBean类，只需使用javax.management.modelmbean.RequiredModelMBean即可。RequiredModelMBean实现了ModelMBean接口，而ModelMBean扩展了DynamicMBean接口，因此与DynamicMBean相似，Model MBean的管理资源也是在运行时定义的。与DynamicMBean不同的是，DynamicMBean管理的资源一般定义在DynamicMBean中（运行时才决定管理那些资源），而model MBean管理的资源并不在MBean中，而是在外部（通常是一个类），只有在运行时，才通过set方法将其加入到model MBean中。后面的例子会有详细介绍 |

### 适配层

MBeanServer 主要是提供对资源你的注册和管理

### 接入层

提供远程访问入口

## 使用方法

- 编写MBean提供的管理接口和监控数据。
- 注册MBean

### 案例：

在启动程序添加注解@EnableMBeanExport，告知spring自动注册MBean

```java
@SpringBootApplication
@MapperScan(basePackages = "com.lyj.demo.mapper")
@EnableMBeanExport // 自动注册MBean
public class SpringbootDemoApplication {

    private static final Logger logger = LoggerFactory.getLogger(SpringbootDemoApplication.class);
    public static void main(String[] args) {
        SpringApplication.run(SpringbootDemoApplication.class, args);
        }
        }
   }
}
```

编写MBean，这里以配置黑名单数据为例，通过JConsole来获取配置数据和添加数据删除数据，并且配置后立即生效不需要重新编译和发布代码。

```java
package com.lyj.demo.mbean;

import org.springframework.jmx.export.annotation.ManagedAttribute;
import org.springframework.jmx.export.annotation.ManagedOperation;
import org.springframework.jmx.export.annotation.ManagedOperationParameter;
import org.springframework.jmx.export.annotation.ManagedResource;
import org.springframework.stereotype.Component;

import java.util.HashSet;
import java.util.Set;

/**
 * @author 凌兮
 * @date 2021/3/25 14:05
 * 黑名单MBean类名必须以MBean结尾，这是规则
 */
@Component
// 表示这是一个MBean，将要被注册到JMX，objectName指定了这个MBean的名字，通常以company:name=Xxx(公司名)；来分类MBean
@ManagedResource(objectName = "company:name = blackList", description = "blackList of Ip address")
public class BlackListMBean {
    private static Set<String> ips = new HashSet<>();

    static {
        ips.add("123");
        ips.add("234");
    }

    /**
     * 对于属性，使用@ManagedAttribute注解标注，本次的MBean只有get属性，没有set
     * 属性，说明这是一个只读属性。
     * @return
     */
    @ManagedAttribute(description = "Get IP addresses in blacklist")
    public String[] getBlacklist() {
        return ips.toArray(new String[1]);
    }

    /**
     * 对于操作，使用@ManagedOperation注解标注，操作有addBlacklist()和
     * removeBlacklist（）其他方法如shouldBlock()不会被暴露给JMX。
     * @param ip
     */
    @ManagedOperation
    @ManagedOperationParameter(name = "ip", description = "Target IP address that will be added to blacklist")
    public void addBlacklist(String ip) {
        ips.add(ip);
    }

    @ManagedOperation
    @ManagedOperationParameter(name = "ip", description = "Target IP address that will be removed from blacklist")
    public void removeBlacklist(String ip) {
        ips.remove(ip);
    }

    public boolean shouldBlock(String ip) {
        return ips.contains(ip);
    }
}
```

拦截接口和实现类：

```
public interface IpInterceptor {

    boolean preHandle(String ip);
}
```

```
@Component
@Slf4j
public class BlackIpHandle implements IpInterceptor {
    @Autowired
    private BlackListMBean blackListMBean;

    @Override
    public boolean preHandle(String ip) {
        return blackListMBean.shouldBlock(ip);
    }
}
```

测试方法：

```
@RequestMapping("/test/Jmx/{str}")
public void jmxTest(@PathVariable(value = "str") String str) {
    if (blackIpHandle.preHandle(str)) {
        loggger.warn("拦截成功");
        return;
    }
    loggger.warn("拦截失败");
    return;
}
```

启动应用程序，打开java里的jdk文件里的bin文件的里JConsole.exe，找到你的应用程序，连接进去，

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021032514592388.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDkzMjU1,size_16,color_FFFFFF,t_70)

在JConsole里可以查看内存和堆栈线程等信息，这里可以看到MBean列表，里面可以找到我们的注册的Mean（company）
,里面有属性，操作，通知等，
首先我们执行获取黑名单数据方法结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210325150229170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDkzMjU1,size_16,color_FFFFFF,t_70)

可以看到三个数据，添加数据也是一样，6666是我后来动态通过JConsole添加的，然后执行请求，直接拦截成功。这里我们直接测试的结果如图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210325150342859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMDkzMjU1,size_16,color_FFFFFF,t_70)

## Java程序开启JMX服务

想监控Java程序，需要在程序启动时加上JMX相关参数（本地连接，这里加不加都可以，这里的端口号是JConsole监听的端口号，不是服务的端口号）。

```
-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9102
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false
```

三个参数分别为：服务端口，安全策略，SSL加密

## 远程连接

测试环境部署在RedHat6.5服务器上，一般说明增加如下参数即可允许远程连接。

```
-Dcom.sun.management.jmxremote.port=8999
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
```

但是实际测试无法连接，经过查询资料，最后配置如下，实现了远程连接。

```
(java -jar -Dcom.sun.management.jmxremote  -Dcom.sun.management.jmxremote.port=8999 -Dcom.sun.management.jmxremote.rmi.port=9999 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false app-1.0.jar&)
```

同时还需要注意服务器的端口是否被屏蔽，hosts是否配置了实际IP。可以使用hostname -i命令来查询ip是否生效。例如实际ip是10.10.10.101，计算机名是mycomputer。hosts配置如下：

```
10.10.10.101   mycomputer
```

