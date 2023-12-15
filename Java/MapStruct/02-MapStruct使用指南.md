# MapStruct使用指南

> https://juejin.cn/post/6956190395319451679

# 介绍

随着微服务和分布式应用程序迅速占领开发领域，数据完整性和安全性比以往任何时候都更加重要。在这些松散耦合的系统之间，安全的通信渠道和有限的数据传输是最重要的。大多数时候，终端用户或服务不需要访问模型中的全部数据，而只需要访问某些特定的部分。

数据传输对象(Data Transfer Objects, DTO)经常被用于这些应用中。DTO只是持有另一个对象中被请求的信息的对象。通常情况下，这些信息是有限的一部分。例如，在持久化层定义的实体和发往客户端的DTO之间经常会出现相互之间的转换。由于DTO是原始对象的反映，因此这些类之间的映射器在转换过程中扮演着关键角色。

这就是MapStruct解决的问题：手动创建bean映射器非常耗时。 但是该库可以自动生成Bean映射器类。

在本文中，我们将深入研究[MapStruct](https://link.juejin.cn?target=https%3A%2F%2Fmapstruct.org%2F)。

## MapStruct

MapStruct是一个开源的基于Java的代码生成器，用于创建实现Java Bean之间转换的扩展映射器。使用MapStruct，我们只需要创建接口，而该库会通过注解在编译过程中自动创建具体的映射实现，大大减少了通常需要手工编写的样板代码的数量。

### MapStruct 依赖

如果你使用Maven的话，可以通过引入依赖安装MapStruct：

```kotlin
<dependencies>
	<dependency>
		<groupId>org.mapstruct</groupId>
		<artifactId>mapstruct</artifactId>
		<version>${org.mapstruct.version}</version>
	</dependency>
</dependencies>
```

这个依赖项会导入MapStruct的核心注释。由于MapStruct在编译时工作，并且会集成到像Maven和Gradle这样的构建工具上，我们还必须在<build中/>标签中添加一个插件`maven-compiler-plugin`，并在其配置中添加`annotationProcessorPaths`，该插件会在构建时生成对应的代码。

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-compiler-plugin</artifactId>
			<version>3.5.1</version>
			<configuration>
				<source>1.8</source>
				<target>1.8</target>
				<annotationProcessorPaths>
					<path>
						<groupId>org.mapstruct</groupId>
						<artifactId>mapstruct-processor</artifactId>
						<version>${org.mapstruct.version}</version>
					</path>
				</annotationProcessorPaths>
			</configuration>
		</plugin>
	</plugins>
</build>
```

如果你使用Gradle的话，安装MapStruct会更简单：

```kotlin
plugins {
	id 'net.ltgt.apt' version '0.20'
}

apply plugin: 'net.ltgt.apt-idea'
apply plugin: 'net.ltgt.apt-eclipse'

dependencies {
	compile "org.mapstruct:mapstruct:${mapstructVersion}"
	annotationProcessor "org.mapstruct:mapstruct-processor:${mapstructVersion}"
}
```

`net.ltgt.apt`插件会负责处理注释。你可以根据你使用的IDE启用插件`apt-idea`或`apt-eclipse`插件。

[MapStruct](https://link.juejin.cn?target=https%3A%2F%2Fsearch.maven.org%2Fclassic%2F%23search%7Cga%7C1%7Cg%3A%22org.mapstruct%22%20AND%20a%3A%22mapstruct%22)及其[处理器](https://link.juejin.cn?target=https%3A%2F%2Fsearch.maven.org%2Fclassic%2F%23search%7Cga%7C1%7Cg%3A%22org.mapstruct%22%20AND%20a%3A%22mapstruct-processor%22)的最新稳定版本都可以从[Maven中央仓库](https://link.juejin.cn?target=https%3A%2F%2Fsearch.maven.org%2Fsearch%3Fq%3Dg%3Aorg.mapstruct)中获得。

# 映射

## 基本映射

我们先从一些基本的映射开始。我们会创建一个Doctor对象和一个DoctorDto。为了方便起见，它们的属性字段都使用相同的名称：

```java
public class Doctor {
	private int id;
	private String name;
	// getters and setters or builder
}
```
```java
public class DoctorDto {
	private int id;
	private String name;
	// getters and setters or builder
}
```

现在，为了在这两者之间进行映射，我们要创建一个`DoctorMapper`接口。对该接口使用`@Mapper`注解，MapStruct就会知道这是两个类之间的映射器。

```java
@Mapper
public interface DoctorMapper {
	DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);
	DoctorDto toDto(Doctor doctor);
}
```

这段代码中创建了一个`DoctorMapper`类型的实例`INSTANCE`，在生成对应的实现代码后，这就是我们调用的“入口”。

我们在接口中定义了`toDto()`方法，该方法接收一个`Doctor`实例为参数，并返回一个`DoctorDto`实例。这足以让MapStruct知道我们想把一个`Doctor`实例映射到一个`DoctorDto`实例。

当我们构建/编译应用程序时，MapStruct注解处理器插件会识别出DoctorMapper接口并为其生成一个实现类。

```java
public class DoctorMapperImpl implements DoctorMapper {
	@Override
	public DoctorDto toDto(Doctor doctor) {
		if ( doctor == null ) {
			return null;
		}
		DoctorDtoBuilder doctorDto = DoctorDto.builder();

		doctorDto.id(doctor.getId());
		doctorDto.name(doctor.getName());

		return doctorDto.build();
	}
}
```

`DoctorMapperImpl`类中包含一个`toDto()`方法，将我们的`Doctor`属性值映射到`DoctorDto`的属性字段中。如果要将`Doctor`实例映射到一个`DoctorDto`实例，可以这样写：

```
DoctorDto doctorDto = DoctorMapper.INSTANCE.toDto(doctor);
```

**注意**：你可能也注意到了上面实现代码中的`DoctorDtoBuilder`。因为builder代码往往比较长，为了简洁起见，这里省略了builder模式的实现代码。如果你的类中包含Builder，MapStruct会尝试使用它来创建实例；如果没有的话，MapStruct将通过`new`关键字进行实例化。

## 不同字段间映射

通常，模型和DTO的字段名不会完全相同。由于团队成员各自指定命名，以及针对不同的调用服务，开发者对返回信息的打包方式选择不同，名称可能会有轻微的变化。

MapStruct通过`@Mapping`注解对这类情况提供了支持。

### 不同属性名称

我们先更新`Doctor`类，添加一个属性`specialty`：

```java
public class Doctor {
	private int id;
	private String name;
	private String specialty;
	// getters and setters or builder
}
```

在`DoctorDto`类中添加一个`specialization`属性：

```java
public class DoctorDto {
	private int id;
	private String name;
	private String specialization;
	// getters and setters or builder
}
```

现在，我们需要让 `DoctorMapper` 知道这里的不一致。我们可以使用 `@Mapping` 注解，并设置其内部的 `source` 和 `target` 标记分别指向不一致的两个字段。

```java
@Mapper
public interface DoctorMapper {
	DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

