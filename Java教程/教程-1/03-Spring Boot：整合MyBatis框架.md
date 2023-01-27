# Spring Boot：整合MyBatis框架

## 综合概述

MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。MyBatis是一款半ORM框架，相对于Hibernate这样的完全ORM框架，MyBatis显得更加灵活，因为可以直接控制SQL语句，所以使用MyBatis可以非常方便的实现各种复杂的查询需求。当然了，有利必有弊，也正因为太过自由，所以需要自己编写SQL语句，而如何编写更为简洁高效的SQL语句，也是一门学问。

## 实现案例

接下来，我们就通过实际案例来讲解MyBatis的整合，然后提供相关的服务来学习了解数据库的操作。

### 生成项目模板

为方便我们初始化项目，Spring Boot给我们提供一个项目模板生成网站。

1.  打开浏览器，访问：https://start.spring.io/

2.  根据页面提示，选择构建工具，开发语言，项目信息等。