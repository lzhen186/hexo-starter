---
title: Docker 06.DockerFile解析
date: 2022-07-06T05:44:41.828Z
tags: [docker]
---
- [1. 什么是 DockerFile](#1-什么是-dockerfile)
- [2. DockerFile 构建过程解析](#2-dockerfile-构建过程解析)
  - [DockerFile 基础知识](#dockerfile-基础知识)
  - [Docker执行Dockerfile的大致流程](#docker执行dockerfile的大致流程)
- [3. DockerFile 体系结构(保留字)](#3-dockerfile-体系结构保留字)
- [4. DockerFile 实战案例](#4-dockerfile-实战案例)
  - [一. 自定义 CentOS 镜像案例](#一-自定义-centos-镜像案例)
  - [二. CMD/ENTRYPOINT 镜像案例](#二-cmdentrypoint-镜像案例)
  - [三. 自定义 Tomcat9 镜像(重要)](#三-自定义-tomcat9-镜像重要)
  - [四. 总结](#四-总结)

# 1. 什么是 DockerFile

Dockerfile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本。

通常使用 DockerFile 的三个步骤都是:

1. 编写 DockerFile 文件
2. 执行 docker build 编译命令
3. 执行docker run 启动容器命令

以 CentOS 为例, Docker Hub 上的 CentOS 的 DockerFile 文件如下

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526132103.png)

# 2. DockerFile 构建过程解析

## DockerFile 基础知识

以上面的 CentOS DockerFile 文件为例

1. 每条保留字指令(红色字体)都必须为大写字母且后面要跟随至少一个参数
2. 指令按照从上到下，顺序执行
3. \#表示注释
4. 每条指令都会创建一个新的镜像层，并对镜像进行提交

## Docker执行Dockerfile的大致流程

1. docker从基础镜像运行一个容器
2. 执行一条指令并对容器作出修改
3. 执行类似docker commit的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行dockerfile中的下一条指令直到所有指令都执行完成

总结:

从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段，

- Dockerfile是软件的原材料
- Docker镜像是软件的交付品
- Docker容器则可以认为是软件的运行态。 Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526132228.png)

Dockerfile：需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等;

Docker镜像：在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时，会真正开始提供服务;

Docker容器：直接提供服务.

# 3. DockerFile 体系结构(保留字)

- FROM : 基础镜像，当前新镜像是基于哪个镜像的

- MAINTAINER : 镜像维护者的姓名和邮箱地址

- RUN : 容器构建时需要运行的命令

- EXPOSE : 当前容器对外暴露出的端口

- WORKDIR : 指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点

- ENV : 用来在构建镜像过程中设置环境变量

  ENV MY_PATH /usr/mytest 这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样； 也可以在其它指令中直接使用这些环境变量，

  比如：WORKDIR $MY_PATH

- ADD : 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包

- COPY : 类似ADD，拷贝文件和目录到镜像中。 将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置

- VOLUME : 容器数据卷，用于数据保存和持久化工作

- CMD : 指定一个容器启动时要运行的命令

  ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526132533.png)

  

  注意: Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换

- ENTRYPOINT : 指定一个容器启动时要运行的命令;ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数,但是不会被 docker run 后面的参数替换,而是追加

- ONBUILD : 当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发

总结:

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526132641.png)

# 4. DockerFile 实战案例

## 一. 自定义 CentOS 镜像案例

首先,我们需要知道Docker Hub 中 99% 的镜像都是通过在 base 镜像中安装和配置需要的软件构建出来的

而这个 base 镜像就是scratch 镜像。

首先停止所有正在运行的 Docker 容器

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526135142.png)

然后看看从阿里云下载的基础版 CentOS 缺失了哪些功能

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526140938.png)

现在我们的目标就是通过编写 DockerFile 为基础版本的 CentOS 镜像加上这些缺失的功能

DockerFile如下：

```shell
FROM centos
MAINTAINER krislin_zhao
 
ENV MYPATH /usr/local
WORKDIR $MYPATH
 
RUN yum -y install vim
RUN yum -y install net-tools
 
EXPOSE 80
 
CMD echo $MYPATH
CMD echo "success--------------ok"
CMD /bin/bash
```

开始构建

```shell
docker build -f centosDockerFile -t krislin/centos:1.1 .
```

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526141551.png)

构建成功,开始运行容器

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526142157.png)

除此之外,还可以查看 Docker 镜像的修改历史

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526142337.png)

## 二. CMD/ENTRYPOINT 镜像案例

CMD 示例:

Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换

以 Tomcat 为例

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526142809.png)

