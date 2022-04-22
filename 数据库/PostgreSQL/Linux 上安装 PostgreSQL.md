# Linux 上安装 PostgreSQL

打开 PostgreSQL 官网 https://www.postgresql.org/，点击菜单栏上的 **Download** ，可以看到这里包含了很多平台的安装包，包括 Linux、Windows、Mac OS等 。

Linux 我们可以看到支持 Ubuntu 和 Red Hat 等各个平台，点击具体的平台链接，即可查看安装方法：



## Ubuntu 安装 PostgreSQL

- Ubuntu 可以使用 apt-get 安装 PostgreSQL：

  ```shell
  sudo apt-get update
  sudo apt-get install postgresql postgresql-client
  ```

- 安装完毕后，系统会创建一个数据库超级用户 postgres，密码为空。

  ```shell
  sudo -i -u postgres
  ```

- 这时使用以下命令进入 postgres，输出以下信息，说明安装成功：

  ```shell
  ~$ psql
  psql (13.5 (Debian 13.5-0+deb11u1))
  输入 "help" 来获取帮助信息.
  
  postgres=# 
  ```

  

- 输入以下命令退出 PostgreSQL 提示符：

  ```shell
  postgres-# \q
  postgres@debian:~$ exit
  注销
  liu@debian:~$ 
  ```

  PostgreSQL 安装完成后默认是已经启动的，但是也可以通过下面的方式来手动启动服务。

  ```shell
  sudo /etc/init.d/postgresql start   # 开启
  sudo /etc/init.d/postgresql stop    # 关闭
  sudo /etc/init.d/postgresql restart # 重启
  ```







## 相关资源

- https://www.runoob.com/postgresql/linux-install-postgresql.html