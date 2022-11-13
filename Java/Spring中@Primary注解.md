#  Spring中@Primary注解

> https://www.cnblogs.com/fnlingnzb-learner/p/16803858.html

## 1.概述

讨论Spring的@Primary注解，该注解是框架在3.0版中引入的。

其作用与功能，当有多个相同类型的bean时，使用@Primary来赋予bean更高的优先级。

## 2.为什么需要@Primary？

在某些情况下，需要注册多个相同类型的bean。

- 在此示例中，有Employee类型的zhangSanEmployee()和liSiEmployee()Bean：

  ```java
  @Configuration
  public class PrimaryConfig {
  	@Bean
  	public Employee zhangSanEmployee() {
  		return new Employee("张三");
  	}
  
  	@Bean
  	public Employee liSiEmployee() {
  		return new Employee("李四");
  	}
  }
  ```

  如果尝试运行应用程序，与@Autowired一起应用于注入。Spring会抛出NoUniqueBeanDefinitionException。

- 要访问相同类型的bean，常使用@Qualifier(“beanName”)注解，通过别名控制访问相同类型。

  ```java
  @Configuration
  public class PrimaryConfig {
  
     @Bean
     @Qualifier("zhangSanEmployee")
     public Employee zhangSanEmployee() {
  	   return new Employee("张三");
     }
  
     @Bean
     @Qualifier("liSiEmployee")
     public Employee liSiEmployee() {
  	   return new Employee("李四");
     }
  }
  ```

  注入

  ```java
  @Resource
  private Employee zhangSanEmployee;
  
  @Resource
  private Employee liSiEmployee;
  ```

## 3.将@Primary和@Bean一起使用

- 看一下配置类：

  ```java
  @Configuration
  public class PrimaryConfig {
  
  	@Bean
  	public Employee zhangSanEmployee() {
  		return new Employee("张三");
  	}
  
  	@Bean
  	@Primary
  	public Employee liSiEmployee() {
  		return new Employee("李四");
  	}
  }
  ```

  用@Primary标记liSiEmployee()bean。 Spring将优先于zhangSanEmployee()注入liSiEmployee()bean。

  ```java
  @Test
  public void test1() {
  	AnnotationConfigApplicationContext context
  		= new AnnotationConfigApplicationContext(PrimaryConfig.class);
  
  	Employee employee = context.getBean(Employee.class);
  	System.out.println(employee);//Employee(name=李四)
  
  }
  ```

## 4.将@Primary与@Component一起使用

- 可以直接在bean上使用@Primary。

  ```java
  public interface Manager {
  	String getManagerName();
  }
  ```

- 有一个Manager接口和两个子类bean

  ```java
  @Component
  public class DepartmentManager implements Manager {
  	@Override
  	public String getManagerName() {
  		return "Department manager";
  	}
  }
  ```

  ```java
  @Component
  @Primary
  public class GeneralManager implements Manager {
  	@Override
  	public String getManagerName() {
  		return "General manager";
  	}
  }
  ```

- 都覆盖Manager接口的getManagerName()。 另外，请注意，用@Primary标记了GeneralManager bean。

  ```java
  @Service
  public class ManagerService {
  
  	@Autowired
  	private Manager manager;
  
  	public Manager getManager() {
  		return manager;
  	}
  }
  ```

- 测试

  ```java
  @RunWith(SpringRunner.class)
  @SpringBootTest
  public class PrimaryTest {
  	@Resource
  	private ApplicationContext context;
  	@Test
  	public void test2() {
  		ManagerService service = context.getBean(ManagerService.class);
  		Manager manager = service.getManager();
  		System.out.println(manager.getManagerName());//General manager
  	}
  }
  ```

  