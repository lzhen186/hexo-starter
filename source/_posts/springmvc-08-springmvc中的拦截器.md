---
title: SpringMVC 08.SpringMVC中的拦截器
date: 2022-07-02T06:43:35.157Z
tags: [springmvc]
---
# 1.拦截器的作用

Spring MVC 的处理器拦截器类似于 Servlet 开发中的过滤器 Filter，用于对处理器进行预处理和后处理。
用户可以自己定义一些拦截器来实现特定的功能。
谈到拦截器，还要向大家提一个词——拦截器链（Interceptor Chain）。拦截器链就是将拦截器按一定的顺
序联结成一条链。在访问被拦截的方法或字段时，拦截器链中的拦截器就会按其之前定义的顺序被调用。

Spring MVC 的处理器拦截器与过滤器的区别：
	过滤器是 servlet 规范中的一部分，任何 java web 工程都可以使用。
	拦截器是 SpringMVC 框架自己的，只有使用了 SpringMVC 框架的工程才能用。
	过滤器在 url-pattern 中配置了/*之后，可以对所有要访问的资源拦截。
	拦截器它是只会拦截访问的控制器方法，如果访问的是 jsp,html,css,image 或者 js 是不会进行拦截的。

它也是 AOP 思想的具体应用。
我们要想自定义拦截器， 要求必须实现：HandlerInterceptor 接口。

# 2.自定义拦截器的步骤

## 1.编写一个普通类实现HandlerInterceptor接口

```java
/**
* 自定义拦截器
*/
public class HandlerInterceptorDemo1 implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, 
                             Object handler) throws Exception {
    	System.out.println("preHandle 拦截器拦截了");
    	return true;
    }
    
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, 
                           Object handler, ModelAndView modelAndView) throws Exception {
    	System.out.println("postHandle 方法执行了");
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, 
                                Object handler, Exception ex)throws Exception {
    	System.out.println("afterCompletion 方法执行了");
    }
}
```

## 2.配置拦截器

```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean id="handlerInterceptorDemo1"class="cn.krislin.web.interceptor.HandlerInterceptorDemo1">
        </bean>
    </mvc:interceptor>
</mvc:interceptors>
```

# 3.拦截器的细节

## 3.1 拦截器的放行

放行的含义是指，如果有下一个拦截器就执行下一个，如果该拦截器处于拦截器链的最后一个，则执行控制器
中的方法。

```java
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        System.out.println("preHandle 拦截器拦截了");
        return false;
    }
```

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200727160031.png)

## 3.2 拦截器中的方法说明

```java
public interface HandlerInterceptor {
    /**
    * 如何调用：
    * 		按拦截器定义顺序调用
    * 何时调用：
    * 		只要配置了都会调用
    * 有什么用：
    * 		如果程序员决定该拦截器对请求进行拦截处理后还要调用其他的拦截器，或者是业务处理器去进行处理，则返回 			  true。
    * 		如果程序员决定不需要再调用其他的组件去处理请求，则返回 false。
    */
    default boolean preHandle(HttpServletRequest request, 
                              HttpServletResponse response, Object handler)throws Exception {
    	return true;
    }
    /**
    * 如何调用：
    * 		按拦截器定义逆序调用
    * 何时调用：
    * 		在拦截器链内所有拦截器成功调用
    * 有什么用：
    * 		在业务处理器处理完请求后，但是 DispatcherServlet 向客户端返回响应前被调用，
    * 		在该方法中对用户请求 request 进行处理。
    */
    default void postHandle(HttpServletRequest request, HttpServletResponse response, 
                            Object handler, @Nullable ModelAndView modelAndView) throws Exception {
    }
    /**
    * 如何调用：
    * 		按拦截器定义逆序调用
    * 何时调用：
    * 		只有 preHandle 返回 true 才调用
    * 有什么用：
    * 		在 DispatcherServlet 完全处理完请求后被调用，
    * 		可以在该方法中进行一些资源清理的操作。
    */
    default void afterCompletion(HttpServletRequest request, HttpServletResponseresponse, 
                                 Object handler, @Nullable Exception ex) throws Exception {
    }
}
```

## 3.3 拦截器的作用路径

作用路径可以通过在配置文件中配置。

```xml
<!-- 配置拦截器的作用范围 -->
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**" /><!-- 用于指定对拦截的 url -->
        <mvc:exclude-mapping path=""/><!-- 用于指定排除的 url-->
        <bean id="handlerInterceptorDemo1"
        class="cn.krislin.web.interceptor.HandlerInterceptorDemo1"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

## 3.4 多个拦截器的执行顺序

**思考**：
如果有多个拦截器，这时拦截器 1 的 preHandle 方法返回 true，但是拦截器 2 的 preHandle 方法返回 false，而此时拦截器 1 的 afterCompletion 方法是否执行？

多个拦截器是按照配置的顺序决定的。

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200727160947.png)

