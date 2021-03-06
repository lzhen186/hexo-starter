---
title: SpringBoot 09.日志框架与全局日志管理 9.2.SEL4J使用
date: 2022-07-03T08:00:05.724Z
tags: [springboot]
---
# 1. 如何使用SLF4J

如何在系统中使用SLF4j ：[https://www.slf4j.org](https://www.slf4j.org/)

以后开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，而是调用日志抽象层里面的方法；

给系统里面导入slf4j的jar和 logback的实现jar

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200524110109.png)

每一个日志的实现框架都有自己的配置文件。使用slf4j以后，配置文件还是做成日志实现框架自己本身的配置文件；

# 2. 遗留问题

项目中依赖的框架可能使用不同的日志：

Spring（commons-logging）、Hibernate（jboss-logging）、MyBatis、xxxx

当项目是使用多种日志API时，可以统一适配到SLF4J，中间使用SLF4J或者第三方提供的日志适配器适配到SLF4J，SLF4J在底层用开发者想用的一个日志框架来进行日志系统的实现，从而达到了多种日志的统一实现。

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200524110305.png)

# 3. 如何让系统中所有的日志都统一到slf4j

1. 将系统中其他日志框架先排除出去；
2. 用中间包来替换原有的日志框架（适配器的类名和包名与替换的被日志框架一致）；
3. 我们导入slf4j其他的实现