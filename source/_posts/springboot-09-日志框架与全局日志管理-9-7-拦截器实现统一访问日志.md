---
title: SpringBoot 09.日志框架与全局日志管理 9.7.拦截器实现统一访问日志
date: 2022-07-03T08:14:04.454Z
tags: [springboot]
---
# 一、定义访问日志内容记录实体类

```java
@Data
public class AccessLog {
    //访问者用户名
    private String username;
    //请求路径
    private String url;
    //请求消耗时长
    private Integer duration;
    //http 方法：GET、POST等
    private String httpMethod;
    //http 请求响应状态码
    private Integer httpStatus;
    //访问者ip
    private String ip;
    //此条记录的创建时间
    private Date createTime;
}
```

# 二、自定义日志拦截器

```java
@Slf4j
public class AccessLogInterceptor implements HandlerInterceptor {
    //请求开始时间标识
    private static final String LOGGER_SEND_TIME = "SEND_TIME";
    //请求日志实体标识
    private static final String LOGGER_ACCESSLOG = "ACCESSLOG_ENTITY";

    /**
     * 进入SpringMVC的Controller之前开始记录日志实体
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object o) throws Exception {

        //创建日志实体
        AccessLog accessLog = new AccessLog();

        // 设置IP地址
        accessLog.setIp(AdrressIpUtils.getIpAdrress(request));

        //设置请求方法,GET,POST...
        accessLog.setHttpMethod(request.getMethod());

        //设置请求路径
        accessLog.setUrl(request.getRequestURI());

        //设置请求开始时间
        request.setAttribute(LOGGER_SEND_TIME,System.currentTimeMillis());

        //设置请求实体到request内，方便afterCompletion方法调用
        request.setAttribute(LOGGER_ACCESSLOG,accessLog);
        return true;
    }


    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object o, Exception e) throws Exception {

        //获取本次请求日志实体
        AccessLog accessLog = (AccessLog) request.getAttribute(LOGGER_ACCESSLOG);

        //获取请求错误码，根据需求存入数据库，这里不保存
        int status = response.getStatus();
        accessLog.setHttpStatus(status);

        //设置访问者(这里暂时写死）
        // 因为不同的应用可能将访问者信息放在session里面，有的通过request传递，
        // 总之可以获取到，但获取的方法不同
        accessLog.setUsername("admin");

        //当前时间
        long currentTime = System.currentTimeMillis();

        //请求开始时间
        long snedTime = Long.valueOf(request.getAttribute(LOGGER_SEND_TIME).toString());


        //设置请求时间差
        accessLog.setDuration(Integer.valueOf((currentTime - snedTime)+""));

        accessLog.setCreateTime(new Date());
        //将sysLog对象持久化保存
        log.info(accessLog.toString());
    }
}
```

# 三、拦截器注册

```java
@Configuration
public class MyWebMvcConfigurer implements WebMvcConfigurer {

    //设置排除路径，spring boot 2.*，注意排除掉静态资源的路径，不然静态资源无法访问
    private final String[] excludePath = {"/static"};

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AccessLogInterceptor()).addPathPatterns("/**").excludePathPatterns(excludePath);
    }
}
```

# 四、获取ip访问地址的工具类

```java
public class AdrressIpUtils {
    /**
     * 获取Ip地址
     * @param request HttpServletRequest
     * @return ip
     */
    public static String getIpAdrress(HttpServletRequest request) {
        String Xip = request.getHeader("X-Real-IP");
        String XFor = request.getHeader("X-Forwarded-For");
        if(StringUtils.isNotEmpty(XFor) && !"unKnown".equalsIgnoreCase(XFor)){
            //多次反向代理后会有多个ip值，第一个ip才是真实ip
            int index = XFor.indexOf(",");
            if(index != -1){
                return XFor.substring(0,index);
            }else{
                return XFor;
            }
        }
        XFor = Xip;
        if(StringUtils.isNotEmpty(XFor) && !"unKnown".equalsIgnoreCase(XFor)){
            return XFor;
        }
        if (StringUtils.isBlank(XFor) || "unknown".equalsIgnoreCase(XFor)) {
            XFor = request.getHeader("Proxy-Client-IP");
        }
        if (StringUtils.isBlank(XFor) || "unknown".equalsIgnoreCase(XFor)) {
            XFor = request.getHeader("WL-Proxy-Client-IP");
        }
        if (StringUtils.isBlank(XFor) || "unknown".equalsIgnoreCase(XFor)) {
            XFor = request.getHeader("HTTP_CLIENT_IP");
        }
        if (StringUtils.isBlank(XFor) || "unknown".equalsIgnoreCase(XFor)) {
            XFor = request.getHeader("HTTP_X_FORWARDED_FOR");
        }
        if (StringUtils.isBlank(XFor) || "unknown".equalsIgnoreCase(XFor)) {
            XFor = request.getRemoteAddr();
        }
        return XFor;
    }
}
```

