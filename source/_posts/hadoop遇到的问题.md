---
title: hadoop遇到的问题
tags:
  - hadoop
categories:
  - hadoop
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: 39e09ce8
date: 2021-04-30 00:10:49
---

## 阿里云服务器hadoop忽然占用大量cpu和内存

​	今天登录服务器看了一眼，好家伙，`cpu`和内存都跑满了，我只开了`hadoop`，但是我关掉以后，还是占用了大量的资源，重启以后恢复正常。

​	重启以后，发现`ssh`无法本地登录了，部署过`hadoop`的应该都知道，`ssh`无法连接本地，`hadoop`就启动不了

### `ssh`报错：

```shell
No route to host
```

​	我是用`finalshell`连接的服务器，当前窗口没有关闭，然而当我想再次打开一个窗口时，我发现我连接不上了，幸亏当前窗口没关闭。

​	网上说防火墙没有关，我删除了防火墙规则，无效

​	卸载了`ssh`以后，重新安装，无效

```shell
# 查看ssh相关服务和安装情况
rpm -qa openssh*

# 卸载
yum remove openssh*
```

​	检查后发现`ssh`没有安装客户端，用下面命令安装`ssh`客户端：

```shell
yum -y install openssh-clients
```

​	查看`ssh`进程发现`ssh`没有开启服务，启动`ssh`命令如下：

```shell
# 查看服务状态
systemctl status sshd.service

# 开启服务
systemctl start sshd.service
```

### `authorized_keys`报错权限不够

​	再次检查`/root/.ssh`文件夹，发现无法对`authorized_keys`进行操作，报错如下：

```shell
bash: authorized_keys: 权限不够
```

​	原因是`authorized_keys`这个文件被锁住，需要对此文件解锁，命令如下：

```shell
chattr -i authorized_keys
```

​	解锁完以后，如果还是无法本地登录，直接删掉这个文件，然后用下面这条命令，生成一个新的文件：

```shell
cat id_rsa.pub >> authorized_keys
```

​	生成新的文件以后，还是无法用`ssh`本地登录的，这个是正常的，原因就是没有给这个文件设置权限，新生成的`authorized_keys`文件默认权限是777，只要检测到777，就会默认不安全，所以不让连接，我们可以用下面的命令来重新设置访问这个文件的权限：

```shell
chmod 755 ~
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys 
```

​	成功！

​	所以至于为什么忽然占用了大量的`cpu`和内存，我觉得可能还是`ssh`的问题，因为我重新配置好`ssh`以后，就没有这个问题了。当然问题可能不止这一个，这个只是我目前遇到的问题，欢迎大家留言谈论~



## 报错：`hadoop`中`datanode`无法启动

原因：由于多次对`namenode`进行格式化，导致`namenode`的`clusterID`变动，与`datanode`的`clusterID`不匹配所致。
 方法1：将我们自己创建的`/usr/local/hadoop/hdfs/name/current/VERSION`中的`clusterID`改成`/usr/local/hadoop/hdfs/data/current/VERSION`中的`clusterID`即可，或者将`data`中的`clusterID`，改成`name`中的也可以，只需要保持一致即可。
 方法2：直接删除`/usr/local/hadoop/hdfs/data`文件夹，再重新使用`./sbin/start-all.sh`命令启动。

搞定！

## 报错：`Permission denied: user=**, access=WRITE, inode="/":root:supergroup:drwxr-x`

`ide`访问`hadoop`报错：

```
Permission denied: user=hua, access=WRITE, inode="/":root:supergroup:drwxr-x
```

解决方法：

```Java
String username = "root";
        System.setProperty("HADOOP_USER_NAME", username);
```



## 报错`Connection refused: no further information`

解决方法：

​	将`/usr/local/hadoop/etc/hadoop/core-site.xml`中的

```xml
<!--定义namenode地址 默认9000-->
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://ip:9000</value>
  </property>
```

`ip`改成私网`ip`

**注：本人使用的是阿里云服务器，其他没有尝试**



## log4j日志报错：

```
log4j:WARN No appenders could be found for logger (org.apache.htrace.core.Tracer). log4j:WARN Please initialize the log4j system properly.
```

解决方法：

在配置文件`log4j.properties`（文件名必须这个，放在`resources`目录） 全选粘贴如下代码：

```properties
log4j.rootLogger=DEBUG, stdout
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```



`winutils.exe`报错：

```
can not find winutils.exe
```

在本地的`hadoop/bin`中加入这个`.exe`可执行程序

注意：需要重启`ide`，如果还是不行，请重启电脑



## 每次访问`datanode`，变成私网`ip`

hosts文件配置：

```
::1     localhost       localhost.localdomain   localhost6      localhost6.localdomain6
127.0.0.1       localhost       localhost.localdomain   localhost4      localhost4.localdomain4

172.22.****    hua     hua
```

添加一句配置，使`NameNode`返回`DataNode`的主机名而不是`IP`：

```java
configuration.set("dfs.client.use.datanode.hostname", "true");
```

本地可以拿到了`DataNode`的主机名，要访问还需要配置本地`Hosts`映射：

```
112.74.****    hua
```

注意：`win10`下的`hosts`文件在`C:\Windows\System32\drivers\etc`路径下



## log4j中出现太多DEBUG日志

​	将`log4j.properties`中的第一行代码：

```properties
log4j.rootLogger=DEBUG, stdout
```

改成

```properties
log4j.rootLogger=WARN, stdout
```

这样输出的日志就只有WARN级别及以上的才会被输出

注：日志输出级别`[level]`共有5级，如下：

```
FATAL 0
ERROR 3
WARN 4
INFO 6
DEBUG 7
```

