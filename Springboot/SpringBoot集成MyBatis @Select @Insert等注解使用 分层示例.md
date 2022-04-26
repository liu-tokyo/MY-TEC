# SpringBoot集成MyBatis @Select @Insert等注解使用 分层示例

## 依赖：

```json
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.4.5</version>
</dependency>
```

## 数据库：

下文省略bean的创建。
![在这里插入图片描述](https://img-blog.csdnimg.cn/4f7bc0baf07b43359466f8d55505f70f.jpg#pic_center)

## 配置文件：

请在src/main/resource文件夹下创建mybatis.xml文件，内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver" />
                <property name="url"
                          value="jdbc:mysql://localhost/csdn_test?useSSL=false&amp;serverTimezone=UTC&amp;allowPublicKeyRetrieval=true" />
                <property name="username" value="root" />
                <property name="password" value="123456a" />
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <!-- 在xml配置下使用resource来指定xml文件 -->
        <!-- <mapper resource="UserMapper.xml" /> -->

        <!-- 注解配置下，使用class来指定Mapper类 -->
        <mapper class="com.example.demo.mapper.UserMapper"/>
    </mappers>

</configuration>
```

并在合适的地方创建**MySqlSessionFactory.java**文件（util或配置文件夹，随意放也可以）。

```java
public class MySqlSessionFactory {
    
    private static SqlSessionFactory sqlSessionFactory = null;
    static{
    
        InputStream input;
        try {
    
            input = Resources.getResourceAsStream("mybatis.xml");
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(input);
        } catch (IOException e) {
    
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

    public static SqlSession getSqlSession(){
    
        return sqlSessionFactory.openSession();
    }

    public static SqlSessionFactory getSqlSessionFactory(){
    
        return sqlSessionFactory;
    }
}
```

## .mapper层

该层是最接近数据库的一层。这里我们采用注解方式进行编写。有的ORM叫做`.reponsitory`层，请注意，`.mapper`层与其是由区别的，其作用是不完全相同的。前者大多为框架对抽象方法和注解的包装使用，并传递到下一层进行数据流转换；而后者是不仅包括框架对抽象方法和注解的包装使用，并传递到下一层进行数据流转换，还包括了动态方法的代理使用，也就是说该层虽然是interface，但可直接调用进行数据流处理。

有关注解的好处，官方是这样与xml进行解释的：

> 使用注解来映射简单语句会使代码显得更加简洁，但对于稍微复杂一点的语句，Java 注解不仅力不从心，还会让你本就复杂的 SQL
> 语句更加混乱不堪。 因此，如果你需要做一些很复杂的操作，最好用 XML 来映射语句。
>
> 选择何种方式来配置映射，以及认为是否应该要统一映射语句定义的形式，完全取决于你和你的团队。
> 换句话说，永远不要拘泥于一种方式，你可以很轻松的在基于注解和 XML 的语句映射方式间自由移植和切换。

参数以`#{para}`形式进行编码。

```java
public interface UserMapper {
    

    @Select("select id, name from business_bean")
    List<Business> findAll();

    @Insert("insert into business_bean (id, name) values(#{id}, #{name})")
    void addUser(Business business);

    @Delete("delete from business_bean where id = #{id}")
    void deleteUser(String id);

    @Update("update business_bean set name = #{name} wehre id = #{id}")
    void updateUser(Business business);
}
```

## .service层

```java
public interface UserService {
    

    List<Business> findAll();

    void addUser(Business business);

    void deleteUser(String id);

    void updateUser(Business business);
}
```

```java
@Service
public class UserServiceImpl implements UserService {
    

    @Override
    public List<Business> findAll() {
    
        List<Business> businessList;
        SqlSessionFactory factory= MySqlSessionFactory.getSqlSessionFactory();
        SqlSession session = factory.openSession();
        UserMapper mapper = session.getMapper(UserMapper.class);
        businessList = mapper.findAll();
        session.close();
        return businessList;
    }

    @Override
    public void addUser(Business business) {
    
        SqlSessionFactory factory= MySqlSessionFactory.getSqlSessionFactory();
        SqlSession session = factory.openSession();
        UserMapper mapper = session.getMapper(UserMapper.class);
        mapper.addUser(business);
        session.commit();
        session.close();
    }

    @Override
    public void deleteUser(String id) {
    
        SqlSessionFactory factory= MySqlSessionFactory.getSqlSessionFactory();
        SqlSession session = factory.openSession();
        UserMapper mapper = session.getMapper(UserMapper.class);
        mapper.deleteUser(id);
        session.commit();
        session.close();
    }

    @Override
    public void updateUser(Business business) {
    
        SqlSessionFactory factory= MySqlSessionFactory.getSqlSessionFactory();
        SqlSession session = factory.openSession();
        UserMapper mapper = session.getMapper(UserMapper.class);
        mapper.updateUser(business);
        session.commit();
        session.close();
    }
}
```

## .controller层

```java
@RestController
public class FindController {
    

    @Resource
    UserService userService;

    @RequestMapping(value = "/test", method = RequestMethod.GET)
    public void testController() {
    
        userService.addUser(new Business("test5", "21"));
    }

}
```