---
title: 初见Vue
tags:
  - Vue
categories:
  - Vue
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: f23d18c8
date: 2021-08-22 15:56:01
---

# 初见`Vue`

## 安装`vue`脚手架

```shell
npm install -g @vue/cli
```



## 新建`vue`项目

```shell
vue create <project name>
```

> 第一次创建需要挺久，看个人网速

创建项目加速

```shell
npm config set registry https://registry.npm.taobao.org
```

报错：

```shell
 ERROR  command failed: npm install --loglevel error
```

解决方法：

打开`C:\Users\admin.vuerc`文件，没有的直接新建一个

```vuerc
{
  "useTaobaoRegistry": true,//将这里的true修改为false,vue创建项目就能成功了
  "presets": {
    "my-project": {
      "useConfigFiles": true,
      "plugins": {
        "@vue/cli-plugin-babel": {},
        "@vue/cli-plugin-router": {
          "historyMode": false
        },
        "@vue/cli-plugin-eslint": {
          "config": "standard",
          "lintOn": [
            "save"
          ]
        }
      }
    }
  }
}
```





# Vite+Vue3+ElementPlus

1. ## 创建项目

   ```shell
   npm init @vitejs/app demo --template vue
   ```

2. ## 安装`ElementPius`

   ```shell
   npm install element-plus --save
   ```

3. ## 引入`ElementPlus`

   ```js
   import { creatAPP } from 'vue'
   import Element from 'element-plus'
   import 'element-plus/lib/theme-chalk/index.css'
   import App from './App.vue'
   
   creatApp(App)
   	.use(ElementPlus)
   	.mount('#app')
   ```

   
