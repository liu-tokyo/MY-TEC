

# Oracle AQ使用实例


- URL  
  https://www.sodocs.net/doc/068039550.html


## 1. AQ的安装

### 1.1 创建用户

```sql
create user bz_adm identified by bz_adm;
create user bz identified by bz;
```

```sql
alter system set job_queue_processes=4;
alter system set aq_tm_processes=4;
grant connect,resource to bz_adm,bz;
GRANT aq_administrator_role TO bz_adm;
GRANT EXECUTE ON dbms_aq TO bz_adm;
GRANT EXECUTE ON dbms_aqadm TO bz_adm;
GRANT select any dictionary to bz_adm;
GRANT EXECUTE ON dbms_aq to bz;
GRANT ExECUTE ON dbms_aqadm to bz;

```

```sql
grant create session,create table,create view,create sequence,create trigger,create procedure,unlimited tablespace to bz_adm;
grant create session,create table,create view,create sequence,create trigger,create procedure,unlimited tablespace to bz;

```

```sql
GRANT EXECUTE ON DBMS_AQ to bz_adm;
GRANT EXECUTE ON DBMS_AQADM to bz_adm;
GRANT AQ_ADMINISTRATOR_ROLE TO bz_adm;
GRANT ADMINISTER DATABASE TRIGGER TO bz_adm;

```

### 1.2 建立payload type

```sql
connect bz_adm/bz_adm
```

```sql
grant create type，create procedure to bz_adm;
grant create type，create procedure to bz;
```

```sql
CREATE OR REPLACE TYPE bzcardorder_typ AS OBJECT (
    employee_id NUMBER(6),
    first_name VARCHAR2(20),
    last_name VARCHAR2(25),
    ordtyp VARCHAR(10));
/
```

```sql
grant execute on bzcardorder_typ to bz;
```



### 1.3 建立queue table

```sql
execute dbms_aqadm.create_queue_table( -
    queue_table => 'bzcardorders_qt', -
    comment => 'Business Card Orders queue table', - 
    multiple_consumers => true, -
    queue_payload_type => 'bzcardorder_typ');

```

OR

```
exec DBMS_AQADM.CREATE_QUEUE_TABLE( -
           queue_table        => 'bzcardorders_qt', -
           multiple_consumers => TRUE, -
           queue_payload_type => 'bzcardorder_typ' - 
);
```

### 1.4 建立Queue

```sql
exec DBMS_AQADM.CREATE_QUEUE( -
	queue_name  => 'bzcardorders_q', -
	queue_table => 'bzcardorders_qt' -
);
```

```sql
select name, queue_table, queue_type from user_queues; 

```

```sql
EXECUTE dbms_aqadm.grant_queue_privilege( -
	privilege => 'ALL', -
	queue_name => 'bzcardorders_q', -
	grantee => 'bz', -
	grant_option => TRUE -
);

```

### 1.5 添加Subscriber

```sql
exec dbms_aqadm.add_subscriber ( -
	queue_name => 'BZCARDORDERS_Q', -
	subscriber => sys.aq$_agent('SHIPPING',null,null) -
);

```

```sql
exec dbms_aqadm.add_subscriber ( -
	queue_name => 'BZCARDORDERS_Q', -
	subscriber => sys.aq$_agent('BILLING',null,null) -
);

```

-- 基于规则的Subscriber

```sql
exec dbms_aqadm.add_subscriber ( -
	queue_name => 'BZCARDORDERS_Q', -
	subscriber => sys.aq$_agent('RUSH_ORDER',null,null), -
	rule => 'tab.user_data.ordtyp = ''RUSH''' -
);

```

### 1.6.启动queue

```sql
exec DBMS_AQADM.START_QUEUE ( -
	queue_name  => 'bzcardorders_q' -
);

```

```sql
SELECT * FROM aq$bzcardorders_qt_s;

```

## 2. AQ的操作

### 2.1 Enqueue

```sql
declare
	enqopt dbms_aq.enqueue_options_t;
	mprop dbms_aq.message_properties_t;
	enq_msgid RAW(16);
begin
	dbms_aq.enqueue(
		queue_name => 'bz_adm.bzcardorders_q',
		enqueue_options => enqopt,
		message_properties => mprop,
		payload => bz_adm.bzcardorder_typ(101, 'First', 'User','NORMAL'),
		msgid => enq_msgid);
	commit;
end;
/

```

-- Subscriber的Enqueue

```sql
declare
	enqopt dbms_aq.enqueue_options_t;
	mprop dbms_aq.message_properties_t;
	enq_msgid RAW(16);
	rcpt_list dbms_aq.aq$_recipient_list_t;
begin
	rcpt_list(0) := sys.aq$_agent('Shipping', null, null);
	rcpt_list(1) := sys.aq$_agent('Billing', null, null);
	mprop.recipient_list := rcpt_list;
	dbms_aq.enqueue(
		queue_name => 'bzcardorders_q',
		enqueue_options => enqopt,
		message_properties => mprop,
		payload =>bzcardorder_typ(102,'Second','User','NORMAL'),
		msgid => enq_msgid);
	commit;
end;
/

```

```sql
select ENQ_TIME, CONSUMER_NAME, USER_DATA from AQ$BZCARDORDERS_QT;

```



### 2.2 Deuque

```sql
declare
	deqopt dbms_aq.dequeue_options_t;
	mprop dbms_aq.message_properties_t;
	msgid RAW(16);
	payload bz_adm.bzcardorder_typ;
begin
	deqopt.consumer_name := 'SHIPPING';
	deqopt.navigation := DBMS_AQ.FIRST_MESSAGE;
	deqopt.wait := 0;
	dbms_aq.dequeue(
		queue_name => 'bz_adm.bzcardorders_q',
		dequeue_options => deqopt,
		message_properties => mprop,
		payload => payload,
		msgid => msgid);
	commit;
end;
/

```

```sql
select enq_time, deq_time, user_data from AQ$BZCARDORDERS_QT;

```

### 2.3 带延时选项的Enqueue

