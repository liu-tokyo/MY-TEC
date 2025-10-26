# 细说SpringBoot的自动装配原理

## 1. 什么是SpringBoot?

对于`Spring`框架，我们接触得比较多的应该是`Spring mvc`、和`Spring`。而`Spring`的核心在于IOC（控制反转对于`Spring`框架来说，就是由`Spring`来负责控制对象的生命周期和对象间的关系）和DI（依赖注入IoC的一个重点是在系统运行中，动态的向某个对象提供它所需要的其他对象。这一点是通过DI（Dependency Injection，依赖注入）来实现的。比如对象A需要操作数据库，以前我们总是要在A中自己编写代码来获得一个Connection对象，有了 `Spring`我们就只需要告诉`Spring`，A中需要一个Connection，至于这个Connection怎么构造，何时构造，A不需要知道）。而这些框架在使用的过程中会需要配置大量的xml，或者需要做很多繁琐的配置。

`Spring Boot`框架是为了能够帮助使用`Spring`框架的开发者快速高效的构建一个基于spirng框架以及`Spring`生态体系的应用解决方案。它是对“约定优于配置”这个理念下的一个最佳实践。因此它是一个服务于框架的框架，服务的范围是简化配置文件。

## 2. 初步认识`Spring Boot`

- 我们可以使用 https://start.spring.io

  ```java
  @SpringBootApplication
  public class SpringBootStudyApplication { 
     
      public static void main(String[] args) { 
     
          SpringApplication.run(SpringBootStudyApplication.class, args);
      }
  }
  ```

- 为了让大家看到效果，我们使用`Spring mvc`来构建一个`web`应用，而`Spring Boot`帮我们简化了非常多的逻辑使得我们非常容易去构建一个`web`项目。

  `Spring Boot`提供了`spring-boot-starter-web`自动装配模块

  ```kotlin
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  ```
  
- 在当前项目下运行`mvn spring-boot:run` 或者直接运行`main`方法就可以启动一个使用了嵌入式`tomcat`服务请求的`web`应用，但是我们没有提供任何服务`web`请求的controller，所以访问任何路径都会返回一个`Spring Boot`默认的错误页面(`whitelabel error page`)

  所以，我们可以创建一个Controller来实现请求

  ```java
  @RestController
  public class HelloController { 
  
      @GetMapping("/say")
      public String sayHello(){ 
     
          return "hello Mic";
      }
  }
  ```

访问http://localhost:8080/say 就可以获得一个请求结果。 这样就完成了一个非常简单的`web`应用。 `Spring Boot`是一个约定优于配置的产物，所以在快速构建`web`应用的背后，其实有很多的约定。

1. 项目结构层面，静态文件和页面模版的存放位置变成了`src/main/resources`对应的子目录下；
2. 自动嵌入`tomcat`作为`web`容器对外提供`http`服务，默认使用`8080`端口监听；
3. 自动装配`springmvc`必要的组件。

## 3. `Spring Boot`四大核心

- `EnableAutoConfiguration` 自动装配
- `Starter` 组件, 开箱即用
- `Actuator` 监控
- `Spring Boot Cli` 为`Spring Cloud` 提供了`Spring Boot` 命令行功能

## 4. `Enable*` 注解的作用

Enable是启用的意思，相当于开启某一个功能

- `EnableScheduling`

- `EnableHystrix`
- `EnableAsync`
- `EnableAutoConfiguration`
- `EnableWebMvc`

仍然是在`spring3.1`版本中，提供了一系列的`@Enable`开头的注解，`Enable`注释应该是在`JavaConfig`框架上更进一步的完善，是的用户在使用`spring`相关的框架时，避免配置大量的代码从而降低使用的难度

- 比如常见的一些Enable注解：`EnableWebMvc`，（这个注解引入了`MVC`框架在`Spring` 应用中需要用到的所有`bean`）；
- 比如说`@EnableScheduling`，开启计划任务的支持；
- 找到`EnableAutoConfiguration`，我们可以看到每一个涉及到`Enable`开头的注解，都会带有一个`@Import`的注解。

## 5. 深入分析`Spring Boot`中的自动装配

在`Spring Boot`中，不得不说的一个点是自动装配，它是`starter`的基础，也是`Spring Boot`的核心， 那什么叫自动装配？或者什么叫装配呢？

回顾一下`Spring Framework`，它最核心的功能是`IOC`和`AOP`， `IoC`容器的主要功能是可以管理对象的生命周期。也就是bean的管理。我们把`Bean`对象托管到`Spring IoC`容器的这个过程称为装配，那什么是自动装配呢？

