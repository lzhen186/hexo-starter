---
title: SpringBoot 06.生命周期内的拦截过滤与监听 6.4.应用启动的监听
date: 2022-07-03T06:49:11.844Z
tags: [springboot]
---
# 一、简介

Spring Boot提供了两个接口：CommandLineRunner、ApplicationRunner，用于启动应用时做特殊处理，这些代码会在SpringApplication的run()方法运行完成之前被执行。
通常用于应用启动前的特殊代码执行、特殊数据加载、垃圾数据清理、微服务的服务发现注册、系统启动成功后的通知等。相当于Spring的ApplicationListener、Servlet的ServletContextListener。**使用二者的好处在于，可以方便的使用应用启动参数**，根据参数不同做不同的初始化操作。

# 二、代码实验

## 通过@Component定义方式实现

CommandLineRunner：参数是字符串数组

```java
@Slf4j
@Component
public class CommandLineStartupRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        log.info("CommandLineRunner传入参数：{}", Arrays.toString(args));
    }
}
```

ApplicationRunner：参数被放入ApplicationArguments，通过getOptionNames()、getOptionValues()、getSourceArgs()获取参数

```java
@Slf4j
@Component
public class AppStartupRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        log.info("ApplicationRunner参数: {}", args.getOptionNames());
    }
}
```

## 通过@Bean定义方式实现

```java
@Configuration
public class BeanRunner {
    @Bean
    @Order(1)
    public CommandLineRunner runner1(){
        return new CommandLineRunner() {
            public void run(String... args){
                System.out.println("CommandLineRunner run1()" + Arrays.toString(args));
            }
        };
    }

    @Bean
    @Order(2)
    public CommandLineRunner runner2(){
        return new CommandLineRunner() {
            public void run(String... args){
                System.out.println("CommandLineRunner run2()" + Arrays.toString(args));
            }
        };
    }

    @Bean
    @Order(3)
    public CommandLineRunner runner3(){
        return new CommandLineRunner() {
            public void run(String... args){
                System.out.println("CommandLineRunner run3()" + Arrays.toString(args));
            }
        };
    }
}
```

可以通过@Order设置执行顺序

# 三、执行测试

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200426154941.png)

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200426155029.png)

# 四、总结

CommandLineRunner、ApplicationRunner的核心用法是一致的，就是用于应用启动前的特殊代码执行。ApplicationRunner的执行顺序先于CommandLineRunner；ApplicationRunner将参数封装成了对象，提供了获取参数名、参数值等方法，操作上会方便一些。
另外可以通过@Order定义执行的书序。