---
title: SpringBoot 12.整合分布式文件系统fastdfs 12.1.fastdfs简介及架构说明
date: 2022-07-03T14:08:22.283Z
tags: [springboot]
---
# 一、简介

- FastDFS是一个轻量级的开源分布式文件系统。
- FastDFS主要解决了大容量的文件存储和高并发访问的问题，文件存取时实现了负载均衡。
- FastDFS实现了软件方式的RAID，可以使用廉价的IDE硬盘进行存储
- 支持存储服务器在线扩容
- 支持相同内容的文件只保存一份，节约磁盘空间
- FastDFS特别适合大中型网站使用，用来存储资源文件（如：图片、文档、音频、视频等等）

# 二、架构说明

- Tracker：管理集群，tracker 也可以实现集群。每个 tracker 节点地位平等。收集 Storage 集群的状态。
- Storage：实际保存文件 Storage 分为多个组，每个组之间保存的文件是不同的。每个组内部可以有多个成员，

组成员内部保存的内容是一样的，组成员的地位是一致的，没有主从的概念。
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200501125955.png)

说明 nginx + fileid（文件路径），http访问
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200501130209.png)

# 三、好处：

1. 将文件的管理与具体业务应用解耦，可以多个应用共用一套fastDFS集群，分成不同的组
2. 图片访问，只需要将http-url交给浏览器。nginx提供访问服务。
3. 方便统一备份，一组的多个storage就是彼此的备份
4. 可以将图片浏览，文件下载的压力分散给nginx服务。应用自己专心做业务。
5. 缩略图，防盗链等等