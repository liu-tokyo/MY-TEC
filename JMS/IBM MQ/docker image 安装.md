# IBM MQ-Docker部署及基本操作

> https://hub.docker.com/r/ibmcom/mq/
>
> https://199604.com/2596

## Docker部署

> Docker仓库：`https://hub.docker.com/r/ibmcom/mq`

- 最新版本

  ```bash
  docker run --name ibmmq -d \
  	-e LICENSE=accept \
  	-e MQ_QMGR_NAME=QM1 \
  	-p 1414:1414 -p 9443:9443 -p 9157:9157 \
  	ibmcom/mq
  ```

- 指定版本 ()

  ```bash
  docker run --name ibmmq -d \
  	-e LICENSE=accept \
  	-e MQ_QMGR_NAME=QM1 \
  	-p 1414:1414 -p 9443:9443 -p 9157:9157 \
  	ibmcom/mq:9.2.0.0-r3
  ```

## 控制台

### 开启

进入docker容器，执行`strmqweb`

## 访问

- 地址

  https://ip:9443/ibmmq/console

- 用户名密码可在运行环境配置,默认

  ```bash
  User: admin
  Password: passw0rd
  ```

- 输出详情控制台url

  ```bash
  bash-4.4$ dspmqweb
  MQWB1124I: Server 'mqweb' is running.
  URLS:
    https://96c67a02cb18:9443/ibmmq/console/
    https://96c67a02cb18:9443/ibmmq/rest/
  bash-4.4$ strmqweb
  ```


## 队列管理器

- 创建：

  ```bash
  crtmqm -q qmgrName
  
  eg:
  
  crtmqm BAASDEC_RDMRCVS_GW
  
  或者是crtmqm -q BAASDEC_RDMRCVS_GW（-q是指创建缺省的队列管理器）
  
  或者是 crtmqm -lc -lf 100 -lp 3 -ls 3 BAASDEC_RDMRCVS_GW
  
  或者是crtmqm -lf 51200 -lp 8 -ls 4 -lc -md /AppData/MQData -ld /AppLogs/MQLogs BAASDEC_RDMRCVS_GW
  ```

  -lc 是采用循环日志  
  -lf 是每块日志的大小，4k为单位的，100就是100*4k  
  -lp 是主逻辑日志的数量  
  -ls 是辅逻辑日志的数量  
  -md 持久化数据文件的目录  
  -ld 日志文件的目录

- 删除：

  ```bash
  dltmqm qmgrName
  
  eg:
  
  dltmqm BAASDEC_RDMRCVS_GW
  ```

- 启动：

  ```bash
  strmqm qmgrName
  
  eg:
  
  strmqm BAASDEC_RDMRCVS_GW
  ```

### 停止

- 受控停止：`endmqm qmgrName` eg:`endmqm BAASDEC_RDMRCVS_GW`

- 立即停止：`endmqm -i qmgrName` eg:`endmqm -i BAASDEC_RDMRCVS_GW`
-  强制停止：`endmqm -p qmgrName` eg:`endmqm -p BAASDEC_RDMRCVS_GW`

### 显示

```bash
dspmq -m qmgrName
## eg.
dspmq -m BAASDEC_RDMRCVS_GW
```

### 运行

```bash
## runmqsc qmgrName
runmqsc BAASDEC_RDMRCVS_GW
```



## 本地队列

### 创建

- #### 指令

  ```bash
  define ql (qlName) defpsist (yes) replace
  
  define qlocal(qlName)
  ```

- #### 简单本地队列

  ```bash
  define qlocal('QLSdcGzGZEportDxp04') defpsist(yes) maxdepth(1000000) maxmsgl(11000000) replace
  ```
  
- #### 传输队列

  简单

  ```bash
  define qlocal ('QXGZEportSdcGzDxp04') usage (xmitq) defpsist(yes) maxdepth(1000000) maxmsgl(11000000)
  ```

  多参数

  ```bash
  DEFINE QLOCAL (BAASDEC_RDMRCVS_TRA) USAGE (XMITQ) DEFPSIST(YES) MAXDEPTH(100000) TRIGGER TRIGTYPE(FIRST) INITQ(SYSTEM.CHANNEL.INITQ) TRIGDATA(BAASDEC.RDMRCVS.CHL) REPLACE
  ```

