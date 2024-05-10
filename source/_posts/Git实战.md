---
title: Git实战
tags:
  - Git
categories:
  - Git
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: ff99adbe
date: 2024-04-22 16:23:38
---

# Git实战

​	先把文件添加到缓存区，从这一步开始，我们就要防止出现冲突，为了防止出现冲突，我们只需要将自己修改过的文件，添加到缓存区即可，例如：

```shell
# 将某个文件夹中的某个文件添加到缓存区
git add <文件路径/文件名>

# 将某个文件夹中所有文件添加到缓存区
git add <文件夹名称>
```

​	然后正常`commit`即可，命令为：

```shell
git commit -m "本次提交的修改"
```

​	如果知道远程代码已经被同事修改，我们就需要多一步操作了，先将代码拉到本地，合并后在提交，命令为：

```shell
# 先将代码拉到本地，--allow-unrelated-histories表示允许存在不同的历史提交
git pull origin master --allow-unrelated-histories
```

​	此时，我们就已经将代码拉到本地了，如果有冲突，就会有提示，此时我们可以用下面命令来查看哪些文件有冲突：

```shell
git status
```

​	此时，我们已经看到了有冲突的文件，接下来就需要修改了，如果是想要远程代码覆盖本地，只需要将远程代码`copy`过来即可，如果想要本地代码覆盖远程，可再次使用`git add <文件路径/文件夹/文件名>`即可，命令为：

```shell
git push -u origin "master"
```

​	至此，冲突解决完成。