	@Mapping(source = "doctor.specialty", target = "specialization")
	DoctorDto toDto(Doctor doctor);
}
```

这个注解代码的含义是：`Doctor`中的`specialty`字段对应于`DoctorDto`类的 `specialization` 。

编译之后，会生成如下实现代码：

```java
public class DoctorMapperImpl implements DoctorMapper {
@Override
	public DoctorDto toDto(Doctor doctor) {
		if (doctor == null) {
			return null;
		}

		DoctorDtoBuilder doctorDto = DoctorDto.builder();

		doctorDto.specialization(doctor.getSpecialty());
		doctorDto.id(doctor.getId());
		doctorDto.name(doctor.getName());

		return doctorDto.build();
	}
}
```

### 多个源类

有时，单个类不足以构建DTO，我们可能希望将多个类中的值聚合为一个DTO，供终端用户使用。这也可以通过在`@Mapping`注解中设置适当的标志来完成。

我们先新建另一个对象 `Education`:

```java
public class Education {
	private String degreeName;
	private String institute;
	private Integer yearOfPassing;
	// getters and setters or builder
}
```

然后向 `DoctorDto`中添加一个新的字段：

```java
public class DoctorDto {
	private int id;
	private String name;
	private String degree;
	private String specialization;
	// getters and setters or builder
}
```

接下来，将 `DoctorMapper` 接口更新为如下代码：

```java
@Mapper
public interface DoctorMapper {
	DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

	@Mapping(source = "doctor.specialty", target = "specialization")
	@Mapping(source = "education.degreeName", target = "degree")
	DoctorDto toDto(Doctor doctor, Education education);
}
```

我们添加了另一个`@Mapping`注解，并将其`source`设置为`Education`类的`degreeName`，将`target`设置为`DoctorDto`类的`degree`字段。

如果 `Education` 类和 `Doctor` 类包含同名的字段，我们必须让映射器知道使用哪一个，否则它会抛出一个异常。举例来说，如果两个模型都包含一个`id`字段，我们就要选择将哪个类中的`id`映射到DTO属性中。

## 子对象映射

多数情况下，POJO中不会*只*包含基本数据类型，其中往往会包含其它类。比如说，一个`Doctor`类中会有多个患者类：

```java
public class Patient {
	private int id;
	private String name;
	// getters and setters or builder
}
```

在Doctor中添加一个患者列表`List`：

```java
public class Doctor {
	private int id;
	private String name;
	private String specialty;
	private List<Patient> patientList;
	// getters and setters or builder
}
```

因为`Patient`需要转换，为其创建一个对应的DTO：

```java
public class PatientDto {
	private int id;
	private String name;
	// getters and setters or builder
}
```

最后，在 `DoctorDto` 中新增一个存储 `PatientDto`的列表：

```java
public class DoctorDto {
	private int id;
	private String name;
	private String degree;
	private String specialization;
	private List<PatientDto> patientDtoList;
	// getters and setters or builder
}
```

在修改 `DoctorMapper`之前，我们先创建一个支持 `Patient` 和 `PatientDto` 转换的映射器接口：

```java
@Mapper
public interface PatientMapper {
	PatientMapper INSTANCE = Mappers.getMapper(PatientMapper.class);
	PatientDto toDto(Patient patient);
}
```

这是一个基本映射器，只会处理几个基本数据类型。

然后，我们再来修改 `DoctorMapper` 处理一下患者列表：

```java
@Mapper(uses = {PatientMapper.class})
public interface DoctorMapper {

	DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

	@Mapping(source = "doctor.patientList", target = "patientDtoList")
	@Mapping(source = "doctor.specialty", target = "specialization")
	DoctorDto toDto(Doctor doctor);
}
```

因为我们要处理另一个需要映射的类，所以这里设置了`@Mapper`注解的`uses`标志，这样现在的 `@Mapper` 就可以使用另一个 `@Mapper`映射器。我们这里只加了一个，但你想在这里添加多少class/mapper都可以。

我们已经添加了`uses`标志，所以在为`DoctorMapper`接口生成映射器实现时，MapStruct 也会把 `Patient` 模型转换成 `PatientDto` ——因为我们已经为这个任务注册了 `PatientMapper`。

编译查看最新想实现代码：

```java
public class DoctorMapperImpl implements DoctorMapper {
	private final PatientMapper patientMapper = Mappers.getMapper( PatientMapper.class );

	@Override
	public DoctorDto toDto(Doctor doctor) {
		if ( doctor == null ) {
			return null;
		}

		DoctorDtoBuilder doctorDto = DoctorDto.builder();

		doctorDto.patientDtoList( patientListToPatientDtoList(doctor.getPatientList()));
		doctorDto.specialization( doctor.getSpecialty() );
		doctorDto.id( doctor.getId() );
		doctorDto.name( doctor.getName() );

		return doctorDto.build();
	}
	
