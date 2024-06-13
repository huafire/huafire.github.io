---
title: springboot中feign调用微服务
tags:
  - java
  - springboot
  - 微服务
categories:
  - java
  - springboot
cover: /image/top_img.jpg
top_img: /image/top_img.jpg
abbrlink: 332af4c7
date: 2024-05-27 10:45:11
---

## `springboot`中`feign`调用微服务

​	在`springcloud`项目中，有可能遇到以下这种情况，需要跨模块调用接口，来实现某一个操作。

​	举个"栗子"：需要在某个模块（简称A模块）中调用权限认证模块（简称B模块）中的用户表信息，此时，可以使用`springboot`的微服务来实现，微服务的实现方法有三种，这里主要介绍通过`Feign`访问，具体步骤如下：

​	前置步骤：

- 需要的依赖为：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

- 在`A`模块中的启动类上，加入`@EnableFeignClients`注解，如下：

  ```java
  @SpringBootApplication
  @EnableFeignClients
  public class DemoApplication {
      public static void main(String[] args) {
          SpringApplication.run(DemoApplication.class, args);
      }
  }
  ```

### 1、实现接口与业务

​	首先被调用的模块（`B`模块）中需要实现接口，但是在实现接口的同时，我们需要加上一个新的注解，用来进行内部调用时，免认证操作，具体实现如下：

```java
@Inner
@PostMapping("/")
public String UserAuth(String test) {
    // 调用业务层方法，实现需求
	return "test";
}
```

​	**注意**：在业务中正常使用时，一定要问清楚，在某个项目中，是否可以使用`@Inner`注解，如果自定义了其他的注解，需要按需使用规定的内部免认证注解。

​	`B`模块操作完成。

### 2、创建Feign客户端接口

​	在调用的模块（`A`模块）中，创建一个`Feign`客户端接口来定义远程服务的调用。使用`@FeignClient`注解来指定服务名称和可选的回退机制。

```java
// RemoteServiceFallbackFactory是自己编写的回退工厂机制，用于报警降级处理，可以优雅的处理异常，其他参数可以根据自己需要填写
@FeignClient(name = "remote-service", url = "http://localhost:8081", fallbackFactory = RemoteServiceFallbackFactory.class)
public interface RemoteServiceClient {
    // 注意此时的PostMapping与上方的PostMapping相对应，注意输入正确的路径
    @PostMapping("/")
    // 此时的方法与上面的方法相对应
    String UserAuth(String test);
}
```

### 3、实现`FallbackFactory`回退机制

​	模块同样为`A`模块，可以在`Feign`调用出现异常时，进行异常处理。

```java
@Component
public class RemoteServiceFallbackFactory implements FallbackFactory<RemoteServiceClient> {
    @Override
    public RemoteServiceClient create(Throwable cause) {
        return new RemoteServiceClient() {
            @Override
            // 这里与上面的方法名，参数相同
            public String getData(String id) {
                // 一般可以写入日志
                System.err.println("Remote service failed: " + cause.getMessage());
                // 返回回退数据
                return "Fallback response for id: " + id;
            }
        };
    }
}
```

### 4、调用微服务

​	此时，微服务已经完成，可以引入文件并进行调用（这就不用问了，那肯定是`A`模块）。

```java
@RestController
public class DemoController {

    @Resource
    private RemoteServiceClient remoteServiceClient;

    @GetMapping("/fetch-data/{id}")
    public String fetchData(@PathVariable("id") String id) {
        return remoteServiceClient.getData(id);
    }
}
```

​	这里只有一个需要注意的地方，在你看到这篇博客时，`@Autowired`早已经不再提供维护了，此时使用`@Autowired`注解，就会发现`ide`在报错，但实际上，测试后发现，并不会影响项目正常运行，`@Autowired`注解已经换成了`@Resource`注解，其中的变化就不在这里展开了。

​	**注1：在上面的步骤`2`，`3`，`4`中的文件均可放在模块`C`中，但是模块`A`需要继承模块`C`，好处为，每次遇到需要调用此微服务时，均可直接继承该模块，直接调用。**

​	**注2：`springboot`中尽量不要循环继承，例如，模块`A`继承了模块`C`，那么模块`C`就不要再继承模块`A`了，但是有时候，需要引用`A`的实例类，此时就需要直接将模块`A`中的实体类复制到模块`C`中。**

​	哦，有一个文件忘记说了，如果有报错的话（具体什么报错忘记了），可以试试在`application.properties`或`application.yml`文件中配置`Feign`和`Hystrix`（如果使用的话）：

```properties
# Feign配置
feign.hystrix.enabled=true
```

​	再见！
