---
title: SpringBoot数据访问
tags:
  - java
  - SpringBoot
categories:
  - SpringBoot
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: 16cf83b7
date: 2021-08-02 17:17:36
---

	`SpringData`是`Spring`提供的一个用于简化数据库访问、支持云服务器的开原框架。`SpringData`提供了多种类型数据库支持，`SpringBoot`对`SpringData`支持的数据库进行了整合管理，提供了各种依赖启动器。如下表：

| 名称                               | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| `spring-boot-starter-data-jpa`     | `Spring Data JPA`与`Hibernate`启动器                         |
| `spring-boot-starter-data-mongodb` | `MongoDB`和`Spring Data MongoDB`的启动器                     |
| `spring-boot-starter-data-neo4j`   | `Neo4j`图数据库和`Spring Data Neo4j`的启动器                 |
| `spring-boot-starter-data-redis`   | `Redis`键值数据存储与`Spring Data Redis`和`Jedis`客户端的启动器 |

​	需要说明的是，`MyBatis`作为操作数据库的流行框架，`SpringBoot`没有提供`MyBatis`场景依赖，但是`MyBatis`开发团队自己适配了`SpringBoot`，提供了`mybatis-spring-boot-starter`依赖启动器实现数据访问操作。

## `SpringBoot`整合`Mybatis`

### 基础环境搭建

​	实现`SpringBoot`与数据访问层框架（例如：`MyBatis`）的整合非常简单，主要是引入对应的依赖启动器，具体步骤如下：

1. 数据准备

   ​	在`MySQL`数据库中，创建一个`springBoot_learn`的数据库，并创建两张表`t_article`和`t_comment`，并放入几条测试数据

   ```sql
   # 创建数据库
   CREATE DATABASE springboot_learn;
   
   # 选择使用数据库
   USE springboot_learn;
   
   # 创建表t_article并插入数据
   USE springboot_learn;
   DROP TABLE
   IF
   	EXISTS `t_article`;
   CREATE TABLE `t_article` (
   	`id` INT ( 20 ) NOT NULL AUTO_INCREMENT COMMENT '文章id',
   	`title` VARCHAR ( 200 ) DEFAULT NULL COMMENT '文章标题',
   	`content` LONGTEXT COMMENT '文章内容',
   	PRIMARY KEY ( `id` ) 
   ) ENGINE = INNODB AUTO_INCREMENT = 2 DEFAULT CHARSET = utf8mb4;
   INSERT INTO `t_article`
   VALUES
   	( '1', 'SpringBoot基础入门', '从入门到精通讲解...' );
   INSERT INTO `t_article`
   VALUES
   	( '2', 'SpringBoot基础入门', '从入门到精通讲解...' );
   
   # 创建表t_comment并插入相关数据
   USE springboot_learn;
   DROP TABLE
   IF
   	EXISTS `t_comment`;
   CREATE TABLE `t_comment` (
   	`id` INT ( 20 ) NOT NULL AUTO_INCREMENT COMMENT '评论id',
   	`content` LONGTEXT COMMENT '评论内容',
   	`author` VARCHAR ( 200 ) DEFAULT NULL COMMENT '评论作者',
   	`a_id` INT ( 20 ) DEFAULT NULL COMMENT '关联的文章id',
   	PRIMARY KEY ( `id` ) 
   ) ENGINE = INNODB auto_increment = 3 DEFAULT CHARSET = utf8mb4;
   INSERT INTO `t_comment`
   VALUES
   	( '1', '很全、很详细', '狂奔的蜗牛', '1' );
   INSERT INTO `t_comment`
   VALUES
   	( '2', '赞一个', 'tom', '1' );
   INSERT INTO `t_comment`
   VALUES
   	( '3', '很详细', 'kitty', '1' );
   INSERT INTO `t_comment`
   VALUES
   	( '4', '很好，非常详细', '张三', '1' );
   INSERT INTO `t_comment`
   VALUES
   	( '5', '很不错', '张扬', '2' );
   ```

   ​	上面的`sql`语句中，先创建了一个数据库`springboot_learn`，然后创建了两个表`t_article`和`t_comment`，并向表中插入数据。其中，评论表`t_comment`的`a_id`与文章表`t_article`的主键`id`相关联。

   

2. 创建项目，引入相应的启动器

   1. 在依赖中引入`MySQL`和`MyBatis`依赖，其中`MySQL`是为了提供`MySQL`数据库连接驱动，`MyBatis`则是为了提供`MyBatis`框架来操作数据库。

      ```xml
      <!--    数据库连接池 jdbc-->
          <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
          </dependency>
      
          <!-- https://mvnrepository.com/artifact/org.mybatis.spring.boot/mybatis-spring-boot-starter -->
          <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.3</version>
          </dependency>
      ```

   2. 编写数据库表对应的实体类。在`com.learn.bean`包下编写`Comment`和`Article`实体类，分别对应数据库表`t_comment`和`t_article`，内容如下：

      `Comment.java`：

      ```java
      import lombok.Data;
      
      @Data
      public class Comment {
          private Integer id;
          private String content;
          private String author;
          private Integer aId;
      }
      ```

      `Article.java`：

      ```java
      import lombok.Data;
      import java.util.List;
      
      @Data
      public class Article {
          private Integer id;
          private String title;
          private String content;
          private List<Comment> commentList;
      }
      ```

   

