---
title: Git报错
tags:
  - Git
categories:
  - Git
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: f04a0382
date: 2021-03-08 13:47:33
---

## Git 报错 nothing to commit, working tree clean

​	就在前两天，在我重新提交在本地部署好的博客时，git 报错 nothing to commit, working tree clean，我确定 git add . 和 git commit 都是正确的，那么问题来了，我分明有改动，为什么说我没有改动呢？

​	我运行 git branch 命令，返回结果如下：

```bash
E:\Hexo> git branch
* (no branch, rebasing hexo)
  hexo
  master
```

​	同时切换分支时和提交结果时报错如下：

```bash
E:\Hexo> git checkout master
  error: Your local changes to the following files would be overwritten by checkout:
        _config.yml
        source/_posts/***.md
        source/_posts/***.md
  Please commit your changes or stash them before you switch branches.
  Aborting
 
 E:\Hexo> git commit -m 处理分支错误
  [detached HEAD c0a7e36] 处理分支错误
   18 files changed, 475 insertions(+)
   create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\347\275\256.md"
   create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\347\275\256/1.jpg"
   create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\ create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\347\275\256/11.jpg"
   create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\347\275\256/12.jpg"
   create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\347\275\256/13.jpg"
  347\275\256/14.jpg"
   create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\347\275\256/15.jpg"
   create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\347\275\256/2.jpg"
   create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\347\275\256/3.jpg"
   create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\ create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\347\275\256/5.png"
   create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\347\275\256/6.png"
   create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\347\275\256/7.png"
   create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\347\275\256/8.png"
   create mode 100644 "source/_posts/Ubuntu\350\231\232\346\213\237\346\234\272\351\205\215\347\275\256/9.png"
   create mode 100644 source/_posts/Windows_Terminal.md
   create mode 100644 source/_posts/Windows_Terminal/1.png
  PS E:\Hexo> git push
  fatal: You are not currently on a branch.
  state now, use

    git push origin HEAD:<name-of-remote-branch>

E:\Hexo> git push origin HEAD:master
To https://gitee.com/likanghua/likanghua.git
 ! [rejected]        HEAD -> master (fetch first)
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
PS E:\Hexo> git pull
remote: Enumerating objects: 765, done.
remote: Compressing objects: 100% (233/233), done.
remote: Total 720 (delta 337), reused 592 (delta 256), pack-reused 0
Receiving objects: 100% (720/720), 4.60 MiB | 545.00 KiB/s, done.
Resolving deltas: 100% (337/337), completed with 15 local objects.
From https://gitee.com/likanghua/likanghua
   be15239..c22922a  master     -> origin/master
You are not currently on a branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

    git pull <remote> <branch>

PS E:\Hexo> git push origin HEAD:master
To https://gitee.com/likanghua/likanghua.git
 ! [rejected]        HEAD -> master (non-fast-forward)
error: failed to push some refs to 'https://gitee.com/likanghua/likanghua.git'
hint: Updates were rejected because a pushed branch tip is behind its remote
hint: counterpart. Check out this branch and integrate the remote changes
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
PS E:\Hexo> git checkout master
error: Your local changes to the following files would be overwritten by checkout:
        _config.yml
        source/_posts/Centos服务器配置.md
        source/_posts/Git命令.md
        source/_posts/Ubuntu虚拟机配置.md
Please commit your changes or stash them before you switch branches.
Aborting
```

​	可能会有小伙伴说，可以试试强行推送，我试了试，还是推不上去，而且，个人不建议强推。

## 解决方法如下：

​	先将代码从远程仓库里拉下来，然后跟自己修改后的对比，比较一下缺少哪一部分，再一点一点的补，切记，一定一定不要着急，慢慢补，只要修改不是特别特别的大，用不了半个小时就好了。



## git切换分支报错：

```bash
error: The following untracked working tree files would be overwritten by checkout:
```

强行切换会将一些文件删掉，注意保存文件！！

```bash
git checkout -f hexo
```

`hexo`为我想切换的分支，切换完以后，再将文件重新复制，粘贴进去，再次推送，一键三连：

```bash
git add .
git commit -m 提交
git push
```

搞定！！