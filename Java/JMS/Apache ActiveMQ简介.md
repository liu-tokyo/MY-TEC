# Apache ActiveMQ简介

## 简介
Apache ActiveMQ是Apache软件基金会的一个开源项目，是一个基于消息的通信中间件。ActiveMQ是JMS的一个具体实现，支持JMS的两种消息模型。ActiveMQ使用AMQP协议集成多平台应用，使用STOMP协议通过websockets在Web应用程序之间交换消息，使用MQTT协议管理物联网设备。（参考ActiveMQ官网）

## JMS
JMS（Java Message Service），是一个基于消息的中间件服务，它是java的一个接口规范，而不是一个具体的软件或者库。它支持两种消息传送模型，点对点模型（Point to Point Model）和发布/订阅模型（Publisher/Subscriber Model）。（参考Java EE 8官方文档）

- 点对点模型

  ![JMS点对点模型](https://img-blog.csdnimg.cn/20190619111329244.png#pic_center)

- 发布/订阅模型

  ![JMS发布/订阅模型](https://img-blog.csdnimg.cn/20190619111347811.png#pic_center)

- JMS编程模型

  ![JMS编程模型](https://img-blog.csdnimg.cn/2019061911210792.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3N3ajYwODYwODU=,size_16,color_FFFFFF,t_70#pic_center)

## ActiveMQ环境配置、安装和运行
(参考ActiveMQ官方文档)
版本：apache-activemq-5.15.9-bin
操作系统：Windows 10

### 环境配置

需要 Java Runtime Environment (JRE) JRE 1.7及之后的版本。
需要配置JAVA_HOME环境变量。

### 安装

- 下载：下载页面 --> ActiveMQ 5 --> apache-activemq-5.15.9-bin.zip

- 解压后即可使用。解压后的目录结构是这样的：bin目录下是ActiveMQ的启动脚本；conf目录下是配置文件；data目录是日志文件和消息持久化存储的地方；lib目录下是activemq分功能的jar包，activemq-all-5.15.9.jar是全功能jar包。

  ![ActiveMQ解压目录](https://img-blog.csdnimg.cn/20190619165429974.PNG)

### 运行

- 启动cmd，cd到“<解压目录>\bin”目录下。执行命令activemq.bat start。或者可以将该目录加入环境变量，以后启动activemq服务时就不用先cd到该目录下了。
- 如果出现类似Apache ActiveMQ 5.11.1 (localhost, ID:ntbk11111-50816-1428933306116-0:1) started | org.apache.activemq.broker.BrokerService | main这样的输出，说明ActiveMQ启动成功。输出日志也可以在data目录下的文件activemq.log中查看。
- 启动前要确保这几个端口未被占用：61616，5672，61613，1883，61614，8161。
- 启动成功后不要关掉cmd窗口。
- 打开浏览器输入URLhttp://127.0.0.1:8161/admin，登录名和密码默认都是admin。可以在conf目录下的文件jetty-realm.properties中修改登录名和密码。
- 直接关掉cmd窗口就可以停止ActiveMQ服务。

## 配置一个ActiveMQ Broker

待续

##代码demo

请参考 ActiveMQ demo

## 参考资料

- Java EE 8官方文档
- ActiveMQ官网
- ActiveMQ官方文档

---
原文链接：https://blog.csdn.net/swj6086085/article/details/92806490