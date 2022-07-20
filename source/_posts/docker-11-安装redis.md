---
title: Docker 11.安装redis
date: 2022-07-06T05:49:05.749Z
tags:
  - docker
---
## 1.下载镜像

docker pull redis

## 2.在opt下创建文件夹

```
cd /opt/redis_docker/
mkdir conf
mkdir data
```


## 3.新建容器

```
 docker run -d -p 6379:6379 --restart always --name some-redis --privileged=true  -v /opt/redis_docker/data:/data  redis redis-server /etc/redis/redis.conf  --requirepass "111111" --appendonly yes
```