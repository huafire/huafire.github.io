---
title: hexo博客
tags:
  - hexo
categories:
  - hexo
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: 4f72e00a
date: 2021-07-11 13:43:51
---

```shell
# 初始化博客
hexo init

# 拉取 butterfly主题
git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
```

修改配置文件`_config.yml`中的主题为`butterfly`

```shell
# 安装一些必须的依赖
npm install --save hexo-renderer-jade hexo-generator-feed hexo-generator-sitemap hexo-browsersync hexo-generator-archive
```

修改`_config.yml`中的`post_asset_folder`属性以及 `permalink` 属性

```bash
# 是否启动资源文件夹，开启后通过 hexo new :title.md 生成新文章会建立一个同名的文件夹
post_asset_folder: true
# 生成文章链接的格式，这是默认的格式；修改的规则也比较简单，标签前面要加英文冒号；（注意图片资源生成的格式必须是这个格式，否则会出现图片加载失败的情况，可见下方第6条生成的图片资源的引入格式）
permalink: :year/:month/:day/:title/
```

```shell
# 安装显示图片的插件
npm install https://github.com/CodeFalling/hexo-asset-image --save
```

