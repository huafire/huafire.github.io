---
title: SpringBoot整合Mybatis-plus
tags:
  - java
  - SpringBoot
categories:
  - java
  - SpringBoot
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: c1b62771
date: 2021-08-28 11:08:32
---

# springboot整合mybatis-plus

## 1、导入依赖包

```xml
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-boot-starter</artifactId>
	<version>3.4.3</version>
</dependency>
```

## 2、编写实体类

```java
import lombok.Data;

@Data
public class User {

    private int id;

    private String username;

    private String password;

}
```

## 3、编写接口

```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.corhor.bean.User;
import org.springframework.stereotype.Repository;


@Repository
public interface UserMapper extends BaseMapper<User> {
}
```

## 4、编写测试类

```java
import com.corhor.bean.User;
import com.corhor.mapper.UserMapper;
import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

@RunWith(SpringRunner.class)
@SpringBootTest
class MybatisPlusApplicationTests {

    @Autowired
    private UserMapper userMapper;

    @Test
    void contextLoads() {
        List<User> users = userMapper.selectList(null);
        users.forEach(System.out::println);
    }
}
```

注意：

- 需要在启动类上加上`@MapperScan("com.xxx.mapper")`，`xxx`是路径。
- `mybatis-plus`可以少写一些简单的数据库查询方法，例如上面的`selectList`方法，但是这种方法并不多，感兴趣的小伙伴可以直接查看源码。

b站视频`Mybatis-plus`详解：https://www.bilibili.com/video/BV1NE411Q7Nx?p=1
