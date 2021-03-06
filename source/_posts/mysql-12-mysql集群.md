---
title: MySQL 12.mysql集群
date: 2022-07-01T11:54:17.258Z
tags: [mysql]
---
- [1. 主从复制](#1-主从复制)
- [2. 主主复制](#2-主主复制)
- [3. 被动模式下主主复制](#3-被动模式下主主复制)
- [4. mysql集群负载均衡](#4-mysql集群负载均衡)
  - [**(1) mysql-proxy说明**](#1-mysql-proxy说明)
  - [**(2) 启动mysql-proxy**](#2-启动mysql-proxy)
  - [**(3) 设置读写分离mysql启动方式**](#3-设置读写分离mysql启动方式)

# 1. 主从复制

MySQL数据库自身提供的主从复制功能，可以方便的实现数据的多处自动备份，实现数据库的拓展。多个数据备份不仅可以加强数据的安全性，通过实现读写分离还能进一步提升数据库的负载性能。

在一主多从的数据库体系中，多个从服务器采用异步的方式更新主数据库的变化，业务服务器在执行写或者相关修改数据库的操作是在主服务器上进行的，读操作则是在各从服务器上进行。

MySQL之间数据复制的基础是二进制日志文件（binary log file）。一台MySQL数据库一旦启用二进制日志后，其作为master，它的数据库中所有操作都会以“事件”的方式记录在二进制日志中，其他数据库作为slave通过一个I/O线程与主服务器保持通信，并监控master的二进制日志文件的变化，如果发现master二进制日志文件发生变化，则会把变化复制到自己的中继日志中，然后slave的一个SQL线程会把相关的“事件”执行到自己的数据库中，以此实现从数据库和主数据库的一致性，也就实现了主从复制。

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200531091948.png)



配置主从复制流程：

- 主mysql配置binlog
- 从mysql器配置relaylog
- 主mysql授权给从mysql器一个帐号
- 从mysql连接主mysql

```bash
# (1) 修改主mysql配置文件
  # 打开配置文件
  sudo vim /etc/mysql/my.cnf
  #在[mysqld]字段下
    # ① 给主mysql起一个名字，在局域网内一般为ip最后一个数字
    server-id = 101
    # ② 开启logbin二进制日志，并给logbin起个名字
    log-bin = mysql_bin
    # ③ 指定日志格式，分别有mixed(由系统决定row还是statement)、row(记录磁盘变化)、statement(记录执行语句)，各有各的应用场景。
    binlog-format = mixed
    # ④ 重启mysql服务器
    sudo service mysql restart

# (2) 给从mysql添加一个授权帐号
GRANT REPLICATION CLIENT,REPLICATION SLAVE ON *.* TO 'repl'@'192.168.8.%' IDENTIFIED BY '123456';
FLUSH PRIVILEGES; # 冲刷权限

# (3) 查看mysql是否具备充当主mysql条件
SHOW MASTER STATUS;

# (4) 从mysql器也开启binlog
  # 打开配置文件
  vim /usr/local/mysql/my.cnf
  # 在[mysqld]字段下
    # ① 给从mysql起一个名字，防止多个主从混乱
    server-id = 102
    # ② 开启relaylog日志，并给relaylog起个名字
    relay-log=mysql_relay
    # ③ 重启mysql服务器：
    sudo service mysql restart

# (5) 在从mysql通过sql命令指定要复制的主mysql (可以一主多从，不能多主)
CHANGE MASTER TO
  MASTER_HOST='192.168.8.101',
  MASTER_USER = 'repl',
  MASTER_PASSWORD = '123456',
  MASTER_LOG_FILE = 'mysql_bin.000003', # 用语句SHOW MASTER STATUS;查看
  MASTER_LOG_POS = 120; # 用语句SHOW MASTER STATUS;查看

  # 配置连接信息后启动从mysql器功能
  START SLAVE;

  # 查看从mysql是否连接到主mysql
  SHOW SLAVE STATUS;

  # 如果有连接错误，ping下能否有网络连接，telnet 判断是否连接上，查看是否开启防火墙service iptables stop

  # 停止主从服务
  STOP SLAVE

# 注：主从复制的时间间隔一般为毫秒级别，达到秒级别的使用时风险比较大。
```

# 2. 主主复制

所谓双主备份，其实也就是互做主从复制，每台master既是master，又是另一台服务器的slave。这样，任何一方所做的变更，都会通过复制应用到另外一方的数据库中。

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200531092528.png)



配置主主复制流程：

- 两台mysql都设置二进制和relay日志
- 两台mysql都设置replication帐号
- 两台mysql都设置对方为自己的master

修改两台主mysql配置文件

```bash
# (1) 编辑配置文件：sudo vim /etc/mysql/my.cnf
  # 在[mysqld]字段下
    # ① 给主mysql起一个名字，在局域网内一般为ip最后一个数字，注意两台主mysql名字不能一样。
    server-id = 101
    # ② 开启logbin二进制日志，并给logbin起个名字
    log-bin = mysql_bin
    # ③ 指定日志格式，分别有mixed（由系统决定row还是statement）、row（记录磁盘变化）、statement（记录执行语句），各有各的应用场景。
    binlog-format = mixed
    # ④ 开启relaylog日志，并给relaylog起个名字
    relay-log=mysql_relay
    # ⑤ 重启mysql服务器
    sudo service mysql restart

# (2) 给从mysql添加一个授权帐号，两个主mysql授权帐号可以相同。
GRANT REPLICATION CLIENT,REPLICATION SLAVE ON *.* TO 'repl'@'192.168.8.%' IDENTIFIED BY '123456';
FLUSH PRIVILEGES; # 冲刷权限

# (3) 查看两个mysql是否具备充当主mysql条件
SHOW MASTER STATUS;

# (4) 互相连接对方的mysql
CHANGE MASTER TO
  MASTER_HOST='192.168.8.101',
  MASTER_USER = 'repl',
  MASTER_PASSWORD = '123456',
  MASTER_LOG_FILE = 'mysql_bin.000002', # 从SHOW MASTER STATUS查看对方的状态得到
  MASTER_LOG_POS = 560; # 从SHOW MASTER STATUS查看对方的状态得到

CHANGE MASTER TO
  MASTER_HOST='192.168.8.102',
  MASTER_USER = 'repl',
  MASTER_PASSWORD = '123456',
  MASTER_LOG_FILE = 'mysql_bin.000005', # 从SHOW MASTER STATUS查看对方的状态得到
  MASTER_LOG_POS = 1212; # 从SHOW MASTER STATUS查看对方的状态得到

# (5) 配置连接信息后都启动从mysql器功能
START SLAVE;

# (6) 查看能否互相连接到mysql
SHOW SLAVE STATUS;
```

这样设置后会有个主键冲突问题，即(表里有主键，并且是自动增长的，当分别同时插入两台主mysql时，同步的时候就出现主键冲突问题)，解决办法：

```bash
# (1) 在mysql表设置自动增长步长为2，一个mysql从1开始自动增长1 3 5......，另一个从2开始自动增长2 4 6......
# 第一台mysql：
set global auto_increment_increment = 2;
set global auto_increment_offset = 1;
set session auto_increment_increment = 2;
set session auto_increment_offset = 1;

# 第二台mysql：
set global auto_increment_increment = 2;
set global auto_increment_offset = 2;
set session auto_increment_increment=2;
set session auto_increment_offset = 2;

#注：auto-increment-increment和auto-increment-offset要写到配置文件中，防止下次重启后失效．这种方式也有缺陷，当新增一台主机作为主mysql时，又要从新修改自动增长步长。

# (2) 从业务上统一主键id
# 比如使用redis的incr命令(自动加1操作)，每插入一行数据前先执行redis的incr命令获取主机id值，然后在插入数据到mysql。
```

# 3. 被动模式下主主复制

被动模式下主主复制和普通主主复制区别就是设置一台主服务器为只读状态，也就是说把只读的主mysql作为另一个可读写主mysql的备份，当可读写的主mysql主机坏了，只需设置只读的主mysql配置为可读写就可以了，实现mysql无缝切换。

在主主配置的基础上，只需配置一台主mysql设置为只读。

```bash
# 打开一台主mysql配置文件
sudo vim /etc/mysql/my.cnf
# 在[mysqld]字段下添加字段
read-only=on

# 重启mysql
sudo service mysql restart
```

# 4. mysql集群负载均衡

通过mysql-proxy中间件来管理mysql集群，用户不需要知道集群中有多少个mysql服务器和地址，只需要连接mysql-proxy中间件，用户增删改查操作都是通过mysql-proxy中间件完成，mysql-proxy既可以mysql进行读写分离，也可以负载均衡，如下图所示：

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200531093124.png)



```bash
# 安装mysql-proxy中间件
# 下载源码
wget http://ftp.ntu.edu.tw/pub/MySQL/Downloads/MySQL-Proxy/mysql-proxy-0.8.4-linux-glibc2.3-x86-64bit.tar.gz

# 解压
tar zxvf mysql-proxy-0.8.4-linux-glibc2.3-x86-64bit.tar.gz

# 把解压文件夹移动到常用的安装软件目录下并修改名字，
mv mysql-proxy-0.8.4-linux-glibc2.3-x86-64bit /usr/local/mysql-proxy
```

## **(1) mysql-proxy说明**

mysql-proxy启动参数思路： - 代理了哪个端口? - 代理了哪写mysql服务? - 对mysql是否进行读写分离?

mysql-proxy是自动负载均衡的，这里的均衡并不是sql语句上的均衡，而是mysql-proxy和用户连接上的均衡，例如当前有20台mysql服务器，有1000个用户连接过来，此时mysql-proxy会把1000个连接数平均分给20台mysql服务器，一旦用户连接上了一台mysql服务器，在用户没有断开连接之前，用户的增删改查或事务操作都是该mysql服务器操作的。



## **(2) 启动mysql-proxy**

```bash
# 查看帮助
/usr/local/mysql-proxy/bin/mysql-proxy --help-all

# 普通启动
/usr/local/mysql-proxy/bin/mysql-proxy -P 192.168.8.102:4040 -b 192.168.8.101:3306 -b 192.168.8.102:3306
# -P 参数指定mysql-proxy运行的ip地址和端口
# -b 参数指定mysql服务器的ip地址和端口
# --daemon 表示在后台启动
```



## **(3) 设置读写分离mysql启动方式**

```bash
# mysql-proxy通过一个脚本文件判断用户输入的sql是写入还是读取，脚本位置在/usr/local/mysql-proxy/share/doc/mysql-proxy/rw-splitting.lua

# 读写分离启动mysql-proxy
/usr/local/mysql-proxy/bin/mysql-proxy -P 192.168.8.102:4040 -b 192.168.8.101:3306 -r 192.168.8.102:3306 -s /usr/local/mysql-proxy/share/doc/mysql-proxy/rw-splitting.lua
# -P参数指定mysql-proxy运行的ip地址和端口
# -b参数指定mysql服务器的ip地址和端口
# -r参数指定只读mysql服务器的ip地址和端口
# -s参数指定脚本文件的路径
# --daemon表示在后台启动

# 注：因为-s指定了脚本，脚本有默认最小和最大连接空闲连接数，当连接数超过默认值时，mysql-proxy才会做负载均衡。
```