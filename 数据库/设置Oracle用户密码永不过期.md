# 设置Oracle用户密码永不过期

- 命令行以sysdba的方式进入

  ```sql
  sqlplus / as sysdba;
  ```

- 查看用户的密码策略，一般是default

  ```sql
  select username,profile from dba_users;
  ```

- 查看指定概要文件（如default）的密码有效期设置

  ```sql
  Select * FROM dba_profiles s Where s.profile='DEFAULT' AND  resource_name='PASSWORD_LIFE_TIME';
  ```

- 将密码有效期由默认的180天修改成“无限制”

  ```sql
  ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;
  ```

- 修改密码，相当于重置密码

  ```sql
  alter user yb_dev identified by yb_dev;
  ```
  
  