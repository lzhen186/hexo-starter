---
title: SpringBoot 12.整合分布式文件系统fastdfs 12.2.使用docker安装fastdfs
date: 2022-07-03T14:11:01.809Z
tags: [springboot]
---
# 一、安装

- 拉取镜像

```
# docker pull delron/fastdfs
Using default tag: latest
latest: Pulling from delron/fastdfs
43db9dbdcb30: Pull complete 
85a9cd1fcca2: Pull complete 
c23af8496102: Pull complete 
e88c36ca55d8: Pull complete 
492fed9ec7f3: Pull complete 
0c1d41dbb2bd: Pull complete 
99b513124929: Pull complete 
bf3f5901a13d: Pull complete 
88bf4f57c2c5: Pull complete 
Digest: sha256:f3fb622783acee7918b53f8a76c655017917631c52780bebb556d29290955b13
Status: Downloaded newer image for delron/fastdfs
```

- 创建本机存储目录

```
rm -fR /home/docker/fastdfs/{tracker,storage} 
mkdir /home/docker/fastdfs/{tracker,storage}  -p
```

- 启动tracker

```
docker run -d \
--network=host \
--name tracker \
-v /home/docker/fastdfs/tracker:/var/fdfs \
delron/fastdfs tracker
```

- 启动storage

```
docker run -d \
--network=host \
--name storage \
-e TRACKER_SERVER=192.168.161.3:22122 \
-v /home/docker/fastdfs/storage:/var/fdfs \
-e GROUP_NAME=group1  \
delron/fastdfs storage
```

# 二、开启宿主机防火墙端口,morunchang/fastdfs镜像在构建的时候为nginx配置的端口为8888

```
firewall-cmd --zone=public --add-port=22122/tcp --permanent
firewall-cmd --zone=public --add-port=23000/tcp --permanent
firewall-cmd --zone=public --add-port=8888/tcp --permanent
firewall-cmd --reload
# 查看是否开放
firewall-cmd --query-port=22122/tcp
firewall-cmd --query-port=23000/tcp
firewall-cmd --query-port=8888/tcp
```

# 三、测试一下安装结果

FastDFS安装包中，自带了客户端程序，可以使用这个命令行客户端进行文件上传及下载测试。
在宿主机执行命令
3.1上传文件（是容器里面的文件）

```
# docker exec -i storage /usr/bin/fdfs_upload_file /etc/fdfs/client.conf ./README
group1/M00/00/00/wKgBW10lZHCAC8TaAAAAMT6WPfM3645854
```

3.2 查看fastdfs文件系统信息

```
# docker exec -i storage fdfs_file_info /etc/fdfs/client.conf group1/M00/00/00/wKgBW10lZHCAC8TaAAAAMT6WPfM3645854
source storage id: 0
source ip address: 192.168.1.91
file create timestamp: 2019-07-10 04:07:12
file size: 49
file crc32: 1050033651 (0x3E963DF3)
```

3.3 下载文件，不会下载到宿主机，去容器里面看

```
# docker exec -i storage fdfs_download_file /etc/fdfs/client.conf group1/M00/00/00/wKgBW10lZHCAC8TaAAAAMT6WPfM3645854
```

3.4 查看集群状态

```
# docker exec -i storage  fdfs_monitor /etc/fdfs/storage.conf
```