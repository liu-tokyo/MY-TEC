# JDBC、Hibernate、HikariCP的区分

JDBC、Hibernate、MyBatis有什么关系？`Hibernate`和`MyBatis`有没有用到`JDBC`？

C3P0、HikariCP连接池为了解决什么问题而出现的？和JDBC是什么关系？

## JDBC:Java Database Connectivity

是一种用于Java与各种数据库连接的标准API,实现java编程和数据库相分离，不同的数据库，只要实现了JDBC接口，java就可以访问对应的数据库。对java程序来说使用什么样的数据库是透明的，由JDBC来处理差异性。

## Hibernate

是一个开源的轻量级的`ORM`框架，它在底层对JDBC进行了封装（ORM:Object Relation Mapping,把面向对象的概念和数据库中表的概念对应起来。一个对象实例对应表中的一条记录）。Hibernate主要优点可以让开发人员以面向对象的思想来操作数据库。MyBatis相较于Hibernate增加了灵活性。

## HikariCP

数据库连接池（C3P0,HikariCP）的主要作用是维护一定数量的数据库连接并对外暴露数据库连接获取及释放的方法。`C3P0`、`HikariCp`是两个开源的数据池连接库，其中`C3P0` 基本已被淘汰，SpringBoot2.0已原生支持`HikariCp`。

## 结论：

1. 只要操作数据库就需要用到JDBC，Hibernate只是为了方便开发人员使用，对JDBC进行了封装。
2. C3P0，HikariCP，druid等是不同的数据库连接池开源库，目的是为了维护与数据库的连接，方便控制应用程序持有的数据库连接数及**快速**操作数据库，作用类似线程池。