	protected List<PatientDto> patientListToPatientDtoList(List<Patient> list) {
		if ( list == null ) {
			return null;
		}

		List<PatientDto> list1 = new ArrayList<PatientDto>( list.size() );
		for ( Patient patient : list ) {
			list1.add( patientMapper.toDto( patient ) );
		}

		return list1;
	}
}
```

显然，除了`toDto()`映射方法外，最终实现中还添加了一个新的映射方法——` patientListToPatientDtoList()`。这个方法是在没有显式定义的情况下添加的，只是因为我们把`PatientMapper`添加到了`DoctorMapper`中。

该方法会遍历一个`Patient`列表，将每个元素转换为`PatientDto`，并将转换后的对象添加到`DoctorDto`对象内中的列表中。

## 更新现有实例

有时，我们希望用DTO的最新值更新一个模型中的属性，对目标对象(我们的例子中是`DoctorDto`)使用`@MappingTarget`注解，就可以更新现有的实例.

```java
@Mapper(uses = {PatientMapper.class})
public interface DoctorMapper {

	DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

	@Mapping(source = "doctorDto.patientDtoList", target = "patientList")
	@Mapping(source = "doctorDto.specialization", target = "specialty")
	void updateModel(DoctorDto doctorDto, @MappingTarget Doctor doctor);
}
```

重新生成实现代码，就可以得到`updateModel()`方法：

```java
public class DoctorMapperImpl implements DoctorMapper {

	@Override
	public void updateModel(DoctorDto doctorDto, Doctor doctor) {
		if (doctorDto == null) {
			return;
		}

		if (doctor.getPatientList() != null) {
			List<Patient> list = patientDtoListToPatientList(doctorDto.getPatientDtoList());
			if (list != null) {
				doctor.getPatientList().clear();
				doctor.getPatientList().addAll(list);
			}
			else {
				doctor.setPatientList(null);
			}
		}
		else {
			List<Patient> list = patientDtoListToPatientList(doctorDto.getPatientDtoList());
			if (list != null) {
				doctor.setPatientList(list);
			}
		}
		doctor.setSpecialty(doctorDto.getSpecialization());
		doctor.setId(doctorDto.getId());
		doctor.setName(doctorDto.getName());
	}
}
```

值得注意的是，由于患者列表是该模型中的子实体，因此患者列表也会进行更新。

# 数据类型转换

## 数据类型映射

MapStruct支持`source`和`target`属性之间的数据类型转换。它还提供了基本类型及其相应的包装类之间的自动转换。

自动类型转换适用于：

- 基本类型及其对应的包装类之间。比如， `int` 和 `Integer`， `float` 和 `Float`， `long` 和 `Long`，`boolean` 和 `Boolean` 等。
- 任意基本类型与任意包装类之间。如 `int` 和 `long`， `byte` 和 `Integer` 等。
- 所有基本类型及包装类与`String`之间。如 `boolean` 和 `String`， `Integer` 和 `String`， `float` 和 `String` 等。
- 枚举和`String`之间。
- Java大数类型(`java.math.BigInteger`， `java.math.BigDecimal`) 和Java基本类型(包括其包装类)与`String`之间。
- 其它情况详见[MapStruct官方文档](https://link.juejin.cn?target=https%3A%2F%2Fmapstruct.org%2Fdocumentation%2Fstable%2Freference%2Fhtml%2F%23implicit-type-conversions)。

因此，在生成映射器代码的过程中，如果源字段和目标字段之间属于上述任何一种情况，则MapStrcut会自行处理类型转换。

我们修改 `PatientDto` ，新增一个 `dateofBirth`字段：

```java
public class PatientDto {
	private int id;
	private String name;
	private LocalDate dateOfBirth;
	// getters and setters or builder
}
```

另一方面，加入 `Patient` 对象中有一个`String` 类型的 `dateOfBirth` ：

```java
public class Patient {
	private int id;
	private String name;
	private String dateOfBirth;
	// getters and setters or builder
}
```

在两者之间创建一个映射器：

```java
@Mapper
public interface PatientMapper {

	@Mapping(source = "dateOfBirth", target = "dateOfBirth", dateFormat = "dd/MMM/yyyy")
	Patient toModel(PatientDto patientDto);
}
```

当对日期进行转换时，我们也可以使用 `dateFormat` 设置格式声明。生成的实现代码形式大致如下：

```java
public class PatientMapperImpl implements PatientMapper {

	@Override
	public Patient toModel(PatientDto patientDto) {
		if (patientDto == null) {
			return null;
		}

		PatientBuilder patient = Patient.builder();

		if (patientDto.getDateOfBirth() != null) {
			patient.dateOfBirth(DateTimeFormatter.ofPattern("dd/MMM/yyyy")
								.format(patientDto.getDateOfBirth()));
		}
		patient.id(patientDto.getId());
		patient.name(patientDto.getName());

		return patient.build();
	}
}
```

可以看到，这里使用了 `dateFormat` 声明的日期格式。如果我们没有声明格式的话，MapStruct会使用 `LocalDate`的默认格式，大致如下：

```java
if (patientDto.getDateOfBirth() != null) {
	patient.dateOfBirth(DateTimeFormatter.ISO_LOCAL_DATE
						.format(patientDto.getDateOfBirth()));
}
```

### 数字格式转换

上面的例子中可以看到，在进行日期转换的时候，可以通过`dateFormat`标志指定日期的格式。

除此之外，对于数字的转换，也可以使用`numberFormat`指定显示格式：

```java
   // 数字格式转换示例
   @Mapping(source = "price", target = "price", numberFormat = "$#.00")
