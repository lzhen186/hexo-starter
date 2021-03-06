---
title: SpringBoot 08.统一全局异常处理 8.5.AOP实现完美异常处理方案
date: 2022-07-03T07:24:00.714Z
tags: [springboot]
---
# 一、页面类异常处理

之前章节给大家讲的都是接口类的异常处理，那我们做页面模板时，Controller发生异常我们该怎么办?应该统一跳转到404页面。
面临的问题：
**程序员抛出自定义异常CustomException，全局异常处理截获之后返回@ResponseBody AjaxResponse，不是ModelAndView，所以我们无法跳转到error.html页面，那我们该如何做页面的全局的异常处理？**
答：

1. 用面向切面的方式，将CustomException转换为ModelAndViewException。
2. 全局异常处理器拦截ModelAndViewException，返回ModelAndView，即error.html页面
3. 切入点是带@ModelView注解的Controller层方法

**使用这种方法处理页面类异常，程序员只需要在页面跳转的Controller上加@ModelView注解即可**

### 错误的写法

```java
@GetMapping("/freemarker")
public String index(Model model) {
    try{
        List<ArticleVO> articles = articleRestService.getAll();
        model.addAttribute("articles", articles);
    }catch (Exception e){
        return "error";
    }
    return "fremarkertemp";
}
```

### 正确的写法

```java
@ModelView
@GetMapping("/freemarker")
public String index(Model model) {
    List<ArticleVO> articles = articleRestService.getAll();
    model.addAttribute("articles", articles);
    return "fremarkertemp";
}
```

# 二、用面向切面的方法处理页面全局异常

因为用到了面向切面编程，所以引入maven依赖包

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
```

新定义一个异常类ModelViewException

```java
public class ModelViewException extends RuntimeException{

    //异常错误编码
    private int code ;
    //异常信息
    private String message;

    public static ModelViewException transfer(CustomException e) {
        return new ModelViewException(e.getCode(),e.getMessage());
    }

    private ModelViewException(int code, String message){
        this.code = code;
        this.message = message;
    }

    int getCode() {
        return code;
    }

    @Override
    public String getMessage() {
        return message;
    }

}
```

ModelView 注解，只起到标注的作用

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})//只能在方法上使用此注解
public @interface ModelView {

}
```

以@ModelView注解为切入点，面向切面编程,将CustomException转换为ModelViewException抛出。

```java
@Aspect
@Component
@Slf4j
public class ModelViewAspect {
    
    //设置切入点：这里直接拦截被@ModelView注解的方法
    @Pointcut("@annotation(club.krislin.exception.ModelView)")
    public void pointcut() { }
    
    /**
     * 当有ModelView的注解的方法抛出异常的时候，做如下的处理
     */
    @AfterThrowing(pointcut="pointcut()",throwing="e")
    public void afterThrowable(Throwable e) {
        log.error("切面发生了异常：", e);
        if(e instanceof  CustomException){
            throw ModelViewException.transfer((CustomException) e);
        }
    }
}
```

全局异常处理器:

```java
@ExceptionHandler(ModelViewException.class)
    public ModelAndView viewExceptionHandler(HttpServletRequest req, ModelViewException e) {
        ModelAndView modelAndView = new ModelAndView();

        //将异常信息设置如modelAndView
        modelAndView.addObject("exception", e);
        modelAndView.addObject("url", req.getRequestURL());
        modelAndView.setViewName("error");

        //返回ModelAndView
        return modelAndView;
    }
```