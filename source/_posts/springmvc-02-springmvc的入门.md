---
title: SpringMVC 02.SpringMVC的入门
date: 2022-07-02T01:56:21.404Z
tags: [springmvc]
---
# 1.SpringMVC的入门案例

## 1.1 项目目录

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200723113656.png)

## 1.2 配置核心控制器和编码过滤器(web.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <!--配置spring mvc的核心控制器-->
  <servlet>
    <servlet-name>SpringMVCDispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--配置初始化参数,用于读取SpringMVC的配置文件-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:SpringMVC.xml</param-value>
    </init-param>
    <!--配置servlet的对象的创建时间点:应用加载时创建
        只能是非0正整数,表示启动顺序-->
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>SpringMVCDispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
  <!--过滤器-->
  <filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
</web-app>
```

## 1.3 创建SpringMVC的配置文件(SpringMVC.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">
    <!--配置spring要扫描的包-->
    <context:component-scan base-package="cn.krislin"></context:component-scan>
    <!--配置视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

## 1.4 编写控制器并使用注解配置

```java
@Controller("helloController")
public class HelloController {

    @GetMapping("/hello")
    public String sayHello(){
        System.out.println("HelloController中的sayHello方法执行了");
        return "success";
    }
}
```

## 1.5 页面

### index.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: LIM
  Date: 20/7/23
  Time: 11:26
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>index</title>
</head>
<body>
    <h2><a href="hello">SpringMVC入门案例1</a></h2>
</body>
</html>
```

### success.jsp

```jsp
<%--
  Created by IntelliJ IDEA.
  User: LIM
  Date: 20/7/23
  Time: 11:05
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>success</title>
</head>
<body>
    <h1>success</h1>
</body>
</html>
```

## 1.6 测试

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200723114933.png)

# 2.入门案例的执行过程及原理分析

## 2.1 案例的执行过程

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200723115226.png)

1. 服务器启动，应用被加载。读取到 web.xml 中的配置创建 spring 容器并且初始化容器中的对象。
2. 浏览器发送请求，被 DispatherServlet 捕获，该 Servlet 并不处理请求，而是把请求转发出去。转发的路径是根据请求 URL，匹配@RequestMapping 中的内容。
3. 匹配到了后，执行对应方法。该方法有一个返回值。
4. 根据方法的返回值，借助 InternalResourceViewResolver 找到对应的结果视图。
5. 渲染结果视图，响应浏览器。

## 2.2 SpringMVC的请求响应流程

![](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.pianshen.com%2Fimages%2F561%2F8efc491831ac1a2742433fd2ad019a69.png&refer=http%3A%2F%2Fwww.pianshen.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1659319596&t=52e32d9c38110e057c9cee6795d68f70)

# 3.入门案例中涉及的组件

## 3.1 DispatcherServlet: 前端控制器

用户请求到达前端控制器，它就相当于MVC模式中的C，DispatcherServlet 是整个流程控制的中心，由它调用其它组件处理用户的请求，DispatcherServlet 的存在降低了组件之间的耦合性。

## 3.2 HandlerMapping: 处理器映射器

HandlerMapping 负责根据用户请求找到 Handler 即处理器，SpringMVC 提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

## 3.3 Handler: 处理器

它就是我们开发中要编写的具体业务控制器。由 DispatcherServlet 把用户请求转发到 Handler。由Handler 对具体的用户请求进行处理。

## 3.4 HandlerAdapter: 处理器适配器

通过 HandlerAdapter 对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

## 3.5 View Resolver: 视图解析器

View Resolver 负责将处理结果生成 View 视图，View Resolver 首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户。 

## 3.6 View: 视图

SpringMVC 框架提供了很多的 View 视图类型的支持，包括：jstlView、freemarkerView、pdfView等。我们最常用的视图就是 jsp。

一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。

## 3.7 `<mvc:annotation-driven>`说明

在 SpringMVC 的各个组件中，处理器映射器、处理器适配器、视图解析器称为 SpringMVC 的三大组件。使 用 `<mvc:annotation-driven>` 自 动加载 RequestMappingHandlerMapping （处理映射器） 和RequestMappingHandlerAdapter （ 处 理 适 配 器 ） ， 可 用 在 SpringMVC.xml 配 置 文 件 中 使 用`<mvc:annotation-driven>`替代注解处理器和适配器的配置。

