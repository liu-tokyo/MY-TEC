# JMS - Java消息服务

**Java消息服务**（**Java Message Service**，**JMS**）[应用程序接口](https://zh.wikipedia.org/wiki/应用程序接口)是一个[Java](https://zh.wikipedia.org/wiki/Java)平台中关于[面向消息中间件](https://zh.wikipedia.org/w/index.php?title=面向消息中间件&action=edit&redlink=1)（MOM）的[API](https://zh.wikipedia.org/wiki/API)，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。Java消息服务是一个与具体平台无关的API，绝大多数MOM提供商都对JMS提供支持。

Java消息服务的规范包括两种消息模式，点对点和发布者／订阅者。许多提供商支持这一通用框架因此，程序员可以在他们的分布式软件中实现面向消息的操作，这些操作将具有不同面向消息中间件产品的可移植性。

Java消息服务支持同步和异步的消息处理，在某些场景下，同步消息是必要的；在其他场景下，异步消息比同步消息操作更加便利。

Java消息服务支持面向事件的方法接收消息，[事件驱动的程序设计](https://zh.wikipedia.org/wiki/事件驅動程式設計)现在被广泛认为是一种富有成效的[程序设计范例](https://zh.wikipedia.org/wiki/编程范型)，程序员们都相当熟悉。

在应用系统开发时，Java消息服务可以推迟选择面对消息中间件产品，也可以在不同的面对消息中间件切换。

## 历史

Java消息服务是一个在 [Java标准化组织](https://zh.wikipedia.org/w/index.php?title=Java标准化组织&action=edit&redlink=1)（[JCP](https://zh.wikipedia.org/wiki/JCP)）内开发的标准（代号JSR 914）。2001年6月25日，Java消息服务发布JMS 1.0.2b，2002年3月18日Java消息服务发布 1.1，统一了消息域。

## 体系架构

### JMS元素

JMS由以下元素组成。

- JMS提供者

  连接面向消息中间件的，JMS接口的一个实现。提供者可以是Java平台的JMS实现，也可以是非Java平台的面向消息中间件的适配器。

- JMS客户

  生产或消费消息的基于Java的应用程序或对象。

- JMS生产者

  创建并发送消息的JMS客户。

- JMS消费者

  接收消息的JMS客户。

- JMS消息

  包括可以在JMS客户之间传递的数据的对象

- JMS队列

  一个容纳那些被发送的等待阅读的消息的区域。队列暗示，这些消息将按照顺序发送。一旦一个消息被阅读，该消息将被从队列中移走。

- JMS主题

  一种支持发送消息给多个订阅者的机制。

### JMS模型

Java消息服务应用程序结构支持两种模型：

- [点对点](https://zh.wikipedia.org/wiki/点对点)或队列模型
- [发布/订阅](https://zh.wikipedia.org/wiki/发布/订阅)模型

在点对点或队列模型下，一个生产者向一个特定的队列发布消息，一个消费者从该队列中读取消息。这里，生产者知道消费者的队列，并直接将消息发送到消费者的队列。这种模式被概括为：

- 只有一个消费者将获得消息
- 生产者不需要在接收者消费该消息期间处于运行状态，接收者也同样不需要在消息发送时处于运行状态。
- 每一个成功处理的消息都由接收者签收

发布者／订阅者模型支持向一个特定的消息主题发布消息。0或多个订阅者可能对接收来自特定消息主题的消息感兴趣。在这种模型下，发布者和订阅者彼此不知道对方。这种模式好比是匿名公告板。这种模式被概括为：

- 多个消费者可以获得消息
- 在发布者和订阅者之间存在时间依赖性。发布者需要创建一个订阅（subscription），以便客户能够订阅。订阅者必须保持持续的活动状态以接收消息，除非订阅者创建了持久的订阅。在那种情况下，在订阅者未连接时发布的消息将在订阅者重新连接时重新发布。

使用Java语言，JMS提供了将应用与提供数据的传输层相分离的方式。同一组Java[类](https://zh.wikipedia.org/wiki/类)可以通过[JNDI](https://zh.wikipedia.org/w/index.php?title=Java命名和目录接口&action=edit&redlink=1)中关于提供者的信息，连接不同的JMS提供者。这一组类首先使用一个连接工厂以连接到队列或主题，然后发送或发布消息。在接收端，客户接收或订阅这些消息。

## JMS应用程序接口

Java消息服务的API在`javax.jms`包中提供。

### `ConnectionFactory` [接口](https://zh.wikipedia.org/wiki/接口)（连接工厂）

用户用来创建到JMS提供者的连接的被管对象。JMS客户通过可移植的接口访问连接，这样当下层的实现改变时，代码不需要进行修改。 管理员在JNDI名字空间中配置连接工厂，这样，JMS客户才能够查找到它们。根据消息类型的不同，用户将使用队列连接工厂，或者主题连接工厂。

### `Connection` 接口（连接）

连接代表了应用程序和消息服务器之间的通信链路。在获得了连接工厂后，就可以创建一个与JMS提供者的连接。根据不同的连接类型，连接允许用户创建会话，以发送和接收队列和主题到目标。

### `Destination` 接口（目标）

目标是一个包装了消息目标标识符的被管对象，消息目标是指消息发布和接收的地点，或者是队列，或者是主题。JMS管理员创建这些对象，然后用户通过JNDI发现它们。和连接工厂一样，管理员可以创建两种类型的目标，点对点模型的队列，以及发布者／订阅者模型的主题。

### `MessageConsumer` 接口（消息消费者）

由会话创建的对象，用于接收发送到目标的消息。消费者可以同步地（阻塞模式），或异步（非阻塞）接收队列和主题类型的消息。

### `MessageProducer` 接口（消息生产者）

由会话创建的对象，用于发送消息到目标。用户可以创建某个目标的发送者，也可以创建一个通用的发送者，在发送消息时指定目标。

### `Message` 接口（消息）

是在消费者和生产者之间传送的对象，也就是说从一个应用程序创送到另一个应用程序。一个消息有三个主要部分：

1. 消息头（必须）：包含用于识别和为消息寻找路由的操作设置。
2. 一组消息属性（可选）：包含额外的属性，支持其他提供者和用户的兼容。可以创建定制的字段和过滤器（消息选择器）。
3. 一个消息体（可选）：允许用户创建五种类型的消息（文本消息，映射消息，字节消息，流消息和对象消息）。

消息接口非常灵活，并提供了许多方式来定制消息的内容。

### `Session` 接口（会话）

表示一个单线程的上下文，用于发送和接收消息。由于会话是单线程的，所以消息是连续的，就是说消息是按照发送的顺序一个一个接收的。会话的好处是它支持事务。如果用户选择了事务支持，会话上下文将保存一组消息，直到事务被提交才发送这些消息。在提交事务之前，用户可以使用回滚操作取消这些消息。一个会话允许用户创建消息生产者来发送消息，创建消息消费者来接收消息。

## JMS提供者实现

要使用Java消息服务，你必须要有一个JMS提供者，管理会话和队列。现在既有[开源](https://zh.wikipedia.org/wiki/开源)的提供者也有专有的提供者。

开源的提供者包括：

- [Kafka](https://zh.wikipedia.org/wiki/Kafka)
- [Apache ActiveMQ](https://zh.wikipedia.org/wiki/Apache_ActiveMQ)
- [JBoss](https://zh.wikipedia.org/wiki/JBoss) 社区所研发的 [HornetQ](https://zh.wikipedia.org/wiki/HornetQ)
- [Joram](http://joram.objectweb.org/) （[页面存档备份](https://web.archive.org/web/20090323071620/http://joram.objectweb.org/)，存于[互联网档案馆](https://zh.wikipedia.org/wiki/互联网档案馆)）
- Coridan的[MantaRay](http://mantaray.coridan.com/) （[页面存档备份](https://web.archive.org/web/20201125230543/http://mantaray.coridan.com/)，存于[互联网档案馆](https://zh.wikipedia.org/wiki/互联网档案馆)）
- [The OpenJMS Group](http://openjms.sourceforge.net/) （[页面存档备份](https://web.archive.org/web/20201220050848/http://openjms.sourceforge.net/)，存于[互联网档案馆](https://zh.wikipedia.org/wiki/互联网档案馆)）的OpenJMS

专有的提供者包括：

- [BEA](https://zh.wikipedia.org/wiki/BEA)的BEA WebLogic Server JMS
- [TIBCO软件公司](https://zh.wikipedia.org/wiki/TIBCO软件公司)的[EMS](https://web.archive.org/web/20060315192527/http://www.tibco.com/software/enterprise_backbone/enterprisemessageservice.jsp)
- [GigaSpaces Technologies](http://www.gigaspaces.com/) （[页面存档备份](https://web.archive.org/web/20210126160246/http://www.gigaspaces.com/)，存于[互联网档案馆](https://zh.wikipedia.org/wiki/互联网档案馆)）的GigaSpaces
- [Softwired 2006](https://archive.is/20140608041926/http://www.softwired-inc.com/)的iBus
- [IONA Technologies](https://zh.wikipedia.org/w/index.php?title=IONA_Technologies&action=edit&redlink=1)的IONA JMS
- [SeeBeyond](http://www.seebeyond.com/) （[页面存档备份](https://web.archive.org/web/20081202040210/http://www.seebeyond.com/)，存于[互联网档案馆](https://zh.wikipedia.org/wiki/互联网档案馆)）的IQManager（2005年8月被[Sun Microsystems](https://zh.wikipedia.org/wiki/Sun_Microsystems)并购）
- [webMethods](https://zh.wikipedia.org/w/index.php?title=WebMethods&action=edit&redlink=1)的JMS+ - [http://www.webmethods.com](http://www.webmethods.com/) （[页面存档备份](https://web.archive.org/web/20081120212119/http://www.webmethods.com/)，存于[互联网档案馆](https://zh.wikipedia.org/wiki/互联网档案馆)）
- [my-channels](http://www.my-channels.com/) （[页面存档备份](https://web.archive.org/web/20191024093707/http://www.my-channels.com/)，存于[互联网档案馆](https://zh.wikipedia.org/wiki/互联网档案馆)）的Nirvana
- [Sonic Software](https://zh.wikipedia.org/w/index.php?title=Sonic_Software&action=edit&redlink=1)的SonicMQ
- [IBM](https://zh.wikipedia.org/wiki/IBM)的WebSphere MQ

一个详尽的JMS提供者对比矩阵可以在[这里](http://www.theserverside.com/reviews/matrix.tss) （[页面存档备份](https://web.archive.org/web/20080820014521/http://www.theserverside.com/reviews/matrix.tss)，存于[互联网档案馆](https://zh.wikipedia.org/wiki/互联网档案馆)）看到。

如果你计划在一个[服务器集群](https://zh.wikipedia.org/wiki/服务器集群)中运行你的程序，你需要检查提供者是否实现了对负载均衡和故障恢复的支持。

## 参见

- [J2EE连接器架构](https://zh.wikipedia.org/wiki/J2EE连接器架构)
- [J2EE](https://zh.wikipedia.org/wiki/J2EE)
- [JDBC](https://zh.wikipedia.org/wiki/JDBC)
- [面向消息中间件](https://zh.wikipedia.org/w/index.php?title=面向消息中间件&action=edit&redlink=1)
- [企业应用集成](https://zh.wikipedia.org/wiki/企业应用集成)

## 外部链接

- [Sun's JMS Overview](http://java.sun.com/products/jms/) （[页面存档备份](https://web.archive.org/web/20090911054525/http://java.sun.com/products/jms/)，存于[互联网档案馆](https://zh.wikipedia.org/wiki/互联网档案馆)）
- [JSR 914](http://www.jcp.org/en/jsr/detail?id=914) （[页面存档备份](https://web.archive.org/web/20201031231550/http://www.jcp.org/en/jsr/detail?id=914)，存于[互联网档案馆](https://zh.wikipedia.org/wiki/互联网档案馆)） (JMS 1.0 and 1.1)
- [GigaSpaces Technologies JavaSpaces approach to JMS](https://web.archive.org/web/20051216025932/http://www.gigaspaces.com/product_4.html)