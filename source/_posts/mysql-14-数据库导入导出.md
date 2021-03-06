---
title: MySQL 14.数据库导入导出
date: 2022-07-01T12:14:44.542Z
tags: [mysql]
---
- [1. 导出数据库](#1-导出数据库)
- [2. 导入数据库](#2-导入数据库)

# 1. 导出数据库

(1) 导出全库备份到本地的目录

> ```bash
> mysqldump -u$USER -p$PASSWD -h127.0.0.1 -P3306 –routines –default-character-set=utf8 –lock-all-tables –add-drop-database -A > db.all.sql
> ```

(2) 导出指定库到本地的目录

> ```bash
> mysqldump -uroot -p123456 -h192.168.0.54 -P3306 –routines –default-character-set=utf8 –databases taobao > db.sql
> ```

(3) 导出某个库的表到本地的目录

> ```bash
> mysqldump -u$USER -p$PASSWD -h127.0.0.1 -P3306 –routines –default-character-set=utf8 –tables mysql user> db.table.sql
> ```

(4) 导出指定库的表(仅数据)到本地的目录(例如mysql库的user表,带过滤条件)

> ```bash
> mysqldump -u$USER -p$PASSWD -h127.0.0.1 -P3306 –routines –default-character-set=utf8 –no-create-db –no-create-info –tables mysql user –where=“host=‘localhost’”> db.table.sql
> ```

(5) 导出某个库的所有表结构

> ```bash
> mysqldump -u$USER -p$PASSWD -h127.0.0.1 -P3306 –routines –default-character-set=utf8 –no-data –databases mysql > db.nodata.sql
> ```

(6) 导出某个查询sql的数据为txt格式文件到本地的目录(各数据值之间用”制表符”分隔)，例如sql为’select user,host,password from mysql.user;’

> ```bash
> mysql -u$USER -p$PASSWD -h127.0.0.1 -P3306 –default-character-set=utf8 –skip-column-names -B -e ‘select user,host,password from mysql.user;’ > mysql_user.txt
> ```

(7) 导出某个查询sql的数据为txt格式文件到MySQL服务器，登录MySQL,将默认的制表符换成逗号(适应csv格式文件)。指定的路径，mysql要有写的权限。

> ```bash
> SELECT user,host,password FROM mysql.user INTO OUTFILE ‘/tmp/mysql_user.csv’ FIELDS TERMINATED BY ‘,’;
> ```

# 2. 导入数据库

(1) 恢复全库数据到MySQL

> ```bash
> mysql -uroot -p123456 -h192.168.0.54 -P3306 –default-character-set=utf8 < db.sql
> ```

(2) 恢复某个库的数据(mysql库的user表)

> ```bash
> mysql -u$USER -p$PASSWD -h127.0.0.1 -P3306 –default-character-set=utf8 mysql < db.table.sql
> ```

(3) 恢复MySQL服务器上面的txt格式文件(需要FILE权限,各数据值之间用”制表符”分隔)

```bash
mysql -u$USER -p$PASSWD -h127.0.0.1 -P3306 --default-character-set=utf8
......
mysql> use mysql;
mysql> LOAD DATA INFILE '/tmp/mysql_user.txt' INTO TABLE user ;
```

(4) 恢复MySQL服务器上面的csv格式文件(需要FILE权限,各数据值之间用”逗号”分隔)

```bash
mysql -u$USER -p$PASSWD -h127.0.0.1 -P3306 --default-character-set=utf8
......
mysql> use mysql;
mysql> LOAD DATA INFILE '/tmp/mysql_user.csv' INTO TABLE user FIELDS TERMINATED BY ',';
```

(5) 恢复本地的txt或csv文件到MySQL

```bash
mysql -u$USER -p$PASSWD -h127.0.0.1 -P3306 --default-character-set=utf8
......
mysql> use mysql;
# txt
mysql> LOAD DATA LOCAL INFILE '/tmp/mysql_user.csv' INTO TABLE user;

# csv
mysql> LOAD DATA LOCAL INFILE '/tmp/mysql_user.csv' INTO TABLE user FIELDS TERMINATED BY ',';
```

(5) 文件导入

```bash
use o2o;
set names utf8;
source D:/o2o.sql
```