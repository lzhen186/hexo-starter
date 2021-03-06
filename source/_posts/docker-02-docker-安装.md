---
title: Docker 02.Docker 安装
date: 2022-07-06T05:41:28.014Z
tags: [docker]
---
- [1. 前提](#1-前提)
- [2. Docker三要素](#2-docker三要素)
- [3. 安装Docker](#3-安装docker)
  - [1. 删除旧版本](#1-删除旧版本)
  - [2. 三种方法](#2-三种方法)
  - [3. Docker版本](#3-docker版本)
  - [4. 启动 Docker](#4-启动-docker)
- [4. Docker 镜像加速](#4-docker-镜像加速)
- [5. hello world](#5-hello-world)
- [6. Docker 和 VM 比较](#6-docker-和-vm-比较)
# 1. 前提

当前 Docker 基本都装在 Linux 环境下,以 CentOS 为例,建议 CentOS6.5 以上版本,目前主流 CentOS6.8 和 CentOS7.x 都支持

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525141301.png)

# 2. Docker三要素

* 镜像(Image)

  Docker 镜像（Image）就是一个只读的模板。镜像可以用来创建 Docker 容器，一个镜像可以创建很多容器。

  ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525141456.png)

* 容器(Container)

  Docker 利用容器（Container）独立运行一个或一组应用。容器是用镜像创建的运行实例。

  它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。

  可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

  容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一区别在于容器的最上面那一层是可读可写的。

* 仓库（Repository）

  仓库（Repository）是集中存放镜像文件的场所。

  *仓库(Repository)和仓库注册服务器（Registry）是有区别的。仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。*

  仓库分为公开仓库（Public）和私有仓库（Private）两种形式。 最大的公开仓库是 Docker Hub([hub.docker.com/](https://hub.docker.com/))， 存放了数量庞大的镜像供用户下载。国内的公开仓库包括阿里云 、网易云等



>需要正确的理解仓库/镜像/容器这几个概念:
>
>Docker 本身是一个容器运行载体或称之为管理引擎。我们把应用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境就是 image镜像文件。只有通过这个镜像文件才能生成容器。image 文件可以看作是容器的模板。Docker 根据 image 文件生成容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。
>
>- image 文件生成容器实例，称为镜像文件。
>- 一个容器运行一种服务，当我们需要的时候，就可以通过docker客户端创建一个对应的运行实例，也就是我们的容器
>- 至于仓库，就是放了一堆镜像的地方，我们可以把镜像发布到仓储中，需要的时候从仓储中拉下来就可以了。

Docker 简易流程图

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525141906.png)

# 3. 安装Docker

可以参考[官方文档](https://docs.docker.com/engine/install/centos/)

## 1. 删除旧版本

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

## 2. 三种方法

可以参考[官方文档](https://docs.docker.com/engine/install/centos/)中的三种方法任选一种进行下载。

我用的是第三种方式：脚本下载

```shell
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

## 3. Docker版本

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525144151.png)

## 4. 启动 Docker

```shell
systemctl start docker
```



![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525144532.png)

# 4. Docker 镜像加速

以阿里云为例,首先注册一个阿里云账号,然后进入镜像加速页面

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525145414.png)

配置本机的 Docker 文件

如果没有`/etc/docker/daemon.json`就自己创建

```shell
sudo vim `/etc/docker/daemon.json`
```

重启 Docker 服务

```shell
systemctl daemon-reload
systemctl restart docker
```

# 5. hello world

直接通过 `docker run hello-world` 命令,我们可以直接从阿里云拉取 hello-world 镜像并创建容器自动运行(在本地没有找到 hello-world 的镜像时)

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525145905.png)

docker run 命令运行流程图

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525145919.png)

# 6. Docker 和 VM 比较

* Docker 工作原理

  Docker是一个Client-Server结构的系统，Docker守护进程运行在主机上，然后通过Socket连接从客户端访问，守护进程从客户端接受命令并管理运行在主机上的容器。容器，是一个运行时环境，就是我们前面说到的集装箱。

  例如下面 Docker 图标(一只鲸鱼背上拖着很多个集装箱, 鲸鱼类似于 Docker,一个个的集装箱就是软件开发环境中的各种软件)

  ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525150049.png)

  以下为 Docker 运行架构图

  ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525150122.png)

* 为什么 Docker 运行速度远大于 VM?

  1. Docker有着比虚拟机更少的抽象层。由于docker不需要Hypervisor实现硬件资源虚拟化,运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。

  2. Docker利用的是宿主机的内核,而不需要CentOS。因此,当新建一个容器时,docker不需要和虚拟机一样重新加载一个操作系统内核。从而避免加载操作系统内核这个比较费时费资源的过程,当新建一个虚拟机时,虚拟机软件需要加载CentOS,整个新建过程是分钟级别的。而docker由于直接利用宿主机的操作系统,因此新建一个docker容器只需要几秒钟。

* Docker 和 VM 对比图

  ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525150538.png)

* Docker 和 VM 特点对比图

  ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200525150633.png)