3. 编写配置文件

   1. 在`application.properties`配置文件中进行数据库连接配置。打开全局配置文件`application.properties`，在配置文件中编写对用的`MySQL`数据库连接配置，内容如下：

      ```properties
      # MySQL数据库连接配置
      spring.datasource.url=jdbc:mysql://localhost:3306/springboot_learn?serverTimezone=UTC
      spring.datasource.username=root
      spring.datasource.password=root
      ```

   2. 数据源类型选择配置。`SpringBoot 1.x`版本默认使用的是`tomcat.jdbc`数据源，`SpringBoot 2.x`版本默认使用的是`hikari`数据源，如果需要其他的数据源，需要额外配置。

      ​	这里使用的是阿里巴巴的`Druid`数据源，在`pom.xml`文件中添加`Druid`数据源的依赖启动器，代码如下：

      ```xml
      <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
      </dependency>
      ```

      ​	上面引入依赖`druid-spring-boot-starter`，同样是阿里巴巴为了迎合`SpringBoot`项目而适配的`Druid`数据源启动器，当在`pom.xml`文件中引入了该启动器后，不需要再进行其他额外配置，`SpringBoot`项目会自动识别配置`Druid`数据源。

      ​	要说明的是，上面配置的`Druid`数据源启动器内部已经初始化了一些运行参数（例如：`initialSize`、`minldle`和`maxActive`等），如果开发时要修改第三方`Druid`的运行参数，则必须在全局配置文件中修改，如下：

      ```properties
      # 添加并配置第三方数据源Druid
      spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
      spring.datasource.druid.initial-size=20
      spring.datasource.druid.min-idle=10
      spring.datasource.druid.max-active=100
      ```

      ​	文件中修改了`Druid`数据源的类型、初始化连接数、最小空闲数和最大连接数属性。当然还可以设置更多属性，参考`Druid`属性设置。

      

### 使用注解的方式整合`MyBatis`

​	相比`Spring`与`MyBatis`的整合，`SpringBoot`与`MyBatis`的整合会使项目开发更加简便，同时还支持`XML`和注解两种配置方式。下面为注解的方式，步骤如下：

1. 创建`Mapper`接口文件，在`com.learn.mapper`包下，创建一个用于对数据库`t_comment`数据操作的接口`CommentMapper`，如下：

   ```java
   import com.kanghua.bean.Comment;
   import org.apache.ibatis.annotations.*;
   
   @Mapper
   public interface CommentMapper {
       @Select("SELECT * FROM t_comment WHERE id =#{id}")
       public Comment findById(Integer id);
       
       @Insert("INSERT INTO t_comment(content, author, a_id)" + "values (#{content}, #{author}, #{aId})")
       public int insertComment(Comment comment);
       
       @Update("UPDATE INTO t_comment SET content=#{content} WHERE id=#{id}")
       public int updateComment(Comment comment);
       
       @Delete("DELETE FROM t_comment WHERE id=#{id}")
       public int deleteComment(Integer id);   
   }
   ```

   ​	文件中，`@Mapper`注解表示该类是一个`MyBatis`接口文件，并保证能够被`SpringBoot`自动扫描到`Spring`容器中，在接口内部，分别通过`@Select`、`@Insert`、`@Update`、`@Delete`注解配合`SQL`语句完成了对数据库表`t_comment`数据的增删改查操作。

   > 上面文件中，在对应的接口类上添加了`@Mapper`注解，如果编写的`Mapper`接口过多时，需要重复为每一个接口文件添加`@Mapper`注解，为了避免这种麻烦，可以直接在`SpringBoot`项目启动器类上添加`@MapperScan("xxx")`注解，不需要再逐个添加`@Mapper`注解。`@MapperScan("xxx")`注解的作用和`@Mapper`注解类似，但是它必须指定需要扫描的具体包名，例如`@MapperScan("com.learn.mapper")`。

2. 编写单元测试进行接口方法测试。代码如下：

   ```java
   @Autowired
   private CommentMapper commentMapper;
   @Test
   public void selectComment(){
       Comment comment = commentMapper.findById(1);
       System.out.println(comment);
   }
   ```

   ​	先通过`@Autowired`注解将`CommentMapper`接口自动装配为`Spring`容器中的`Bean`，然后使用`@Test`注解标注`selectComment()`方法是单元测试方法，这里仅演示了`Mapper`接口中的数据查询。

