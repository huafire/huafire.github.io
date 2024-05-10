---
title: Mybatis框架之复杂映射&配置文件深入
tags:
  - javaWeb
  - Mybatis
categories:
  - javaWeb
  - Mybatis
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: d12da41c
date: 2021-04-24 17:03:31
---

>补充`sql`注入：
>
>​	简单的说，就是通过修改`sql`语句里的参数，来获取数据库里的一些数据。
>
>​	我们从数据库里查询数据时，我们有时候需要向`sql`语句中传入参数，如果直接通过拼接字符串的方式来拼接`sql`语句的话，那么别人就有可能通过参数，来直接获取到数据库里的一些原来无权访问的数据。



​		`sql`查询方式有好多种，但是个人感觉直接在程序里写死的情况是很少见的，这种数据我们可以直接放入`Redis`数据库中，不必放入`mysql`中。所以下面我就直接记录高级查询方法了。

### 多条件查询

需求：根据`id`和`username`查询`user`表

#### 	方式一：

​		使用`#{arg0}-#{argn}`或者`#{param1}-#{paramn}`获取参数

`UserMapper`接口：

```java
public interface UserMapper {
	public List<User> findByIdAndUsername1(Integer id, String username);
}
```

`UserMapper.xml`：

```xml
<mapper namespace="com.lagou.mapper.UserMapper">
	<select id="findByIdAndUsername1" resultType="user">
		<!-- select * from user where id = #{arg0} and username = #{arg1} -->
		select * from user where id = #{param1} and username = #{param2}
	</select>
</mapper>
```

测试：

```java
@Test
public void testFindByIdAndUsername() throws Exception {
    InputStream is = Resources.getResourceAsStream("mybatisConfig.xml");
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
    SqlSession sqlSession = sqlSessionFactory.openSession();
	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
	List<User> list = userMapper.findByIdAndUsername1(1 , "子慕");
	System.out.println(list);
}
```

#### 	方式二：

​		使用注解，引入@Param() 注解获取参数。

`UserMapper`接口：

```java
public interface UserMapper {
		public List<User> findByIdAndUsername2(@Param("id") Integer
id,@Param("username") String username);
}
```

`UserMapper.xml`文件：

```xml
<mapper namespace="com.lagou.mapper.UserMapper">
	<select id="findByIdAndUsername2" resultType="user">
select * from user where id = #{id} and username = #{username}
	</select>
</mapper>
```

测试类：

```java
@Test
public void testFindByIdAndUsername() throws Exception {
	InputStream is = Resources.getResourceAsStream("mybatisConfig.xml");
	SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession();
	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
	List<User> list = userMapper.findByIdAndUsername2(1, "子慕");
	System.out.println(list);
}
```

#### 	方式三：（推荐）

使用`pojo`对象传递参数

`UserMapper`接口：

```java
public List<User> findByIdAndUsername3(User user);
```

`UserMapper.xml`文件：

```xml
<mapper namespace="com.lagou.mapper.UserMapper">
	<select id="findByIdAndUsername3" parameterType="com.lagou.domain.User" resultType="com.lagou.domain.User">
		select * from user where id = #{id} and username = #{username}
	</select>
</mapper>
```

测试类：

```java
@Test
    public void testFindByIdAndUsername() throws Exception{
        InputStream is = Resources.getResourceAsStream("mybatisConfig.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        User param = new User();
        param.setId(1);
        param.setUsername("子慕");
        List<User> list = userMapper.findByIdAndUsername3(param);
        System.out.println(list);
    }
```



### 模糊查询：

​		根据`username`模糊查询`user`表

#### 方式一：

​		`UserMapper`接口：

```java
public List<User> findByUsername1(String username);
```

`UserMapper.xml`文件：

```xml
<mapper namespace="com.lagou.mapper.UserMapper">
	<select id="findByUsername1" parameterType="string" resultType="user">
		select * from user where username like #{username}
	</select>
</mapper>
```

测试类：

```java
@Test
    public void testFindByUsername1() throws Exception{
        InputStream is = Resources.getResourceAsStream("mybatisConfig.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        List<User> list = userMapper.findByUsername1("%子%");
        for (User user : list){
            System.out.println(user);
        }
    }
```

