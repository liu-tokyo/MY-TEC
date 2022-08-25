# @component的作用详细介绍

最近项目要采用spring boot在学习的spring boot 的过程中第一次见到@[component](https://so.csdn.net/so/search?q=component&spm=1001.2101.3001.7020)注解，特意在网上搜索下，摘录在此方便日后查阅。

## 注释

1. @controller 控制器（注入服务）
   用于标注控制层，相当于[struts](https://so.csdn.net/so/search?q=struts&spm=1001.2101.3001.7020)中的action层

2. @service 服务（注入dao）
   用于标注服务层，主要用来进行业务的逻辑处理

3. @repository（实现dao访问）
   用于标注数据访问层，也可以说用于标注数据访问组件，即DAO组件.

4. @component （把普通pojo实例化到spring容器中，相当于配置文件中的 <bean id="" class=""/>）
   泛指各种组件，就是说当我们的类不属于各种归类的时候（不属于@Controller、@Services等的时候），我们就可以使用@Component来标注这个类。

## 说明： 

下面写这个是引入component的扫描组件 （这是在配置文件中的书写格式,如spring mvc中的applicationcontent.xml，在spring boot中的话，因采用的是零配置所以要直接在类上加入@component注解就可以了）

```
<context:component-scan base-package=”com.mmnc”> 
```

上面的这个例子是引入Component组件的例子，其中base-package表示为需要扫描的所有子包。 
共同点：被@controller 、@service、@repository 、@component 注解的类，都会把这些类纳入进spring容器中进行管理

 

转载于:https://www.cnblogs.com/w-essay/p/11493023.html