---
title: Docker 13.测试本地和阿里云仓库的镜像发布和拉取
date: 2022-07-06T05:50:48.618Z
tags: [docker]
---
- [1. 登陆阿里云的容器镜像服务](#1-登陆阿里云的容器镜像服务)
  - [创建镜像仓库](#创建镜像仓库)
- [2. 在本地生成容器镜像](#2-在本地生成容器镜像)
- [3. 然后根据指示推送本地 image 到阿里云的 repository](#3-然后根据指示推送本地-image-到阿里云的-repository)

# 1. 登陆阿里云的容器镜像服务

##  创建镜像仓库

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526161730.png)

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526161823.png)

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526162018.png)

# 2. 在本地生成容器镜像

首先,我们知道了 image 的生成方式有两种,一种是根据 DockerFile 构建;一种是根据容器 commit 新的 image

示例

首先运行一个 Docker 容器

```shell
docker run -it krislin/centos:1.1
```

然后提交一个新的 image

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526162247.png)

# 3. 然后根据指示推送本地 image 到阿里云的 repository

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526161906.png)

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526163110.png)

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200526163137.png)