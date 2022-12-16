# IBM MQ 创建以及常见问题集锦

参考：

- https://www.modb.pro/db/167690

## 消息队列+发送队列+消息通道

接收通道名称与发送端的发送通道名称要一致，修改通道信息后要执行 start channle(chlname) 重启通道。

### 常用的MQ命令
66.0.42.240 用户 mqm/mqm  
88.0.52.40 mq队列服务器:mqm/1qaz2wsx  
二代：88.0.65.91 vlog/1qaz2wsx  
监控：/cnaps/maintain/bin  
管理台：admin/698825 240环境：admin/123456 698825  
通讯前置:88.0.65.94 pmts+1qaz2wsx  
查看队列属性 pmtsstat [disp] qrinfo  
关闭gaps: gapsshutdown.sh  
开启gaps: gapsboot.sh  

/* MQ创建 */  
显示队列管理器 dspmq  
进入队列管理器控制台 runmqsc 队列管理器名  
显示通道状态 dis chs(*)  
显示通道 dis chl(*)  

启动mq通道报错如下信息，为CCSID错误  
The local and remote queue managers do not agree on the next message sequence  
number. A message with sequence number 214 has been sent when sequence number  
使用命令重置CCSID可行reset chl(MBA.MBFEA) seqnum(819)  

STATUS(BINDING)  
主要是看mqm  

### 队列管理器相关指令

- 创建mq: crtmqm -q qmgrname
  -q是指创建缺省的队列管理器

- 删除mq: dltmqm qmgrname
- 启动mq: strmqm qmgrname
- 停止mq: endmqm qmgrname 受控停止  
  endmqm -i qmgrname 立即停止  
  endmqm -p qmgrname 强制停止  
- 显示队列管理器 dspmq -m qmgrname
- 运行mqseries命令（启动用于运行队列管理器MQSC命令的控制台（runmqsc）） runmqsc qmgrname

- 往队列中放消息 amqsput qname qmgrname
- 从队列中取消息 amqsget qname qmgrname
- 启动通道 runmqchl -c ChlName -m QmgrName
- 启动侦听 runmqlsr -t Type -p Port -m QMgrName
- 停止侦听 endmqlsr -m QmgrName

MQSeries命令

- 显示队列的所有属性 DISPLAY QUEUE （QName） [ALL]
- 显示队列的所选属性 DISPLAY QUEUE （QName）DESCR GET PUT
- DISPLAY QUEUE （QName）MAXDEPTH CURDEPT