它就相当于在 xml 中配置了：

```xml
<!-- 上面的标签相当于 如下配置-->
<!-- Begin -->
<!-- HandlerMapping -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerM
apping"></bean>
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"></bean>
<!-- HandlerAdapter -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerA
dapter"></bean>
<bean class="org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter"></bean>
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean>
<!-- HadnlerExceptionResolvers -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExcept
ionResolver"></bean>
<bean class="org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolv
er"></bean>
<bean class="org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver"
></bean>
<!-- End -->
```

> 注意：
> 一般开发中，我们都需要写上此标签（虽然从入门案例中看，我们不写也行，随着课程的深入，该标签还有具体的使用场景）。
>
> 明确：
> 我们只需要编写处理具体业务的控制器以及视图。

# 4.RequestMapping注解

## 4.1 使用说明

### 源码

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
}
```

### 作用

用于建立请求 URL 和处理请求方法之间的对应关系。

### 出现位置

1. 类上

   请求 URL 的第一级访问目录。此处不写的话，就相当于应用的根目录。写的话需要以/开头。
   它出现的目的是为了使我们的 URL 可以按照模块化管理。
   例如：
   账户模块：
   /account/add
   /account/update
   /account/delete
   ...
   订单模块：
   /order/add
   /order/update
   /order/delete
   红色的部分就是把 RequsetMappding 写在类上，使我们的 URL 更加精细。
2. 方法上
   请求 URL 的第二级访问目录。

### 属性

* **value**：用于指定请求的 URL。它和 path 属性的作用是一样的。
* **method**：用于指定请求的方式。
* **params**：用于指定限制请求参数的条件。它支持简单的表达式。要求请求参数的 key 和 value 必须和配置的一模一样。
  例如：
  params = {"accountName"}，表示请求参数必须有 accountName
  params = {"moeny!100"}，表示请求参数中 money 不能是 100。
* **headers**：用于指定限制请求消息头的条件。

> 注意：
> 以上四个属性只要出现 2 个或以上时，他们的关系是与的关系。

## 4.2 使用示例

### 1.出现位置示例

**控制器代码**

```java
@Controller("accountController")
@RequestMapping("/account")
public class AccountController {
    @RequestMapping("/findAccount")
    public String findAccount() {
        System.out.println("查询了账户。。。。");
        return "success";
    }
}
```

**jsp代码**

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
"http://www.w3.org/TR/html4/loose.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>requestmapping 的使用</title>
    </head>
    <body>
        <!-- 第一种访问方式 -->
        <a href="${pageContext.request.contextPath}/account/findAccount">查询账户</a>
        <br/>
        <!-- 第二种访问方式 -->
        <a href="account/findAccount">查询账户</a>
    </body>
</html>
```

> 注意：
> 在当我们使用此种方式配置时，在 jsp 中第二种写法时，不要在访问 URL 前面加/ ，否则无法找到资源。

### 2.method属性的示例

**控制器代码**

```java
/**
* 保存账户
* @return
*/
@RequestMapping(value="/saveAccount",method=RequestMethod.POST)
public String saveAccount() {
    System.out.println("保存了账户");
    return "success";
}
```

**jsp代码**

```jsp
<!-- 请求方式的示例 -->
<a href="account/saveAccount">保存账户，get 请求</a>
<br/>
<form action="account/saveAccount" method="post">
	<input type="submit" value=" 保存账户， post 请求 ">
</form>
```

> 注意：
> 当使用 get 请求时，提示错误信息是 405，信息是方法不支持 get 方式请求
>
> ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200723142250.png)

### 3. params属性示例

**控制器代码**

```java
/**
* 删除账户
* @return
*/
@RequestMapping(value="/removeAccount",params= {"accountName","money>100"})
public String removeAccount() {
    System.out.println("删除了账户");
    return "success";
}
```

**jsp代码**

```jsp
<!-- 请求参数的示例 -->
<a href="account/removeAccount?accountName=aaa&money>100">删除账户，金额 100</a>
<br/>
<a href="account/removeAccount?accountName=aaa&money>150">删除账户，金额 150</a>
```

> 注意：
> 当我们点击第一个超链接时,可以访问成功。
> 当我们点击第二个超链接时，无法访问。如下图：
>
> ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200723142523.png)