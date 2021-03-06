# 创建 `USB存储器` 、轻松安装各种操作系统 `UNetbootin` 

安装程序自动从Web获取，支持的操作系统有“Ubuntu”、“FreeBSD”等25种。

- 如果您指定操作系统类型和版本，安装“**USB存储器**”将完全自动创建。如果您指定操作系统类型和版本，安装“**USB 存储器**”将自动创建。

- “UNetbootin”是可以从Web获取各种OS的安装包，写入USB存储器等外部媒体，制作可启动安装媒体的软件。是一款兼容Windows 2000/XP/Vista等的免费软件，经编辑部确认在Windows Vista上运行。您可以从该软件的官方网站下载。该软件的分发文件不是压缩的，而是可执行文件本身。

- 该软件可以从网络下载各种操作系统的安装程序，并创建一个非安装CD的安装“USB存储器”。除了“Ubuntu”、“Fedora”、“Debian”、“CentOS”等各种著名的Linux发行版外，还支持“FreeBSD”、“NetBSD”等25种操作系统。推荐给想要在没有内置 CD 驱动器（如上网本）的 PC 上尝试各种操作系统的用户。

- 它易于使用，只需指定操作系统类型和版本。之后，本软件会自动从网络上获取安装包，并将其写入到U盘等外部媒体中，创建可启动的安装媒体。在某些操作系统上，还可以获得每日更新的开发版本（夜间构建、每日构建），其中包含最新的错误修复和实验性功能。

- 此外，由于它还支持写入 ISO 格式的映像文件和软盘映像文件，因此可以创建该软件不支持自动获取安装程序的 OS 安装介质。

- 您也可以在创建安装媒体后重新启动 PC。如果您保留插入的媒体，您可以在重新启动后继续进行操作系统安装工作。

## 官方说明：

官网：https://unetbootin.github.io/

- Ubuntu 如下命令安装该软件：

  ```shell
  sudo add-apt-repository ppa:gezakovacs/ppa
  sudo apt-get update
  sudo apt-get install unetbootin
  ```

  安装出现问题。这个软件源存在问题，无法正确加入。
  
  可能是该软件在旧版本的Linux上运行，还是可以的，我用的是Ubuntu20.04，感觉就是不行。
  
- Windows 下面安装尝试：

  相对比较顺利！
  
  注意：因为有一个巨大文件需要拷贝，所以过程中，会出现类似于停止的感觉。只要去看 **U盘** 的容量在变化，就能确认其实拷贝一直在进行。