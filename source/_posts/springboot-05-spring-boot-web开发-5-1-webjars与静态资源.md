---
title: SpringBoot 05.spring boot web开发 5.1.webjars与静态资源
date: 2022-07-03T04:19:08.638Z
tags: [springboot]
---
# 一、spring boot静态资源

## 静态资源目录

SpringBoot默认配置下，提供了以下几个静态资源目录：

```
/static：  classpath:/static/
/public： classpath:/public/
/resources： classpath:/resources/
/META-INF/resources：classpath:/META-INF/resources/
```

当然，可以通过spring.resources.static-locations配置指定静态文件的位置。**但是要特别注意，一旦自己指定了静态资源目录，系统默认的静态资源目录就会失效。所以系统默认的就已经足够使用了，尽量不要自定义。**

```yaml
    #配置静态资源
    spring:
      resources:
        #指定静态资源目录
        static-locations: classpath:/mystatic/
```

## favicon.ico图标

如果在配置的静态资源目录中有favicon.ico文件，SpringBoot会自动将其设置为应用图标。

## 欢迎页面

SpringBoot支持静态和模板欢迎页，它首先在静态资源目录查看index.html文件做为首页，若未找到则查找index模板。

# 二、使用WebJars管理css&js

**为什么使用 WebJars？**
显而易见，因为简单。但不仅是依赖这么简单：

- 清晰的管理 web 依赖
- 通过 Maven, Gradle 等项目管理工具就可以下载 web 依赖
- 解决 web 组件中传递依赖的问题以及版本问题
- 页面依赖的版本自动检测功能

WebJars是将这些通用的Web前端资源打包成Java的Jar包，然后借助Maven工具对其管理，保证这些Web资源版本唯一性，升级也比较容易。关于webjars资源，有一个专门的网站https://www.webjars.org/，我们可以到这个网站上找到自己需要的资源，在自己的工程中添加入maven依赖，即可直接使用这些资源了。

## 1.pom中引入依赖

我们可以从WebJars官方查看maven依赖，如下图
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200425143510.png)
例如:将jquery引入pom文件中

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.4.1</version>
</dependency>
```

## 2.访问引入的js文件

SpringBoot将webjar中路径`/webjars/**`的访问重定向到项目的`classpath:/META-INF/resources/webjars/*`。例如：在html内访问静态资源可以使用目录`/webjars/jquery/3.4.1/jquery.js`.

```html
<script src="/webjars/jquery/3.4.1/jquery.min.js "></script>
```

# 三、自动检测依赖的版本

如果使用 Spring 4.2 以上的版本，并且加入 webjars-locator 组件，就不需要在 html 添加依赖的时候填写版本。

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>webjars-locator</artifactId>
    <version>0.30</version>
</dependency>
```

引入 webjars-locator 值后可以省略版本号:

```html
<script src="/webjars/jquery/jquery.min.js"></script>
```

注意：只能去掉版本号