---
title: SpringBoot 01.spring boot 2.x基础及概念入门 1.1.Spring Boot的核心概念
date: 2022-07-02T08:43:33.539Z
tags: [springboot]
---
# 一、Spring Boot 、 Spring MVC 、Spring对比

**Spring 框架**

Spring框架最核心的特性就是依赖注入DI（Dependency Injecttion）和控制反转IOC（Inversion Of Control）。如果你能够合理的使用DI和IOC，可以开发出松耦合、扩展性好的的应用程序。

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200414140300.png)

**Spring MVC**

Spring MVC提供了一种友好的方式来开发Web应用程序。 通过使用诸如Dispatcher Servlet，ModelAndView和View Resolver，可以轻松开发Web应用程序。

**Spring Boot**

Spring 和 Spring MVC最大的弊病在于存在大量的配置，并且这些配置在不同的项目中具有很高的相似性。从而导致重复配置，繁琐而且杂乱！

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200414140521.png)

Spring Boot期望通过结合自动配置和starters来解决了这个问题。 另外，Spring Boot还提供了一些功能，可以更快地构建可用于生产环境的应用程序。

# 二、Spring Boot 自动配置

Spring和Spring MVC应用程序里面有大量的XML或Java Bean配置。Spring Boot为解决这个问题，提供一种新的解决方案，新的思维方式。

springboot思考的方式：是不是可以更加智能一点，当Spring中加入一些新的jar包，可以自动的配置一些bean。 比如：Spring MVC JAR位于类路径中时，自动配置Dispatcher Servlet。当然，当这些自动的默认配置不符合我们的要求的时候，我们可以修改。

# 三、什么是Spring Boot Starter？

Spring Boot Starter是一组被依赖第三方类库的集合。

如果你要开发一个web应用程序，就通过包管理工具(如maven)引入spring-boot-starter-web就可以了，而不用分别引入下面这么多依赖类库，spring-boot-starter-web一次性帮你引入下面的这些常用类库。

- Spring — spring 核心, beans, context上下文, AOP面向切面
- Web MVC — Spring MVC
- Jackson — JSON数据的序列化与反序列化
- Validation — Hibernate参数校验及校验API
- 嵌入式 Servlet Container — Tomcat
- 日志框架Logging — logback, slf4j

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200414140923.png)

# 四、什么是Spring Boot Starter Parent

所有的Spring Boot项目默认使用spring-boot-starter-parent作为应用程序的父项目。

```properties
  <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/>
    </parent>
```

继承父项目的好处在于： 统一java版本配置和其他的一些依赖类库的版本。也就是说，你引入的第三方类库不要加版本号，父项目替你管理版本，而且是经过兼容性测试的。比你自己随便引入一个版本兼容性更好。

> 当然父项目只能帮你管理一些常用类库的版本，如果你引入一些不常用的jar，还是要自己管理版本号及兼容性！

# 五、嵌入式web容器

Spring boot打成jar包，默认包含嵌入式的web容器：tomcat。你可以简单的使用如下命令启动一个web服务：

```shell
java -jar springboot-demo.jar
```

这更有利于微服务的部署及微服务的构建、启动、扩容。Spring Boot还支持Jetty和Undertow作为web容器。

# 六、Spring Data

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200414141336.png)

Spring Data的目标是提供一种更友好的方式或者是API来存取数据。包括对于关系型数据库和NOSQL数据的支持。比如：

- Spring Data JPA — 关系型数据库操作的API，友好且易于使用
- Spring Data MongoDB -MongoDB的操作API
- Spring Data REST — 从持久层Repositories自动生成服务层API，暴露 REST APIs 接口。超级好用！

# 七、spring boot2.x新特性

## 7.1.基础环境升级

- 最低 JDK 8，支持 JDK 9，不再支持 Java 6 和 7。Spring Boot 2.0 要求 Java 8 作为最低版本，许多现有的 API 已更新，以利用 Java 8 的特性。
  例如，接口上的默认方法，函数回调以及新的 API，如 javax.time。
- 如果你正在使用 Java 7 或更早版本，则在开发 Spring Boot 2.0 应用程序之前，需要升级你的 JDK。

## 7.2.依赖组件升级

- Jetty 9.4，Jetty 是一个开源的 Servlet 容器，它为基于 Java 的 Web 内容，例如 JSP 和 Servlet 提供运行环境。Jetty 是使用 Java 语言编写的，它的 API 以一组 JAR 包的形式发布。
- Tomcat 8.5，Apache Tomcat 8.5.x 旨在取代 8.0.x，完全支持 Java 9。
- Flyway 5，Flyway 是独立于数据库的应用、管理并跟踪数据库变更的数据库版本管理工具。用通俗的话讲，Flyway 可以像 SVN 管理不同人的代码那样，管理不同人的 SQL 脚本，从而做到数据库同步。
- Hibernate 5.2，Hibernate 是一款非常流行的 ORM 框架。
- Gradle 3.4，Spring Boot 的 Gradle 插件在很大程度上已被重写，有了重大的改进。
- Thymeleaf 3.0，Thymeleaf 3 相对于 Thymeleaf 2 有非常大的性能提升。

## 7.3. 默认软件替换

- 默认数据库连接池已从 Tomcat 切换到 HikariCP，HikariCP 是一个高性能的 JDBC 连接池，Hikari 是日语“光”的意思。
- redis客户端默认使用 Lettuce，替换掉Jedis.Lettuce 是一个可伸缩的线程安全的 Redis 客户端，用于同步、异步和反应使用。多个线程可以共享同一个 RedisConnection，它利用优秀 Netty NIO 框架来高效地管理多个连接，支持先进的 Redis 功能，如 Sentinel、集群、流水线、自动重新连接和 Redis 数据模型。

## 7.4.新技术的引入

- 响应式编程WebFlux,重要的变革，后续章节会详细展示
- 支持 Quartz,Spring Boot 1.0 并没有提供对 Quartz 的支持，之前出现了各种集成方案，Spring Boot 2.0 给出了最简单的集成方式。
- 对Kotlin 的支持
- JOOQ 的支持,JOOQ 是基于 Java 访问关系型数据库的工具包。JOOQ 既吸取了传统 ORM 操作数据的简单性和安全性，又保留了原生 SQL 的灵活性，它更像是介于 ORMS 和 JDBC 的中间层。

## 7.5.彩蛋

在 Spring Boot 1.0 项目中 src/main/resources 路径下新建一个 banner.txt 文件，文件中写入一些字符，启动项目时就会发现默认的 Banner 被替换了，到了 Spring Boot 2.0 现在可以支持 Gif 文件的打印，Spring Boot 2.0 在项目启动的时候，会将 Gif 图片的每一个画面，按照顺序打印在日志中，所有的画面打印完毕后，才会启动 Spring Boot 项目。