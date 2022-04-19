# 万字详解！Spring Bean的自动装配

## 何为自动装配

自动装配是 Spring 满足 bean 依赖的一种方式。

在使用 Spring 配置 bean 时,我们都要给配置的 bean 的属性设置一个值,如果不手动设置则都是空。而自动的好处就在于，我们不用手动去设置一个值，spring 会在上下文中自动寻找并装配合适的值。

在 Spring 中有三种装配的方式：

- 在 XML 中显示配置
- 在 Java 代码中显示的配置
- 隐式的自动装配

本文重点放在隐式自动装配的说明，手动装配的部分可以参照Spring对象如何创建与管理,又如何使用巧妙的方法注入属性呢?

## 环境

测试自动装配前，先来定义好三个 POJO 类，Cat、Dog、Person。并且有一个关系就是 Person 拥有狗和猫。

```java
package com.javastudyway.pojo;
public class Cat {
    public void shout(){
        System.out.println("我是猫");
    }
}

package com.javastudyway.pojo;
public class Dog {
    public void shout(){
        System.out.println("I am Dog");
    }
}

package com.javastudyway.pojo;

import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class Person {
    /* 人有猫和狗 */
    private Cat cat;
    private Dog dog;
    private String name;

    /* getter/setter 和 toString */
    public Cat getCat() {
        return cat;
    }
    
    public void setCat(Cat cat) {
        this.cat = cat;
    }
    
    public Dog getDog() {
        return dog;
    }
    
    public void setDog(Dog dog) {
        this.dog = dog;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    @Override
    public String toString() {
        return "People{" +
                "Cat = " + cat +
                ", Dog = " + dog +
                ", name = '" + name + '\'' +
                '}';
    }
}
```

Spring 配置文件部分：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="cat" class="com.javastudyway.pojo.Cat"/>
    <bean id="dog" class="com.javastudyway.pojo.Dog"/>
    <bean id="person" class="com.javastudyway.pojo.Person">
        <property name="name" value="Java学习之道"/>
        <property name="cat" ref="cat"/>
        <property name="dog" ref="dog"/>
    </bean>
</beans>
```

最后是测试类（MyTest）：

```java
import com.javastudyway.pojo.Person;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MyTest {
    @Test
    public void test1(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        Person person = context.getBean("person", Person.class);
        person.getDog().shout();
        person.getCat().shout();
    }
}
```

测试普通装配的结果：装配没有问题。

![img](https://pic.rmb.bdstatic.com/bjh/news/a5a0954447df08676fdc1d5e2fb32c3e.png)

## 配置实现自动装配

自动装配有三种方式：

- byName
- byType
- constructor

在配置 bean 时，我们可以加入一个 **「autowired」** 属性，来使用自动装配。

### byName

`byName` 通过匹配 bean 的 id是否跟 setter 对应，对应则自动装配。

意思就是说，如果我的 Person 中有一个 `setCat()` 而配置文件中有一个 **「bean 的 id 为 cat」**，则能够自动装配。

把刚刚手动装配方式的 bean 做一些修改，来演示自动装配：

```xml
<bean id="cat" class="com.javastudyway.pojo.Cat"/>
<bean id="dog" class="com.javastudyway.pojo.Dog"/>
<!--加上了 person 的 bean 的 autowired 属性
    并去掉了手动装配的 cat 和 dog -->
<bean id="person" class="com.javastudyway.pojo.Person" autowire="byName">
    <property name="name" value="Java学习之道"/>
</bean>
```

刚刚的测试类运行结果依旧正常，表明了自动装配的确是运行了。

![img](https://pic.rmb.bdstatic.com/bjh/news/a5a0954447df08676fdc1d5e2fb32c3e.png)

如若我 bean 的 id 跟 setter 不对应呢？

```xml
<bean id="dog11" class="com.javastudyway.pojo.Dog"/>
```

把刚刚的 dog id改掉，其余不变再来测试：

![img](https://pic.rmb.bdstatic.com/bjh/news/0fec08e828ea44cfa0c6ad9031fbf656.png)

结果是出现了空指针异常，说明没有对象，也就反映出了自动装配失败了。

### byType

`byType` 通过匹配 bean 中所需要的依赖类型在容器上下文中自动寻找装配。

byName 方式 dog11 自动装配失败了，那么 `byType` 呢?

稍稍改动一下 `person`，把自动装配方式改为 **「byType」**:

```xml
<bean id="person" class="com.javastudyway.pojo.Person" autowire="byType">
    <property name="name" value="Java学习之道"/>
