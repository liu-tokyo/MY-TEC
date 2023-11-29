# JAVAオブジェクトマッピングライブラリ

> https://qiita.com/chenglin/items/2dd06d979341625e6e82

## はじめに

Beanの変換に詰め替え作業は手動で実施するとコード量は増えてしまい、メンテナンス性は良くないです。
 BeanUtils.copyPropertiesのような便利メソッドがありますが、ほかのマッピングライブラリをまとめてみます。

## FasterXML ObjectMapper

JSON関連する処理はこれを使うと便利です。

- build.gradleに追加

  ```kotlin
  // https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind
  compile group: 'com.fasterxml.jackson.core', name: 'jackson-databind', version: '2.9.8'
  ```

- Beanクラス作成

  ```java
  package com.test.lombok;
  
  import lombok.Getter;
  import lombok.Setter;
  import lombok.ToString;
  
  @Getter
  @Setter
  @ToString
  public class User {
      private String userId;
  
      private String userName;
  
      private String age;
  
      private String gender;
  }
  ```

- user.jsonサンプル作成

  ```json
  {
  "userId": "user001",
  "userName": "テスト 太郎",
  "age": 20,
  "gender": "famale"
  }
  ```

- テストコード

  ```java
  package com.test.lombok;
  
  import java.io.File;
  import java.util.Map;
  
  import com.fasterxml.jackson.databind.ObjectMapper;
  
  public class TestMain {
  
      public static void main(String[] args) throws Exception {
          ObjectMapper mapper = new ObjectMapper();
  
          // ファイルパスで指定
          User user1 = mapper.readValue(
                  new File("C:\\workspace_sts\\lombok\\src\\main\\java\\com\\test\\lombok\\user.json"), User.class);
          System.out.println(user1);
  
          // URL形式で指定
          User user2 = mapper.readValue(TestMain.class.getResource("user.json"), User.class);
          System.out.println(user2);
  
          // 文字列から変換
          User user3 = mapper.readValue(
                  "{\"userId\":\"user001\", \"userName\": \"テスト 太郎２\", \"age\": 22, \"gender\": \"male\"}", User.class);
          System.out.println(user3);
  
          // オブジェクトから文字列に変換
          String jsonString = mapper.writeValueAsString(user3);
          System.out.println(jsonString);
  
          // 文字列からMapに変換
          Map<String, String> userMap = mapper.readValue(jsonString, Map.class);
          System.out.println(userMap);
  
      }
  
  }
  ```

  結果：

  > User(userId=user001, userName=テスト 太郎, age=20, gender=famale)
  >  User(userId=user001, userName=テスト 太郎, age=20, gender=famale)
  >  User(userId=user001, userName=テスト 太郎２, age=22, gender=male)
  >  {"userId":"user001","userName":"テスト 太郎２","age":"22","gender":"male"}
  >  {userId=user001, userName=テスト 太郎２, age=22, gender=male}

## ModelMapper

> http://modelmapper.org/

- build.gradleに追加

  ```kotlin
      // https://mvnrepository.com/artifact/org.modelmapper/modelmapper
      compile group: 'org.modelmapper', name: 'modelmapper', version: '2.3.5'
  ```

- 元Beanクラス

  ```java
  package com.test.lombok;
  
  import lombok.Data;
  
  @Data
  public class User {
      private String userId;
  
      private String userName;
  
      private String age;
  
      private String gender;
  
      private Address addr;
  }
  
  @Data
  class Address {
      private String zipCode;
      private String address1;
      private String address2;
      private String address3;
  }
  ```

- DTOクラス

  ```java
  package com.test.lombok;
  
  import lombok.Getter;
  import lombok.Setter;
  import lombok.ToString;
  
  @Getter
  @Setter
  @ToString
  public class UserDTO {
      private String userId;
  
      private String userName;
  
      private String age;
  
      private String gender;
  
      private String addrZipCode;
  
      private String addrAddress1;
  
      private String addrAddress2;
  
      private String addrAddress3;
  }
  ```

- テストコード

  ```java
  package com.test.lombok;
  
  import org.modelmapper.ModelMapper;
  
  public class TestMain {
  
      public static void main(String[] args) throws Exception {
          ModelMapper modelMapper = new ModelMapper();
  
          User user = new User();
          user.setUserId("user001");
          user.setUserName("テスト");
          user.setAge("18");
          user.setGender("male");
          Address addr = new Address();
          addr.setZipCode("984-0031");
          addr.setAddress1("○○県");
          user.setAddr(addr );
  
          UserDTO userDTO = modelMapper.map(user, UserDTO.class);
          System.out.println(userDTO);
  
      }
  
  }
  ```

  結果：

  > UserDTO(userId=user001, userName=テスト, age=18, gender=male,  addrZipCode=984-0031, addrAddress1=○○県, addrAddress2=null,  addrAddress3=null)