- 首先大家看下这张图，我们先不解释。等把今天的内容讲完，我们再回头来通过这张图来总结~  
  ![image-20240219234054354](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20240219234054354.png)

自动装配在`SpringBoot`是基于`EnableAutoConfiguration`来实现的。那么在深入分析`EnableAutoConfiguration`之前，我们来看一下传统意义上的装配方式。

### 5.1 简单分析`@Configuration`

`Configuration`这个注解大家应该有用过，它是`JavaConfig`形式的基于`Spring IoC`容器的配置类使用的一种注解。因为`SpringBoot`本质上就是一个`spring`应用，所以通过这个注解来加载`IoC`容器的配置是很正常的。所以在启动类里面标注了`@Configuration`，意味着它其实也是一个`IoC`容器的配置类。

- DemoClass

  ```java
  public class DemoClass { 
  
      public void say(){ 
     
          System.out.println("sya hello ... ");
      }
  }
  ```

- DemoConfiguration

  ```java
  @Configuration
  @Import(UserClass.class)
  public class DemoConfiguration { 
  
      @Bean
      public DemoClass getDemoClass(){ 
     
          return new DemoClass();
      }
  }
  ```

- DemoConfigurationMain

  ```java
  public class DemoConfigurationMain { 
  
      public static void main(String[] args) { 
     
          ApplicationContext ac = new AnnotationConfigApplicationContext(DemoConfiguration.class);
          DemoClass bean = ac.getBean(DemoClass.class);
          bean.say();
      }
  }
  ```

这种形式就是通过注解的方式来实现`IoC`容器，传统意义上的`spring`应用都是基于`xml`形式来配置`bean`的依赖关系。然后通过`spring`容器在启动的时候，把`bean`进行初始化并且，如果`bean`之间存在依赖关系，则分析这些已经在`IoC`容器中的`bean`根据依赖关系进行组装。

直到`Java5`中，引入了`Annotations`这个特性，`Spring`框架也紧随大流并且推出了基于Java代码和`Annotation`元信息的依赖关系绑定描述的方式。也就是`JavaConfig`。

从`spring3`开始，spring就支持了两种bean的配置方式，一种是基于`xml`文件方式、另一种就是`JavaConfig`

**Configuration的本质**

如果大家细心一点就会发现`Configuration`注解的本质是一个`Component`注解，这个注解会被`AnnotationConfigApplicationContext`加载，而`AnnotationConfigApplicationContext`是`ApplicationContext`的一个具体实现，表示根据配置注解启动应用上下文。  
因此我们在`Main`方法中通过`AnnotationConfigApplicationContext`去加载`JavaConfig`后，可以得到`IoC`容器中的`bean`的实例。  
`JavaConfig`的方式在前面的代码中已经演示过了，任何一个标注了`@Configuration`的`Java`类定义都是一个`JavaConfig`配置类。而在这个配置类中，任何标注了`@Bean`的方法，它的返回值都会作为`Bean`定义注册到`Spring`的`IoC`容器，方法名默认成为这个`bean`的`id`。

## 5.2 简单分析`@ComponentScan`

`ComponentScan`这个注解是大家接触得最多的了，相当于`xml`配置文件中的`<context:component-scan>`。 它的主要作用就是扫描指定路径下的标识了需要装配的类，自动装配到`spring`的`IoC`容器中。

- 标识需要装配的类的形式主要是：`@Component`、`@Repository`、`@Service`、`@Controller`这类的注解标识的类。

  ```java
  public static void main(String[] args) { 
     
      ApplicationContext ac = new AnnotationConfigApplicationContext(DemoConfiguration.class);
      String[] beanDefinitionNames = ac.getBeanDefinitionNames();
      System.out.println("********************************");
      for (String beanName:beanDefinitionNames
           ) { 
     
          System.out.println(beanName);
      }
  }
  ```

### 5.3 Import注解

`import` 注解是什么意思呢？ 联想到xml形式下有一个`<import resource/>` 形式的注解，就明白它的作用了。`import`就是把多个分来的容器配置合并在一个配置中。在JavaConfig中所表达的意义是一样的。为了能更好的理解后面讲的`EnableAutoConfiguration`，我们详细的和大家来介绍下`import`注解的使用

#### 5.3.1 直接填class数组

我们先在不同的两个`package`下创建对应的`bean`。

- 比如：  
  ![image-20240219235217361](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20240219235217361.png)

- 那么`DemoConfigurationMain`中执行的代码应该是加载不到`demo2`中的`UserClass`的  
  ![image-20240219235247717](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20240219235247717.png)

- 这时我们可以通过`@import`来直接加载  
  ![image-20240219235314315](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20240219235314315.png)

  或者  
  ![image-20240219235339695](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20240219235339695.png)