</bean>
```

![img](https://pic.rmb.bdstatic.com/bjh/news/ee1a71e7132006e89e26f236e3f71e78.png)

空指针异常不再出现，自动装配成功！

自动装配跟 id 无关？它真的还就没有关系，甚至在配置 cat 和 dog 这两个 bean 时不写 id 都不会报错。

但是 `byType` 的自动装配存在一个很严重的问题，因为不是通过唯一的 id 来匹配，而是通过类型来匹配，所以容器中不能存在多个相同类型的 bean。

![img](https://pic.rmb.bdstatic.com/bjh/news/3b92d6963959add937584cf58a524adb.png)这是官方文档中给出的自动装配的说明，Spring 官方也不是很支持使用 byType 来实现自动装配，使用唯一标识 id 来匹配也确实更为安全。

### 优异

byName 和 byType 也不能非说孰强孰弱，只是各有各的好处。

> 使用 byName 需要保证 bean 的 id 唯一，且这个 bean 需要自动注入的属性和set方法与 bean 的 id 要一致。
>
> 使用 byType 需要保证 bean 的 class 唯一，且这个 bean 需要自动注入的属性和类型要一致。

说到这里，对于自动装配也有一个初步的感悟，接下来就来使用**「注解开发」**的方式来实现自动装配。

## 使用注解实现自动装配

注解在 JDK1.5 之后被引入，而 Spring2.5 版本后也开始支持注解了。这些版本也都可以算得上远古版本，所以不用担心版本问题了。

![img](https://pic.rmb.bdstatic.com/bjh/news/3d41cc51fd3958b4fd606753376b86ff.png)关于使用`注解`还是 `XML` 官方给出了答案，使用注解比使用 XML 配置更好。

不多吹，下面开始演示注解。

### 使用注解的准备

使用注解我们还需要对原本的配置文件做一些修改：

- 导入约束导入约束，即增加 **「context」** 的约束。
- 配置注解支持在配置中加入 -- **「`<context:annotation-config/>`」**。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 开启注解支持 -->
    <context:annotation-config/>

    <bean id="cat" class="com.javastudyway.pojo.Cat"/>
    <bean id="dog" class="com.javastudyway.pojo.Dog"/>
    <bean id="person" class="com.javastudyway.pojo.Person"/>

</beans>
```

这样一来，我们的配置文件就变得极度简洁了，剩下的事情全部都交给注解来搞定就行。

### @AutoWired注解的使用

