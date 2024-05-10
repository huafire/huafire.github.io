---
title: Hbase单机部署
tags:
  - hbase
categories:
  - hbase
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: dbf36185
date: 2021-11-06 16:33:31
---

# Hbase单机部署

```shell
# 解压缩
tar -zxvf hbase-2.3.6-bin.tar.gz -C /usr/local

# 重命名
cd /usr/local
mv hbase-2.3.6-bin hbase

# 配置环境变量
vi /etc/profile
```

```profile
export HBASE_HOME=/usr/local/hbase
export PATH=$PATH:$HBASE_HOME/bin
```

```shell
# 刷新环境变量
source /etc/profile
```

​	修改配置文件`/usr/lcoal/hbase/conf/hbase-site.xml`

```xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>file:///usr/local/hbase/hbasedata</value>
  </property>
  <property>
    <name>fs.defaultFS</name>
    <value>file:///usr/local/hbase</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/usr/local/hbase/hbasezookeeper</value>
  </property>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>false</value>
  </property>
  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
</configuration>
```

```shell
# 启动命令
start-hbase.sh

# 查看进程
jps
```

​	如果是虚拟机，可以考虑关掉防火墙，`hbase`默认端口是`16010`，如果需要更改端口，可以在`hbase-site.xml`中，添加下面代码：

```xml
  <property>
      <name>hbase.master.info.port</name>
      <value>60010</value>
  </property>
```

