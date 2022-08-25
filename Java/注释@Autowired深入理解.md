# 深入理解@Autowired

在开发中经常使用到@Autowired注解，一般用在Service前面，Dao前面，Config前面，但是这个注解究竟是什么作用呢。

网上经常有这样的解释：

> @Autowired 是一个注解，它可以对类成员变量、方法及构造函数进行标注，让 spring 完成 bean 自动装配的工作

但是自动装配做了什么呢，或者说，我直接new一个不可以吗？



实际情况其实更复杂，比如一个系统需要一个“paymentService”的对象做支付，也许整个系统只需要一个paymentService的实例，所以必须得判断那个要用的Object有没有，没有才创建。但有时，对于一些Object可能每次都需要一个新的实例。一个更好的办法是，每个Object可以定义自己的初始化静态工厂方法完成初始化，都封装在里面，然后别的地方都调用这个方法。



## 静态注入
静态注入的本质是弄一个全局的大KV，key是那个Object的类，value就是new出来的所有Object。这时你编写代码的逻辑就是：

1. 创建这个大KV
2. 手工new或者通过静态方法创建第一个没有任何依赖的Object，并记录到大KV里
3. 手工new或者通过静态方法创建其他Object，如果创建时需要依赖已经创建好的Object，就从KV里取出来直接用
4. 所有的Object都创建完了，服务就可以启动了

## 动态注入
另一类被称为运行时注入，又叫做动态注入。代表是Spring和Guice。以Spring为例，你只需要定义两类信息：

- 你要如何初始化Object（如何创建/单例、多例还是要弄个什么Factory/有没有自定义初始化代码等）
- 你的Object依赖什么Object（可以通过class或者name来描述）

其余的Spring都帮你直接搞定。使用Spring的程序一启动就开始根据当前的配置信息，classpath等**各种静态 + 运行时信息**帮你搞定一切。



**以下是@Autowired在实际中的策略：**

- 如果required属性被设置为true，那么只有一个构造方法可以用@Autowired注解。

- 通过在Spring容器中匹配bean可以满足的依赖关系最多的构造方法将被选择。

- 如果不存在满足匹配的候选构造方法，则将会使用primary/default 构造方法

- 如果存在多个同类型的Bean实例，需要注意的是，在使用@Autowired注入时，会根据字段的名称去容器中找实例。所以，OrderController注入的是coffee2实例。



[Guide to Spring @Autowired | Baeldung](https://link.zhihu.com/?target=https%3A//www.baeldung.com/spring-autowire)这里是一个Spring Boot对@Aotowired的教程