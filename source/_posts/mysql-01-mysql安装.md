---
title: MySQL 01.mysql安装
date: 2022-07-01T07:50:12.829Z
tags: [mysql]
---
- [1. 在主机上安装mysql](#1-在主机上安装mysql)
- [2. 在docker安装mysql](#2-在docker安装mysql)
  - [(1) 默认配置安装mysql](#1-默认配置安装mysql)
  - [**(2) 自定义配置安装mysql**](#2-自定义配置安装mysql)

# 1. 在主机上安装mysql

下面是安装在Centos7系统上

```bash
# 安装前创建用户和数据文件存储文件夹
mkdir -p /data/mysql
groupadd mysql
useradd -g mysql mysql
chown mysql.mysql -R /data/mysql

# 安装依赖包
yum -y install ncurses-devel

# 下载mysql源码
wget http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-boost-5.7.15.tar.gz

# 解压
tar -zxvf mysql-boost-5.7.15.tar.gz

# 切换目录
cd mysql-5.7.15

# 检查编译环境，
    # 注：编译MySQL5.7以及更高的版本时，都需要下载并引用或者直接安装boost库，否则在执行cmake命令时会报如下错误，在下载mysql源码时最好下载带有boost库的版本。解决办法：在cmake命令后面添加参数-DDOWNLOAD_BOOST=1 -DWITH_BOOST=Boost库路径
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/data/mysql \
-DSYSCONFDIR=/usr/local/mysql \
-DMYSQL_USER=mysql \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DMYSQL_UNIX_ADDR=/var/run/mysql/mysql.sock \
-DMYSQL_TCP_PORT=3306 \
-DENABLED_LOCAL_INFILE=1 \
-DENABLE_DOWNLOADS=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_DEBUG=0 \
-DMYSQL_MAINTAINER_MODE=0 \
-DWITH_SSL:STRING=bundled \
-DWITH_ZLIB:STRING=bundled \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=/home/vison/mysql-5.7.15/boost

# 编译安装(注：编译比较耗时间和cpu，内存建议2G以上)
make && make install

# 添加mysql的环境变量
export PATH=$PATH:/usr/local/mysql/bin && source /etc/profile

# 初始化MySQL自身的数据库(指定安装目录和数据存放目录)
/usr/local/mysql/bin/mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql

# 设置mysqld的开机启动
cp /usr/local/mysql/support-files/mysql.server  /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
chkconfig --add mysqld
chkconfig mysqld on
```

修改配置文件

```bash
# 打开配置文件
vim /usr/local/mysql/my.cnf

# 添加日志信息内容如下：
    # 设置普通日志
    general_log=ON
    general_log_file=/data/mysql/log/mysql.log

    # 设置错误日志
    log_error=/data/mysql/log/error.log

    # 记录没用到索引的语句
    log-queries-not-using-indexes=ON

    # 设置慢查询日志
    slow_query_log=ON
    slow-query-log-file=/data/mysql/log/mysql-slow.log
    # 慢查询时间
    long_query_time=2
```

mysql启动和登录

```bash
# 启动
systemctl start mysql

# 查看是否启动成功
ps -ef | grep mysqld

# 登录mysql
mysql -u root -p
```

# 2. 在docker安装mysql

官方docker安装说明：https://hub.docker.com/_/mysql

## (1) 默认配置安装mysql

```yaml
version: '3'

services:
  mysql:
    image: mysql:8.0.18
    container_name: mysql-8.0
    restart: always
    # 映射宿主机端口33060
    ports:
      - 33060:3306
      - 33062:33062
    environment:
      - MYSQL_ROOT_PASSWORD=123456
    # mysql数据映射宿主机路径/data/docker-mysql
    volumes:
      - /data/docker-mysql:/var/lib/mysql
```

## **(2) 自定义配置安装mysql**

获取mysql配置模板：

```bash
# 先使用默认配置启动mysql容器，进入容器：
docker exec -it mysql-8.0 bash
cd /etc
tar zcvf mysql.tar.gz mysql
exit

# 从容器导出文件到宿主机
docker exec mysql-8.0 sh -c 'exec cat /etc/mysql.tar.gz' > ./mysql.tar.gz
```

获取到配置模板后添加自定义配置参数，然后启动时候映射过去即可。

```yaml
version: '3'

services:
  mysql:
    image: mysql:8.0.18
    container_name: mysql-8.0
    restart: always
    # 映射宿主机端口33060
    ports:
      - 33060:3306
      - 33062:33062
    environment:
      - MYSQL_ROOT_PASSWORD=123456
    # mysql数据映射宿主机路径/data/docker-mysql
    volumes:
      - /data/docker-mysql:/var/lib/mysql
      # 配置文件映射
      - ./mysql:/etc/mysql
```

本地连接：

> mysql -u root -h 127.0.0.1 -P 33060 -p

远程连接：

> mysql -u root -h 192.168.8.202 -P 33060 -p



注：如果安装了mysql8版本，旧版本的客户端是无法连接的，因为mysql8之前的版本加密规则是mysql_native_password，mysql8之后加密规则是caching_sha2_password，解决方法把mysql用户登录密码加密规则还原成mysql_native_password，解决方法如下：

```bash
# 进去mysql容器，然后登陆mysql
mysql -p

# 修改加密规则
use mysql;
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
FLUSH PRIVILEGES;
```