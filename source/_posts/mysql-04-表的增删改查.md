---
title: MySQL 04.表的增删改查
date: 2022-07-01T08:05:12.685Z
tags: [mysql]
---
- [1. insert 插入数据操作](#1-insert-插入数据操作)
- [2. delete 删除数据操作](#2-delete-删除数据操作)
- [3. update 修改数据操作](#3-update-修改数据操作)
- [4. select 查询数据操作](#4-select-查询数据操作)

# 1. insert 插入数据操作

语法：

- 往哪张表添加行：INSERT INTO 表名
- 给那几个列添加值：列名称(可省略)
- VALUES：列对应的值

插入操作时注意事项：列和值必须一一对应，并且符合类型要求。

```sql
# user表有id、name、age三列

INSERT INTO user (id, name, age) VALUES (1, '张三', 25);
INSERT INTO user (name, age) VALUES ('李四', 26);
INSERT INTO user (name) VALUES ('王五');

# 忽略列时需要写完所有对应列的值
INSERT INTO user VALUES (5, '刘备', 52);
# 列没有严格对应，执行错误
#INSERT INTO user  VALUES ('关羽',45);

# 一次插入多行数据
INSERT INTO user (name, age) VALUES ('关羽', 45),('张飞', 46);
```

# 2. delete 删除数据操作

语法：

- 删除一张表的数据：DELETE FROM 表名
- 删掉表中的哪些行：WHERE 表达式

注意事项：删除必须写where约束条件，也不能写常量，如where 1，否则会删除整张表数据。

```sql
# user表有id、name、age三列

DELETE FROM user WHERE age=26;
DELETE FROM user WHERE uid>2;
```

# 3. update 修改数据操作

语法：

- 改哪一张表：UPDATE 表名
- 改哪几列的值：SET 列名=值1，列名=值2 ……
- 在哪些行生效：WHERE 表达式

注意事项：一定要有where约束条件，即在哪些行生效。

```sql
# user表有id、name、age三列

UPDATE user SET age=27 WHERE name='王五';

# 修改多列用逗号隔开
UPDATE user SET name='赵六',age=28 WHERE uid=4;
```

# 4. select 查询数据操作

语法：

- 查询哪些列数据：SELECT列名1 列名2 ……
- 从哪张表查询：FROM 表名
- 选择哪些行生效：WHERE 表达式

```sql
# user表有id、name、age三列

SELECT * FROM user; # 实际开发中很少使用
SELECT * FROM user WHERE name='关羽';
SELECT name,age FROM user WHERE age<30; # 查询符合条件的指定列
```

