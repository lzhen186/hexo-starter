---
title: MySQL 11.表分区
date: 2022-07-01T11:48:09.579Z
tags: [mysql]
---
- [1. 按范围来分区](#1-按范围来分区)
- [2. 按散列点对表进行分区](#2-按散列点对表进行分区)

表分区是把原来一张表按一定方式拆分为多个相同类型的表，同时原来一张表的数据文件也会拆分为多个数据文件，拆分方式可以按范围、散列点方式来分区。

表分区的好处：拆分后可以打开的线程数更多了，因为数据在不同的文件上，文件被锁的可能性会降低，一定程度上提高了读写速度。 例如单个.myd文件都达到了10G，读取数据时效率降低，通过表分区把10拆分为1G，可以提高效率。

对用户使用来说，表有没有分区是无影响的，增删改查操作还是一样。

# 1. 按范围来分区

```bash
# 例如有一张用户表，按主键id的范围对用户表分区
CREATE TABLE user(
  id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT, # 主键
  name VARCHAR(20) NOT NULL DEFAULT '',   # 名称
  aid TINYINT UNSIGNED NOT NULL DEFAULT 0 # 地区代号
)ENGINE myisam CHARSET utf8
  PARTITION BY RANGE (id)(    # 按主键id范围对表进行分区
  PARTITION u0 VALUES LESS THAN (10), # 第1个分区范围
  PARTITION u1 VALUES LESS THAN (20), # 第2个分区范围
  PARTITION u2 VALUES LESS THAN (MAXVALUE)    # 最后一个分区范围
);
# 也可以按时间按范围来分表，例如年、月
```

# 2. 按散列点对表进行分区

```bash
# 假如有一张地区表和一张用户表，按地区代号对用户表进行分区
# 地区表：
CREATE TABLE area(
  aid INT UNSIGNED PRIMARY KEY AUTO_INCREMENT, # 主键
  addr VARCHAR(10) NOT NULL DEFAULT '' # 地区名称，不要为null
)ENGINE myisam CHARSET utf8;

# 用户表：
CREATE TABLE user(
  id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT, # 主键
  name VARCHAR(20) NOT NULL DEFAULT '',   # 名称
  aid TINYINT UNSIGNED NOT NULL DEFAULT 0 # 地区代号
)ENGINE myisam CHARSET utf8
  PARTITION BY LIST (aid)(    # 按散列点(地区代号)对表进行分区
  PARTITION a1 VALUES IN (1), # 按地区代号分区
  PARTITION a2 VALUES IN (2), # 按地区代号分区
  PARTITION a3 VALUES IN (3), # 按地区代号分区
  PARTITION a4 VALUES IN (4)  # 按地区代号分区
);
```