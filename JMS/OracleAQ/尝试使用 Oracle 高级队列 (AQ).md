# 尝试使用 Oracle 高级队列 (AQ) 



## 前提

- 本次实验，针对队列消息发送、接收，提前确定以下 7 点：

  |  No  | 项目               | 说明                                                 | 数据设置                     |
  | :--: | ------------------ | ---------------------------------------------------- | ---------------------------- |
  |  1   | 角色(schema)       | 发送方模式，接收方模式。                             | USER1（发送）、USER2（接收） |
  |  2   | 数据库链接名称     | 仅限发送方。需要公共数据库链接。                     | DBLINK_USER2                 |
  |  3   | 消息的用户定义类型 | 发送者和接收者具有相同的定义和名称。定义消息的结构。 | MYMESG                       |
  |  4   | 队列表名称         | 发送者和接收者具有相同的定义和名称。                 | MYMESG_TAB                   |
  |  5   | 队列名称           | 发送者和接收者具有相同的定义和名称。                 | MYMESG_Q                     |
  |  6   | 订户名称           | 发送者和接收者具有相同的定义和名称。目的地的标识符。 | SUBSCR                       |
  |  7   | 回调函数           | 仅在接收端定义。定义异步业务处理。。                 | AUTO_RECV                    |

  



## 1. 创建新的用户

- 创建如下用户（口令：`Zliu0922#`）：

  ```sql
  create user USER1 identified by Zliu0922#;
  create user USER2 identified by Zliu0922#;
  ```

  **注意**：12.2.0版本比较奇怪，需要按照如下写法：

  ```sql
  SQL> create user c##USER1 identified by Zliu0922#;
  SQL> create user c##USER2 identified by Zliu0922#;
  ```

  ※用户的名称前面加 `c##` ，就是这么任性。

- 赋予权限：

  ```sql
  grant create session,create table,create view,create sequence,create trigger,create procedure,unlimited tablespace to USER1;
  grant create session,create table,create view,create sequence,create trigger,create procedure,unlimited tablespace to USER2;
  ```

  ```sql
  grant create type，create procedure to USER1;
  grant create type，create procedure to USER2;
  ```

  ※权限不足，需要增加相应权限。

- Oracle官方简易权限（SCOTT用户）

  ```sql
  GRANT EXECUTE ON DBMS_AQ to USER1;
  GRANT EXECUTE ON DBMS_AQADM to USER1;
  GRANT AQ_ADMINISTRATOR_ROLE TO USER1;
  GRANT ADMINISTER DATABASE TRIGGER TO USER1;
  ```

  ```sql
  GRANT EXECUTE ON DBMS_AQ to USER2;
  GRANT EXECUTE ON DBMS_AQADM to USER2;
  GRANT AQ_ADMINISTRATOR_ROLE TO USER2;
  GRANT ADMINISTER DATABASE TRIGGER TO USER2;
  ```

- 常用权限参考

  |  No  | 权限                  | 说明             |
  | :--: | --------------------- | ---------------- |
  |  １  | create session        | 连接数据库       |
  |  2   | create view           | 建视图           |
  |  3   | create procedure      | 建过程、函数、包 |
  |  4   | create cluster        | 建簇             |
  |  5   | create table          | 建表             |
  |  6   | create public synonym | 建同义词         |
  |  7   | create trigger        | 建触发器         |

  



## 2. 发送用户设置

- 创建公共数据库链接

  ```sql
  SQL> CREATE PUBLIC DATABASE LINK DBLINK_USER2 CONNECT TO USER2 IDENTIFIED BY USER2 USING 'ORCL';
  ```

- 赋予权限

  ```sql
  SQL> GRANT AQ_ADMINISTRATOR_ROLE TO USER1;
  ```

- 创建用户定义的类型、队列表、队列和启动队列

  按照按照顺序，4个指令逐个执行：
  
  ```sql
  SQL> CREATE TYPE MYMESG AS OBJECT (
             ID    NUMBER(10,0),
             MESG  VARCHAR2(30)
           )
  /
  SQL> exec DBMS_AQADM.CREATE_QUEUE_TABLE( -
             queue_table        => 'MYMESG_TAB', -
             queue_payload_type => 'MYMESG', -
             multiple_consumers => TRUE -
         );
  SQL> exec DBMS_AQADM.CREATE_QUEUE( -
             queue_name  => 'MYMESG_Q', -
             queue_table => 'MYMESG_TAB' -
         );
  SQL> exec DBMS_AQADM.START_QUEUE ( -
             queue_name  => 'MYMESG_Q' -
         );
  ```
  
  

## 3. 接收用户设置

- 赋予权限

  ```sql
  SQL> GRANT AQ_ADMINISTRATOR_ROLE TO USER2;
  SQL> GRANT EXECUTE ON DBMS_AQ    TO USER2;
  ```

