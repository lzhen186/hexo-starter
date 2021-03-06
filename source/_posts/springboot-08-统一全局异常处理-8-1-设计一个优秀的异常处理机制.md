---
title: SpringBoot 08.统一全局异常处理 8.1.设计一个优秀的异常处理机制
date: 2022-07-03T07:04:56.722Z
tags: [springboot]
---
# 一、异常处理的乱象例举

## 乱象一：捕获异常后只输出到控制台

前端js-ajax代码

```js
$.ajax({
    type: "GET",
    url: "/user/add",
    dataType: "json",
    success: function(data){
        alert("添加成功");
    }
});
```

后端业务代码

```java
try {
    // do something
} catch (XyyyyException e) {
    e.printStackTrace();
}
```

问题：

1. 后端直接将异常捕获，而且只做了日志打印。用户体验非常差，一旦后台出错，用户没有任何感知，页面无状态，。
2. 如果没有人去经常关注日志，不会有人发现系统出现异常

## 乱象二：混乱的返回方式

前端代码

```js
$.ajax({
    type: "GET",
    url: "/goods/add",
    dataType: "json",
    success: function(data) {
        if (data.flag) {
            alert("添加成功");
        } else {
            alert(data.message);
        }
    },
    error: function(data){
        alert("添加失败");
    }
});
```

后端代码

```java
@RequestMapping("/goods/add")
@ResponseBody
public Map add(Goods goods) {
    Map map = new HashMap();
    try {
        // do something
        map.put(flag, true);
    } catch (Exception e) {
        e.printStackTrace();
        map.put("flag", false);
        map.put("message", e.getMessage());
    }
    reutrn map;
}
```

问题：

1. 每个人返回的数据有每个人自己的规范，你叫flag他叫isOK，你的成功code是0，它的成功code是0000。这样导致后端书写了大量的异常返回逻辑代码，前端也随之每一个请求一套异常处理逻辑。很多重复代码。
2. 如果是前端后端一个人开发还勉强能用，如果前后端分离，就是系统灾难。

参考:
https://juejin.im/post/5c3ea92a5188251e101598aa

# 二、该如何设计异常处理

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200427102245.png)

## 面向相关方友好

1. 后端开发人员职责单一，只需要将异常捕获并转换为自定义异常一直对外抛出。不需要去想页面跳转404，以及异常响应的数据结构的设计。
2. 面向前端人员友好，后端返回给前端的数据应该有统一的数据结构，统一的规范。不能一个人一个响应的数据结构。而在此过程中不需要后端开发人员做更多的工作，交给全局异常处理器去处理“异常”到“响应数据结构”的转换。
3. 面向用户友好，用户能够清楚的知道异常产生的原因。这就要求自定义异常，全局统一处理，ajax接口请求响应统一的异常数据结构，页面模板请求统一跳转到404页面。
4. 面向运维友好，将异常信息合理规范的持久化，以便查询。

**为什么要将系统运行时异常捕获，转换为自定义异常抛出？**
答：因为用户不认识ConnectionTimeOutException类似这种异常是什么东西，但是转换为自定义异常就要求程序员对运行时异常进行一个翻译，比如：自定义异常里面应该有message字段，后端程序员应该明确的在message字段里面用面向用户的友好语言，说明发生了什么。

# 三、开发规范

1. Controller、Service、DAO层拦截异常转换为自定义异常，不允许将异常私自截留。必须对外抛出。
2. 统一数据响应代码，使用httpstatusode，不要自定义。自定义不方便记忆。200请求成功，400用户输入错误导致的异常，500系统内部异常，999未知异常。
3. 自定义异常里面有message属性，一定用友好的语言描述异常，并赋值给message.
4. 不允许对父类Excetion统一catch，要分小类catch，这样能够清楚地将异常转换为自定义异常传递给前端。