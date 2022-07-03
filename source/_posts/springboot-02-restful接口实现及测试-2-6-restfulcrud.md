---
title: SpringBoot 02.RESTful接口实现及测试 2.6.RESTfulCRUD
date: 2022-07-02T11:37:42.780Z
tags: [springboot]
---
1. 将静态资源(css,img,js)添加到项目中，放到springboot默认的静态资源文件夹下
2. 将模板文件(html)放到template文件夹下



> 如果你的静态资源明明放到了静态资源文件夹下却无法访问，请检查一下是不是在自定义的配置类上加了**@EnableWebMvc注解**