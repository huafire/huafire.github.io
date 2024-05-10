---
title: Centos7安装zookeeper
tags:
  - Hbase
  - zookeeper
categories:
  - Hbase
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: 5f822e77
date: 2021-09-26 10:35:59
---

# Centos7安装zookeeper

> 环境：CentOS7、jdk1.8.0_281、hadoop-3.2.2
>
> zookeeper版本：3.6.3

```shell
# 解压apache-zookeeper-3.6.3-bin.tar.gz
tar -zxvf apache-zookeeper-3.6.3-bin.tar.gz -C /usr/local

# 进入/usr/local，改一下文件夹名字，方便后续操作
cd /usr/local/
mv apache-zookeeper-3.6.3-bin/ zookeeper

# 创建用来存放数据的文件夹
mkdir -p /usr/local/zookeeper/data

# 创建用来存放日志的文件夹
mkdir -p /usr/local/zookeeper/logs

# zookeeper默认启动配置文件名字为zoo.cfg，所以将样例配置文件复制一下，原文件做备份
cp /usr/local/zookeeper/conf/zoo_sample.cfg /usr/local/zookeeper/conf/zoo.cfg

# 将/usr/local/zookeeper/conf/下的案例，复制一个简单的案例
cp /usr/local/zookeeper/conf/zoo_sample.cfg /usr/local/zookeeper/conf/zoo.cfg
```

​	修改`zoo.cfg`文件，找到下面内容：

```
dataDir=/tmp/zookeeper
```

​	改为下面内容：

```
dataDir=/usr/local/zookeeper/data
dataLogDir=/usr/local/zookeeper/logs
server.1=master:2890:3890
server.2=slave1:2890:3890
server.3=slave2:2890:3890
```

​	在`/usr/local/zookeeper/data`下，建立一个文件`myid`用于区别主机和备用机

```shell
vim /usr/local/zookeeper/data/myid
```

​	`CentOS01`中写入1，`CentOS02`中写入2，`CentOS03`写入3

配置环境变量`/etc/profile`

```profile
export ZOOKEEPER_HOME=/home/program/zookeeper
PATH=$PATH:$ZOOKEEPER_HOME/bin
```

刷新环境变量

```shell
source /etc/profile
```



```shell
# 启动zookeeper
zkServer.sh start

# 检查进程（正常有一个进程QuorumPeerMain
jps

# 检查状态（正常状态应该是一台是leader，另两台是follower）
zkServer.sh status

# 关闭zookeeper
zkServer.sh stop
```

​	**注意事项：日志中报错信息为拒绝连接，在`zoo.cfg`文件中，`master、slave1、slave2`不可以使用`localhost`或者`127.0.0.1`，同时在`/etc/hosts`中，不可以使用`127.0.0.1`来映射`master`等。**
