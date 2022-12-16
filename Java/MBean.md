# MBean



描述一个可管理的资源。是一个java对象，遵循以下一些规则：

1. 必须是公用的，非抽象的类 ；
2. 必须有至少一个公用的[构造器](https://baike.baidu.com/item/构造器/9844976) ；
3. 必须实现它自己的相应的MBean接口或者实现javax.management.DynamicMBean接口；
4. 可选的，一个MBean可以实现javax.management.NotificationBroadcaster接口MBean的类型。

## MBean的类型

- 标准MBean

- 动态MBean

- 模型MBean

- 开放MBean

## 注册 MBean

一个MBeanServer的主要职责是在一个[JMX](https://baike.baidu.com/item/JMX)代理中维护一个MBean的[注册表](https://baike.baidu.com/item/注册表/101856)。能够以下面两个方法中的任意一个注册：

1. 创建一个MBean实例并注册它到MBeanServer用方法：

   public ObjectInstance registerMBean(Object object， ObjectName name) 这里，object是创建的MBean实例，name部分是MBean的一个唯一标志符。

2. 使用createMBean 方法中的一个：

   createMBean方法使用java自身来创建一个MBean实例。对所有的createMBean方法，有两个变量是通用的。

   - String classname -- 要创建的MBean的实例的类名。

   - ObjectName name -- MBean要注册的对象名称。
   
   如果用两个变量的createMBean[构造器](https://baike.baidu.com/item/构造器)，缺省的类装载器(class loader)用来装载MBean类，MBean被使用缺省的构造器[初始化](https://baike.baidu.com/item/初始化/100108)。如果MBean已经被初始化，这个实例将被MBeanServer用第二个变量的对象名称注册。如果你想使用指定的类装载器，那么可以使用三个变量的构造器。这个类装载器将被MBeanServer注册为一个MBean。这个类装载器的对象名称将用做第三个变量。

当实例化一个类，如果任何参数必须传入，那么可以使用四个参数的createMBean方法。这个createMBean方法的最后的两个参数分别包含[对象数组](https://baike.baidu.com/item/对象数组)(类的初始化必须的)和他们的署名。如果MBean类必须用其他指定的类装载器装载，那么应该使用五个参数的[构造器](https://baike.baidu.com/item/构造器)。

## 注销MBean

MBean能够用MBeanServer 的下面方法注销：

- public void unregisterMBean(ObjectName name) 这里name 是这个MBean实例注册的对象名称。

  注销MBean后， MBeanServer将不再保存任何这个MBean实例的关联。

- 控制MBeanRegistration

  JMX定义了一个接口叫MBeanRegistration。这个接口的目的是允许MBean开发者对在MBeanServer上注册和注销进行一些控制。这通过MBean实现javax.management.MBeanRegistration接口来达到。

MBeanRegistration接口定义注册控制机制的行为。它定义了以下四个方法：

- public ObjectName preRegister(MBeanServer mbs, ObjectName name) 
- public void postRegister() 
- public void preDeRegister() 
- public void postDeRegister()

所有以上的方法都是[回调](https://baike.baidu.com/item/回调/9837525)方法，

MBeanServer将在恰当的时机调用这些方法。

- 如果一个MBean实现MBeanRegistration并且这个MBean被注册，MBeanServer在注册前调用 preRegister方法。这个方法返回的对象名称将在MBean注册过程中使用。
- 在成功完成MBean注册后，MBeanServer调用postRegister方法。
- 如果以上的MBean被注销，在注销前MBeanServer调用preDeRegister方法。
- 如果注销成功，MBeanServer调用postDeRegister方法.

## 注意事项

于一个单一MBean类，多个实例能够被创建，并能够被注册为注册时MBean提供的对象名称。

无论何时一个MBean被注册，MBeanServer创建一个类型为 jmx.mbean.created 的消息。 MBeanServerDelegate MBean 广播这个消息到所有注册的监听者。

MBeanRegistration接口提供注册和注销过程监控的钩点。