#### 5.3.2 `ImportSelector`方式【重点】

第一种方式如果导入的是一个配置类，那么该配置类中的所有的都会加载，如果想要更加的灵活，动态的去加载的话可以通过`Import`接口的第二种使用方式，也就是`ImportSelector`这种方式，我们来看看怎么使用。

- LogService

  ```java
  public class LogService { 
     
  }
  ```

- CacheService

  ```java
  public class CacheService { 
     
  }
  ```

- GpDefineImportSelector

  ```java
  public class GpDefineImportSelector implements ImportSelector { 
     
      /** * AnnotationMetadata 注解元数据 * @param annotationMetadata * @return * 要被IOC容器加载的bean信息 */
      @Override
      public String[] selectImports(AnnotationMetadata annotationMetadata) { 
     
          // 我们可以基于注解元数据信息来动态的返回要加载的bean信息
          annotationMetadata
                  .getAllAnnotationAttributes(EnableDefineService.class.getName(),true)
          .forEach((k,v)->{ 
     
              System.out.println(annotationMetadata.getClassName());
              System.out.println("---> " + k+":" + String.valueOf(v));
          });
  
          return new String[]{ 
     CacheService.class.getName()};
      }
  }
  ```

- EnableDefineService

  ```java
  @Target({ ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Inherited
  @Import(GpDefineImportSelector.class)
  public @interface EnableDefineService { 
  
      String[] packages() default "";
  }
  ```
  
- EnableDemoTest

  ```java
  @EnableDefineService()
  @SpringBootApplication
  public class EnableDemoTest { 
  
      public static void main(String[] args) { 
     
          ApplicationContext ac = new AnnotationConfigApplicationContext(EnableDemoTest.class);//SpringApplication.run(EnableDemoTest.class,args);
          System.out.println(ac.getBean(CacheService.class));
          System.out.println(ac.getBean(LogService.class));
      }
  }
  ```

#### 5.3.3 `ImportBeanDefinitionRegistrar`方式

- 这种方式和第二种方式很相似，同样的要实现 `ImportBeanDefinitionRegistrar` 接口

  ```java
  public class GpImportDefinitionRegister implements ImportBeanDefinitionRegistrar { 
     
      @Override
      public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) { 
     
          // 指定bean的定义信息
          RootBeanDefinition beanDefinition = new RootBeanDefinition(CacheService.class);
          RootBeanDefinition beanDefinition1 = new RootBeanDefinition(LogService.class);
          // 注册一个bean
          beanDefinitionRegistry.registerBeanDefinition("cacheService1111",beanDefinition);
          beanDefinitionRegistry.registerBeanDefinition("cacheService2222",beanDefinition1);
  
      }
  }
  ```

  ![image-20240219235746878](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20240219235746878.png)
  
  ![image-20240219235820315](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20240219235820315.png)

### 5.4 深入分析`EnableAutoConfiguration`原理

了解了`ImportSelector`和`ImportBeanDefinitionRegistrar`后，对于`EnableAutoConfiguration`的理解就容易一些了

- 它会通过`import`导入第三方提供的bean的配置类：AutoConfigurationImportSelector  
  ![image-20240219235932487](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20240219235932487.png)
  
- AutoConfigurationImportSelector

  ```java
  public String[] selectImports(AnnotationMetadata annotationMetadata) { 
  	
  	if (!this.isEnabled(annotationMetadata)) { 
  		return NO_IMPORTS;
  	} else { 
  		try { 
  			// 加载META-INF/spring-autoconfigure-metadata.properties 下的元数据信息
  			AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
  			AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
  			// 获取候选加载的配置信息 META-INF/spring.factories
  			List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
  			// 去掉重复的配置信息
  			configurations = this.removeDuplicates(configurations);
  			// 排序
  			configurations = this.sort(configurations, autoConfigurationMetadata);
  			// 获取 注解中配置的 exclusion 信息
  			Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
  			// 检查
  			this.checkExcludedClasses(configurations, exclusions);
  			// 移除需要排除的信息
  			configurations.removeAll(exclusions);
  			// 过滤，检查候选配置类上的注解@ConditionalOnClass，如果要求的类不存在，则这个候选类会被过滤不被加载
  			configurations = this.filter(configurations, autoConfigurationMetadata);
  			// 广播事件
  			this.fireAutoConfigurationImportEvents(configurations, exclusions);
  			// 返回要被加载的类数组
  			return (String[])configurations.toArray(new String[configurations.size()]);
  		} catch (IOException var6) { 
  			throw new IllegalStateException(var6);
  		}
  	}
  }
  ```

