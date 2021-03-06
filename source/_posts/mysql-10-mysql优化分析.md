---
title: MySQL 10.mysql优化分析
date: 2022-07-01T11:33:14.369Z
tags: [mysql]
---
- [1. 获取查询和连接数量](#1-获取查询和连接数量)
- [2. mysql进程状态分布](#2-mysql进程状态分布)
- [3. 使用profile记录各个sql执行时间](#3-使用profile记录各个sql执行时间)
- [4. 使用explain分析sql执行效果](#4-使用explain分析sql执行效果)
- [5. 开启慢查询日志](#5-开启慢查询日志)

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200531090221.png)

# 1. 获取查询和连接数量

```bash
# 在mysql命令行查看服务器状态
SHOW STATUS;

# 在linux命令行查看服务器状态
mysqladmin -u root -p 123456 ext

# 在linux命令行获取上面三个参数值：
mysqladmin -u root -p 123456 ext | awk '/Queries/{q=$4} /Threads_connected/{c=$4} /Threads_running/{r=$4} END{printf "%d %d %d\n", q, c, r}'
```

获得mysql服务器状态数据，主要看3个参数：

- Queries: 查询次数
- Threads_connected: 线程连接数
- Threads_running: 线程运行数



一般使用脚本循环获取mysql状态，脚本内容如下：

```bash
#!/bin/bash

while true
do
   mysqladmin -uroot ext | awk '/Queries/{q=$4} /Threads_connected/{c=$4} /Threads_running/{r=$4} END{printf "%d  %d  %d\n", q,c,r}' >> status.txt
   sleep 1
done
```

# 2. mysql进程状态分布

```bash
# 在mysql命令行查看服务器状态
SHOW PROCESSLIST;

# 在linux命令行查看服务器状态：
mysql -uroot -p123456 -e 'show processlist \G'

# 如果只关心State状态这行，在linux下执行命令获取State
mysql -uroot -p123456 -e 'show processlist \G' | grep State
```

一般使用脚本循环获取mysql进程状态，脚本内容如下：

```bash
#!/bin/bash

while true
do
    mysql -uroot -e 'show processlist \G' | grep State >> process.txt
    usleep 100000
done
```

# 3. 使用profile记录各个sql执行时间

```bash
# (1) 打开profile功能，打开profile功能后，执行的每一条sql语句都会被记录下来。
SET PROFILING =1;

# (2) 查看执行所有sql耗时时间列表(在phpstorm可以按使用时间排序)，可以找到有问题的语句(耗时比较大的)
SHOW PROFILES;

# (3) 查看sql语句详细执行过程，可以分析该语句耗时都用在哪个状态下，其中数字72是在SHOW PROFILES命令显示的查询id。
SHOW PROFILE for query 72;

# (4) 关闭profile功能：
SET PROFILING =0;
```

# 4. 使用explain分析sql执行效果

explain分析结果字段说明：

| 属性名        | 说明                                                         |
| :------------ | :----------------------------------------------------------- |
| id            | 查询的编号，简单查询通常为1，如果有子查询会增加子查询行编号  |
| select_type   | (1) simple：不含子查询的简单的查询； (2) primary：含有子查询或派生查询， (subquery表示非from子查询)、(derived表示from型子查询)、 (union)、(union result) |
| table         | 查询针对的表名，如果原表名加了别名，就显示表的别名，也有可能是null |
| type          | (1) All: 全表扫描，在磁盘全表扫描是非常慢的，尽量避免出现全表扫描这样的查询； (2) index: 扫描索引的所有节点，因为在内存扫描，性能比All要好，虽然说尽量使用索引，但不希望从头到尾扫描一遍； (3) range: 根据索引做范围扫描，比index性能好； (4) ref: 通过索引列直接引用到某些数据行，比range性能好； (5) eq_ref: 从表中读取一行，在联表查询使用到索引时经常出现，比ref性能好； (6) const |
| possible_keys | 可能用到的索引                                               |
| key           | 最终使用的索引                                               |
| key_len       | 索引的长度，长度越短越好。                                   |
| ref           | 引用了哪索引或列                                             |
| rows          | 估计扫描的行                                                 |
| Extra         | (1) using index：使用索引覆盖； (2) using where：使用了条件查询； (3) using temporary：使用临时表，尽量不出现； (4) using filesort：使用了排序，尽量不出现； (5) range checked for each record：使用了在范围检查每个记录，不要出现扫描的行 |

使用：

explain命令+耗时比较多sql查询语句可以查看该语句是否使用了索引，使用索引说明已经优化过，没有使用索引则对表进行优化。

```bash
# 查看有索引的查询
EXPLAIN SELECT * FROM plan_task WHERE id=100000;
# 从结果可以看出已经使用了索引，type为const，extra为null。

# 查看没有索引的查询
EXPLAIN SELECT * FROM plan_task WHERE background_color=100000;
# 从结果看出没有使用到索引，type为ALL，extra为using where。
```

# 5. 开启慢查询日志

(1) 开启慢查询日志有两种，一是全局变量设置，二是设置配置文件。

全局变量设置：

```bash
# 将slow_query_log全局变量设置为“ON”状态
mysql> set global slow_query_log='ON';

# 设置慢查询日志存放的位置
mysql> set global slow_query_log_file='/usr/local/mysql/data/slow.log';

# 查询超过1秒就记录
mysql> set global long_query_time=1;
```



修改配置文件my.cnf，在[mysqld]下的下方加入下面内容：

```bash
[mysqld]
slow_query_log = ON
slow_query_log_file = /usr/local/mysql/data/slow.log
long_query_time = 1
```

重启mysql服务：service mysqld restart



(2) 查看设置后的参数

```bash
show variables like 'slow_query%';
+---------------------+--------------------------------+
| Variable_name       | Value                          |
+---------------------+--------------------------------+
| slow_query_log      | ON                             |
| slow_query_log_file | /usr/local/mysql/data/slow.log |
+---------------------+--------------------------------+

show variables like 'long_query_time';
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 1.000000 |
+-----------------+----------+
```



(3) 验证慢查询查日志是否开启成功

```bash
# 执行一条慢查询SQL语句
select sleep(2);

# 查看是否生成慢查询日志
cat /usr/local/mysql/data/slow.log
```