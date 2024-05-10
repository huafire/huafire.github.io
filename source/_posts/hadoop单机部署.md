---
title: hadoop单机部署
tags:
  - hadoop
categories:
  - hadoop
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: '91054617'
date: 2021-04-28 09:15:30
---

## 部署`hadoop3.1.4`

> 说明：`hadoop`单机部署与分布式部署基本一样，但是需要注意的是，`hadoop2`与`hadoop3`差距较大，这里部署的是`hadoop3.1.4`
>
> 环境：阿里云服务器`Centos7.6`，`jdk1.8.0_281`

首先创建几个目录，如下：

```shell
mkdir /usr/local/hadoop/data
mkdir /usr/local/hadoop/hdfs/name
mkdir /usr/local/hadoop/hdfs/data
```

### 修改配置文件

以下文件均在`/usr/local/hadoop/etc/hadoop`下

#### `core-site.xml`

​	添加如下配置：

```xml
<configuration>
<!--定义namenode地址 默认9000-->
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://hadoop1:9003</value>  <!-- 注意：hadoop1要改成自己的主机号 -->
  </property>
 <!--修改用于hadoop存储数据的默认位置-->
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/usr/local/hadoop/data</value>
  </property>
</configuration>
```



#### `hdfs-site.xml `

添加如下配置：

```xml
<configuration>
	<property>
  <name>dfs.namenode.name.dir</name>
  <value>/usr/local/hadoop/hdfs/name</value>
	</property>
	<property>
    <name>dfs.datanode.data.dir</name>
    <value>/usr/local/hadoop/hdfs/data</value>
	</property>
	<property>
    <name>dfs.replication</name>
    <value>1</value>
	</property>
     <property>
        <name>dfs.permissions</name>
        <value>false</value>
    </property>

</configuration>
```

#### `mapred-site.xml `

添加配置如下：

```xml
<configuration>
<property>
       <name>mapreduce.framework.name</name>
       <value>yarn</value>
   </property>
</configuration>
```



#### `yarn-site.xml`

添加如下配置：

```xml
问题出在hadoop文件夹下/etc/hadoop/目录下的配置文件：yarn-site.xml

修改yarn-site.xml文件，将其<configuration></configuration>中的配置修改为：

<configuration>
 
<!-- Site specific YARN configuration properties -->
        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>xxxx</value>   <!--填写自定义的主机名/ip-->
        </property>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>0.0.0.0:8032</value>
         </property>
         <property>
                <name>yarn.resourcemanager.scheduler.address</name>
                <value>0.0.0.0:8030</value>
         </property>
         <property>
                <name>yarn.resourcemanager.resource-tracker.address</name>
                <value>0.0.0.0:8031</value>
         </property>
         <property>
                <name>yarn.resourcemanager.admin.address</name>
                <value>0.0.0.0:8033</value>
         </property>
         <property>
                <name>yarn.resourcemanager.webapp.address</name>
                <value>0.0.0.0:8088</value>
         </property>
</configuration>
```



#### `yarn-env.sh`

添加如下配置：

```sh
export $JAVA_HOME=/usr/local/java/jdk1.8.0_281
```



#### `hadoop-env.sh`

添加如下配置：

```sh
JAVA_HOME=/usr/local/java/jdk1.8.0_281
HADOOP_SHELL_EXECNAME=root
```



#### `hadoop/bin/hdfs`

```
把HADOOP_SHELL_EXECNAME="hdfs"修改为HADOOP_SHELL_EXECNAME="root"即可
```



#### `.sh`文件

为防止启动失败，需修改`start-dfs.sh`、`start-yarn.sh`、`stop-dfs.sh`、`stop-yarn.sh`，这四个文件均在`/usr/local/hadoop/sbin/`中

修改`start-dfs.sh`、`stop-dfs.sh`，在文件头添加如下设置：

```sh
HDFS_DATANODE_USER=root
HDFS_DATANODE_SECURE_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```



修改`start-yarn.sh`、`stop-yarn.sh`，在文件头添加如下设置：

```sh
YARN_RESOURCEMANAGER_USER=root
HADOOP_SECURE_DN_USER=yarn
YARN_NODEMANAGER_USER=root
```



### 配置Hadoop环境变量

在`~/.bashrc`中追加如下内容：

```bash
export HADOOP_HOME=/usr/local/hadoop/hadoop-3.1.4
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

刷新`~/.bashrc`：

```bash
source ~/.bashrc
```



### 格式化`namenode`，并启动`hadoop`：

```shell
# 格式化namenode
./bin/hadoop namenode -format

# 启动hdfs和yarn
./sbin/start-all.sh

# 只启动hdfs
./sbin/start-dfs.sh

# 只启动yarn
./sbin/start-yarn.sh
```

### 关闭`hadoop`

```shell
# 关闭hadoop
./sbin/stop-all.sh
```

## 部署`hadoop2.7.1`

> hadoop2.7.1的配置比3.1.4要简单一点，少配置了几个文件

修改配置文件，路径为`/usr/local/hadoop/etc/hadoop`

### 下面是`hdfs`的配置

#### `hadoop-env.sh`

```shell
vim /opt/hadoop-2.7.5/etc/hadoop/hadoop-env.sh
```

找到`JAVA_HOME`，并将其改成如下形式，根据自己`jdk`路径配置

```sh
export JAVA_HOME=/usr/local/java/jdk1.8.0_281
```



#### `core-site.xml `

5.2和5.3中配置文件里的文件路径和端口随自己习惯配置

```shell
vim /opt/hadoop-2.7.5/etc/hadoop/core-site.xml
```



```xml
<configuration>
<property>
        <name>hadoop.tmp.dir</name>
        <value>file:///opt/hadoop-2.7.5</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://192.168.44.128:8888</value>
    </property>
</configuration>
```

#### `hdfs-site.xml`

```shell
vim /opt/hadoop-2.7.5/etc/hadoop/hdfs-site.xml
```

```xml
<configuration>
        <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///opt/hadoop-2.7.5/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///opt/hadoop-2.7.5/tmp/dfs/data</value>
    </property>
</configuration>
```

### 下面是`yarn`的配置

#### mapred-site.xml

```xml
cd /opt/hadoop-2.7.5/etc/hadoop/
cp mapred-site.xml.template mapred-site.xml
vim mapred-site.xml
<configuration>
    <!-- 通知框架MR使用YARN -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```



#### yarn-site.xml

```xml
vim yarn-site.xml
<configuration>
    <!-- reducer取数据的方式是mapreduce_shuffle -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```



#### yarn-env.sh

添加如下配置：

```sh
export $JAVA_HOME=/usr/local/java/jdk1.8.0_281
```



### 配置Hadoop环境变量

在`~/.bashrc`中追加如下内容：

```bash
export HADOOP_HOME=/usr/local/hadoop/hadoop-2.7.1
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

刷新`~/.bashrc`：

```bash
source ~/.bashrc
```



### 格式化`namenode`，并启动`hadoop`：

```shell
# 格式化namenode
./bin/hadoop namenode -format

# 启动hdfs和yarn
./sbin/start-all.sh

# 只启动hdfs
./sbin/start-dfs.sh

# 只启动yarn
./sbin/start-yarn.sh
```

### 关闭`hadoop`

```shell
# 关闭hadoop
./sbin/stop-all.sh
```

