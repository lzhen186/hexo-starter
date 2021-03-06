---
title: SpringBoot 14.消息队列的整合与使用 14.5.docker安装RocketMQ
date: 2022-07-03T15:06:52.947Z
tags: [springboot]
---
# 一、rocketMQ部署架构

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200506132452.png)

1. NameServer：消息队列的大脑，同时监控消息队列，存储关于队列的基本信息，如各个队列的分布，ip、服务地址、目前的健康状况、队列的处理进度等。多个Nmaeserver之间没有通信。
2. broker：队列服务，负责接受请求并分发请求。数据存储、消息分发的负载均衡。可以是master-slave的主从结构。
3. console-ng：消息队列的监控控制台

# 二、安装nameserver和broker

> 注意：安装rocketMQ，虚拟机磁盘空间要足够。最小10G空余磁盘空间为好。

下载rocketMQ镜像

```
docker pull rocketmqinc/rocketmq
```

新建本机数据存储文件夹

```
rm -fR /home/rocketmq/data/;
mkdir -p /home/rocketmq/data/namesrv/{logs,store};
mkdir -p /home/rocketmq/data/broker/{logs,store,conf};
```

启动nameserver

```
docker run -d -p 9876:9876 \
-v /home/rocketmq/data/namesrv/logs:/root/logs \
-v /home/rocketmq/data/namesrv/store:/root/store \
--name rmqnamesrv \
-e "MAX_POSSIBLE_HEAP=256000000" \
rocketmqinc/rocketmq  sh mqnamesrv
```

将以下配置文件放入本机目录/home/rocketmq/data/broker/conf/broker.conf

```
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
#修改这个ip为你的物理主机ip
brokerIP1=192.168.18.137
```

参考第四小节内容，先把防火墙的9876端口开放，启动broker

```
docker run -d -p 10911:10911 -p 10909:10909 \
-v /home/rocketmq/data/broker/logs:/root/logs \
-v /home/rocketmq/data/broker/store:/root/store \
-v /home/rocketmq/data/broker/conf/broker.conf:/opt/rocketmq-4.4.0/conf/broker.conf \
--name rmqbroker --link rmqnamesrv:namesrv \
-e "NAMESRV_ADDR=namesrv:9876" \
-e "MAX_POSSIBLE_HEAP=256000000" \
rocketmqinc/rocketmq sh mqbroker   -c /opt/rocketmq-4.4.0/conf/broker.conf
```

# 三、RocketMQ控制台

1. 拉取镜像

```
 docker pull styletang/rocketmq-console-ng
```

1. 创建container，并启动

```
docker run -d \
-e "JAVA_OPTS=-Drocketmq.namesrv.addr=172.17.0.2:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" \
-p 8999:8080 -t styletang/rocketmq-console-ng
```

因为服务器的8080端口经常冲突，我们使用8999端口提供控制台容器服务

> 特别注意
>
> 1. 必须先启动`namesrv`，再启动`broker`，否则报错
> 2. 创建控制台的container时，`-Drocketmq.namesrv.addr`必须指定为`namesrv`所在container的IP

可以通过以下命令来查看每个container的IP地址

```
docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)
```

# 四、防火墙开放端口

```
firewall-cmd --zone=public --add-port=9876/tcp --permanent
firewall-cmd --zone=public --add-port=10911/tcp --permanent
firewall-cmd --zone=public --add-port=10909/tcp --permanent
firewall-cmd --zone=public --add-port=8999/tcp --permanent
firewall-cmd --reload

#查看是否开放端口

firewall-cmd --query-port=9876/tcp
firewall-cmd --query-port=10911/tcp
firewall-cmd --query-port=10909/tcp
firewall-cmd --query-port=8999/tcp
```

访问http://192.168.18.137:8999