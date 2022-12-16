# @Component注解的用法



## 遇到的问题

踩到一个坑，有一个接口，在这个接口的实现类里，需要用到@Autowired注解，一时大意，没有在实现类上加上@Component注解，导致了Spring报错，找不到这个类

一旦使用关于Spring的注解出现在类里，例如我在实现类中用到了@Autowired注解，被注解的这个类是从Spring容器中取出来的，那调用的实现类也需要被Spring容器管理，加上@Component

```java
@Component("conversionImpl")
//其实默认的spring中的Bean id 为 conversionImpl(首字母小写)
public class ConversionImpl implements Conversion {
    @Autowired
    private RedisClient redisClient;
}
```

## 介绍

开发中难免会遇到这个这个注解@Component

- @Controller 控制器（注入服务）
   用于标注控制层，相当于struts中的action层

- @Service 服务（注入dao）
   用于标注服务层，主要用来进行业务的逻辑处理

- @Repository（实现dao访问）
   用于标注数据访问层，也可以说用于标注数据访问组件，即DAO组件
   .

- @Component （把普通pojo实例化到spring容器中，相当于配置文件中的 ）

  泛指各种组件，就是说当我们的类不属于各种归类的时候（不属于@Controller、@Services等的时候），我们就可以使用@Component来标注这个类。