搞定了配置文件后就可以开始使用注解了，在 POJO 类需要自动装配的属性上加 `@AutoWired` 注解实现自动装配。![img](https://pic.rmb.bdstatic.com/bjh/news/68149941df9ab44f772b9c0d976677b7.png)

> 小贴士：
>
> - @AutoWired注解也可以在 setter 上使用；
> - 如果使用了 @AutoWired 注解，POJO 类中的 setter 方法也可以省略，因为自动装配注解是用反射来实现的。

@AutoWired 这个注解还有一个属性，`required`。来看看 @AutoWired 的源码：

```java
public@interface Autowired {
    boolean required() default true;
}
```

**「required」** 的默认值为 `true`，那么，这个值的作用是什么呢？required -- 必需的，如果它的值为 true，则说明这个属性不是必需的(允许为null)。

```java
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
publicclass Person {
    /* 
     * 如此声明 required 为 false，
     * 则该对象可以没有 cat 这个属性
     */
    @Autowired(required = false)
    private Cat cat;
    @Autowired
    private Dog dog;
    private String name;

    public Cat getCat() {
        return cat;
    }

    public Dog getDog() {
        return dog;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return"People{" +
                "Cat = " + cat +
                ", Dog = " + dog +
                ", name = '" + name + '\'' +
                '}';
    }
}
```

### @Qualifier

一般情况下，使用 `@AutoWired` 注解类似于 byType 的装配方式，但是如果我们在配置文件中，配置了同一个类的多个 bean，那么这时候 @AutoWired 该作何选择呢？

如果遇到这个情况我们就需要使用 `@Qualifier` 注解来辅助实现了。先来看看 @Qualifier 注解的源码：

```java
public@interface Qualifier {
    String value() default "";
}
```

可以看到这个注解需要一个 value 的参数，这个参数即为我们要选择的 bean 的 id。

在配置文件中增加一个 dog 的 bean。

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

     <!--开启注解的支持-->
    <context:annotation-config/>

    <bean id="cat" class="com.javastudyway.pojo.Cat"/>
    
    <bean id="dog" class="com.javastudyway.pojo.Dog"/>
    <bean id="dog2" class="com.javastudyway.pojo.Dog"/>
    
    <bean id="person" class="com.javastudyway.pojo.Person" autowire="byType">
        <property name="name" value="Java学习之道"/>
    </bean>

</beans>
```

再搭配上 `@Qualifier` 注解

```java
import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;

publicclass Person {
    @Autowired(required = false)
    private Cat cat;
    @Autowired
    @Qualifier(value = "dog2")
    private Dog dog;
    private String name;

    public Cat getCat() {
        return cat;
    }

    public Dog getDog() {
        return dog;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return"People{" +
                "Cat = " + cat +
                ", Dog = " + dog +
                ", name = '" + name + '\'' +
                '}';
    }
}
```

这样便能在多个同类 bean 实现使用 @AutoWired 自动装配任意一个 bean。

### @Resource

前文说到的两个注解皆是 Spring 的注解，在 Java 中还有一个原生注解，**「@Resource」**,这个注解的作用类似于 @AutoWired 和 @Qualifier的结合体。

在 bean 的自动装配情况下，可以使用到 `@Resource` 的 name 参数，这个 name 的参数相当于 @Qualifier 的 value 参数。

@Resource 与 @AutoWired 的不同在于：

- @Resource 默认是按照 名称 来装配注入的，当找不到与名称匹配的 bean 才会按照类型来装配注入。
- @Autowired 默认是按照 类型 装配注入的，如果想按照名称来装配注入，则需要结合 @Qualifier 一起使用；

将配置的两个 dog 改成 dog1 和 dog2，再来使用 **「@Resource」** 注解。

```java
<bean id="dog1" class="com.javastudyway.pojo.Dog"/>
<bean id="dog2" class="com.javastudyway.pojo.Dog"/>
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import javax.annotation.Resource;

publicclass Person {
    @Resource
    private Cat cat;
    @Resource
    private Dog dog;
    private String name;

    public Cat getCat() {
        return cat;
    }

    public Dog getDog() {
        return dog;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return"People{" +
                "Cat = " + cat +
                ", Dog = " + dog +
                ", name = '" + name + '\'' +
                '}';
    }
}
```

运行的结果为：![img](https://pic.rmb.bdstatic.com/bjh/news/3cfabbe00aa7c0298b04e49c1a2b6453.png)

因为 @Resource 默认是通过 id 来查找 bean，而配置的两个 dog 的 bean 没有一个名称为默认查找的 dog，故而创建失败。

修改 @Resource，加入参数：

```
@Resource(name = "dog1")
private Dog dog;
```

重新测试结果：![img](https://pic.rmb.bdstatic.com/bjh/news/c4418c2443bf2417349407e57d803dad.png)

因为能够通过 name 这个参数匹配到 Dog 类型的 bean，所以程序便不会出现问题。

#### @Resource总结

@Resource 注解默认通过 byName 匹配 bean，如果 id 匹配失败则会通过类型匹配，但是如果同个类型的 bean 不唯一或者该类的 bean 不存在则装配失败。



## 参照资源

- [万字详解！Spring Bean的自动装配 (baidu.com)](https://baijiahao.baidu.com/s?id=1728141477130284488&wfr=spider&for=pc)