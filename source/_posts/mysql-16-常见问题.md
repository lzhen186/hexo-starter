---
title: MySQL 16.常见问题
date: 2022-07-01T12:26:30.653Z
tags: [mysql]
---
- [常见问题](#常见问题)

# 常见问题

mysql拒绝连接问题

```bash
# (1) 首先检查网络是否连通

  # 是否能ping通网络
  ping <ip>

  # 检测端口是否有开放，先要安装telnet工具
  telnet <ip> <port>

# (2) 从机器内部定位
  # 查看mysql服务是否开启
  ps -ef | grep mysqld
  # 查看tcp端口是否开启
  netstat -lnpt
  # 查看mysql是否指定了bind-address
  vim /etc/mysql/my.cnf
  #查看mysql的账号是否允许外部连接(需要重启mysql：/etc/init.d/mysqld restart)
    mysql -u root -p
    use mysql
    select user,host from user;
    update user set host=‘%’ where user=‘root’;
    或
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
  # 查看防火墙是否开启mysql端口
    iptables -S
    /sbin/iptables -I INPUT -p tcp --dport 8011 -j ACCEPT #开启8011端口
    /etc/init.d/iptables restart
```