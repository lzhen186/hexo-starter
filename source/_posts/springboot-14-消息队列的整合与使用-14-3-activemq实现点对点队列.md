---
title: SpringBoot 14.消息队列的整合与使用 14.3.activeMQ实现点对点队列
date: 2022-07-03T14:59:11.259Z
tags: [springboot]
---
# 一、整合ActiveMQ

```xml
        <!--整合ActiveMQ-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-activemq</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-pool</artifactId>
        </dependency>

```

```yaml
spring:
    activemq:
      user: admin
      password: admin
      broker-url: tcp://192.168.18.137:61616
      in-memory: false
      pool:
        enabled: true
        max-connections: 50
      packages:
        trust-all: true   #是否信任所有包，信任后所有包内的对象均可序列化传输
```



```java
@EnableJms //开启消息中间件的服务能力
@Configuration
public class CostomActiveMQConfig {

    //配置一个消息队列（P2P模式）
    @Bean
    public Queue messageQueue() {
        //这里相当于为消息队列起一个名字用于生产消费用户访问日志
        return new ActiveMQQueue("message.queue");
    }
}
```

# 二、实现点对点模型消息

消息对象,必须实现Serializable 接口

```java
@Data
@AllArgsConstructor
public class QueenMessage implements Serializable {

    private String title;

    private String content;

}
```

消费者示例

```java
@Component
@Slf4j
public class P2pConsumerListener {
   @JmsListener(destination = "message.queue")
    public void insertVisitLog(QueenMessage queenMessage) {
        log.info("消费者接收数据 : " + queenMessage);
    }
}
```

生产者测试示例

```java
@RestController
public class P2pProducerController {
    @Resource
    private JmsMessagingTemplate jmsMessagingTemplate;

    @Resource
    private Queue messageQueue;

    @RequestMapping("/send")
    public QueenMessage send(){
        QueenMessage queenMessage = new QueenMessage("测试","测试内容");

        jmsMessagingTemplate.convertAndSend(messageQueue,queenMessage);
        return queenMessage;
    }
}
```

浏览器访问http://localhost:8080/send，后台打印出：
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200506131745.png)