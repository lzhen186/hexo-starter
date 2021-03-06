---
title: SpringBoot 08.统一全局异常处理 8.2.自定义异常和相关数据结构
date: 2022-07-03T07:10:28.200Z
tags: [springboot]
---
# 一、该如何设计数据结构

1. CustomException 自定义异常。核心要素：异常错误编码（200正常,400,500），异常错误信息message。
2. ExceptionTypeEnum 枚举异常分类，将异常分类固化下来，防止开发人员思维发散。 核心要素 异常分类编码（200正常,400,500），异常分类描述。
3. AjaxResponse 用于响应Ajax请求。核心要素：是否请求成功 isok；响应code零与非零，零表示成功（200,400,500）；响应成功与否信息描述message；响应成功的数据data。
4. error.html
   另外还需要有一个统一处理CustomException的地方。即@ControllerAdvice和@ExceptionHandler，后文会说明

# 二、枚举异常的类型

为了防止开发人员大脑发散，每个开发人员都不断的发明自己的异常类型，我们需要规定好异常的类型(枚举)。比如：系统异常、用户（输入）操作导致的异常、其他异常等。

```java
public enum CustomExceptionType {
    USER_INPUT_ERROR(400,"用户输入异常"),
    SYSTEM_ERROR (500,"系统服务异常"),
    OTHER_ERROR(999,"其他未知异常");

    CustomExceptionType(int code, String typeDesc) {
        this.code = code;
        this.typeDesc = typeDesc;
    }

    private String typeDesc;//异常类型中文描述

    private int code; //code

    public String getTypeDesc() {
        return typeDesc;
    }

    public int getCode() {
        return code;
    }
}
```

- 以笔者的经验，最好不要超过5个，否则开发人员将会记不住，也不愿意去记。对于我来说上面的三种异常类型就足够了。
- 这里的code表示异常类型的唯一编码，为了方便大家记忆，就使用Http状态码400、500

# 三、自定义异常

- 自定义异常有两个核心内容，一个是code。使用CustomExceptionType 来限定范围。
- 另外一个是message，这个message信息是要最后返回给前端的，所以需要用友好的提示来表达异常发生的原因或内容

```java
public class CustomException extends RuntimeException {
    //异常错误编码
    private int code ;
    //异常信息
    private String message;

    private CustomException(){}

    public CustomException(CustomExceptionType exceptionTypeEnum, 
                           String message) {
        this.code = exceptionTypeEnum.getCode();
        this.message = message;
    }

    public int getCode() {
        return code;
    }

    @Override
    public String getMessage() {
        return message;
    }
}
```

# 四、统一响应数据结构

为了解决不同的开发人员使用不同的结构来响应给前端，导致规范不统一，开发混乱的问题。我们使用如下代码定义统一数据响应结构

- isok表示该请求是否处理成功（即是否发生异常）。true表示请求处理成功，false表示处理失败。
- code对响应结果进一步细化，200表示请求成功，400表示用户操作导致的异常，500表示系统异常，999表示其他异常。与CustomExceptionType枚举一致。
- message：友好的提示信息，或者请求结果提示信息。如果请求成功这个信息通常没什么用，如果请求失败，该信息需要展示给用户。
- data：通常用于查询数据请求，成功之后将查询数据响应给前端。

```java
/**
 * 接口数据请求统一响应数据结构
 */
@Data
public class AjaxResponse {

    private  boolean isok;
    private  int code;
    private  String message;
    private  Object data;

    private AjaxResponse() {

    }

    //请求出现异常时的响应数据封装
    public static AjaxResponse error(CustomException e) {
        AjaxResponse resultBean = new AjaxResponse();
        resultBean.setIsok(false);
        resultBean.setCode(e.getCode());
        if(e.getCode() == CustomExceptionType.USER_INPUT_ERROR.getCode()){
            resultBean.setMessage(e.getMessage());
        }else if(e.getCode() == CustomExceptionType.SYSTEM_ERROR.getCode()){
            resultBean.setMessage(e.getMessage() + ",请将该异常信息发送给管理员!");
        }else{
            resultBean.setMessage("系统出现未知异常，请联系管理员!");
        }
        //TODO 这里最好将异常信息持久化
        return resultBean;
    }

    //请求出现异常时的响应数据封装
    public static AjaxResponse error(CustomExceptionType customExceptionType,
                                     String errorMessage) {
        AjaxResponse resultBean = new AjaxResponse();
        resultBean.setIsok(false);
        resultBean.setCode(customExceptionType.getCode());
        resultBean.setMessage(errorMessage);
        return resultBean;
    }

    //请求处理成功时的数据响应
    public static AjaxResponse success() {
        AjaxResponse resultBean = new AjaxResponse();
        resultBean.setIsok(true);
        resultBean.setCode(200);
        resultBean.setMessage("success");
        return resultBean;
    }

    //请求处理成功，并响应结果数据
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

对于不同的场景，提供了四种构建AjaxResponse 的方法。

- 当请求成功的情况下，可以使用`AjaxResponse.success()`构建返回结果给前端。
- 当查询请求等需要返回业务数据，请求成功的情况下，可以使用`AjaxResponse.success(data)`构建返回结果给前端。携带结果数据。
- 当请求处理过程中发生异常，需要将异常转换为CustomException ，然后在控制层使用`AjaxResponse error(CustomException)`构建返回结果给前端。
- 在某些情况下，没有任何异常产生，我们判断某些条件也认为请求失败。这种使用`AjaxResponse error(customExceptionType,errorMessage)`构建响应结果。

# 五、使用示例

例如：更新操作，Controller无需返回额外的数据

```java
return AjaxResponse.success();
```

​												![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200427104313.png)
例如：查询接口，Controller需返回结果数据(data可以是任何类型数据)

```java
 return AjaxResponse.success(data);
```

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200427104408.png)