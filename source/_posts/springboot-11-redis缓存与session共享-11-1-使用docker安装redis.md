---
title: SpringBoot 11.redis缓存与session共享 11.1.使用docker安装redis
date: 2022-07-03T08:46:06.418Z
tags: [springboot]
---
# 一、获取 redis 镜像

```
docker search redis
docker pull redis:5.0.5
docker images
```

# 二、创建容器

创建宿主机 redis 容器的数据和配置文件目录

```
# 这里我们在 /home/docker 下创建
mkdir /home/docker/redis/{conf,data} -p
cd /home/docker/redis
```

**注意：后面所有的操作命令都要在这个目录`/home/docker/redis`下进行**

获取 redis 的默认配置模版

```
# 获取 redis 的默认配置模版
# 这里主要是想设置下 redis 的 log / password / appendonly
# redis 的 docker 运行参数提供了 --appendonly yes 但没 password
wget https://raw.githubusercontent.com/antirez/redis/4.0/redis.conf -O conf/redis.conf

# 直接替换编辑
sed -i 's/logfile ""/logfile "access.log"/' conf/redis.conf;
sed -i 's/# requirepass foobared/requirepass 123456/' conf/redis.conf;
sed -i 's/appendonly no/appendonly yes/' conf/redis.conf;
sed -i 's/bind 127.0.0.1/bind 0.0.0.0/' conf/redis.conf;
```

> protected-mode 是在没有显式定义 bind 地址（即监听全网段），又没有设置密码 requirepass时，protected-mode 只允许本地回环 127.0.0.1 访问。改为bind 0.0.0.0

创建并运行一个名为 myredis 的容器,放到start-redis.sh脚本里面

```
# 创建并运行一个名为 myredis 的容器
docker run \
-p 6379:6379 \
-v $PWD/data:/data \
-v $PWD/conf/redis.conf:/etc/redis/redis.conf \
--privileged=true \
--name myredis \
-d redis:5.0.5 redis-server /etc/redis/redis.conf

# 命令分解
docker run \
-p 6379:6379 \ # 端口映射 宿主机:容器
-v $PWD/data:/data:rw \ # 映射数据目录 rw 为读写
-v $PWD/conf/redis.conf:/etc/redis/redis.conf:ro \ # 挂载配置文件 ro 为readonly
--privileged=true \ # 给与一些权限
--name myredis \ # 给容器起个名字
-d redis redis-server /etc/redis/redis.conf # deamon 运行容器 并使用配置文件启动容器内的 redis-server 
```

查看活跃的容器

```
# 查看活跃的容器
docker ps
# 如果没有 myredis 说明启动失败 查看错误日志
docker logs myredis
# 查看 myredis 的 ip 挂载 端口映射等信息
docker inspect myredis
# 查看 myredis 的端口映射
docker port myredis
```

## 三、访问 redis 容器服务

```
docker exec -it myredis bash
redis-cli
```

## 四、开启防火墙端口，提供外部访问

```
firewall-cmd --zone=public --add-port=6379/tcp --permanent
firewall-cmd --reload
firewall-cmd --query-port=6379/tcp
```