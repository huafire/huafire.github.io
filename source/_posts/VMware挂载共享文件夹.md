---
title: VMware挂载共享文件夹
tags:
  - VMware
  - 虚拟机
categories:
  - VMware
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: b71f1281
date: 2021-10-12 11:31:53
---

```shell
sudo /usr/bin/vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other -o uid=1000 -o gid=1000 -o umask=022
```

