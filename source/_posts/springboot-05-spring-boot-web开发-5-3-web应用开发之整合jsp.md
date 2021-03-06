---
title: SpringBoot 05.spring boot web开发 5.3.web应用开发之整合jsp
date: 2022-07-03T04:32:15.514Z
tags: [springboot]
---
# 一、集成jsp

spring-boot-starter-web 包依赖了 spring-boot-starter-tomcat 不需要再单独配置。
引入 jstl 和内嵌的 tomcat，jstl 是一个 JSP 标签集合，它封装了 JSP 应用的通用核心功能。
tomcat-embed-jasper 主要用来支持 JSP 的解析和运行。

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!-- spring boot 内置tomcat jsp支持 -->
<dependency>
  <groupId>org.apache.tomcat.embed</groupId>
  <artifactId>tomcat-embed-jasper</artifactId>
</dependency>
<!--jsp页面使用jstl标签-->
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>jstl</artifactId>
</dependency>
```

spring.mvc.view.prefix 指明 jsp 文件在 webapp 下的哪个目录
spring.mvc.view.suffix 指明 jsp 以什么样的后缀结尾

```yaml
spring:
  mvc:
    view:
      suffix: .jsp
      prefix: /WEB-INF/jsp/

debug: true
```

# 二、目录结构

这个目录结构和配置文件一一对应，一定不要放错了。
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200426121706.png)

- 静态资源，如：图片放在resources/static目录下面
- jsp文件放在webapp.WEB-INF.jsp的下面

# 三、代码测试

```java
@Controller
@RequestMapping("/template")
public class TemplateController {

    @Resource(name="articleMybatisRestServiceImpl")
    ArticleRestService articleRestService;

    @GetMapping("/jsp")
    public String index(String name, Model model) {

        List<ArticleVO> articles = articleRestService.getAll();

        model.addAttribute("articles", articles);

        //模版名称，实际的目录为：src/main/webapp/WEB-INF/jsp/jsptemp.jsp
        return "jsptemp";
    }
}
```

jsptemp.jsp

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<table class="">
    <tr>
        <td>作者</td>
        <td>教程名称</td>
        <td>内容</td>
    </tr>
    <c:forEach var="article" items="${articles}">
        <tr class="text-info">
            <td>${article.author}</td>
            <td>${article.title}</td>
            <td>${article.content}</td>
        </tr>
    </c:forEach>


</table>
<img src="/image/jsp.png">
</body>
</html>
```

**注意img标签的静态资源引用路径与实际存放路径之间的关系。**]

# 四、运行方法测试

因为jsp对jar运行的方式支持不好，所以要一一进行测试:

1. 使用IDEA启动类启动测试，没有问题
2. 使用`spring-boot:run -f pom.xml`测试，没有问题
3. 打成jar包通过`java -jar`方式运行，页面报错
4. 打成war包，运行于外置的tomcat，没有问题

所以，无法用jar包的形式运行jsp应用。