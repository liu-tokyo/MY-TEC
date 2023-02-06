# Linux中安装 IBM-MQ 及常用命令

## 软件下载

- 官网下载：

  http://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/messaging/mqadv/

## 软件安装

### 软件解压

- 将下载的安装包传到虚拟机中并解压：

  ```
  tar -zxvf mqadv_dev90_linux_x86-64.tar.gz
  cd MQServer/
  ```

  文件详细：

  ```
  [root@localhost MQServer]# ls
  copyright                          MQSeriesExplorer-9.0.0-0.x86_64.rpm   MQSeriesJRE-9.0.0-0.x86_64.rpm     MQSeriesMsg_ja-9.0.0-0.x86_64.rpm     MQSeriesSamples-9.0.0-0.x86_64.rpm
  crtmqpkg                           MQSeriesFTAgent-9.0.0-0.x86_64.rpm    MQSeriesMan-9.0.0-0.x86_64.rpm     MQSeriesMsg_ko-9.0.0-0.x86_64.rpm     MQSeriesSDK-9.0.0-0.x86_64.rpm
  lap                                MQSeriesFTBase-9.0.0-0.x86_64.rpm     MQSeriesMsg_cs-9.0.0-0.x86_64.rpm  MQSeriesMsg_pl-9.0.0-0.x86_64.rpm     MQSeriesServer-9.0.0-0.x86_64.rpm
  licenses                           MQSeriesFTLogger-9.0.0-0.x86_64.rpm   MQSeriesMsg_de-9.0.0-0.x86_64.rpm  MQSeriesMsg_pt-9.0.0-0.x86_64.rpm     MQSeriesXRService-9.0.0-0.x86_64.rpm
  mqlicense.sh                       MQSeriesFTService-9.0.0-0.x86_64.rpm  MQSeriesMsg_es-9.0.0-0.x86_64.rpm  MQSeriesMsg_ru-9.0.0-0.x86_64.rpm     PreReqs
  MQSeriesAMQP-9.0.0-0.x86_64.rpm    MQSeriesFTTools-9.0.0-0.x86_64.rpm    MQSeriesMsg_fr-9.0.0-0.x86_64.rpm  MQSeriesMsg_Zh_CN-9.0.0-0.x86_64.rpm  READMES
  MQSeriesAMS-9.0.0-0.x86_64.rpm     MQSeriesGSKit-9.0.0-0.x86_64.rpm      MQSeriesMsg_hu-9.0.0-0.x86_64.rpm  MQSeriesMsg_Zh_TW-9.0.0-0.x86_64.rpm  repackage
  MQSeriesClient-9.0.0-0.x86_64.rpm  MQSeriesJava-9.0.0-0.x86_64.rpm       MQSeriesMsg_it-9.0.0-0.x86_64.rpm  MQSeriesRuntime-9.0.0-0.x86_64.rpm
  ```

### 软件许可

- 执行接受许可脚本

  ```
  ./mqlicense.sh -accpet
  ```

  许可详细：

  ```
  WARNING: Unable to determine distribution and release for this system.
           Check that it is supported before continuing with installation.
  
  Licensed Materials - Property of IBM
  
   5724-H72
  
   (C) Copyright IBM Corporation 1993, 2016
  
  US Government Users Restricted Rights - Use, duplication or disclosure
  restricted by GSA ADP Schedule Contract with IBM Corp.
  
  
  
  
  Displaying license agreement on :0
  
  Agreement accepted:  Proceed with install.
  ```

### 安装服务器

