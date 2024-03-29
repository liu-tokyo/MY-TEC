# EJB - 百度百科

EJB是Enterprise Java Beans技术的简称, 又被称为企业Java Beans。这种技术最早是由美国计算公司研发出来的。EJB技术的诞生标志着Java Beans的运行正式从客户端领域扩展到服务器领域。在[电子商务](https://baike.baidu.com/item/电子商务/98106)领域运用EJB技术可以简化应用系统的开发, 这是由该技术的结构和特点所决定的。

## EJB概念

EJB (Enterprise Java Beans) 是基于分布式事务处理的企业级应用程序的组件。Sun公司发布的文档中对EJB的定义是：EJB是用于开发和部署多层结构的、分布式的、面向对象的Java应用系统的跨平台的构件体系结构。

在开发分布式系统时, 采用EJB可以使得开发商业应用系统变得容易, 应用系统可以在一个支持EJB的环境中开发, 开发完之后部署在其它的EJB环境中, 随着需求的改变, 应用系统可以不加修改地迁移到其它功能更强、更复杂的服务器上。EJB在系统实现业务逻辑层里面负责表示程序的逻辑和提供访问数据库的接口。

## EJB历史

从拥抱到抛弃

由于[IBM](https://baike.baidu.com/item/IBM)和[Sun Microsystems](https://baike.baidu.com/item/Sun Microsystems)等EJB提倡者力推其前景，起初一些大公司纷纷采用EJB部署他们的系统。然而随后各种问题便接踵而至，对EJB的恶评短时间内激增。对于初学者，EJB的[API](https://baike.baidu.com/item/API)显得太过困难；对于许多程序员来说，书写那些必须抛出特定异常的接口并将bean类作为抽象类实现的做法既不直观也不正常。当然，EJB所被赋予的使命，如[对象关系映射](https://baike.baidu.com/item/对象关系映射/311152)和事务管理确实有其天然复杂性，但其API之复杂还是令开发人员们觉得望而却步，一些人开始怀疑EJB除了引入了复杂的实现手段以外似乎并未带来什么实际好处。

另外，实际运用中被发现，如果使用EJB来封装业务逻辑会带来性能上的下降。这是因为，最早的EJB规范只允许客户端通过特定协议（如[CORBA](https://baike.baidu.com/item/CORBA)）进行远程方法调用来调用，即使大部分实际应用根本就不需要分布式计算。直到EJB 2.0才引入了本地接口，以支持可以开发不通过网络就能直接本地调用的EJB系统。

尽管如此，EJB的广泛普及仍然为其复杂度所制约。尽管已经有一些高质量的集成开发工具可以协助开发人员通过自动编码解决一部分重复作业，但这并不能降低学习此项技术的难度。另一方面，“草根阶层”的编程爱好者们发起了一场旨在使用 “轻量级”技术以代替复杂的EJB的运动。这些技术包括[Hibernate](https://baike.baidu.com/item/Hibernate)（用于提供[数据持久化](https://baike.baidu.com/item/数据持久化/5777076)和对象-关系映射）及[Spring](https://baike.baidu.com/item/Spring)框架（用于封装业务逻辑）。尽管它们不像EJB那样有巨头支持，但其在庶民间却更加流行，并且也被一些对EJB深感失望的企业所采用。

### 重生

EJB规范起初的一个主要价值—对分布式应用进行事务管理—在随后的实践中被一致认为几乎没能派上用场。对于企业级应用来说，Spring和Hibernate等简化框架更加实用。因此，EJB 3.0规范（JSR 220）为了迎合这个趋势相比于其前辈进行了一次激进的大跳跃。受到Spring 影响，EJB 3.0也使用所谓的“传统简单Java对象（[POJO](https://baike.baidu.com/item/POJO)）”；同时，支持依赖注入来简化全异系统的集成与配置。Hibernate的创始人Gavin King参与了这一新版规范的制订，并对EJB大加提倡。Hibernate的许多特性也被引入到Java持久化API当中，从而取代原来的实体bean。EJB 3.0规范大幅采用[Java注释](https://baike.baidu.com/item/Java注释)（annotation）来对代码进行元数据修饰，从而消减了此前EJB编程的冗杂性。

相应地，EJB 3.0几乎成为了一个全新的API，与此前的数版可谓毫无相似度可言

## 技术特点

- 具有可复制性。由于EJB是以组件为基础的技术模型, 所以在任何一个EJB服务器中都可以随意部署, 或者将其他模式直接进行移植和复制, 不需要定制专用的EJB服务器系统和容器。EJB之所以能够被复制, 是因为让自身的服务器已经定义了一些规范服务, 而这些服务又可以对EJB容器和组件之间的关系进行定义, 即使进行复制也不会破坏其内部结构的稳定性。

- 具有重复性特点。因为EJB是服务器上运行程序的一个功能片段, 可以进行重新组装和重新定义, 所以它能够利用自身的这一特性和其他组间一起组建出符合使用需求的应用系统, 而且可以同时为多个系统提供服务。

- 具有独立的平台。和计算机网络中任何一个特殊平台、[Internet协议](https://baike.baidu.com/item/Internet协议/11049108)和中间件或其他基础设施不同, EJB结构平台完全是独立的, 具有极强的独立性。也正是因为如此, 它所开发出来的应用程序才能不进行任何修改, 完全被复制到另外的平台中。

- 具有广阔的拓展空间。EJB模型是以多层[分布式体系结构](https://baike.baidu.com/item/分布式体系结构/53356845)为基础构建起来的, 因为该系统的多功能, 所以不管是规模较小的应用程序, 还是规模较大的事务处理, 都可以运用高模型。随着电子信息和通信技术的不断发展, 应用程序的需求量不断增加, 为了让这些应用程序可以在不改变核心程序的前提下被复制到操作功能更强大、运行环境更先进的系统中, 就可以运用EJB模型来实现。所以, EJB具有极强的扩展性, 而且电子技术的进步会为它的应用提供一个非常广阔的空间。

## EJB组件的工作流程

**EJB Component**在部署到应用服务器上之后, 客户端就可以调用它来完成各种功能。工作过程如下:

1. 客户端首先通过JNDI服务检索Home对象。在EJB应用部署到应用服务器上之后, 容器会自动获得Home对象的信息并将其加入到JNDI中。

2. JNDI服务返回所查找的Home对象的引用。

3. Home对象的创建或者查找EJB对象。

4. Home对象将获得的EJB对象返回给客户端。

5. 客户端利用获得的EJB对象引用, 调用业务方法。

6. EJB对象获得对应bean的一个实例并将相应的业务方法调用传递给该实例。

7. Bean实例通过其实现代码, 完成相应的业务逻辑并将结果返回给EJB对象。

8. EJB对象将方法的结果返回给客户端。

## EJB种类

EJB容器可以接受三类EJB

- 会话Bean（Session Beans）

- - 无状态会话Bean（Stateless Session Beans）
  - 有状态会话Bean（Stateful Session Beans）

- 实体Bean（Entity Beans）

- 消息驱动Bean（Message Driven Beans ，MDBs）

**无状态会话Bean**是一类不包含状态信息的分布式对象，允许来自数个客户端的并发访问。实例变量的内容在前后数次呼出中不被保留（确切地说是不保证保留）。由于不必控制与用户间的对话信息而减少了开销，无状态会话Bean不像有状态会话Bean那样具有资源集约性。举例来说，一个发送邮件的EJB就可被设计为一个无状态会话Bean。在整个会话期，用户只向服务器提交一个动作：发送指定[邮件](https://baike.baidu.com/item/邮件)到指定地址。（称为开关行为）

**有状态会话Bean**是包含状态的分布式对象，即是说，贯穿整个会话它们都要保有客户端信息。举例而言，在一个网上商店进行实施结账很可能就需要一个有状态会话Bean，因为结账是一个多步动作，服务器端必须可以随时了解到用户已经进行到了哪一步。此外，尽管有状态会话Bean的状态信息可被保持，但始终只能是由同一个用户来访问之。

**实体Bean**是含有持久化状态的分布式对象。这个持久化状态的管理既可以交给Bean自身（Bean-Managed Persistence，BMP），也可以托付于外部机制（Container-Managed Persistence，CMP）。

**消息驱动Bean**是支持异步行为的分布式对象。它们并不对请求进行当即响应。比方说，某网站用户点击“请通知我更新信息”按钮，将会触发某个MDB将这名用户加入到数据库的希望获得更新信息用户列表中。这个动作就是一个异步的消息驱动过程，因为用户不必等待当时会返回某个结果。MDB的消息源来自Java消息服务（JMS）提供的消息队列或消息主题。自EJB 2.0规范起，JMS被加入进来以允许在容器内部实施事件驱动处理。与其他EJB不同，MDB不存在一个用户视图（如需要用户引用的远程接口），用户也不能通过资源定位获得一个MDB实例。MDB只在后台监听消息源并实施自动处理。

除了上述以外，当前还有一些EJB处于设想阶段，如JSR 86提出了用于在Java EE应用中集成多媒体对象的媒体Bean（Enterprise Media Beans）。

## EJB实行

EJB部署于应用服务器端的EJB容器中。规范给定了EJB与EJB容器之间，以及用户代码与EJB/EJB容器之间的交互方式。对于Java EE API，javax.ejb包定义了EJB类，javax.ejb.spi包定义了EJB容器应当实现的各个接口。

在EJB 2.1和以前的版本中，每个EJB都由一个类和两个接口组成。EJB容器负责创建这个类的实例，接口则供客户端调用。

两个接口分别被称为Home接口和组件接口，负责提供各个EJB远程方法声明。这些EJB远程方法可分成两组：

- 类方法：由Home接口提供。与特定实例无关，仅负责一些公共内容，比如创建一个新的EJB实例（create方法），或寻找一个已经存在的EJB实例（find方法）等等。
- 接口方法：由组件接口提供的针对特定实例的业务方法。

EJB容器将为这些接口提供对应的实现类以充当客户远程代理，当客户端调用这个生成的代理类的某个方法时，代理类内部会将此调用的方法和参数封装成一个消息发送给服务器。服务器收到消息后在转发给真实的EJB实例，后者负责执行真正的业务逻辑。

### 远程通信

EJB规范要求EJB容器能够支持基于RMI-IIOP的EJB访问。EJB既可被任何CORBA应用访问，也能提供[Web服务](https://baike.baidu.com/item/Web服务/2837593)。

### 事务

EJB容器必须支持符合[ACID](https://baike.baidu.com/item/ACID)（原子性/一致性/独立性/持久性）特性的容器级事务管理，以及bean内部事务管理。容器级事务需在部署描述符中（EJB应用的配置文件）进行声明。

### 事件

EJB使用JMS向客户对象发送消息，客户则可以异步地接受这些消息。MDB则接受来自客户端的消息。

### 命名和目录服务

EJB客户端使用JNDI或CORBA名字服务定位Home接口实现 对象。通过此Home接口，用户还可以寻找，创建或删除实体对象。

### 安全

EJB容器对客户端的访问权限负责。

### 部署EJB

EJB规范还定义了一个跨平台的统一部署机制。部署描述符中定义了关于EJB应用的一切相关内容。文件名通常为ejb-jar.xml。

部署描述符是一个XML文档，负责为该EJB应用中的每一个EJB定义入口。部署描述符的主要内容包括：

- Home接口名
- Bean的Java类名
- Home接口的Java接口名
- 组件接口的Java接口名
- 持久化存储（针对实体Bean）
- 安全策略和角色分配

通常EJB容器提供者还定义了一些额外的XML或其他格式描述文件来强化其容器的功能。他们还同时提供这些描述文件的解读工具类和对Home接口的自动实现类生成。

EJB3.0起开始广泛使用Java注释替代传统的部署描述符ejb-jar.xml。但后者仍然有效。

## 版本变化

EJB 3.0

2006年5月2日发布，JSR 220定义。

- 全面采用Java注释代替部署描述符。（后者仍可使用，并且具有更高优先级）
- 把2.X版的EntityBean改为由JPA支持。

### EJB 2.1

2003年11月24日发布，JSR 153定义。

- Web服务：可将无状态会话bean暴露为Web服务；EJB可通过引用访问Web服务。
- EJB定时器服务：提供一种新的基于定时器的事件驱动方式。可供消息驱动bean作为消息源使用。
- 增加了消息目的地。
- 进一步丰富了EJB查询语言，支持ORDER BY, AVG, MIN, MAX, SUM, COUNT和MOD。
- 使用XML schema代替DTD以定义部署描述符。

### EJB 2.0

2001年8月22日发布，JSR 19 定义。

- 制定了构建面向对象商务应用的标准组建结构。
- 支持构筑使用不同开发工具所开发之组件的联合应用部署。
- 在多线程，连接池，事务管理等方面对用户透明化。
- 使符合“一次写成，多次运行”的Java思想。
- 关注企业级应用生命期间的开发，部署，运行等动作。
- 定义了不同开发工具所需遵守的契约，以便其产品能够在运行期交互。
- 支持与现行系统兼容，开发者可以扩展现有产品以使之支持EJB。
- 与其他Java API兼容。
- 支持EJB与Java2平台企业版或者其他非Java应用程序之间的互操作性。
- 支持与CORBA兼容的RMI-IIOP。

### EJB 1.1

1999年12月17日发布。

- 开始采用XML部署描述符，默认的JNDI上下文以及可支持IIOP的RMI。
- 安全机制由角色（Role）驱动，而非方法。
- 支持实体类，且必须在应用中实现。

### EJB 1.0

1998年3月24日发布。

- 定义了EJB和EJB容器的作用，实现与互动。
- 提供了最早的开发者与用户视图。