---
title: SpringBoot 05.spring boot web开发 5.4.web应用开发之整合freemarker
date: 2022-07-03T04:38:24.586Z
tags: [springboot]
---
# 一、Freemarker简介

FreeMarker是一个模板引擎，一个基于模板生成文本输出的通用工具，使用纯Java编写。FreeMarker我们的第一印象是用来替代JSP的，但是与JSP 不同的是FreeMarker 模板可以在 Servlet容器之外使用。可以使用它们来生成电子邮件、 配置文件、 XML 映射等。或者直接生成HTML。

虽然FreeMarker具有一些编程的能力，但通常由Java程序准备要显示的数据，由FreeMarker生成页面，通过模板显示准备的数据（如下图）

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200426122116.png)

# 二、整合

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency> 
```

```yaml
spring:
  freemarker:
    cache: false # 缓存配置 开发阶段应该配置为false 因为经常会改
    suffix: .html # 模版后缀名 默认为ftl
    charset: UTF-8 # 文件编码
    template-loader-path: classpath:/templates/ 
```



目录结构：

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200426130310.png)

# 三、代码测试

```java
@Controller
@RequestMapping("template")
public class TemplateController {
    @GetMapping("/freemarker")
    public String index(Model model){
        Article article = new Article();
        article.setAuthor("krislin");
        article.setTitle("spring boot");
        article.setContent("spring boot学习");

        model.addAttribute("article",article);

        //模版名称，实际的目录为：resources/templates/freemarkerTemplate.html
        return "freemarkerTemplate";
    }
}
```

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
        <td>作者</td>
        <td>教程名称</td>
        <td>内容</td>
    </tr>

    <tr>
        <td>${article.author}</td>
        <td>${article.title}</td>
        <td>${article.content}</td>
    </tr>

</table>


<img src="/img/template.jpg">
</body>
</html>
```

访问测试地址: http://localhost:8080/template/freemarker
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200426130441.png)

# 四、推荐

如果想进一步学习freemarker，请参考:
http://freemarker.foofun.cn/index.html