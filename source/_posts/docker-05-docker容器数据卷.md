---
title: Docker 05.Docker容器数据卷
date: 2022-07-06T05:43:57.193Z
tags: [docker]
---
- [1. 什么是 Docker 容器数据卷](#1-什么是-docker-容器数据卷)
- [2. 数据卷的用处](#2-数据卷的用处)
- [3. 数据卷使用](#3-数据卷使用)
  - [一. 直接通过命令添加数据卷](#一-直接通过命令添加数据卷)
  - [二. DockerFile添加数据卷](#二-dockerfile添加数据卷)
  - [三. 数据卷容器](#三-数据卷容器)


# 1. 什么是 Docker 容器数据卷

需求:

- Docker 可以将运行的环境打包形成容器运行，但是我们对 Docker 容器的数据的要求希望是持久化的
- 容器之间希望共享数据

Docker容器产生的数据，如果不通过docker commit生成新的镜像，使得数据做为镜像的一部分保存下来， 那么当容器删除后，数据自然也就没有了。

为了能保存数据在docker中我们使用数据卷。

类似 Redis里面的rdb和aof文件或者我们平时使用的移动硬盘

# 2. 数据卷的用处

数据卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性：

数据卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷

特点：

1. 数据卷可在容器之间共享或重用数据
2. 卷中的更改可以直接生效
3. 数据卷中的更改不会包含在镜像的更新中
4. 数据卷的生命周期一直持续到没有容器使用它为止

# 3. 数据卷使用

## 一. 直接通过命令添加数据卷

```
docker run -it -v /宿主机绝对路径目录:/容器内目录 镜像名
```

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526102438.png)

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526102631.png)

验证数据卷是否挂载成功

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526102934.png)

宿主机和容器之间的数据交互

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200713083636.png)

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526103254.png)

同理, 容器可以对数据进行修改并同步到宿主机

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526103411.png)

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526103517.png)

容器停止退出后，主机修改后数据是否同步

通过 exit 命令停止容器并退出终端

然后在宿主机对数据进行修改

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526103801.png)

重新开启容器

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526103919.png)

可以发现,即便是容器关闭,宿主机依然可以对数据卷进行数据操作,当容器重新开启时,数据卷会自动进行同步

如果想要设置权限,例如容器只能对数据卷进行读和同步,宿主机可以操作数据卷,那么只需要添加一个参数即可

```
docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名
```

`ro` 就表示 read-only 权限(针对容器)

## 二. DockerFile添加数据卷

DockerFile 简单来说,就是描述 Docker 镜像的描述文件

流程简单梳理如下

1. 宿主机新建一个 DockerFile

   可在Dockerfile中使用VOLUME指令来给镜像添加一个或多个数据卷

   ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526104550.png)

   ps: 出于可移植和分享的考虑，用-v 主机目录:容器目录这种方法不能够直接在Dockerfile中实现。 **由于宿主机目录是依赖于特定宿主机的，并不能够保证在所有的宿主机上都存在这样的特定目录**。

2. build 构建镜像

   ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526104923.png)

3. 根据镜像运行容器

   ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526105208.png)

4. 测试数据卷

   ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526105424.png)

   根据 inspect 命令查看对应的宿主机数据卷目录

   ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526105821.png)

   ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526105932.png)

   默认宿主机挂载地址需要通过 inspect 命令查看

## 三. 数据卷容器

主要用于容器和容器之间的数据共享

命令: --volumes-from

示例如下:

1. 启动一个父容器

   ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526110256.png)

   启动后我们在指定目录下创建一个文件

2. 新建两个子容器,继承自父容器

   ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526110556.png)

   ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526110648.png)

   可以发现成功同步了父容器的数据

   同时修改任意一个子容器的数据卷数据,都会同步到其他容器

   即便是删除任意一个容器,数据卷的数据同步不会停止

   

3. 结论

   容器之间共享数据的传递，数据卷的生命周期一直持续到没有容器使用它为止