#### 方式二：（不推荐使用，会出现sql注入）

`UserMapper`接口：

```java
public List<User> findByUsername2(String username);
```

`UserMapper.xml`文件：

```xml
<mapper namespace="com.lagou.mapper.UserMapper">
	<!--不推荐使用，因为会出现sql注入问题-->
	<select id="findByUsername2" parameterType="string" resultType="user">
		select * from user where username like '${value}'
	</select>
</mapper>
```

测试类：

```java
@Test
    public void testFindByUsername2() throws Exception{
        InputStream is = Resources.getResourceAsStream("mybatisConfig.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        List<User> list = userMapper.findByUsername2("%子%");
        for (User user : list){
            System.out.println(user);
        }
    }
```

#### ${} 与 #{} 区别【笔试题】

#{} :表示一个占位符号

- 通过#{} 可以实现preparedStatement向占位符中设置值，自动进行java类型和jdbc类型转换，#{}可以有效防止sql注入。

- #{} 可以接收简单类型值或pojo属性值。

- 如果parameterType传输单个简单类型值， #{} 括号中名称随便写。

  

${} :表示拼接sql串

  - 通过${} 可以将parameterType 传入的内容拼接在sql中且不进行jdbc类型转换，会出现sql注入
    问题。
  - ${} 可以接收简单类型值或pojo属性值。
  - 如果parameterType传输单个简单类型值， ${} 括号中只能是value。
  - 补充：TextSqlNode.java 源码可以证明

### Mybatis映射文件深入

#### 返回主键

应用场景

​		我们很多时候有这种需求，向数据库插入一条记录后，希望能立即拿到这条记录在数据库中的主键值。

#### `useGeneratedKeys`：

```java
public interface UserMapper {
	// 返回主键
	public void save(User user);
}
```

```xml
<!--
		useGeneratedKeys="true" 声明返回主键
		keyProperty="id" 把返回主键的值，封装到实体的id属性中
		注意：只适用于主键自增的数据库，mysql和sqlserver支持，oracle不支持
-->
<insert id="save" parameterType="user" useGeneratedKeys="true" keyProperty="id">
	INSERT INTO `user`(username,birthday,sex,address)
		values(#{username},#{birthday},#{sex},#{address})
</insert>
```

**注意：只适用于主键自增的数据库，`mysql`和`sqlserver`支持，`oracle`不行。**

#### `selectKey`：

```java
public interface UserMapper {
	// 返回主键
	public void save(User user);
}
```

```xml
<!--
	selectKey 适用范围广，支持所有类型数据库
	keyColumn="id" 指定主键列名
	keyProperty="id" 指定主键封装到实体的id属性中
	resultType="int" 指定主键类型
	order="AFTER" 设置在sql语句执行前（后），执行此语句
-->
<insert id="save" parameterType="user">
	<selectKey keyColumn="id" keyProperty="id" resultType="int" order="AFTER">
		SELECT LAST_INSERT_ID();
	</selectKey>
		INSERT INTO `user`(username,birthday,sex,address)
			values(#{username},#{birthday},#{sex},#{address})
</insert>
```

测试类：

```java
@Test
public void testSave() throws Exception {
	InputStream is = Resources.getResourceAsStream("mybatisConfig.xml");
	SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession();
	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
	User user = new User();
	user.setUsername("子慕");
	user.setAddress("北京");
	user.setBirthday(new Date());
	user.setSex("男");
	userMapper.save(user);
	System.out.println("返回主键:" + user.getId());
}
```



### 动态`Sql`：

​	需求：根据`id`和`username`查询，但是不确定两个都有值。

#### 动态`Sql`之`<if>`标签

`UserMapper`接口：

```java
public List<User> findByIdAndUsernameIf(User user);
```

`UserMapper.xml`映射：

```xml
<!--
	where标签相当于 where 1=1，但是如果没有条件，就不会拼接where关键字
-->
<select id="findByIdAndUsernameIf" parameterType="user" resultType="user">
	SELECT * FROM `user`
	<where>
		<if test="id != null">
			AND id = #{id}
		</if>
		<if test="username != null">
			AND username = #{username}
		</if>
	</where>
</select>
```