- 安装WebSphere MQ for linux服务器（Runtime、SDK 和 Server 软件包）

  ```
  rpm -ivh MQSeriesRuntime-9.0.0-0.x86_64.rpm
  rpm -ivh MQSeriesSDK-9.0.0-0.x86_64.rpm
  rpm -ivh MQSeriesServer-9.0.0-0.x86_64.rpm
  rpm -ivh MQSeriesClient-9.0.0-0.x86_64.rpm
  ```

  安装详细：

  ```
  [root@localhost MQServer]# rpm -ivh MQSeriesRuntime-9.0.0-0.x86_64.rpm
  Preparing...                          ################################# [100%]
  Creating group mqm
  Creating user mqm
  Updating / installing...
     1:MQSeriesRuntime-9.0.0-0          ################################# [100%]
  [root@localhost MQServer]# rpm -ivh MQSeriesSDK-9.0.0-0.x86_64.rpm
  Preparing...                          ################################# [100%]
  Updating / installing...
     1:MQSeriesSDK-9.0.0-0              ################################# [100%]
  [root@localhost MQServer]# rpm -ivh MQSeriesServer-9.0.0-0.x86_64.rpm
  Preparing...                          ################################# [100%]
  Updating / installing...
     1:MQSeriesServer-9.0.0-0           ################################# [100%]
  Updated PAM configuration in /etc/pam.d/ibmmq
  
  WARNING: System settings for this system do not meet recommendations for this product
           See the log file at "/tmp/mqconfig.52675.log" for more information
  [root@localhost MQServer]# rpm -ivh MQSeriesClient-9.0.0-0.x86_64.rpm
  Preparing...                          ################################# [100%]
  Updating / installing...
     1:MQSeriesClient-9.0.0-0           ################################# [100%]
  ```

  ps：MQSeriesRuntime-9.0.0-0.x86_64.rpm时候,安装程序为系统自动创建了一个mqm用户和mqm组。对mq的配置需要使用该用户

### 安装样本

- 安装 WebSphere MQ 样本程序

  其中包括amqsput、amqsget、amqsgbr和amqsbcg等命令

  ```
  rpm -ivh MQSeriesSamples-9.0.0-0.x86_64.rpm
  ```

  安装信息：

  ```
  [root@localhost MQServer]# rpm -ivh MQSeriesSamples-9.0.0-0.x86_64.rpm
  Preparing...                          ################################# [100%]
  Updating / installing...
     1:MQSeriesSamples-9.0.0-0          ################################# [100%]
  ```

### 其它软件包

- 安装 MQSeriesMan

  ```
  rpm -ivh MQSeriesMan-9.0.0-0.x86_64.rpm
  ```

  安装详细：

  ```
  [root@localhost MQServer]# rpm -ivh MQSeriesMan-9.0.0-0.x86_64.rpm
  Preparing...                          ################################# [100%]
  Updating / installing...
     1:MQSeriesMan-9.0.0-0              ################################# [100%]
  ```

- 安装 MQSeriesJava

  ```
  rpm -ivh MQSeriesJava-9.0.0-0.x86_64.rpm
  ```

  若已有该版本或更高版本的jdk可以不安装

  安装详细：

  ```
  [root@localhost MQServer]# rpm -ivh MQSeriesJava-9.0.0-0.x86_64.rpm
  Preparing...                          ################################# [100%]
  Updating / installing...
     1:MQSeriesJava-9.0.0-0             ################################# [100%]
  ```

  

## 配置环境

- 修改mqm用户的密码

  安装的过程中 ，创建了一个mqm的用户和同名的组，该用户目前时锁定的，需要修改密码以解锁，需要root用户。

  ```
  passwd mqm
  ```

- 修改环境变量

  ```
  vim /etc/profile
  ```

  在末尾添加如下配置

  ```
  #set ibmmq environment
  MQ_HOME=/opt/mqm/bin
  PATH=$MQ_HOME:$PATH
  export PATH
  ```

  保存退出后，source /etc/profile使之生效。

  ```
  source /etc/profile
  ```

- `su - mqm`后使用`PATH=$PATH:/opt/mqm/samp/bin`，将 WebSphere MQ 样本程序目录添加到 PATH，就可以使用amqsput等命令。现在就可以正常使用了

  ```
  su - mqm
  PATH=$PATH:/opt/mqm/samp/bin
  ```

  