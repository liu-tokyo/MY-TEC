# MapStruct的使用方法

## 1. 环境设置

## 2. 创建办法

### 2.1 创建映射类

- Mapper类是通过在接口类 `interface` 或抽象类 `abstract` 上添加`@Mapper`来创建的。

  ```java
  @Mapper
  public interface FooMapper {
  	// omit..
  }
  
  @Mapper
  public abstract FooMapper {
  	// omit..
  }
  ```

### 2.2 映射类实例

- 可以使用 `Mappers#getMapper` 创建 `Mapper` 类的实例：

  ```java
  Mappers.getMapper(FooMapper.class);
  ```
  
- 在映射器的接口中实现实例：

  ```java
  @Mapper
  public interface CustomerMapper {
  	CustomerMapper INSTANCE = Mappers.getMapper(CustomerMapper.class);
  	CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

  调用方法：

  ```java
  CustomerDto dto = CustomerMapper.INSTANCE.customerToCustomerDto(customer);
  ```

- 使用抽象类时，定义如下：

  ```java
  @Mapper
  public abstract class CustomerMapper {
  	public static final CustomerMapper INSTANCE = Mappers.getMapper(CustomerMapper.class);
  	CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

- 如果是在CDI环境中使用：

  您可以通过指定 `ComponentModel.CDI` 将其用作 `@ApplicationScoped` 的 `CDI bean`。

  ```java
  @Mapper(componentModel = MappingConstants.ComponentModel.CDI)
  public interface CustomerMapper {
  	CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

  外部调用：

  ```java
  @Inject
  private CustomerMapper mapper;
  ```

  

### 2.3 指定组件模型

您可以通过指定@Mapper的componentModel属性来更改实例创建方法。

- 使用 Spring 框架启用 DI

  ```java
  // spring指定
  @Mapper(componentModel = "spring")
  public interface FooMapper {
  	// omit..
  }
  ```

  如果你查看自动生成的Mapper类，你会发现@Component已经被添加了。

  ```java
  import org.springframework.stereotype.Component;
  
  @Component
  public class FooMapperImpl implements FooMapper {
  	// omit..
  }
  ```

- 其他方法参见官方文档

  http://mapstruct.org/documentation/stable/reference/html/#retrieving-mapper

## 3. 自定义映射

MapStruct 提供了多种自定义映射过程的方法。

使用以下例子：

- `Customer.java`

  ```java
  public class Customer {
      private Long id;
      private String name;
  
      public Customer(Long id, String name) {
          this.id = id;
          this.name = name;
      }
  
      public Long getId() {
          return id;
      }
  
      public String getName() {
          return name;
      }
  
      @Override
      public String toString() {
          return String.format("Customer{id=%d, name=%s}", id, name);
      }
  }
  ```

- `CustomerDto:java` (简单的使用公共字段来定义)

  ```java
  public class CustomerDto {
      public Long id;
      public String name;
  
      @Override
      public String toString() {
          return String.format("CustomerDto{id=%d, name=%s}", id, name);
      }
  }
  ```

- `CustomerDto:java` （使用 `final` 来定义字段）

  ```java
  public class CustomerDto {
      public final Long id;
      public final String name;
  
      public CustomerDto(Long id, String name) {
          this.id = id;
          this.name = name;
      }
  
      @Override
      public String toString() {
          return String.format("CustomerDto{id=%d, name=%s}", id, name);
      }
  }
  ```

  

### 3.1 映射字段名

- 通过定义如下所示的映射，AccountDto 的内容将自动映射到 Customer 中的同名属性：

  ```java
  @Mapper
  public interface CustomerMapper {
      @Mapping(target = "customerName", source = "name")
      CustomerDto customerToCustomerDto(Customer customer);
      // 数组、列表之类的映射方法
      List<CustomerDto> customersToCustomerDtos(List<Customer> customers);
      CustomerDto[] customersToCustomerDtos(Customer[] customers);
  }
  ```

  可以在映射定义中指定默认值（defaultValue）和常量值（constant）

  ```java
  @Mapping(target = "stringProperty", source = "stringProp", defaultValue = "undefined")
  @Mapping(target = "longProperty", source = "longProp", defaultValue = "-1")
  @Mapping(target = "stringConstant", constant = "Constant Value")
  ```

### 3.2 自定义映射 `default`

- 编写自己的映射逻辑作为默认方法：

  ```java
  @Mapper
  public interface CustomerMapper {
      default CustomerDto customerToCustomerDto(Customer customer) {
          // ...
      }
  }
  ```

  

### 3.3 映射常数 `constant`

- 使用@Mapping的`constant`属性

  ```java
  @Mapper
  public interface CustomerMapper {
  	@Mapping(target="name", constant = "张三")
  	CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

### 3.4 设置默认值 `defaultValue`

- 默认值可以在`@Mapping`的`defaultValue`属性中设置

  当要映射的值为`空`时适用。

  ```java
  @Mapper
  public interface CustomerMapper {
  	@Mapping(target="name", defaultValue = "张三")
  	CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

### 3.5 执行任意Java代码 `expression`

- 任何`Java`代码都可以在`@Mapping`的`Expression`属性中指定进行映射处理。
  用 `java()` 包围 `Java` 代码。

  ```java
  @Mapper
  public interface CustomerMapper {
  	@Mapping(target="now", expression = "java(java.time.LocalDate.now())")
  	CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

  如果使用`@Mapper`的`imports`属性，则不必先写包名。

  ```java
  @Mapper(imports = LocalDate.class)
  public interface CustomerMapper {
  	@Mapping(target="now", expression = "java(LocalDate.now())")
  	CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

### 3.6 格式化数字 `numberFormat`

- 在`@Mapping`的`numberFormat`属性中指定数字格式。

  ```java
  @Mapper
  public interface CustomerMapper {
  	@Mapping(target="num", numberFormat = "000") // 零填充
  	CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

### 3.7 格式化日期 `dateFormat`

- 在`@Mapping`的`dateFormat`属性中指定日期格式。

  ```java
  @Mapper
  public interface CustomerMapper {
  	@Mapping(target="date", dateFormat = "yyyy/MM/dd")
  	CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

### 3.8 不同枚举之间的映射 `ValueMapping`

- 使用@ValueMapping进行枚举映射。

  可以映射不同的枚举。

  ```java
  @Mapper
  public interface CustomerMapper {
  	@ValueMapping(source = "SMALL", target = "SHORT")
  　　　　　　　　@ValueMapping(source = "MEDIUM", target = "TALL")
  　　　　　　　　@ValueMapping(source = "LARGE", target = "GRANDE")
  	BarEnum fooToBar(FooEnum foo);
  }
  ```

  如果没有映射目标，则通过在 `taget` 属性中指定 `MappingConstants.NULL` 来设置 `null`。

  ```java
  @ValueMapping(source = "VENTI", target = MappingConstants.NULL)
  ```

  如果要指定默认值，请在`source`属性中指定 `MappingConstants.ANY_REMAINING`。

  ```java
  @ValueMapping(source = MappingConstants.ANY_REMAINING, target = "LARGE")
  ```

### 3.9 使用 `@Qualifier`

想要添加特殊行为时可以使用此功能。例如，添加处理以转换为大写字母。

#### (a) 创建注释

创建两个组合 @Qualifier 的注释：一个用于类级别，一个用于方法级别。

- 类级别

  ```java
  @Qualifier
  @Retention(CLASS)
  @Target(TYPE)
  public @interface Converter {
  }
  ```

- 方法级别

  ```java
  @Qualifier
  @Retention(CLASS)
  @Target(METHOD)
  public @interface ToUpper {
  }
  ```

#### (b) 行为定义

- 创建一个定义行为的类，添加上面创建的注释。

  ```java
  @Converter
  public class StringConverter {
  	@ToUpper
  	public String upperCase(String string) {
  		return (string == null) ? null : string.toUpperCase();
  	}
  }
  ```

#### (c) 创建映射

- 在@Mapper的uses属性中指定定义行为的类。

  在@Mapping的qualifiedBy属性中指定两个注释类，并将行为添加到映射中。

  ```java
  @Mapper(uses = StringConverter.class)
  public interface CustomerMapper {
  	@Mapping(target="name", qualifiedBy = { Converter.class, ToUpper.class })
  	CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

  

### 3.10 使用 `@Context`

通过使用`@Context`，可以从外部更改映射行为。

#### (a) 创建映射

- 添加一个参数，并将 @Context 添加到映射方法参数中。

  ```java
  @Mapper
  public interface CustomerMapper {
  	CustomerDto customerToCustomerDto(Customer customer, @Context Locale locale);
  }
  ```

#### (b) 添加自定义方法

- 使用给定@Context 的参数定义自定义方法。

  在下面的示例中，`LocalDate` 类型的字段被格式化并映射到指定的 Local。

  ```java
  @Mapper
  public interface CustomerMapper {
  	CustomerDto customerToCustomerDto(Customer customer, @Context Locale locale);
  
  	default String format(LocalDate date, @Context Locale locale) {
  		// 格式化以适应本地等。
  	}
  }
  ```

### 3.11 使用装饰器 `Decorator`

通过使用`Decorator`，您可以覆盖映射过程并添加特殊处理。

- 首先，创建一个Mapper类。

  ```java
  @Mapper
  public interface CustomerMapper {
  	CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

#### (a) 创建装饰器类

- `Decorator`类被创建为抽象类，是`Mapper`的子类型，可以自定义。

  ```java
  public abstract class CustomerMapperDecorator implements CustomerMapper {
  
  	private final CustomerMapper delegate;
  
  	public CustomerMapperDecorator(CustomerMapper delegate) {
  		this.delegate = delegate;
  	}
  
  	// 覆盖您想要自定义的方法
  	@Override
  	public Bar fooToBar(Customer customer) {
  		CustomerDto target = delegate.customerToCustomerDto(customer);
  		// 添加特殊处理
  		return target;
  	}
  }
  ```

#### (b) 应用装饰器

- 将`Decorator`类应用到`Mapper`类。

  要应用，请使用`@DecolatedWith`。

  ```java
  @Mapper
  @DecoratedWith(CustomerMapperDecorator.class)
  public interface CustomerMapper {
  	CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

### 3.12 映射前后处理

通过使用`@BeforeMapping`和`@AfterMapping`，可以在映射过程`之前`和`之后`执行自定义的处理。

- 映射之前的自定义处理

  ```java
  @Mapper
  public abstract class CustomerMapper {
  	// 映射之前要执行的方法
  	@BeforeMapping
  	protected void before(Customer customer) {
  		// omit..
  	}
  
  	// 映射后要执行的方法
  	@AfterMapping
  	protected void after(@MappingTarget CustomerDto customerDto) {
  		// omit..
  	}
  
  	public abstract CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

- 自动生成的映射器

  ```java
  public class CustomerMapperImpl extends CustomerMapper {
  
  	@Override
  	public CustomerDto customerToCustomerDto(Customer customer) {
  		// 映射之前运行
  		before( customer );
  
  		if ( customer == null ) {
  			return null;
  		}
  
  		// 测绘流程
  
  		// 映射后运行
  		after( customerDto );
  
  		return customerDto;
  	}
  }
  ```


### 3.13 忽略指定字段 `ignore`

- 一个常见的示例是在更新 Customer 实体时使用下面的映射器来更新除 id 之外的任何内容。

  ```java
  @Mapper
  public interface CustomerMapper {
  	@Mapping(target = "id", ignore = true)
  	CustomerDto customerToCustomerDto(Customer customer);
  }
  ```

### 3.14 排除更新 `null` 值

- 如果您想要排除更新 `null` 值，请指定 `nullValuePropertyMappingStrategy` 

  ```java
  @Mapper(nullValuePropertyMappingStrategy = IGNORE)
  public interface CustomerPartialUpdateMapper {
      void mapPartialUpdate(Customer input, @MappingTarget Customer target);
  }
  ```

### 3.15 使用 `@MappingTarget`

- 通过指定 `@MappingTarget` 更新对象：

  ```java
  @Mapper
  public interface CustomerMapper {
  	void updateCustomerFromDto(CustomerDto dto, @MappingTarget Customer customer);
  }
  ```

- 还可以在返回值中指定要更新的类：

  ```java
  @Mapper
  public interface CustomerMapper {
  	Customer updateCustomerFromDto(CustomerDto dto, @MappingTarget Customer customer);
  }
  ```


### 3.16 嵌套映射

考虑一个结构，其中 `CustomerDto` 具有 `AccountDto`。

- `AccountDto.java`

  ```java
  public class AccountDto {
  	public final String name;
  	
  	public AccountDto(String name) {
  		this.name = name;
  	}
  }
  ```

- `CustomerDto.java`

  ```java
  public class CustomerDto {
  	public final Long id;
  	public final AccountDto account;
  	
  	public CustomerDto(Long id, AccountDto account) {
  		this.id = id;
  		this.account = account;
  	}
  }
  ```

可以将嵌套对象的映射定义简化为 `target = "."`。

- 通过定义如下所示的映射，`AccountDto`的内容将自动映射到`Customer`中的同名属性：

  ```java
  @Mapper
  public interface CustomerMapper {
  	@Mapping( target = ".", source = "account" )
  	Customer customerDtoToCustomer(CustomerDto customerDto);
  }
  ```

### 3.17 多对一映射

- MapStruct 可以将几种类型的对象映射为另外一种类型，比如：将多个 DO 对象转换为 DTO

  ```java
  @Mappings({
  			 @Mapping(source = "userDO.birthday", target = "userBirthday",dateFormat = "yyyy-MM-dd HH:mm:ss"),
  			@Mapping(source = "userDO.time", target = "userTime",dateFormat = "yyyy-MM-dd"),
  			@Mapping(source = "userDO.email", target = "userEmail"),
  			@Mapping(source = "cardDO.userCardName", target = "cardName")
  	}
  	)
  	UserDTO toUserDTO(UserDO userDO, CardDO cardDO);
  ```

- 测试

  ```java
  @Test
  	void test4(){
  		UserDO userDo = new UserDO();
  		userDo.setId(8888L);
  		userDo.setUserName("gongjie");
  		userDo.setBirthday(new Date());
  		userDo.setTime("2021-05-09");
  		userDo.setEmail("99@163.com");
  
  		CardDO cardDO = new CardDO();
  		cardDO.setUserCardId(999L);
  		cardDO.setUserCardName("ZSYH");
  
  		UserDTO userDTO = MapStructConverter.INSTANCE.toUserDTO(userDo,cardDO);
  		System.out.println("userDTO = "+ userDTO.toString());
  	}
  ```

  > 输出结果：
  > userDTO = UserDTO(id=8888, userName=gongjie, userBirthday=2021-05-09 22:12:04, userTime=Sun May 09 00:00:00 CST 2021, userEmail=99@163.com, cardName=ZSYH)

### 3.18 自定义类型转换 `uses`

- MapStructConverter进行修改，@Mapper注解增加属性 uses，导入目标类。并增加方法：testCustom

  ```java
  @Mapper(imports = {JoinUtil.class},uses = {JoinUtil.class})
  
  
  @Mappings({
  			@Mapping(source = "id",target = "id",ignore = true),
  			@Mapping(source = "birthday", target = "userBirthday",dateFormat = "yyyy-MM-dd HH:mm:ss"),
  	 	   @Mapping(source = "time", target = "userTime",dateFormat = "yyyy-MM-dd"),
  			@Mapping( target = "userEmail"),
  			@Mapping(source = "card.userCardName", target = "cardName")
  	}
  	)
  	UserDTO testCustom(UserDO userDO);
  ```

### 3.19 @Named注解

- MapStructConverter 内增加一个不带 public 的JoinUtil3类，在要被执行的方法上加注解 @Named并指定名称。

  ```java
  class JoinUtil3{
  
  	@Named("join123")
  	public static String join(String oldStr){
  		return oldStr + "====expression====";
  	}
  	
  	@Named("join1234")
  	public static String join1(String oldStr){
  		return oldStr + "====另外一种拼接方式====";
  	}
  }
  ```

- MapStructConverter 进行修改，uses属性指向 JoinUtil3，再增加一个方法：testNamed，在userEmail映射关系上新增qualifiedByName属性，该属性的值要与 @Named的值保持一致，根据该值来判断需要执行那个方法。

  ```java
  @Mapper(imports = {JoinUtil.class},uses = {JoinUtil3.class})
  
    @Mappings({
  			@Mapping(source = "id",target = "id",ignore = true),
  			@Mapping(source = "birthday", target = "userBirthday",dateFormat = "yyyy-MM-dd HH:mm:ss"),
  			@Mapping(source = "time", target = "userTime",dateFormat = "yyyy-MM-dd"),
  	 	   @Mapping(source="email",target = "userEmail",qualifiedByName = "join123"),
  			@Mapping(source = "card.userCardName", target = "cardName")
  	}
  	)
  	UserDTO testNamed(UserDO userDO);
  ```

- 测试代码

  ```java
  	@Test
  	void test9(){
  		UserDO userDo = new UserDO();
  		userDo.setId(8888L);
  		userDo.setUserName("gongjie");
  		userDo.setBirthday(new Date());
  		userDo.setTime("2021-05-09");
  		userDo.setEmail("99@163.com");
  
  		System.out.println("userDo = "+ userDo.toString());
  		UserDTO userDTO = MapStructConverter.INSTANCE.testNamed(userDo);
  		System.out.println("userDTO = "+ userDTO.toString());
  	}
  ```

  > 结果：
  > userDo = UserDO(id=8888, userName=gongjie, birthday=Mon May 10 22:41:43 CST 2021, time=2021-05-09, email=99@163.com, card=null)
  > 	
  > userDTO = UserDTO(id=null, userName=gongjie, userBirthday=2021-05-10 22:41:43, userTime=Sun May 09 00:00:00 CST 2021, userEmail=99@163.com====expression====, cardName=null)

## 4. 映射重用

- 通过使用`@InheritConfiguration`，定义可以重用的映射。

  ```java
  @Mapper(config = FooConfig.class)
  public interface CustomerMapper {
  	@Mapping(...)
  	Bar fooToBar(Foo foo);
  
  	@InheritConfiguration
  	Bar fuga(Foo foo);
  }
  ```

- 通过使用`@InheritInverseConfiguration`，可以被重用为反向映射。

  ```java
  @Mapper(config = FooConfig.class)
  public interface FooMapper {
  	@Mapping(...)
  	Bar fooToBar(Foo foo);
  
  	@InheritInverseConfiguration
  	Foo barToFoo(Bar bar);
  }
  ```

  

## 5. 配置类

- 使用`@MappingConfig` 创建配置类。  
  在配置类中，您可以指定组件模型并进行设置，例如当存在未映射的项时是否生成警告或错误。  
  有关详细信息，请参阅 [Javadoc](http://mapstruct.org/documentation/stable/api/org/mapstruct/MapperConfig.html)。

  ```java
  @MapperConfig(unmappedTargetPolicy = ReportingPolicy.IGNORE
  	 , mappingInheritanceStrategy = MappingInheritanceStrategy.AUTO_INHERIT_ALL_FROM_CONFIG)
  public interface FooConfig {
  }
  ```

- 将创建的类设置为`Mapper`类。  
  在`@Mapper`的`config`属性中指定创建的配置类。  

  ```java
  @Mapper(config = FooConfig.class)
  public interface FooMapper {
  	// omit..
  }
  ```


## 6. 注册到Spring

`@Mapper`注解中有个属性为 `componentModel`，`componentModel`的值有四种：

- 
  default: 默认值，不使用组件类型, 通过Mappers.getMapper(Class)方式获取实例对象


- 
  cdi: 生成的映射是一个应用程序范围的 cdi 的bean，可以通过@Inject进行检索


- 
  spring: 生成的实现类上面会自动添加一个@Component注解，可以通过 Spring 的 @Autowired方式进行注入


- 
  jsr330: 生成的实现类上会添加@javax.inject.Named注解，可以通过 @Inject 注解获取。

## Z. 参照资料

1. https://qiita.com/kentama/items/7c2693c0b4311c64ea27

2. 关于使用 MapStruct 映射模型和 DTO

   https://qiita.com/yu-F/items/351988becbaf00cb5ae5