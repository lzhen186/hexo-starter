---
title: SpringBoot 16.响应式框架webflux 16.1.webflux快速入门
date: 2022-07-04T00:18:57.142Z
tags: [springboot]
---
# 一、Reactive Stack简介

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200507101657.png)
由上图对比可以看出，Spring boot2.0推出了全新的开发技术栈：reactive，相对于Servlet Stack技术栈，它的最大优势在于：
使用了非阻塞的编程模型，能够有效地利用下一代的多核CPU处理器，处理海量的并发连接。
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200507101817.png)
boosGroup用于Accetpt连接建立事件并分发请求， workerGroup用于处理I/O读写事件和业务逻辑
每个Boss NioEventLoop循环执行的任务包含3步：

- 1 轮询accept事件
- 2 处理accept I/O事件，与Client建立连接，生成NioSocketChannel，并将NioSocketChannel注册到某个Worker NioEventLoop的Selector上
- 3 处理任务队列中的任务，runAllTasks。任务队列中的任务包括用户调用eventloop.execute或schedule执行的任务，或者其它线程提交到该eventloop的任务。

每个Worker NioEventLoop循环执行的任务包含3步：

- 1 轮询read、write事件；
- 2 处I/O事件，即read、write事件，在NioSocketChannel可读、可写事件发生时进行处理
- 3 处理任务队列中的任务，runAllTasks。

# 二、webflux性能会提高么？

官网原话：

> Reactive and non-blocking generally do not make applications run faster.
> https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-performance

所以，使用webflux，并不会缩短一个服务接口的响应时间。webflux的优势在于：在同样的资源（CPU和内存）下，提供了更大的吞吐量和更好的伸缩性。
它特别适合应用在 IO 密集型的服务中，比如微服务网关这样的应用中。

> PS: IO 密集型包括：磁盘IO密集型, 网络IO密集型，微服务网关就属于网络 IO 密集型，使用异步非阻塞式编程模型，能够显著地提升网关对下游服务转发的吞吐量。

# 三、Spring Boot 2.0 WebFlux 组件

Spring Boot WebFlux 官方提供了很多 Starter 组件，每个模块会有多种技术实现选型支持，来实现各种复杂的业务需求：
Web：Spring WebFlux
模板引擎：Thymeleaf
存储：Redis、MongoDB、Cassandra，注意目前不支持 MySQL这种关系型数据库
内嵌容器：Tomcat、Jetty、Undertow

# 四、编程模型

Spring 5 web 模块包含了 Spring WebFlux 的 HTTP 抽象。类似 Servlet API , WebFlux 提供了 WebHandler API 去定义非阻塞 API 抽象接口。可以选择以下两种编程模型实现：

- 注解式编程模型。和 MVC 保持一致，WebFlux 也支持响应性 @RequestBody 注解。
- Functional Endpoint编程模型(国内有人翻译叫做功能性端点，我觉得不准确我就不翻译）。函数式编程模型，用来路由和处理请求的小工具。

## 4.1使用Functional Endpoint实现helloworld

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200507104331.png)

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200507104430.png)

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200507104511.png)

下一步，下一步等待项目创建成功。
创建成功后，maven依赖是这个样子的

```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
            <scope>test</scope>
        </dependency>
 </dependencies>
```

编写处理器类 Handler

```java
@Component
public class HelloHandler {
    public Mono<ServerResponse> helloCity(ServerRequest request) {
        return ServerResponse.ok().contentType(MediaType.TEXT_PLAIN)
                .body(BodyInserters.fromObject("Hello, City!"));
    }
}
```

ServerResponse 是对响应的封装，可以设置响应状态、响应头、响应正文。比如 ok 代表的是 200 响应码、MediaType 枚举是代表这文本内容类型、返回的是 String 的对象。
这里用 Mono 作为返回对象，是因为返回包含了一个 ServerResponse 对象。返回多个元素用Flux。

编写路由器类

```java
@Configuration
public class HelloRouter {
    @Bean
    public RouterFunction<ServerResponse> route(HelloHandler cityHandler) {
        return RouterFunctions
                .route(RequestPredicates.GET("/hello")
                                .and(RequestPredicates.accept(MediaType.TEXT_PLAIN)),
                        cityHandler::helloCity);
    }
}
```

RouterFunctions 对请求路由处理类，即将请求路由到处理器，这里将一个 GET 请求 /hello 路由到处理器 cityHandler 的 helloCity 方法上。跟 Spring MVC 模式下的 HandleMapping 的作用类似。
RouterFunctions.route(RequestPredicate, HandlerFunction) 方法，对应的入参是请求参数和处理函数，如果请求匹配，就调用对应的处理器函数。
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200507110323.png)

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200507110441.png)
