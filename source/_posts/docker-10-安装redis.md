---
title: Docker 11.安装redis
date: 2022-07-06T05:49:05.749Z
---
## 1.下载镜像

docker pull redis

## 2.在opt下创建文件夹

```
cd /opt/redis_docker/
mkdir conf
mkdir data
```

在conf下新建redis.conf配置文件

```
requirepass 123456  #默认空, 连接时需要输入的密码
appendonly yes  # redis持久化（可选）
databases 16    # 数据库个数（可选），可以改改看，看看能不能生效
port 6379   # redis监听的端口号
daemonize no    # 默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程，改为yes会使配置文件方式启动redis失败
protected-mode no   # 默认yes，开启保护模式会限制为本地访问
bind 127.0.0.1        # 注释掉这部分，这是限制redis只能本地访问
```

## 3.新建容器

```
 docker run -d -p 6379:6379 --restart always --name some-redis --privileged=true  -v /opt/redis_docker/conf/redis.conf:/etc/redis/redis.conf  -v /opt/redis_docker/data:/data  redis redis-server /etc/redis/redis.conf  --requirepass "111111" --appendonly yes
```