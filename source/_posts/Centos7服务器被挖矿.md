---
title: Centos7服务器被挖矿
tags:
  - Centos
  - 防火墙
categories:
  - Centos
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: '79477970'
date: 2021-05-14 17:28:32
---

​	某天的一个早晨，打开电脑，忽然发现服务器`cpu`和内存基本跑满，我尝试着将所有进程都关掉，`cpu`由100变成了97，好家伙，我开始尝试着抓包，又问了度娘，结果度娘告诉我我的服务器很可能中了挖矿病毒...这是一个悲伤的故事...

​	由于本人太菜，找不到占用大量`cpu`的进程在哪里，所以只能先在阿里云的控制台上关掉了所有自定义端口，然后保存数据后，重装系统...

​	由于一开始配置了`hadoop`，懒得配置防火墙，就直接把防火墙关掉了，应该就是这样了...所以各位童鞋一定不要学我

​	一定一定不要嫌麻烦，一定要开启防火墙！！！



​	防火墙命令如下：

```shell
# 打开防火墙
systemctl start firewalld

# 关闭防火墙
systemctl stop firewalld

# 查看防火墙状态
systemctl status firewalld

# 设置开机启动
systemctl enable firewalld

# 停止并禁用开机启动
sytemctl disable firewalld

# 重启防火墙
firewall-cmd --reload
```

```shell
# 添加80端口（--permanent 为永久生效）
firewall-cmd --zone=public --add-port=80/tcp --permanent 

# 更新防火墙规则
firewall-cmd --reload

# 查看端口状态
firewall-cmd --zone=public --query-port=80/tcp

# 删除开放的端口
firewall-cmd --zone=public --remove-port=80/tcp --permanent

# 查看所有开启的端口
firewall-cmd --zone=public --list-ports

# 拒绝所有包
firewall-cmd --panic-on

# 取消拒绝状态
firewall-cmd --panic-off

# 更新防火墙规则
firewall-cmd --reload
```

​	**注意**：**每次都更改防火墙规则，都需要重新更新：`firewall-cmd --reload`，更新状态**

