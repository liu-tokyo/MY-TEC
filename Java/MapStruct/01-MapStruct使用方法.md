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

- 可以使用 `Mappers#getMapper` 创建 `Mapper` 类的实例

  ```java
  Mappers.getMapper(FooMapper.class);
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

### 3.1 映射常数

- 使用@Mapping的`constant`属性

  ```java
  @Mapper
  public interface FooMapper {
  	@Mapping(target="name", constant = "张三")
  	Bar fooToBar(Foo foo);
  }
  ```

### 3.2 设置默认值

- 默认值可以在`@Mapping`的`defaultValue`属性中设置

  当要映射的值为空时适用。

  ```java
  @Mapper
  public interface FooMapper {
  	@Mapping(target="name", defaultValue = "张三")
  	Bar fooToBar(Foo foo);
  }
  ```

### 3.3 执行任意Java代码

- 任何`Java`代码都可以在`@Mapping`的`Expression`属性中指定进行映射处理。
  用 `java()` 包围 `Java` 代码。

  ```java
  @Mapper
  public interface FooMapper {
  	@Mapping(target="now", expression = "java(java.time.LocalDate.now())")
  	Bar fooToBar(Foo foo);
  }
  ```

  如果使用@Mapper的imports属性，则不必先写包名。

  ```java
  @Mapper(imports = LocalDate.class)
  public interface FooMapper {
  	@Mapping(target="now", expression = "java(LocalDate.now())")
  	Bar fooToBar(Foo foo);
  }
  ```

### 3.4 格式化数字

- 在`@Mapping`的`numberFormat`属性中指定数字格式。

  ```java
  @Mapper
  public interface FooMapper {
  	@Mapping(target="num", numberFormat = "000") // 零填充
  	Bar fooToBar(Foo foo);
  }
  ```

### 3.5 格式化日期

- 在`@Mapping`的`dateFormat`属性中指定日期格式。

  ```java
  @Mapper
  public interface FooMapper {
  	@Mapping(target="date", dateFormat = "yyyy/MM/dd")
  	Bar fooToBar(Foo foo);
  }
  ```

### 3.6 不同枚举之间的映射

- 使用@ValueMapping进行枚举映射。

  可以映射不同的枚举。

  ```java
  @Mapper
  public interface FooMapper {
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

### 3.7 使用 `@Qualifier`

当您想要添加特殊行为时可以使用此功能。例如，添加处理以转换为大写字母。

#### 创建注释

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

#### 行为定义

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

#### 创建映射

- 在@Mapper的uses属性中指定定义行为的类。

  在@Mapping的qualifiedBy属性中指定两个注释类，并将行为添加到映射中。

  ```java
  @Mapper(uses = StringConverter.class)
  public interface FooMapper {
  	@Mapping(target="name", qualifiedBy = { Converter.class, ToUpper.class })
  	Bar fooToBar(Foo foo);
  }
  ```

  

### 3.8 使用 `@Context`

通过使用`@Context`，可以从外部更改映射行为。

#### 创建映射

- 添加一个参数，并将 @Context 添加到映射方法参数中。

  ```java
  @Mapper
  public interface FooMapper {
  	Bar fooToBar(Foo foo, @Context Locale locale);
  }
  ```

#### 添加自定义方法

- 使用给定@Context 的参数定义自定义方法。

  在下面的示例中，LocalDate 类型的字段被格式化并映射到指定的 Local。

  ```java
  @Mapper
  public interface FooMapper {
  	Bar fooToBar(Foo foo, @Context Locale locale);
  
  	default String format(LocalDate date, @Context Locale locale) {
  		// 格式化以适应本地等。
  	}
  }
  ```

### 3.9 使用装饰器 `Decorator`

通过使用`Decorator`，您可以覆盖映射过程并添加特殊处理。

- 首先，创建一个Mapper类。

  ```java
  @Mapper
  public interface FooMapper {
  	Bar fooToBar(Foo foo);
  }
  ```

#### 创建装饰器类

- Decorator类被创建为抽象类，是Mapper的子类型，可以自定义。

  ```java
  public abstract class FooMapperDecorator implements FooMapper {
  
  	private final FooMapper delegate;
  
  	public FooMapperDecorator(FooMapper delegate) {
  		this.delegate = delegate;
  	}
  
  	// 覆盖您想要自定义的方法
  	@Override
  	public Bar fooToBar(Foo foo) {
  		Bar bar = delegate.fooToBar(foo);
  		// 添加特殊处理
  		return bar;
  	}
  }
  ```

#### 应用装饰器

- 将`Decorator`类应用到`Mapper`类。

  要应用，请使用`@DecolatedWith`。

  ```java
  @Mapper
  @DecoratedWith(FooMapperDecorator.class)
  public interface FooMapper {
  	Bar fooToBar(Foo foo);
  }
  ```

### 3.10 映射前后处理

通过使用`@BeforeMapping`和`@AfterMapping`，可以在映射过程`之前`和`之后`执行自定义的处理。

- 映射之前的自定义处理

  ```java
  @Mapper
  public abstract class FooMapper {
  	// 映射之前要执行的方法
  	@BeforeMapping
  	protected void before(Foo foo) {
  		// omit..
  	}
  
  	// 映射后要执行的方法
  	@AfterMapping
  	protected void after(@MappingTarget Bar bar) {
  		// omit..
  	}
  
  	public abstract Bar fooToBar(Foo foo);
  }
  ```

- 自动生成的映射器

  ```java
  public class FooMapperImpl extends FooMapper {
  
  	@Override
  	public Bar fooToBar(Foo foo) {
  		// 映射之前运行
  		before( foo );
  
  		if ( foo == null ) {
  			return null;
  		}
  
  		// 测绘流程
  
  		// 映射后运行
  		after( bar );
  
  		return bar;
  	}
  }
  ```

  

## 4. 映射重用

- 通过使用`@InheritConfiguration`，定义可以重用的映射。

  ```java
  @Mapper(config = FooConfig.class)
  public interface FooMapper {
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

  