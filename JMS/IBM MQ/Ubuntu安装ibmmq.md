# Ubuntu安装ibmmq

> http://wjhsh.net/lemon-le-p-14928457.html

ibmmq一共有三个版本，开发版、试用版、正式版，具体的区别如下；

- 开发版（dev version）：不需要注册IBM

- 试用版（trial version）：需要注册IBM，试用90天；

- 正式版（product version）：需要注册IBM，需要License；

## **1、下载地址**

https://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/messaging/mqadv/

## 2、解压

```bash
tar -zxvf mqadv_dev922_ubuntu_x86-64.tar.gz
```

## 3、执行接受license

输入1，同意许可

```
./mqlicense.sh
```

## 4、安装runtime

```
dpkg -i ibmmq-runtime_9.2.2.0_amd64.deb
```

## 5、安装samples

```
dpkg -i ibmmq-samples_9.2.2.0_amd64.deb
```

## 6、安装server

```
dpkg -i ibmmq-gskit_9.2.2.0_amd64.deb
dpkg -i ibmmq-server_9.2.2.0_amd64.deb
```

## 7、安装client

```
dpkg -i ibmmq-client_9.2.2.0_amd64.deb
```

## 8、检查环境是否可用

```
/opt/mqm/bin/mqconfig
```

可以看到都是ok的

![img](https://img2020.cnblogs.com/blog/746846/202106/746846-20210624202138835-1981489428.png)

## 9、导入环境变量

```
vim /etc/profile
export IBMMQ_HOME=/opt/mqm
export PATH=$PATH:$IBMMQ_HOME/bin:$IBMMQ_HOME/samp/bin
```

```
source /etc/profile
```

## 10、配置信息

安装后会默认创建mqm的用户和组，相关的目录信息；

安装目录：/opt/mqm
数据目录：/var/mqm
日志目录：/var/mqm/log
错误目录：/var/mqm/errors

运维部分就结束了，可交付给开发作为测试环境；如果需要学习如何使用，则需要查看其他的文档。

原文地址：https://www.cnblogs.com/lemon-le/p/14928457.html