# 4.正常流程测试

## 4.1 配置文件

```xml
<!--配置拦截器-->
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean id="handlerInterceptorDemo1" class="cn.krislin.web.interceptor.HandlerInterceptorDemo1"></bean>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean id="handlerInterceptorDemo2" class="cn.krislin.web.interceptor.HandlerInterceptorDemo2"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

## 4.2 拦截器1的代码

```java
package cn.krislin.web.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @Package cn.krislin.web.interceptor
 * @ClassName HandlerInterceptorDemo1
 * @Description TODO
 * @Date 20/7/27 15:51
 * @Author krislin
 * @Version V1.0
 */
public class HandlerInterceptorDemo1 implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        System.out.println("拦截器1:preHandle 拦截器拦截了");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器1:postHandle 方法执行了");
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler, Exception ex) throws Exception {
        System.out.println("拦截器1:afterCompletion 方法执行了");
    }
}
```

## 4.3 拦截器2的代码

```java
package cn.krislin.web.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @Package cn.krislin.web.interceptor
 * @ClassName HandlerInterceptorDemo2
 * @Description TODO
 * @Date 20/7/27 16:13
 * @Author krislin
 * @Version V1.0
 */
public class HandlerInterceptorDemo2 implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        System.out.println("拦截器2:preHandle 拦截器拦截了");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器2:postHandle 方法执行了");
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler, Exception ex) throws Exception {
        System.out.println("拦截器2:afterCompletion 方法执行了");
    }
}
```

## 4.4 执行结果

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200727161736.png)

# 5.中断流程测试

## 5.1 配置文件

```xml
<!--配置拦截器-->
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean id="handlerInterceptorDemo1" class="cn.krislin.web.interceptor.HandlerInterceptorDemo1"></bean>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean id="handlerInterceptorDemo2" class="cn.krislin.web.interceptor.HandlerInterceptorDemo2"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

## 5.2 拦截器1的代码

```java
package cn.krislin.web.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @Package cn.krislin.web.interceptor
 * @ClassName HandlerInterceptorDemo1
 * @Description TODO
 * @Date 20/7/27 15:51
 * @Author krislin
 * @Version V1.0
 */
public class HandlerInterceptorDemo1 implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        System.out.println("拦截器1:preHandle 拦截器拦截了");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器1:postHandle 方法执行了");
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler, Exception ex) throws Exception {
        System.out.println("拦截器1:afterCompletion 方法执行了");
    }
}
```

## 5.3 拦截器2的代码

```java
package cn.krislin.web.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @Package cn.krislin.web.interceptor
 * @ClassName HandlerInterceptorDemo2
 * @Description TODO
 * @Date 20/7/27 16:13
 * @Author krislin
 * @Version V1.0
 */
public class HandlerInterceptorDemo2 implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) throws Exception {
        System.out.println("拦截器2:preHandle 拦截器拦截了");
        return false;
    }

    @Override
    public void postHandle(HttpServletRequest request,
                           HttpServletResponse response,
                           Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器2:postHandle 方法执行了");
    }

    @Override
    public void afterCompletion(HttpServletRequest request,
                                HttpServletResponse response,
                                Object handler, Exception ex) throws Exception {
        System.out.println("拦截器2:afterCompletion 方法执行了");
    }
}
```

## 5.4 执行结果

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200727162308.png)

# 6.拦截器的简单案例(验证用户是否登陆)

## 6.1 实现思路

1. 有一个登录页面，需要写一个 controller 访问页面

2. 登录页面有一提交表单的动作。需要在 controller 中处理。

   2.1. 判断用户名密码是否正确

   2.2. 如果正确 向 session 中写入用户信息

   2.3. 返回登录成功。

3. 拦截用户请求，判断用户是否登录

   3.1. 如果用户已经登录。放行

   3.2. 如果用户未登录，跳转到登录页面

## 6.2 控制器代码

```java
package cn.krislin.web.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

/**
 * @Package cn.krislin.web.interceptor
 * @ClassName LoginInterceptor
 * @Description TODO
 * @Date 20/7/27 16:45
 * @Author krislin
 * @Version V1.0
 */
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //如果是登陆页面则放行
        if (request.getRequestURI().indexOf("login.action")>0){
            return true;
        }
        //如果用户已登陆也放行
        HttpSession session = request.getSession();
        if (session.getAttribute("activeUser")!=null){
            return true;
        }
        //用户没有登陆就跳转到登陆页面
        request.getRequestDispatcher("/WEB-INF/pages/login.jsp").forward(request,response);
        return false;
    }
}
```

## 6.3 配置拦截器

```xml
    <!--配置拦截器-->
    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**"/>
            <bean id="loginInterceptor" class="cn.krislin.web.interceptor.LoginInterceptor"></bean>
        </mvc:interceptor>
    </mvc:interceptors>
```