3. 运行`selectComment()`方法，控制台输出结果。

   ```
   Comment(id=1, content=很全、很详细, author=狂奔的蜗牛, aId=null)
   ```

   我们会发现，`aId`的结果是`null`，并不是我们设置的1，这是因为编写的实体类`Comment`中使用了驼峰命令法，将`t_comment`表中的`a_id`字段设计成了`aId`属性，所以无法正确映射查询结果，为了解决上面的问题，可以在`application.properties`中添加一下配置：

   ```properties
   mybatis.configuration.map-underscore-to-camel-case=true
   ```

   再次测试即可得到结果：

   ```
   Comment(id=1, content=很全、很详细, author=狂奔的蜗牛, aId=1)
   ```

   

### 用配置文件的方式整合`MyBatis`

​	`SpringBoot`整合`MyBatis`时，不仅支持注解方式，还支持`XML`配置文件的方式，具体步骤如下：

1. 创建一个`Mapper`接口文件，在`com.learn.mapper`包中，创建一个操作数据表`t_article`的接口`ArticleMapper`，代码如下：

   ```java
   @Mapper
   public interface ArticleMapper {
       public Article selectArticle(Integer id);
       public int updateArticle(Article article);
   }
   ```

2. 创建`XML`映射文件，在`resource`目录下，创建一个统一管理映射文件的包`mapper`，并在该包下编写与`ArticleMapper`接口对应的映射文件`ArticleMapper.xml`，内容如下：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.kanghua.mapper.ArticleMapper">
       <select id="selectArticle" resultMap="articleWithComment">
           SELECT a.*,c.id c_id, c.content c_content, c.author
           FROM t_article a, t_comment c
           WHERE a.id=c.a_id AND a.id=#{id}
       </select>
   
       <resultMap id="articleWithComment" type="Article">
           <id property="id" column="id"/>
           <result property="title" column="title"/>
           <result property="content" column="content"/>
           <collection property="commentList" ofType="Comment">
               <id property="id" column="c_id"/>
               <result property="content" column="c_content"/>
               <result property="author" column="author"/>
           </collection>
       </resultMap>
   
       <update id="updateArticle" parameterType="Article">
           UPDATE t_article
           <set>
               <if test="title !=null and title != ''">
                   title=#{title}
               </if>
               <if test="content !=null and content !=''">
                   content=#{content}
               </if>
           </set>
           WHERE id=#{id}
       </update>
   </mapper>
   ```

   ​	上面代码中，`<mapper>`标签中的`namespace`属性值对用的是`ArticleMapper`接口文件全路径名称，在映射文件中根据`ArticleMapper`接口文件中的方法，编写两个对应的`SQL`语句，同时配置数据类型映射时，没有使用类的全路径名称，而是使用了类的别名，（例如，没有使用`com.learn.Article`而是使用了`Article`）。

3. 配置`XML`映射文件路径，我们在项目中编写的`XML`映射文件，`SpringBoot`无法扫描到自定义编写的`XML`配置文件，还必须在全局配置文件`application.properties`中添加`MyBatis`映射文件路径的配置，同时需要添加实体类别名映射路径，代码如下：

   ```properties
   # 配置MyBatis的XML配置文件路径
   mybatis.mapper-locations=classpath:mapper/*.xml
   # 配置XML映射文件中指定的实体类别名路径
   mybatis.type-aliases-package=com.bean
   ```

   ​	上面代码中，在使用配置文件方式整合`MyBatis`时，`MyBatis`映射文件路径的配置必不可少；而实体类别名的配置是根据前面创建的`XML`映射文件别名使用情况来确定的，如果`XML`映射文件中使用的都是类的全路径名称，则不需要配置。

4. 编写测试类。代码如下：

   ```java
   @Autowired
   private ArticleMapper articleMapper;
   @Test
   public void selectArticle(){
   Article article = articleMapper.selectArticle(1);
   System.out.println(article);
   }
   ```

控制台中打印出了下面的信息，说明配置完成。

```
Article(id=1, title=SpringBoot基础入门, content=从入门到精通讲解..., commentList=[Comment(id=1, content=很全、很详细, author=狂奔的蜗牛, aId=null), Comment(id=2, content=赞一个, author=tom, aId=null), Comment(id=3, content=很详细, author=kitty, aId=null), Comment(id=4, content=很好，非常详细, author=张三, aId=null)])
```

> ​	对于`SpringBoot`支持与`MyBatis`整合的两种方式而言，使用注解的方式比较合适简单的增删改查操作；而使用配置文件的方式稍微麻烦，但对于复杂的数据操作却显得比较实用。实际开发中，使用`SpringBoot`整合`MyBatis`进行项目开发时，通常会混合使用两种整合方式。
