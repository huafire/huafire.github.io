---
title: SpringBoot写入MongoDB数据库
tags:
  - java
  - springboot
categories:
  - java
  - springboot
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: 1defbece
date: 2024-05-28 10:51:40
---

## `SpringBoot`写入`MongoDB`数据库

​	基本配置是一样的，首先引入依赖，然后写好配置文件。

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

```yml
# MongoDB配置
data:
  mongodb:
    authentication-database: admin
    host: 127.0.0.1
    port: 27017
    database: test
    username: 
    password: 
```

### 1、`MongoDB`数据库插入数据

​	`MongoDB`数据库没有固定的表格式，可以直接进行插入操作，在插入时，每一条数据格式应为`Map`类型的数据，每一个`key`对应的均为表中的列名，例如：

​	插入前数据为：

```json
{
  "test": 1,
}
```

​	插入后的表为：

<img src="E:/github_Hexo/source/_posts/SpringBoot写入MongoDB数据库/1.png" alt="image-20240528105241652" style="zoom:100%;" />

​	其中，`_id`为`MongoDB`自己生成。

​	`spring-boot-starter-data-mongodb`依赖中，提供了`MongoTemplate`类，可以直接调用其中的`insert`语句进行插入操作，`insert`语句中，一般为两个参数，第一个参数为`Map`类型数据，第二个参数为表名（String）。

未完持续...
