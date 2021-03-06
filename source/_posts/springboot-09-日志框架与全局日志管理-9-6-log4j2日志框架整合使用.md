---
title: SpringBoot 09.日志框架与全局日志管理 9.6.log4j2日志框架整合使用
date: 2022-07-03T08:11:32.005Z
tags: [springboot]
---
# 一、引入maven依赖

Spring Boot默认使用LogBack，但是我们没有看到显示依赖的jar包，其实是因为所在的jar包spring-boot-starter-logging都是作为spring-boot-starter-web或者spring-boot-starter依赖的一部分。
如果这里要使用Log4j2，需要从spring-boot-starter-web中去掉spring-boot-starter-logging依赖，同时显示声明使用Log4j2的依赖jar包，具体如下:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

另外log4j是之前使用比较广泛的软件，容易与log4j2发生冲突，如果冲突将它从相应的软件里面排除掉,比如:dozer

```xml
<dependency>
    <groupId>net.sf.dozer</groupId>
    <artifactId>dozer</artifactId>
    <version>5.4.0</version>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

## 二、添加配置文件log4j2.xml

在resources目录下新建一个log4j2.xml文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <Appenders>
        <Console name="CONSOLE" target="SYSTEM_OUT">
            <PatternLayout charset="UTF-8" pattern="[%-5p] %d %c - %m%n" />
        </Console>

        <RollingFile name="runtimeFile" fileName="./logs/boot-launch.log" filePattern="./logs/boot-launch-%d{yyyy-MM-dd}.log"
                     append="true">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS Z}\t%level\t%class\t%line\t%thread\t%msg%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
            </Policies>
            <!-- 此行以下为自动清理日志的配置 -->
            <DefaultRolloverStrategy>
                <Delete basePath="./logs">
                    <!-- glob 项为需要自动清理日志的pattern -->
                    <IfFileName glob="*.log"/>
                    <!-- 30d 表示自动清理掉30天以前的日志文件 -->
                    <IfLastModified age="30d"/>
                </Delete>
            </DefaultRolloverStrategy>
            <!-- 此行以上为自动清理日志的配置 -->
        </RollingFile>


    </Appenders>

    <Loggers>
        <root level="info">
            <AppenderRef ref="CONSOLE" />
            <AppenderRef ref="runtimeFile" />
        </root>
    </Loggers>
</configuration>
```

注意：关于log4j2的定时删除如果filePattern的粒度为HH，那么在中如果age=30d则不生效

## 三、修改application.yml配置

但是这样还不够，Spring Boot并不知道log4j2.xml是干嘛的，需要通过在application.properties文件中显示声明才行

```yaml
logging:
    config: classpath:log4j2.xml
```

# 四、测试

```java
@RestController
@Slf4j
public class LogDemoController {
    //private static Logger log= LoggerFactory.getLogger(LogDemo.class);
 
    @GetMapping("/logdemo")
    public String log(){
        log.trace("======trace");
        log.debug("======debug");
        log.info("======info");
        log.warn("======warn");
        log.error("======error");
        return "logok";
    }
}
```

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200524125929.png)