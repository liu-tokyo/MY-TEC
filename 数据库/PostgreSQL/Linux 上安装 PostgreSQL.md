# Linux 上安装 PostgreSQL

## 目录

- [Linux 上安装 PostgreSQL](#linux-上安装-postgresql)
  - [目录](#目录)
  - [1. Ubuntu 安装 PostgreSQL](#1-ubuntu-安装-postgresql)
  - [2. 设置超级用户密码](#2-设置超级用户密码)
    - [2.1 修改数据库的 PostgreSQL 用户密码](#21-修改数据库的-postgresql-用户密码)
    - [2.2 修改 Linux 系统 PostgreSQL 用户密码](#22-修改-linux-系统-postgresql-用户密码)
  - [3. 设置PostgreSQL允许被远程访问](#3-设置postgresql允许被远程访问)
  - [4. 下载数据库开发套件](#4-下载数据库开发套件)
  - [相关资源](#相关资源)

---

打开 PostgreSQL 官网 https://www.postgresql.org/，点击菜单栏上的 **Download** ，可以看到这里包含了很多平台的安装包，包括 Linux、Windows、Mac OS等 。

Linux 我们可以看到支持 Ubuntu 和 Red Hat 等各个平台，点击具体的平台链接，即可查看安装方法：



## 1. Ubuntu 安装 PostgreSQL

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



## 2. 设置超级用户密码



### 2.1 修改数据库的 PostgreSQL 用户密码

安装的时候，PostgreSQL数据库创建一个postgres用户作为数据库的管理员，密码随机，所以需要修改密码，方式如下：

1. 登录PostgreSQL

   ```shell
   sudo -u postgres psql
   ```

2. 修改登录PostgreSQL密码

   ```shell
   ALTER USER postgres WITH PASSWORD 'postgres';
   ```
   **注：**

   - 密码postgres要用引号引起来
   - 命令最后有分号

3. 退出PostgreSQL客户端

   ```
   \q
   ```

### 2.2 修改 Linux 系统 PostgreSQL 用户密码

PostgreSQL会创建一个默认的linux用户postgres，修改该用户密码的方法如下：

1. 删除用户postgres的密码：

   ```shell
   sudo passwd -d postgres
   ```

2. 设置用户postgres的密码：

   ```shell
   sudo -u postgres passwd
   ```

   系统提示输入新的密码：

   ```shell
   Enter new UNIX password:
   Retype new UNIX password:
   passwd : password updated successfully
   ```



## 3. 设置PostgreSQL允许被远程访问

1. 修改 **postgresql.conf**

   `postgresql.conf`存放位置在`/etc/postgresql/13.x/main`下，这里的`x`取决于你安装PostgreSQL的版本号：

   ```shell
   cd /etc/postgresql/
   
   ## 查找相应的版本号
   ls -al
   
   cd 13.x
   cd main
   
   sudo nano postgresql.conf
   ```

   

   编辑或添加下面一行，使PostgreSQL可以接受来自任意IP的连接请求。

   ```shell
   listen_addresses = '*'
   ```

2. 修改 **pg_hba.conf**

   `pg_hba.conf`，位置与`postgresql.conf`相同，虽然上面配置允许任意地址连接PostgreSQL，但是这在pg中还不够，我们还需在`pg_hba.conf`中配置服务端允许的认证方式。

   ```shell
   sudo nano pg_hba.conf
   ```

   

   任意编辑器打开该文件，编辑或添加下面一行。

   ```shell
   # TYPE  DATABASE  USER  CIDR-ADDRESS  METHOD
   host  all  all 0.0.0.0/0 md5
   ```

   默认pg只允许本机通过密码认证登录，修改为上面内容后即可以对任意IP访问进行密码验证。对照上面的注释可以很容易搞明白每列的含义，具体的支持项可以查阅文末参考引用。

3. 重启启动 PostgreSQL 服务

   完成上两项配置后执行`sudo service postgresql restart`重启PostgreSQL服务后，允许外网访问的配置就算生效了。
   
   ```shell
   sudo service postgresql restart
   ```

上述配置结束后，就可以通过远程进行访问。



## 4. 下载数据库开发套件

- A5M2
  - https://a5m2.mmatsubara.com/



## 相关资源

- https://www.runoob.com/postgresql/linux-install-postgresql.html

