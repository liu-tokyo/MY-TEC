# MySQL 安装

当前数据 **Oracle** 旗下的一款产品。

所有平台的 MySQL 下载地址为： [MySQL 下载](https://dev.mysql.com/downloads/mysql/) 。 挑选你需要的 ***MySQL Community Server*** 版本及对应的平台。

> **注意：**安装过程我们需要通过开启管理员权限来安装，否则会由于权限不足导致无法安装。

## **Ubuntu 安装 MySQL**

- 在撰写本文时，Ubuntu存储库中可用的MySQL的最新版本是MySQL 8.0。要安装它，请运行以下命令：

  ```shell
  sudo apt update
  sudo apt install mysql-server
  ```

- 安装完成后，MySQL服务将自动启动。要验证MySQL服务器正在运行，请输入：

  ```shell
  sudo systemctl status mysql
  ```

  显示安装数据库版本：

  ```shell
  mysql --version
  ```

  

- 保护MySQL

  MySQL安装随附一个名为的脚本`mysql_secure_installation`，可让您轻松提高数据库服务器的安全性。

  调用不带参数的脚本：

  ```shell
  sudo mysql_secure_installation
  ```

  - 系统将要求您配置`VALIDATE PASSWORD PLUGIN`用来测试MySQL用户密码强度并提高安全性的密码：Y
  - 密码验证策略分为三个级别：低，中和强。按下`y`如果你想设置的验证密码插件或任何其他键移动到下一个步骤：1
  - 在下一个提示符下，将要求您设置MySQL root用户的密码：qwer1234
  - 如果您设置了验证密码插件，该脚本将向您显示新密码的强度。键入`y`以确认密码：Y
  - 接下来，将要求您删除匿名用户，限制root用户对本地计算机的访问，删除测试数据库并重新加载特权表。您应该回答`y`所有问题。

- 以root身份登录

  要从命令行与MySQL服务器进行交互，请使用MySQL客户端实用程序，该实用程序是作为MySQL服务器软件包的依赖项安装的。

  在MySQL 8.0上，`auth_socket`默认情况下，root用户通过插件进行身份验证。

  该`auth_socket`插件对`localhost`通过Unix套接字文件从进行连接的用户进行身份验证。这意味着您不能通过提供密码来以root用户身份进行身份验证。

  要以root用户身份登录到MySQL服务器，请输入：

  ```shell
  sudo mysql
  ```

  将为您提供MySQL Shell，如下所示，理出所有数据库：

  ```shell
  SHOW DATABASES;
  
  ## 退出数据库管理
  exit
  ```
  
  如果要使用外部程序（例如phpMyAdmin）以root用户身份登录到MySQL服务器，则有两个选择。

  第一个是将身份验证方法从更改`auth_socket`为`mysql_native_password`。您可以通过运行以下命令来做到这一点：

  ```shell
  mysql > ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'very_strong_password';
  mysql > FLUSH PRIVILEGES;
  ```
  
  推荐的第二个选项是创建一个新的专用管理用户，该用户可以访问所有数据库：
  
  ```shell
  GRANT ALL PRIVILEGES ON *.* TO 'administrator'@'localhost' IDENTIFIED BY 'very_strong_password';
  ```

## 配置mysql允许远程连接的方法

默认情况下，mysql只允许本地登录，如果要开启远程连接，则需要修改/etc/mysql/my.conf文件。

1. 修改 **/etc/mysql/my.conf**

   ```shell
   sudo nano /etc/mysql/mysql.cnf
   ```

   找到bind-address = 127.0.0.1这一行

   改为bind-address = 0.0.0.0即可
