# Spring中的singleton和prototype的实现

> 这篇文章主要介绍了Spring中的singleton和prototype的实现，文中通过示例代码介绍的非常详细，对大家的学习或者工作具有一定的参考学习价值，需要的朋友们下面随着小编来一起学习学习吧

关于spring bean作用域，基于不同的容器，会有所不同，如BeanFactory和ApplicationContext容器就有所不同,在本篇文章，主要讲解基于ApplicationContext容器的bean作用域。

关于bean的作用域，在spring中，主要包括singleton,prototype,session,request,global,本篇文章主要讲解常用的两种，即：singleton和prototype.

## **一 singleton**

singleton为单例模式，即scope="singleton"的bean，在容器中，只实例化一次。

- dao示例代码：

  ```java
  package com.demo.dao;
   
  public class UserDao {
   
    public UserDao(){
      System.out.println("UserDao 无参构造函数被调用");
    }
    //获取用户名
    public String getUserName(){
      //模拟dao层
      return "Alan_beijing";
    }
  }
  ```

- applicationContext.xml

  ```java
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
    <bean class="com.demo.dao.UserDao" id="userDao" scope="singleton"/>
  </beans>
  ```

- test:

  ```java
  public class MyTest {
   
    @Test
    public void test(){
      //定义容器并初始化
      ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
   
      //定义第一个对象
      UserDao userDao = applicationContext.getBean(UserDao.class);
      System.out.println(userDao.getUserName());
   
      //定义第二个对象
      UserDao userDao2 = (UserDao) applicationContext.getBean("userDao");
      System.out.println(userDao2.getUserName());
      //比较两个对象实例是否是同一个对象实例
      System.out.println("第一个实例："+userDao+"\n"+"第二个实例："+userDao2);
    }
  }
  ```

- 测试结果：

  ![img](https://img.jbzj.com/file_images/article/202007/202007230833491.png) 

分析：在测试代码中，将bean定义为singleton，并先后2次通过ApplicationContext的getBean()方法获取bean(userDao),却返回相同的实例对象：com.demo.dao.UserDao@27a5f880，仔细观察，虽然获取bean两次，但是UserDao的无参构造函数却只被调用一次，这也证明了在容器中，singleton实际只被实例化一次，需要注意的是，Singleton模式的bean,ApplicationContext加载bean时，就实例化了bean。

- 定义bean:

  ![img](https://img.jbzj.com/file_images/article/202007/202007230833492.png) 

- 测试结果：

  如下代码只是加载bean，却没调用getBean方法获取bean,但UserDao却被调用了一次，即实例化。

  ![img](https://img.jbzj.com/file_images/article/202007/202007230833493.png) 

## **二 prototype**

prototype即原型模式，调用多少次bean，就实例化多少次。

- 将singleton代码改为原型

  ```java
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
   
    <bean class="com.demo.dao.UserDao" id="userDao" scope="prototype"/>
  </beans>
  ```

- 测试代码与singleton一样，但结果却不一样：

  ![img](https://img.jbzj.com/file_images/article/202007/202007230833494.png) 

分析：通过测试结果，不难发现，调用两次bean，就实例化两次UserDao对象，且对象不一样，需要注意的是，prototype类型的bean，只有在获取bean时，才会实例化对象。

## **三 singleton和prototype区别**

1. singleton在容器中，只被实例化一次，而prototype在容器中，调用几次，就被实例化几次；

2. 在AppplicationContext容器中，singleton在applicaitonContext.xml加载时就被预先实例化，而prototype必须在调用时才实例化

### singleton:

- 定义bean:

  ![img](https://img.jbzj.com/file_images/article/202007/202007230833492.png) 

- 测试：

  ![img](https://img.jbzj.com/file_images/article/202007/202007230833493.png) 

### prototype：

- 定义bean:

  ![img](https://img.jbzj.com/file_images/article/202007/202007230833495.png) 

- 测试：不调用

  ![img](https://img.jbzj.com/file_images/article/202007/202007230833496.png) 

- 测试：调用

  ![img](https://img.jbzj.com/file_images/article/202007/202007230833497.png) 

## 四 singleton和prototype的小结

singleton比prototype消耗性能，在web开发中，推荐使用singleton模式，在app开发中，推荐使用prototype模式。

到此这篇关于Spring中的singleton和prototype的实现的文章就介绍到这了,更多相关Spring singleton和prototype内容请搜索脚本之家以前的文章或继续浏览下面的相关文章希望大家以后多多支持脚本之家！