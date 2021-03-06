---
title: MySQL 06.索引
date: 2022-07-01T08:19:19.685Z
tags: [mysql]
---
- [1. 索引长度](#1-索引长度)
- [2. 单列索引](#2-单列索引)
- [3. 多列索引](#3-多列索引)
- [4. 冗余索引](#4-冗余索引)
- [5. 索引操作](#5-索引操作)

# 1. 索引长度

对于定长数据类型(char，int，datetime)，如果有是否为NULL的标记，这个标记需要占用1个字节。

对于变长数据类型(varchar)，除了是否为NULL的标记外，还需要有长度信息，需要占用2个字节，当字段定义为NOT NULL的时候，是否为NULL的标记将不占用字节。

不同的字符集： - latin1编码一个字符一个字节； - gbk编码的为一个字符2个字节； - utf8编码的一个字符3个字节。

创建索引的时候可以指定索引的长度，例如：

> alter table test add index uri(uri(30));

长度30指的是字符的个数，如果为utf8编码varchar(255)，key_length=30*3+2=92个字节。

对于多数名称的前10个字符通常不同的列，用前10个字符作为索引不会比使用列全名创建的索引速度慢很多。另外，使用列的一部分创建索引可以使索引文件大大减小，从而节省了大量的磁盘空间，有可能提高INSERT操作的速度。

# 2. 单列索引

```sql
# 普通索引
key 列名(列名)
key cat_id(cat_id)

# 唯一索引
unique key
unique key email(email)

# 主键索引，一张表只能存在一个
primary key (列名)
primary key (id)

# 全文索引 fulltext
# 在中文环境下几乎无效。一般用第三方解决方案sphinx
```

索引长度：建索引时，可以取列的部分字符作为索引，节省空间， 例如：unique key email(email(10)); 取email列的前10个字符作为索引。

# 3. 多列索引

把两列或多列的值作为整体后再建索引，左前缀规则,例如key xm(xing, ming)

```sql
CREATE TABLE name (
  xing CHAR(2),
  ming CHAR(10),
  KEY xm(xing,ming)
);

# 在select前面添加explain关键字，得到结果的possible_key可以查看是否使用到索引查询。
# 使用索引xm查询
EXPLAIN SELECT * FROM name WHERE xing='刘' AND ming='备';

# 使用索引xm查询
EXPLAIN SELECT * FROM name WHERE xing='刘';

# 没有使用索引xm查询
EXPLAIN SELECT * FROM name WHERE ming='备';
```

# 4. 冗余索引

在某个列上可能存在多个索引,比如某个表：key xm(xing, ming)，key ming(ming)，对于列ming来说，索引xm和ming两个索引覆盖，叫做冗余索引。

# 5. 索引操作

```sql
# 查看索引
SHOW INDEX FROM 表名;

# 删除索引
ALTER TABLE 表名 DROP INDEX 索引名;

# 添加索引
# 添加普通索引
ALTER TABLE 表名 ADD INDEX 索引名(列名...);
# 添加唯一索引
ALTER TABLE 表名 ADD UNIQUE 索引名(列名);
# 添加主键索引
ALTER TABLE 表名 ADD PRIMARY KEY(列名);
```