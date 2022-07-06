---
title: Docker 07.centos7 在docker中安装mongodb
date: 2022-07-06T05:45:23.796Z
---
# [centos7 在docker中安装mongodb](https://www.cnblogs.com/nickchou/p/13676111.html)

一、**搜索docker的mongo镜像**

```
docker search mongo

```

二、**拉取mongo最新镜像**

```
docker pull mongo

```

三、**创建好mongo存储路径，便于后面做挂载**

```
mkdir -p /data/mongo

```

四、**运行镜像**

```
docker run --restart=always --name mongo -v /data/mongo:/data/db -p 27017:27017 -d mongo --auth

```

返回dockerid说明执行成功
![img](https://img2020.cnblogs.com/blog/308699/202009/308699-20200915224225786-1996957973.png)

指令说明
`--restart=always` 表示重启自动运行
`--name` 设置容器名称
`-v` 挂载目录 宿主机目录/容器目录
`-p` 端口映射 宿主机/容器
`-d` 表示后台运行
`--auth` 表示链接需要认证，推荐加上，也可以不加

五、**查看运行的容器**

```
docker ps -a

```

![img](https://img2020.cnblogs.com/blog/308699/202009/308699-20200915225159952-81532718.png)

六、**创建mongo的用户及密码**
首先进入容器

```
docker exec -it mongo bash

```

进入mongo

```
mongo

```

使用admin

```
use admin

```

创建一个账户密码.（注意：没有创建过用户才可以不需要auth直接创建，否则先登录`db.auth('zhangsan','123456')`）才能创建，也就是只要创建过一次用户了都需要先auth才能操作，或者也可以把/data/mongo目录全部清空创建新容器(会丢失数据)

```
db.createUser({user:"zhangsan",pwd:"123456",roles:[{role:'root',db:'admin'}]})

```

如果需要退出mongo，执行指令`exit`，图如下
![img](https://img2020.cnblogs.com/blog/308699/202009/308699-20200915225950688-1417424406.png)

MongoDB基本的角色
1.数据库用户角色：read、readWrite;
2.数据库管理角色：dbAdmin、dbOwner、userAdmin；
3.集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
4.备份恢复角色：backup、restore；
5.所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
6.超级用户角色：root 这个角色的权限最大

七、**配置mongo的远程访问，在mongo容器里修改配置文件**
继续第六步骤不要退出容器，或者再重新进入容器`docker exec -it mongo bash`
先安装好vim

```
apt-get update
apt-get install vim -y

```

修改mongo配置文件，运行远程访问

```
vim /etc/mongod.conf.orig

```

将其中的 bindIp: 127.0.0.1 注释或者改为0.0.0.0，保存:wq并退出
![img](https://img2020.cnblogs.com/blog/308699/202009/308699-20200915230740601-386899478.png)
然后`exit` 退出mongo容器回到宿主机，重启docker让配置生效

```
docker restart mongo

```

八、**测试远程连接mongodb，下图用的navicat**
1、如果不使用账号密码登陆的话会报错
![img](https://img2020.cnblogs.com/blog/308699/202009/308699-20200915231630492-882700766.png)
2、使用正确的账号密码
![img](https://img2020.cnblogs.com/blog/308699/202009/308699-20200915231439548-1326967576.png)

九、**测试常用的一些mongo指令**
1、进入容器、mongo、授权认证

```
docker exec -it mongo bash
mongo
use admin
db.auth('zhangsan','123456')

```

返回1说明登录成功了
![img](https://img2020.cnblogs.com/blog/308699/202009/308699-20200915232457424-969120824.png)
2、创建数据库，并插入和查询测试数据

```
use test
db.test.save({name: 'test', age: '18'})
db.test.find();

```

![img](https://img2020.cnblogs.com/blog/308699/202009/308699-20200915232830630-1066069423.png)
navicat中查看
![img](https://img2020.cnblogs.com/blog/308699/202009/308699-20200915233028580-536805247.png)

其他指令：
`show dbs` 查看数据库
`show collections` 查看集合