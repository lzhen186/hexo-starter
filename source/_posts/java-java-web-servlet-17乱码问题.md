---
title: Java Java Web Servlet 17乱码问题
date: 2022-07-06T01:46:05.299Z
tags: [javaweb]
---
# 请求乱码
## get请求：

　　　　经过了两次编码，所以就要两次解码

　　　　第一次解码：xxx.getBytes("ISO-8859-1");得到yyy

　　　　第二次解码：new String(yyy,"utf-8");

　　　    连续写：new String(xxx.getBytes("ISO-8859-1"),"UTF-8");

​			解决：修改tomcat的conf目录下的server.xml，在8080端口前加上`URIEncoding="UTF-8"`

## post请求：
只经过一次编码，所以也就只要一次解码,使用ServletAPI
`request.setCharacterEncoding()`;
~~~java
//不一定解决，取决于浏览器是用什么码表来编码，浏览器用UTF-8，那么这里就写UTF-8。
request.setCharacterEncoding("UTF-8");　　
~~~

# 响应乱码

* `getOutputStream();`
 使用该字节输出流，不能直接输出中文，会出异常，要想输出中文，解决方法如下
`getOutputStream().write(xxx.getBytes("UTF-8"));`　　
手动将中文用UTF-8码表编码，变成字节传输，变成字节后，就不会报异常，并且tomcat也不会在编码，因为已经编码过了，所以到浏览器后，如果浏览器使用的是UTF-8码表解码，那么就不会出现中文乱码，反之则出现中文乱码，所以这个方法，不能完全保证中文不乱码

* `getWrite();`
使用字符输出流，能直接输出中文，不会出异常，但是会出现乱码。能用三种方法解决，一直使用第二种方法
解决：通知tomcat和浏览器使用同一张码表。
~~~java
//通知浏览器使用UTF-8解码 
response.setContentType("text/html;charset=utf-8");
~~~
通知tomcat和浏览器使用UTF-8编码和解码。这个方法的底层原理是这句话：`response.setHeader("contentType","text/html;charset=utf-8"); `