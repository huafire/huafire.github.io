---
title: hadoop打jar包
tags:
  - hadoop
categories:
  - hadoop
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: 58e1e24f
date: 2021-07-11 13:50:27
---

​	默认情况下，在`Maven`中`Lifecycle`中有`package`这个选项，我们可以直接双击来进行`java`项目的打包。

## 报错：Retrying connect to server: 0.0.0.0/0.0.0.0:8032

```shell
[root@hua hadoop]# hadoop jar java_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar mapreduce.MyDriver
2021-07-05 19:28:42,671 INFO client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
2021-07-05 19:28:43,924 INFO ipc.Client: Retrying connect to server: 0.0.0.0/0.0.0.0:8032. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
2021-07-05 19:28:44,925 INFO ipc.Client: Retrying connect to server: 0.0.0.0/0.0.0.0:8032. Already tried 1 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
2021-07-05 19:28:45,925 INFO ipc.Client: Retrying connect to server: 0.0.0.0/0.0.0.0:8032. Already tried 2 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)
2021-07-05 19:28:46,926 INFO ipc.Client: Retrying connect to server: 0.0.0.0/0.0.0.0:8032. Already tried 3 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=10, sleepTime=1000 MILLISECONDS)

```

解决方法：

```shell
start.yarn.sh
```





## 报错代码127

```shell
[root@hua hadoop]# hadoop jar java_hadoop-1.0-SNAPSHOT-jar-with-dependencies.jar mapreduce.MyDriver
2021-07-05 19:35:59,384 INFO client.RMProxy: Connecting to ResourceManager at /0.0.0.0:8032
2021-07-05 19:35:59,864 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
2021-07-05 19:35:59,877 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/root/.staging/job_1625484948353_0001
2021-07-05 19:36:00,259 INFO input.FileInputFormat: Total input files to process : 1
2021-07-05 19:36:00,698 INFO mapreduce.JobSubmitter: number of splits:1
2021-07-05 19:36:00,817 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1625484948353_0001
2021-07-05 19:36:00,818 INFO mapreduce.JobSubmitter: Executing with tokens: []
2021-07-05 19:36:00,953 INFO conf.Configuration: resource-types.xml not found
2021-07-05 19:36:00,953 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2021-07-05 19:36:01,147 INFO impl.YarnClientImpl: Submitted application application_1625484948353_0001
2021-07-05 19:36:01,179 INFO mapreduce.Job: The url to track the job: http://hua:8088/proxy/application_1625484948353_0001/
2021-07-05 19:36:01,180 INFO mapreduce.Job: Running job: job_1625484948353_0001
2021-07-05 19:36:06,220 INFO mapreduce.Job: Job job_1625484948353_0001 running in uber mode : false
2021-07-05 19:36:06,221 INFO mapreduce.Job:  map 0% reduce 0%
2021-07-05 19:36:06,231 INFO mapreduce.Job: Job job_1625484948353_0001 failed with state FAILED due to: Application application_1625484948353_0001 failed 2 times due to AM Container for appattempt_1625484948353_0001_000002 exited with  exitCode: 127
Failing this attempt.Diagnostics: [2021-07-05 19:36:05.216]Exception from container-launch.
Container id: container_1625484948353_0001_02_000001
Exit code: 127

[2021-07-05 19:36:05.218]Container exited with a non-zero exit code 127. Error file: prelaunch.err.
Last 4096 bytes of prelaunch.err :
Last 4096 bytes of stderr :
/bin/bash: /bin/java: No such file or directory


[2021-07-05 19:36:05.218]Container exited with a non-zero exit code 127. Error file: prelaunch.err.
Last 4096 bytes of prelaunch.err :
Last 4096 bytes of stderr :
/bin/bash: /bin/java: No such file or directory


For more detailed output, check the application tracking page: http://hua:8088/cluster/app/application_1625484948353_0001 Then click on links to logs of each attempt.
. Failing the application.
2021-07-05 19:36:06,250 INFO mapreduce.Job: Counters: 0

```

解决方法：在`etc/hadoop/yarn-env.sh`中配置`java`变量

```shell
export $JAVA_HOME=/usr/local/java/jdk1.8.0_281
```

 

`hadoop`报错`/bin/java: No such file or directory`：

```shell
[2021-07-06 15:06:11.970]Container exited with a non-zero exit code 127. Error file: prelaunch.err.
Last 4096 bytes of prelaunch.err :
Last 4096 bytes of stderr :
/bin/bash: /bin/java: No such file or directory


[2021-07-06 15:06:11.970]Container exited with a non-zero exit code 127. Error file: prelaunch.err.
Last 4096 bytes of prelaunch.err :
Last 4096 bytes of stderr :
/bin/bash: /bin/java: No such file or directory
```

解决：

```shell
ln -s /usr/local/jdk1.8.0_112/bin/java /bin/java
```





## `hadoop`报错：无法找到或加载主类 `org.apache.hadoop.mapreduce.v2.app.MRAppMaster`

解决：

运行命令：

```shell
hadoop classpath
```

将自己的命令行中的输出内容复制到`yarn-site.xml`中：

**注意：一定是自己的输出内容**

```xml
<property>
                <name>yarn.application.classpath</name>
                <value>/usr/local/hadoop/etc/hadoop:/usr/local/hadoop/share/hadoop/common/lib/*:/usr/local/hadoop/share/hadoop/common/*:/usr/local/hadoop/share/hadoop/hdfs:/usr/local/hadoop/share/hadoop/hdfs/lib/*:/usr/local/hadoop/share/hadoop/hdfs/*:/usr/local/hadoop/share/hadoop/mapreduce/lib/*:/usr/local/hadoop/share/hadoop/mapreduce/*:/usr/local/hadoop/share/hadoop/yarn:/usr/local/hadoop/share/hadoop/yarn/lib/*:/usr/local/hadoop/share/hadoop/yarn/*</value>
        </property>
```

