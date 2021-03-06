---
title: SpringBoot 15.邮件发送的整合与使用 15.1.基础协议及邮件配置整合
date: 2022-07-04T00:06:00.897Z
tags: [springboot]
---
# 一、名词概念解释

- 什么是POP3、SMTP和IMAP？
  简单的说：POP3和IMAP是用来从服务器上下载邮件的。SMTP适用于发送或中转信件时找到下一个目的地。所以我们发送邮件应该使用SMTP协议。
  [POP3、SMTP和IMAP协议介绍](http://help.mail.163.com/faqDetail.do?code=d7a5dc8471cd0c0e8b4b8f4f8e49998b374173cfe9171305fa1ce630d7f67ac22dc0e9af8168582a)
  [IMAP和POP3有什么区别？](http://help.mail.163.com/faqDetail.do?code=d7a5dc8471cd0c0e8b4b8f4f8e49998b374173cfe9171305fa1ce630d7f67ac2f56104105f35a05d)
- 什么是免费邮箱客户端授权码功能？
  邮箱客户端授权码是为了避免您的邮箱密码被盗后，盗号者通过客户端登录邮箱而独特设计的安防功能。

# 二、 整合邮件发送功能

### 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

### QQ邮箱配置

官方配置说明：[参考官方帮助中心](http://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=369)

获取客户端授权码：[参考官方帮助中心](http://service.mail.qq.com/cgi-bin/help?subtype=1&&no=1001256&&id=28)

详细的配置如下：

```yaml
spring:
  mail:
    host: smtp.qq.com #发送邮件服务器
    username: xx@qq.com #QQ邮箱
    password: xxxxxxxxxxx #客户端授权码
    protocol: smtp #发送邮件协议
    properties.mail.smtp.auth: true
    properties.mail.smtp.port: 465 #端口号465或587
    properties.mail.display.sendmail: Javen #可以任意
    properties.mail.display.sendname: Spring Boot Guide Email #可以任意
    properties.mail.smtp.starttls.enable: true
    properties.mail.smtp.starttls.required: true
    properties.mail.smtp.ssl.enable: true
    default-encoding: utf-8
```

> 说明：开启SSL时使用587端口时无法连接QQ邮件服务器

### 网易系(126/163/yeah)邮箱配置

网易邮箱客户端授码:[参考官方帮助中心](http://help.mail.163.com/faq.do?m=list&categoryID=197)

客户端端口配置说明:[参考官方帮助中心](http://mail.163.com/html/110127_imap/index.htm#tab=android)

详细的配置如下：

```yaml
spring:
  mail:
    host: smtp.126.com
    username: xx@126.com
    password: xxxxxxxx
    protocol: smtp
    properties.mail.smtp.auth: true
    properties.mail.smtp.port: 994 #465或者994
    properties.mail.display.sendmail: Javen
    properties.mail.display.sendname: Spring Boot Guide Email
    properties.mail.smtp.starttls.enable: true
    properties.mail.smtp.starttls.required: true
    properties.mail.smtp.ssl.enable: true
    default-encoding: utf-8
    from: xx@126.com
```

> 特别说明:

- 126邮箱SMTP服务器地址:smtp.126.com,端口号:465或者994
- 163邮箱SMTP服务器地址:smtp.163.com,端口号:465或者994
- yeah邮箱SMTP服务器地址:smtp.yeah.net,端口号:465或者994

# 三、发送简单邮件

```java
@Service
public class MailService {
    @Resource
    private JavaMailSender mailSender;

    @Value("${spring.mail.username}")
    private String fromEmail;

    /**
     * 发送文本邮件
     */
    public void sendSimpleMail(String to, String subject, String content) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(fromEmail);
        message.setTo(to);
        message.setSubject(subject);
        message.setText(content);
        mailSender.send(message);
    }

}
```

测试代码:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MailServiceTest {

    @Resource
    MailService mailService;

    @Test
    public void sendSimpleMail() {
        mailService.sendSimpleMail("431899405@qq.com","普通文本邮件","普通文本邮件内容测试");
    }
}
```

测试结果
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200506140850.png)
