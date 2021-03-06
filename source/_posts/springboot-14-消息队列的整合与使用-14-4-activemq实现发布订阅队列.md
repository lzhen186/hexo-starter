---
title: SpringBoot 14.消息队列的整合与使用 14.4.activeMQ实现发布订阅队列
date: 2022-07-03T15:03:47.043Z
tags: [springboot]
---
+# 一、实现发布订阅模型消息

发布订阅模型：比如抖音小视频，某网红发布新视频，多名粉丝收到消息

默认ActiveMQ只支持点对点模型，想要开启发布订阅模型，需要进行配置

```yaml
jms:
  pub-sub-domain: true
```

在CostomActiveMQConfig管理主题对象,这回return的是ActiveMQTopic，而不是ActiveMQQueue

```java
@Bean
public Topic topic() {
    return new ActiveMQTopic("message.topic");
}
```

定义多个消费者，注意发送端的参数类型要与接收端一致，都是QueenMessage queenMessage

```java
@Component
public class TopicConsumer {
    @JmsListener(destination = "message.topic")
    public void receiver1(QueenMessage queenMessage) {
        System.out.println("TopicConsumer : receiver1 : " + queenMessage);
    }

    @JmsListener(destination = "message.topic")
    public void receiver2(QueenMessage queenMessage) {
        System.out.println("TopicConsumer : receiver2 : " + queenMessage);
    }

    @JmsListener(destination = "message.topic")
    public void receiver3(QueenMessage queenMessage) {
        System.out.println("TopicConsumer : receiver3 : " + queenMessage);
    }
}
```

定义一个生产者控制器

```java
@RestController
public class TopicProducerController {
    
    @Resource
    private JmsMessagingTemplate jmsMessagingTemplate;

    @Resource
    private Topic messageTopic;

    @RequestMapping("/sendtopic")
    public QueenMessage send(){
        QueenMessage queenMessage = new QueenMessage("测试","测试内容");
        jmsMessagingTemplate.convertAndSend(messageTopic,queenMessage);
        return queenMessage;
    }
}
```

再试试,输出的结果如下:

```
TopicConsumer : receiver1 : QueenMessage(title=测试, content=测试内容)
TopicConsumer : receiver3 : QueenMessage(title=测试, content=测试内容)
TopicConsumer : receiver2 : QueenMessage(title=测试, content=测试内容)
```