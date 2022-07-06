---
title: Docker 09.安装mysql
date: 2022-07-06T05:47:37.409Z
---
## 1.下载镜像

docker pull mysql:5.7

## 2.在opt下创建查看log数据

```
cd /opt/mysql_docker/
echo $PWD
```

## 3.新建容器

```dockerfile
docker run --name mysqlserver --privileged=true  -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=111111 -d -i -p 3306:3306 mysql:5.7
```

查看容器位置 ： cd /var/lib/docker/containers/

## 4.进入mysql容器，并登陆mysql

```
docker start mysqlserver
docker exec -it mysqlserver bash
mysql -uroot -p111111
```

## 5.开启远程访问权限

```
use mysql;
select host,user from user;
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '111111';
flush privileges;
```

## 6.查看docker日志

docker logs -f --tail 10  *containerID*