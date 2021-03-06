---
title: SpringBoot 05.spring boot web开发 5.6.thymeleaf基础语法讲解
date: 2022-07-03T04:45:39.397Z
tags: [springboot]
---
# 一、基础语法

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525101013.png)

## 变量表达式 `${}`

使用方法：直接使用`th:xx = "${}"` 获取对象属性 。例如：

```html
<form id="userForm">
    <input id="id" name="id" th:value="${user.id}"/>
    <input id="username" name="username" th:value="${user.username}"/>
    <input id="password" name="password" th:value="${user.password}"/>
</form>

<div th:text="hello"></div>

<div th:text="${articles[0].username}"></div>
```

## 选择变量表达式 `*{}`

使用方法：首先通过`th:object` 获取对象，然后使用`th:xx = "*{}"`获取对象属性。

```html
<form id="userForm" th:object="${user}">
    <input id="id" name="id" th:value="*{id}"/>
    <input id="username" name="username" th:value="*{username}"/>
    <input id="password" name="password" th:value="*{password}"/>
</form>
```

## 链接表达式 `@{}`

使用方法：通过链接表达式`@{}`直接拿到应用路径，然后拼接静态资源路径。例如：

```html
<script th:src="@{/webjars/jquery/jquery.js}"></script>
<link th:href="@{/webjars/bootstrap/css/bootstrap.css}" rel="stylesheet" type="text/css">
```

## 其它表达式

在基础语法中，默认支持字符串连接、数学运算、布尔逻辑和三目运算等。例如：

```html
<input name="name" th:value="${'I am '+(user.name!=null?user.name:'NoBody')}"/>
```

# 二、迭代循环

想要遍历`List`集合很简单，配合`th:each` 即可快速完成迭代。例如遍历用户列表：

```html
<div th:each="article:${articles}" >
    作者：<input th:value="${article.author}"/>
    标题：<input th:value="${article.title}"/>
    内容：<input th:value="${article.content}"/>
</div>
```

在集合的迭代过程还可以获取状态变量，只需在变量后面指定状态变量名即可，状态变量可用于获取集合的下标/序号、总数、是否为单数/偶数行、是否为第一个/最后一个。例如：

```html
<div th:each="article,stat:${articles}" th:class="${stat.even}?'even':'odd'">
    下标：<input th:value="${stat.index}"/>
    序号：<input th:value="${stat.count}"/>
    作者：<input th:value="${article.author}"/>
    标题：<input th:value="${article.title}"/>
    内容：<input th:value="${article.content}"/>
</div>
```

**迭代下标变量用法：**
状态变量定义在一个th:每个属性和包含以下数据:

1. 当前迭代索引,从0开始。这是索引属性。index
2. 当前迭代索引,从1开始。这是统计属性。count
3. 元素的总量迭代变量。这是大小属性。　size
4. iter变量为每个迭代。这是目前的财产。　current
5. 是否当前迭代是奇数还是偶数。这些even/odd的布尔属性。
6. 是否第一个当前迭代。这是first布尔属性。
7. 是否最后一个当前迭代。这是last布尔属性。

# 三、条件判断

条件判断通常用于动态页面的初始化，例如：

```html
<div th:if="${articles}">
    <div>的确存在..</div>
</div>
```

如果想取反则使用unless 例如：

```html
<div th:unless="${articles}">
    <div>不存在..</div>
</div>
```