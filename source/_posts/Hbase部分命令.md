---
title: Hbase部分命令
tags:
  - hbase
categories:
  - hbase
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: 15d0cd02
date: 2021-11-07 17:23:46
---

# Hbase部分命令

```shell
# 进入hbase shell
hbase shell

# 新建表
create 'behavior', 'pc', 'ph'

# 查看所有表
list

# 查看表结构
describe 'behavior'

# 将数据添加到表中
put 'behavior','zhang_2020101021723_51001_1','pc:v','1001'
put 'behavior','wang_2020101121723_51001_1','pc:v','1002'
put 'behavior','zhao_2020101221723_51001_1','ph:v','1003'

# 将第二条数据的值改为1004
put 'behavior', 'wang_2020101121723_51001_1', 'pc:v','1004'

# 删除第三条数据
deleteall 'behavior','zhao_2020101221723_51001_1'

# 清空表数据
truncate 'behavior'

# 禁用表，然后删除表
disable 'behavior'
drop 'hehavior'
```