- 显示队列管理器的所有属性 DISPLAY QMGR[ALL]  
  mq配置信息：根目录的mqs.ini(mq配置文件） qm.ini(队列管理器配置文件，它的属性仅影响某个队列管理器，在节点中的每个队列管理器都有一个qm.ini,所在目录 ./pmts/qmgrs/QMUMBFEA/qm.ini）
- 查看通道状态  
  DISPLAY CHSTATUS(*) CURRENT

- MQ属性
  队列管理器名称、队列、进程、名称列表、群集、认证信息对象 最长48个字符，通道名最长20个字符，队列名区分大小写，所有控制命令区分大小写
  USAGE(NORMAL/XMITQ):

- BOTHRESH 和 BOQNAME属性：

  当处理backout消息时，如4 中所言，如果某个消息是在同步点控制之下读取的，并且由于某种原因消息被回滚，消息描述符中的BackoutCount字段的值将被加1，你需要判断该数值，如果它大于某个阈值，你需要使用其它手段来处理该消息。在处理该消息的应用中，你可以将其与设定的阈值做比较，这时，阈值会被写死在程序中，为了提高其灵活性，你可以使用队列的BOTHRESH 和 BOQNAME属性。这样，你可以在例外处理中，利用MQINQ查询得到阈值的大小，如果超出，可以将消息转发到BOQNAME指定的队列中，继而对该队列进行相应的处理。这种方法大大增强了应用程序的灵活性。

DESCR:描述
MAXMSGL：消息大小
队列管理器MAXMSGL，默认值：4M，可以调整范围：32K - 100M；
通道MAXMSGL，默认值：4M，可以调整范围：0-队列管理器MAXMSGL；
队列MAXMSGL，默认值：4M，可以调整范围：0-队列管理器MAXMSGL；
使用alter命令，即可对其MAXMSGL进行修改。
三者之间的关系：
队列管理器MAXMSGL>=队列MAXMSGL；
通道MAXMSGL：
队列MAXMSGL：仅对本地队列和模型队列有效，表示队列中可以容纳的最大消息长度，这个属性的调整范围在各个平台上的上限是不一样的。通道在建立 的时候会有一个握手过程，双方会交换各自通道定义上的MAXMSGL，最后协商出通道使用的最大消息长度，一般会取双方定义中较小的那一个。
系统缺省对象：
系统缺省对象是一组每次创建队列管理器时自动创建的对象定义。您可以复制和修改这些对象定义中的任何一个，以在安装时用于应用程序。
缺省对象名具项SYSTEM.DEFAULT；例如，缺省本地队列是SYSTEM.DEFAULT.LOCAL.QUEUE，并且缺省接收方通道是SYSTEM.DEFAULT.RECEIVER。您无法重命名这些对象；这些名称的缺省对象是必需的。
当您定义对象时，从相应的缺省对象复制您不明确指定的任何属性。例如，如果您定义本地队列，则从缺省队列SYSTEM.DEFAULT.LOCAL.QUEUE 获取您未指定的那些属性。
请参阅附录1, "系统和缺省对象"以获取关系统缺省的更多信息。

---

win下使用 dspmq.exe

uninx/linux 下
\#su - mqm
\#dspmq

最近在配置MQ,记下了一些常用的MQ命令,如下:

创建队列管理器
crtmqm –q QMgrName

删除队列管理器
dltmqm QmgrName

启动队列管理器
strmqm QmgrName
如果是启动默认的队列管理器，可以不带其名字

停止队列管理器
endmqm QmgrName 受控停止
endmqm –i QmgrName 立即停止
endmqm –p QmgrName 强制停止

显示队列管理器
dspmq –m QmgrName

往队列中放消息
amqsput QName QmgrName
如果队列是默认队列管理器中的队列，可以不带其队列管理器的名字

从队列中取出消息
amqsget QName QmgrName
如果队列是默认队列管理器中的队列，可以不带其队列管理器的名字

启动通道
runmqchl –c ChlName –m QmgrName

启动侦听
runmqlsr –t TYPE –p PORT –m QMgrName

停止侦听
endmqlsr -m QmgrName

下面是在MQ环境中可以执行的MQ命令(即在runmqsc环境下可以敲的命令)

定义持久信队列
DEFINE QLOCAL（QNAME） DEFPSIST（YES） REPLACE
DEFPSIST：队列中消息持久性默认值。
NO 该队列上的消息在队列管理器重新启动时丢失
YES 该队列上的消息在队列管理器重新启动时保存了下来。
关于消息在队列中的保存时间：消息在队列的保存时间与个设置关：队列defpsist属性、消息Persistence持久性属性和消息Expiry消息到期时间属性，其中队列defpsist 属性是在创建队列时设置，消息Persistence和Expiry属性是应用程序往队列放入消息时指定。消息本身的Persistence值优先于队列defpsist值。Expiry指消息到期 时间，即经过指定的时间后，消息如果还没被取走，此消息将过期（无效。消息过期后，可能会自动从队列中删除（取决于不同操作系统的MQ实现。对于非持久性消息， 即使Expiry设为永不过期，重启队列管理器时，消息也将丢失。

定义队列允许的最大消息数目
MAXDEPTH
maxdepth：队列上允许的最大消息数；

设定队列管理器的持久信队列
ALTER QMGR DEADQ（QNAME）

定义本地队列
DEFINE QL（QNAME） REPLACE

定义别名队列
DEFINE QALIAS(QALIASNAME) TARGQ(QNAME)

远程队列定义
DEFINE QREMOTE（QRNAME） +
RNAME（AAA） RQMNAME（QMGRNAME） +
XMITQ（QTNAME）

定义模型队列
DEFINE QMODEL（QNAME） DEFTYPE（TEMPDYN）

定义本地传输队列
DEFINE QLOCAL(QTNAME) USAGE(XMITQ) DEFPSIST(YES) +
INITQ（SYSTEM.CHANNEL.INITQ）+
PROCESS(PROCESSNAME) REPLACE

创建进程定义
DEFINE PROCESS（PRONAME） +
DESCR（‘STRING’）+
APPLTYPE（WINDOWSNT）+
APPLICID（’ runmqchl -c SDR_TEST -m QM_ TEST’）
其中APPLTYPE的值可以是：CICS、UNIX、WINDOWS、WINDOWSNT等

创建发送方通道
DEFINE CHANNEL（SDRNAME） CHLTYPE（SDR）+
CONNAME（‘100.100.100.215(1418)’） XMITQ（QTNAME） REPLACE
其中CHLTYPE可以是：SDR、SVR、RCVR、RQSTR、CLNTCONN、SVRCONN、CLUSSDR和CLUSRCVR。

创建接收方通道
DEFINE CHANNEL（SDR_ TEST） CHLTYPE（RCVR） REPLACE

创建服务器连接通道
DEFINE CHANNEL（SVRCONNNAME） CHLTYPE（SVRCONN） REPLACE

显示队列的所有属性
DISPLAY QUEUE（QNAME） [ALL]

显示队列的所选属性
DISPLAY QUEUE（QNAME） DESCR GET PUT
DISPLAY QUEUE（QNAME）MAXDEPTH CURDEPTH

显示队列管理器的所有属性
DISPLAY QMGR [ALL]

显示进程定义
DISPLAY PROCESS（PRONAME）

更改属性
ALTER QMGR DESCR（‘NEW DESCRIPTION’）
ALTER QLOCAL（QNAME） PUT（DISABLED）
ALTER QALIAS（QNAME） TARGQ（TARGQNAME）

删除队列
DELETE QLOCAL（QNAME）
DELETE QREMOTE（QRNAME）

清除队列中的所有消息
CLEAR QLOCAL（QNAME）

## 高级配置命令

以下是一些高级配置的命令:

- amqmcert 配置SSL证书
- amqmdain 配置windows上的MQ服务
- crtmqcvx 转换数据
- dmpmqaut 转储对象权限管理
- dmpmqlog 转储日志管理
- dspmq 显示队列管理器
- dspmqaut 显示打开对象的权限
- dmpmqcap 显示处理程序容量和处理程序数
- dspmqcsv 显示命令服务器状态
- dspmqfls 显示文件名
- dspmqtrc 跟踪MQ输出(HP-UNIX LINUX Solaris)
- dspmqrtn 显示事务的详细信息
- endmqcsv 停止队列管理器上的命令服务器
- strmqcsv 启动队列管理器上的命令服务器
- endmqtrc 停止跟踪
- rcdmqimg 向日志写对象的映像
- rcmqobj 根据日志中的映像重新创建一个对象
- rsvmqtrn 提交或逆序恢复事务

## MQ 控制命令

MQ 控制命令的参考信息。

- addmqinf  
  添加 WebSphere MQ 配置信息（仅限于 Windows? 和 UNIX 平台）。
- amqccert
  检查不完整的证书链（仅限于 Windows）。
- amqmdain
  配置或控制 WebSphere MQ 服务控制（仅限于 Windows）。
- amqmfsck（文件系统检查）
  检查文件系统是否与 POSIX 标准一致并能够共享队列管理器数据以支持多实例队列管理器。
- amqtcert
  从 WebSphere MQ 5.3 或 5.3.1 迁移证书（仅限于 Windows）。
- crtmqcvx
  根据数据类型结构来创建数据转换代码。
- crtmqm
  创建队列管理器。
- dltmqm
  删除队列管理器。
- dmpmqaut
  转储一组 WebSphere MQ 对象类型和概要文件的当前权限列表。
- dmpmqlog
  显示并格式化 WebSphere MQ 系统日志的部分内容。
- dspmq
  显示关于队列管理器的信息。
- dspmqaut
  dspmqaut 显示特定 WebSphere MQ 对象的权限。
- dspmqcsv
  显示命令服务器的状态
- dspmqfls
  显示与 WebSphere MQ 对象相对应的文件名。
- dspmqinf
  显示 WebSphere MQ 配置信息（仅限于 Windows 和 Unix 平台）。
- dspmqrte
  确定消息通过队列管理器网络时采用的路由。
- dspmqtrc
  格式化并显示 WebSphere MQ 跟踪（仅限于 Unix 平台）。
- dspmqtrn
  显示不确定的事务。
- dspmqver
  显示 WebSphere MQ 版本和构建信息。
- endmqcsv
  为队列管理器停止命令服务器。
- endmqlsr
  结束队列管理器的所有侦听器进程。
- endmqdnm
  对某个队列停止 .NET 监视器（仅限于 Windows）。
- endmqm
  停止队列管理器或者切换到备用队列管理器。
- endmqtrc
  对所跟踪的某些或全部实体结束跟踪。
- migmbbrk
  migmbbrk 命令将发布/预订配置数据从 WebSphere Event Broker V6.0 或者 WebSphere Message Broker V6.0 或 V6.1 迁移到 WebSphere MQ V7.0.1 或更高版本。
- mqftapp
  启动文件传输应用程序的图形界面（仅限于 Windows 和 Linux x86 平台）。
- mqftrcv
  处理在服务器上使用 WebSphere MQ 文件传输应用程序接收到的文件（仅限于 Windows 和 Linux x86 平台）。
- mqftrcvc
  处理在客户机上接收到的文件（仅限于 Windows 和 Linux x86 平台）。
- mqftsnd
  使用 WebSphere MQ 文件传输应用程序从服务器发送文件（仅限于 Windows 和 Linux x86 平台）。
- mqftsndc
  使用 WebSphere MQ 文件传输应用程序从客户机发送文件（仅限于 Windows 和 Linux x86 平台）。
- rcdmqimg
  将一个对象或一组对象的映像写入日志，以便进行介质恢复。
- rcrmqobj
  根据日志中包含的一个或一组对象的映像来重新创建这些对象。
- rmvmqinf
  除去 WebSphere MQ 配置信息（仅限于 Windows 和 Unix 平台）。
- rsvmqtrn
  解决不确定的事务。
- runmqchi
  运行通道启动程序进程，以便自动启动通道。
- runmqchl
  启动发送方或请求方通道
- runmqdlq
  启动死信队列处理程序，以便监视和处理死信队列中的消息。
- runmqdnm
  使用 .NET 监视器来开始处理某个队列中的消息（仅限于 Windows）。
- runmqlsr
  运行侦听器进程，以便侦听各种通信协议的远程请求。
- runmqsc
  对队列管理器运行 WebSphere MQ 命令。
- runmqtmc
  在客户机上启动触发器监视器。
- runmqtrm
  在服务器上启动触发器监视器。
- setmqaut
  更改概要文件、对象或对象类的权限。可以对任意数目的主体或组授予权限或从中撤销权限。
- setmqcrl
  在 Active Directory 中管理 CRL（证书撤销列表）LDAP 定义（仅限于 Windows）。
- setmqprd
  登记 WebSphere MQ 生产许可证。
- setmqscp
  在 Active Directory 中发布客户机连接通道定义（仅限于 Windows）。
- strmqcfg
  启动 WebSphere MQ 资源管理器（仅限于 Windows 和 Linux x86 平台）。
- strmqcsv
  为队列管理器启动命令服务器。
- strmqm
  启动队列管理器或者使其准备好执行备用操作。
- strmqtrc

## 常用命令

创建队列管理器
crtmqm –q QMgrName
-q是指创建缺省的队列管理器
删除队列管理器
dltmqm QmgrName
启动队列管理器
strmqm QmgrName
如果是启动默认的队列管理器，可以不带其名字
停止队列管理器
endmqm QmgrName 受控停止
endmqm –i QmgrName 立即停止
endmqm –p QmgrName 强制停止
显示队列管理器
dspmq –m QmgrName
运行MQSeries命令
runmqsc QmgrName
如果是默认队列管理器，可以不带其名字

往队列中放消息
amqsput QName QmgrName
如果队列是默认队列管理器中的队列，可以不带其队列管理器的名字
从队列中取出消息
amqsget QName QmgrName
如果队列是默认队列管理器中的队列，可以不带其队列管理器的名字
启动通道
runmqchl –c ChlName –m QmgrName

启动侦听
runmqlsr –t TYPE –p PORT –m QmgrName

停止侦听
endmqlsr -m QmgrName

MQSeries命令
定义死信队列
DEFINE QLOCAL（QNAME） DEFPSIST（YES） REPLACE
设定队列管理器的死信队列
ALTER QMGR DEADQ（QNAME）
定义本地队列
DEFINE QL（QNAME） REPLACE
定义别名队列
DEFINE QALIAS(QALIASNAME) TARGQ(QNAME)
远程队列定义
DEFINE QREMOTE（QRNAME） +
RNAME（AAA） RQMNAME（QMGRNAME） +
XMITQ（QTNAME）
定义模型队列
DEFINE QMODEL（QNAME） DEFTYPE（TEMPDYN）
定义本地传输队列
DEFINE QLOCAL(QTNAME) USAGE(XMITQ) DEFPSIST(YES) +
INITQ（SYSTEM.CHANNEL.INITQ）+
PROCESS(PROCESSNAME) REPLACE

创建进程定义
DEFINE PROCESS（PRONAME） +
DESCR（‘STRING’）+
APPLTYPE（WINDOWSNT）+
APPLICID（’ runmqchl -c SDR_TEST -m QM_ TEST’）
其中APPLTYPE的值可以是：CICS、UNIX、WINDOWS、WINDOWSNT等

创建发送方通道
DEFINE CHANNEL（SDRNAME） CHLTYPE（SDR）+
CONNAME（‘100.100.100.215(1418)’） XMITQ（QTNAME） REPLACE
其中CHLTYPE可以是：SDR、SVR、RCVR、RQSTR、CLNTCONN、SVRCONN、CLUSSDR和CLUSRCVR。

创建接收方通道
DEFINE CHANNEL（SDR_ TEST） CHLTYPE（RCVR） REPLACE

创建服务器连接通道
DEFINE CHANNEL（SVRCONNNAME） CHLTYPE（SVRCONN） REPLACE

显示队列的所有属性
DISPLAY QUEUE（QNAME） [ALL]

显示队列的所选属性
DISPLAY QUEUE（QNAME） DESCR GET PUT
DISPLAY QUEUE（QNAME）MAXDEPTH CURDEPTH

显示队列管理器的所有属性
DISPLAY QMGR [ALL]

显示进程定义
DISPLAY PROCESS（PRONAME）

更改属性
ALTER QMGR DESCR（‘NEW DESCRIPTION’）
ALTER QLOCAL（QNAME） PUT（DISABLED）
ALTER QALIAS（QNAME） TARGQ（TARGQNAME）

删除队列
DELETE QLOCAL（QNAME）
DELETE QREMOTE（QRNAME）

清除队列中的所有消息
CLEAR QLOCAL（QNAME）

常用补充命令
显示队列管理器 dspmq
显示文件名 dspmqfls

启动本地队列管理器 strmqm
结束本地队列管理器 endmqm
启动通道启动进程 runmqchi/runmqchl

## MQ常用的一些管理操作
发布: 2008-12-15 10:49 | 作者: 胡伟红 | 来源: 本站原创 | 查看: 28次

觉得很有参考价值，跟大家分享。

### 1. 示例代码

安装好MQ之后，本身MQ会给你提供一些src的sample，有些是非常有用的，前提是你要装sample，可以将那些src的东东按照自己的需求修改后编译使用，exam（linux）：gcc -o destname srcname -lmqm

### 2. 资源导出工具

生产环境的QM由于某种需求，要更换服务器，又要保证在很短的时间内切换完成，那么怎么将原有定义的mq资源导出来，并形成ddl脚本呢？IBM提供了这么一个简单有效的导出工具（里面又各种平台的导出脚本），可以在ibm官方网站上搜索ms03，将它下载下来使用；注意要启动MQ的commandserver
strmqcsv QMname
exam（linux）：
tar -zxvf ms03.tar.Z
ls
make -f makefile.linux
ldaf s
ls *.TST
./saveqmgr.linux -m TIANJIN_QM -v '53'
VI SAVEQMGR.TST
导出命令saveqmgr.linux会在本目录下生成一个SAVEQMGR.TST的文件里面就是所有QM对象资源的定义。

### 3. 常用命令备忘
\#修改通道
alter CHANNEL(GUOJIA_SHENZHEN) CHLTYPE(SDR) CONNAME('10.12.131.12(1414)') XMITQ(GUOJIA_SHENZHEN_XMIT) DISCINT(0) TRPTYPE(TCP)
\#启动通道（runmqsc）
start chl(通道名称)
\#检察通道状态
dis chs(GUIZHOU_GY_GUOJIA)
\#允许远程客户端管理器连接
su - mqm
runmqsc QM
ALTER CHANNEL (SYSTEM.ADMIN.SVRCONN) CHLTYPE (SVRCONN) MCAUSER ("mqm")
\#启动一个QM
strmqm Heaven
\#启动一个QM的command server
strmqcsv Heaven
\#启动一个侦听
runmqlsr -m Heaven -t TCP -p 1415 &

\#停止队列管理器
endmqm QmgrName 受控停止

endmqm –i QmgrName 立即停止

endmqm –p QmgrName 强制停止

\# 显示QM的状态
dspmq –m QmgrName
\#channel不通的解决办法
runmqsc
stop chl(guojia_hainan)
resolve chl(guojia_hainan) action(commit)
reset chl(guojia_hainan) seqnum(1)

\# 创建缺省队列管理器，拥有1000个句柄， 使用线性循环日志，容量为 10240 × 4 K / 文件， 主文件30个，辅文件32个
crtmqm -t 5000 -h 1000 -lc -lf 10240 -lp 30 -ls 32 Heaven

\# 设置cpu个数为1
setmqcap 1

\# 从配置文件中读入初始化命令
runmqsc Heaven &lt; mqconfig.txt

\#创建完QM之后在/var/mqm/qmgrs/[Your Queue Manager Name]/qm.ini文件最后加入以下行：
Channels:
AdoptNewMCA=ALL
MaxChannels=1000
MaxActiveChannels=1000

\#如何清除MQ 队列管理器遗留的共享内存和信号量
解决方法：运行以下脚本或命令。
ipcs -m| grep mqm | awk '{print $2}'|xargs -i ipcrm -m {}
ipcs -s| grep mqm | awk '{print $2}'|xargs -i ipcrm -s {}

## mq经验总结
首先了解什么是mq？mq的作用是什么？
mq是通讯中间件。他的作用是省去开发人员开发通讯工具的时间，节省开发成本，提高开发效

率。
mq的使用，如何安装mq？
根据以往的经验，win版的mq比较容易安装，傻瓜式，一路next就可以。
aix版本的用smitty安装。
linux版本用rpm -ivh 安装
mq中一些名称的概念：
队列管理器：简单的说就是一个大容器的管理员，这个大容器里放了很多东西。
队列：大容器里的东西，存放消息的盒子。
通道：大容器和大容器之间，程序和容器之间进行通讯的途径。
mq是如何实现通讯的？
mq的通讯方式有两种，通俗的说就是mq之间进行通讯，开发的程序和mq之间的通讯。
mq之间进行通讯：通过发送接收通道建立tcp连接进行消息传输，称为server对server
开发的程序和mq之间的通讯：通过服务器连接通道进行传输，client对server
如何配置两台mq使之相互进行通讯？
首先要规划好两个队列管理器之间使用的ip和端口，假设我们使用
ip 端口
192.168.0.1 1414
192.168.0.2 1415
第一步 建立队列管理器
crtmqm -lc -lf 100 -lp 3 -ls 3 QM1
解释下：
-lc 是采用循环日志
-lf 是每块日志的大小，4k为单位的，100就是100*4k
-lp 是主逻辑日志的数量
-ls 是辅逻辑日志的数量
QM1 是队列管理器名称
第二步 启动队列管理器
strmqm QM1
第三步 定义队列管理器中的队列和通道等
先运行runmqsc QM1首先要保证运行该命令的用户属于mqm组
运行完后进入mq命令窗口
定义本地队列 def ql(QL1)
先解释什么是本地队列，然后解释命令的含义（以下同）
本地队列是存储信息的盒子，用户可以从本地队列里取消息，对方发送消息的目的地也是本地

队列。
def是 define的缩写，mq支持一些命令的缩写。
ql是queue local的缩写，表示本地队列，括号内是本地队列名
定义远程队列 def qr(QR1) rname(QL2) rqname(QM2) xmitq(QT1)
远程队列是相对于本地队列的，当用户希望往另一个队列管理器发消息的时候，配置好远程队

列，用户直接放消息到该队列就可以，mq会传输到另一方的本地队列中。
以上面的例子说明，当我们把消息放入该远程队列后，消息会传输到QM2队列管理器中的QL2队

列中。
qr queue remote的缩写
rname 指定的远程队列管理器上的队列名
rqname 远程队列管理器
xmitq 所要用的传输队列

定义传输队列 def ql(QT1) usage(xmitq) trigger trigtype(first) initq

(system.channel.initq) trigdata(QM1.QM2)
传输队列是传输的介质，消息是通过传输队列进行传输的。
usage 用途xmitq是传输队列
trigger 消息触发开关
trigtype 触发类型第一条消息触发
initq 初始队列
trigdata 触发数据

定义发送通道 def chl(QM1.QM2) chltype(sdr) conname('192.168.0.2(1415)') trptype

(tcp) xmitq(QT1)
发送通道就相当于建立一个tcp的连接
chl channel的缩写
chltype 通道类型sdr是发送通道
conname 连接名包括对方的ip和端口
trptype 通讯类型tcp通讯
xmitq 使用的传输队列

定义接受通道 def chl(QM2.QM1)
接收通道是被动的，只定义名字就可以。大家注意，接收通道的名字一定要和发送通道名一致

，他们是靠名字来匹配。

第四步 配置监听器
是对方mq管理器来探测，本地要给对方一个回应，监听器就是起这个作用的。
如果是5.3版本 只能在命令行里运行 runmqlsr -m QM1 -t tcp -p 1414
如果是6.0版本 可以runmqsc QM1里运行 def listener(LSR.QM1) trptype(tcp) port(1414)

control(qmgr)
解释下 trptype 监听类型
port 监听端口
control 监听控制，如果是qmgr则在队列管理器启动的时候监听也自动启动。

第五步 配置另外一个队列管理器
简单的说一下，和上面的差不多，只不过名字不一样。大家自己尝试下：）
写的手累了,下次补充!

继续
上面我们说完了如何建队列管理器，接下来我们说说建完以后如何测试两边是不是正常传输，
我们可以用命令行方式向远程队列中放入测试消息，以aix为例，
用/usr/mqm/samp/bin/amqsput命令就可以放入消息，格式为：
amqsput QR1 QM1
解释下：QR1是你要放入的队列名，QM1是你要放入的队列管理器名。
输入以上命令后就可以写入消息了，一下回车就是发送一条消息，两次回车就是退出。

用/usr/mqm/samp/bin/amqsget命令就可以取消息，格式为
amqsget QL2 QM2
解释下：QL1是你要取消息的队列名，QM2是你要取消息的队列管理器名。
输入完命令后就会显示所有消息。
不过要注意一点，命令行方式的输入和取消息有字节限制。

如果只是简单的浏览一下消息可以使用
/usr/mqm/samp/bin/amqsbcg命令
格式和取消息一样，但是该命令不会把消息取出来，运行完该命令后消息还是保存在队列中。

正常情况下消息的传输流程（只说正常的，排错一会再说）
QR1 -> QT1 -> QL2
消息被放入到远程队列中，远程队列通过传输队列传输，最后传输到QM2中的本地队列。

出现问题，我们怎么办？
1 不能放入消息。
一般这种情况应该大部分是远程队列中的传输队列那个参数配置的不正确。
还有可能是队列的允许放入这个参数设置成了禁止。基本上就这两种情况。

2 发送通道和接收通道的状态不是running
首先说明，如果长时间没有消息传输，通道的状态会变成不活动状态，这是正常现象。
如果你手动启动通道后，通道状态还不是running，那先查看错误日志（两边的队列管理器都要查看）
/var/mqm/qmgrs/QM1/errors中的错误日志，通常编号01的日志是最新日志。
常见情况是网络不通导致的通道不通！所以首先要保证网络是正常的，我们可以同过telnet对方的IP加监听端口的方法来查看是不是正常。
telnet 192.168.0.2 1415

再有的情况是两边的配置属性有问题，如两边发送和接收通道名不一致，发送通道的连接名配置错误，发送通道中的传输队列配置错误。
我们也可以执行mq中的一个命令来查看通道是不是正常
runmqsc QM1
ping chl(QM1.QM2)
ping操作来查看两边的通道是不是正常，如果正常会返回ping完成。

3 放入的消息没有到QM2的队列中
注意：消息一定要放入远程队列中，如果放入传输队列中消息会被放入死信队列中。（上面忘记定义死信队列了，晕）
再有看看远程队列中的属性是不是配置错误，如rname，rqname，xmitq等属性。
也有可能是发送接收队列的消息序列号不一致。如果不一致做一下reset操作。
还有可能是上一批消息没有提交。可以做一个resolve操作。
也是要先看错误日志

4 消息到达QM2队列QL2中，但是取不出来
QL2的允许取出属性是不是被禁止了。

这样再查看以下QM2到QM1的传输是不是正常。都正常就OK了。

### 一、MQ命令集合
MQ命令集合有三种命令：控制命令、MQSC（MQ脚本命令）和PCF（Programmable Command Formats，可编程的命令格式）。

### 二、控制命令
控制命令：用于管理 WebSphere MQ的系统配置,包括队列管理器、侦听器、通道、日志的管理。
例如：创建队列管理器（crtmqm）,启动队列管理器（strmqm）,启动用于运行队列管理器MQSC命令的控制台（runmqsc）、运行通道（runmqchl）
对于Linux，WebSphere MQ 控制命令都从 shell输入和执行。

控制命令列表如下所示：
addmqinf（添加配置信息）
amqccert（检查证书链）
amqmdain（WebSphere MQ 服务控制）
amqmfsck（文件系统检查）
amqtcert（传送证书）
crtmqcvx（数据转换）
crtmqm（创建队列管理器）
dltmqm（删除队列管理器）
dmpmqaut（转储权限）
dmpmqlog（转储日志）
dspmq（显示队列管理器）
dspmqaut（显示权限）
dspmqcsv（显示命令服务器）
dspmqfls（显示文件）
dspmqinf（显示配置信息）
dspmqrte（WebSphere MQ 显示路由应用程序）
dspmqtrc（显示格式化的跟踪输出）
dspmqtrn（显示事务）
dspmqver（显示版本信息）
endmqcsv（结束命令服务器）
endmqlsr（结束侦听器）
endmqdnm（停止 .NET 监视器）
endmqm（结束队列管理器）
endmqtrc（结束跟踪）
migmbbrk（迁移发布/预订信息）
mqftapp（运行文件传输应用程序 GUI）
mqftrcv（在服务器上接收文件）
mqftrcvc（在客户机上接收文件）
mqftsnd（从服务器发送文件）
mqftsndc（从客户机发送文件）
rcdmqimg（记录介质映像）
rcrmqobj（重新创建对象）
rmvmqinf（除去配置信息）
rsvmqtrn（解决事务）
runmqchi（运行通道启动程序）
runmqchl（运行通道）
runmqdlq（运行死信队列处理程序）
runmqdnm（运行 .NET 监视器）
runmqlsr（运行侦听器）
runmqsc（运行 MQSC 命令）
runmqtmc（启动客户机触发器监视器）
runmqtrm（启动触发器监视器）
setmqaut（授予或撤销权限）
setmqcrl（设置证书撤销列表 (CRL) LDAP 服务器定义）
setmqprd（登记生产许可证）
setmqscp（设置服务连接点）
strmqcfg（启动 WebSphere MQ 资源管理器）
strmqcsv（启动命令服务器）
strmqm（启动队列管理器）
strmqtrc（启动跟踪）

### 三、MQSC

MQSC全称为MQ Script Command，MQ脚本命令
MQSC用于管理队列管理器对象，包括队列管理器本身、通道、队列、侦听器和进程定义。
对于Linux，若要执行MQSC,则需要启动脚本命令控制台；启动方式：在shell执行控制命令runmqsc
WebSphere MQ V7.0 的MQSC列表如下所示：
ALTER AUTHINFO
ALTER BUFFPOOL
ALTER CFSTRUCT
ALTER CHANNEL
ALTER LISTENER
ALTER NAMELIST
ALTER PROCESS
ALTER PSID
ALTER QMGR
ALTER 队列
ALTER SECURITY
ALTER SERVICE
ALTER STGCLASS
ALTER SUB
ALTER TOPIC
ALTER TRACE
ARCHIVE LOG
BACKUP CFSTRUCT
CLEAR QLOCAL
CLEAR TOPICSTR
DEFINE AUTHINFO
DEFINE BUFFPOOL
DEFINE CFSTRUCT
DEFINE CHANNEL
DEFINE LISTENER
DEFINE LOG
DEFINE MAXSMSGS
DEFINE NAMELIST
DEFINE PROCESS
DEFINE PSID
DEFINE QUEUES
DEFINE SERVICE
DEFINE STGCLASS
DEFINE SUB
DEFINE TOPIC
DELETE AUTHINFO
DELETE BUFFPOOL
DELETE CFSTRUCT
DELETE CHANNEL
DELETE LISTENER
DELETE NAMELIST
DELETE PROCESS
DELETE PSID
DELETE QUEUES
DELETE SERVICE
DELETE SUB
DELETE STGCLASS
DELETE TOPIC
DISPLAY ARCHIVE
DISPLAY AUTHINFO
DISPLAY CFSTATUS
DISPLAY CFSTRUCT
DISPLAY CHANNEL
DISPLAY CHINIT
DISPLAY CHSTATUS
DISPLAY CLUSQMGR
DISPLAY CMDSERV
DISPLAY CONN
DISPLAY GROUP
DISPLAY LISTENER
DISPLAY LOG
DISPLAY LSSTATUS
DISPLAY MAXSMSGS
DISPLAY NAMELIST
DISPLAY PROCESS
DISPLAY PUBSUB
DISPLAY QMGR
DISPLAY QMSTATUS
DISPLAY QSTATUS
DISPLAY QUEUE
DISPLAY SBSTATUS
DISPLAY SECURITY
DISPLAY SERVICE
DISPLAY STGCLASS
DISPLAY SUB
DISPLAY SVSTATUS
DISPLAY SYSTEM
DISPLAY THREAD
DISPLAY TOPIC
DISPLAY TPSTATUS
DISPLAY TRACE
DISPLAY USAGE
MOVE QLOCAL
PING CHANNEL
PING QMGR
RECOVER BSDS
RECOVER CFSTRUCT
REFRESH CLUSTER
REFRESH QMGR
REFRESH SECURITY
RESET CHANNEL
RESET CLUSTER
RESET QMGR
RESET QSTATS
RESET TPIPE
RESOLVE CHANNEL
RESOLVE INDOUBT
RESUME QMGR
RVERIFY SECURITY
SET ARCHIVE
SET LOG
SET SYSTEM
START CHANNEL
START CHINIT
START CMDSERV
START LISTENER
START QMGR
START SERVICE
START TRACE
STOP CHANNEL
STOP CHINIT
STOP CMDSERV
STOP CONN
STOP LISTENER
STOP QMGR
STOP SERVICE
STOP TRACE
SUSPEND QMGR

### 四、PCF

PCF,全称为Programmable Command Formats，可编程的命令格式。
WebSphere MQ PCF用于MQ的系统管理编程，应用程序使用PCF实现MQSC的功能，使得MQ管理任务可编写到应用程序中，PCF 命令和MQSC 命令具有相同的命令集；例如，PCF使得可以在程序中创建队列和进程定义和更改队列管理器。
下面的Java代码描述MQ客户机端程序通过PCF更改远程服务器上所以的队列的名称，并打印到控制台的过程。
Java代码 收藏代码
public static void main (String [] args){
PCFMessageAgent agent;
PCFMessage request;
PCFMessage [] responses;
String [] names;


agent = new PCFMessageAgent ("192.168.222.132",1414,"JAVA.CHANNEL");
System.out.println ("Connected.");

// Build the PCF request

request = new PCFMessage (CMQCFC.MQCMD_INQUIRE_Q_NAMES);
request.addParameter (CMQC.MQCA_Q_NAME, "*");
request.addParameter (CMQC.MQIA_Q_TYPE, CMQC.MQQT_ALL);

System.out.print ("Sending PCF request... ");

// Use the agent to send the request

responses = agent.send (request);
System.out.println ("Received reply.");

// Extract the MQCACF_Q_NAMES parameter from the response

names = (String []) responses [0].getParameterValue (CMQCFC.MQCACF_Q_NAMES);

// Display the results

System.out.println ("Queue names:");

for (int i = 0; i < names.length; i++)
{
System.out.println ("\t" + names [i]);
}

// Disconnect the agent

System.out.print ("Disconnecting... ");
agent.disconnect ();
System.out.println ("Done.");
}

MQAI,全称为MQ Administration Interface，MQ管理接口
MQAI：除了PCF的系统管理编程接口之外，WebSphere MQ还提供另外一种系统管理编程接口，即：MQ管理接口（MQ Administration Interface，简称为MQAI），MQAI是MQ 提供的一种简化的、实现发送和接收PCF命令消息和回复消息的接口，MQAI通过使用数据包（Data Bags）来处理对象的属性，这样比直接使用PCF更简单。
MQAI的底层工作机制同PCF一样，也是通过发送PCF命令消息到MQ命令服务器队列，从而被命令服务器解释执行，并等待回复消息来管理WebSphere MQ，如图所示：



MQAI是PCF的易用版本。

有关PCF和MQAI的详细信息，请参考MQ的帮助文档和IBM工程师编写的MQ系统管理编程概述一文
http://www.ibm.com/developerworks/cn/websphere/library/techarticles/loulijun/0402_mqsysm/mqsysm.html

### 五、其他命令

例如amqsput（向队列放入消息）、amqsget（从队列取消息）为MQ的内置样本程序。

---

## MQ系列之二、维护MQ

### 一、命令是开始

维护MQ，就必须要了解MQ的作用和其相关命令。
MQ作为通信中间件，其通讯方式有两种：MQ之间进行通讯、开发的程序和mq之间的通讯。
MQ之间进行通讯：通过发送接收通道建立tcp连接进行消息传输，称为server对server
开发的程序和MQ之间的通讯：通过服务器连接通道进行传输，client对server。
MQ有很多有用的命令，以下是一些常用命令：
$dspmq /*列出现有队列*/
$strmqm TEST /*启动队列管理器*/
$runmqsc TEST /*定义管理器资源*/
end /*退出*/
$runmqlsr -t tcp -m TEST -p 1442 /*启动侦听器*/
$endmqm TEST /*停止MQ服务*/
$display q(QTEST) all /*显示队列的所有属性*/
$display q(QTEST） /*显示队列的所有属性*/
$display q(QTEST） maxdepth curdepth /*显示队列的所选属性*/
$dis qmgr all /*显示队列管理器的所有属性*/
MQ有很强大的命令输入帮助，当你对命令不熟悉或是不确定时，可以输入部分命令来查询；MQ还支持很多的简写，熟悉之后可以很大程度的提高效率。

### 二、故障是历练

MQ作为通信中间件，可能涉及不止一个平台，故障也就多种多样。无论是什么维护，学会看日志总是一个好的开始，MQ的错误日志存放在$mq/qmgrs/$qm_name/errors下，日志是轮流写入的，要注意看对日志。mq的日志比较详尽，具体怎么看我就不累述了。下面列举几个常见问题的恢复方法：
1、无法将消息数据放到队列中
一般这种情况应该大部分是远程队列中的传输队列那个参数配置的不正确，还有可能是队列的允许放入这个参数设置成了禁止或是队列堵塞了。
a、检查队列属性的CURDEPTH达到MAXDEPTH值，如果CURDEPTH达到MAXDEPTH，表明队列深度已经达到最大值，这样就导致新的消息数据无法放入。
$display queue（q_name）maxdepth curdepth
b、检查队列属性PUT是否为ENABLED，如果不是则说明队列不允许放入消息，可以用下面命令修改
$alter qlocal(q_name) get (ENABLED)
c、检查队列属性MAXMSGL的值（默认4M），如果准备放入消息的大小大于队列这个值，则需要修改队列的MAXMSGL属性值，并将该值相应扩大。
注意：修改该属性值时也要同时修改远端对应的队列的值以及对应的通道的值，否则即便是消息能放入该对列，也无法传输到远端对应的队列中。

2、发送通道和接收通道的状态不是running
a、首先检查网络是否正常
我们可以执行mq中的一个命令来查看通道是不是正常
runmqsc QM1
ping chl(QM1.QM2)
ping操作来查看两边的通道是不是正常，如果正常会返回ping完成。
b、停止通道，然后复位两端通道 RESET CHANNEL(channel_name) [ SEQNUM( 1 | integer) ]。复位完成后，启动通道。
c、如果还是不行，可以执行RESOLVE CHANNEL(channel_name) ACTION （BACKOUT）进行回退，ACTION有两个参数 BACKOUT和Commit,BACKOUT 将把可疑消息恢复到传输队列,而Commit将丢弃可疑消息。通常选择使用BACKOUT。

3、MQ异常终止产生残留的信号灯和共享内存处理方法。
a、MQ异常停止会产生残留的信号灯和共享内存，导致MQ无法正常启动和开启队列管理器。
$dspmq
QMNAME(TEST) STATUS(Quiescing)
$strmqm TEST
WebSphere MQ queue manager 'TEST' ending.
b、可以执行$ipcs -a | grep mq列出相应的信号灯和内存
如果有，使用ipcrm命令清除：
ipcrm -s <semphore id>
ipcrm -m <shared memory id >
c、再次启动MQ队列管理器即可正常。
MQ队列管理器搭建之(一)
多应用单MQ使用场景

如上图所示，MQ独立安装，或者与其中一个应用同处一机。Application1与Application2要进行通信，但因为跨系统，所以引入中间件来实现需求。

Application1需要连接MQ，并将消息放入队列Queue中，Application2同样连接MQ，监听在Queue队列上，一旦发现有消息进入则取出该消息进行处理。
下面将给出创建队列管理器和队列的示例:
定义队列管理器名称为Qm1，本地队列名称为Queue，服务器连接通道CHAN_SERVER_CON，监听端口为1414，死性队列QDEAD
搭建MQ队列可以使用图形用户界面也可以使用命令进行，此处使用命令进行。
1.创建MQ队列管理器，使用mqm用户登录MQ所在机器
mqm@localhos ~>$crtmqm Qm1
2.启动Qm1队列管理器
mqm@localhos ~>$strmqm Qm1
3.进入Qm1命令行
mqm@localhos ~>$runmqsc Qm1
4.定义一个本地队列Queue
DEFINE QLOCAL ('Queue') DEFPSIST (YES) MAXDEPTH(100) REPLACE
'Queue'为队列名称，至于使用单引号的原因是，如果在shell脚本中不加单引号的话，最后创建出来的会变成大写QUEUE.。
DEFPSIST(YES)表示该队列为持久化队列，MAXDEPTH(100)代表该队列的最大深度为100，如果消息超过了100的话，则会被放入死性队列。
5.在定义一个死性队列QDEAD
DEFINE QLOCAL ('QDEAD') DEFPSIST (YES) MAXDEPTH(100) REPLACE
6..给Qm1设置指定的死性队列，当消息无法到达指定的Queue中时，会被放入死性队列QDEAD
ALTER QMGR DEADQ(‘QDEAD’)
7.定义服务器连接通道CHAN_SERVER_CON，该通道的用途是供应用程序连接的，应用程序通过服务器连接通道从而连接MQ。
DEFINE CHANNEL(‘CHAN_SERVER_CON’) CHLTYPE(SVRCONN) REPLACE
8.定义监听器LISTENER.TCP，该端口1414应用程序连接时需要指定。
DEFINE LISTENER('LISTENER.TCP') TRPTYPE(TCP) CONTROL(QMGR) PORT(1414) REPLACE
9.启动监听器LISTENER.TCP
START LISTENER('LISTENER.TCP')

到此为止这个需求中的MQ队列管理器已经创建完毕了。如果在创建过程中出现错误，或者想停止队列管理器，或者想删除重新创建，则执行下述命令：
1.删除前先停止队列管理器
ctrl+c可以冲命令行跳出，或者输入end回车也可以。
mqm@localhos ~>$endmqm Qm1 停止队列管理器
mqm@localhos ~>$dspmq 查看当前队列管理器的执行状态，当队列管理器状态变为Ended normally时才能删除
mqm@localhos ~>$dltmqm Qm1 删除队列管理器，它会级联删除该队列管理器中的队列和监听器等等。

2.至于java如何与MQ通信，如何连接MQ队列此处不做过多的阐述了！

## 第三章 MQ队列管理器搭建之(二)

MQ级联方式使用场景


### 使用场景：

如上图所示，Application1与Application2要进行通信或者消息互换，使用MQ中间件作为中介。上图中，Application1与Application2通信不进行直接连接，而是通过与MQ通信从而实现二者的通信。图中两个MQ的信息如上描述。其中RemoteQueue为远程队列，该队列指定了目标端对应的队列为Queue，并且该远程队列指定了传输所使用的传输队列尾TransQueue；而此传输队列TransQueue与发送通道CHAN_QMGR1_TO_QMGR2相关联，并且可以在该传输队列上设置触发器。Application1连接通过CHAN_SERVER_CON服务器连接通道连接MQ，将消息放入RemoteQueue远程队列，MQ的远程队列收到放入的消息后将消息放入与之关联的传输队列，传输队列中有消息后么，触发器会产生触发消息，通过发送通道将该条消息发送到目标端。此处需要注意的是，在发送通道中会指定目标端的ip和端口号，并且发送通道的名称需要与目标端接收通道的名称一致，即一个发送通道要对应目标端的一个接收通道，并且名称相同。如此消息便发到了MQ2的接收通道中，MQ2拿到消息后，该消息描述了它的目标点是Queue队列，则MQ2会将消息放入MQ2的Queue本地队列中去。

MQ级联方式的搭建，左边的MQ队列管理器名称叫做MQ1，右边的叫做MQ2：
MQ1的搭建：
1.创建队列管理器MQ1。(使用mqm用户连接MQ所在的机器，dspmq查看队列的状态，查看MQ1是否已经创建，如果已经创建则更换名称，或者删掉重建)
mqm@localhos ~>$crtmqm MQ1
2.启动队列管理器
mqm@localhos ~>$strmqmMQ1
3.进入MQ1的命令行模式
mqm@localhos ~>$runmqsc MQ1
4.定义本地队列Queue，下面的含义不再赘述参见本章的上一节。
DEFINE QLOCAL ('Queue') DEFPSIST (YES) MAXDEPTH(100) REPLACE
5.定义一个远程队列 RemoteQueue
DEFINE QREMOTE('RemoteQueue') RNAME('Queue') RQMNAME('MQ2') XMITQ('TransQueue')
RNAME('Queue') 指定了对应的目标端的队列是Queue，RQMNAME('MQ2')指定了目标端的队列管理器名称为MQ2， XMITQ('TransQueue')指定了该远程队列关联的传输队列为‘TransQueue’。
6.定义一个传输队列TransQueue
DEFINE QLOCAL('TransQueue') usage(XMITQ) DEFPSIST(YES) INITQ(SYSTEM.CHANNEL.INITQ) TRIGDATA('CHAN_QMGR1_TO_QMGR2') TRIGTYPE(FIRST) TRIGGER REPLACE
与本地队列不同的是 usage(XMITQ) ，它指定了该队列为传输队列。DEFPSIST(YES)代表队列持久化， INITQ(SYSTEM.CHANNEL.INITQ) TRIGDATA('CHAN_QMGR1_TO_QMGR2') TRIGTYPE(FIRST) TRIGGER 与设置触发器相关，初始化队列为SYSTEM.CHANNEL.INITQ，被触发的通道为'CHAN_QMGR1_TO_QMGR2'，触发方式为First，每个消息到达时产生触发事件。
7.定义一个发送通道CHAN_QMGR1_TO_QMGR2
DEFINE CHANNEL('CHAN_QMGR1_TO_QMGR2') CHLTYPE(SDR) CONNAME('192.168.xx.xx(1414)') XMITQ('TransQueue')
CONNAME指定了目标端ip和端口号，CHLTYPE(SDR)指定了通道的类型为发送，XMITQ指定了传输队列的名称。
8.定义一个接收通道CHAN_QMGR2_TO_QMGR1，该名称与发送端的发送通道名称一致。
DEFINE CHANNEL(CHAN_QMGR2_TO_QMGR1) CHLTYPE(RCVR)
9.定义一个服务器连接通道
DEFINE CHANNEL(CHAN_SERVER_CON) CHLTYPE(SVRCONN) REPLACE
10.在定义一个死性队列QDEAD
DEFINE QLOCAL ('QDEAD') DEFPSIST (YES) MAXDEPTH(100) REPLACE
11.给MQ1设置指定的死性队列，当消息无法到达指定的Queue中时，会被放入死性队列QDEAD
ALTER QMGR DEADQ(‘QDEAD’)
12.定义监听器LISTENER.TCP，该端口1414应用程序连接时需要指定。
DEFINE LISTENER('LISTENER.TCP') TRPTYPE(TCP) CONTROL(QMGR) PORT(4141) REPLACE
13.启动监听器LISTENER.TCP
START LISTENER('LISTENER.TCP')

如此发送的一方就搭建完了。

### MQ2的搭建：

1.创建队列管理器MQ2。(使用mqm用户连接MQ所在的机器，dspmq查看队列的状态，查看MQ2是否已经创建，如果已经创建则更换名称，或者删掉重建)
mqm@localhos ~>$crtmqm MQ2
2.启动队列管理器
mqm@localhos ~>$strmqm MQ2
3.进入MQ2的命令行模式
mqm@localhos ~>$runmqsc MQ2
4.定义本地队列Queue，下面的含义不再赘述参见本章的上一节。
DEFINE QLOCAL ('Queue') DEFPSIST (YES) MAXDEPTH(100) REPLACE
5.定义一个远程队列RemoteQueue
DEFINE QREMOTE('RemoteQueue') RNAME('Queue') RQMNAME('MQ1') XMITQ('TransQueue')
RNAME('Queue') 指定了对应的目标端的队列是Queue，RQMNAME('MQ1')指定了目标端的队列管理器名称为MQ1， XMITQ('TransQueue')指定了该远程队列关联的传输队列为‘TransQueue’。
6.定义一个传输队列TransQueue
DEFINE QLOCAL('TransQueue') usage(XMITQ) DEFPSIST(YES) INITQ(SYSTEM.CHANNEL.INITQ) TRIGDATA('CHAN_QMGR2_TO_QMGR1') TRIGTYPE(FIRST) TRIGGER REPLACE
与本地队列不同的是 usage(XMITQ) ，它指定了该队列为传输队列。DEFPSIST(YES)代表队列持久化， INITQ(SYSTEM.CHANNEL.INITQ) TRIGDATA('CHAN_QMGR2_TO_QMGR1') TRIGTYPE(FIRST) TRIGGER 与设置触发器相关，初始化队列为SYSTEM.CHANNEL.INITQ，被触发的通道为'CHAN_QMGR2_TO_QMGR1'，触发方式为First，每个消息到达时产生触发事件。
7.定义一个发送通道CHAN_QMGR2_TO_QMGR1
DEFINE CHANNEL('CHAN_QMGR2_TO_QMGR1') CHLTYPE(SDR) CONNAME('192.168.xx.xx(4141)') XMITQ('TransQueue')
CONNAME指定了目标端ip和端口号，CHLTYPE(SDR)指定了通道的类型为发送，XMITQ指定了传输队列的名称。
8.定义一个接收通道CHAN_QMGR1_TO_QMGR2，该名称与发送端的发送通道名称一致。
DEFINE CHANNEL(CHAN_QMGR1_TO_QMGR2) CHLTYPE(RCVR)
9.定义一个服务器连接通道
DEFINE CHANNEL(CHAN_SERVER_CON) CHLTYPE(SVRCONN) REPLACE
10.在定义一个死性队列QDEAD
DEFINE QLOCAL ('QDEAD') DEFPSIST (YES) MAXDEPTH(100) REPLACE
11.给MQ1设置指定的死性队列，当消息无法到达指定的Queue中时，会被放入死性队列QDEAD
ALTER QMGR DEADQ(‘QDEAD’)
12.定义监听器LISTENER.TCP，该端口1414应用程序连接时需要指定。
DEFINE LISTENER('LISTENER.TCP') TRPTYPE(TCP) CONTROL(QMGR) PORT(1414) REPLACE
13.启动监听器LISTENER.TCP
START LISTENER('LISTENER.TCP')
这样就完成了MQ1与MQ2的互相通信了，需要注意的是，此处两个队列管理器的监听端口不能一样。
小结：此种连接方式适用于小批量消息的发送及接收，即单个队列管理器便能满足需求。此外这种方式是将队列管理器与远程队列进行了绑定，不便于扩展。
下一节内容中将会提到MQ集群的搭建，以及网关队列管理器的搭建。

## 第三章 MQ队列管理器搭建之(三)

MQ集群及网关队列管理器的搭建

描述：
如上图所示，为MQ的集群搭建部署图。CLUSTERA、CLUSTERB分别是两个集群，其中Qm1-Qm3、GateWayA为CLUSTERA集群中的队列管理器；Qm1-Qm3、GateWayB是CLUSTERB集群中的队列管理器。GateWayA与GateWayB负责网络路由和消息分发，使用集群的方式可以达到负载均衡的目的，除此之外还能提高MQ使用的稳定性。同一个集群中除网关队列管理器外的任意队列管理器因故关闭或停止工作后，其他的队列管理器可以接管它的工作从而保证业务应用的正常运行。

使用场景：
为了提高分布式应用异步消息传输及处理的效率，从中间件的角度来优化，除此之外要保证消息传输过程的可靠性。Application1通过网关队列管理器A将消息发送到网关队列管理器GateWayB中，GateWayB收到消息后根据自身负载均衡算法将消息分发到不同的队列管理器对应的队列中，Application2使用监听的方式监听于Qm1-Qm3的队列上，一旦有消息被分发到各自的队列时，应用程序则会获取消息进行处理。

集群及网关队列管理器的搭建：(左边的未A机器、右边的为B机器)
Qm1、Qm2、Qm3、GateWayA、GateWayB为队列管理。
Queue为本地队列，分别在A中的Qm1、Qm2、Qm3和B中的Qm1、Qm2、Qm3创建。
TransQueue为传输队列，分别在GateWayA和GateWayB中创建。
CHAN_GATEWAYA_TO_GATEWAYB、CHAN_GATEWAYB_TO_GATEWAYA为发送、接收通道，在GateWayA中创建。
CHAN_GATEWAYB_TO_GATEWAYA、CHAN_GATEWAYA_TO_GATEWAYB为发送、接收通道，在GateWayB中创建。
ANY.TO.CLUSTERB为远程队列，在GateWayA中创建。
ANY.TO.CLUSTERA为远程队列，在GateWayB中创建。
RemoteQueue为远程队列，分别在GateWayA、GateWayB中创建。
CLUSTERA、CLUSTERB分别为A、B机器的MQ集群名称。
QEDAD分别在A、B机器的所有队列管理器中创建。

1.分别在A、B机器上创建队列管理器Qm1、Qm2、Qm3。
--参见《第三章 MQ队列管理器搭建之(二)》

2.创建网关队列管理器GateWayA、GateWayB。
--网关队列管理器的创建与Qm1-Qm3方式相同，不同的是网关队列管理器中需要创建一个特殊的远程队列，此外其他的远程队列指向的目标队列管理器名称为目标网关队列管理器中的远程队列名。
GateWayA中
--DEFINE QREMOTE('RemoteQueue') RNAME('Queue') RQMNAME('ANY.TO.CLUSTERB') XMITQ('TransQueue')
--DEFINE QREMOTE(ANY.TO.CLUSTERA) RNAME('') RQMNAME('')
GateWayB中
--DEFINE QREMOTE('RemoteQueue') RNAME('Queue') RQMNAME('ANY.TO.CLUSTERA') XMITQ('TransQueue')
--DEFINE QREMOTE(ANY.TO.CLUSTERB) RNAME('') RQMNAME('')

3创建Queue、RemoteQueueCHAN_GATEWAYA_TO_GATEWAYB、CHAN_GATEWAYB_TO_GATEWAYA及监听端口。
--参见《第三章 MQ队列管理器搭建之(二)》

4.创建本地队列Queue，创建中需要指定集群名称。
A机器的Qm1-Qm3
--DEFINE QLOCAL ('Queue') CLUSTER(CLUSTERA) DEFPSIST (YES) MAXDEPTH(100) REPLACE
B机器的Qm1-Qm3
--DEFINE QLOCAL ('Queue') CLUSTER(CLUSTERB) DEFPSIST (YES) MAXDEPTH(100) REPLACE
5.定义完全存储仓库(Qm1与Qm3为完全存储仓库、网关队列管理器和Qm2为部分存储仓库)
--runmqsc Qm1
--ALTER QMGR REPOS(CLUSTERA)
--runmqsc Qm3
--ALTER QMGR REPOS(CLUSTERA)
6.定义集群发送通道与集群接收通道
A:
GateWayA:
--DEFINE CHANNEL(TO.GateWayA) CHLTYPE(CLUSRCVR) TRPTYPE(TCP) CONNAME('192.168.x.x(1414)') CLUSTER (CLUSTERA)
--DEFINE CHANNEL(TO.Qm1) CHLTYPE(CLUSSDR) TRPTYPE(TCP) CONNAME('192.168.x.x(2414)') CLUSTER (CLUSTERA)
Qm1:
--DEFINE CHANNEL(TO.Qm1) CHLTYPE(CLUSRCVR) TRPTYPE(TCP) CONNAME('192.168.x.x(2414)') CLUSTER (CLUSTERA)
--DEFINE CHANNEL(TO.Qm3) CHLTYPE(CLUSSDR) TRPTYPE(TCP) CONNAME('192.168.x.x(4414)') CLUSTER (CLUSTERA)
Qm2:
--DEFINE CHANNEL(TO.Qm2) CHLTYPE(CLUSRCVR) TRPTYPE(TCP) CONNAME('192.168.x.x(3414)') CLUSTER (CLUSTERA)
--DEFINE CHANNEL(TO.Qm1) CHLTYPE(CLUSSDR) TRPTYPE(TCP) CONNAME('192.168.x.x(2414)') CLUSTER (CLUSTERA)
Qm3:
--DEFINE CHANNEL(TO.Qm3) CHLTYPE(CLUSRCVR) TRPTYPE(TCP) CONNAME('192.168.x.x(4414)') CLUSTER (CLUSTERA)
--DEFINE CHANNEL(TO.Qm1) CHLTYPE(CLUSSDR) TRPTYPE(TCP) CONNAME('192.168.x.x(2414)') CLUSTER (CLUSTERA)
B:参见A类似
CCSID
MQCCSID
值为队列管理器的CCSID或与之相匹配的CCSID。
例如Windows上的队列管理器的CCSID为1381，AIX上可设为1386或1208
export MQCCSID=1208
export MQCCSID=1386
\--------------------------------------
CCSID是一个字符集的标识。作为unicode标准通过定义一个字符集内每个字符要对应那个数字值的方式定义了一个字符集。这说明CCSID就是一个定义字符集顺序的标识数码罢了。CCSID是IBM用来标识字符序列的标识代码。这个架构定义了SDCS(单字符集)的CCSID值，MBCS(多字符集)的CCSID值和混合单字符多字符集的混合CCSID值。多字符集的CCSID一般用于语言，比如中文，日文，韩文，这些语言的字符量很大，无法用单字节的码值来代表。
CCSID间的转换有多种类型。其中一种转换就是从一种CCSID到另一种CCSID的转换，举例来说从ASCII(CCSID 1252)到EBCDIC(CCSID 37)。另一种是从串数据到另一种数据类型的转换。举例来说转换字符串数据到数值。在所有的这种类型的转换中都必须标识CCSID值来保证转换的正确进行。但是转换是有要求的，第一种转换的前提是转到的CCSID的类型中要包含转换前的CCSID类型中要转换的字符，比如，如果从CCSID1381(S-CHGBPC-DATA)类型的简体中文的PC编码中的一个中文字符"中"字到其他CCSID编码转换到的编码起码要求这个CCSID编码的字符集中包含同样的"中"字。
\-----------------------------------------------
WebSphere MQ 无法将 CCSID 1381 中标记的字符串数据转换为 CCSID 5488 中的数据。1381属于简体中文，5488属于GB18030，虽然都是中文，但是在语言集上是两个不相同的语言集，所以不能相互转换。实际上GB18030包含了简体中文，繁体中文以及几种少数民族的语言，后面两种字符都是在简体中文集中找不到对应映射的，所以不能转换。有一种可行的解决办法就是用UTF-8（1208）作为两种语言集的中介。

### MQCCSID 到底是什么，在干什么

首先，安装好了MQ Server端并新建了QMGR，可以更改其MQCCSID, 我们环境中所有QMGR的MQCCSID都是1386

再次，在MQ Server上面任何用户在往该MQGR的queue里面放消息时，消息使用的字符编码都是1386与QMGR的设定相同，这里即使在用户的profile里面export MQCCSID等于别的，结果消息也是始终与QMGR的设定保持一样。

最后，export MQCCSID只有在MQ Client端才有用，通过export MQSERVER 连接到MQ Server的MQ Client端可以给用户的profile里面设定MQCCSID, 我们这边的环境AIX 上面如果没有给用户特别设定MQCCSID则default的是819即en_US.ISO8859-1

ccsid 英文意思是双字节字符集标识，是IBM的企业标准。中国汉字有多种汉字字符集标准，最常用的有两种。ccsid与中国国标汉字字符集编码不一样。ccsid是IBM把世界范围的需用两个字节表示的文字代码，根据国家区域，设置为不同的ccsid标识。所以在建立db2/400文件都要对应一个ccsid，以便数据字段能够存放这个ccsid中的字符。通常这个ccsid是采用系统默认值，也可以特殊设置某个pf的ccsid。因为400ccsid是基于ebcdic码编写的，而目前unix和pc机是基于acsii码编写的，这两者的相同国家的字符代码存在差异，所以在进行基于不同代码基础的计算机进行通讯，就要进行代码集的字符转换。通常我们常用到ftp进行400与pc机，或400与UNIX平台的数据传输。

如果进行上述数据传输，通讯端必须设置pc端，或unix端和400端的交换字符集。幸运的是，ibm的通讯线路已经设置好ebcdic与ascii的自动转入和转出，我们只要选择设置ccsid的具体代码就可以了。

Borlan公司也有一套自己的类似ibm的ccsid，在使用基于Borlan数据和编程软件时也要设置这些‘ccsid’，否则通讯就有问题。

顺便说一下，我曾经遇到使用国内一家金融服务的加密软件公司的产品，他们就是利用ibm ccsid与Borland的字符集的设置差异，达到其这类加密产品固定在一定环境下才能使用的目的。
为了帮助网友解决“MQ中如何查看CCSID是多少？”相关的问题，中国学网通过互联网对“MQ中如何查看CCSID是多少？”相关的解决方案进行了整理,用户详细问题包括:默认1381就不用说了 我想知道在哪看 如果要改 怎么改？，具体解决方案如下：
解决方案1：
runmqsc MQ名

dis QMGR
显示全信息 其中就有CCSID
解决方案2：
进入console
运行

ALTER QMGR CCSID(你要改成多少？)

如“1381”
解决方案3：
runmqsc queue_name
ALTER QMGR CCSID("xxxxx")

原文链接：

https://blog.csdn.net/paopaohehe/article/details/107583677

https://www.cnblogs.com/siwei1988/p/5923038.html