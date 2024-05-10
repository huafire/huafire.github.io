---
title: 修复同时使用hexo-asset-image和hexo-abbrlink时导致图片加载失败的问题
tags:
  - hexo
  - hexo-asset-image
  - hexo-abbrlink
categories:
  - hexo
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: 81ea4a90
date: 2021-03-08 14:02:24
---

# 修复同时使用hexo-asset-image和hexo-abbrlink时导致图片加载失败的问题

> 转载自：
> 文章作者: [Valley](mailto:793986752@qq.com)
> 文章链接: http://example.com/posts/81ea4a90.html



​	当`hexo-abbrlink` 与 `hexo-asset-image` 一起用时图片展示出现问题，因为所需要的相对路径为 `abbrlink/`，但是编写的是 `title/`，那么需要实现类似重定向的功能即可解决问题，添加此功能的 `hexo-asset-image` 插件已经有人发布在 `GitHub` 上了

```bash
CODE
https://github.com/foreveryang321/hexo-asset-image
```

​	网上有些方法是修改`hexo-asset-image`的源码来解决的，但是有些繁琐，直接使用别人造好的轮子多香啊。

#### 步骤

​	卸载掉原来的`hexo-asset-image`

```bash
CODE
npm uninstall hexo-asset-image
```

​	安装新的插件

```bash
CODE
npm install https://github.com/foreveryang321/hexo-asset-image.git --save
```

​	然后一键三连

```bash
CODE
hexo clean && hexo g && hexo d
```

#### 用法

​	必需确认`_config.yml`文件中

```bash
CODE
post_asset_folder: true
```

​	可以使用下面几种方式来引入图片`logo.jpg`

```bash
CODE
![logo](logo.jpg)
![logo](MacGesture2-Publish/logo.jpg)
![logo](D:/MacGesture2-Publish/logo.jpg)
{% asset_img logo.jpg %}
```