```

## 枚举映射

枚举映射的工作方式与字段映射相同。MapStruct会对具有相同名称的枚举进行映射，这一点没有问题。但是，对于具有不同名称的枚举项，我们需要使用`@ValueMapping`注解。同样，这与普通类型的`@Mapping`注解也相似。

我们先创建两个枚举。第一个是 `PaymentType`:

```java
public enum PaymentType {
	CASH,
	CHEQUE,
	CARD_VISA,
	CARD_MASTER,
	CARD_CREDIT
}
```

比如说，这是一个应用内可用的支付方式，现在我们要根据这些选项创建一个更一般、有限的识图：

```java
public enum PaymentTypeView {
	CASH,
	CHEQUE,
	CARD
}
```

现在，我们创建这两个`enum`之间的映射器接口：

```java
@Mapper
public interface PaymentTypeMapper {

	PaymentTypeMapper INSTANCE = Mappers.getMapper(PaymentTypeMapper.class);

	@ValueMappings({
			@ValueMapping(source = "CARD_VISA", target = "CARD"),
			@ValueMapping(source = "CARD_MASTER", target = "CARD"),
			@ValueMapping(source = "CARD_CREDIT", target = "CARD")
	})
	PaymentTypeView paymentTypeToPaymentTypeView(PaymentType paymentType);
}
```

这个例子中，我们设置了一般性的`CARD`值，和更具体的 `CARD_VISA`, `CARD_MASTER` 和 `CARD_CREDIT` 。两个枚举间的枚举项数量不匹配—— `PaymentType` 有5个值，而 `PaymentTypeView` 只有3个。

为了在这些枚举项之间建立桥梁，我们可以使用`@ValueMappings`注解，该注解中可以包含多个`@ValueMapping`注解。这里，我们将`source`设置为三个具体枚举项之一，并将`target`设置为`CARD`。

MapStruct自然会处理这些情况：

```java
public class PaymentTypeMapperImpl implements PaymentTypeMapper {

	@Override
	public PaymentTypeView paymentTypeToPaymentTypeView(PaymentType paymentType) {
		if (paymentType == null) {
			return null;
		}

		PaymentTypeView paymentTypeView;

		switch (paymentType) {
			case CARD_VISA: paymentTypeView = PaymentTypeView.CARD;
			break;
			case CARD_MASTER: paymentTypeView = PaymentTypeView.CARD;
			break;
			case CARD_CREDIT: paymentTypeView = PaymentTypeView.CARD;
			break;
			case CASH: paymentTypeView = PaymentTypeView.CASH;
			break;
			case CHEQUE: paymentTypeView = PaymentTypeView.CHEQUE;
			break;
			default: throw new IllegalArgumentException( "Unexpected enum constant: " + paymentType );
		}
		return paymentTypeView;
	}
}
```

`CASH`和`CHEQUE`默认转换为对应值，特殊的 `CARD` 值通过`switch`循环处理。

但是，如果你要将很多值转换为一个更一般的值，这种方式就有些不切实际了。其实我们不必手动分配每一个值，只需要让MapStruct将所有剩余的可用枚举项（在目标枚举中找不到相同名称的枚举项），直接转换为对应的另一个枚举项。

可以通过 `MappingConstants`实现这一点：

```java
@ValueMapping(source = MappingConstants.ANY_REMAINING, target = "CARD")
PaymentTypeView paymentTypeToPaymentTypeView(PaymentType paymentType);
```

在这个例子中，完成默认映射之后，所有剩余（未匹配）的枚举项都会映射为`CARD`：

```java
@Override
public PaymentTypeView paymentTypeToPaymentTypeView(PaymentType paymentType) {
	if ( paymentType == null ) {
		return null;
	}

	PaymentTypeView paymentTypeView;

	switch ( paymentType ) {
		case CASH: paymentTypeView = PaymentTypeView.CASH;
		break;
		case CHEQUE: paymentTypeView = PaymentTypeView.CHEQUE;
		break;
		default: paymentTypeView = PaymentTypeView.CARD;
	}
	return paymentTypeView;
}
```

还有一种选择是使用`ANY UNMAPPED`：

```java
@ValueMapping(source = MappingConstants.ANY_UNMAPPED, target = "CARD")
PaymentTypeView paymentTypeToPaymentTypeView(PaymentType paymentType);
```

采用这种方式时，MapStruct不会像前面那样先处理默认映射，再将剩余的枚举项映射到`target`值。而是，直接将*所有*未通过`@ValueMapping`注解做显式映射的值都转换为`target`值。

## 集合映射

简单来说，使用MapStruct处理集合映射的方式与处理简单类型相同。

我们创建一个简单的接口或抽象类并声明映射方法。 MapStruct将根据我们的声明自动生成映射代码。 通常，生成的代码会遍历源集合，将每个元素转换为目标类型，并将每个转换后元素添加到目标集合中。

### List映射

我们先定义一个新的映射方法：

```java
@Mapper
public interface DoctorMapper {
	List<DoctorDto> map(List<Doctor> doctor);
}
```

生成的代码大致如下：

```java
public class DoctorMapperImpl implements DoctorMapper {

	@Override
	public List<DoctorDto> map(List<Doctor> doctor) {
		if ( doctor == null ) {
			return null;
		}

		List<DoctorDto> list = new ArrayList<DoctorDto>( doctor.size() );
		for ( Doctor doctor1 : doctor ) {
			list.add( doctorToDoctorDto( doctor1 ) );
		}

		return list;
	}

	protected DoctorDto doctorToDoctorDto(Doctor doctor) {
		if ( doctor == null ) {
			return null;
		}

		DoctorDto doctorDto = new DoctorDto();

		doctorDto.setId( doctor.getId() );
		doctorDto.setName( doctor.getName() );
		doctorDto.setSpecialization( doctor.getSpecialization() );

		return doctorDto;
	}
}
```

可以看到，MapStruct为我们自动生成了从`Doctor`到`DoctorDto`的映射方法。

但是需要注意，如果我们在DTO中新增一个字段`fullName`，生成代码时会出现错误：

```bash
警告: Unmapped target property: "fullName".
```

基本上，这意味着MapStruct在当前情况下无法为我们自动生成映射方法。因此，我们需要手动定义`Doctor`和`DoctorDto`之间的映射方法。具体参考之前的小节。

### Set和Map映射

Set与Map型数据的处理方式与List相似。按照以下方式修改`DoctorMapper`：

```java
@Mapper
public interface DoctorMapper {

