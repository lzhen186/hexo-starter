---
title: SpringMVC 07.SpringMVC实现文件上传
date: 2022-07-02T06:37:02.776Z
tags: [springmvc]
---
# 1.一般的文件上传

## 1.1 文件上传的必要前提

* form 表单的 enctype 取值必须是：multipart/form-data(默认值是:application/x-www-form-urlencoded) 

  enctype:是表单请求正文的类型

* method 属性取值必须是 Post

* 提供一个文件选择域`<input type="file" />`

## 1.2 文件上传的原理分析

当 form 表单的 enctype 取值不是默认值后，request.getParameter()将失效。

当enctype=”application/x-www-form-urlencoded”时，form 表单的正文内容是：key=value&key=value&key=value

当enctype 取值为 Mutilpart/form-data 时，请求正文内容就变成：每一部分都是 MIME 类型描述的正文

```
-----------------------------7de1a433602ac 			分界符
Content-Disposition: form-data; name="userName" 	协议头
aaa 											协议的正文
-----------------------------7de1a433602ac
Content-Disposition: form-data; name="file";
filename="C:\Users\krislin\Desktop\fileupload_demofile\b.txt"
Content-Type: text/plain 						协议的类型（MIME 类型）
bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
-----------------------------7de1a433602ac--
```

## 1.3 借助第三方组件实现文件上传

使用 Commons-fileupload 组件实现文件上传，需要导入该组件相应的支撑 jar 包：Commons-fileupload 和commons-io。commons-io 不属于文件上传组件的开发 jar 文件，但Commons-fileupload 组件从 1.1 版本开始，它工作时需要 commons-io 包的支持。

# 2.SpringMVC传统方式的文件上传

传统方式的文件上传，指的是我们上传的文件和访问的应用存在于同一台服务器上。并且上传完成之后，浏览器可能跳转。

## 2.1 实现步骤

### 1.添加maven依赖

```xml
<!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.4</version>
</dependency>
<!-- https://mvnrepository.com/artifact/commons-io/commons-io -->
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
```

### 2.编写jsp页面

```jsp
<form action="/fileUpload" method="post" enctype="multipart/form-data">
    名称：<input type="text" name="picname"/><br/>
    图片：<input type="file" name="uploadFile"/><br/>
    <input type="submit" value=" 上传 "/>
</form>
```

### 3.编写控制器

```java
/**
* 文件上传的的控制器
*/
@Controller("fileUploadController")
public class FileUploadController {
    /**
    * 文件上传
    */
    @RequestMapping("/fileUpload")
    public String testResponseJson(String picname,MultipartFile uploadFile,HttpServletRequest request) throws Exception{
        //定义文件名
        String fileName = "";
        //1.获取原始文件名
        String uploadFileName = uploadFile.getOriginalFilename();
        //2.截取文件扩展名
        String extendName = uploadFileName.substring(uploadFileName.lastIndexOf(".")+1,uploadFileName.length());
        //3.把文件加上随机数，防止文件重复
        String uuid = UUID.randomUUID().toString().replace("-", "").toUpperCase();
        //4.判断是否输入了文件名
        if(!StringUtils.isEmpty(picname)) {
        	fileName = uuid+"_"+picname+"."+extendName;
        }else {
        	fileName = uuid+"_"+uploadFileName;
        }
        System.out.println(fileName);
        //2.获取文件路径
        ServletContext context = request.getServletContext();
        String basePath = context.getRealPath("/uploads");
        //3.解决同一文件夹中文件过多问题
        String datePath = new SimpleDateFormat("yyyy-MM-dd").format(new Date());
        //4.判断路径是否存在
        File file = new File(basePath+"/"+datePath);
        if(!file.exists()) {
        	file.mkdirs();
        }
        //5.使用 MulitpartFile 接口中方法，把上传的文件写到指定位置
        uploadFile.transferTo(new File(file,fileName));
        return "success";
        }
}
```

### 4.配置文件解析器

