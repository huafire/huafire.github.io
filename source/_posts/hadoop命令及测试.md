---
title: hadoop命令及测试
tags:
  - hadoop
categories:
  - hadoop
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: 7e1d5767
date: 2021-05-14 17:25:03
---

## hadoop命令及测试

```shell
# 启动hadoop
start-all.sh

# 关闭hadoop
stop-all.sh

# 只启动hdfs
start-dfs.sh

# 只启动yarn
start-yarn.sh

# 格式化namenode
hadoop namenode -format
```

说明：`bin/hadoop fs 具体命令`和`bin/hdfs/ dfs 具体命令`这两个是完全相同的

```shell
# 查看某个命令的参数(这里以rm命令为例)
hadoop fs -help rm

# 新建文件夹test
hadoop fs -mkdir test

# 显示目录信息
hadoop fs -ls /

# 从本地剪切到到HDFS(注意：是剪切，不是复制)
hadoop fs -moveFromLocal ./test.txt /hadoop

# 追加一个文件到已经存在的文件末尾
hadoop fs -appendToFile test1.txt /hadoop

# 显示文件内容
hadoop fs -cat /hadoop/test.txt

# -chgrp 、-chmod、-chown命令与Linux文件系统中的用法一样，修改文件所属权限
hadoop fs -chmod 666 /hadoop/test.txt
hadoop fs -chown root:root /hadoop/test.txt

# 从本地文件系统中拷贝文件到HDFS路径去
hadoop fs -copyFromLocal test.txt /hadoop/test

# put命令:等同于copyFromLocal
hadoop fs -put test.txt /hadoop/test

# 从HDFS拷贝到本地
hadoop fs -copyToLocal /hadoop/test.txt ./

# 从HDFS下载文件到本地(等于copyToLocal)
hadoop fs -get /hadoop/test.txt ./

# 从HDFS的一个路径拷贝到HDFS的另一个路径
hadoop fs -cp /hadoop/test/test1.txt /hadoop/test2.txt

# 在HDFS目录中移动文件
hadoop fs -mv /hadoop/test.txt /

# 合并下载多个文件,比如HDFS的目录 /user/test下有多个文件:log.1, log.2,log.3,...
hadoop fs -getmerge /user/test/* ./log.txt

# 显示一个文件的末尾
hadoop fs -tail /hadoop/test/test.txt

# 删除文件或者文件夹
hadoop fs -rm /hadoop/test/test.txt

# 删除空目录
hadoop fs -rmdir /test

# 统计文件夹的大小信息(第一列标示该目录下总文件大小,第二列标示该目录下所有文件在集群上的总存储大小和你的副本数相关,第二列内容=文件大小*副本数)
hadoop fs -du -s -h /hadoop/test
62  62  /hadoop/test

hadoop fs -du -h /hadoop/test
62  62  /hadoop/test/hadoop_test.txt

# 设置HDFS中文件的副本数量(注意:这里设置的副本数量只是记录在NameNode的元数据中的，是否真的会有这么多副本，还得看DataNode的数量，比如我只有3台Data)
hadoop fs -setrep 10 /hadoop/test/test.txt
```

​	`hadoop`文件系统中文件默认放置位置：`/usr/local/hadoop/hdfs/data/current/BP-2106138772-172.22.43.97-1620789083115/current/finalized/subdir0/subdir0`



## ide远程访问hadoop

> 环境：Centos7.6，已经部署好了hadoop3.1.4

注意：

1. 服务器上的hadoop的环境变量一定要先配置好，确保hdfs命令可以使用！
2. 配置好本地的hadoop环境变量，不需要配置其中的文件，不需要启动，解压缩以后，只配置环境变量就可以了(经过本人的多次尝试，发现需要特定的`bin`文件夹，所以就找了一份，因为网上的不好找，如有需要的请私聊我！)

在ide中创建一个空的工程，在配置文件中添加一下的配置

```xml
<dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client-minicluster</artifactId>
            <version>3.1.4</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.10.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>2.10.1</version>
        </dependency>
    </dependencies>
```

在main中创建一个类，代码如下：

```java
import org.apache.log4j.BasicConfigurator;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;

public class HDFSFileIfExist {
    public static void main(String[] args){
        try {
            BasicConfigurator.configure();
            String fileName = "/home/test.txt";
            Configuration conf = new Configuration();
            conf.set("fs.defaultFS", "hdfs://localhost:9000");
            conf.set("fs.hdfs.impl", "org.apache.hadoop.hdfs.DistributedFileSystem");
            FileSystem fs = FileSystem.get(conf);
            if(fs.exists(new Path(fileName))){
                System.out.println("文件存在");
            }else{
                System.out.println("文件不存在");
            }
        }
        catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

其中，`/home/test.txt`是服务器上的路径，可以随意修改，localhost改成自己的服务器的ip地址即可

这样，一个简单的hadoop查询功能便做好了