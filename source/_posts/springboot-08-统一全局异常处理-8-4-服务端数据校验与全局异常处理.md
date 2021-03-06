---
title: SpringBoot 08.统一全局异常处理 8.4.服务端数据校验与全局异常处理
date: 2022-07-03T07:19:49.932Z
tags: [springboot]
---
> 通常，服务端的数据校验通常不是面向用户的，提示信息还是应该以面向程序员和运维人员为主，在数据进入后台之前进行一道拦截。前端js的数据校验提示信息，是面向用户的，要更加的友好！

# 一、异常校验的规范及常用注解

在web开发时，对于请求参数，一般上都需要进行参数合法性校验的，原先的写法时一个个字段一个个去判断，这种方式太不通用了，所以java的JSR 303: Bean Validation规范就是解决这个问题的。
JSR 303只是个规范，并没有具体的实现，目前通常都是才有hibernate-validator进行统一参数校验。

JSR303定义的校验类型

| Constraint                  | 详细信息                                                 |
| :-------------------------- | :------------------------------------------------------- |
| @Null                       | 被注释的元素必须为 null                                  |
| @NotNull                    | 被注释的元素必须不为 null                                |
| @AssertTrue                 | 被注释的元素必须为 true                                  |
| @AssertFalse                | 被注释的元素必须为 false                                 |
| @Min(value)                 | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @Max(value)                 | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @DecimalMin(value)          | 被注释的元素必须是一个数字，其值必须大于等于指定的最小值 |
| @DecimalMax(value)          | 被注释的元素必须是一个数字，其值必须小于等于指定的最大值 |
| @Size(max, min)             | 被注释的元素的大小必须在指定的范围内                     |
| @Digits (integer, fraction) | 被注释的元素必须是一个数字，其值必须在可接受的范围内     |
| @Past                       | 被注释的元素必须是一个过去的日期                         |
| @Future                     | 被注释的元素必须是一个将来的日期                         |
| @Pattern(value)             | 被注释的元素必须符合指定的正则表达式                     |

Hibernate Validator 附加的 constraint

| Constraint | 详细信息                               |
| :--------- | :------------------------------------- |
| @Email     | 被注释的元素必须是电子邮箱地址         |
| @Length    | 被注释的字符串的大小必须在指定的范围内 |
| @NotEmpty  | 被注释的字符串的必须非空               |
| @Range     | 被注释的元素必须在合适的范围内         |

**用法:把以上注解加在ArticleVO的属性字段上，然后在参数校验的方法上加@Valid注解**
如:

```java
@PutMapping("/article/{id}")
public @ResponseBody AjaxResponse updateArticle(
                                 @Valid @RequestBody ArticleVO article) {
        //业务
    return AjaxResponse.success();
}
```

如果感觉以上的注解仍然不够用，可以自定义注解，参考：
http://blog.lqdev.cn/2018/07/20/springboot/chapter-eight/

# 二、友好的数据校验异常处理（用户输入异常的全局处理）

我们已知当数据校验失败的时候，会抛出异常BindException或MethodArgumentNotValidException。所以我们对这两种异常做全局处理，防止程序员重复编码带来困扰。

```java
@ExceptionHandler(MethodArgumentNotValidException.class)
@ResponseBody
public AjaxResponse handleBindException(MethodArgumentNotValidException ex) {
    FieldError fieldError = ex.getBindingResult().getFieldError();
    return AjaxResponse.error(new CustomException(CustomExceptionType.USER_INPUT_ERROR,fieldError.getDefaultMessage()));
}


@ExceptionHandler(BindException.class)
@ResponseBody
public AjaxResponse handleBindException(BindException ex) {
    FieldError fieldError = ex.getBindingResult().getFieldError();
    return AjaxResponse.error(new CustomException(CustomExceptionType.USER_INPUT_ERROR,fieldError.getDefaultMessage()));
}
```