- #### 参数介绍

  > `QLOCAL (BAASDEC_RDMRCVS_TRA)`：BAASDEC_RDMRCVS_TRA为传输队列名称；
  >
  > USAGE (XMITQ)：定义该队列为传输队列
  >
  > (参数详解：USAGE
  >
  > 如果是NORMAL代表的是本地队列
  >
  > 如果是XMITQ代表的是传输队列)；
  >
  > `DEFPSIST(YES)`：队列上的消息在队列管理器重新启动时保存了下来
  >
  > (参数详解：DEFPSIST：队列中消息持久性默认值。
  > NO 该队列上的消息在队列管理器重新启动时丢失
  > YES 该队列上的消息在队列管理器重新启动时保存了下来。)；
  >
  > `MAXDEPTH(100000)`：队列上允许的最大消息数为100000；
  >
  > `TRIGTYPE(FIRST)`：触发类型第一条消息触发，有第一条消息之后开始创建队列
  >
  > `INITQ(SYSTEM.CHANNEL.INITQ)`：初始队列为SYSTEM.CHANNEL.INITQ
  >
  > `TRIGDATA(BAASDEC.RDMRCVS.CHL)`：触发数据，也就是触发后要启动的通道为BAASDEC.RDMRCVS.CHL
  >
  > 其中：DEFINE可以缩写为DEF，QLOCAL可以缩写为QL。

### 删除

- #### 指令

  ```bash
  delete qlocal (qlName)
  ```

  

## 远程队列

### 创建

- #### 指令

  ```bash
  define qremote (qrName) rname (AAA) rqmname (qmgrName) xmitq (qtName)
  ```

  等同于如下：

  #### 简单：

  ```bash
  define qremote ('QRGZEportSdcGzDxp04') defpsist(yes) rname ('QLGZEportSdcGzDxp04') rqmname('QMGzMgr') xmitq ('QXGZEportSdcGzDxp04')
  ```

  #### 较多参数：

  ```bash
  DEFINE QREMOTE (BAASDEC_RDMRCVS_REMOTEQ) RNAME (RDMRCVS.RECV.BAASDEC.QUEUE) RQMNAME (RDMRCVS.CLUSTER) XMITQ (BAASDEC_RDMRCVS_TRA) CLUSTER (BAASDEC) REPLACE
  ```

  #### 参数介绍：

  > QREMOTE (BAASDEC_RDMRCVS_REMOTEQ)：发送方远程队列名称BAASDEC_RDMRCVS_REMOTEQ；
  >
  > RNAME (RDMRCVS.RECV.BAASDEC.QUEUE)：接收方的本地接收队列名`RDMRCVS.RECV.BAASDEC.QUEUE`，这个需要和对手系统确认好，这个就是对方系统配置的本地队列名；
  >
  > RQMNAME (RDMRCVS.CLUSTER)：接收方队列管理器名，RDMRCVS.CLUSTER；
  >
  > XMITQ (BAASDEC_RDMRCVS_TRA)：发送方传输队列名，传输队列BAASDEC_RDMRCVS_TRA；
  >
  > CLUSTER (BAASDEC)：所属的集群为BAASDEC。
  >
  > 其中：DEFINE可以缩写为DEF，QREMOTE 可以缩写为QL。
  >
  > 注意：RNAME 与对方本地队列名相同是需要一样的，否则消息无法发送出去。

### 删除

- ```bash
  delete qremote (qlName)
  ```

  

## 死信队列

### 定义

- 定义一个死信队列DEADQ

  ```bash
  DEFINE QLOCAL（DEADQ）MAXDEPTH(290000) DEFPSIST（YES） REPLACE
  ```

### 设定

- 设定队列管理器的死信队列

  ```bash
  ALTER QMGR DEADQ(DEADQ)
  ```

  

## 通道

> 发送通道：CHLTYPE (SDR)
>
> 接收通道：CHLTYPE (RCVR)
>
> 群集发送通道：CHLTYPE (CLUSSDR)
>
> 群集接收通道：CHLTYPE (CLUSRCVR)
>
> MQI连接服务器通道：CHLTYPE (SVRCONN)

### 发送通道

启动：start channel(cName)
停止：stop channel(cName)
重置：reset channel(cName)

```bash
define channel ('CHGZEportSdcGzDxp04') chltype (sdr) conname ('172.16.208.194(1415)') xmitq ('QXGZEportSdcGzDxp04') maxmsgl(11000000) discint(0) trptype (tcp)
DEFINE CHANNEL(BAASDEC.RDMRCVS.CHL) CHLTYPE(SDR) TRPTYPE (TCP) DISCINT(0) CONNAME('22.188.183.244(1817)') XMITQ (BAASDEC_RDMRCVS_TRA) REPLACE
```

参数介绍：

