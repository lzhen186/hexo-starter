---
title: SpringBoot 15.邮件发送的整合与使用 15.2.发送html和基于模板的邮件
date: 2022-07-04T00:09:08.499Z
tags: [springboot]
---
# 一、发送html邮件服务

```java
/**
 * 发送html邮件
 */
public void sendHtmlMail(String to, String subject, String content) throws MessagingException {
    //注意这里使用的是MimeMessage
    MimeMessage message = mailSender.createMimeMessage();
    MimeMessageHelper helper = new MimeMessageHelper(message, true);
    helper.setFrom(fromEmail);
    helper.setTo(to);
    helper.setSubject(subject);
    //第二个参数是否是html，true
    helper.setText(content, true);

    mailSender.send(message);
}
```

测试用例

```java
@Test
public void sendHtmlMail() throws MessagingException {
    mailService.sendHtmlMail("3162507037@qq.com","一封html测试邮件","<body style=\"text-align: center;margin-left: auto;margin-right: auto;\">\n"
            + " <div id=\"welcome\" style=\"text-align: center;position: absolute;\" >\n"
            +"      <h3>\"一封html测试邮件\"</h3>\n"
            +"      <span>http://www.zimug.com</span>"
            + "     <div style=\"text-align: center; padding: 10px\"><a style=\"text-decoration: none;\" href=\"https://github.com/krislinzhao\" target=\"_bank\" >"
            + "           <strong>我很用心，希望你有所收获</strong></a></div>\n"
            + " </div>\n" + "</body>");
}
```

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200506143955.png)

# 二、基于freemarker模板的邮件

基于freemarker模板邮件本质上，还是发送html邮件，只不过是有一个把模板转换成html字符串的过程。thymeleaf也可以实现，不妨试一试。

```java
@Autowired
private FreeMarkerConfigurer freeMarkerConfigurer;

@Test
public void sendTemplateMail() throws IOException, TemplateException, MessagingException {
    List<ArticleVO> articles = new ArrayList<>();
    articles.add(new ArticleVO(1L,"zimug","手摸手教你学spring boot","内容一",new Date(),null));
    articles.add(new ArticleVO(2L,"zimug","手摸手教你学spring boot","内容二",new Date(),null));
    Template template = freeMarkerConfigurer.getConfiguration().getTemplate("fremarkertemp.html");

    Map<String,Object> model = new HashMap<>();
    model.put("articles",articles);
    String templateHtml = FreeMarkerTemplateUtils.processTemplateIntoString(template,model);


    mailService.sendHtmlMail("431899405@qq.com","一封freemarker模板的html测试邮件",templateHtml);
}
```

模板文件

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8" />
    <title>freemarker简单示例</title>
</head>
<body>
<h1>Hello Freemarker</h1>

<table class="">
    <tr>
        <td>firstName</td>
        <td>lastName</td>
    </tr>

    <#list people as person>
        <tr>
            <td>${person.firstname}</td>
            <td>${person.lastname}</td>
        </tr>
    </#list>

</table>
</body>
</html>
```



![image-20200506145734905](C:\Users\LIM\AppData\Roaming\Typora\typora-user-images\image-20200506145734905.png)
