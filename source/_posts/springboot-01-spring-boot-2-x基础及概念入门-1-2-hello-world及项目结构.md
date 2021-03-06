---
title: SpringBoot 01.spring boot 2.x基础及概念入门 1.2.Hello World及项目结构
date: 2022-07-02T09:08:05.691Z
tags: [springboot]
---
# 一、使用IntellijIDEA建立第一个spring boot 项目

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200414142145.png)

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200414142345.png)

在这里可以选择我们需要依赖的第三方软件类库，包括spring-boot-web，mysql驱动，mybatis等。

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200414142531.png)

项目创建过程可能因为maven依赖项较多，下载时间比较长，耐心等待。项目构建完成之后删掉下面的这几个文件，这几个文件是maven版本控制相关的文件。我们结合IDEA管理maven，一般来说这几个文件用不到。

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200414143027.png)

# 二、Hello World示例程序

将application.properties改成application.yml。yml文件和properties配置文件具有同样的功能。二者的区别在于：

- yml文件的层级更加清晰直观，但是书写时需要注意格式缩进对齐。yml格式配置文件更有利于表达复杂数据结构的配置。比如：列表，对象（后面章节会详细说明）。
- properties阅读上不如yml直观，好处在于书写时不用特别注意格式缩进对齐。

```yam
server:
  port: 8888   # web应用服务端口
```

引入spring-boot-starter-web依赖

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

一个hello world测试Controller

```java
@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello(String name) {
        return "hello world, " +name;
    }
}
```

测试

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200414150524.png)

# 三、项目结构目录结构简介

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200414150822.png)

项目结构目录整体上符合maven规范要求：

| 目录位置                                  | 功能                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| src/main/java                             | 项目java文件存放位置，初始化包含主程序入口 XxxApplication，可以通过直接运行该类来 启动 Spring Boot应用 |
| src/main/resources                        | 存放静态资源，图片、CSS、JavaScript、web页面模板文件等       |
| src/test                                  | 单元测试代码目录                                             |
| .gitignore                                | git版本管理排除文件                                          |
| target文件夹                              | 项目代码构建打包结果文件存放位置，不需要人为维护             |
| pom.xml                                   | maven项目配置文件                                            |
| application.properties（application.yml） | 用于存放程序的各种依赖模块的配置信息，比如服务端口，数据库连接配置等 |

- src/main/resources/static主要用来存放css、图片、js等开发用静态文件
- src/main/resources/public用来存放可以直接用于访问的html文件
- src/main/resources/templates用来存放web开发模板文件