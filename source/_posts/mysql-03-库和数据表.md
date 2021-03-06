---
title: MySQL 03.库和数据表
date: 2022-07-01T07:57:48.668Z
tags: [mysql]
---
- [1. 库](#1-库)
- [2. 数据表](#2-数据表)
  - [2.1 表的列操作](#21-表的列操作)
    - [**(1) 查看表字段信息**](#1-查看表字段信息)
    - [**(2) 查看表的所有信息**](#2-查看表的所有信息)
    - [**(3) 查看当前库下所有表信息**](#3-查看当前库下所有表信息)
    - [**(4) 创建表**](#4-创建表)
    - [**(5) 在原表上添加新的列**](#5-在原表上添加新的列)
    - [**(6) 删除表**](#6-删除表)
    - [**(7) 删除列**](#7-删除列)
    - [**(8) 清空表内容**](#8-清空表内容)
    - [**(9) 修改表名称**](#9-修改表名称)
    - [**(10) 修改列属性**](#10-修改列属性)
    - [**(11) 修改列名称和属性**](#11-修改列名称和属性)
    - [**(12) 添加主键约束**](#12-添加主键约束)
    - [**(13) 删除主键约束**](#13-删除主键约束)
    - [**(14) 删除查询缓存**](#14-删除查询缓存)
  - [2.2 列的属性](#22-列的属性)
    - [**(1) 列的默认属性**](#1-列的默认属性)
    - [**(2) 主键与自增**](#2-主键与自增)
  - [2.3 表引擎](#23-表引擎)

# 1. 库

数据库就是存放数据的仓库，这个仓库是按照一定的数据结构来组织存储的。

```sql
# 显示数据库
show databases;

# 选择数据库
use dbname;

# 创建数据库
create database dbname charset utf8;

# 删除数据库
drop database dbname;
```

# 2. 数据表

数据表是关系数据库中一个非常重要的对象，是其它对象的基础，也是一系列二维数组的集合，用来存储和操作数据的逻辑结构。

## 2.1 表的列操作

### **(1) 查看表字段信息**

```sql
# 语法
DESC 表名

# 示例
DESC student;
```

### **(2) 查看表的所有信息**

```sql
# 语法
SHOW CREATE TABLE 表名

# 示例
SHOW CREATE TABLE student;
```



### **(3) 查看当前库下所有表信息**

```sql
# 语法
SHOW TABLE status

# 示例
# 列形式输出
SHOW TABLE status \G;

#或过滤表输出
SHOW TABLE status WHERE name='goods' \G;
```



### **(4) 创建表**

```sql
# 语法
CREATE TABLE 表名(列名 列类型 列属性 ......)

# 示例
CREATE TABLE student (
  sid TINYINT UNSIGNED,
  name VARCHAR(20),
  age INT
);
```



### **(5) 在原表上添加新的列**

```sql
# 语法
ALTER TABLE 表名 ADD COLUMN 列名 列类型 列属性 ......

# 示例
# 在最后添加列
ALTER TABLE student ADD COLUMN sn TINYINT(6) ZEROFILL;

# 在指定位置添加列：
ALTER TABLE user ADD height TINYINT AFTER weight;
```



### **(6) 删除表**

```sql
# 语法
DROP TABLE 表名

# 示例
DROP TABLE student;
```



### **(7) 删除列**

```sql
# 语法
ALTER TABLE 表名 DROP 列名

# 示例
ALTER TABLE student DROP COLUMN sn;
```



### **(8) 清空表内容**

TRUNCATE相当与drop和creat表两个操作。

```sql
# 语法
TRUNCATE 表名

# 示例
TRUNCATE student;
```



### **(9) 修改表名称**

```sql
# 语法
ALTER TABLE 旧表名 RENAME TO 新表名

# 示例
ALTER TABLE student RENAME TO stu;
```



### **(10) 修改列属性**

```sql
# 语法
ALTER TABLE 表名 MODIFY sid 列名 列类型 列属性

# 示例
ALTER TABLE student MODIFY sid TINYINT UNSIGNED;
```



### **(11) 修改列名称和属性**

```sql
# 语法
ALTER TABLE 表名 CHANGE 旧列名 新列名 列类型 列属性

# 示例
ALTER TABLE student CHANGE sid id INT UNSIGNED;
```



### **(12) 添加主键约束**

```sql
# 语法
ALTER TABLE 表名 ADD CONSTRAINT PRIMARY KEY 表名(列名)

# 示例
ALTER TABLE student ADD CONSTRAINT PRIMARY KEY student(sid);
```



### **(13) 删除主键约束**

```sql
# 语法
ALTER TABLE 表名 DROP PRIMARY KEY

# 示例
ALTER TABLE student DROP PRIMARY KEY;
```



### **(14) 删除查询缓存**

```sql
# 语法
RESET QUERY CACHE
```

## 2.2 列的属性

### **(1) 列的默认属性**

实际使用中避免列的默认值为null，创建表时给列一个初始值，在列的类型后面添加NOT NULL DEFAULT值，示例：id INT UNSIGNED NOT NULL DEFAULT 0

### **(2) 主键与自增**

主键(primary key)，能够区分每一行，一般和auto_increment一起使用，有两种方式声明该列为主键：

- 在列的类型之后声明primary key，例如：id INT UNSIGNED PRIMARY KEY
- 声明完列之后，在最后声明那一列为主键primary key(列名)，PRIMARY KEY(id)

主键与自增搭配使用，例如：id INT PRIMARY KEY AUTO_INCREMENT

提高效率建表原则：定长与变长分离，常用和不常用分离。



## 2.3 表引擎

MySQL常用两种表引擎是Myisam和InnoDB，引擎不同，组织数据方式也不同，新版本MySQL可以统一使用InnoDB，性能比Myisam差别不是太大。

```sql
CREATE TABLE account(
  id INT UNSIGNED PRIMARY KEY,
  name CHAR(10) NOT NULL DEFAULT '',
  balance INT NOT NULL DEFAULT 0
)ENGINE innodb CHARSET utf8;
```