	Set<DoctorDto> setConvert(Set<Doctor> doctor);

	Map<String, DoctorDto> mapConvert(Map<String, Doctor> doctor);
}
```

生成的最终实现代码如下：

```java
public class DoctorMapperImpl implements DoctorMapper {

	@Override
	public Set<DoctorDto> setConvert(Set<Doctor> doctor) {
		if ( doctor == null ) {
			return null;
		}

		Set<DoctorDto> set = new HashSet<DoctorDto>( Math.max( (int) ( doctor.size() / .75f ) + 1, 16 ) );
		for ( Doctor doctor1 : doctor ) {
			set.add( doctorToDoctorDto( doctor1 ) );
		}

		return set;
	}

	@Override
	public Map<String, DoctorDto> mapConvert(Map<String, Doctor> doctor) {
		if ( doctor == null ) {
			return null;
		}

		Map<String, DoctorDto> map = new HashMap<String, DoctorDto>( Math.max( (int) ( doctor.size() / .75f ) + 1, 16 ) );

		for ( java.util.Map.Entry<String, Doctor> entry : doctor.entrySet() ) {
			String key = entry.getKey();
			DoctorDto value = doctorToDoctorDto( entry.getValue() );
			map.put( key, value );
		}

		return map;
	}

	protected DoctorDto doctorToDoctorDto(Doctor doctor) {
		if ( doctor == null ) {
			return null;
		}

		DoctorDto doctorDto = new DoctorDto();

		doctorDto.setId( doctor.getId() );
		doctorDto.setName( doctor.getName() );
		doctorDto.setSpecialization( doctor.getSpecialization() );

		return doctorDto;
	}
}
```

与List映射类似，MapStruct自动生成了`Doctor`转换为`DoctorDto`的映射方法。

### 集合映射策略

很多场景中，我们需要对具有父子关系的数据类型进行转换。通常来说，会有一个数据类型（父），其字段是另一个数据类型（子）的集合。

对于这种情况，MapStruct提供了一种方法来选择如何将子类型设置或添加到父类型中。具体来说，就是`@Mapper `注解中的`collectionMappingStrategy`属性，该属性可以取值为`ACCESSOR_ONLY`， `SETTER_PREFERRED`， `ADDER_PREFERRED` 或`TARGET_IMMUTABLE`。

这些值分别表示不同的为子类型集合赋值的方式。默认值是`ACCESSOR_ONLY`，这意味着只能使用访问器来设置子集合。

当父类型中的*Collection*字段`setter`方法不可用，但我们有一个子类型`add`方法时，这个选项就派上用场了；另一种有用的情况是父类型中的*Collection*字段是不可变的。

我们新建一个类：

```java
public class Hospital {
	private List<Doctor> doctors;
	// getters and setters or builder
}
```

同时定义一个映射目标DTO类，同时定义子类型集合字段的getter、setter和adder：

```java
public class HospitalDto {

	private List<DoctorDto> doctors;

		// 子类型集合字段getter
	public List<DoctorDto> getDoctors() {
		return doctors;
	}
		// 子类型集合字段setter
	public void setDoctors(List<DoctorDto> doctors) {
		this.doctors = doctors;
	}
		// 子类型数据adder
	public void addDoctor(DoctorDto doctorDTO) {
		if (doctors == null) {
			doctors = new ArrayList<>();
		}

		doctors.add(doctorDTO);
	}
}
```

创建对应的映射器：

```java
@Mapper(uses = DoctorMapper.class)
public interface HospitalMapper {
	HospitalMapper INSTANCE = Mappers.getMapper(HospitalMapper.class);

	HospitalDto toDto(Hospital hospital);
}
```

生成的最终实现代码为：

```java
public class HospitalMapperImpl implements HospitalMapper {

	@Override
	public HospitalDto toDto(Hospital hospital) {
		if ( hospital == null ) {
			return null;
		}

		HospitalDto hospitalDto = new HospitalDto();

		hospitalDto.setDoctors( doctorListToDoctorDtoList( hospital.getDoctors() ) );

		return hospitalDto;
	}
}
```

可以看到，在默认情况下采用的策略是`ACCESSOR_ONLY`，使用setter方法`setDoctors()`向`HospitalDto`对象中写入列表数据。

相对的，如果使用 `ADDER_PREFERRED` 作为映射策略：

```java
@Mapper(collectionMappingStrategy = CollectionMappingStrategy.ADDER_PREFERRED,
		uses = DoctorMapper.class)
public interface HospitalMapper {
	HospitalMapper INSTANCE = Mappers.getMapper(HospitalMapper.class);

	HospitalDto toDto(Hospital hospital);
}
```

此时，会使用adder方法逐个将转换后的子类型DTO对象加入父类型的集合字段中。

```java
public class CompanyMapperAdderPreferredImpl implements CompanyMapperAdderPreferred {

	private final EmployeeMapper employeeMapper = Mappers.getMapper( EmployeeMapper.class );