- 创建用户定义的类型、队列表、队列和启动队列

  ※与发件人相同，已经创建，无需再次创建。

  ```sql
  SQL> CREATE TYPE MYMESG AS OBJECT (
             ID    NUMBER(10,0),
             MESG  VARCHAR2(30)
           )
  /
  SQL> exec DBMS_AQADM.CREATE_QUEUE_TABLE( -
                  queue_table        => 'MYMESG_TAB', -
                  queue_payload_type => 'MYMESG', -
                  multiple_consumers => TRUE -
           );
  SQL> exec DBMS_AQADM.CREATE_QUEUE( -
                queue_name  => 'MYMESG_Q', -
                queue_table => 'MYMESG_TAB' -
       );
  SQL> exec DBMS_AQADM.START_QUEUE ( -
                queue_name  => 'MYMESG_Q' -
       );
  ```

- 注册订阅者 - Consumer

  ```sql
  SQL> exec DBMS_AQADM.ADD_SUBSCRIBER( -
                  queue_name  => 'MYMESG_Q', -
                  subscriber  => SYS.AQ$_AGENT('SUBSCR',NULL,NULL), -
                  rule        => NULL -
           );
  ```

  

## 4. 队列发送设置

- 注册订阅者

  ```sql
  SQL> exec DBMS_AQADM.ADD_SUBSCRIBER( -
                  queue_name  => 'MYMESG_Q', -
                  subscriber  => SYS.AQ$_AGENT('SUBSCR','USER2.MYMESG_Q@DBLINK_USER2',NULL), -
                  rule        => NULL -
           );
  ```

- 开始传播（预定传输到远程实例）

  ```sql
  SQL> exec DBMS_AQADM.SCHEDULE_PROPAGATION( -
             queue_name    => 'MYMESG_Q', -
             destination   => 'DBLINK_USER2', -
             duration      => NULL, -
             latency       => 0 -
           );
  ```

  ※应该为每个 DB 链接目标启动一次传播。



## 5. 队列接收设置

- 创建接收数据表

  ```sql
  CREATE table LOG_MESG (
             ID    NUMBER(10,0) primary key,
             MESG  VARCHAR2(30) not null 
           );
  /
  ```

  

- 创建一个在接收方收到消息时将消息自动**出列（收信）**的过程：

  ```sql
  SQL> CREATE OR REPLACE PROCEDURE USER2.AUTO_RECV(
               context    IN RAW,
               reginfo    IN SYS.AQ$_REG_INFO,
               descr      IN SYS.AQ$_DESCRIPTOR,
               payload    IN VARCHAR2,
               payloadl   IN NUMBER
           )IS
               mesg    MYMESG;
               dequeue_options     DBMS_AQ.DEQUEUE_OPTIONS_T;
               message_properties  DBMS_AQ.MESSAGE_PROPERTIES_T;
               message_handle      RAW(16);
           BEGIN
               -- サブスクライバ名を設定
               dequeue_options.CONSUMER_NAME := 'SUBSCR';
               DBMS_AQ.DEQUEUE(queue_name         => 'USER2.MYMESG_Q',
                               dequeue_options    => dequeue_options,
                               message_properties => message_properties,
                               payload            => mesg,
                               msgid              => message_handle);
               --
               -- ここから業務処理を実装する。今回はINSERTのみする。
               --
               INSERT INTO LOG_MESG VALUES(mesg.ID,  mesg.MESG);
               COMMIT;
           EXCEPTION
               WHEN OTHERS THEN Null;
           END AUTO_RECV;
  /
  ```

- 将上述过程注册到自动出列。

  ```sql
  SQL> DECLARE
               REGINFO  SYS.AQ$_REG_INFO;
               REGINFOS SYS.AQ$_REG_INFO_LIST;
       BEGIN
           REGINFO := SYS.AQ$_REG_INFO(
                       'USER2.MYMESG_Q:SUBSCR',
                        DBMS_AQ.NAMESPACE_AQ,
                        'plsql://USER2.AUTO_RECV?PR=1',
                        NULL
                      );
           REGINFOS := SYS.AQ$_REG_INFO_LIST(REGINFO);
           DBMS_AQ.REGISTER( REGINFOS,1);
       END;
  /
  ```
  
  

## 6. 队列送信示例

设置完成。  
Enqueue（消息传输）执行以下 PL/SQL。

实际上，创建包装此处理的过程或函数很方便。

- 送信指令

  ```sql
  DECLARE
      mesg MYMESG;
      enqueue_options     dbms_aq.enqueue_options_t;
      message_properties  dbms_aq.message_properties_t;
      message_handle      RAW(16);
  BEGIN
      -- ひとつめのメッセージを送信
      mesg := MYMESG( 1 ,'こんにちは' );
      dbms_aq.enqueue(queue_name         => 'MYMESG_Q',
                      enqueue_options    => enqueue_options,
                      message_properties => message_properties,
                      payload            => mesg,
                      msgid              => message_handle
      );
      -- ふたつめのメッセージを送信
      mesg := MYMESG( 2 ,'今晩は' );
      dbms_aq.enqueue(queue_name         => 'MYMESG_Q',
                      enqueue_options    => enqueue_options,
                      message_properties => message_properties,
                      payload            => mesg,
                      msgid              => message_handle
      );
      -- トランザクションを終了（COMMIT or ROLLBACK)しないとキューに溜められない
      COMMIT;
  END;
  /
  ```

  执行上述操作后，将执行自动出列程序，并将消息注册到 LOG_MESG 表中。



