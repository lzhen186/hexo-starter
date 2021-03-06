---
title: SpringBoot 06.生命周期内的拦截过滤与监听 6.3.自定义事件的发布与监听
date: 2022-07-03T06:44:23.980Z
tags: [springboot]
---
# 一、概述:

**自定义事件和自定义监听器类的实现方式**
自定义事件：继承自ApplicationEvent抽象类，然后定义自己的构造器
自定义监听：实现ApplicationListener接口，然后实现onApplicationEvent方法

**springboot进行事件监听有四种方式**
1.手工向ApplicationContext中添加监听器
2.将监听器装载入spring容器
3.在application.properties中配置监听器
4.通过@EventListener注解实现事件监听

# 二、代码实现

下面讲下4种事件监听的具体实现

## 方式1.

首先创建MyListener1类

```java
@Slf4j
public class MyListener1 implements ApplicationListener<MyEvent> {
    public void onApplicationEvent(MyEvent event) {
        log.info(String.format("%s监听到事件源：%s.", MyListener1.class.getName(), event.getSource()));
    }
}
```

然后在springboot应用启动类中获取ConfigurableApplicationContext上下文，装载监听

```java
@SpringBootApplication
public class BootLaunchApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(BootLaunchApplication.class, args);
        //装载监听
        context.addApplicationListener(new MyListener1());
    }
}
```

## 方式2（推荐）.

创建MyListener2类，并使用@Component注解将该类装载入spring容器中

```java
@Component
@Slf4j
public class MyListener2 implements ApplicationListener<MyEvent> {

    public void onApplicationEvent(MyEvent event) {
        log.info(String.format("%s监听到事件源：%s.", MyListener2.class.getName(), event.getSource()));
    }

}
```

## 方式3.

首先创建MyListener3类

```java
@Slf4j
public class MyListener3 implements ApplicationListener<MyEvent> {
    public void onApplicationEvent(MyEvent event) {
        log.info(String.format("%s监听到事件源：%s.", MyListener3.class.getName(), event.getSource()));
    }
}
```

然后在application.yml中配置监听

```yaml
context:
  listener:
    classes: club.krislin.customlistener.MyListener3
```

## 方式4（推荐）.

创建MyListener4类，该类无需实现ApplicationListener接口，使用@EventListener装饰具体方法

```java
@Slf4j
@Component
public class MyListener4 {
    @EventListener
    public void listener(MyEvent event) {
        log.info(String.format("%s监听到事件源：%s.", MyListener4.class.getName(), event.getSource()));
    }
}
```

自定义事件代码如下：

```java
@SuppressWarnings("serial")
public class MyEvent extends ApplicationEvent
{
 public MyEvent(Object source)
 {
  super(source);
 }
}
```

# 三、测试监听事件发布

有了applicationContext，想在哪发布事件就在哪发布事件

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class CustomListenerTest {

    @Resource private
    ApplicationContext applicationContext;

    @Test
    public void testEvent(){
        applicationContext.publishEvent(new MyEvent("测试事件."));
    }
}
```

启动后，日志打印如下。（下面截图是在启动类发布事件后的截图，在单元测试里面监听器1监听不到，执行顺序问题）：

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200426153729.png)

由日志打印可以看出，SpringBoot四种事件的实现方式监听是有序的。无论执行多少次都是这个顺序。