	@Override
	public CompanyDTO map(Company company) {
		if ( company == null ) {
			return null;
		}

		CompanyDTO companyDTO = new CompanyDTO();

		if ( company.getEmployees() != null ) {
			for ( Employee employee : company.getEmployees() ) {
				companyDTO.addEmployee( employeeMapper.map( employee ) );
			}
		}

		return companyDTO;
	}
}
```

如果目标DTO中既没有`setter`方法也没有`adder`方法，会先通过`getter`方法获取子类型集合，再调用集合的对应接口添加子类型对象。

可以在[参考文档](https://link.juejin.cn?target=https%3A%2F%2Fmapstruct.org%2Fdocumentation%2Fstable%2Freference%2Fhtml%2F%23collection-mapping-strategies)中看到不同类型的DTO定义（是否包含setter方法或adder方法），采用不同的映射策略时，所使用的添加子类型到集合中的方式。

### 目标集合实现类型

MapStruct支持将集合接口作为映射方法的目标类型。

在这种情况下，在生成的代码中会使用一些集合接口默认实现。 例如，上面的示例中，`List`的默认实现是`ArrayList`。

常见接口及其对应的默认实现如下：

| Interface type  | Implementation type |
| --------------- | ------------------- |
| `Collection`	| `ArrayList`         |
| `List`          | `ArrayList`         |
| `Map`           | `HashMap`           |
| `SortedMap`     | `TreeMap`           |
| `ConcurrentMap` | `ConcurrentHashMap` |

你可以在[参考文档](https://link.juejin.cn?target=https%3A%2F%2Fmapstruct.org%2Fdocumentation%2Fstable%2Freference%2Fhtml%2F%23implementation-types-for-collection-mappings)中找到MapStruct支持的所有接口列表，以及每个接口对应的默认实现类型。

# 进阶操作

## 依赖注入

到目前为止，我们一直在通过`getMapper()`方法访问生成的映射器：

```java
DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);
```

但是，如果你使用的是Spring，只需要简单修改映射器配置，就可以像常规依赖项一样注入映射器。

修改 `DoctorMapper` 以支持Spring框架：

```java
@Mapper(componentModel = "spring")
public interface DoctorMapper {}
```

在`@Mapper`注解中添加`（componentModel = "spring"）`，是为了告诉MapStruct，在生成映射器实现类时，我们希望它能支持通过Spring的依赖注入来创建。现在，就不需要在接口中添加 `INSTANCE` 字段了。

这次生成的 `DoctorMapperImpl` 会带有 `@Component` 注解：

```java
@Component
public class DoctorMapperImpl implements DoctorMapper {}
```

只要被标记为`@Component`，Spring就可以把它作为一个bean来处理，你就可以在其它类（如控制器）中通过`@Autowire`注解来使用它：

```java
@Controller
public class DoctorController() {
	@Autowired
	private DoctorMapper doctorMapper;
}
```

如果你不使用Spring, MapStruct也支持[Java CDI](https://link.juejin.cn?target=https%3A%2F%2Fdocs.oracle.com%2Fjavaee%2F6%2Ftutorial%2Fdoc%2Fgiwhl.html)：

```java
@Mapper(componentModel = "cdi")
public interface DoctorMapper {}
```

## 添加默认值

`@Mapping` 注解有两个很实用的标志就是常量 `constant` 和默认值 `defaultValue` 。无论`source`如何取值，都将始终使用常量值； 如果`source`取值为`null`，则会使用默认值。

修改一下 `DoctorMapper` ，添加一个 `constant` 和一个 `defaultValue` ：

```java
@Mapper(uses = {PatientMapper.class}, componentModel = "spring")
public interface DoctorMapper {
	@Mapping(target = "id", constant = "-1")
	@Mapping(source = "doctor.patientList", target = "patientDtoList")
	@Mapping(source = "doctor.specialty", target = "specialization", defaultValue = "Information Not Available")
	DoctorDto toDto(Doctor doctor);
}
```

如果`specialty`不可用，我们会替换为`"Information Not Available"`字符串，此外，我们将`id`硬编码为`-1`。

生成代码如下：

```java
@Component
public class DoctorMapperImpl implements DoctorMapper {

	@Autowired
	private PatientMapper patientMapper;
	
	@Override
	public DoctorDto toDto(Doctor doctor) {
		if (doctor == null) {
			return null;
		}

		DoctorDto doctorDto = new DoctorDto();

		if (doctor.getSpecialty() != null) {
			doctorDto.setSpecialization(doctor.getSpecialty());
		}
		else {
			doctorDto.setSpecialization("Information Not Available");
		}
		doctorDto.setPatientDtoList(patientListToPatientDtoList(doctor.getPatientList()));
		doctorDto.setName(doctor.getName());

		doctorDto.setId(-1);

		return doctorDto;
	}
}
```

可以看到，如果 `doctor.getSpecialty()` 返回值为`null`，则将`specialization`设置为我们的默认信息。无论任何情况，都会对 `id`赋值，因为这是一个`constant`。

## 添加表达式

MapStruct甚至允许在`@Mapping`注解中输入Java表达式。你可以设置 `defaultExpression` （ `source` 取值为 `null`时生效），或者一个`expression`（类似常量，永久生效）。

在 `Doctor` 和 `DoctorDto`两个类中都加了两个新属性，一个是 `String` 类型的 `externalId` ，另一个是`LocalDateTime`类型的 `appointment` ，两个类大致如下：

```java
public class Doctor {

	private int id;
	private String name;
	private String externalId;
	private String specialty;
	private LocalDateTime availability;
	private List<Patient> patientList;
	// getters and setters or builder
}
```

```java
public class DoctorDto {

	private int id;
	private String name;
	private String externalId;
	private String specialization;
	private LocalDateTime availability;
	private List<PatientDto> patientDtoList;
	// getters and setters or builder
}
```

修改 `DoctorMapper`：

```java
@Mapper(uses = {PatientMapper.class}, componentModel = "spring", imports = {LocalDateTime.class, UUID.class})
public interface DoctorMapper {

