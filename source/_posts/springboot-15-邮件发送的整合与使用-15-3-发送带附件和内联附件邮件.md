---
title: SpringBoot 15.邮件发送的整合与使用 15.3.发送带附件和内联附件邮件
date: 2022-07-04T00:12:18.579Z
tags: [springboot]
---
# 一、发送带附件的邮件

```java
/**
 * 发送带附件的邮件
 */
public void sendAttachmentsMail(String to, String subject, String content, String filePath) throws MessagingException {
    MimeMessage message = mailSender.createMimeMessage();
    //带附件第二个参数true
    MimeMessageHelper helper = new MimeMessageHelper(message, true);
    helper.setFrom(fromEmail);
    helper.setTo(to);
    helper.setSubject(subject);
    helper.setText(content, true);

    FileSystemResource file = new FileSystemResource(new File(filePath));
    String fileName = filePath.substring(filePath.lastIndexOf(File.separator));
    helper.addAttachment(fileName, file);

    mailSender.send(message);
}
```

测试

```java
@Test
public void sendAttachmentsMail() throws MessagingException {
    String filePath = "D:\\courseview\\springboot\\template.png";
    mailService.sendAttachmentsMail("3162507037@qq.com", "这是一封带附件的邮件", "邮件中有附件，请注意查收！", filePath);
}
```

邮件结果展示
![image-20200506150442771](C:\Users\LIM\AppData\Roaming\Typora\typora-user-images\image-20200506150442771.png)

# 二、发送内联附件的邮件

```java
/**
 * 发送正文中有静态资源的邮件
 */
public void sendResourceMail(String to, String subject, String content, String rscPath, String rscId) throws MessagingException {
    MimeMessage message = mailSender.createMimeMessage();

    MimeMessageHelper helper = new MimeMessageHelper(message, true);
    helper.setFrom(fromEmail);
    helper.setTo(to);
    helper.setSubject(subject);
    helper.setText(content, true);

    FileSystemResource res = new FileSystemResource(new File(rscPath));
    helper.addInline(rscId, res);

    mailSender.send(message);
}
```

测试

```java
@Test
    public void sendResourceMail() throws MessagingException {
        String rscId = "krislin";
        String content = "<html><body>这是有图片的邮件<br/><img src=\'cid:" + rscId + "\' ></body></html>";
        String imgPath = "E:\\picture\\壁纸\\wallhaven-698428.jpg";
        mailService.sendResourceMail("3162507037@qq.com", "这邮件中含有图片", content, imgPath, rscId);

    }
```

邮件结果展示:

![image-20200506150834983](C:\Users\LIM\AppData\Roaming\Typora\typora-user-images\image-20200506150834983.png)
