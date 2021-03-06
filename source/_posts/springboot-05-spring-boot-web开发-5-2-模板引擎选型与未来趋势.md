---
title: SpringBoot 05.spring boot web开发 5.2.模板引擎选型与未来趋势
date: 2022-07-03T04:26:06.661Z
tags: [springboot]
---
# 一、java web开发经历的几个阶段

1. jsp开发阶段：现在仍然有很多企业项目使用jsp开发。可以说jsp就是页面端的servlet，jsp文件糅合了三种元素：Java代码、动态的数据、HTML代码结构。从抽象层次来看，Java代码部分不仅用来组织数据，还被用来控制HTML页面结构。这样在层次划分上属于比较含糊不清的。当然企业可以通过规范的方式去限制，不允许在jsp页面写java代码，但这只是规范层面的事，实际怎样无法控制。
2. 使用java模板引擎：在这个阶段就出现了freemarker、velocity这样的严格数据模型与业务代码分离的模板引擎。实现了严格的MVC分离，模板引擎的另外一个好处就是：宏定义或者说是组件模板，比jsp标签好用，极大的减少了重复页面组件元素的开发。另外，相对于jsp而言，模板引擎的开发效率会更高。我们都知道，JSP在第一次执行的时候需要转换成Servlet类，开发阶段进行功能调适时，需要频繁的修改JSP，每次修改都要编译和转换，那么试想一天中我们浪费在程序编译的时间有多少。但是java模板引擎，仍然是使用的服务器端的渲染技术，也就是没有办法将html页面和后台服务层面全面解耦，这就要求前端工程师和后端工程师在同一个项目结构下工作，而且前端工程师及其依赖于后端的业务数据，页面无法脱离于后端请求数据在浏览器独立运行。
3. 前端工程化：随着VUE、angularjs、reactjs的大行其道，开始实现真正的前后端分离技术。前端的工程师负责页面的美化与结构，后端工程师可以专注于业务的实现。在ajax和nodejs出现之后，可以说为前端的发展带来了革命性的变化，前端可以做自己的工程化实践。这些新的前端技术通常是“所见即所得”，写完的代码可以直接在浏览器上查看，将前端后端的串行化工作模式转变为并行工作的模式。前端专注于布局、美化，后端专注于业务。专业的人越来越专业，工作效率也更高。

# 二、java模板引擎的选型。

常见的模板引擎有Freemarker、Thymeleaf、Velocity等，下面我们就分别来说一下。

spring boot目前官方集成的框架只有freemarker和Thymeleaf，官方明确建议放弃velocity。很多人说thymeleaf是官方推荐的模板引擎，说实话我没找到这个说法的出处。
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200425144629.png)

## Thymeleaf

Thymeleaf的最大优点也是他的最大的缺点，就是它使用静态html嵌入标签属性，浏览器可以直接打开模板文件，便于前后端联调。也就是贴近于“所见即所得”。但是也正是因为，thyme使用标签属性去放数据，也导致它的语法违反了自然人对于html的理解。另外Thymeleaf的性能一直为人所诟病。
Thymeleaf代码和下面freemarker对一个对象数组遍历的代码对比一下：

```html
<tr th:each="item : ${users}">
    <td th:text="${item.userId}"></td>
    <td th:text="${item.username}"></td>
    <td th:text="${item.password}"></td>
    <td th:text="${item.email}"></td>
    <td th:text="${item.mobile}"></td>
</tr>
```

FreeMarker代码：

```html
<#list users as item>
    <tr>
        <td>${item.userId}</td>
        <td>${item.username}</td>
        <td>${item.password}</td>
        <td>${item.email}</td>
        <td>${item.mobile}</td>
    </tr>
</#list>
```

# 三、最后

综上，目前为止如果使用java模板引擎，我还是推荐freemarker。当然，我还有一个建议，去学vue、angularjs、reactjs。不要用java模板引擎。
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200425145118.png)