---
title: SpringBoot 06.生命周期内的拦截过滤与监听 6.1.servlet与filter与listener的实现
date: 2022-07-03T06:22:39.594Z
tags: [springboot]
---
# 一、监听器

## 1.1 定义

Servlet 监听器是 Servlet 规范中定义的一种特殊类，用于监听 ServletContext、HttpSession 和 ServletRequest 等域对象的创建与销毁事件，以及监听这些域对象中属性发生修改的事件。监听器是观察者模式的应用，它关注特定事物，并伺机而动，所以监听器具有**异步**的特性。

Servlet Listener 监听三大域对象的创建和销毁事件，三大对象分别是：

1. ServletContext：application 级别，整个应用只存在一个，可以进行全局应用配置。
2. HttpSession：session 级别，针对每一个对话，如统计会话总数。
3. ServletRequest：request 级别，针对每一个客户请求，

## 1.2 使用场景

Servlet 规范设计监听器的作用是在事件发生前、发生后进行一些处理，一般可以用来统计在线人数和在线用户、统计网站访问量、系统启动时初始化信息等。我们可以在容器启动时初始化 Log4j 信息，添加自己对容器状态的监控，初始化 Spring 组件等。

## 1.3 监听器的实现

创建一个ServletRequest监听器(其他监听器类似创建)

```java
@WebListener
@Slf4j
public class Customlister implements ServletRequestListener{

    @Override
    public void requestDestroyed(ServletRequestEvent sre) {
        log.info(" request监听器：销毁");
    }

    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        log.info(" request监听器：可以在这里记录访问次数哦");
    }

}
```

在启动类中加入@ServletComponentScan进行自动注册即可。

# 二、过滤器

## 2.1 定义

过滤器是一种可重用的代码，可以转换 HTTP 请求、响应和头信息，通俗来说就是过滤器可以在请求到达服务器之前，对请求头进行预先处理，在响应内容到达客户端之前，对服务器做出的响应进行后置处理。
根据这个定义，我们就不难理解为什么它是位于 Listener 的下游，Servlet 的上游了？过滤器必须是在容器启动之后，接收到请求后开始处理，所以它是在 Listener 执行之后；在请求到达 Servlet 之前进行预处理，所以它又处于 Servlet 之前的位置。
从命名上理解，它就是对**请求和响应的数据**内容进行过滤处理的，并非前端传过来的数据都全单接收，而是有原则地进行过滤处理，以保证后端服务器业务的安全。

## 2.2 使用场景

Servlet 3.1 中定义了几种常见的过滤器组件：

| 过滤器                                        | 作用                         |
| :-------------------------------------------- | :--------------------------- |
| Authentication filters：                      | 授权类，如用户登陆会话校验； |
| Logging and auditing filters：                | 日志和安全审计类；           |
| Image conversion filters：                    | 图片转换；                   |
| Data compression filters：                    | 数据压缩；                   |
| Encryption filters：                          | 加密、解密类；               |
| Tokenizing filters：                          | 词法类；                     |
| Filters that trigger resource access events： | 触发资源访问事件类；         |
| XSL/T filters that transform XML content：    | XML文件转换类；              |
| MIME-type chain filters：                     | MIME文件；                   |
| Caching filters：                             | 缓存类；                     |

或者我们社交应用经常需要的敏感词过滤，都可以使用过滤器。过滤器主要的特点在于，它能够改变请求内容。

## 2.3 过滤器的实现

过滤器Filter，是Servlet的的一个实用技术了。可通过过滤器，对请求进行拦截，比如读取session判断用户是否登录、判断访问的请求URL是否有访问权限(黑白名单)等。主要还是可对请求进行预处理。接下来介绍下，在springboot如何实现过滤器功能。

**1.1 实现方式一:利用WebFilter注解配置**

@WebFilter是Servlet3.0新增的注解，原先实现过滤器，需要在web.xml中进行配置，而现在通过此注解，启动启动时会自动扫描自动注册。

编写Filter类：

```java
//注册器名称为customFilter,拦截的url为所有
@WebFilter(filterName="customFilter",urlPatterns={"/*"})
@Slf4j
public class CustomFilter implements Filter{

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("filter 初始化");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        log.info("customFilter 请求处理之前");
        //对request、response进行一些预处理
        // 比如设置请求编码
        // request.setCharacterEncoding("UTF-8");
        // response.setCharacterEncoding("UTF-8");

        //链路 直接传给下一个过滤器
        chain.doFilter(request, response);

        log.info("customFilter 请求处理之后");
    }

    @Override
    public void destroy() {
        log.info("filter 销毁");
    }
}
```

然后在启动类加入@ServletComponentScan注解即可。
使用这种方法，当注册多个过滤器时，无法指定执行顺序的，原本使用web.xml配置过滤器时，是可指定执行顺序的，但使用@WebFilter时，没有这个配置属性的(需要配合@Order进行)，所以接下来介绍下通过FilterRegistrationBean进行过滤器的注册。

> –小技巧–
> 通过过滤器的java类名称，进行顺序的约定，比如LogFilter和AuthFilter，此时AuthFilter就会比LogFilter先执行，因为首字母A比L前面。

**1.2.FilterRegistrationBean方式**
FilterRegistrationBean是springboot提供的，此类提供setOrder方法，可以为filter设置排序值，让spring在注册web filter之前排序后再依次注册。
首先要改写filter, 其实就除了@webFilter注解即可。其他的都没有变化。启动类中利用@bean注册FilterRegistrationBean

```java
@Configuration
public class FilterRegistration {
    @Bean
    public FilterRegistrationBean filterRegistrationBean() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        //当过滤器有注入其他bean类时，可直接通过@bean的方式进行实体类过滤器，这样不可自动注入过滤器使用的其他bean类。
        //当然，若无其他bean需要获取时，可直接new CustomFilter()，也可使用getBean的方式。
        registration.setFilter(customFilter());
        //过滤器名称
        registration.setName("customFilter");
        //拦截路径
        registration.addUrlPatterns("/*");
        //设置顺序
        registration.setOrder(10);
        return registration;
    }

    @Bean
    public Filter customFilter() {
        return new CustomFilter();
    }
}
```

注册多个时，就注册多个FilterRegistrationBean即可,启动后，效果和第一种是一样的。

# 三、servlet

## 3.1定义：

在java程序员10年以前做web开发的时候，所有的请求都是由servlet来接受并响应的。每一个请求，就要写一个servlet。这种方式很麻烦，大家就想能不能根据请求的路径以及参数不同，映射到不同的方法上去执行，这样就可以在一个servlet类里面处理多个请求，每个请求就是一个方法。这个思想后来就主键发展为structs、SpringMVC。

## 3.2使用场景

目前来看，servlet使用的场景已经被springMVC架构全面覆盖。但是不排除，老项目向spring boot项目迁移，需要支持servlet的情况。

## 3.3 实现

下面我们就看一下，在spring boot里面如何实现servlet。

```java
@WebServlet(name = "firstServlet", urlPatterns = "/firstServlet") //标记为servlet，以便启动器扫描。
public class FirstServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().append("firstServlet");
    }

}
```

然后在启动类加入@ServletComponentScan注解即可。