> CHANNEL(BAASDEC.RDMRCVS.CHL)：发送通道名为BAASDEC.RDMRCVS.CHL；
>
> CHLTYPE(SDR)：CHLTYPE设置通道类型，发送通道；
>
> TRPTYPE (TCP)：TRPTYPE 设置通讯协议，TCP；
>
> DISCINT(0) ：DISCINT设置通道停止时间间隔（0为永不停止）；
>
> CONNAME('22.188.XXX.XXX(1X1X)')：连接地址为22.188.XXX.XXX(1X1X)；
>
> XMITQ (BAASDEC_RDMRCVS_TRA)：传输队列名称为BAASDEC_RDMRCVS_TRA。

### 接收通道

创建：`define channel(cName) chltype(rcvr) trptype(tcp)`

```bash
define channel ('CHSdcGzGZEportDxp04') chltype (rqstr) conname('172.16.208.194(1415)') maxmsgl(11000000) trptype (tcp)
DEFINE CHANNEL (RDMRCVS.BAASDEC.CHL) CHLTYPE (RCVR) TRPTYPE(TCP) REPLACE
```

参数介绍：

> CHANNEL (RDMRCVS.BAASDEC.CHL)：接收通道名为RDMRCVS.BAASDEC.CHL，通常和发送通道的名称互换一下部分内容。
>
> CHLTYPE (RCVR)：接收通道。
>
> CONNAME('22.188.XXX.XXX(1X1X)')：连接地址为22.188.XXX.XXX(1X1X)；

## 进程定义



```bash
创建：define process(pName) appltype(unix) applicid('runmqchl –zzbankO –c oTocchannel')
```

## 监听器



监听器是用于监听外界对队列管理器的连接的程序，就是对方MQ管理器来探测，本地要给对方一个回应，监听器就是起这个作用的

### 创建：



```bash
define listener(lName) trptype(tcp) control(qmgr) port(1415)
```

eg:

创建监听器LSNRPMTS_BAAS

```bash
DEFINE LISTENER(LSNRPMTS_BAAS) TRPTYPE(TCP) PORT(4001) BACKLOG(0) CONTROL(QMGR) REPLACE
```

参数介绍：

> LISTENER(LSNRPMTS_BAAS)：监听器的名称为LSNRPMTS_BAAS，这个在一众队列管理器中创建一个就好。但是这些队列管理器的端口不能一样；
>
> TRPTYPE(TCP)：传输协议类型为TCP；
>
> PORT(4001)：端口号为4001；
>
> BACKLOG(0)：侦听器支持的最大并发连接请求数，需要根据实际情况来确定；
>
> CONTROL(QMGR)：启动/停止控制方式为QMGR
>
> (参数详情：
>
> MANUAL：手动启动/停止
>
> OMGR：随QM启动/停止
>
> STARTONLY：随QM启动，不随其停止)

### 启动：

```bash
start listener(lName)
```

## 状态查看

```bash
通道：dis chl(*)
通道详细信息：dis chl(name)
监听：dis listener(*)
通道状态：dis chs(*)
序列号查看：dis chs(name) curseqno
通道序列号重置：reset chl(name) seqnum(1)
监听详细信息：dis listener(name)
监听状态：dis lsstatus(*)
队列：dis q(*)
队列详细信息：dis q(name)
查询：

1）dis qmgr或 dis qmgr all

显示队列管理器的全信息。

2）dis qmgr ccsid

显示队列管理器的部分属性信息。

3）dis q(*)

显示某个队列管理器下的全部队列。

4）dis q(*) all

显示某个队列管理器下的全部队列的全部信息。

5）dis ql(*) 和dis ql(*) all

显示某个队列管理器下的全部本地队列（的全部信息）。

6）dis qr(*) 和dis qr(*) all

显示某个队列管理器下的全部远程队列（的全部信息）。

7）dis q(name)

显示某个特定队列的全部信息。

8）dis q(name) CURDEPTH

显示某个特定队列的特定属性信息。

9）dis chl(*)

显示某个队列管理器下的通道信息。

10）dis chs(*)

显示某个队列管理器下的通道状态信息。

11）dis chs(name) curseqno

显示某个队列管理器下的某个通道的某个属性信息。



修改：

这块用来修改一些配置，知道特定的语法格式就好了

1）alter qmgr ccsid(1381)

将队列管理器的ccsid改成1381

2）alter ql(name) get (ENABLED)
将该队列管理器下的本地队列的get参数改为ENABLED。
```

## ccsid

```bash
查看：display qmgr ccsid
修改：alter qmgr ccsid(1381)
```

参考：`https://www.cnblogs.com/zhuozige/p/17015682.html`
