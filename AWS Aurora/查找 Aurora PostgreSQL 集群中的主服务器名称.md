



## 查找 Aurora PostgreSQL 集群中的主服务器名称

- 参照URL：https://aws.amazon.com/cn/blogs/database/failover-with-amazon-aurora-postgresql/

```sql
select server_id from aurora_replica_status() where session_id='MASTER_SESSION_ID';
```

