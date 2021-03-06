---
title: SpringBoot 07.嵌入式容器的配置与应用 7.1.嵌入式的容器配置与调整
date: 2022-07-03T06:55:04.790Z
tags: [springboot]
---
# 两种方法配置调整SpringBoot应用容器的参数

- 修改配置文件
- 自定义配置类

## 一、使用配置文件定制修改相关配置

在application.properties / application.yml配置所需要的属性
server.xx开头的是所有servlet容器通用的配置，server.tomcat.xx开头的是tomcat特有的参数，其它类似。

关于修改配置文件application.properties。
[SpringBoot项目详细的配置文件修改文档](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html#common-application-properties)

其中比较重要的有：

| 参数                                | 说明                                                       |
| :---------------------------------- | :--------------------------------------------------------- |
| server.connection-timeout=          | 连接的超时时间                                             |
| server.tomcat.max-connections=10000 | 接受的最大请求连接数                                       |
| server.tomcat.accept-count=100      | 当所求的线程处于工作中，被放入请求队列等待的最大的请求数量 |
| server.tomcat.max-threads=200       | 最大的工作线程池数量                                       |
| server.tomcat.min-spare-threads=10  | 最小的工作线程池数量                                       |

## 二、SpringBoot2.x定制和修改Servlet容器的相关配置，使用配置类

步骤：
1.建立一个配置类，加上@Configuration注解
2.添加定制器ConfigurableServletWebServerFactory
3.将定制器返回

```java
@Configuration
public class TomcatCustomizer {

    @Bean
    public ConfigurableServletWebServerFactory configurableServletWebServerFactory(){
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
        factory.setPort(8585);
        return factory;
    }
}
```