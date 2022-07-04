# **Kafka快速入门（七）- Kafka集群监控**

## 一、Kafka监控指标
### 1.1、Kafka主机监控指标
主机监控是监控Kafka集群Broker所在的节点机器的性能。常见的主机监控指标包括：
1. 机器负载（Load）
2. CPU使用率
3. 内存使用率，包括空闲内存（Free Memory）和已使用内存（Used Memory）
4. 磁盘I/O使用率，包括读使用率和写使用率网络
5. I/O使用率
6. TCP连接数
7. 打开文件数
8. inode使用情况

### 1.2、JVM监控指标
Kafka Broker进程是一个普通的Java进程，因此所有关于JVM的监控方式都可以用于对Kafka Broker进程的监控。
1. Full GC发生频率和时长，用于评估Full GC对Broker进程的影响。长时间的停顿会令Broker端抛出各种超时异常。
2. 活跃对象大小，是设定堆大小的重要依据，能帮助细粒度地调优JVM各个代的堆大小。
3. 应用线程总数。了解Broker进程对CPU的使用情况。

`2019-07-30T09:13:03.809+0800: 552.982: [GC cleanup 827M->645M(1024M), 0.0019078 secs]`

Broker JVM进程默认使用G1的GC算法，当cleanup步骤结束后，堆上活跃对象大小从827MB缩减成645MB。Kafka 0.9.0.0版本起，默认GC收集器为G1，而G1中的Full GC是由单线程执行的，速度非常慢。因此，需要监控Broker GC日志，即以kafkaServer-gc.log开头的文件。如果发现Broker进程频繁Full GC，可以开启G1的-XX:+PrintAdaptiveSizePolicy开关，让JVM指明是谁引发Full GC。

### 1.3、集群监控指标
1. 查看Broker进程是否启动，端口是否建立。在容器化的Kafka环境中，使用Docker启动Kafka Broker时，Docker容器虽然成功启动，但网络设置如果配置有误，就可能会出现进程已经启动但端口未成功建立监听的情形。
2. 查看Broker端关键日志。Broker端服务器日志server.log，控制器日志controller.log以及主题分区状态变更日志state-change.log。
3. 查看Broker端关键线程的运行状态。Kafka Broker进程会启动十几个甚至是几十个线程。在实际生产环境中，Log Compaction线程是以kafka-log-cleaner-thread开头的，负责日志Compaction；副本拉取消息的线程，通常以ReplicaFetcherThread开头，负责执行Follower副本向Leader副本拉取消息的逻辑。
4. 查看Broker端的关键JMX指标。
BytesIn/BytesOut：即Broker端每秒入站和出站字节数，如果值接近网络带宽，很容易出现网络丢包的情形。
NetworkProcessorAvgIdlePercent：即网络线程池线程平均的空闲比例，通常需要确保其值长期大于30%。如果小于30%，表明网络线程池非常繁忙，需要通过增加网络线程数或将负载转移给其它服务器的方式，来给Broker减负。
RequestHandlerAvgIdlePercent：即I/O线程池线程平均的空闲比例。如果值长期小于30%，需要调整I/O线程池的数量或者减少 Broker端的负载。
UnderReplicatedPartitions：即未充分备份的分区数。所谓未充分备份，是指并非所有的Follower副本都和Leader副本保持同步。
ISRShrink/ISRExpand：即ISR收缩和扩容的频次指标。如果生产环境中出现ISR中副本频繁进出的情形，其值一定是很高的。需要诊断下副本频繁进出ISR的原因，并采取适当的措施。
ActiveControllerCount：即当前处于激活状态的控制器的数量。通常，Controller所在Broker上的ActiveControllerCount指标值是1，其它Broker上的值是 0。如果发现存在多台Broker上ActiveControllerCount值都是1，表明Kafka集群出现了脑裂，必须尽快处理，处理方式主要是查看网络连通性。脑裂问题是非常严重的分布式故障，Kafka目前依托ZooKeeper来防止脑裂，一旦出现脑裂，Kafka无法保证正常工作。
5. 监控Kafka客户端。客户端所在的机器与Kafka Broker机器之间的网络往返时延（Round-Trip Time，RTT）。对于生产者，以kafka-producer-network-thread开头的线程负责实际消息发送，一旦挂掉，Producer将无法正常工作，但Producer进程不会自动挂掉。对于消费者，以kafka-coordinator-heartbeat-thread 开头的心跳线程事关Rebalance。

从Producer角度，需要关注的JMX指标是request-latency，即消息生产请求的延时，最直接地表征Producer程序的TPS；从 Consumer角度，records-lag和records-lead是两个重要的JMX 指标。如果使用Consumer Group，需要关注join rate和sync rate指标，其表明Rebalance的频繁程度。

## 二、JMX监控Kafka
### 2.1、JMX简介
JMX（Java Management Extensions）可以管理、监控正在运行中的Java程序，用于管理线程、内存、日志Level、服务重启、系统环境等。

### 2.2、Kafka开启JMX
开启JMX端口的方式有两种：
1. 启动Kafka时设置JMX_PORT
    export JMX_PORT=9999 kafka-server-start.sh -daemon config/server.properties