## Explicit Mapping

- 明示的にマッピング

  ```java
  package com.test.lombok;
  
  import org.modelmapper.ModelMapper;
  import org.modelmapper.TypeMap;
  
  public class TestMain {
  
      public static void main(String[] args) throws Exception {
          ModelMapper modelMapper = new ModelMapper();
  
          User user = new User();
          user.setUserId("user001");
          user.setUserName("テスト");
          user.setAge("18");
          user.setGender("male");
          Address addr = new Address();
          addr.setZipCode("984-0031");
          addr.setAddress1("○○県");
          user.setAddr(addr);
  
          TypeMap<User, UserDTO> typeMap = modelMapper.typeMap(User.class, UserDTO.class).addMappings(mapper -> {
              mapper.map(src -> src.getAddr().getZipCode(), UserDTO::setAddrZipCode);
              mapper.map(src -> src.getAddr().getAddress1(), UserDTO::setAddrAddress1);
          });
  
          System.out.println(typeMap.map(user));
  
      }
  
  }
  ```

  結果：

  > UserDTO(userId=user001, userName=テスト, age=18, gender=male,  addrZipCode=984-0031, addrAddress1=○○県, addrAddress2=null,  addrAddress3=null)

  SpringBootのControllerにてFormのBeanから自分のDTOに変換するに使うと便利です。

## MapStruct

> https://mapstruct.org/

- build.gradleに追加

  ```kotlin
  plugins {
      id 'java'
      id 'net.ltgt.apt' version '0.20'
  }
  
  repositories {
      jcenter()
  }
  
  dependencies {
      // lombok
      compileOnly "org.projectlombok:lombok:1.18.12"
      annotationProcessor "org.projectlombok:lombok:1.18.12"
  
      // https://mvnrepository.com/artifact/org.mapstruct/mapstruct
      implementation  group: 'org.mapstruct', name: 'mapstruct', version: '1.3.1.Final'
  
      // https://mvnrepository.com/artifact/org.mapstruct/mapstruct-processor
      annotationProcessor group: 'org.mapstruct', name: 'mapstruct-processor', version: '1.3.1.Final'
  
  }
  
  [compileJava, compileTestJava]*.options*.encoding= "UTF-8"
  
  sourceSets {
      main.java.srcDirs += "build/generated/sources/annotationProcessor/java/main"
  }
  ```

- Mapperインタフェース

  ```java
  package com.test.lombok;
  
  import org.mapstruct.Mapper;
  import org.mapstruct.Mapping;
  import org.mapstruct.factory.Mappers;
  
  @Mapper
  public interface UserMapper {
      UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);
  
      @Mapping(source = "addr.zipCode", target = "addrZipCode")
      @Mapping(source = "addr.address1", target = "addrAddress1")
      @Mapping(source = "addr.address2", target = "addrAddress2")
      UserDTO userToUserDTO(User user);
  }
  ```

  上記定義し、ビルドすると`build\generated\sources\annotationProcessor\java\main\com\test\lombok`に`UserMapperImpl.java`クラスが生成されます。

  `UserMapperImpl.java`

  ```java
  package com.test.lombok;
  
  import javax.annotation.Generated;
  
  @Generated(
      value = "org.mapstruct.ap.MappingProcessor",
      date = "2020-06-28T21:47:47+0900",
      comments = "version: 1.3.1.Final, compiler: javac, environment: Java 1.8.0_201 (Oracle Corporation)"
  )
  public class UserMapperImpl implements UserMapper {
  
      @Override
      public UserDTO userToUserDTO(User user) {
          if ( user == null ) {
              return null;
          }
  
          UserDTO userDTO = new UserDTO();
  
          userDTO.setAddrAddress2( userAddrAddress2( user ) );
          userDTO.setAddrAddress1( userAddrAddress1( user ) );
          userDTO.setAddrZipCode( userAddrZipCode( user ) );
          userDTO.setUserId( user.getUserId() );
          userDTO.setUserName( user.getUserName() );
          userDTO.setAge( user.getAge() );
          userDTO.setGender( user.getGender() );
  
          return userDTO;
      }
  
      private String userAddrAddress2(User user) {
          if ( user == null ) {
              return null;
          }
          Address addr = user.getAddr();
          if ( addr == null ) {
              return null;
          }
          String address2 = addr.getAddress2();
          if ( address2 == null ) {
              return null;
          }
          return address2;
      }
  
      private String userAddrAddress1(User user) {
          if ( user == null ) {
              return null;
          }
          Address addr = user.getAddr();
          if ( addr == null ) {
              return null;
          }
          String address1 = addr.getAddress1();
          if ( address1 == null ) {
              return null;
          }
          return address1;
      }
  
      private String userAddrZipCode(User user) {
          if ( user == null ) {
              return null;
          }
          Address addr = user.getAddr();
          if ( addr == null ) {
              return null;
          }
          String zipCode = addr.getZipCode();
          if ( zipCode == null ) {
              return null;
          }
          return zipCode;
      }
  }
  ```