	@Mapping(target = "externalId", expression = "java(UUID.randomUUID().toString())")
	@Mapping(source = "doctor.availability", target = "availability", defaultExpression = "java(LocalDateTime.now())")
	@Mapping(source = "doctor.patientList", target = "patientDtoList")
	@Mapping(source = "doctor.specialty", target = "specialization")
	DoctorDto toDtoWithExpression(Doctor doctor);
}
```

可以看到，这里将 `externalId`的值设置为 `java(UUID.randomUUID().toString())` ，如果源对象中没有 `availability` 属性，则会把目标对象中的 `availability` 设置为一个新的 `LocalDateTime`对象。

由于表达式只是字符串，我们必须在表达式中指定使用的类。但是这里的表达式并不是最终执行的代码，只是一个字母的文本值。因此，我们要在 `@Mapper` 中添加 `imports = {LocalDateTime.class, UUID.class}` 。

## 添加自定义方法

到目前为止，我们一直使用的策略是添加一个“占位符”方法，并期望MapStruct能为我们实现它。其实我们还可以向接口中添加自定义的`default`方法，也可以通过`default`方法直接实现一个映射。然后我们可以通过实例直接调用该方法，没有任何问题。

为此，我们创建一个 `DoctorPatientSummary`类，其中包含一个 `Doctor` 及其 `Patient`列表的汇总信息：

```java
public class DoctorPatientSummary {
	private int doctorId;
	private int patientCount;
	private String doctorName;
	private String specialization;
	private String institute;
	private List<Integer> patientIds;
	// getters and setters or builder
}
```

接下来，我们在 `DoctorMapper`中添加一个`default`方法，该方法会将 `Doctor` 和 `Education` 对象转换为一个 `DoctorPatientSummary`:

```java
@Mapper
public interface DoctorMapper {

	default DoctorPatientSummary toDoctorPatientSummary(Doctor doctor, Education education) {

		return DoctorPatientSummary.builder()
				.doctorId(doctor.getId())
				.doctorName(doctor.getName())
				.patientCount(doctor.getPatientList().size())
								.patientIds(doctor.getPatientList()
						.stream()
					  .map(Patient::getId)
						.collect(Collectors.toList()))
					.institute(education.getInstitute())
				.specialization(education.getDegreeName())
				.build();
	}
}
```

这里使用了Builder模式创建`DoctorPatientSummary`对象。

在MapStruct生成映射器实现类之后，你就可以使用这个实现方法，就像访问任何其它映射器方法一样：

```java
DoctorPatientSummary summary = doctorMapper.toDoctorPatientSummary(dotor, education);
```

## 创建自定义映射器

前面我们一直是通过接口来设计映射器功能，其实我们也可以通过一个带 `@Mapper` 的 `abstract` 类来实现一个映射器。MapStruct也会为这个类创建一个实现，类似于创建一个接口实现。

我们重写一下前面的示例，这一次，我们将它修改为一个抽象类：

```java
@Mapper
public abstract class DoctorCustomMapper {
	public DoctorPatientSummary toDoctorPatientSummary(Doctor doctor, Education education) {

		return DoctorPatientSummary.builder()
				.doctorId(doctor.getId())
				.doctorName(doctor.getName())
				.patientCount(doctor.getPatientList().size())
				.patientIds(doctor.getPatientList()
						.stream()
						.map(Patient::getId)
						.collect(Collectors.toList()))
				.institute(education.getInstitute())
				.specialization(education.getDegreeName())
				.build();
	}
}
```

你可以用同样的方式使用这个映射器。由于限制较少，使用抽象类可以在创建自定义实现时给我们更多的控制和选择。另一个好处是可以添加`@BeforeMapping`和`@AfterMapping`方法。

### @BeforeMapping 和 @AfterMapping

为了进一步控制和定制化，我们可以定义 `@BeforeMapping` 和 `@AfterMapping`方法。显然，这两个方法是在每次映射之前和之后执行的。也就是说，在最终的实现代码中，会在两个对象真正映射之前和之后添加并执行这两个方法。

可以在 `DoctorCustomMapper`中添加两个方法：

```java
@Mapper(uses = {PatientMapper.class}, componentModel = "spring")
public abstract class DoctorCustomMapper {

	@BeforeMapping
	protected void validate(Doctor doctor) {
		if(doctor.getPatientList() == null){
			doctor.setPatientList(new ArrayList<>());
		}
	}

	@AfterMapping
	protected void updateResult(@MappingTarget DoctorDto doctorDto) {
		doctorDto.setName(doctorDto.getName().toUpperCase());
		doctorDto.setDegree(doctorDto.getDegree().toUpperCase());
		doctorDto.setSpecialization(doctorDto.getSpecialization().toUpperCase());
	}

	@Mapping(source = "doctor.patientList", target = "patientDtoList")
	@Mapping(source = "doctor.specialty", target = "specialization")
	public abstract DoctorDto toDoctorDto(Doctor doctor);
}
```

基于该抽象类生成一个映射器实现类：

```java
@Component
public class DoctorCustomMapperImpl extends DoctorCustomMapper {
	
	@Autowired
	private PatientMapper patientMapper;
	
	@Override
	public DoctorDto toDoctorDto(Doctor doctor) {
		validate(doctor);

		if (doctor == null) {
			return null;
		}

		DoctorDto doctorDto = new DoctorDto();

		doctorDto.setPatientDtoList(patientListToPatientDtoList(doctor
			.getPatientList()));
		doctorDto.setSpecialization(doctor.getSpecialty());
		doctorDto.setId(doctor.getId());
		doctorDto.setName(doctor.getName());

		updateResult(doctorDto);

		return doctorDto;
	}
}
```

可以看到， `validate()` 方法会在 `DoctorDto` 对象实例化之前执行，而`updateResult()`方法会在映射结束之后执行。

## 映射异常处理

异常处理是不可避免的，应用程序随时会产生异常状态。MapStruct提供了对异常处理的支持，可以简化开发者的工作。

考虑这样一个场景，我们想在 `Doctor` 映射为`DoctorDto`之前校验一下 `Doctor` 的数据。我们新建一个独立的 `Validator` 类进行校验：

```java
public class Validator {
	public int validateId(int id) throws ValidationException {
		if(id == -1){
			throw new ValidationException("Invalid value in ID");
		}
		return id;
	}
}
```

我们修改一下 `DoctorMapper` 以使用 `Validator` 类，无需指定实现。跟之前一样， 在`@Mapper`使用的类列表中添加该类。我们还需要做的就是告诉MapStruct我们的 `toDto()` 会抛出 `throws ValidationException`：

```java
@Mapper(uses = {PatientMapper.class, Validator.class}, componentModel = "spring")
public interface DoctorMapper {

