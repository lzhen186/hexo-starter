---
title: SpringBoot 05.spring boot web开发 5.0.web开发原理
date: 2022-07-03T04:04:53.060Z
tags: [springboot]
---
# 1. 简介

1. 创建Spring Boot应用，选中我们需要的模块
2. Spring Boot已经默认将这些场景配置好了，只需要在配置文件中指定少量配置就可以运行起来
3. 自己编写业务代码

**自动配置原理？**

这个场景Spring Boot帮我们配置了什么？能不能修改？能修改哪些配置？能不能扩展？xxx

```
xxxxAutoConfiguration：帮我们给容器中自动配置组件；
xxxxProperties:配置类来封装配置文件的内容；
```

# 2. web自动配置规则

1. `WebMvcAutoConfiguration`
2. `WebMvcProperties`
3. `ViewResolver`自动配置
4. 静态资源自动映射
5. Formatter与Converter自动配置
6. `HttpMessageConverter`自动配置
7. 静态首页
8. favicon
9. 错误处理

# 3. Spring Boot对静态资源的映射规则

```java
@ConfigurationProperties(
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties {
    //可以设置和静态资源有关的参数，缓存时间等
```



`WebMvcAutoConfiguration`类的`addResourceHandlers`方法：（添加资源映射）

```java
        //配置欢迎页映射
		@Bean
        public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext, FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
            WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(new TemplateAvailabilityProviders(applicationContext), applicationContext, this.getWelcomePage(), this.mvcProperties.getStaticPathPattern());
            welcomePageHandlerMapping.setInterceptors(this.getInterceptors(mvcConversionService, mvcResourceUrlProvider));
            welcomePageHandlerMapping.setCorsConfigurations(this.getCorsConfigurations());
            return welcomePageHandlerMapping;
        }


public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
            } else {
                Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
                CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
                //webjars映射
                if (!registry.hasMappingForPattern("/webjars/**")) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

                String staticPathPattern = this.mvcProperties.getStaticPathPattern();
                //静态资源文件夹映射
                if (!registry.hasMappingForPattern(staticPathPattern)) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

            }
        }
```

所有 `/webjars/**` ，都去 `classpath:/META-INF/resources/webjars/` 找资源

`webjars`：以jar包的方式引入静态资源；

[webjars官网](https://www.webjars.org/)

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525092551.png)

例如：添加jquery的webjars

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.4.1</version>
</dependency>
```

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525093050.png)

访问地址对应就是：http://localhost:8080/webjars/jquery/3.4.1/jquery.js

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525093242.png)

# 4. 非webjars，自己的静态资源怎么访问

**资源配置类：**

```java
@ConfigurationProperties(    //说明可以在配置文件中配置相关参数
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties {
    private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
    private String[] staticLocations;
    private boolean addMappings;
    private final ResourceProperties.Chain chain;
    private final ResourceProperties.Cache cache;

    public ResourceProperties() {
        this.staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
        this.addMappings = true;
        this.chain = new ResourceProperties.Chain();
        this.cache = new ResourceProperties.Cache();
    }
```

`WebMvcAuotConfiguration`:

```java
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
            } else {
                Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
                CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
                if (!registry.hasMappingForPattern("/webjars/**")) {
                    //添加webjars资源映射
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

                String staticPathPattern = this.mvcProperties.getStaticPathPattern();
                if (!registry.hasMappingForPattern(staticPathPattern)) {
                    //添加静态资源映射
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(WebMvcAutoConfiguration.getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

            }
        }
```

上面代码中添加的映射访问路径`staticPathPattern`值是`/**`，对应的资源文件夹就是上面配置类`ResourceProperties`中的`CLASSPATH_RESOURCE_LOCATIONS`数组中的文件夹：

| 数组中的值                     | 在项目中的位置                         |
| ------------------------------ | -------------------------------------- |
| classpath:/META-INF/resources/ | src/main/resources/META-INF/resources/ |
| classpath:/resources/          | src/main/resources/resources/          |
| classpath:/static/             | src/main/resources/static/             |
| classpath:/public/             | src/main/resources/public/             |

localhost:8080/abc ---> 去静态资源文件夹里面找abc

# 5. 欢迎页映射

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525095930.png)

`location`就是静态资源路径，所以欢迎页的页面就是上面静态资源下的`index.html`，被`/**`映射，因此直接访问项目就是访问欢迎页

# 6. 网站图标映射（favicon.ico）

所有的 favicon.ico 都是在静态资源文件下找；