测试类：

```java
// if标签 where标签
@Test
public void testFindByIdAndUsernameIf() throws Exception {
    InputStream is = Resources.getResourceAsStream("mybatisConfig.xml");
	SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession();
	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
	User param = new User();
	// param.setId(42);
	// param.setUsername("王小二");
	List<User> list = userMapper.findByIdAndUsernameIf(param);
	System.out.println(list);
}
```

#### 动态`Sql`之`<set>`标签

`UserMapper`接口：

```java
public void updateIf(User user);
```

`UserMapper.xml`映射：

```xml
<!--
	set标签在更新的时候，自动加上set关键字，然后去掉最后一个条件的逗号
-->
<update id="updateIf" parameterType="user">
	UPDATE `user`
	<set>
		<if test="username != null">
			username = #{username},
		</if>
		<if test="birthday != null">
			birthday = #{birthday},
		</if>
		<if test="sex !=null">
			sex = #{sex},
		</if>
		<if test="address !=null">
			address = #{address},
		</if>
	</set>
	WHERE id = #{id}
</update>
```

测试类：

```java
// set标签
@Test
public void testUpdateIf()throws Exception{
    InputStream is = Resources.getResourceAsStream("mybatisConfig.xml");
	SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession();
	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
	User user = new User();
	user.setId(1);
	user.setUsername("小二王");
	user.setSex("女");
	userMapper.updateIf(user);
}
```

#### 动态`Sql`之`<foreach>`标签

​		**`foreach`主要是用来做数据的循环遍历**

​		例如： `select * from user where id in (1,2,3)`在这样的语句中，传入的参数部分必须依靠`foreach`遍历才能实现。

`<foreach>`标签用于遍历集合，它的属性：

- `collection`代表要遍历的集合元素
- `open`代表语句的开始部分
- `close`代表结束语句
- `item`代表遍历集合的每个元素，生成的变量名
- `sperator`代表分隔符

##### 集合

​	`UserMapper`接口：

```java
public List<User> findByList(List<Integer> ids);
```

​	`UserMaper.xml`映射：

```xml
<!--
	如果查询条件为普通类型 List集合，collection属性值为：collection 或者 list
-->
<select id="findByList" parameterType="list" resultType="user" >
	SELECT * FROM `user`
	<where>
		<foreach collection="collection" open="id in(" close=")" item="id" separator=",">
		#{id}
		</foreach>
	</where>
</select>
```

测试类：

```java
// foreach标签 list
@Test
public void testFindByList() throws Exception {
    InputStream is = Resources.getResourceAsStream("mybatisConfig.xml");
	SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession();
	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
	List<Integer> ids = new ArrayList<>();
	ids.add(46);
	ids.add(48);
	ids.add(51);
	List<User> list = userMapper.findByList(ids);
	System.out.println(list);
}
```

##### 数组

`UserMapper`接口：

```java
public List<User> findByArray(Integer[] ids);
```

`UserMaper.xml`映射：

```xml
<!--
	如果查询条件为普通类型 Array数组，collection属性值为：array
-->
<select id="findByArray" parameterType="int" resultType="user">
	SELECT * FROM `user`
	<where>
	<foreach collection="array" open="id in(" close=")" item="id" separator=",">
		#{id}
	</foreach>
	</where>
</select>
```

测试类：

```java
// foreach标签 array
@Test
public void testFindByArray() throws Exception {
    InputStream is = Resources.getResourceAsStream("mybatisConfig.xml");
	SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
	SqlSession sqlSession = sqlSessionFactory.openSession();
	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
	Integer[] ids = {46, 48, 51};
	List<User> list = userMapper.findByArray(ids);
	System.out.println(list);
}
```

##### `Sql`片段：

应用场景：

​	映射文件中可将重复的`sql`提取出来，使用时用`include`引用即可，最终达到`sql`重用的目的。

