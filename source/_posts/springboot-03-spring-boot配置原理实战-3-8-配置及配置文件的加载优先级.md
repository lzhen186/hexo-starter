---
title: SpringBoot 03.spring boot配置原理实战 3.8.配置及配置文件的加载优先级
date: 2022-07-03T00:47:33.846Z
tags: [springboot]
---
# 一、项目内部配置文件加载位置

spring boot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

```
–file:./config/
–file:./
–classpath:/config/
–classpath:/
```

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200421135439.png)
以上是按照优先级从高到低的顺序，所有位置的文件都会被加载，高优先级配置内容会覆盖低优先级配置内容。

SpringBoot会从这四个位置全部加载主配置文件，如果高优先级中配置文件属性与低优先级配置文件不冲突的属性，则会共同存在—互补配置。

我们也可以通过配置spring.config.location来改变默认配置。

```
java -jar Xxx-version.jar  --spring.config.location=D:/application.properties
```

项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置。

# 二、配置文件的加载顺序

配置加载顺序

SpringBoot也可以从以下位置加载配置：优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置。

1. 命令行参数
2. 来自java:comp/env的JNDI属性
3. Java系统属性（System.getProperties()）
4. 操作系统环境变量
5. RandomValuePropertySource配置的random.*属性值
6. jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件
7. jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件
8. jar包外部的application.properties或application.yml(不带spring.profile)配置文件
9. jar包内部的application.properties或application.yml(不带spring.profile)配置文件
10. @Configuration注解类上的@PropertySource
11. 通过SpringApplication.setDefaultProperties指定的默认属性

参考:[官方文档](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-external-config)