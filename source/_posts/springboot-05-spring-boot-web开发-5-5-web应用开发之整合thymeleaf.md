---
title: SpringBoot 05.spring boot web开发 5.5.web应用开发之整合thymeleaf
date: 2022-07-03T04:41:25.320Z
tags: [springboot]
---
# 一、Thymeleaf简介

Thymeleaf 是一个服务器端 Java 模板引擎，能够处理 HTML、XML、CSS、JAVASCRIPT 等模板文件。Thymeleaf 模板可以直接当作静态原型来使用，它主要目标是为开发者的开发工作流程带来优雅的自然模板，也是 Java 服务器端 HTML5 开发的理想选择。

# 二、集成

```xml
<dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

ThymeleafProperties:

```java
@ConfigurationProperties(
    prefix = "spring.thymeleaf"
)
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    public static final String DEFAULT_PREFIX = "classpath:/templates/";
    public static final String DEFAULT_SUFFIX = ".html";
    private boolean checkTemplate = true;
    private boolean checkTemplateLocation = true;
    private String prefix = "classpath:/templates/";
    private String suffix = ".html";
    private String mode = "HTML";
```

默认只要我们把HTML页面放在`classpath:/templates/`，thymeleaf就能自动渲染。

```yaml
spring:
  thymeleaf:
    cache: false # 启用缓存:建议生产开启
    check-template-location: true # 检查模版是否存在
    enabled: true # 是否启用
    encoding: UTF-8 # 模版编码
    excluded-view-names: # 应该从解析中排除的视图名称列表（用逗号分隔）
    mode: HTML5 # 模版模式
    prefix: classpath:/templates/ # 模版存放路径
    suffix: .html # 模版后缀
```

项目目录：

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200426142831.png)

# 三、写个简单的小例子

```java
@Controller
@RequestMapping("template")
public class TemplateController {
    @GetMapping("/thymeleaf")
    public String index(Model model){
        List<Article> articles = new ArrayList<>();
        Article article = new Article();
        article.setAuthor("krislin");
        article.setTitle("spring boot");
        article.setContent("spring boot学习");

        Article article1 = new Article();
        article1.setAuthor("krislin1");
        article1.setTitle("spring boot1");
        article1.setContent("spring boot学习1");

        articles.add(article);
        articles.add(article1);

        model.addAttribute("articles",articles);

        return "thymeleafTemplate";
    }
}
```

```html
<!DOCTYPE html>
<--!导入thymeleaf的名称空间-->
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">
    <meta charset="UTF-8" />
    <title>thymeleaf简单示例</title>
</head>
<body>
<h1>Hello Thymeleaf</h1>

<table class="">
    <tr>
        <td>作者</td>
        <td>教程名称</td>
        <td>内容</td>
    </tr>
    <tr th:each="item : ${articles}">
        <td th:text="${item.author}"></td>
        <td th:text="${item.title}"></td>
        <td th:text="${item.content}"></td>
    </tr>
</table>

<img src="/img/template.jpg">
</body>
</html>
```

访问测试地址: http://localhost:8080/template/thymeleaf

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200426142908.png)

