---
title: SpringBoot 05.spring boot web开发 5.7.thymeleaf内置对象与工具类
date: 2022-07-03T04:47:57.115Z
tags: [springboot]
---
# 一、内置对象

> 官方文档： [Thymeleaf 3.0 基础对象](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#appendix-a-expression-basic-objects)

## 七大基础对象：

- `${#ctx}` 上下文对象，可用于获取其它内置对象。
- `${#param}`: 上下文变量。
- `${#locale}`：上下文区域设置。
- `${#request}`: `HttpServletRequest`对象。
- `${#response}`: `HttpServletResponse`对象。
- `${#session}`: `HttpSession`对象。
- `${#servletContext}`: `ServletContext`对象。

## 用法示例

**locale对象操作：**

```html
<div th:text="${#locale.getLanguage() + '_' + #locale.getCountry()}"></div>
```

**session对象操作：**

```html
<div th:text="${session.foo}?:('zoo')"></div>
<div th:text="${session.size()}"></div>
<div th:text="${session.isEmpty()}"></div>
<div th:text="${session.containsKey('foo')}"></div>
```

# 二、 常用的工具类：

> 官方文档： [Thymeleaf 3.0 工具类](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#appendix-b-expression-utility-objects)

- `#strings`：字符串工具类
- `#lists`：List 工具类
- `#arrays`：数组工具类
- `#sets`：Set 工具类
- `#maps`：常用Map方法。
- `#objects`：一般对象类，通常用来判断非空
- `#bools`：常用的布尔方法。
- `#execInfo`：获取页面模板的处理信息。
- `#messages`：在变量表达式中获取外部消息的方法，与使用＃{...}语法获取的方法相同。
- `#uris`：转义部分URL / URI的方法。
- `#conversions`：用于执行已配置的转换服务的方法。
- `#dates`：时间操作和时间格式化等。
- `#calendars`：用于更复杂时间的格式化。
- `#numbers`：格式化数字对象的方法。
- `#aggregates`：在数组或集合上创建聚合的方法。
- `#ids`：处理可能重复的id属性的方法。

## 用法举例：

**date工具类之日期格式化**
使用默认的日期格式(`toString`方法) 并不是我们预期的格式：`Mon Dec 03 23:16:50 CST 2018`
此时可以通过时间工具类`#dates`来对日期进行格式化：`2018-12-03 23:16:50`

```html
<input type="text" th:value="${#dates.format(user.createTime,'yyyy-MM-dd HH:mm:ss')}"/>
```

**首字母大写**

```html
/*
 * Convert the first character of every word to upper-case
 */
${#strings.capitalizeWords(str)}                    // also array*, list* and set*
```

**list方法**

```html
/*
 * Compute size
 */
${#lists.size(list)}

/*
 * Check whether list is empty
 */
${#lists.isEmpty(list)}
```