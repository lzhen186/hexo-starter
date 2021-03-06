---
title: SpringBoot 14.消息队列的整合与使用 14.6.RocketMQ实现2种消费模式
date: 2022-07-03T15:09:56.528Z
tags: [springboot]
---
# 一、引入依赖

新建SpringBoot项目，引入如下依赖：

```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.0.3</version>
</dependency>
```

# 二、编写配置

```yaml
rocketmq:
  name-server: 192.168.18.137:9876 # 自己的RocketMQ服务地址
  producer:
    send-message-timeout: 300000
    group: launch-group
```

# 三、编写接口

定义一个常量配置类

```java
public class RocketContants {

    public static final String TEST_TOPIC = "test-topic";

    public static final String CONSUMER_GROUP1 = "my-consumer_test-topic";
}
```

对外暴露一个接口用于发送消息

```java
@RestController
public class ProducerController {

    @Resource
    private RocketMQTemplate rocketMQTemplate;

    @GetMapping("/rmqsend")
    public String send(String msg) {
        rocketMQTemplate.convertAndSend(RocketContants.TEST_TOPIC,msg);
        return "success";
    }
}
```

# 四、消费监听

消费者需要实现`RocketMQListener`接口：

```java
@Service
@RocketMQMessageListener(consumerGroup = RocketContants.CONSUMER_GROUP1, topic = RocketContants.TEST_TOPIC)
public class RocketConsumer implements RocketMQListener<String> {

    @Override
    public void onMessage(String message) {
        System.err.println("接收到消息：" + message);
    }
}
```

输出:
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200506135142.png)

# 五、消费者组

- 新建一个RocketConsumer2，如果和RocketConsumer 是同一个组consumerGroup ，一条消息二者选其一消费一次，而且呈现负载均衡分布。即：同一个消费者组的消费者订阅同一话题，组内只能消费一次消息。
- 新建一个RocketConsumer2，如果和RocketConsumer 不是同一个组consumerGroup ，一条消息被消费两次。即：不同消费者组订阅同一话题，组内只消费一次，但多组消费多次。
  ![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200506135216.png)

