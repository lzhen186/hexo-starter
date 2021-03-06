---
title: SpringMVC 06.响应数据和结果视图
date: 2022-07-02T06:29:39.782Z
tags: [springmvc]
---
# 1.返回值分类

## 1.1 字符串

controller 方法返回字符串可以指定逻辑视图名，通过视图解析器解析为物理视图地址。

```java
@RequestMapping("/testReturnString")
public String testReturnString() {
    System.out.println("AccountController 的 testReturnString 方法执行了。。。。");
    return "success";
}
```

## 1.2 void

在controller 方法形参上可以定义 request和 response，使用 request 或 response 指定响应结果

```java
@RequestMapping("/testReturnVoid")
public void testReturnVoid(HttpServletRequest request,HttpServletResponse response)throws Exception {
}
```

1. **使用 request 转向页面，如下：**

   ```java
   request.getRequestDispatcher("/WEB-INF/pages/success.jsp").forward(request,response);
   ```

2. 也可以通过 response 页面重定向：

   ```java
   response.sendRedirect("testRetrunString")
   ```

3. 也可以通过 response 指定响应结果，例如响应 json 数据：

   ```java
   response.setCharacterEncoding("utf-8");
   response.setContentType("application/json;charset=utf-8");
   response.getWriter().write("n json 串" ");
   ```

## 1.3 ModelAndView

ModelAndView 是 SpringMVC 为我们提供的一个对象，该对象也可以用作控制器方法的返回值。该对象中有两个方法：

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200724125317.png)

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200724125441.png)

**控制器代码**

```java
/**
* 返回 ModeAndView
* @return
*/
@RequestMapping("/testReturnModelAndView")
public ModelAndView testReturnModelAndView() {
    ModelAndView mv = new ModelAndView();
    mv.addObject("username", "张三");
    mv.setViewName("success");
    return mv;
}
```

**响应的jsp代码**

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN""http://www.w3.org/TR/html4/loose.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>执行成功</title>
    </head>
    <body>
        执行成功！
        ${requestScope.username}
    </body>
</html>
```

> 注意：
> 我们在页面上上获取使用的是 requestScope.username 取的，所以返回 ModelAndView 类型时，浏览器跳转只能是请求转发。

# 2.转发和重定向

## 2.1 forward转发

controller 方法在提供了 String 类型的返回值之后，默认就是请求转发。我们也可以写成：

```java
/**
* 转发
* @return
*/
@RequestMapping("/testForward")
public String testForward() {
    System.out.println("AccountController 的 testForward 方法执行了。。。。");
    return "forward:/WEB-INF/pages/success.jsp";
}
```

需要注意的是，如果用了`formward:`则路径必须写成实际视图 url，不能写逻辑视图。

它相当于`request.getRequestDispatcher("url").forward(request,response)`。使用请求转发，既可以转发到 jsp，也可以转发到其他的控制器方法。

## 2.2 Redirect重定向

controller方法提供了一个 String 类型返回值之后，它需要在返回值里使用:`redirect:`

```java
/**
* 重定向
* @return
*/
@RequestMapping("/testRedirect")
public String testRedirect() {
    System.out.println("AccountController 的 testRedirect 方法执行了。。。。");
    return "redirect:testReturnModelAndView";
}
```

它相当于1response.sendRedirect(url)1。**需要注意的是，如果是重定向到 jsp 页面，则 jsp 页面不能写在 WEB-INF 目录中，否则无法找到。**

# 3.ResponseBody响应json数据

## 3.1 使用说明

**作用：**该注解用于将 Controller 的方法返回的对象，通过 HttpMessageConverter 接口转换为指定格式的数据如:json,xml 			等，通过 Response 响应给客户端

## 3.2 使用示例

**需求：**使用`@ResponseBody`注解实现将 controller 方法返回对象转换为 json 响应给客户端。

**前置知识点：**

​	SpringMVC 默认用 `MappingJacksonHttpMessageConverter` 对 json 数据进行转换，需要加入jackson 的包。

​	![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200724130742.png)

> 注意：2.7.0以下的版本用不了

**jsp代码**

```jsp
<script type="text/javascript"src="${pageContext.request.contextPath}/js/jquery.min.js"></script>
<script type="text/javascript">
    $(function(){
        $("#testJson").click(function(){
            $.ajax({
                type:"post",
                url:"${pageContext.request.contextPath}/testResponseJson",
                contentType:"application/json;charset=utf-8",
                data:'{"id":1,"name":"test","money":999.0}',
                dataType:"json",
                success:function(data){
                    alert(data);
                }
            });
        });
    })
</script>
<!-- 测试异步请求 -->
<input type="button" value=" 测试 ajax 请求 json 和响应 json" id="testJson"/>
```

**控制器代码**

```java
/**
* 响应 json 数据的控制器
*/
@Controller("jsonController")
public class JsonController {
    /**
    * 测试响应 json 数据
    */
    @RequestMapping("/testResponseJson")
    public @ResponseBody Account testResponseJson(@RequestBody Account account) {
        System.out.println("异步请求："+account);
        return account;
     }
}
```