	@Mapping(source = "doctor.patientList", target = "patientDtoList")
	@Mapping(source = "doctor.specialty", target = "specialization")
	DoctorDto toDto(Doctor doctor) throws ValidationException;
}
```

最终生成的映射器代码如下：

```java
@Component
public class DoctorMapperImpl implements DoctorMapper {

	@Autowired
	private PatientMapper patientMapper;
	@Autowired
	private Validator validator;

	@Override
	public DoctorDto toDto(Doctor doctor) throws ValidationException {
		if (doctor == null) {
			return null;
		}

		DoctorDto doctorDto = new DoctorDto();

		doctorDto.setPatientDtoList(patientListToPatientDtoList(doctor
			.getPatientList()));
		doctorDto.setSpecialization(doctor.getSpecialty());
		doctorDto.setId(validator.validateId(doctor.getId()));
		doctorDto.setName(doctor.getName());
		doctorDto.setExternalId(doctor.getExternalId());
		doctorDto.setAvailability(doctor.getAvailability());

		return doctorDto;
	}
}
```

MapStruct自动将`doctorDto`的`id`设置为`Validator`实例的方法返回值。它还在该方法签名中添加了一个throws子句。

注意，如果映射前后的一对属性的类型与`Validator`中的方法出入参类型一致，那该字段映射时就会调用`Validator`中的方法，所以该方式请谨慎使用。

## 映射配置

MapStruct为编写映射器方法提供了一些非常有用的配置。多数情况下，如果我们已经定义了两个类型之间的映射方法，当我们要添加相同类型之间的另一个映射方法时，我们往往会直接复制已有方法的映射配置。

其实我们不必手动复制这些注解，只需要简单的配置就可以创建一个相同/相似的映射方法。

### 继承配置

我们回顾一下“[更新现有实例](https://juejin.cn/post/6956190395319451679#更新现有实例)”，在该场景中，我们创建了一个映射器，根据DoctorDto对象的属性更新现有的Doctor对象的属性值：

```java
@Mapper(uses = {PatientMapper.class})
public interface DoctorMapper {

	DoctorMapper INSTANCE = Mappers.getMapper(DoctorMapper.class);

	@Mapping(source = "doctorDto.patientDtoList", target = "patientList")
	@Mapping(source = "doctorDto.specialization", target = "specialty")
	void updateModel(DoctorDto doctorDto, @MappingTarget Doctor doctor);
}
```

假设我们还有另一个映射器，将 `DoctorDto`转换为 `Doctor` ：

```java
@Mapper(uses = {PatientMapper.class, Validator.class})
public interface DoctorMapper {

	@Mapping(source = "doctorDto.patientDtoList", target = "patientList")
	@Mapping(source = "doctorDto.specialization", target = "specialty")
	Doctor toModel(DoctorDto doctorDto);
}
```

这两个映射方法使用了相同的注解配置， `source`和 `target`都是相同的。其实我们可以使用`@InheritConfiguration`注释，从而避免这两个映射器方法的重复配置。

如果对一个方法添加 `@InheritConfiguration` 注解，MapStruct会检索其它的已配置方法，寻找可用于当前方法的注解配置。一般来说，这个注解都用于`mapping`方法后面的`update`方法，如下所示：

```java
@Mapper(uses = {PatientMapper.class, Validator.class}, componentModel = "spring")
public interface DoctorMapper {

	@Mapping(source = "doctorDto.specialization", target = "specialty")
	@Mapping(source = "doctorDto.patientDtoList", target = "patientList")
	Doctor toModel(DoctorDto doctorDto);

	@InheritConfiguration
	void updateModel(DoctorDto doctorDto, @MappingTarget Doctor doctor);
}
```

### 继承逆向配置

还有另外一个类似的场景，就是编写映射函数将***Model*** 转为 ***DTO***，以及将 ***DTO*** 转为 ***Model***。如下面的代码所示，我们必须在两个函数上添加相同的注释。

```java
@Mapper(componentModel = "spring")
public interface PatientMapper {

	@Mapping(source = "dateOfBirth", target = "dateOfBirth", dateFormat = "dd/MMM/yyyy")
	Patient toModel(PatientDto patientDto);

	@Mapping(source = "dateOfBirth", target = "dateOfBirth", dateFormat = "dd/MMM/yyyy")
	PatientDto toDto(Patient patient);
}
```

两个方法的配置不会是完全相同的，实际上，它们应该是相反的。将***Model*** 转为 ***DTO***，以及将 ***DTO*** 转为 ***Model***——映射前后的字段相同，但是源属性字段与目标属性字段是相反的。

我们可以在第二个方法上使用`@InheritInverseConfiguration`注解，避免写两遍映射配置：

```java
@Mapper(componentModel = "spring")
public interface PatientMapper {

	@Mapping(source = "dateOfBirth", target = "dateOfBirth", dateFormat = "dd/MMM/yyyy")
	Patient toModel(PatientDto patientDto);

	@InheritInverseConfiguration
	PatientDto toDto(Patient patient);
}
```

这两个Mapper生成的代码是相同的。

# 总结

在本文中，我们探讨了MapStruct——一个用于创建映射器类的库。从基本映射到自定义方法和自定义映射器，此外， 我们还介绍了MapStruct提供的一些高级操作选项，包括依赖注入，数据类型映射、枚举映射和表达式使用。

MapStruct提供了一个功能强大的集成插件，可减少开发人员编写模板代码的工作量，使创建映射器的过程变得简单快捷。

如果要探索更多、更详细的使用方式，可以参考MapStruct官方提供的[参考指南](https://link.juejin.cn?target=https%3A%2F%2Fmapstruct.org%2Fdocumentation%2Fstable%2Freference%2Fhtml%2F)。