本质上来说，其实`EnableAutoConfiguration`会帮助`Spring Boot`应用把所有符合`@Configuration`配置都加载到当前`SpringBoot`创建的`IoC`容器，而这里面借助了`Spring`框架提供的一个工具类`SpringFactoriesLoader`的支持。以及用到了`Spring`提供的条件注解`@Conditional`，选择性的针对需要加载的`bean`进行条件过滤。

### 5.5 SpringFactoriesLoader

为了给大家补一下基础，我在这里简单分析一下`SpringFactoriesLoader`这个工具类的使用。它其实和`java`中的`SPI`机制的原理是一样的，不过它比`SPI`更好的点在于不会一次性加载所有的类，而是根据key进行加载。

首先，`SpringFactoriesLoader`的作用是从`classpath/META-INF/spring.factories`文件中，根据`key`来加载对应的类到`spring IoC`容器中。接下来带大家实践一下

通过mybatis整合来看

- 引入mybatis的依赖

  ```kotlin
  <dependency>
  	<groupId>org.mybatis.spring.boot</groupId>
  	<artifactId>mybatis-spring-boot-starter</artifactId>
  	<version>2.1.2</version>
  </dependency>
  ```

  ![image-20240220000336471](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20240220000336471.png)

  查看 MybatisAutoConfiguration 里面的源码，发现在其中加载了SqlSessionFactory等信息。

**通过实际案例来实现**

- 创建一个新的maven项目，引入相关的依赖

  ```kotlin
  <dependencies>
      <dependency>
          <groupId>org.springframework</groupId>
          <artifactId>spring-context</artifactId>
          <version>5.2.6.RELEASE</version>
      </dependency>
  </dependencies>
  ```

- 创建一个bean类

  ```java
  public class GpCoreService { 
  
      public void crudService(){ 
     
          System.out.println("gupao core service run ...");
      }
  }
  ```

- 以及对应的配置类

  ```java
  @Configuration
  public class GpConfiguration { 
  
      @Bean
      public GpCoreService gpCoreService(){ 
     
          return new GpCoreService();
      }
  
  }
  ```

- 然后打包

  ![image-20240220000534922](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20240220000534922.png)

- 在另一个项目中导入该jar包

  ```kotlin
  <dependency>
      <groupId>org.gupao.edu</groupId>
      <artifactId>Gp-Core</artifactId>
      <version>1.0-SNAPSHOT</version>
  </dependency>
  ```

- 通过下面代码获取依赖包中的属性

  运行结果会报错，原因是GuPaoCore并没有被Spring的IoC容器所加载，也就是没有被`EnableAutoConfiguration`导入

  ```java
  public static void main(String[] args) { 
     
      ConfigurableApplicationContext run = SpringApplication.run(DemoApplication.class, args);
      String[] beanDefinitionNames = run.getBeanDefinitionNames();
      System.out.println(run.getBean(GpCoreService.class));
  }
  ```

**解决方案**

- 在GuPao-Core项目resources下新建文件夹META-INF，在文件夹下面新建spring.factories文件，文件中配置，key为自定配置类`EnableAutoConfiguration`的全路径，value是配置类的全路径

  ```
  org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.gupao.edu.GpConfiguration
  ```

重新打包，重新运行`SpringBootStudyApplication`这个类。  
可以发现，我们编写的那个类，就被加载进来了。

### 5.6 深入理解条件过滤

在分析`AutoConfigurationImportSelector`的源码时，会先扫描`spring-autoconfiguration-metadata.properties`文件，最后在扫描`spring.factories`对应的类时，会结合前面的元数据进行过滤，为什么要过滤呢？ 原因是很多的`@Configuration`其实是依托于其他的框架来加载的，如果当前的`classpath`环境下没有相关联的依赖，则意味着这些类没必要进行加载，所以，通过这种条件过滤可以有效的减少`@configuration`类的数量从而降低`SpringBoot`的启动时间。

- 修改`Gupao-Core`

  在META-INF/增加配置文件，`spring-autoconfigure-metadata.properties`。

  ```
  com.gupao.edu.GpConfiguration.ConditionalOnClass=com.gupao.edu.service.GpTestService
  ```

  ![image-20240220000849953](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20240220000849953.png)

> 格式：自动配置的类全名.条件=值

上面这段代码的意思就是，如果当前的`classpath`下存在`TestClass`，则会对`GuPaoConfig`这个`Configuration`进行加载

演示过程(spring-boot)

> 1. 沿用前面`spring-boot`工程的测试案例，直接运行main方法，发现原本能够被加载的`GuPaoCore`，发现在`IoC`容器中找不到了。
> 2. 在当前工程中指定的包`com.gupaoedu`下创建一个`TestClass`以后，再运行上面这段代码，程序能够正常执行。