## 7. 队列接收指令

- 检查自动出队设置

  `system` 用户

  ```sql
  SQL> SELECT * FROM SYS.REG$;
  ```

- 删除自动出队

  ```sql
  DECLARE
      REGINFO  SYS.AQ$_REG_INFO;
      REGINFOS SYS.AQ$_REG_INFO_LIST;
  BEGIN
      REGINFO := SYS.AQ$_REG_INFO(
                   'USER2.MYMESG_Q:SUBSCR',
                   DBMS_AQ.NAMESPACE_AQ,
                   'plsql://USER2.AUTO_RECV?PR=1',
                   ''
                 );
      REGINFOS := SYS.AQ$_REG_INFO_LIST(REGINFO);
      DBMS_AQ.UNREGISTER( REGINFOS,1);
  END;
  /
  ```

- 手动出队

  前提：设置输出显示有效

  ```sql
  set serveroutput on
  ```

  手动出队：Consumer

  ```sql
  DECLARE
      mesg MYMESG;
      dequeue_options     dbms_aq.dequeue_options_t;
      message_properties  dbms_aq.message_properties_t;
      message_handle      RAW(16);
  BEGIN
      dequeue_options.CONSUMER_NAME := 'SUBSCR';
  
      DBMS_AQ.DEQUEUE(queue_name         => 'MYMESG_Q',
                      dequeue_options    => dequeue_options,
                      message_properties => message_properties,
                      payload            => mesg,
                      msgid              => message_handle);
  
      DBMS_OUTPUT.PUT_LINE ('Message[' || mesg.id || '][' || mesg.mesg || ']');
      COMMIT;
  END;
  /
  ```

- 将 AQ 迁移到 PostgreSQL 时

  从 Oracle 迁移到 PostgreSQL 时，您似乎可以通过结合 PostgreSQL 的 dblink 和 NOTIFY/LISTEN 函数来做类似于 AQ 的事情。 还没有尝试过，但它使功能更类似于 DBMS_PIPE 而不是 AQ。

如果要进行完整的异步处理，可能需要引入一个特殊的中间件，但对于小型系统，可以使用 Oracle 数据库的标准函数 AQ 来实现。

由于它是 9i 中的一个函数，并且也在数据库内部使用，所以我认为它是一个死函数。

除了 AQ，Oracle 数据库还具有 RDBMS 范围之外的各种功能。 我认为付费选项的功能已被最大限度地使用，但为什么不尝试使用其他免费功能呢？ . 我认为这将提高开发效率。

## 8. 如何解锁用户

如果在Oracle11g在安装过程中忘了进行口令配置，不要着急，无需卸载重载。可以通过[命令行](https://so.csdn.net/so/search?q=命令行&spm=1001.2101.3001.7020)进行修改。  
比如我们忘记解锁scott用户，可以先打开SQL plus工具。

- 输入用户：`sys`

- 输入口令：`sys as sysdba`

  ```sql
  sys as sysdba
  ```

- 然后输入解锁scott用户语句（分号不要忘了）：

  ```sql
  alter user scott account unlock;
  ```

- 然后输入修改scott密码为tiger（分号不要忘了）：

  ```sql
  alter user scott identified by Zliu0922#;
  ```

- 提交修改：

  ```sql
  commit;
  ```

- 用该用户登录

  ```sql
  connect scott
  ```

  输入口令即可正常登录。

- 显示当前用户

  ```sql
  show user
  ```

- 赋予权限

  ```sql
  GRANT AQ_ADMINISTRATOR_ROLE TO scott;
  ```

- 查看一下所有用户所在的表空间

  ```sql
  select username,default_tablespace from dba_users;
  ```

## 9. 删除用户信息

- 查看用户

  ```sql
  select * from all_users
  select * from user_users
  select * from dba_users
  ```

- 查看用户的连接状况

  ```sql
  select username,sid,serial# from v$session where username = 'NCC'
  ```

- 找到要删除用户的sid,和serial，并删除

  ```sql
  alter system kill session '4521,27770'
  ```

- 删除用户

  ```sql
  drop user ncc cascade
  ```

  如果在 drop 后还提示 ORA-01940:无法删除当前已链接的用户，
  说明还有连接的 session，可以通过查看 session 的状态来确定该 session 是否被 kill 了，
  用如下语句查看：

  ```sql
  select saddr,sid,serial#,paddr,username,status from v$session where username is not null
  ```

  也可以使用 Oracle 自带的工具来管理用户及其信息。

  

## 官方编程参照

- [Oracleアドバンスト・キューイング(AQ)](https://docs.oracle.com/cd/E16338_01/java.112/b56281/streamsaq.htm)