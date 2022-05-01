# HikariCP参数配置

## 目录

- [HikariCP参数配置](#hikaricp参数配置)
  - [目录](#目录)
  - [必须配置](#必须配置)
  - [非必须参数配置](#非必须参数配置)
  - [非常用参数配置](#非常用参数配置)

---

## 必须配置



| -    | 参数                        | 说明                                                         | 默认值 |
| ---- | --------------------------- | ------------------------------------------------------------ | ------ |
| 1    | dataSourceClassName/jdbcUrl | 两种数据源配置方式，dataSourceClassName 是JDBC驱动提供的DataSource的名字，dataSourceClassName 不支持XA 的数据源配置，如果使用jdbcUrl 基于驱动管理的程序配置，则不用配置dataSourceClassName属性。 | 无     |
| 2    | username                    | 从基础驱动程序获取Connections时使用的默认身份验证用户名。    | 无     |
| 3    | password                    | 从基础驱动程序获取Connections时使用的默认身份验证密码。      | 无     |



## 常用配置

|  -   | 参数                | 说明                                                         | 默认值              |
| :--: | ------------------- | ------------------------------------------------------------ | ------------------- |
|  1   | autoCommit          | 此属性控制从池返回的连接的默认自动提交行为。它是一个布尔值。 | true                |
|  2   | connectionTimeout   | 此属性控制客户端（即用户的程序）等待池中连接的最长毫秒数。如果在没有连接可用的情况下超过此时间，则将抛出SQLException异常。最低可接受的连接超时为250毫秒。 | 30000（30秒）       |
|  3   | idleTimeout         | 此属性控制连接允许被闲置在池中的最大时间。此设置仅适用于minimumIdle定义为比maximumPoolSize小的时候 ,minimum允许的最小值为10000毫秒（10秒)。<br /> 如果 idleTimeout + SECONDS.toMillis(1) > maxLifetime && maxLifetime > 0 && minIdle < maxPoolSize ,则idleTimeout = 0 ,表示永不退出，设想一下，如果此时应用有大量请求进入，连接池中的空闲连接被清理掉，此时还没有连接进行补充进来，那么应用中每个线程都会因获取不到连接增加了响应延迟，所以，源代码中关于这个条件下的idleTimeout = 0 设置合理;<br/> 如果 idleTimeout != 0 && idleTimeout < SECONDS.toMillis(10) && minIdle < maxPoolSize,idleTimeout = 10s;<br/> 连接退出的条件: 最大变化为+30秒，平均变化为+15秒。在此超时之前，连接永远不会因空闲状态而退役。 | 600000(10分钟）     |
|  4   | maxLifetime         | 此属性控制池中连接的最大生命周期。使用中的连接永远不会退出，除非它被关闭然后移除，值为0表示没有最大生存期（无限生存期）。 | 1800000（30分钟）   |
|  5   | connectionTestQuery | 一个检测查询，在数据库连接池给出连接之前进行查询，以验证与数据库的连接是否仍然存在且有效,如果追求极致性能的话，**建议不要配置该属性，因为不配置的时候会通过ping命令进行连接检测，性能会更高(> select)** | 无                  |
|  6   | minimumIdle         | 此属性控制HikariCP尝试在池中维护的最小空闲连接数。若空闲连接低于此值且池中的总连接数小于maximumPoolSize，则HikariCP将尽最大努力快速有效地添加其他连接。建议不设置此值,而是允许HikariCP充当一个固定大小的连接池（如果minimumIdle未设置则默认为是maximumPoolSize，因此即使idleTimeout设置为1分钟，一旦连接关闭，它将在池中被替换） | maximumPoolSize相同 |
|  7   | maximumPoolSize     | 此属性控制允许数据库连接池到达的最大大小，包括空闲和正在使用的连接。 | 10                  |
|  8   | metricRegistry      | 此属性仅通过编程配置或IoC容器可用,配置用来进行连接池度量指标监控。 | 无                  |
|  9   | healthCheckRegistry | 来报告当前系统的健康信息。                                   | 无                  |
|  10  | poolName            | 此属性表示连接池的用户定义名称，主要显示在日志记录和JMX管理控制台中，以标识池和池配置。 | 自动生成            |



## 非常用配置




|  -   | 参数                      | 说明                                                         | 默认值       |
| :--: | ------------------------- | ------------------------------------------------------------ | ------------ |
|  1   | initializationFailTimeout | 如果池无法成功初始化连接，则此属性控制池是否“快速失败”。任何正数都被认为是尝试获取初始连接的毫秒数；在此期间，应用程序线程将被阻塞。如果在超时发生之前无法获取连接，则将引发异常; | 1毫秒        |
|  2   | isolatelnternalQueries    | 此属性决定HikariCP是否在自己的事务中隔离内部池查询，例如连接存活测试;<br />由于这些查询通常是只读查询，因此很少有必要将它们封装在自己的事务中。此属性仅在autoCommit 禁用时适用。 | false        |
|  3   | allowPoolSuspension       | 此属性控制池是否可以通过JMX挂起和恢复;                       | false        |
|  4   | readOnly                  | 此属性控制默认情况下从池中获取的Connections是否处于只读模式; | false        |
|  5   | registerMbeans            | 此属性控制是否注册JMX管理Bean（“MBean”);                     | false        |
|  6   | catalog                   | 此属性为支持catalog的数据库设置默认catalog。如果未指定此属性，则使用JDBC驱动程序定义的默认catalog; | 驱动程序默认 |
|  7   | connectionlnitSql         | 此属性设置一个SQL语句，该语句将在每次创建新连接之后执行，然后再将该连接添加到池中。如果此SQL无效或抛出异常，它将被视为连接失败; | 无           |
|  8   | driverClassName           | HikariCP将尝试仅基于jdbcUrl通过DriverManager解析驱动程序，但对于某些较旧的驱动程序必须指定driverClassName。除非用户收到明显的错误消息，表明未找到驱动程序，否则可忽略此属性。 | 无           |
|  9   | transactionlsolation      | 此属性控制从池返回的连接的默认事务隔离级别;                  | 驱动程序默认 |
|  10  | validationTimeout         | 此属性控制连接测试活性的最长时间。该值必须小于connectionTimeout。最低可接受的验证超时为250毫秒。 | 5000         |
|  11  | leakDetectionThreshold    | 此属性控制连接在记录一条指示可能连接泄漏的消息之前流出池的时间。值为0表示禁用泄漏检测。启用泄漏检测的最低可接受值是2000（2秒); | 0            |
|  12  | dataSource                | 此属性允许用户直接设置DataSource要由池包装的实例;            | 无           |
|  13  | schema                    | 该属性为支持schema概念数据库设置默认schema;                  | 驱动程序默认 |
|  14  | threadFactory             | 此属性允许设置java.util.concurrent. ThreadFactory将用于创建池使用的所有线程的实例; | 无           |
|  15  | scheduledExecutor         | 允许设置java.util.concurrent.Scheduled-ExecutorService用于各种内部调度任务的实例; | 无           |

