---
title: Docker 11.部署jar
date: 2022-07-06T05:50:05.572Z
---
## 1.新建Dockerfile  （和jar文件一个目录）

```
FROM java:8
VOLUME /tmp   ##Tomcat默认使用/tmp作为工作目录
ADD 文件.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```

## 2.构建镜像

 docker build -t jerome.xin/docker-demo:latest .

## 3.运行容器

```
docker run -d --restart=always --name docker-demo -v /opt/apps/docker-demo/logs:/home/docker-demo/logs -p 8000:8000 jerome.xin/docker-demo
```

## 4.查看日志

docker logs --tail  300 -f  7c1e4