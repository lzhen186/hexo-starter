---
title: SpringBoot 02.RESTful接口实现及测试 2.2.2.常用注解开发一个RESTful接口
date: 2022-07-02T10:49:06.169Z
tags: [springboot]
---
# 一、开发REST接口

## 1. 第一步：定义资源（对象）

```java
@Data
@Builder
public class Article {
    private Long  id;
    private String author;
    private String title;
    private String content;
    private Date createTime;
}
```

`Data`、`Builder`都是`lombok`提供给我们的注解，有利于我们简化代码。

- `@Builder`为我们提供了通过对象属性的链式赋值构建对象的方法，下文中代码会有详细介绍。
- `@Data`注解帮我们定义了一系列常用方法，如：`getters`、`setters`、`hashcode`、`equals`等

## 2.第二步：HTTP方法与Controller（动作）

我们实现一个简单的RESTful接口

- 增加一篇Article ，使用POST方法
- 删除一篇Article，使用DELETE方法，参数是id
- 更新一篇Article，使用PUT方法，以id为主键进行更新
- 获取一篇Article，使用GET方法

下面代码中并未真正的进行数据库操作

```java
@Slf4j
@RestController
@RequestMapping("/rest")
public class ArticleRestController {
 
    //增加一篇Article ，使用POST方法
    @RequestMapping(value = "/article", method = POST, produces = "application/json")
    public AjaxResponse saveArticle(@RequestBody Article article) {
        //因为使用了lombok的Slf4j注解，这里可以直接使用log变量打印日志
        log.info("saveArticle：{}",article);
        return  AjaxResponse.success(article);
    }
 
    
    //删除一篇Article，使用DELETE方法，参数是id
    @RequestMapping(value = "/article/{id}", method = DELETE, produces = "application/json")
    public AjaxResponse deleteArticle(@PathVariable Long id) {
        log.info("deleteArticle：{}",id);
        return AjaxResponse.success(id);
    }
 
     //更新一篇Article，使用PUT方法，以id为主键进行更新
    @RequestMapping(value = "/article/{id}", method = PUT, produces = "application/json")
    public AjaxResponse updateArticle(@PathVariable Long id, @RequestBody Article article) {
        article.setId(id);
        log.info("updateArticle：{}",article);
        return AjaxResponse.success(article);
    }
 
    //获取一篇Article，使用GET方法
    @RequestMapping(value = "/article/{id}", method = GET, produces = "application/json")
    public AjaxResponse getArticle(@PathVariable Long id) {

        //使用lombok提供的builder构建对象
        Article article = Article.builder()
                .id(1L)
                .author("krislin")
                .title("t1")
                .content("spring boot学习")
                .createTime(new Date()).build();
        return AjaxResponse.success(article);
    }
}
```

因为使用了`lombok`的`@Slf4j`注解（类的定义处），就可以直接使用log变量打印日志。不需要写下面的这行代码。

```java
private static final Logger log = LoggerFactory.getLogger(HelloController.class);
```

# 二、统一规范接口响应的数据格式

下面这个类是用于统一数据响应接口标准的。它的作用是：统一所有开发人员响应前端请求的返回结果格式，减少前后端开发人员沟通成本，是一种RESTful接口标准化的开发约定。下面代码只对请求成功的情况进行封装，在后续的异常处理相关的章节会做更加详细的说明。

```java
@Data
public class AjaxResponse {
    private boolean isOk; //请求是否处理成功
    private int code;   //请求响应状态码（200、400、500）
    private String message; //请求结果描述信息
    private Object data; //请求结果数据

    public AjaxResponse() {
    }

    //请求出现异常时的响应数据封装
    public static AjaxResponse error(CustomException e){
        AjaxResponse resultBean = new AjaxResponse();
        resultBean.setOk(false);
        resultBean.setCode(e.getCode());
        if (e.getCode() == CustomExceptionType.USER_INPUT_ERROR.getCode()){
            resultBean.setMessage(e.getMessage());
        }else if (e.getCode() == CustomExceptionType.SYSTEM_ERROR.getCode()){
            resultBean.setMessage(e.getMessage()+",系统出现异常，请联系管理员");
        }else {
            resultBean.setMessage("系统出现未知异常，请联系管理员");
        }
        return resultBean;
    }

    //请求成功的响应，不带查询数据（用于删除、修改、新增接口）
    public static AjaxResponse success(){
        AjaxResponse resultBean = new AjaxResponse();
        resultBean.setOk(true);
        resultBean.setCode(200);
        resultBean.setMessage("success");
        return resultBean;
    }
    
    //请求成功的响应，带有查询数据（用于数据查询接口）
    public static AjaxResponse success(Object data){
        AjaxResponse resultBean = new AjaxResponse();
        resultBean.setOk(true);
        resultBean.setCode(200);
        resultBean.setMessage("success");
        resultBean.setData(data);
        return resultBean;
    }
}
```