```xml
<!--抽取的sql片段-->
<sql id="selectUser">
	SELECT * FROM `user`
</sql>

<select id="findByList" parameterType="list" resultType="user" >
	<!--引入sql片段-->
	<include refid="selectUser"></include>
	<where>
		<foreach collection="collection" open="id in(" close=")" item="id" separator=",">
			#{id}
		</foreach>
	</where>
</select>

<select id="findByArray" parameterType="integer[]" resultType="user">
	<!--引入sql片段-->
	<include refid="selectUser"></include>
	<where>
		<foreach collection="array" open="id in(" close=")" item="id" separator=",">
			#{id}
		</foreach>
	</where>
</select>
```

#### 小结

`Mybatis`映射文件配置：

```xml
<select>：查询
<insert>：插入
<update>：修改
<delete>：删除
<selectKey>：返回主键
<where>：where条件
<if>：if判断
<foreach>：for循环
<set>：set设置
<sql>：sql片段抽取
```

## Mybatis核心配置文件深入

### 	plugins标签

简介：`MyBatis`可以使用第三方的插件来对功能进行扩展，分页助手`PageHelper`是将分页的复杂操作进行封装，使用简单的方式即可获得分页的相关数据。

​	开发步骤：

1. 导入通用`PageHelper`的坐标。
2. 在`mybatis`核心配置文件中配置`PageHelper`插件
3. 测试分页数据获取

#### 导入通用`PageHelper`坐标

```xml
<!-- 分页助手 -->
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper</artifactId>
	<version>3.7.5</version>
</dependency>
<dependency>
	<groupId>com.github.jsqlparser</groupId>
	<artifactId>jsqlparser</artifactId>
	<version>0.9.5</version>
</dependency>
```

注意：`jsqlparser`需要用`0.9.5`版本，如果是使用`0.9.1`会直接报错。

#### 在`mybatis`核心配置文件中配置`PageHelper`插件

```xml
<!-- 分页助手的插件 -->
<plugin interceptor="com.github.pagehelper.PageHelper">
	<!-- 指定方言 -->
	<property name="dialect" value="mysql"/>
</plugin>
```

​	注意：`plugin`标签外应该有`plugins`标签，而且`plugins`标签应该写在`environments`标签前，不然会报如下错误！

​	报错：

```
The content of element type "configuration" must match "(properties?,settings?,typeAliases?,typeHandlers?,objectFactory?,objectWrapperFactory?,reflectorFactory?,plugins?,environments?,databaseIdProvider?,mappers?)".
```

#### 测试分页代码实现

```java
@Test
    public void test_select() {
        try {
            InputStream is = Resources.getResourceAsStream("mybatisConfig.xml");

            SqlSessionFactory build = new SqlSessionFactoryBuilder().build(is);

            SqlSession sqlSession = build.openSession();

            UserMapper mapper = sqlSession.getMapper(UserMapper.class);
            // 设置分页助手
            PageHelper.startPage(1,4);
            
            List<User> all = mapper.findAll();
            all.forEach(System.out::println);
            // 获得分页相关的其他参数，并且传入参数all
            PageInfo<User> pageInfo = new PageInfo<>(all);
            System.out.println("总条数："+pageInfo.getTotal());
            System.out.println("总页数："+pageInfo.getPages());
            System.out.println("当前页："+pageInfo.getPageNum());
            System.out.println("每页显示长度："+pageInfo.getPageSize());
            System.out.println("是否第一页："+pageInfo.isIsFirstPage());
            System.out.println("是否最后一页："+pageInfo.isIsLastPage());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

#### 获得分页相关的其他参数

```java
//其他分页的数据，select为对象参数
PageInfo<User> pageInfo = new PageInfo<User>(select);
System.out.println("总条数："+pageInfo.getTotal());
System.out.println("总页数："+pageInfo.getPages());
System.out.println("当前页："+pageInfo.getPageNum());
System.out.println("每页显示长度："+pageInfo.getPageSize());
System.out.println("是否第一页："+pageInfo.isIsFirstPage());
System.out.println("是否最后一页："+pageInfo.isIsLastPage());
```

#### 知识小结

`MyBatis`核心配置文件常用标签：

1. `properties`标签：该标签可以加载外部的`properties`文件
2. `typeAliases`标签：设置类型别名
3. `environments`标签：数据源环境配置标签
4. `plugins`标签：配置`MyBatis`的插件