2. 修改kafka-run-class.sh
    在kafka-run-class.sh文件开始增加下列行：
    JMX_PORT=9999
    修改kafka-run-class.sh文件后重启Kafka集群。

3. Kafka Docker容器服务的JMX开启
   
   Kafka容器服务的docker-compose.yml文件导入KAFKA_JMX_OPTS和JMX_PORT环境变量。

   ```yaml
   KAFKA_JMX_OPTS: "-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Djava.rmi.server.hostname=192.168.0.105 -Dcom.sun.management.jmxremote.rmi.port=9999"
   JMX_PORT: 9999
   ```
   
   将相应的JMX端口对外暴露
   
   ```yaml
   ports:
         - "9999:9999" # 对外暴露端口号
   ```
   
### 2.3、JMX_PORT占用问题
Kafka需要监控Broker和Topic数据时，需要开启JMX_PORT，通常在脚本kafka-run-class.sh里面定义JMX_PORT变量，但JMX_PORT定义完成后，执行bin目录下脚本工具会报错。原因在于kafka-run-class.sh是被调用脚本，当被其它脚本调用时，Java会绑定JMX_PORT，导致端口被占用。

- ![Kafka快速入门（七）——Kafka集群监控_Kafka 监控](https://s2.51cto.com/images/blog/202005/25/5642d270bda0d81acf4bca5c459e4ae2.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

解决方法是在执行Kafka启动时指定JMX_PORT。

1. supervisor启动Kafka，在supervisor服务启动配置文件中加入environment=JMX_PORT=9999。

2. kafka-server-start.sh脚本启动Kafka，在启动时export JMX_PORT=9999或者在kafka-server-start.sh脚本指定。

3. 修改kafka-run-class.sh脚本

  修改Kafka安装目录下的bin/Kafka-run-class.sh文件：

  ![Kafka快速入门（七）——Kafka集群监控_Kafka 监控_02](https://s2.51cto.com/images/blog/202005/25/f372ef051cf6a9cf6b2f1761851dd9f2.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)



## 三、Kafka监控工具
### 3.1、JMXTool工具
JMXTool是Kafka社区的工具，能够实时查看Kafka JMX指标。

- `kafka-run-class.sh kafka.tools.JmxTool`
  
  - attributes:指定要查询的JMX属性名称，是以逗号分隔的CSV格式。
  - date-format：指定显示的日志格式
  - jmx-url：指定要连接的JMX接口，默认格式是`service:jmx:rmi:///jndi/rmi://:JMX端口/jmxrmi`。
  - object-name：指定要查询的JMX MBean名称。
  - reporting-interval：指定实时查询的时间间隔，默认2s。
  
- 每秒查询一次过去1分钟的Broker端每秒入站的流量（BytesInPerSec）命令如下：

  `kafka-run-class.sh kafka.tools.JmxTool --object-name kafka.server:type=BrokerTopicMetrics,name=BytesInPerSec --jmx-url service:jmx:rmi:///jndi/rmi://:9999/jmxrmi --date-format "YYYY-MM-dd HH:mm:ss" --attributes OneMinuteRate --reporting-interval 1000`

- ActiveController JMX指标查看命令如下：
  `kafka-run-class.sh kafka.tools.JmxTool --object-name kafka.controller:type=KafkaController,name=ActiveControllerCount --jmx-url service:jmx:rmi:///jndi/rmi://:9999/jmxrmi --date-format "YYYY-MM-dd HH:mm:ss" --reporting-interval 1000`

### 3.2、Kafka Manager
Kafka Manager是雅虎公司于2015年开源的一个Kafka监控框架，使用Scala语言开发，主要用于管理和监控Kafka集群。
Kafka Manager目前已经改名为CMAK (Cluster Manager for Apache Kafka）。
GitHub地址： https://github.com/yahoo/CMAK
Kafka Manager Docker镜像：kafkamanager/kafka-manager

- 如果需要设置Kafka Manager基本安全认证，可以为Kafka Manager设置环境变量：

  ```yaml
  KAFKA_MANAGER_AUTH_ENABLED: "true"
  KAFKA_MANAGER_USERNAME: username
  KAFKA_MANAGER_PASSWORD: password
  ```

  

- Kafka-Manager服务部署Docker-Compose.yml文件如下：

  ```yaml
  # 定义kafka-manager服务
  kafka-manager-test:
    image: kafkamanager/kafka-manager # kafka-manager镜像
    restart: always
    container_name: kafka-manager-test
    hostname: kafka-manager-test
    ports:
      - "9000:9000"  # 对外暴露端口，提供web访问
      depends_on:
          - kafka-test # 依赖
        environment:
              ZK_HOSTS: zookeeper-test:2181 # 宿主机IP
              KAFKA_BROKERS: kafka-test:9090 # kafka
              KAFKA_MANAGER_AUTH_ENABLED: "true"
              KAFKA_MANAGER_USERNAME: admin
              KAFKA_MANAGER_PASSWORD: password
  ```

- 启动Kafka Manager服务，登录Kafka Manager Web。

  Web地址： http://127.0.0.1:9000

  ![Kafka快速入门（七）——Kafka集群监控_Kafka 监控_03](https://s2.51cto.com/images/blog/202005/25/9ac080541867e753d7afa6dcba4cc32d.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

- 增加Kafka-Manager管理Kafka Broker节点：

  ![Kafka快速入门（七）——Kafka集群监控_Kafka 监控_04](https://s2.51cto.com/images/blog/202005/25/fc3cd2686b2f666744960697cfb1349a.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 3.3、JMXTrans + InfluxDB + Grafana
通常，监控框架可以使用JMXTrans + InfluxDB + Grafana组合，由于Grafana支持对JMX指标的监控，因此很容易将Kafka各种 JMX指标集成进来，对于已经采用JMXTrans + InfluxDB + Grafana监控方案的公司来说，可以直接复用已有的监控框架，可以极大地节省运维成本。

### 3.4、Confluent Control Center
Control Center能够实时地监控Kafka集群，同时还能够帮助操作和搭建基于Kafka的实时流处理应用。Control Center不是免费的，必须使用Confluent Kafka Platform企业版才能使用。

- ![Kafka快速入门（七）——Kafka集群监控_Kafka 监控_05](https://s2.51cto.com/images/blog/202005/25/32cdbe14a4b191acf6af3c9dd3e2ae24.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 3.5、jconsole
- Jconsole（Java Monitoring and Management Console）是一种基于JMX的可视化监视、管理工具，提供概述、内存、线程、类、VM概要、MBean的监控。
  在Linux Terminal执行jsoncole，在弹出的窗口的远程进程中输入`service:jmx:rmi:///jndi/rmi://192.168.0.105:9999/jmxrmi`或`192.168.0.105:9999`。

  ![Kafka快速入门（七）——Kafka集群监控_Kafka 监控_06](https://s2.51cto.com/images/blog/202005/25/50d06b5471c5c216d5937197b47e551f.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

- 选择MBeans选项卡，

  ![Kafka快速入门（七）——Kafka集群监控_Kafka 监控_07](https://s2.51cto.com/images/blog/202005/25/9c7da600b641fe72a951b5e8b353e016.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

### 3.6、KafkaCenter
KafkaCenter是EC Bigdata Team多年kafka使用经验的落地实践，整合集群管理、集群运维、生产监控、消费监控、周边生态等统一一站式解决方案，目前已经开源。
KafkaCenter主要功能模块：
1. Home：查看平台管理的Kafka Cluster集群信息及监控信息。
2. Topic：用户可以查看自己的Topic，发起申请新建Topic，同时可以对Topic进行生产消费测试。
3. Monitor：用户可以查看Topic的生产以及消费情况，同时可以针对消费延迟情况设置预警信息。
4. Kafka Connect：实现用户快速创建自己的Connect Job，并对自己的Connect进行维护。
5. KSQL：实现用户快速创建自己的KSQL Job，并对自己的Job进行维护。
6. Approve：主要用于当普通用户申请创建Topic，管理员进行审批操作。
7. Setting：主要功能为管理员维护User、Team以及kafka cluster信息。
8. Kafka Manager：用于管理员对集群的正常维护操作。

GitHub地址： https://github.com/xaecbd/KafkaCenter



## 四、JMXTrans
### 4.1、JMXTrans简介
JMXTrans是一个通过JMX采集Java应用程序的数据采集器，只要Java应用程序开启JMX端口，就可以进行采集。
JMXTrans以后台deamon形式运行，每隔1分钟采集一次数据。
GitHub地址： https://github.com/jmxtrans/jmxtrans
JMXTrans Docker容器镜像下载：
`docker pull jmxtrans/jmxtrans`

### 4.2、JMXTrans配置文件
JMXTrans默认读取/var/lib/jmxtrans目录下所有数据源配置文件(json格式文件)，实时从数据源中获取数据，解析数据后存储到InfluxDB中。

- JMXTrans配置JSON文件如下：

  ```yaml
  {
     "servers": [{
        "port": "9901",
        "host": "192.168.0.105",
        "queries": [{
           "obj": "kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec",
           "attr": ["MeanRate", "OneMinuteRate", "FiveMinuteRate", "FifteenMinuteRate"],
           "resultAlias": "kafkaServer",
           "outputWriters": [{
              "@class": "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
              "url": "http://192.168.0.105:8086/",
              "username": "admin",
              "password": "123456",
              "database": "jmx",
              "tags": {
                 "application": "kafka_server"
              }
           }]
        }]
     }]
  }
  ```

  ```
  servers:数组，数据源配置。
  port:字符串，接收jmx的json数据的端口。
  host:字符串，接收jmx的json数据的IP地址。
  queries:数组，具体监控指标项，按JSON格式列出多个指标项，监控指标可以通过jconsole工具（JDK自带的工具）获取。
  obj:字符串，监控指标的名称。
  attr:数组，需要存储的指标项字段，是数据目标表的字段名。
  resultAlias:字符串，InfluxDB中的表名。
  outputWriters:数组，数据目的地。
  @class:字符串，数据目的地的类。
  url:字符串，数据目的地( InfluxDb )的url。
  username:字符串，InfluxDB登录名。
  password:字符串，InfluxDB登录密码。
  database:字符串，InfluxDB数据库名(需要预先创好)。
  tags:json，避免指标项在 InfluxDbB表中所对应的字段重名的情况。
  ```

  

### 4.3、Kafka JMX监控指标
Kafka的JMX监控指标可以通过jconsole进行获取。

- 对于BytesInPerSec监控指标，在jconsole的MBeans选项页找到BytesInPerSe。

  ![Kafka快速入门（七）——Kafka集群监控_Kafka 监控_08](https://s2.51cto.com/images/blog/202005/25/716b4e5da4745c74aa841b1125ff84b1.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

- ObjectName的值是监控指标obj的值。
  
  ObjectName的属性是"attr"对应的指标值，可以选择一个或多个。
  
  metric名称是resultAlias对应的指标值，在InfluxDB中是MEASUREMENTS名。
  
  “tags” 对应InfluxDB的tag功能，用于与存储在同一个MEASUREMENTS里的不同监控指标做区分。
  
  ```yaml
  {      
     "obj":"kafka.server:type=BrokerTopicMetrics,name=BytesInPerSec",
           "attr":[ "Count", "EventType"，"RateUnit"，"OneMinuteRate" ],
           "resultAlias":"BytesInPerSec",
           "outputWriters": [{
        "@class" :   "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
                "url" :   "http://192.168.0.105:8086/",
                "username" :   "admin",
                "password" :   "123456",
                "database" :   "jmx",
                "tags"     :  {
           "application" :   "BytesInPerSec"
        }
     } ]
  }
  ```
  
  
  
- 对于全局监控，每一个监控指标对应一个InfluxDB的MEASUREMENTS，所有的Kafka节点的同一个监控指标数据写同一个MEASUREMENTS；对于Topic的监控指标，同一个Topic的所有Kafka节点写到同一个MEASUREMENTS，并且以Topic名称命名。

  ```yaml
  {
    "servers" : [ {
      "port" : "9999",
      "host" : "192.168.0.105",
      "queries" : [ {
        "obj" : "java.lang:type=Memory",
        "attr" : [ "HeapMemoryUsage", "NonHeapMemoryUsage" ],
        "resultAlias":"jvmMemory",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      },{
        "obj" : "kafka.server:type=BrokerTopicMetrics,name=MessagesInPerSec",
        "attr" : [ "MeanRate","OneMinuteRate","FiveMinuteRate","FifteenMinuteRate" ],
        "resultAlias":"kafkaServer",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      },{
        "obj" : "kafka.server:type=BrokerTopicMetrics,name=BytesInPerSec",
        "attr" : [ "MeanRate","OneMinuteRate","FiveMinuteRate","FifteenMinuteRate" ],
        "resultAlias":"kafkaServer",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      },{
        "obj" : "kafka.server:type=BrokerTopicMetrics,name=BytesOutPerSec",
        "attr" : [ "MeanRate","OneMinuteRate","FiveMinuteRate","FifteenMinuteRate" ],
        "resultAlias":"kafkaServer",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      },{
        "obj" : "kafka.server:type=BrokerTopicMetrics,name=FailedProduceRequestsPerSec",
        "attr" : [ "MeanRate","OneMinuteRate","FiveMinuteRate","FifteenMinuteRate" ],
        "resultAlias":"kafkaServer",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      },{
        "obj" : "kafka.server:type=BrokerTopicMetrics,name=FailedFetchRequestsPerSec",
        "attr" : [ "MeanRate","OneMinuteRate","FiveMinuteRate","FifteenMinuteRate" ],
        "resultAlias":"kafkaServer",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      },{
        "obj" : "kafka.server:type=BrokerTopicMetrics,name=TotalFetchRequestsPerSec",
        "attr" : [ "MeanRate","OneMinuteRate","FiveMinuteRate","FifteenMinuteRate" ],
        "resultAlias":"kafkaServer",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      },{
        "obj" : "kafka.server:type=BrokerTopicMetrics,name=TotalProduceRequestsPerSec",
        "attr" : [ "MeanRate","OneMinuteRate","FiveMinuteRate","FifteenMinuteRate" ],
        "resultAlias":"kafkaServer",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      },{
        "obj" : "kafka.server:type=ReplicaManager,name=UnderReplicatedPartitions",
        "attr" : [ "Value" ],
        "resultAlias":"underReplicated",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      },{
        "obj" : "kafka.controller:type=KafkaController,name=ActiveControllerCount",
        "attr" : [ "Value" ],
        "resultAlias":"activeController",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      },{
        "obj" : "java.lang:type=OperatingSystem",
        "attr" : [ "FreePhysicalMemorySize","SystemCpuLoad","ProcessCpuLoad","SystemLoadAverage" ],
        "resultAlias":"jvmMemory",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      } ,{
        "obj" : "kafka.network:type=SocketServer,name=NetworkProcessorAvgIdlePercent",
        "attr" : [ "Value" ],
        "resultAlias":"network",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      },{
        "obj" : "kafka.server:type=KafkaRequestHandlerPool,name=RequestHandlerAvgIdlePercent",
        "attr" : [ "MeanRate","OneMinuteRate","FiveMinuteRate","FifteenMinuteRate" ],
        "resultAlias":"network",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      },{
        "obj" : "java.lang:type=GarbageCollector,name=G1 Young Generation",
        "attr" : [ "CollectionCount","CollectionTime" ],
        "resultAlias":"gc",
        "outputWriters" : [ {
          "@class" : "com.googlecode.jmxtrans.model.output.InfluxDbWriterFactory",
          "url" : "http://192.168.0.105:8086/",
          "username" : "admin",
          "password" : "123456",
          "database" : "jmx",
          "tags"     : {"application" : "kafka_server"}
        } ]
      }]
    } ]
  }
  ```

  

### 4.4、JMXTrans部署
JMX通过网络连接，因此JMXtrans有2种部署方案：
1. 集中式。在一台服务器上部署JMXtrans，分别连接所有的Kafka Broker实例，并将数据写入到InfluxDB。为了减少网络传输，通常部署到InfluxDB所在服务器上。
2. 分布式。每个Kafka Broker实例部署一个JMXtrans。

JMXTrans配置文件分全局指标（每个Kafka节点）和Topic指标，全局指标是每个节点一个配置文件，命名规则：kafka-brokerxx.json，Topic指标是每个Topic一个配置文件，命名规则：TopicName.json。



## 五、Kafka监控方案实例
### 5.1、Kafka监控架构方案选择
监控系统架构通常分为三部分：数据采集、分析与转换、数据展示(可视化)。
1. 数据采集
数据采集通常先开发数据采集程序，然后使用Nagios、Zabbix等监控软件来调度执行，并将采集到的数据进行上报。对于Java程序，可以使用JMXTrans采集数据。
2. 分析与转换
Kafka是Java应用程序，所提供的性能指标数据已经非常全面，指标的直方图、次数、最大最小、标准方差都已经计算好，因此不需要再对数据进行分析加工，直接将MBeans数据存储到InfluxDB。
3. 数据可视化
Grafana是一个开源的可视化面板（Dashboard），支持Graphite、Zabbix、InfluxDB、Prometheus和OpenTSDB作为数据源。

### 5.2、InfluxDB部署
- InfluxDB是一款用Go语言编写的开源分布式时序、事件和指标数据库，无需外部依赖，主要用于存储涉及大量的时间戳数据，如DevOps监控数据、APP metrics、lOT传感器数据和实时分析数据。
  
  ```
  docker pull influxdb
  ```
  
  
- influxdb.yml文件：
  
  ```yaml
  version: '2'
  services:
    influxdb:
      image: influxdb
      container_name: influxdb
      volumes:
        - /data/influxdb/conf:/etc/influxdb
        - /data/influxdb/data:/var/lib/influxdb/data
        - /data/influxdb/meta:/var/lib/influxdb/meta
        - /data/influxdb/wal:/var/lib/influxdb/wal
      ports:
        - "8086:8086"
      restart: always
  ```
  
- 结果查看：
  
  ```
  docker exec -it influxdb influx
  ```
  
  

### 5.3、JMXTrans部署
- JMXTrans是一个通过JMX采集Java应用程序的数据采集器，只要Java应用程序开启JMX端口，就可以进行采集。

  ```
  docker pull jmxtrans/jmxtrans
  ```



- JMXTrans默认读取/var/lib/jmxtrans目录下所有数据源配置文件(json格式文件)，实时从数据源中获取数据，解析数据后存储到InfluxDB中。

  ```yaml
  version: '2'
  services:
    # JMXTrans服务
    jmxtrans:
      image: jmxtrans/jmxtrans
      container_name: jmxtrans
      volumes:
        - ./jmxtrans:/var/lib/jmxtrans
  ```

### 5.4、Grafana部署
Grafana是一个可视化面板（Dashboard），有非常漂亮的图表和布局展示，功能齐全的度量仪表盘和图形编辑器，支持Graphite、zabbix、InfluxDB、Prometheus和OpenTSDB作为数据源。
Grafana主要特性如下：

1. 展示方式：快速灵活的客户端图表，面板插件有许多不同方式的可视化指标和日志，官方库中具有丰富的仪表盘插件，比如热图、折线图、图表等多种展示方式。
2. 数据源：Graphite，InfluxDB，OpenTSDB，Prometheus，Elasticsearch，CloudWatch和KairosDB等。
3. 通知提醒：以可视方式定义最重要指标的警报规则，Grafana将不断计算并发送通知，在数据达到阈值时通过Slack、PagerDuty等获得通知。
4. 混合展示：在同一图表中混合使用不同的数据源，可以基于每个查询指定数据源，甚至自定义数据源。
5. 注释：使用来自不同数据源的丰富事件注释图表，将鼠标悬停在事件上会显示完整的事件元数据和标记。
6. 过滤器：Ad-hoc过滤器允许动态创建新的键/值过滤器，这些过滤器会自动应用于使用该数据源的所有查询。



- GitHub地址： https://github.com/grafana/grafanaGrafana容器镜像下载：
  
   ```
   docker pull grafana/grafana:6.5.0
   ```
   
   
   
- Grafana容器启动：
  
   ```
   docker run -d --name=grafana -p 3000:3000 grafana/grafana:6.5.0
   ```
   
- Web登录:192.168.0.105:3000

   ![Kafka快速入门（七）——Kafka集群监控_Kafka 监控_09](https://s2.51cto.com/images/blog/202005/25/fe77f15871a152c30ee00c8ad9c3c107.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

   初次登录默认使用admin/admin登录，登录后会强制要求修改密码。
   增加数据源：

   ![Kafka快速入门（七）——Kafka集群监控_Kafka 监控_10](https://s2.51cto.com/images/blog/202005/25/5d3fec54b81df885b3b4463cb47f5581.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

   导入DashBoard模板：

   ![Kafka快速入门（七）——Kafka集群监控_Kafka 监控_11](https://s2.51cto.com/images/blog/202005/25/a08188f81e8d14a8dda8fb0f82a42ddf.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)

   DashBoard模板json文件如下：

   ```yaml
  {
  "__inputs": [
    {
      "name": "DS_KAFKAMONITOR",
      "label": "KafkaMonitor",
      "description": "",
      "type": "datasource",
      "pluginId": "influxdb",
      "pluginName": "InfluxDB"
    }
  ],
  "__requires": [
    {
      "type": "grafana",
      "id": "grafana",
      "name": "Grafana",
      "version": "6.7.3"
    },
    {
      "type": "panel",
      "id": "graph",
      "name": "Graph",
      "version": ""
    },
    {
      "type": "datasource",
      "id": "influxdb",
      "name": "InfluxDB",
      "version": "1.0.0"
    }
  ],
  "annotations": {
    "list": [
      {
        "$$hashKey": "object:318",
        "builtIn": 1,
        "datasource": "-- Grafana --",
        "enable": true,
        "hide": true,
        "iconColor": "rgba(0, 211, 255, 1)",
        "name": "Annotations & Alerts",
        "type": "dashboard"
      }
    ]
  },
  "editable": true,
  "gnetId": null,
  "graphTooltip": 0,
  "id": null,
  "links": [],
  "panels": [
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "java.lang:type=OperatingSystem",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 12,
        "w": 8,
        "x": 0,
        "y": 0
      },
      "hiddenSeries": false,
      "id": 6,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "jvmMemory",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "A",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "ProcessCpuLoad"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "进程CPU使用率"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": []
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Kafka进程CPU使用率",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:1134",
          "format": "percentunit",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:1135",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "服务器CPU使用率",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 12,
        "w": 8,
        "x": 8,
        "y": 0
      },
      "hiddenSeries": false,
      "id": 2,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "jvmMemory",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "A",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "SystemCpuLoad"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "CPU使用率"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": []
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "CPU使用率",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:369",
          "format": "percentunit",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:370",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "java.lang:type=OperatingSystem\nLinux系统负载",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 12,
        "w": 8,
        "x": 16,
        "y": 0
      },
      "hiddenSeries": false,
      "id": 4,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": true,
        "min": false,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "jvmMemory",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "A",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "SystemLoadAverage"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "系统负载"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": []
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "系统负载",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:656",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:657",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "Kafka每个broker每秒中的数据量，包括__consumer_offsets topic",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 12,
        "w": 8,
        "x": 0,
        "y": 12
      },
      "hiddenSeries": false,
      "id": 34,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            }
          ],
          "hide": false,
          "measurement": "kafkaServer",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "D",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "OneMinuteRate"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "平均每秒"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": [
            {
              "key": "typeName",
              "operator": "=",
              "value": "type=BrokerTopicMetrics,name=MessagesInPerSec"
            }
          ]
        },
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            }
          ],
          "hide": false,
          "measurement": "kafkaServer",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "A",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "OneMinuteRate"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "sum"
              },
              {
                "params": [
                  "所有broker平均每秒"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": [
            {
              "key": "typeName",
              "operator": "=",
              "value": "type=BrokerTopicMetrics,name=MessagesInPerSec"
            }
          ]
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Kafka Topic 每秒数据量",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:2118",
          "format": "none",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:2119",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "java.lang:type=OperatingSystem\n服务器可用物理内存",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 12,
        "w": 8,
        "x": 8,
        "y": 12
      },
      "hiddenSeries": false,
      "id": 32,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "jvmMemory",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "A",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "FreePhysicalMemorySize"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "系统剩余物理内存"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": []
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "可用物理内存",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:2324",
          "format": "decbytes",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:2325",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "cacheTimeout": null,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "kafka.controller:type=KafkaController,name=ActiveControllerCount\n\nKafka控制器数量，每个集群只有一台机器为1，为1的机器是Kafka控制器Crontroller",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 12,
        "w": 8,
        "x": 16,
        "y": 12
      },
      "hiddenSeries": false,
      "id": 26,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": false,
        "min": false,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pluginVersion": "6.7.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            }
          ],
          "measurement": "activeController",
          "orderByTime": "ASC",
          "policy": "default",
          "query": "SELECT sum(\"Value\") AS \"获取控制器数量\" FROM \"activeController\" WHERE $timeFilter GROUP BY time($__interval), \"hostname\"",
          "rawQuery": false,
          "refId": "A",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "Value"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "获取控制器数量"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": [],
          "tz": ""
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Kafka控制器数量",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:4446",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:4447",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "监控 kafka.server:type=BrokerTopicMetrics,name=BytesOutPerSec 指标",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 9,
        "w": 8,
        "x": 0,
        "y": 24
      },
      "hiddenSeries": false,
      "id": 16,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "kafkaServer",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "A",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "FiveMinuteRate"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "mean"
              },
              {
                "params": [
                  "每秒拉取字节数"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": [
            {
              "key": "typeName",
              "operator": "=",
              "value": "type=BrokerTopicMetrics,name=BytesOutPerSec"
            }
          ]
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Kafka每秒拉取流量",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:77",
          "format": "decbytes",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:78",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "监控 kafka.server:type=BrokerTopicMetrics,name=BytesInPerSec 指标",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 9,
        "w": 8,
        "x": 8,
        "y": 24
      },
      "hiddenSeries": false,
      "id": 14,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "kafkaServer",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "F",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "OneMinuteRate"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "平均每秒进入字节数"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": [
            {
              "key": "typeName",
              "operator": "=",
              "value": "type=BrokerTopicMetrics,name=BytesInPerSec"
            }
          ]
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Kafka每秒进入流量",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:77",
          "format": "decbytes",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:78",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "监控 kafka.server:type=BrokerTopicMetrics,name=TotalFetchRequestsPerSec 和 kafka.server:type=BrokerTopicMetrics,name=TotalProduceRequestsPerSec 指标",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 9,
        "w": 8,
        "x": 16,
        "y": 24
      },
      "hiddenSeries": false,
      "id": 20,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "kafkaServer",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "A",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "OneMinuteRate"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "每秒Fetch(获取)的请求数量"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": [
            {
              "key": "typeName",
              "operator": "=",
              "value": "type=BrokerTopicMetrics,name=TotalFetchRequestsPerSec"
            }
          ]
        },
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "kafkaServer",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "D",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "MeanRate"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "每秒Producer发送的请求数量"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": [
            {
              "key": "typeName",
              "operator": "=",
              "value": "type=BrokerTopicMetrics,name=TotalProduceRequestsPerSec"
            }
          ]
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Kafka生产、消费每秒请求数量",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:77",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:78",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "java.lang:type=Memory",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 13,
        "w": 8,
        "x": 0,
        "y": 33
      },
      "hiddenSeries": false,
      "id": 8,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "jvmMemory",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "E",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "HeapMemoryUsage_used"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "堆内存使用"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": []
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Kafka使用堆内存",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:1850",
          "format": "decbytes",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:1851",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "java.lang:type=Memory",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 13,
        "w": 8,
        "x": 8,
        "y": 33
      },
      "hiddenSeries": false,
      "id": 30,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "jvmMemory",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "E",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "NonHeapMemoryUsage_used"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "对外内存使用"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": []
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Kafka使用堆外内存",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:1850",
          "format": "decbytes",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:1851",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "kafka.server:type=ReplicaManager,name=UnderReplicatedPartitions\n不为0则说明有的副本跟不上leader",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 13,
        "w": 8,
        "x": 16,
        "y": 33
      },
      "hiddenSeries": false,
      "id": 24,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pluginVersion": "6.7.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "underReplicated",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "A",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "Value"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "未充分备份的分区数"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": []
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "未充分备份的分区数监控",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:11235",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:11236",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "cacheTimeout": null,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 13,
        "w": 8,
        "x": 0,
        "y": 46
      },
      "hiddenSeries": false,
      "id": 12,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pluginVersion": "6.7.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "5m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "network",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "A",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "Value"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "mean"
              },
              {
                "params": [
                  "网络线程池空闲比例"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": []
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Kafka网络线程池线程平均的空闲比例",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:13734",
          "format": "percentunit",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:13735",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "cacheTimeout": null,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "kafka.server:type=KafkaRequestHandlerPool,name=RequestHandlerAvgIdlePercent",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 13,
        "w": 8,
        "x": 8,
        "y": 46
      },
      "hiddenSeries": false,
      "id": 22,
      "legend": {
        "alignAsTable": true,
        "avg": false,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "links": [],
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pluginVersion": "6.7.3",
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            }
          ],
          "measurement": "network",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "A",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "OneMinuteRate"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "IO空闲比例"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": [
            {
              "key": "typeName",
              "operator": "=",
              "value": "type=KafkaRequestHandlerPool,name=RequestHandlerAvgIdlePercent"
            }
          ]
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": " I/O 线程池线程平均的空闲比例",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:13517",
          "format": "percentunit",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:13518",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    },
    {
      "aliasColors": {},
      "bars": false,
      "dashLength": 10,
      "dashes": false,
      "datasource": "${DS_KAFKAMONITOR}",
      "description": "监控 kafka.server:type=BrokerTopicMetrics,name=FailedFetchRequestsPerSec 和 kafka.server:type=BrokerTopicMetrics,name=TotalFetchRequestsPerSec 指标",
      "fill": 1,
      "fillGradient": 0,
      "gridPos": {
        "h": 13,
        "w": 8,
        "x": 16,
        "y": 46
      },
      "hiddenSeries": false,
      "id": 18,
      "legend": {
        "alignAsTable": true,
        "avg": true,
        "current": true,
        "max": true,
        "min": true,
        "show": true,
        "total": false,
        "values": true
      },
      "lines": true,
      "linewidth": 1,
      "nullPointMode": "null",
      "options": {
        "dataLinks": []
      },
      "percentage": false,
      "pointradius": 2,
      "points": false,
      "renderer": "flot",
      "seriesOverrides": [],
      "spaceLength": 10,
      "stack": false,
      "steppedLine": false,
      "targets": [
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "kafkaServer",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "H",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "OneMinuteRate"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "每秒Fetch(获取)异常的请求"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": [
            {
              "key": "typeName",
              "operator": "=",
              "value": "type=BrokerTopicMetrics,name=FailedFetchRequestsPerSec"
            }
          ]
        },
        {
          "alias": "",
          "groupBy": [
            {
              "params": [
                "1m"
              ],
              "type": "time"
            },
            {
              "params": [
                "hostname"
              ],
              "type": "tag"
            },
            {
              "params": [
                "null"
              ],
              "type": "fill"
            }
          ],
          "measurement": "kafkaServer",
          "orderByTime": "ASC",
          "policy": "default",
          "refId": "J",
          "resultFormat": "time_series",
          "select": [
            [
              {
                "params": [
                  "MeanRate"
                ],
                "type": "field"
              },
              {
                "params": [],
                "type": "last"
              },
              {
                "params": [
                  "每秒Producer异常的请求"
                ],
                "type": "alias"
              }
            ]
          ],
          "tags": [
            {
              "key": "typeName",
              "operator": "=",
              "value": "type=BrokerTopicMetrics,name=FailedProduceRequestsPerSec"
            }
          ]
        }
      ],
      "thresholds": [],
      "timeFrom": null,
      "timeRegions": [],
      "timeShift": null,
      "title": "Kafka生产、消费请求失败数量",
      "tooltip": {
        "shared": true,
        "sort": 0,
        "value_type": "individual"
      },
      "type": "graph",
      "xaxis": {
        "buckets": null,
        "mode": "time",
        "name": null,
        "show": true,
        "values": []
      },
      "yaxes": [
        {
          "$$hashKey": "object:77",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        },
        {
          "$$hashKey": "object:78",
          "format": "short",
          "label": null,
          "logBase": 1,
          "max": null,
          "min": null,
          "show": true
        }
      ],
      "yaxis": {
        "align": false,
        "alignLevel": null
      }
    }
  ],
  "refresh": false,
  "schemaVersion": 22,
  "style": "dark",
  "tags": [],
  "templating": {
    "list": []
  },
  "time": {
    "from": "now-1h",
    "to": "now"
  },
  "timepicker": {
    "refresh_intervals": [
      "5s",
      "10s",
      "30s",
      "1m",
      "5m",
      "15m",
      "30m",
      "1h",
      "2h",
      "1d"
    ]
  },
  "timezone": "",
  "title": "Kafka集群监控模板",
  "uid": "PkULDneZkALL",
  "variables": {
    "list": []
  },
  "version": 27
  }
  ```


### 5.5、docker-compose.yml文件
将InfluxDB、JMXTrans、Grafana部署整合使用Docker-Compose进行部署，创建KafkaMonitor目录，在KafkaMonitor目录内创建influxdb目录和jmxtrans目录以及docker-compose.yml文件，将jmxtrans.json文件放到jmxtrans目录。

- docker-compose.yml文件如下：

  ```yaml
version: '2'
services:
  # JMXTrans服务
  jmxtrans:
    image: jmxtrans/jmxtrans
    container_name: jmxtrans
    volumes:
      - ./jmxtrans:/var/lib/jmxtrans
  # InfluxDB服务
  influxdb:
    image: influxdb
    container_name: influxdb
    volumes:
      - ./influxdb/conf:/etc/influxdb
      - ./influxdb/data:/var/lib/influxdb/data
      - ./influxdb/meta:/var/lib/influxdb/meta
      - ./influxdb/wal:/var/lib/influxdb/wal
    ports:
      - "8086:8086" # 对外暴露端口，提供Grafana访问
    restart: always
  # Grafana服务
  grafana:
    image: grafana/grafana:6.5.0  #高版本可能存在bug
    container_name: grafana
    ports:
      - "3000:3000"  # 对外暴露端口，提供web访问
  ```

- 启动监控框架服务:

  ```
  docker-compose -f docker-compose.yml up -d
  ```


  需要Web登录Grafana服务，配置相应的数据源和模板。

### 5.6、监控查看

- 项目GitHub地址： https://github.com/scorpiostudio/Kafka

  ![Kafka快速入门（七）——Kafka集群监控_Kafka 监控_12](https://s2.51cto.com/images/blog/202005/25/4dc1d672b66598c834db54ccc2e80be2.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_30,g_se,x_10,y_10,shadow_20,type_ZmFuZ3poZW5naGVpdGk=)



https://blog.51cto.com/quantfabric/2498434
