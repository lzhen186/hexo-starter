---
title: SpringBoot 06.生命周期内的拦截过滤与监听 6.2.spring拦截器及请求链路说明
date: 2022-07-03T06:35:06.394Z
tags: [springboot]
---
# 一、拦截器

## 定义

在 Servlet 规范中并没有拦截器的概念，它是面向切面编程的一种应用：在需要对方法进行增强的场景下，例如在方法调用前执行一段代码，或者在方法完成后额外执行一段操作，拦截器的一种实现方式就是动态代理。
把过滤器和拦截器放在一起比较，我觉得是没有意义的，本质上是不同概念，并没有可比性，它位于过滤器的下游，是面向 Servlet 方法的。

## 使用场景

AOP 编程思想面对的是横向的切面，而非纵向的业务。举个简单的例子，每个方法处理过程中，除了业务逻辑外，我们都会有一些相同的操作：参数校验，日志打印等，虽然这些处理代码并不多，但是每个方法都要写这类东西，工作量就不小了。
能否使用程序来统一加入这类操作，而不用程序员自己手写呢？这就是切面编程思想的应用，利用 Java 的代理，在调用真正的方法之前或者之后，添加一些额外的增强功能。

# 二、拦截器的实现

以上的过滤器、监听器都属于Servlet的api，我们在开发中处理利用以上的进行过滤web请求时，还可以使用Spring提供的拦截器(HandlerInterceptor)进行更加精细的控制。

编写自定义拦截器类

```java
@Slf4j
public class CustomHandlerInterceptor implements HandlerInterceptor{

 @Override
 public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
   throws Exception {
  log.info("preHandle:请求前调用");
  //返回 false 则请求中断
  return true;
 }

 @Override
 public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
   ModelAndView modelAndView) throws Exception {
  log.info("postHandle:请求后调用");

 }

 @Override
 public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
   throws Exception {
  log.info("afterCompletion:请求调用完成后回调方法，即在视图渲染完成后回调");

 }

}
```

通过继承WebMvcConfigurerAdapter注册拦截器。笔者在写作完成后，发现WebMvcConfigurerAdapter类已经被废弃，请实现WebMvcConfigurer接口完成拦截器的注册。

```java
@Configuration
//废弃：public class MyWebMvcConfigurer extends WebMvcConfigurerAdapter{
public class MyWebMvcConfigurer implements WebMvcConfigurer 
 @Override
  public void addInterceptors(InterceptorRegistry registry) {
   //注册拦截器 拦截规则
  //多个拦截器时 以此添加 执行顺序按添加顺序
  registry.addInterceptor(getHandlerInterceptor()).addPathPatterns("/*");
  }
	
 @Bean
 public static HandlerInterceptor getHandlerInterceptor() {
  return new CustomHandlerInterceptor();
 }
}
```

# 三、请求链路说明

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200426151010.png)

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200426151106.png)

![](images/9af9dc6946b149be878a0b3ec1529d33.png)