```xml
<!-- 配置文件上传解析器 -->
<bean id="multipartResolver" <!-- id 的值是固定的-->
    class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!-- 设置上传文件的最大尺寸为 5MB -->
    <property name="maxUploadSize">
    	<value>5242880</value>
    </property>
</bean>
```

> 注意：
> 文件上传的解析器 id 是固定的，不能起别的名称，否则无法实现请求参数的绑定。（不光是文件，其他
> 字段也将无法绑定）

# 3.SpringMVC跨服务器方式的文件上传

## 3.1 分服务器的目的

在实际开发中，我们会有很多处理不同功能的服务器。例如：
	应用服务器：负责部署我们的应用
	数据库服务器：运行我们的数据库
	缓存和消息服务器：负责处理大并发访问的缓存和消息
	文件服务器：负责存储用户上传文件的服务器。
(注意：此处说的不是服务器集群)

分服务器处理的目的是让服务器各司其职，从而提高我们项目 的运行效率。

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200725155841.png)

## 3.2 准备个 两个 tomcat 服务器的 ，并创建一个用于存放图片的 web 工程

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200725161011.png)

在文件服务器的 tomcat 配置中加入，允许读写操作。文件位置：

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200725161124.png)

加入内容：

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200725161352.png)

加入此行的含义是：接收文件的目标服务器可以支持写入操作。

## 3.3 添加maven依赖

在我们负责处理文件上传的项目中添加文件上传的必备maven依赖

```xml
    <!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
    <dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
      <version>1.4</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/commons-io/commons-io -->
    <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
      <version>2.6</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.sun.jersey/jersey-client -->
    <dependency>
      <groupId>com.sun.jersey</groupId>
      <artifactId>jersey-client</artifactId>
      <version>1.19.4</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.sun.jersey/jersey-core -->
    <dependency>
      <groupId>com.sun.jersey</groupId>
      <artifactId>jersey-core</artifactId>
      <version>1.19.4</version>
    </dependency>
```

## 3.4 编写控制器实现上传图片

```java
/**
* 响应 json 数据的控制器
*/
@Controller("fileController")
public class FileController {
	public static final String FILESERVERURL ="http://localhost:9090/day06_spring_image/uploads/";
    /**
    * 文件上传，保存文件到不同服务器
    */
    @RequestMapping("/fileUpload2")
    public String testResponseJson(String picname,MultipartFile uploadFile) throws Exception{
        //定义文件名
        String fileName = "";
        //1.获取原始文件名
        String uploadFileName = uploadFile.getOriginalFilename();
        //2.截取文件扩展名
        String extendName =
        uploadFileName.substring(uploadFileName.lastIndexOf(".")+1,
        uploadFileName.length());
        //3.把文件加上随机数，防止文件重复
        String uuid = UUID.randomUUID().toString().replace("-", "").toUpperCase();
        //4.判断是否输入了文件名
        if(!StringUtils.isEmpty(picname)) {
        fileName = uuid+"_"+picname+"."+extendName;
        }else {
        fileName = uuid+"_"+uploadFileName;
        }
        System.out.println(fileName);
        //5.创建 sun 公司提供的 jersey 包中的 Client 对象
        Client client = Client.create();
        //6.指定上传文件的地址，该地址是 web 路径
        WebResource resource = client.resource(FILESERVERURL+fileName);
        //7.实现上传
        String result = resource.put(String.class,uploadFile.getBytes());
        System.out.println(result);
        return "success";
    }
}
```

## 3.5 编写jsp页面

```jsp
<h2>文件上传-不同服务器之间</h2>
<form action="${pageContext.request.contextPath}/fileUpload2" method="post" enctype="multipart/form-data">
    名称: <input type="text" name="picname"><br/>
    图片: <input type="file" name="uploadFile"><br/>
    <input type="submit" value="上传">
</form>
```

## 3.6 配置解析器

```xml
<!-- 配置文件上传解析器 -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!-- 设置上传文件的最大尺寸为 5MB -->
    <property name="maxUploadSize">
        <value>5242880</value>
    </property>
</bean>
```

