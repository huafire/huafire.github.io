---
title: shell脚本初见
tags:
  - shell
  - 脚本
categories:
  - shell
  - 脚本
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: 4d566b3e
date: 2021-09-18 12:48:48
---

# shell脚本初见

​	脚本有两种形式，一种是正常的`shell`命令，另一种是二进制文件，目前对于我这种初学者来说，感觉区别不是太大。

​	`shell`命令形式是

```shell
#!/bin/sh
echo "Hello World"
```

​	二进制形式

```shell
#!/bin/bash
echo "Hello World"
```

- ​	二进制文件的权限较为严格，必须先赋权，命令如下：

  ```shell
  chmod +x <filename.sh>
  ```

​    个人比较喜欢用`shell`命令的格式，下面记录一些最基本的操作，如有不合适的地方，请留言

​	常规的命令就不多写了，例如`mkdir test`等命令直接写就ok。

​	请在操作前确定当前用户对想要操作的文件有操作权限！！！

## 替换文件

​	将整个文件中的内容都替换掉，请注意文件一旦被替换，其中的内容是无法找回的

```shell
echo 'Hello World'>/home/test.txt
```

​	注意后面的路径尽量用绝对路径，个人没有尝试过相对路径

## 在文件最后添加一行或者多行

```shell
echo 'Hello World'>>/home/test.txt
```

## 替换掉文件中特定内容

```shell
sed -i 's/aaaaaa/Hello World/g' /home/test.txt
```

​	其中，`aaaaaa`是我们想要的内容，`Hello World`是我们想要替换的内容

## 在文件头添加一行

​	注意：只能添加一行，添加多行会报错

```shell
sed -i '1i\Hello World !!' /home/test.txt
```



​	目前个人用到的只有这四种，如有需要，后期再添加，但是这四种操作，已经可以完成大部分的部署操作了。

​	`shell`脚本可以帮助我们避免重复的操作，在部署多台服务器的环境的时候，非常方便！
