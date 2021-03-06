---
title: SpringBoot 05.spring boot web开发 5.8.公共片段(标签)与内联js
date: 2022-07-03T04:50:28.262Z
tags: [springboot]
---
# 一、片段表达式(标签) `~{}`

使用方法：首先通过`th:fragment`定制片段 ，然后通过`th:replace` 填写片段路径和片段名。例如：
我们通常将项目里面经常重用的代码抽取为代码片段(标签)

```html
<!-- /views/common/head.html-->
<head th:fragment="static(version)">
        <script th:src="@{/webjars/jquery/${version}/jquery.js}"></script>
</head>
```

然后在不同的页面引用该片段，达到代码重用的目的

```html
<!-- /views/your.html -->
<div th:replace="~{common/head::static(1.12.4)}"></div>
```

在实际使用中，我们往往使用更简洁的表达，去掉表达式外壳直接填写片段名。例如：

```html
<!-- your.html -->
<div th:replace="common/head::static"></div>
<div th:insert="common/head::static"></div>
<div th:include="common/head::static"></div>
```

关于thymeleaf th:replace th:include th:insert 的区别

- th:insert ：保留自己的主标签，保留th:fragment的主标签。
- th:replace ：不要自己的主标签，保留th:fragment的主标签。
- th:include ：保留自己的主标签，不要th:fragment的主标签。（官方3.0后不推荐）

> 值得注意的是，使用替换路径`th:replace` 开头请勿添加斜杠`/`，避免部署运行的时候出现路径报错。（因为默认拼接的路径为`spring.thymeleaf.prefix = classpath:/templates/`）

片段表达式是Thymeleaf的特色之一，细粒度可以达到标签级别，这是JSP无法做到的。
片段表达式拥有三种语法：

- `~{ viewName } 表示引入完整页面`
- `~{ viewName ::selector} 表示在指定页面寻找片段 其中selector可为片段名、jquery选择器等`,即可以在一个html页面内定义多个片段.
- `~{ ::selector} 表示在当前页寻找`

# 二、内联语法

我们之前所讲的内容都是在html标签上使用的thymeleaf的语法，那么如果我们需要在html上使用上述所讲的语法表达式，怎么做？
答：标准格式为：`[[${xx}]]` ，可以读取服务端变量，也可以调用内置对象的方法。例如获取用户变量和应用路径：

```html
    <script th:inline="javascript">
        var user = [[${user}]];
        var APP_PATH = [[${#request.getContextPath()}]];
        var LANG_COUNTRY = [[${#locale.getLanguage()+'_'+#locale.getCountry()}]];
    </script>
```

- 标签（代码片段）内引入的JS里面能使用内联表达式吗？答：不能！内联表达式仅在页面生效，因为`Thymeleaf`只负责解析一级视图，不能识别外部标签JS里面的表达式。