---
title: SpringBoot 14.消息队列的整合与使用 14.2.使用docker安装activeMQ
date: 2022-07-03T14:42:30.326Z
tags: [springboot]
---
# 一、docker安装activeMQ

查询activeMQ可用镜像版本

```
docker search activemq
```

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200506095940.png)

拉取镜像

```
docker pull  webcenter/activemq
```

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200506100006.png)

```
docker run -d --name myactivemq -p 61616:61616 -p 8161:8161 webcenter/activemq:latest
```

# 二、开放防火墙

```
firewall-cmd --zone=public --add-port=61616/tcp --permanent
firewall-cmd --zone=public --add-port=8161/tcp --permanent
firewall-cmd --reload
# 查看是否开放
firewall-cmd --query-port=61616/tcp
firewall-cmd --query-port=8161/tcp
```

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200506100037.png)

# 三、访问admin管理界面

http://192.168.18.137:8161/admin/topics.jsp
默认账号密码都是admin

点击manage activemq broker就可以进入管理页面（需要输入账号密码）。
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200506100108.png)
表示安装成功了！