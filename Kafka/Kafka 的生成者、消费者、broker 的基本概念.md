# Kafka 的生成者、消费者、broker 的基本概念

kafka是一款基于发布与订阅的消息系统。它一般被称为“分布式提交日志”或者“分布式流平台”。文件系统或者数据库提交日志用来提供所有事物的持久化记录，通过重建这些日志可以重建系统的状态。同样地，kafka的数据是按照一定顺序持久化保存的，可以按需读取。





# 1、kafka拓扑结构



![img](https://pics5.baidu.com/feed/63d0f703918fa0ecff35229350f816e73c6ddb33.jpeg?token=038692feaa82d68e7c48c31300050d4e)



# 2、Kafka的特点





同时为分布和订阅提供高吞吐量。据了解，Kafka每秒可以生产约25万条消息（50MB），每秒处理55万条消息（110MB）这里说条数，可能不上特别准确，因为消息的大小可能不一致；





可进行持久化操作，将消息持久化到到磁盘，以日志的形式存储，因此可用于批量消费，例如ETL，以及实时应用程序。 通过将数据持久化到硬盘以及replication防止数据丢失。
分布式系统，易于向外拓展。所有的Producer、broker和consumer都会有多个，均为分布式。无需停机即可拓展机器。





消息被处理的状态是在consumer端维护，而不是由server端维护，当失败时能自动平衡。
支持Online和offline的场景。





# 3、Kafka的核心概念





名词 解释
Producer 消息的生成者
Consumer 消息的消费者
ConsumerGroup 消费者组，可以并行消费Topic中的partition的消息
Broker 缓存代理，Kafka集群中的一台或多台服务器统称broker.
Topic Kafka处理资源的消息源(feeds of messages)的不同分类
Partition Topic物理上的分组，一个topic可以分为多个partion,每个partion是一个有序的队列。partion中每条消息都会被分 配一个 有序的Id(offset)
Message 消息，是通信的基本单位，每个producer可以向一个topic（主题）发布一些消息
Producers 消息和数据生成者，向Kafka的一个topic发布消息的 过程叫做producers
Consumers 消息和数据的消费者，订阅topic并处理其发布的消费过程叫做consumers





- **3.1 Producers的概念**

  消息和数据生成者，向Kafka的一个topic发布消息的过程叫做producers
  Producer将消息发布到指定的Topic中，同时Producer也能决定将此消息归属于哪个partition；比如基于round-robin方式 或者通过其他的一些算法等；
  异步发送批量发送可以很有效的提高发送效率。kafka producer的异步发送模式允许进行批量发送，先将消息缓存到内存中，然后一次请求批量发送出去。





- **3.2 broker的概念:**

  Broker没有副本机制，一旦broker宕机，该broker的消息将都不可用。
  Broker不保存订阅者的状态，由订阅者自己保存。
  无状态导致消息的删除成为难题（可能删除的消息正在被订阅），Kafka采用基于时间的SLA（服务保证），消息保存一定时间（通常7天）后会删除。
  消费订阅者可以rewind back到任意位置重新进行消费，当订阅者故障时，可以选择最小的offset(id)进行重新读取消费消息





- **3.3 Message组成**

  Message消息：是通信的基本单位，每个producer可以向一个topic发布消息。
  Kafka中的Message是以topic为基本单位组织的，不同的topic之间是相互独立的，每个topic又可以分成不同的partition每个partition储存一部分
  partion中的每条Message包含以下三个属性：
  offset long
  MessageSize int32
  data messages的具体内容





- **3.4 Consumers的概念**

  消息和数据消费者，订阅topic并处理其发布的消息的过程叫做consumers. 在kafka中，我们可以认为一个group是一个“订阅者”，一个topic中的每个partions只会被一个“订阅者”中的一个consumer 消费，不过一个consumer可以消费多个partitions中的消息 注: Kafka的设计原理决定，对于一个topic，同一个group不能多于partition个数的consumer同时消费，否则将意味着某些 consumer无法得到消息



**原文链接：http://click.aliyun.com/m/1000302400/**