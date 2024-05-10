---
title: hadoop分布式部署
tags:
  - hadoop
categories:
  - hadoop
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: 1be7f29f
date: 2021-04-02 11:07:58
---

## hadoop 分布式部署

> 环境：3 台服务器，配置如下：
>
> ​	4核8G一台（主节点）系统：Centos7.6
>
> ​	1核2G两台（副节点）系统：Centos7.4
>
> jdk 版本：1.8.0_281
>
> hadoop版本：3.1.4

​	`hadoop`版本下载地址：https://archive.apache.org/dist/hadoop/common/

​	`jdk`环境就不说了，下面开始配置`ssh`

### hosts文件修改

​	将我们准备用来部署的服务器的地址放进去：

```shell
vim /etc/hosts
```

添加映射：

```
112.74.82.83 master
182.92.113.245 slave1
```

修改以后，测试一下是否可以`ping`通：

```shell
[root@hua ~]# ping 112.74.82.83
PING 112.74.82.83 (112.74.82.83) 56(84) bytes of data.
64 bytes from 112.74.82.83: icmp_seq=1 ttl=56 time=47.3 ms
64 bytes from 112.74.82.83: icmp_seq=2 ttl=56 time=47.3 ms
64 bytes from 112.74.82.83: icmp_seq=3 ttl=56 time=47.2 ms
^C
--- 112.74.82.83 ping statistics ---
4 packets transmitted, 3 received, 25% packet loss, time 3003ms
rtt min/avg/max/mdev = 47.265/47.328/47.387/0.049 ms

```

​	然后可以按照配置的顺序来重新命令三台服务器：

​	**（注意：下面为命令，按照自己刚刚配置的文件名称来命令）**

```
hostnamectl set-hostname master
hostnamectl set-hostname slave1
hostnamectl set-hostname slave2
```



### ssh免秘钥登录

​	可能会有小伙伴想问了，`ssh`是什么，干什么用的

​	其实我一开始也挺懵，但是想想就明白了，我想要搭建的是`hadoop`集群，既然是集群，那么各个节点就需要互相`ping`通，既要安全，又要互相通讯，`ssh`就满足了我们的需要。

​	那么为什么要免密登录呢，`hadoop`运行时，各个主机需要频繁的通讯，我们不希望因为频繁的输入密码，而浪费了大量的时间。所以需要设置各个主机间的免密登录。

​	执行下面的命令：

```shell
ssh-keygen -t rsa
```

​	执行后，需要几次回车，就会出现如下的代码：

`slave1`:

```shell
[root@hua ~]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:V6Czlhyaj9c5exDnIi7u9TIkims2y4U6ZTLRZDknwoE root@hua
The key's randomart image is:
+---[RSA 2048]----+
|o...      .      |
|E.* .    . .     |
| = +    +   .    |
|. .    + =...    |
| .    o S .+     |
|o o.  .+ooo..    |
| =.....++.+o     |
|.o=.. .o+. o.    |
|.+++ oo. oo.     |
+----[SHA256]-----+

```

​	`slave2`和`master`同理。



​	执行完以后，我们会在`/root/.ssh/`文件夹下看到以下几个文件：

```shell
[root@slave1 .ssh]# ls
id_rsa  id_rsa.pub
```

​	其中`id_rsa`是私钥`id_rsa.pub`是公钥，我们登录时，一般要用到公钥。

​	然后，我们需要将公钥加入到一个文件中，然后将这个文件发送到每台机器上。

```shell
# 在slave1和slave2下分别把id_rsa.pub发送到主机上
scp id_rsa.pub root@master:~/.ssh/id_rsa.pub.slave1
scp id_rsa.pub root@master:~/.ssh/id_rsa.pub.slave2
```

​	`scp`命令本人运行时报错，所以本人没有使用这种方法，而是直接下载，并手动上传，覆盖掉了原来的文件夹。如有高人知道原因，请在评论区留言，感激不尽！



​	在`master`中的`/root/.ssh`下把`id_rsa.pub`、`id_rsa.pub.slave1`、`id_rsa.pub.slave2`追加到`authorized_keys`中。

```shell
cat id_rsa.pub >> authorized_keys 
cat id_rsa.pub.slave1 >> authorized_keys 
cat id_rsa.pub.slave2 >> authorized_keys
```

​	然后把`authorized_keys`传回到`slave1`和`slave2`中

```shell
scp authorized_keys root@slave1:~/.ssh 
scp authorized_keys root@slave2:~/.ssh
```

​	最后修改文件权限。

```shell
chmod 755 ~
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys 
```

​	这个时候，我们再用命令从`master`登录`slave1`，就不需要密码了。出现下面的命令，就说明我们的`ssh`配置好了。

```shell
[root@master ~]# ssh slave1
Last login: Thu Apr  1 13:01:50 2021 from 123.7.182.188

Welcome to Alibaba Cloud Elastic Compute Service !
```



### 安装并配置Hadoop

```shell
# 解压缩：
[root@master local]# tar -zxvf /home/tool/hadoop-2.7.1.tar.gz -C /usr/local/

# 重命名为hadoop
[root@master local]# mv hadoop-2.7.1/ hadoop
```