- テストコード

  ```java
  package com.test.lombok;
  
  public class TestMain {
  
      public static void main(String[] args) throws Exception {
  
          User user = new User();
          user.setUserId("user001");
          user.setUserName("テスト");
          user.setAge("18");
          user.setGender("male");
          Address addr = new Address();
          addr.setZipCode("984-0031");
          addr.setAddress1("○○県");
          user.setAddr(addr);
  
  
          UserDTO userDTO = UserMapper.INSTANCE.userToUserDTO(user);
  
          System.out.println(userDTO);
  
      }
  }
  ```

  結果：

  > UserDTO(userId=user001, userName=テスト, age=18, gender=male,  addrZipCode=984-0031, addrAddress1=○○県, addrAddress2=null,  addrAddress3=null)

- ClassNotFoundエラー

  ```
  Exception in thread "main" java.lang.ExceptionInInitializerError
      at com.test.lombok.TestMain.main(TestMain.java:18)
  Caused by: java.lang.RuntimeException: java.lang.ClassNotFoundException: Cannot find implementation for com.test.lombok.UserMapper
      at org.mapstruct.factory.Mappers.getMapper(Mappers.java:61)
      at com.test.lombok.UserMapper.<clinit>(UserMapper.java:9)
      ... 1 more
  Caused by: java.lang.ClassNotFoundException: Cannot find implementation for com.test.lombok.UserMapper
      at org.mapstruct.factory.Mappers.getMapper(Mappers.java:75)
      at org.mapstruct.factory.Mappers.getMapper(Mappers.java:58)
  ```

  build.gradleにsourceSetsを追加

  ```kotlin
  sourceSets {
      main.java.srcDirs += "build/generated/sources/annotationProcessor/java/main"
  }
  ```

  詳細Doc：https://mapstruct.org/documentation/stable/reference/html/

## Dozer

> https://github.com/DozerMapper/dozer

- build.gradleに追加

  ```kotlin
  // https://mvnrepository.com/artifact/com.github.dozermapper/dozer-core
  compile group: 'com.github.dozermapper', name: 'dozer-core', version: '6.5.0'
  ```

- テストコード

  ```java
  package com.test.lombok;
  
  import org.modelmapper.ModelMapper;
  import org.modelmapper.TypeMap;
  
  import com.github.dozermapper.core.DozerBeanMapperBuilder;
  import com.github.dozermapper.core.Mapper;
  
  public class TestMain {
  
      public static void main(String[] args) throws Exception {
  
          User user = new User();
          user.setUserId("user001");
          user.setUserName("テスト");
          user.setAge("18");
          user.setGender("male");
          Address addr = new Address();
          addr.setZipCode("984-0031");
          addr.setAddress1("○○県");
          user.setAddr(addr);
  
          Mapper mapper = DozerBeanMapperBuilder.buildDefault();
          UserDTO userDTO = mapper.map(user, UserDTO.class);
  
          System.out.println(userDTO);
  
      }
  }
  ```

  結果：

  > UserDTO(userId=user001, userName=テスト, age=18, gender=male,  addrZipCode=null, addrAddress1=null, addrAddress2=null,  addrAddress3=null)

  同じ項目名はコピーされましたが、それ以外nullでした。

  **違う項目の場合はXML、API、アノテーションで定義して対応可能**

  違う項目名の場合は `@Mapping("binaryData")`のように設定で対応可能ですが、複雑のタイプの場合はXMLで設定する必要があります。