# Hibernate - 百度百科

Hibernate是一个[开放源代码](https://baike.baidu.com/item/开放源代码/114160)的[对象关系映射](https://baike.baidu.com/item/对象关系映射/311152)框架，它对[JDBC](https://baike.baidu.com/item/JDBC/485214)进行了非常轻量级的对象封装，它将POJO与数据库表建立映射关系，是一个全自动的orm框架，hibernate可以自动生成SQL语句，自动执行，使得Java程序员可以随心所欲的使用对象编程思维来操纵数据库。 Hibernate可以应用在任何使用JDBC的场合，既可以在Java的客户端程序使用，也可以在Servlet/JSP的Web应用中使用，最具革命意义的是，Hibernate可以在应用EJB的[JavaEE](https://baike.baidu.com/item/JavaEE)架构中取代CMP，完成[数据持久化](https://baike.baidu.com/item/数据持久化/5777076)的重任。

## 简介

Hibernate作为数据库与界面之间的桥梁，需要面向对象思想操纵对象。对象可能是普通JavaBeans/POJO。应用程序通过抽象将应用从底层事务隔离开。使用底层的API或Transaction对象完成轻量级框架提供一级缓存和二级缓存。Hibernate直接提供相关支持，底层驱动可以随意切换数据库，快速简洁。使业务层与具体数据库分开，只针对Hibernate 进行开发，完成数据和对象的持久化。针对不同的数据库形成不同的SQL 查询语句，降低数据库之间迁移的成本。Hibernate支持多种缓存机制，Hibernate适配MS SQLSERVER、ORACLE、SQL、H2、Access和[Mysql](https://baike.baidu.com/item/Mysql/471251)等多种数据库。

Hibernate用反射机制实现持久化对象操作，实现与[IDE](https://baike.baidu.com/item/IDE/8232086)(Integrated Development Environment)的耦合度。Hibernate使用数据库和配置信息为应用程序提供持久化服务。从配置文件中读取数据库相关参数，将持久化类和数据表对应使用。用Hibernate API对象持久化，利用映像信息将持久化操作翻译为SQL语句进行查询。

Hibernate框架技术最关键是[数据持久化](https://baike.baidu.com/item/数据持久化/5777076)，是将数据保存到持久层的过程。持久层的数据在掉电后也不会丢失的数据。持久层是基于Hibernate技术的检索系统开发的基本。系统结构的层次模型有三个阶段。

整个过程首先实现应用层和数据层。数据层保存持久化数据，应用层接收输入的数据。然后通过MVC 模式实现业务逻辑与表示层的分开。表示层和用户实现交互，业务逻辑层处理数据持久化操作。将第二阶段业务逻辑层的功能部署拆分后，业务逻辑层完成核心业务逻辑处理，持久层完成对象持久化。降低业务逻辑层复杂度的同时将数据持久化让其他组件完成。 

## 1. 发展历程

2001年，[澳大利亚墨尔本](https://baike.baidu.com/item/澳大利亚墨尔本/9042461)一位名为Gavin King的27岁的程序员，上街买了一本SQL编程的书，他厌倦了实体bean，认为自己可以开发出一个符合对象关系映射理论，并且真正好用的Java持久化层框架，因此他需要先学习一下SQL。这一年的11月，Hibernate的第一个版本发布了。

2002年，已经有人开始关注和使用Hibernate了。

2003年9月，Hibernate开发团队进入JBoss公司，开始全职开发Hibernate，从这个时候开始Hibernate得到了突飞猛进的普及和发展。

2004年，整个Java社区开始从实体bean向Hibernate转移，特别是在Rod Johnson的著作《Expert One-on-One J2EE Development without EJB》出版后，由于这本书以扎实的理论、充分的论据和详实的论述否定了EJB，提出了轻量级敏捷开发理念之后，以Hibernate和Spring为代表的轻量级开源框架开始成为Java世界的主流和事实标准。在2004年Sun领导的J2EE5.0标准制定当中的持久化框架标准正式以Hibernate为蓝本。

2006年，J2EE5.0标准正式发布以后，持久化框架标准Java Persistent API（简称JPA）基本上是参考Hibernate实现的，而Hibernate在3.2版本开始，已经完全兼容JPA标准。

## 2. 编程开发

编程环境

Hibernate是一个以[LGPL](https://baike.baidu.com/item/LGPL)（Lesser GNU Public License）许可证形式发布的开源项目。在Hibernate官网上有下载Hibernate包的说明。Hibernate包以源代码或者二进制的形式提供。 [2] 

### 2.1 编程工具

[![img](https://bkimg.cdn.bcebos.com/pic/5d6034a85edf8db11480f6470923dd54564e7463?x-bce-process=image/resize,m_lfit,w_440,limit_1/format,f_auto)](https://baike.baidu.com/pic/Hibernate/206989/0/a75fb6d3228cfe27970a168c?fr=lemma&ct=single)

- [Eclipse](https://baike.baidu.com/item/Eclipse/61703)：一个开放源代码的、基于Java的可扩展开发平台。

- [NetBeans](https://baike.baidu.com/item/NetBeans)：开放源码的Java集成开发环境，适用于各种客户机和Web应用。

- [IntelliJ IDEA](https://baike.baidu.com/item/IntelliJ IDEA)：在代码自动提示、代码分析等方面的具有很好的功能。

- [MyEclipse](https://baike.baidu.com/item/MyEclipse)：由[Genuitec](https://baike.baidu.com/item/Genuitec)公司开发的一款商业化软件，是应用比较广泛的Java应用程序集成开发环境。 [3] 

- [EditPlus](https://baike.baidu.com/item/EditPlus)：如果正确配置Java的[编译器](https://baike.baidu.com/item/编译器)“[Javac](https://baike.baidu.com/item/Javac)”以及解释器“Java”后，可直接使用EditPlus编译执行Java程序。

## 3. 语言特点

- 将对数据库的操作转换为对Java对象的操作，从而简化开发。通过修改一个“持久化”对象的属性从而修改数据库表中对应的记录数据。
- 提供线程和进程两个级别的缓存提升应用程序性能。
- 有丰富的映射方式将Java对象之间的关系转换为数据库表之间的关系。
- 屏蔽不同数据库实现之间的差异。在Hibernate中只需要通过“方言”的形式指定当前使用的数据库，就可以根据底层数据库的实际情况生成适合的SQL语句。
- 非侵入式：Hibernate不要求持久化类实现任何接口或继承任何类，POJO即可。

## 4. 核心API

Hibernate的API一共有6个，分别为:[Session](https://baike.baidu.com/item/Session/479100)、[SessionFactory](https://baike.baidu.com/item/SessionFactory)、[Transaction](https://baike.baidu.com/item/Transaction/4114110)、[Query](https://baike.baidu.com/item/Query)、[Criteria](https://baike.baidu.com/item/Criteria)和Configuration。通过这些[接口](https://baike.baidu.com/item/接口)，可以对持久化对象进行存取、事务控制。

**Session**

[Session](https://baike.baidu.com/item/Session)接口负责执行被持久化对象的[CRUD](https://baike.baidu.com/item/CRUD)操作(CRUD的任务是完成与数据库的交流，包含了很多常见的SQL语句)。但需要注意的是[Session对象](https://baike.baidu.com/item/Session对象)是非[线程安全](https://baike.baidu.com/item/线程安全)的。同时，Hibernate的[session](https://baike.baidu.com/item/session)不同于JSP应用中的HttpSession。这里当使用session这个术语时，其实指的是Hibernate中的session，而以后会将HttpSession对象称为用户session。

**SessionFactory**

SessionFactory接口负责初始化Hibernate。它充当数据存储源的代理，并负责创建[Session对象](https://baike.baidu.com/item/Session对象/5250998)。这里用到了[工厂模式](https://baike.baidu.com/item/工厂模式)。需要注意的是SessionFactory并不是轻量级的，因为一般情况下，一个项目通常只需要一个SessionFactory就够，当需要操作多个数据库时，可以为每个数据库指定一个SessionFactory。

**Transaction**

Transaction 接口是一个可选的API，可以选择不使用这个接口，取而代之的是Hibernate 的设计者自己写的底层事务处理代码。 Transaction 接口是对实际事务实现的一个抽象，这些实现包括JDBC的事务、JTA 中的UserTransaction、甚至可以是CORBA 事务。之所以这样设计是能让开发者能够使用一个统一事务的操作界面，使得自己的项目可以在不同的环境和容器之间方便地移植。

**Query**

Query接口让你方便地对数据库及持久对象进行查询，它可以有两种表达方式：HQL语言或本地数据库的SQL语句。Query经常被用来绑定查询参数、限制查询记录数量，并最终执行查询操作。

**Criteria**

Criteria接口与Query接口非常类似，允许创建并执行面向对象的标准化查询。值得注意的是Criteria接口也是轻量级的，它不能在Session之外使用。

**Configuration**

Configuration 类的作用是对Hibernate 进行配置，以及对它进行启动。在Hibernate 的启动过程中，Configuration 类的实例首先定位映射文档的位置，读取这些配置，然后创建一个SessionFactory对象。虽然Configuration 类在整个Hibernate 项目中只扮演着一个很小的角色，但它是启动hibernate 时所遇到的第一个对象。

## 5. 回调接口

当⼀个对象发生了特定的事件，例如对象被保存，删除，更新和装载时，Hibernate应用可以通过回调接口来响应这⼀事件，做相应的操作，主要有两类实现方式： 

1）持久化类实现LifeCycle和Validatabale接口。这种方式使Hibernate 接口渗透到了持久化类中，会影响持久化类的可移植性。

2）Interceptor接口，应用程序可以定义专门的拦截器实现类，由它负责事件的响应，这种方式持久化类不会受到影响。

## 6. 可扩展点

1）定制主键的生成策略，IdentifierGenarator接口

2）定制本地SQL方言，Dialect抽象类

3）定制缓存机制，Cache和CacheProvider接口 

4）定制JDBC连接管理，ConnectionProvider接口 

5）定制事务管理，TransactionFactory，Transaction，TransactionManagerLookup接口

6）定制ORM策略，classPersister接口及子接口

7）定制属性访问策略，PropertyAccessor接口 

8）创建代理，ProxyFactory接口

9）定制客户化映射关系，UserType和CompositeUserType接口

## 7. 版本介绍

Hibernate版本更新速度很快，有多个阶段性的版本：Hibernate3、Hibernate4、Hibernate5、Hibernate ORM 5.2.12.Final Released。Hibernate2系列的最高版本是Hibernate2.1.8，Hibernate3系列的最高版本是hibernate-distribution-3.6.10.Final-dist版，但使用较多且较稳定的版本是Hibernate 3.1.3或Hibernate 3.1.2。

另外，自Hibernate3发布以来，其产品线愈加成熟，相继出现了Hibernate注释、Hibernate实体管理器、Hibernate[插件](https://baike.baidu.com/item/插件)工具等一系列产品套件。在方便程序员使用Hibernate进行应用程序的开发的同时，也逐渐增强了Hibernate产品线的实力。Hibernate已经出现了4.0以及5.0的版本

## 8. 主键介绍

**Assigned**

Assigned方式由用户生成主键值，并且要在save()之前指定否则会[抛出异常](https://baike.baidu.com/item/抛出异常)

特点：主键的生成值完全由用户决定，与底层数据库无关。用户需要维护主键值，在调用session.save()之前要指定主键值。

**Hilo**

Hilo使用高低位算法生成主键，高低位算法使用一个高位值和一个低位值，然后把算法得到的两个值拼接起来作为数据库中的唯一主键。Hilo方式需要额外的数据库表和字段提供高位值来源。默认情况下使用的表是

hibernate_unique_key，默认字段叫作next_hi。next_hi必须有一条记录否则会出现错误。

特点：需要额外的数据库表的支持，能保证同一个数据库中主键的唯一性，但不能保证多个数据库之间主键的唯一性。Hilo主键生成方式由Hibernate 维护，所以Hilo方式与底层数据库无关，但不应该手动修改hi/lo算法使用的表的值，否则会引起主键重复的异常。

**Increment**

[Increment](https://baike.baidu.com/item/Increment)方式对主键值采取自动增长的方式生成新的主键值，但要求底层数据库的主键类型为long,int等数值型。主键按数值顺序递增，增量为1。

/*特点：由Hibernate本身维护，适用于所有的数据库，不适合[多进程](https://baike.baidu.com/item/多进程)并发更新数据库，适合单一进程访问数据库。不能用于群集环境。*/

**Identity**

Identity方式根据底层数据库，来支持自动增长，不同的数据库用不同的主键增长方式。

特点：与底层数据库有关，要求数据库支持Identity，如MySQl中是auto_increment, SQL Server 中是Identity，支持的数据库有MySql、SQL Server、DB2、Sybase和HypersonicSQL。 Identity无需Hibernate和用户的干涉，使用较为方便，但不便于在不同的数据库之间移植程序。

**Sequence**

Sequence需要底层数据库支持Sequence方式，例如[Oracle数据库](https://baike.baidu.com/item/Oracle数据库)等

特点：需要底层数据库的支持序列，支持序列的数据库有DB2、PostgreSql、Oracle、SAPDb等在不同数据库之间移植程序，特别从支持序列的数据库移植到不支持序列的数据库需要修改[配置文件](https://baike.baidu.com/item/配置文件)。

**Native**

Native主键生成方式会根据不同的底层数据库自动选择Identity、Sequence、Hilo主键生成方式

特点：根据不同的底层数据库采用不同的主键生成方式。由于Hibernate会根据底层数据库采用不同的映射方式，因此便于程序移植，项目中如果用到多个数据库时，可以使用这种方式。

**UUID**

UUID使用128位UUID算法生成主键，能够保证网络环境下的主键唯一性，也就能够保证在不同数据库及不同服务器下主键的唯一性。特点：能够保证数据库中的主键唯一性，生成的主键占用比较多的存贮空间

**Foreign GUID**

Foreign用于一对一关系中。GUID主键生成方式使用了一种特殊算法，保证生成主键的唯一性，支持SQL Server和MySQL

## 9. 包的作用

[编辑](javascript:;)[ 播报](javascript:;)

**net.sf.hibernate.\***

该包的类基本上都是接口类和异常类

**net.sf.hibernate.cache.\***

JCS的实现类

**net.sf.hibernate.cfg.\***

[配置文件](https://baike.baidu.com/item/配置文件)读取类

**net.sf.hibernate.collection.\***

Hibernate集合接口实现类，例如List，Set，Bag等等，Hibernate之所以要自行编写集合接口实现类是为了支持lazy loading

**net.sf.hibernate.connection.\***

几个[数据库连接池](https://baike.baidu.com/item/数据库连接池)的Provider

**net.sf.hibernate.dialect.\***

支持多种数据库特性，每个Dialect实现类代表一种数据库，描述了该数据库支持的数据类型和其它特点，例如是否有AutoIncrement，是否有Sequence，是否有[分页](https://baike.baidu.com/item/分页)sql等等

**net.sf.hibernate. eg.\***

Hibernate文档中用到的例子

**net.sf.hibernate.engine.\***

这个包的类作用比较散

**net.sf.hibernate.expression.\***

HQL支持的表达式

**net.sf.hibernate.hq.\***

HQL实现

**net.sf.hibernate. id.\***

ID生成器

**net.sf.hibernate.impl.\***

最核心的包，一些重要接口的实现类，如Session，SessionFactory，Query等

**net.sf.hibernate.jca.\***

[JCA](https://baike.baidu.com/item/JCA)支持，把Session包装为支持JCA的接口实现类

**net.sf.hibernate.jmx.\***

JMX是用来编写App Server的[管理程序](https://baike.baidu.com/item/管理程序)的，大概是JMX部分接口的实现，使得App Server可以通过JMX接口管理Hibernate

**net.sf.hibernate.loader.\***

也是很核心的包，主要是生成sql语句。

**net.sf.hibernate.lob.\***

Blob和Clob支持

**net.sf.hibernate.mapping.\***

hbm文件的属性实现

**net.sf.hibernate.metadata.\***

PO的Meta实现

**net.sf.hibernate.odmg.\***

ODMG是一个ORM标准，这个包是ODMG标准的实现类

**net.sf.hibernate.persister.\***

核心包，实现持久对象和表之间的映射

**net.sf.hibernate.proxy.\***

Proxy和Lazy Loading支持

**net.sf.hibernate. ps.\***

该包是PreparedStatment Cache

**net.sf.hibernate.sql.\***

生成JDBC [sql语句](https://baike.baidu.com/item/sql语句)的包

**net.sf.hibernate.test.\***

测试类，你可以用[junit](https://baike.baidu.com/item/junit)来测试Hibernate

**net.sf.hibernate.tool.hbm2ddl.\***

用hbm[配置文件](https://baike.baidu.com/item/配置文件)生成DDL

**net.sf.hibernate.transaction.\***

Hibernate Transaction实现类

**net.sf.hibernate.type.\***

Hibernate中定义的持久对象的属性的数据类型

**net.sf.hibernate.util.\***

一些工具类，作用比较散

**net.sf.hibernate.xml.\***

XML数据绑定

## 10. 缓存管理

[编辑](javascript:;)[ 播报](javascript:;)

Hibernate 中提供了两级Cache（[高速缓冲存储器](https://baike.baidu.com/item/高速缓冲存储器/9027270)），第一级别的缓存是Session级别的缓存，它是属于[事务](https://baike.baidu.com/item/事务)范围的缓存。这一级别的缓存由hibernate管理的，一般情况下无需进行干预；第二级别的缓存是SessionFactory级别的缓存，它是属于进程范围或集群范围的缓存。这一级别的缓存可以进行配置和更改，并且可以动态加载和[卸载](https://baike.baidu.com/item/卸载)。 Hibernate还为查询结果提供了一个查询缓存，它依赖于第二级缓存。

事务范围，每个事务都有单独的第一级缓存进程范围或集群范围，缓存被同一个进程或集群范围内的所有事务共享 并发访问策略由于每个事务都拥有单独的第一级缓存，不会出现并发问题，无需提供并发访问策略由于多个事务会同时访问第二级缓存中相同数据，因此必须提供适当的并发访问策略，来保证特定的[事务隔离级别](https://baike.baidu.com/item/事务隔离级别)数据过期策略没有提供数据过期策略。处于一级缓存中的对象永远不会过期，除非应用程序显式清空缓存或者清除特定的对象必须提供数据过期策略，如基于内存的缓存中的对象的最大数目，允许对象处于缓存中的最长时间，以及允许对象处于缓存中的最长空闲时间。

物理存储介质内存和硬盘 对象的散装数据首先存放在基于内存的缓存中，当内存中对象的数目达到数据过期策略中指定上限时，就会把其余的对象写入基于硬盘的缓存中。

缓存的软件实现 在Hibernate的Session的实现中包含了缓存的实现由第三方提供，Hibernate仅提供了缓存适配器(CacheProvider)。用于把特定的缓存[插件](https://baike.baidu.com/item/插件)集成到Hibernate中。启用缓存的方式只要应用程序通过Session接口来执行保存、更新、删除、加载和查询数据库数据的操作，Hibernate就会启用第一级缓存，把数据库中的数据以对象的形式拷贝到缓存中，对于批量更新和批量删除操作，如果不希望启用第一级缓存，可以绕过Hibernate API，直接通过JDBC　API来执行指操作。用户可以在单个类或类的单个集合的粒度上配置第二级缓存。如果类的实例被经常读但很少被修改，就可以考虑使用第[二级缓存](https://baike.baidu.com/item/二级缓存)。只有为某个类或集合配置了第二级缓存，Hibernate在运行时才会把它的实例加入到第二级缓存中。 用户管理缓存的方式第一级缓存的物理介质为内存，由于内存容量有限，必须通过恰当的检索策略和检索方式来限制加载对象的数目。Session的evict()方法可以显式清空缓存中特定对象，但这种方法不值得推荐。 第二级缓存的物理介质可以是内存和硬盘，因此第二级缓存可以存放大量的数据，数据过期策略的maxElementsInMemory属性值可以控制内存中的对象数目。管理第二级缓存主要包括两个方面：选择需要使用第二级缓存的[持久类](https://baike.baidu.com/item/持久类)，设置合适的并发访问策略：选择缓存适配器，设置合适的数据过期策略。

### 10.1 一级缓存

当应用程序调用Session的save()、update()、saveOrUpdate()、get()或load()，以及调用查询接口的 list()、iterate()或filter()方法时，如果在Session缓存中还不存在相应的对象，Hibernate就会把该对象加入到第一级缓存中。当清理缓存时，Hibernate会根据缓存中对象的状态变化来同步更新数据库。 Session为应用程序提供了两个管理缓存的方法： evict(Object obj)：从缓存中清除参数指定的持久化对象。 clear()：清空缓存中所有持久化对象。

### 10.2 二级缓存

3.1. Hibernate的二级缓存策略的一般过程如下：

- 条件查询的时候，总是发出一条select * from table_name where …. （选择所有字段）这样的SQL语句查询数据库，一次获得所有的[数据对象](https://baike.baidu.com/item/数据对象)。

- 把获得的所有数据对象根据ID放入到第二级缓存中。

- 当Hibernate根据ID访问数据对象的时候，首先从Session一级缓存中查；查不到，如果配置了[二级缓存](https://baike.baidu.com/item/二级缓存)，那么从二级缓存中查；查不到，再查询数据库，把结果按照ID放入到缓存。

- 删除、更新、增加数据的时候，同时更新缓存。

Hibernate的二级缓存策略，是针对于ID查询的缓存策略，对于条件查询则毫无作用。为此，Hibernate提供了针对条件查询的Query Cache。

3.2. 什么样的数据适合存放到第二级缓存中？ 1 很少被修改的数据 2 不是很重要的数据，允许出现偶尔并发的数据 3 不会被并发访问的数据 4 参考数据,指的是供应用参考的常量数据，它的实例数目有限，它的实例会被许多其他类的实例引用，实例极少或者从来不会被修改。

3.3. 不适合存放到第二级缓存的数据？ 1 经常被修改的数据 2 财务数据，绝对不允许出现并发 3 与其他应用共享的数据。

3.4. 常用的缓存[插件](https://baike.baidu.com/item/插件) Hibernater 的二级缓存是一个插件，下面是几种常用的缓存插件：

- EhCache：可作为进程范围的[缓存](https://baike.baidu.com/item/缓存)，存放数据的物理介质可以是内存或硬盘，对Hibernate的查询缓存提供了支持。

- OSCache：可作为进程范围的缓存，存放数据的物理介质可以是内存或硬盘，提供了丰富的缓存数据过期策略，对Hibernate的查询缓存提供了支持。

- SwarmCache：可作为群集范围内的缓存，但不支持Hibernate的查询缓存。

- [JBossCache](https://baike.baidu.com/item/JBossCache)：可作为群集范围内的[缓存](https://baike.baidu.com/item/缓存)，支持[事务](https://baike.baidu.com/item/事务)型并发访问策略，对Hibernate的查询缓存提供了支持。

上述4种缓存[插件](https://baike.baidu.com/item/插件)的对比情况列于表9-3中。

表9-3 4种缓存插件的对比情况

| 缓 存 插 件 | 支 持 只 读 | 支持非严格读写 | 支 持 读 写 | 支 持 事 务 |
| ----------- | ----------- | -------------- | ----------- | ----------- |
| EhCache     | 是          | 是             | 是          |             |
| OSCache     | 是          | 是             | 是          |             |
| SwarmCache  | 是          | 是             |             |             |
| JBossCache  | 是          |                |             | 是          |

它们的提供器列于表9-4中。

表9-4 [缓存](https://baike.baidu.com/item/缓存)策略的提供器

| 缓 存 插 件                 | 提供器（Cache Providers）                  |
| --------------------------- | ------------------------------------------ |
| Hashtable（只能测试时使用） | org.hibernate.cache.HashtableCacheProvider |
| EhCache                     | org.hibernate.cache.EhCacheProvider        |
| OSCache                     | org.hibernate.cache.OSCacheProvider        |

在默认情况下，Hibernate使用EhCache进行JVM级别的缓存。用户可以通过设置Hibernate[配置文件](https://baike.baidu.com/item/配置文件)中的hibernate.cache.provider_class的属性，指定其他的缓存策略，该缓存策略必须实现org.hibernate.cache.CacheProvider接口。配置[二级缓存](https://baike.baidu.com/item/二级缓存)的主要步骤：

1. 选择需要使用二级缓存的[持久化类](https://baike.baidu.com/item/持久化类)，设置它的命名缓存的并发访问策略。这是最值得认真考虑的步骤。

2. 选择合适的缓存[插件](https://baike.baidu.com/item/插件)，然后编辑该插件的配置文件。

## 11. 延迟加载

Hibernate[对象关系映射](https://baike.baidu.com/item/对象关系映射)提供延迟的与非延迟的对象初始化。非[延迟加载](https://baike.baidu.com/item/延迟加载)在读取一个对象的时候会将与这个对象所有相关的其他对象一起读取出来。这有时会导致成百的（如果不是成千的话）select语句在读取对象的时候执行。这个问题有时出现在使用双向关系的时候，经常会导致整个数据库都在初始化的阶段被读出来了。当然，你可以不厌其烦地检查每一个对象与其他对象的关系，并把那些最昂贵的删除，但是到最后，我们可能会因此失去了本想在ORM工具中获得的便利。

一个明显的解决方法是使用Hibernate提供的延迟加载机制。这种初始化策略只在一个对象调用它的一对多或[多对多关系](https://baike.baidu.com/item/多对多关系/665737)时才将关系对象读取出来。这个过程对开发者来说是透明的，而且只进行了很少的数据库操作请求，因此会得到比较明显的性能提升。这项技术的一个缺陷是[延迟加载](https://baike.baidu.com/item/延迟加载)技术要求一个Hibernate会话要在对象使用的时候一直开着。这会成为通过使用DAO模式将[持久层](https://baike.baidu.com/item/持久层)抽象出来时的一个主要问题。为了将持久化机制完全地抽象出来，所有的数据库逻辑，包括打开或关闭会话，都不能在[应用层](https://baike.baidu.com/item/应用层)出现。最常见的是，一些实现了简单接口的DAO实现类将数据库逻辑完全封装起来了。一种快速但是笨拙的解决方法是放弃DAO模式，将数据库连接逻辑加到应用层中来。这可能对一些小的应用程序有效，但是在大的系统中，这是一个严重的设计缺陷，妨碍了系统的可扩展性。

幸运的是，Spring框架为Hibernate[延迟加载](https://baike.baidu.com/item/延迟加载)与DAO模式的整合提供了一种方便的解决方法。以一个Web应用为例，Spring提供了OpenSessionInViewFilter和OpenSessionInViewInterceptor。我们可以随意选择一个类来实现相同的功能。两种方法唯一的不同就在于interceptor在Spring容器中运行并被配置在web应用的上下文中，而Filter在Spring之前运行并被配置在[web.xml](https://baike.baidu.com/item/web.xml)中。不管用哪个，他们都在请求将当前会话与当前（数据库）线程绑定时打开Hibernate会话。一旦已绑定到线程，这个打开了的Hibernate会话可以在DAO实现类中透明地使用。这个会话会为[延迟加载](https://baike.baidu.com/item/延迟加载)数据库中值对象的[视图](https://baike.baidu.com/item/视图)保持打开状态。一旦这个逻辑视图完成了，Hibernate会话会在Filter的doFilter方法或者Interceptor的postHandle方法中被关闭。

- 实现方法在web.xml中加入

  ```xml
  <filter>
      <filter-name>hibernateFilter</filter-name>
      <filter-class>
          org.springframework.orm.hibernate3.support.OpenSessionInViewFilter
      </filter-class>
  </filter>
      <filter-mapping>
      	<filter-name>hibernateFilter</filter-name>
      <url-pattern>*.do</url-pattern>
  </filter-mapping>
  ```

  

## 12. 性能优化

初用HIBERNATE的人也许都遇到过性能问题，实现同一功能，用HIBERNATE与用JDBC性能相差十几倍很正常，如果不及早调整，很可能影响整个项目的进度。 大体上，对于HIBERNATE性能调优的主要考虑点如下：

- [数据库设计](https://baike.baidu.com/item/数据库设计/222831)调整

- HQL优化

- API的正确使用(如根据不同的业务类型选用不同的集合及查询API)

- 主配置参数(日志，查询缓存，fetch_size, batch_size等)

- 映射文件优化(ID生成策略，二级缓存，[延迟加载](https://baike.baidu.com/item/延迟加载)，关联优化)

- [一级缓存](https://baike.baidu.com/item/一级缓存)的管理

- 针对[二级缓存](https://baike.baidu.com/item/二级缓存)，还有许多特有的策略

- 事务控制策略。

### 12.1 数据库设计

- 降低关联的复杂性

- 尽量不使用联合主键- 

- ID的生成机制，不同的数据库所提供的机制并不完全一样

- 适当的冗余数据，不过分追求高范式

### 12.2 HQL优化

HQL如果抛开它同HIBERNATE本身一些缓存机制的关联，HQL的优化技巧同普通的SQL优化技巧一样，可以很容易在网上找到一些经验之谈。

### 12.3 主配置

- 查询缓存，同下面讲的缓存不太一样，它是针对HQL语句的缓存，即完全一样的语句再次执行时可以利用缓存数据。但是，查询缓存在一个交易系统(数据变更频繁，查询条件相同的机率并不大)中可能会起反作用:它会白白耗费大量的系统资源但却难以派上用场。

- etch_size，同JDBC的相关参数作用类似，参数并不是越大越好，而应根据业务特征去设置

- batch_size同上。

- 生产系统中，切记要关掉SQL语句打印。

### 12.4 缓存

- 数据库级缓存:这级缓存是最高效和安全的，但不同的数据库可管理的层次并不一样，比如，在Oracle中，可以在建表时指定将整个表置于缓存当中。

- SESSION缓存:在一个HibernateSESSION有效，这级缓存的可干预性不强，大多于HIBERNATE自动管理，但它提供清除缓存的方法，这在大批量增加/更新操作是有效的。比如，同时增加十万条记录，按常规方式进行，很可能会发现OutofMemeroy的异常，这时可能需要手动清除这一级缓存:Session.evict以及 Session.clear

- 应用缓存:在一个SESSIONFACTORY中有效，因此也是优化的重中之重，因此，各类策略也考虑的较多，在将数据放入这一级缓存之前，需要考虑一些前提条件：

  1. 数据不会被第三方修改(比如，是否有另一个应用也在修改这些数据?)
  2. 数据不会太大
  3. 数据不会频繁更新(否则使用CACHE可能适得其反)
  4. 数据会被频繁查询
  5. 数据不是关键数据(如涉及钱，安全等方面的问题)。

  [缓存](https://baike.baidu.com/item/缓存)有几种形式，可以在映射文件中配置:read-only(只读，适用于很少变更的静态数据/历史数据)，nonstrict-read- write，read-write(比较普遍的形式，效率一般)，transactional(JTA中，且支持的缓存产品较少)

- 分布式缓存:同c)的配置一样，只是缓存产品的选用不同，[oscache](https://baike.baidu.com/item/oscache), jboss cache，的大多数项目，对它们的用于[集群](https://baike.baidu.com/item/集群)的使用(特别是关键交易系统)都持保守态度。在集群环境中，只利用数据库级的缓存是最安全的。

### 12.5 延迟加载

- 实体[延迟加载](https://baike.baidu.com/item/延迟加载):通过使用动态代理实现

- 集合延迟加载:通过实现自有的SET/LIST，HIBERNATE提供了这方面的支持

- 属性延迟加载:

### 12.6 方法选用

- 完成同样一件事，Hibernate提供了可供选择的一些方式，使用不同的编码方式对性能有不同的影响。比如：一次返回十万条记录，如果用 (List/Set/Bag/Map等)进行处理，很可能导致内存不够的问题，而如果用基于[游标](https://baike.baidu.com/item/游标)(ScrollableResults)或 Iterator的[结果集](https://baike.baidu.com/item/结果集)，则不存在这样的问题。

- Session的load/get方法，前者会使用[二级缓存](https://baike.baidu.com/item/二级缓存)，而后者则不使用。

- Query和list/iterator，如果去仔细研究一下它们，你可能会发现很多有意思的情况，二者主要区别(如果使用了Spring，在HibernateTemplate中对应find,iterator方法):

  1. list只能利用查询缓存(但在交易系统中查询缓存作用不大)，无法利用二级缓存中的单个实体，但list查出的对象会写入二级缓存，但它一般只生成较少的执行SQL语句，很多情况就是一条(无关联)。

  2. iterator则可以利用二级缓存，对于一条查询语句，它会先从数据库中找出所有符合条件的记录的ID，再通过ID去缓存找，对于缓存中没有的记录，再构造语句从数据库中查出，因此很容易知道，如果缓存中没有任何符合条件的记录，使用iterator会产生N+1条SQL语句(N为符合条件的记录数)

  3. 通过iterator，配合缓存管理API，在海量数据查询中可以很好的解决内存问题，如:

     ```java
     while(it.hasNext()){
         YouObject object = (YouObject)it.next();
         session.evict(youObject);
         sessionFactory.evice(YouObject.class, youObject.getId());
     }
     
     
     ```

     如果用list方法，很可能就出OutofMemory错误了。

### 12.7 集合的选用

在Hibernate3.1文档的“19.5. Understanding Collection performance”中有详细的说明。

### 12.8 事务控制

事务方面对性能有影响的主要包括:事务方式的选用，[事务隔离级别](https://baike.baidu.com/item/事务隔离级别)以及锁的选用

- [事务](https://baike.baidu.com/item/事务)方式选用:如果不涉及多个事务管理器事务的话，不需要使用JTA，只有JDBC的事务控制就可以。

- 事务隔离级别:参见标准的SQL事务隔离级别

- 锁的选用:[悲观锁](https://baike.baidu.com/item/悲观锁)(一般由具体的事务管理器实现)，对于长事务效率低，但安全。[乐观锁](https://baike.baidu.com/item/乐观锁)(一般在应用级别实现)，如在HIBERNATE中可以定义 VERSION字段，显然，如果有多个应用操作数据，且这些应用不是用同一种乐观锁机制，则乐观锁会失效。因此，针对不同的数据应有不同的策略，同前面许多情况一样，很多时候我们是在效率与安全/准确性上找一个平衡点，无论如何，优化都不是一个纯技术的问题，你应该对你的应用和业务特征有足够的了解。

### 12.9 批量操作

即使是使用JDBC，在进行大批[数据更新](https://baike.baidu.com/item/数据更新)时，BATCH与不使用BATCH有效率上也有很大的差别。可以通过设置`batch_size`来让其支持批量操作。

举个例子，要批量删除某表中的对象，如“delete Account”，打出来的语句，HIBERNATE找出了所有ACCOUNT的ID，再进行删除，这主要是为了维护二级缓存，这样效率肯定高不了，在后续的版本中增加了bulk delete/update，但这也无法解决缓存的维护问题。也就是说，由于有了二级缓存的维护问题，HIBERNATE的批量操作效率并不尽如人意。

**hibernate工作原理：**

1. 通过Configuration().configure();读取并解析hibernate.cfg.xml[配置文件](https://baike.baidu.com/item/配置文件)。

2. 由hibernate.cfg.xml中的<mappingresource="com/xx/User.hbm.xml"/>读取解析映射信息。

3. 通过config.buildSessionFactory();//得到sessionFactory。

4. sessionFactory.openSession();//得到session。

5. session.beginTransaction();//开启事务。

6. persistent operate;

7. session.getTransaction().commit();//提交事务

8. 关闭session;

9. 关闭sessionFactory;

**hibernate优点：**

1. 封装了jdbc，简化了很多重复性代码。

2. 简化了DAO层编码工作，使开发更对象化了。

3. 移植性好，支持各种数据库，如果换个数据库只要在[配置文件](https://baike.baidu.com/item/配置文件)中变换配置就可以了，不用改变hibernate代码。

4. 支持透明持久化，因为hibernate操作的是纯粹的（pojo）java类，没有实现任何接口，没有侵入性。所以说它是一个轻量级框架。

**hibernate**[延迟加载](https://baike.baidu.com/item/延迟加载)：

get不支持延迟加载，load支持延迟加载。

1. hibernate2对 实体对象和集合 实现了延迟加载

2. hibernate3对 提供了属性的延迟加载功能

hibernate延迟加载就是当使用session.load(User.class,1)或者session.createQuery()查询对象或者属性的时候

这个对象或者属性并没有在内存中，只有当程序操作数据的时候，才会存在内存中，这样就实现[延迟加载](https://baike.baidu.com/item/延迟加载)，节省了内存的开销，从而提高了服务器的性能。

**Hibernate的缓存机制**

一级缓存：session级的缓存也叫事务级的缓存，只缓存实体，生命周期和session一致。不能对其进行管理。

不用显式的调用。

二级缓存：sessionFactory缓存，也叫进程级的缓存，使用第3方[插件](https://baike.baidu.com/item/插件)实现的，也只缓存实体，生命周期和sessionFactory一致，可以进行管理。

首先配置第3方插件，我们用的是EHCache，在hibernate.cfg.[xml文件](https://baike.baidu.com/item/xml文件/1994443)中加入

<propertyname="hibernate.cache.user_second_level_cache">true</property>

在映射中也要显式的调用，<cacheusage="read-only"/>

二级缓存之查询缓存：对普通属性进行缓存。如果关联的表发生了修改，那么查询缓存的生命周期也结束了。

在程序中必须手动启用查询缓存：query.setCacheable(true);

**优化Hibernate**

1. 使用一对多的双向关联，尽量从多的一端维护。

2. 不要使用一对一，尽量使用多对一。

3. 配置对象缓存，不要使用集合缓存。

4. 表字段要少，表关联不要怕多，有二级缓存撑腰。

**hibernate 类与类之间关系**

- 关联关系

- 聚集关系

- 继承关系

Hibernate继承关系映射策略分为三种：一张表对应一整棵类继承树、一个类对应一张表、每一个具体类对应一张表。

![概述图册](https://bkimg.cdn.bcebos.com/pic/d009b3de9c82d158cb08fea58f0a19d8bc3e421f?x-bce-process=image/resize,m_lfit,w_235,h_235,limit_1/format,f_auto)



参照URL：

- https://baike.baidu.com/item/Hibernate/206989?fr=aladdin