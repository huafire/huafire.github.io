---
title: git报错failed to push some refs to
tags:
  - Git
categories:
  - Git
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: 323fc1e4
date: 2021-08-19 20:09:50
---

### 情况一：

```shell
To https://gitee.com/likanghua/springboot_test.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'https://gitee.com/likanghua/springboot_test.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

解决方法：

> 注意！！！
>
> 如果不是第一次提交代码，运行下面的命令前，一定要先备份！！！
>
> 因为同名文件会被远程的替换掉，远程没有的不会被替换。

```shell
git pull --rebase origin master
```

​	然后再次推送

```shell
git push origin master
```



### 情况二：

报错：

```shell
error: failed to push some refs to 'https://...'
```

如果只报错了上面的代码，是因为没有`commit`

解决方法：

```shell
# 注：xxx是本次提交的理由
git commit -m xxx
```

然后再次推送：

```shell
git push origin master
```



> ​	`git push origin master`命令将本地的`master`分支推送到`origin`主机
>
> ​	如果加上了`-u`参数，即`git push -u origin master`。`git`不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令为`git push`。
>
> ​	但是`git push`，默认只推送当前分支
