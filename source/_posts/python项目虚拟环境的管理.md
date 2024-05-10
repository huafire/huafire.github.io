---
title: python项目虚拟环境的管理
tags:
  - python
categories:
  - python
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: '83572680'
date: 2024-04-01 11:35:25
---

## python项目虚拟环境的管理

​	python库一般不安装在全局环境，一般虚拟机环境更安全。

​	新建一个项目时候，应该新建一个虚拟环境，来保证项目的稳定性。

​	在刚拿到一个没有虚拟环境的项目时，应该新建一个虚拟环境，来保证项目的稳定性，在`pycharm`中，`File->Settings->Project: project_name->Python Interpreter`找到`Add Interpreter`，从而新建一个新的虚拟环境。

​	**激活虚拟环境：** 打开命令行界面（如终端或命令提示符），然后导航到你的项目目录。进入到项目目录后，激活虚拟环境。此时项目中已经有一个虚拟环境，可以执行下面命令来激活虚拟环境：

在 Windows 上：

```bash
.\venv\Scripts\activate
```

在 macOS/Linux 上：

```bash
source venv/bin/activate
```

​	**安装库：** 激活虚拟环境后，你可以使用 `pip` 命令安装你需要的库。比如，要安装名为 `requests` 的库，可以执行：

```bash
pip install requests
```

​	**退出虚拟环境：** 当你完成项目工作后，可以使用以下命令退出虚拟环境：

```bash
deactivate
```

这将会停用虚拟环境，并回到系统的默认 Python 环境（全局环境）。
