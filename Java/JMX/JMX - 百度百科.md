# JMX - 百度百科

JMX（Java Management Extensions，即Java管理扩展）是一个为应用程序、设备、系统等[植入](https://baike.baidu.com/item/植入/7958584)管理功能的框架。JMX可以跨越一系列异构操作系统平台、[系统体系结构](https://baike.baidu.com/item/系统体系结构/6842760)和[网络传输协议](https://baike.baidu.com/item/网络传输协议/332131)，灵活的开发无缝集成的系统、网络和服务管理应用。

- 外文名  
  Java Management Extensions
  
- 简  称  
  JMX  
  
- 属  性  
  Java管理扩展
  
- 作  用  
  为应用程序、设备植入管理功能
  
- 特  点  
  JMX可以跨越异构操作系统平台
  
- 优  点  
非常容易的使应用程序具有被管理

## 目录
- [JMX - 百度百科](#jmx---百度百科)
  - [目录](#目录)
  - [JMX简介](#jmx简介)
  - [分层](#分层)
  - [设备层](#设备层)
    - [管理构件（MBean）](#管理构件mbean)
    - [通知模型](#通知模型)
    - [辅助元数据类](#辅助元数据类)
  - [代理层](#代理层)
    - [MBean服务器](#mbean服务器)
    - [协议适配器和连接器](#协议适配器和连接器)
    - [代理服务](#代理服务)
    - [JMX规范](#jmx规范)
  - [服务层](#服务层)
  - [管理协议](#管理协议)
  - [实现应用](#实现应用)
  - [小结](#小结)
  - [架构](#架构)

--- 

## JMX简介

JMX在Java编程语言中定义了应用程序以及网络管理和监控的[体系结构](https://baike.baidu.com/item/体系结构)、设计模式、[应用程序接口](https://baike.baidu.com/item/应用程序接口)以及服务。通常使用JMX来[监控系统](https://baike.baidu.com/item/监控系统)的运行状态或管理系统的某些方面，比如清空缓存、重新加载配置文件等

优点是可以非常容易的使应用程序被管理

伸缩性的架构使每个JMX Agent服务可以很容易的放入到Agent中，每个JMX的实现都提供几个核心的Agent服务，你也可以自己编写服务，服务可以很容易的部署，取消部署。

主要作用是提供接口，允许有不同的实现

## 分层

JMX体系结构分为以下四个层次：

1. 设备层

   设备层（Instrumentation Level）：主要定义了信息模型。在JMX中，各种管理对象以[管理构件](https://baike.baidu.com/item/管理构件)的形式存在，需要管理时，向MBean[服务器](https://baike.baidu.com/item/服务器)进行注册。该层还定义了通知机制以及一些辅助元数据类。

2. 代理层

   代理层（Agent Level）：主要定义了各种服务以及通信模型。该层的核心是一个MBean[服务器](https://baike.baidu.com/item/服务器)，所有的[管理构件](https://baike.baidu.com/item/管理构件)都需要向它注册，才能被管理。注册在MBean服务器上管理构件并不直接和远程应用程序进行通信，它们通过协议适配器和连接器进行通信。而协议适配器和连接器也以管理构件的形式向MBean服务器注册才能提供相应的服务。

3. 分布服务层

   分布服务层（Distributed Service Level）：主要定义了能对代理层进行操作的管理接口和构件，这样管理者就可以操作代理。然而，当前的[JMX规范](https://baike.baidu.com/item/JMX规范/14769057)并没有给出这一层的具体规范。

4. 附加管理协议API

   定义的API主要用来支持当前已经存在的[网络管理协议](https://baike.baidu.com/item/网络管理协议)，如SNMP、TMN、CIM/WBEM等。

## 设备层

该层定义了如何实现JMX管理资源的规范。一个JMX管理资源可以是一个Java应用、一个服务或一个设备，它们可以用Java开发，或者至少能用Java进行包装，并且能被置入JMX框架中，从而成为JMX的一个[管理构件](https://baike.baidu.com/item/管理构件)(Managed Bean)，[简称](https://baike.baidu.com/item/简称)MBean。管理构件可以是标准的，也可以是动态的，标准的管理构件遵从JavaBeans构件的设计模式；动态的管理构件遵从特定的接口，提供了更大的灵活性。

该层还定义了通知机制以及实现管理构件的辅助元数据类。

### 管理构件（MBean）

在[JMX规范](https://baike.baidu.com/item/JMX规范/14769057)中，[管理构件](https://baike.baidu.com/item/管理构件)定义如下：它是一个能代表管理资源的Java对象，遵从一定的设计模式，还需实现该规范定义的特定的接口。该定义了保证了所有的管理构件以一种标准的方式来表示被管理资源。

管理接口就是被管理资源暴露出的一些信息，通过对这些信息的修改就能控制被管理资源。一个管理构件的管理接口包括：

1. 能被接触的属性值；

2. 能够执行的操作；

3. 能发出的通知事件；

4. 管理构件的构建器。

管理构件通过公共的方法以及遵从特定的设计模式封装了属性和操作，以便暴露给[管理应用程序](https://baike.baidu.com/item/管理应用程序/16079363)。例如，一个只读属性在[管理构件](https://baike.baidu.com/item/管理构件)中只有Get方法，既有Get又有Set方法表示是一个可读写的属性。

其余的JMX的构件，例如JMX代理提供的各种服务，也是作为一个管理构件注册到代理中才能提供相应的服务。

JMX对管理构件的存储位置没有任何限制，管理构件可以存储在运行JMX代理的Java[虚拟机](https://baike.baidu.com/item/虚拟机)的类路径的任何位置，也可以从网络上的任何位置导入。

JMX定义了四种管理构件：标准、动态、开放和模型管理构件。每一种管理构件可以根据不同的环境需要进行制定。

- 1.标准管理构件

  标准管理构件的设计和实现是最简单的，它们的管理接口通过方法名来描述。标准[管理构件](https://baike.baidu.com/item/管理构件)的实现依靠一组命名规则，称之为设计模式。这些命名规则定义了属性和操作。检查标准管理构件接口和应用设计模式的过程被称为内省（Introspection）[22]。JMX代理通过内省来查看每一个注册在MBean [服务器](https://baike.baidu.com/item/服务器)上的管理构件的方法和[超类](https://baike.baidu.com/item/超类)，看它是否遵从一定设计模式，决定它是否代表了一个管理构件，并辨认出它的属性和操作。

- 2.动态管理构件

  动态管理构件提供了更大的灵活性，它可以在运行期暴露自己的管理接口。它的实现是通过实现一个特定的接口DynamicMBean。

  JMX代理通过getMBeanInfo方法来获取该动态[管理构件](https://baike.baidu.com/item/管理构件)暴露的管理接口，该方法返回的对象是MbeanInfo类的实例，包含了属性和操作的签名。由于该方法的调用是发生在动态管理构件向MBean[服务器](https://baike.baidu.com/item/服务器)注册以后，因此管理接口是在运行期获取的。不同于标准管理构件，JMX代理不需要通过内省机制来确定动态管理构件的管理接口。由于DynamicMBean的接口是不变的，因此可以屏蔽实现细节。由于这种在运行期获取管理接口的特性，动态管理构件提供了更大的灵活性。

- 3.开放管理构件

  开放管理构件是一种专门化的动态管理构件，其中所有的与该管理构件相关的参数、返回类型和属性都围绕一组预定义的数据类型（String、Integer、Float 等）来建立，并且通过一组特定的接口来进行自我描述。JMX代理通过获得一个OpenMBeanInfo对象来获取开放[管理构件](https://baike.baidu.com/item/管理构件)的管理接口，OpenMBeanInfo是MbeanInfo的子类。

- 4.模型管理构件

  模型管理构件也是一种专门化的动态管理构件。它是预制的、通用的和动态的 MBean 类，已经包含了所有必要缺省行为的实现，并允许在运行时添加或覆盖需要定制的那些实现。[JMX规范](https://baike.baidu.com/item/JMX规范/14769057)规定该类必须实现为[javax.management.modelmbean](https://baike.baidu.com/item/javax.management.modelmbean).RequiredModelMBean，管理者要做的就是实例化该类，并配置该构件的默认行为并注册到JMX代理中，即可实现对资源的管理。JMX代理通过获得一个ModelMBeanInfo对象来获取管理接口。

模型[管理构件](https://baike.baidu.com/item/管理构件)具有以下新的特点[23]：

1. 持久性

   定义了持久机制，可以利用Java的序列化或JDBC来存储模型MBean的状态。

2. 通知和日志功能

   能记录每一个发出的通知，并能自动发出属性变化通知。

3. 属性值缓存

   具有缓存属性值的能力。

### 通知模型

一个[管理构件](https://baike.baidu.com/item/管理构件)提供的管理接口允许代理对其管理资源进行控制和配置。然而，对管理复杂的[分布式系统](https://baike.baidu.com/item/分布式系统)来说，这些接口只是提供了一部分功能。通常，管理应用程序需要对状态变化或者当特别情况发生变化时作出反映。

为此，JMX定义了通知模型。通知模型仅仅涉及了在同一个JMX代理中的管理构件之间的事件传播。JMX通知模型依靠以下几个部分：

1. Notification，一个通用的事件类型，该类标识事件的类型，可以被直接使用，也可以根据传递的事件的需要而被扩展。

2. NotificationListener接口，接受通知的对象需实现此接口。

3. NotificationFilter接口，作为通知过滤器的对象需实现此接口，为通知监听者提供了一个过滤通知的过滤器。

4. NotificationBroadcaster接口，通知发送者需实现此接口，该接口允许希望得到通知的监听者注册。

发送一个通用类型的通知，任何一个监听者都会得到该通知。因此，监听者需提供过滤器来选择所需要接受的通知。

任何类型的[管理构件](https://baike.baidu.com/item/管理构件)，标准的或动态的，都可以作为一个通知发送者，也可以作为一个通知监听者，或两者都是。

### 辅助元数据类

辅助元数据类用来描述[管理构件](https://baike.baidu.com/item/管理构件)。辅助元数据类不仅被用来内省标准管理构件，也被动态管理构件用来进行自我描述。这些类根据属性、操作、构建器和通告描述了管理接口。JMX代理通过这些元数据类管理所有管理构件，而不管这些管理构件的类型。

部分辅助元类如下：

1. MBeanInfo--包含了属性、操作、构建器和通知的信息。

2. MBeanFeatureInfo--为下面类的超类。

3. MBeanAttributeInfo--用来描述管理构件中的属性。

4. MBeanConstructorInfo--用来描述[管理构件](https://baike.baidu.com/item/管理构件)中的构建器。

5. MBeanOperationInfo--用来描述管理构件中的操作。

6. MBeanParameterInfo--用来描述管理构件操作或构建器的参数。

7. MBeanNotificationInfo--用来描述管理构件发出的通知。



## 代理层

[![图1代理层的组成](https://bkimg.cdn.bcebos.com/pic/f3d3572c11dfa9ec26401e1c62d0f703918fc14b?x-bce-process=image/resize,m_lfit,w_440,limit_1/format,f_auto)](https://baike.baidu.com/pic/JMX/2829357/0/08b68e5234710f350cf3e3b4?fr=lemma&ct=single)图1代理层的组成

代理层是一个运行在Java[虚拟机](https://baike.baidu.com/item/虚拟机)上的管理实体，它活跃在管理资源和管理者之间，用来直接管理资源，并使这些资源可以被远程的管理程序所控制。代理层由一个MBean[服务器](https://baike.baidu.com/item/服务器)和一系列处理被管理资源的服务所组成。图1表示了代理层的组成：

### MBean服务器

Mbean[服务器](https://baike.baidu.com/item/服务器)为代理层的核心，设备层的所有[管理构件](https://baike.baidu.com/item/管理构件)都在其注册，管理者只有通过它才能访问管理构件。

管理构件可以通过以下三种方法实例化和注册：

1. 通过另一个管理构件

2. 管理代理本身

3. 远程应用程序

注册一个管理构件时，必须提供一个唯一的对象名。[管理应用程序](https://baike.baidu.com/item/管理应用程序/16079363)用这个对象名进行标识管理构件并对其操作。这些操作包括：

1. 发现管理构件的管理接口

2. 读写属性值

3. 执行管理构件中定义的操作

4. 获得管理构件发出的通告

5. 基于对象名和属性值来查询管理构件

### 协议适配器和连接器

MBean[服务器](https://baike.baidu.com/item/服务器)依赖于协议适配器和连接器来和运行该代理的Java[虚拟机](https://baike.baidu.com/item/虚拟机)之外的管理应用程序进行通信。协议适配器通过特定的协议提供了一张注册在MBean服务器的[管理构件](https://baike.baidu.com/item/管理构件)的视图。例如，一个HTML适配器可以将所有注册过的管理构件显示在Web 页面上。不同的协议，提供不同的视图。

连接器还必须提供管理应用一方的接口以使代理和[管理应用程序](https://baike.baidu.com/item/管理应用程序/16079363)进行通信，即针对不同的协议，连接器必须提供同样的远程接口来封装通信过程。当远程应用程序使用这个接口时，就可以通过网络透明的和代理进行交互，而忽略协议本身。

适配器和连接器使MBean服务器与管理应用程序能进行通信。因此，一个代理要被管理，它必须提供至少一个协议适配器或者连接器。面临多种管理应用时，代理可以包含各种不同的协议适配器和连接器。

当前已经实现和将要实现的协议适配器和连接器包括：

1. RMI连接器

2. [SNMP协议](https://baike.baidu.com/item/SNMP协议)适配器

3. [IIOP](https://baike.baidu.com/item/IIOP)协议适配器

4. [HTML](https://baike.baidu.com/item/HTML)协议适配器

5. [HTTP](https://baike.baidu.com/item/HTTP)连接器

### 代理服务

代理服务可以对注册的[管理构件](https://baike.baidu.com/item/管理构件)执行管理功能。通过引入智能管理，JMX可以帮助我们建立强有力的管理解决方案。代理服务本身也是作为管理构件而存在，也可以被MBean[服务器](https://baike.baidu.com/item/服务器)控制。

[JMX规范](https://baike.baidu.com/item/JMX规范/14769057)定义了代理服务有：

1. 动态类装载--通过管理小程序服务可以获得并实例化新的类，还可以使位于网络上的类库本地化。

2. 监视服务--监视管理构件的属性值变化，并将这些变化通知给所有的监听者。

3. 时间服务--定时发送一个消息或作为一个调度器使用。

[![图2JMX](https://bkimg.cdn.bcebos.com/pic/a044ad345982b2b722f571cf31adcbef76099b57?x-bce-process=image/resize,m_lfit,w_440,limit_1/format,f_auto)](https://baike.baidu.com/pic/JMX/2829357/0/2e6fa738690732f9d56225b0?fr=lemma&ct=single)图2JMX

4)关系服务--定义并维持管理构件之间的相互关系。1.动态类装载

动态类装载是通过m-let（management applet）服务来实现的，它可以从网络上的任何URL处下载并实例化[管理构件](https://baike.baidu.com/item/管理构件)，然后向MBean服务器注册。在一个M-let服务过程中，首先是下载一个m-let文本文件，该文件是XML格式的文件，文件的内容标识了管理构件的所有信息，比如构件名称、在MBean[服务器](https://baike.baidu.com/item/服务器)中唯一标识该构件的对象名等。然后根据这个文件的内容，m-let服务完成剩余的任务。图2例示这一过程：

2.监视服务

通过使用监视服务，管理构件的属性值就会被定期监视，从而保证始终处于一个特定的范围。当监视的属性值的变化超出了预期定义的范围，一个特定的通告就会发出。[JMX规范](https://baike.baidu.com/item/JMX规范/14769057)当前规定了三种监视器：

1)计数器监视器，监视计数器类型的属性值，通常为整型，且只能按一定规律递增。

2)度量监视器，监视度量类型的属性值，通常为实数，值能增能减。

3)字符串监视器，监视字符串类型的属性值。

每一个监视器都是作为一个标准[管理构件](https://baike.baidu.com/item/管理构件)存在的，需要提供服务时，可以由相应的管理构件或远程管理应用程序动态创建并配置注册使用。

图3例示了计数器监视器的使用情况：

3.时间服务

   [![图3JMX](https://bkimg.cdn.bcebos.com/pic/b2de9c82d158ccbf40dafd3119d8bc3eb1354152?x-bce-process=image/resize,m_lfit,w_440,limit_1/format,f_auto)](https://baike.baidu.com/pic/JMX/2829357/0/7e7f7909e5660df43bc763b3?fr=lemma&ct=single)图3JMX

   时间服务可以在制定的时间和日期发出通告，也可以定期的周期性的发出通告，依赖于[管理应用程序](https://baike.baidu.com/item/管理应用程序/16079363)的配置。时间服务也是一个[管理构件](https://baike.baidu.com/item/管理构件)，它能帮助管理应用程序建立一个可配置的备忘录，从而实现智能管理服务。

4. 关系服务

### JMX规范

[JMX规范](https://baike.baidu.com/item/JMX规范/14769057)定义了管理构件之间的关系模型。一个关系是用户定义的管理构件之间的N维联系。

关系模型定义如下一些术语：

1. 角色：就是是一个关系中的一类成员身份，它含有一个角色值。

2. 角色信息：描述一个关系中的一个角色。

3. [关系类型](https://baike.baidu.com/item/关系类型)：由角色信息组成，作为创建和维持关系的模板。

4. 关系：[管理构件](https://baike.baidu.com/item/管理构件)之间的当前联系，且必须满足一个关系类型的要求。

5. 角色值：在一个关系中当前能满足给定角色的管理构件的列表。

6. 关系服务：是一个管理构件，能接触和维持所有关系类型和关系实例之间的一致性。

在关系服务中，管理构件之间的关系由通过关系类型确定的关系实例来维护。仅仅只有注册到MBean[服务器](https://baike.baidu.com/item/服务器)上并且能被对象名标识的管理构件才能成为一个关系的成员。关系服务从来就不直接操作它的成员--管理构件，为了方便查找它仅仅提供了对象名。

关系服务能锁定不合理[关系类型](https://baike.baidu.com/item/关系类型)的创建，同样，不合理的关系的创建也会被锁定。角色值的修正也要遵守[一致性检查](https://baike.baidu.com/item/一致性检查/2457781)。

由于关系是定义在注册的[管理构件](https://baike.baidu.com/item/管理构件)之间的联系，所以当其中的管理构件[卸载](https://baike.baidu.com/item/卸载)时，就会更改关系。关系服务会自动更改角色值。所有对关系实例的操作比如创建、更新、删除等都会使关系服务发出通告，通告会提供有关这次操作的信息。

JMX关系模型只能保证所有的管理构件满足它的设计角色，也就是说，不允许一个管理构件同时出现在许多关系中。



## 服务层

当前，SUN并没有给出这一层的具体规范，下面给出的只是一个简要描述。

该层规定了实现JMX应用管理平台的接口。这一层定义了能对代理层进行操作的管理接口和组件。这些组件能：

1. 为[管理应用程序](https://baike.baidu.com/item/管理应用程序/16079363)提供一个接口，以便它通过一个连接器能透明和代理层或者JMX管理资源进行交互。

2. 通过各种协议的映射（如SNMP、HTML等），提供了一个JMX代理和所有可管理组件的视图。

3. 分布管理信息，以便构造一个[分布式系统](https://baike.baidu.com/item/分布式系统)，也就是将高层管理平台的管理信息向其下众多的JMX代理发布。

4. 收集多个JMX 代理端的管理信息并根据管理终端用户的需要筛选用户感兴趣的信息并形成逻辑视图送给相应的终端用户。

5. 提供了安全保证。

通过管理[应用层](https://baike.baidu.com/item/应用层)和另一管理代理和以及他的设备层的联合，就可以为我们提供一个完整的网络管理的解决方案。这个解决方案为我们带来了独一无二的一些优点：轻便、根据需要部署、动态服务、还有安全性。



## 管理协议

该层提供了一些API来支持当前已经存在的一些管理协议。

这些附加的协议API并没有定义管理应用的功能，或者管理平台的[体系结构](https://baike.baidu.com/item/体系结构)，他们仅仅定义了标准的Java API和现存的[网络管理技术](https://baike.baidu.com/item/网络管理技术)通信，例如SNMP。

网络管理平台和应用的开发者可以用这些API来和他们的管理环境进行交互，并将这个交互过程封装在一个JMX管理资源中。例如，通过SNMP可以对一个运行有SNMP代理的[交换机](https://baike.baidu.com/item/交换机)进行管理，并将这些管理接口封装成为一个[管理构件](https://baike.baidu.com/item/管理构件)。在动态网络管理中，可以随时更换这些管理构件以适应需求。

这些API可以帮组开发者根据最通常的工业标准来部署他们的管理平台和应用。新的网路管理的解决方案可以和现存的基础结构合为一体，这样，现存的网络管理也能很好的利用基于Java技术的网络管理应用。

这些API目前在JCP（Java Community Process）内作为独立的JSR（Java Specification Request）开发。

他们包括：

1. SNMP Manager API

2. CIM/WBEM manager and protocol API



## 实现应用

自从SUN发布了[JMX规范](https://baike.baidu.com/item/JMX规范/14769057)，许多大公司纷纷行动起来，实现规范或者实现相应的基于JMX的[网络管理系统](https://baike.baidu.com/item/网络管理系统)，下面列出了当前的主要实现及应用情况：

1. SUN为[JMX规范](https://baike.baidu.com/item/JMX规范/14769057)了作出了相应的参考实现，并在此基础上开发了一个全新的用于网络管理的产品[JDMK](https://baike.baidu.com/item/JDMK)（Java动态管理工具集），其中定义了资源的开发过程和方法、动态JMX代理的实现、远程管理应用的实现。同时，JDMK也提供了一个完整的[体系结构](https://baike.baidu.com/item/体系结构)用来构造分布式的网络管理系统，并提供了多种协议适配器和连接器，如[SNMP协议](https://baike.baidu.com/item/SNMP协议/421695)适配器、HTML协议适配器、HTTP连接器、RMI连接器。

2. IBM Tivoli实现了JMX规范的产品为TivoliJMX，它为JAVA[管理应用程序](https://baike.baidu.com/item/管理应用程序/16079363)和网络提供了架构、设计模式、一些API集和一些服务。

3. Adventnet开发的关于JMX的产品为AdventNet Agent Toolkit，它使得定义新的SNMP MIB、开发JMX和Java SNMP Agent的[过程自动化](https://baike.baidu.com/item/过程自动化)。

4. JBoss实现的J2EE[应用服务器](https://baike.baidu.com/item/应用服务器)以JMX为[微内核](https://baike.baidu.com/item/微内核)，各个模块以[管理构件](https://baike.baidu.com/item/管理构件)的形式提供相应的服务。

5. BEA的Weblogic应用服务器也将JMX技术作为自己的管理基础。

6. 金蝶的Apusic也是一个以JMX为[内核](https://baike.baidu.com/item/内核)开发出的[J2EE应用服务器](https://baike.baidu.com/item/J2EE应用服务器/12678909)。



## 小结

本文详细介绍了[JMX规范](https://baike.baidu.com/item/JMX规范/14769057)。JMX[体系结构](https://baike.baidu.com/item/体系结构)分为四层，即设备层、代理层、分布服务层和附加协议API。但SUN当前只实现了前两层的具体规范，其余的规范还在制定当中。JMX代理要和远程应用程序通信，需要提供至少一个连接器和协议适配器。



## 架构

JMX应该说是关于网络应用管理的框架，如果你开发了一个比较复杂的系统，无疑你要提供这个系统的自身管理 系统，JMX更多应用是体现在Server上，如果你要使用java开发一个自己Server或复杂的应用系统，那么推荐你基于JMX架构来开发， JBoss 3.0 weblogic等就是基于JMX开发的符合J2EE规范的[服务器软件](https://baike.baidu.com/item/服务器软件)。

了解JMX可以使你深入了解J2EE[服务器](https://baike.baidu.com/item/服务器)， 为什么我们平时说 "[EJB](https://baike.baidu.com/item/EJB)"是个比较"Weight"的方案选择，其中一个原因是J2EE服务器软件本身 也是你的系统中一部分，它作为你系统的容器，对你的系统有至关重要的作用，如果无法直接介入 管理或“调教”它，那么无疑你的系统本身存在着隐含的危险， 现在，通过JMX，你现在可以深入到你J2EE容器内部的管理了。 (好像国内出现了第一个自己J2ee服务器，不知道那是不是基于JMX开发的?)

J2EE并不能概括所有的应用领域，比如对速度和性能要求极高的游戏或股票行情等系统就需要自己直接来开发Server， 如果是能够基于JMX开发，那么可以说就大大提高编写管理程序的效率，可以将你的模块变成JMX的MBean，可以通过Agent在程序内部或者通过 WEB管理页面对你的MBean模块进行初始化 重启 以及参数设置。

[![Sun公司JMX规定的架构图](https://bkimg.cdn.bcebos.com/pic/32fa828ba61ea8d3c8413c0f970a304e251f585d?x-bce-process=image/resize,m_lfit,w_440,limit_1/format,f_auto)](https://baike.baidu.com/pic/JMX/2829357/0/cc506c8b724ec429c8fc7abe?fr=lemma&ct=single)Sun公司JMX规定的架构图

JMX的好处还有：可以方便整合连接现有的Java技术，如JNDI JDBC JTS及其它。特别是能够使用Jini的查询 发现机制以及协议，我们知道,Jini提供了一种服务的查询和发现机制，这些services都可以通过JMX 来实现管理。现在我们开始JMX的了解：

1. 到java.sun首页的JMX页面，下载JMX的规定说明和Samples程序。

2. 按照JMX的说明进行一次Tutorial，了解如何加入 删除 配置一个MBean，Tutorial中是以SimpleMBean为例，那么我们能否建立一个自己的MBean?

   我们来做一个Hello 的MBean，这里有一个小关键点，你的[class](https://baike.baidu.com/item/class)取名有个规则， 需要以MBean为结尾，如这里我们取名为HelloMbean:

   ```java
   public interface HelloMBean {
       // management attributes
       public String getName();
       public void setName(String name);
       // management operations
       public void print();
   }
   
   
   ```

   在这个Class里，有一个隐含attributes: name, 提供了set和get的方法，同时有一个操作方法print()：

   再定义一个concrete类:

   ```java
   public class Hello implements HelloMBean {
   
       private String name = "";
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
       }
   
       public void print() {
           System.out.println("Hello, " + name + "!!" );
       }
   }
   ```

   这样一个简单的MBean就做好了，我们可以通过admin界面加入这个Hello，

   再按 Tutorial启动BaseAgent，在Agent Administration中参考Simple填入：

   ```java
   Domain: Standard_Hello_MBeans
   Keys : name=Hello,number=1
   Java Class: Hello
   ```

   将出现Create Successful信息。进入MBean View 给Name赋值，点按Apply ，然后再按print，这是你的Hello中的方法，在控制台你会看到输出。

   是不是很惊奇Hello中的 attributes 和operations能被动态的访问和控制？ 已经隐约感到JMX的架构原理了吧？

   下面再深入明确一些概念：

   上面HelloMBean资源是通过admin这样的HTTP WEB界面管理，这种管理资源方式是属于JMX的Distributed服务层， JMX 通过Distributed层能够部署和管理MBean资源。就象上面的例子，是通过HtmlAdaptor提供的HTTP WEB界面来方面的维护管理HelloMBean.

   那么我们能否在程序中自动管理和部署我的MBean？当然可以，这是通过Agent层来完成，现在我们已经有了这个层次，MBean所在的资源层，最外面的Distributed服务层，Distributed服务层是通过Agent层来访问MBean资源的。

   Agent Level(Agent层)包括MBean Server和Agent Services,那么我们来做一个上面例子HelloMBean的Agent：

   ```java
   // CREATE the MBeanServer
   //
   System.out.println("\n\tCREATE the MBeanServer.");
   
   MBeanServer server = MBeanServerFactory.createMBeanServer();
   
   // CREATE Registe HelloMBean
   //
   System.out.println("\n\tCREATE, REGISTER a new Hello Standard_MBean:");
   HelloMBean helloMBean = new Hello();
   ObjectName hello_name = null;
   
   try {
       hello_name = new ObjectName("Standard_Hello_MBeans:name=Hello,number=1");
       System.out.println("\tOBJECT NAME = " + hello_name);
       //将HelloMBean注册到MBeanServer中去
       server.registerMBean(helloMBean, hello_name);
   }
   catch (Exception e) {
       e.printStackTrace();
       return;
   }
   ```

   向MBeanServer注册后，以后JMX就知道有了这个HelloMBean资源。

管理一个agent的MBean资源或使用它提供的服务必须通过一个protocol adaptor 或者connector,adaptor 或者connector属于Distributed layer level(Distributed服务层)，我们上面例子中通过HTTP WEB界面管理HelloMBean就是浏览器通过HtmlAdaptor这个adaptor来实现的。