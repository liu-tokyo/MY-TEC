# JMS 规范是什么 

## 1、JMS 的基础 
JMS 是什么：JMS 是 Java 提供的一套技术规范，即 Java 消息服务（Java Message Service） 应用程序接口。是一个 Java 平台中关于面向消息中间件的 API。用于在两个应用程序之间或 分布式系统中发送消息，进行异步通信。Java 消息服务是一个与具体平台无关的 API。 

JMS 干什么用：用来异构系统集成通信，缓解系统瓶颈，提高系统的伸缩性增强系统用户体 验，使得系统模块化和组件化变得可行并更加灵活 

通过什么方式：生产消费者模式（生产者、服务器、消费者）通常消息传递有两种类型的消 息模式可用: 

- 一种是点对点 queue 队列模式(p2p)；

- 一种是 topic 发布-订阅模式(public-subscribe)。 

## 2、JMS 消息传输模型 

点对点模式（一对一，消费者主动拉取数据，消息收到后消息清除） 

点对点模型通常是一个基于拉取或者轮询的消息传送模型，这种模型从队列中请求信息，而 不是将消息推送到客户端。这个模型的特点是发送到队列的消息被一个且只有一个接收者接 收处理，即使有多个消息监听者也是如此。 

![img](https://img-blog.csdnimg.cn/20190206033336510.png)

发布/订阅模式（一对多，数据生产后，推送给所有订阅者） 

发布订阅模型则是一个基于推送的消息传送模型。发布订阅模型可以有多种不同的订阅者， 临时订阅者只在主动监听主题时才接收消息，而持久订阅者则监听主题的所有消息，即时当 前订阅者不可用，处于离线状态。

![img](https://img-blog.csdnimg.cn/20190206035559245.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hpYW9kdW5MUA==,size_16,color_FFFFFF,t_70)

queue.put(object)   数据生产

queue.take(object)   数据消费 

## 3、JMS 核心组件 

Destination：消息发送的目的地，也就是前面说的 Queue 和 Topic

Message：从字面上就可以看出是被发送的消息

Producer：消息的生产者，要发送一个消息，必须通过这个生产者来发送

MessageConsumer：与生产者相对应，这是消息的消费者或接收者，通过它来接收一个消息 
![img](https://img-blog.csdnimg.cn/20190206042150849.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hpYW9kdW5MUA==,size_16,color_FFFFFF,t_70)

通过与 ConnectionFactory 可以获得一个 connection

通过 connection 可以获得一个 session 会话  

补充 Message 的分类：

StreamMessage：Java 数据流消息，用标准流操作来顺序的填充和读取

MapMessage：一个 Map 类型的消息；名称为 string 类型，而值为 Java 的基本类型

TextMessage：普通字符串消息，包含一个 String

ObjectMessage：对象消息，包含一个可序列化的 Java 对象

BytesMessage：二进制数组消息，包含一个 byte[]

XMLMessage:  一个 XML 类型的消息

最常用的是 TextMessage 和 ObjectMessage


## 4、常见的类 JMS 消息服务器 
### 4.1、JMS 消息服务器 ActiveMQ 

ActiveMQ 是 Apache 出品，最流行的，能力强劲（？）的开源消息总线。ActiveMQ 是一个完全支 持 JMS1.1 和 J2EE 1.4 规范的。

主要特点： 

- 多种语言和协议编写客户端。语言: Java, C, C++, C#, Ruby, Perl, Python, PHP。应用协议: OpenWire，Stomp REST，WS Notification，XMPP，AMQP

- 完全支持 JMS1.1 和 J2EE 1.4 规范 (持久化,XA 消息,事务)

- 对 Spring 的支持，ActiveMQ 可以很容易内嵌到使用 Spring 的系统里面去,而且也支持 Spring2.0 的特性

- 通过了常见 J2EE 服务器(如 Geronimo，JBoss 4，GlassFish，WebLogic)的测试,其中通过 JCA 1.5 resource adaptors 的配置，可以让 ActiveMQ 可以自动的部署到任何兼容 J2EE 1.4 商业服务器上 

- 支持多种传送协议：in-VM，TCP，SSL，NIO，UDP，JGroups，JXTA

- 支持通过 JDBC 和 journal 提供高速的消息持久化

- 从设计上保证了高性能的集群,客户端-服务器，点对点

- 支持 Ajax

- 支持与 Axis 的整合

- 可以很容易得调用内嵌 JMS provider 进行测试 

### 4.2、分布式消息中间件 Metamorphosis 

Metamorphosis(MetaQ) 是一个高性能、高可用、可扩展的分布式消息中间件，类似于 LinkedIn 的 Kafka，具有消息存储顺序写、吞吐量大和支持本地和 XA 事务等特性，适用于大吞吐量、 顺序消息、广播和日志数据传输等场景，在淘宝和支付宝有着广泛的应用，现已开源。 

![img](https://img-blog.csdnimg.cn/20190206042343903.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hpYW9kdW5MUA==,size_16,color_FFFFFF,t_70)



### 4.3、分布式消息中间件 RocketMQ 

RocketMQ 是一款分布式、队列模型的消息中间件 

![img](https://img-blog.csdnimg.cn/20190206042402579.png)



### 4.4、其他 MQ 

![img](https://img-blog.csdnimg.cn/20190206042412776.png)

为什么没有我所熟知的 **IBM MQ**、**Oracle AQ***(AdancedQueuing) ？不过感觉还是Kafka最流行。

---

原文链接：https://blog.csdn.net/XiaodunLP/article/details/86767341