​	新建几个文件夹：

```shell
mkdir  /root/hadoop
mkdir  /root/hadoop/tmp
mkdir  /root/hadoop/var
mkdir  /root/hadoop/dfs
mkdir  /root/hadoop/dfs/name
mkdir  /root/hadoop/dfs/data
```



#### 	修改配置文件

​	我的配置文件在`/usr/local/hadoop/etc/hadoop`下

##### 配置`core-site.xml`

​	修改`core-site.xml`，在`<configuration></configuration>`之间插入如下语句：

```xml
<configuration>
    <!-- 指定 namenode 的通信地址 默认 8020 端口 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://master/</value>
    </property>

    <!-- 指定 hadoop 运行时产生文件的存储路径 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/local/hadoop/tmp</value>
    </property>
</configuration>
```



##### 配置`hadoop-env.sh`

​	修改`hadoop-env.sh`，将`export JAVA_HOME=${JAVA_HOME}`中的`${JAVA_HOME}`改成自己的`jdk`安装路径。

```sh
export JAVA_HOME=/usr/local/java/jdk1.8.0_281
```



##### 配置`hdfs-site.xml`

​	修改`hdfs-site.xml`文件，在`<configuration></configuration>`中加入如下配置：

```xml
<configuration>

    <!-- namenode 上存储 hdfs 名字空间元数据-->
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/usr/local/hadoop/namenode</value>
    </property>

    <!-- datanode 上数据块的物理存储位置-->
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/usr/local/hadoop/datanode</value>
    </property>

    <!-- 设置 hdfs 副本数量 -->
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    
    <property>
  		<name>dfs.http.address</name>
  		<value>0.0.0.0:50070</value>
	</property>
    
     <property>
        <name>dfs.permissions</name>
        <value>false</value>
    </property>

</configuration>
```



##### 配置`mapred-site.xml`

​	修改`mapred-site.xml`，并在`<configuration></configuration>`中配置：

```xml
<configuration>

    <!-- 指定yarn运行-->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
    </property>

    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
    </property>

    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=/usr/local/hadoop</value>
    </property>

</configuration>

```

##### 配置`workers`文件

​	修改`workers`文件，添加如下内容：

```
slave1
slave2
```



##### 配置`yarn-site.xml`

​	修改`yarn-site.xml`文件，在`<configuration></configuration>`中加入配置：

```xml
<configuration>
<!-- Site specific YARN configuration properties -->
    <!-- 指定ResourceManager的地址 -->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>master</value>
    </property>

    <!-- reducer取数据的方式是mapreduce_shuffle -->  
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

</configuration>
```



##### 配置`start-dfs.sh`和`stop-dfs.sh`

​	这两个文件在 `/usr/local/hadoop/sbin/` 中，分别在`start-dfs.sh`和`stop-dfs.sh`文件的最前面，添加如下内容：

```sh
HDFS_DATANODE_USER=root
HDFS_DATANODE_SECURE_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```



##### 配置`start-yarn.sh` 和 `stop-yarn.sh`

这两个文件在 `/usr/local/hadoop/sbin/` 中，分别在`start-yarn.sh`和`stop-yarn.sh`文件的最前面，添加如下内容：

```sh
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```



### 配置Hadoop环境变量

在 `~/.bashrc` 中追加如下内容：

```bash
export HADOOP_HOME=/usr/local/hadoop/hadoop-3.1.4
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

刷新`~/.bashrc`：

```bash
source ~/.bashrc
```



### 启动Hadoop

​	先在`namenode`上进行初始化：

```shell
[root@master hadoop]# ./bin/hadoop namenode -format
```

​	启动`hadoop`：

```shell
[root@master hadoop]# ./sbin/start-all.sh
```

​	输入两次 yes 后，即可成功。

​	关闭`hadoop`：

```shell
[root@master hadoop]# ./sbin/stop-all.sh
```



### 踩过的坑：

#### 	`master`节点中的`namenode`无法启动

​	原因：因为我用的是三台阿里云的服务器，度娘说，好像只有阿里云的服务器才会这样，在`master`，`slave1`和`slave2`节点中`/etc/hosts`下，将`master`的`ip`改成私网`ip`，注意：只修改`master`的`ip`，`slave1`和`slave2`节点`ip`还是公网`ip`。



#### 	`namenode`节点启动后，无法访问50070端口

​	已经配置过了阿里云的安全组，但是还是无法访问，问过度娘以后，找到如下方法：

​	在`hdfs-site.xml`文件中，加入下面的配置：

```xml
<property>
  <name>dfs.http.address</name>
  <value>0.0.0.0:50070</value>
</property>
```

​	重启`hadoop`，重新格式化`namenode`，成功访问50070端口。



​	还要注意一点哈，`hadoop`成功运行以后，是有两个端口的，一个是50070，另一个是8088，需要将`master`的这两个节点开放。



​	注意：还有一件小事，不要随便关掉防火墙，cpu占用一直是百分百，各种方法尝试后无效，无奈之下，只能重装系统。

> hadoop应该是我们的必备技能，之后还会扩展，持续更新哦~