原因就在于我们的 ls -l 参数替换掉了原来的启动参数,如下

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526142848.png)

相当于在这行参数后面又添加了 `CMD ls -l`

那么容器启动时就会执行最后的 ls -l 命令

如果是 ENTRYPOINT

docker run 之后的参数会被当做参数传递给 ENTRYPOINT，之后形成新的命令组合

示例如下

编写 DockerFile

```shell
FROM centos
RUN yum install -y curl
ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]
```

然后构建

```shell
docker build -f centosDockerFile2 -t krislin/ipcentos .
```

以添加参数的形式启动容器

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526143614.png)

**ONBUILD 示例**

类似于触发器,在镜像编译以及子镜像编译的时候触发

新建一个 DockerFile , centosDockerFileFather

```dockerfile
FROM centos
RUN yum install -y curl
ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]
ONBUILD RUN echo "father is building -------------->"
```

进行编译后,又新建一个子 DockerFile, centosDockerFileSon

```dockerfile
FROM cris/ipcentos_father
RUN yum install -y curl
ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]
```

cris/ipcentos_father 就是上面编译完成的父镜像

当我们开始编译子镜像时,就会触发 ONBUILD 操作

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526143938.png)

## 三. 自定义 Tomcat9 镜像(重要)

在新建目录,并且添加以下文件

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526153354.png)

Dockerfile 内容如下

```dockerfile
FROM         centos
MAINTAINER    krislin_zhao
#把宿主机当前上下文的c.txt拷贝到容器/usr/local/路径下
COPY copy.txt /usr/local/cincontainer.txt
#把java与tomcat添加到容器中
ADD jdk-8u171-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.35.tar.gz /usr/local/
#安装vim编辑器
RUN yum -y install vim
#设置工作访问时候的WORKDIR路径，登录落脚点
ENV MYPATH /usr/local
WORKDIR $MYPATH
#配置java与tomcat环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_171
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.35
ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.35
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
#容器运行时监听的端口
EXPOSE  8080
#启动时运行tomcat
# ENTRYPOINT ["/usr/local/apache-tomcat-9.0.35/bin/startup.sh" ]
# CMD ["/usr/local/apache-tomcat-9.0.35/bin/catalina.sh","run"]
CMD /usr/local/apache-tomcat-9.0.35/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.35/bin/logs/catalina.out
```

然后开始编译

```shell
docker build -t krislin_tomcat9 .
```

如果不加 -f 参数,默认从当前目录下的 Dockerfile 文件开始编译

编译成功后,直接运行

```shell
docker run -d -p 9090:8080 --name krisslintomcat9 
-v /root/mytomcat/test:/usr/local/apache-tomcat-9.0.35/webapps/test 
-v /root/mytomcat/logs:/usr/local/apache-tomcat-9.0.35/logs  
--privileged=true 
krislin_tomcat9
```

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526154158.png)

查看数据卷对应的目录

```shell
docker inspect krislintomcat9
```

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526154333.png)

验证 Tomcat 是否启动

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526154532.png)

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526155722.png)

**测试web 工程发布**

我们在宿主机的目录上新建一个简单的 web 工程

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526160229.png)

test.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
  </head>
  <body>
    -----------welcome------------
    <%="i am in docker tomcat self "%>
    <br>
    <br>
    <% System.out.println("=============docker tomcat self");%>
  </body>
</html>
```

然后是 WEB-INF 目录下的 web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns="http://java.sun.com/xml/ns/javaee"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
  id="WebApp_ID" version="2.5">
  
  <display-name>test</display-name>
 
</web-app>
```

最后重启容器

```shell
docker restart krislintomcat9
```

测试

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526160538.png)

我们在宿主机修改 jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
  </head>
  <body>
    -----------welcome------------
    <br>
    <%="i am in docker tomcat self krislin "%>
    <br>
    <br>
    <% System.out.println("=============docker tomcat self");%>
  </body>
</html>
```

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526160800.png)

## 四. 总结

![img](https://user-gold-cdn.xitu.io/2019/4/19/16a34edc0a8c65e1?imageslim)