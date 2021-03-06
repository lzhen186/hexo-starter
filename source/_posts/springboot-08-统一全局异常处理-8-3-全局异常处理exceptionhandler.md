---
title: SpringBoot 08.统一全局异常处理 8.3.全局异常处理ExceptionHandler
date: 2022-07-03T07:17:12.291Z
tags: [springboot]
---
# 一、全局异常处理器

ControllerAdvice注解的作用就是监听所有的Controller，一旦Controller抛出CustomException，就会在@ExceptionHandler(CustomException.class)对该异常进行处理。

```java
@ControllerAdvice
public class WebExceptionHandler {

    @ExceptionHandler(CustomException.class)
    @ResponseBody
    public AjaxResponse customerException(CustomException e) {
        if(e.getCode() == CustomExceptionType.SYSTEM_ERROR.getCode()){
                 //400异常不需要持久化，将异常信息以友好的方式告知用户就可以
                //TODO 将500异常信息持久化处理，方便运维人员处理
        }
        return AjaxResponse.error(e);
    }

    @ExceptionHandler(Exception.class)
    @ResponseBody
    public AjaxResponse exception(Exception e) {
        //TODO 将异常信息持久化处理，方便运维人员处理

        //没有被程序员发现，并转换为CustomException的异常，都是其他异常或者未知异常.
        return AjaxResponse.error(new CustomException(CustomExceptionType.OTHER_ERROR,"未知异常"));
    }


}
```

# 二、人为制造异常测试一下

```java
@Service
public class ExceptionService {

    //服务层，模拟系统异常
    public void systemBizError() throws CustomException {
        try {
            Class.forName("com.mysql.jdbc.xxxx.Driver");
        } catch (ClassNotFoundException e) {
            throw new CustomException(CustomExceptionType.SYSTEM_ERROR,"在XXX业务，myBiz()方法内，出现ClassNotFoundException");
        }
    }

     //服务层，模拟用户输入数据导致的校验异常
    public List<String> userBizError(int input) throws CustomException {
        if(input < 0){ //模拟业务校验失败逻辑
            throw new CustomException(CustomExceptionType.USER_INPUT_ERROR,"您输入的数据不符合业务逻辑，请确认后重新输入！");
        }else{ //
            List<String> list = new ArrayList<>();
            list.add("科比");
            list.add("詹姆斯");
            list.add("库里");
            return list;
        }
    }

}
```