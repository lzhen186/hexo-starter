---
title: SpringMVC 09.SpringMVC中异常处理
date: 2022-07-02T06:49:28.182Z
tags: [springmvc]
---
# 1.异常处理的思路

系统中异常包括两类：预期异常和运行时异常 RuntimeException，前者通过捕获异常从而获取异常信息，后者主要通过规范代码开发、测试通过手段减少运行时异常的发生。

系统的 dao、service、controller 都通过 throws Exception 向上抛出，最后由SpringMVC前端控制器交由异常处理器进行异常处理，如下图：

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200725164413.png)

# 2.实现步骤

## 2.1 编写异常类和错误页面

```java
/**
* 自定义异常
*/
public class CustomException extends Exception {
    private String message;
    public CustomException(String message) {
    	this.message = message;
    }
    public String getMessage() {
    	return message;
    }
    
    public void setMessage(String message) {
        this.message = message;
    }
}
```

**jsp页面**

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN""http://www.w3.org/TR/html4/loose.dtd">
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>执行失败</title>
    </head>
    <body>
    	执行失败！
    	${message }
    </body>
</html>
```

## 2.2 自定义异常处理器

```java
/**
* 自定义异常处理器
*/
public class CustomExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request,
                                         HttpServletResponse response, 
                                         Object handler, Exception ex) {
        ex.printStackTrace();
        CustomException customException = null;
        //如果抛出的是系统自定义异常则直接转换
        if(ex instanceof CustomException){
        	customException = (CustomException)ex;
        }else{
        	//如果抛出的不是系统自定义异常则重新构造一个系统错误异常。
        	customException = new CustomException("系统错误，请与系统管理 员联系！");
        }
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("message", customException.getMessage());
        modelAndView.setViewName("error");
        return modelAndView;
    }
}
```

## 2.3 在springmvc.xml配置全局异常处理器

```xml
<!-- 配置自定义异常处理器 -->
<bean id="handlerExceptionResolver" class="cn.krislin.exception.CustomExceptionResolver"/>
```

# 3.异常测试

在controller、service、dao中任意一处需要手动抛出异常。如果是程序中手动抛出的异常，在错误页面中显示自定义的异常信息，如果不是手动抛出异常说明是一个运行时异常，在错误页面只显示“未知错误”。

- 在商品修改的controller方法中抛出异常 .

```java
public String editItems(Model model,@RequestParam(value="id",required=true) Integer items_id)throws Exception {

    //调用service根据商品id查询商品信息
    ItemsCustom itemsCustom = itemsService.findItemsById(items_id);

    //判断商品是否为空，根据id没有查询到商品，抛出异常，提示用户商品信息不存在
    if(itemsCustom == null){
		throw new CustomException("修改的商品信息不存在!");
    }

    //通过形参中的model将model数据传到页面
    //相当于modelAndView.addObject方法
    model.addAttribute("items", itemsCustom);

    return "items/editItems";
}
```

- 在service接口中抛出异常：

```java
public ItemsCustom findItemsById(Integer id) throws Exception {
    Items items = itemsMapper.selectByPrimaryKey(id);
    if(items==null){
        throw new CustomException("修改的商品信息不存在!");
    }
    //中间对商品信息进行业务处理
    //....
    //返回ItemsCustom
    ItemsCustom itemsCustom = null;
    //将items的属性值拷贝到itemsCustom
    if(items!=null){
        itemsCustom = new ItemsCustom();
        BeanUtils.copyProperties(items, itemsCustom);
    }

    return itemsCustom;
}
```


- 如果与业务功能相关的异常，建议在service中抛出异常。
- 与业务功能没有关系的异常，建议在controller中抛出。

上边的功能，建议在service中抛出异常。

