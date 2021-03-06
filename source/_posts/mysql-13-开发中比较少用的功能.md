---
title: MySQL 13.开发中比较少用的功能
date: 2022-07-01T12:05:11.768Z
tags: [mysql]
---
- [1. 触发器](#1-触发器)
- [2. 存储过程](#2-存储过程)
  - [**(1) 声明变量**](#1-声明变量)
  - [**(2) 参数**](#2-参数)
  - [**(3) if条件控制结构**](#3-if条件控制结构)
  - [**(4) case选择控制结构**](#4-case选择控制结构)
  - [**(5) while循环结构**](#5-while循环结构)
  - [**(6) repeat循环结构**](#6-repeat循环结构)
- [3. 游标](#3-游标)

# 1. 触发器

触发器是一类特殊的事务，可以监视某种数据操作(insert|update|delete)，并触发相关操作(insert|update|delete)。

使用场合：有时碰到表中某些数据改变，希望同时引起改变其他数据改变的需求，利用触发器可以满足这样的需求。例如商城中的有客户下订单后，库存量、购买人数等这些数据需要跟着改变。

作用：在表中某些特定数据变化时自动完成某些查询，运用触发器不仅可以简化程序，并且可以增加程序的灵活性。

创建触发器语法的四要素：

- 监视地点(table表)
- 监视事件(insert | update | delete)
- 触发时间(before | after)
- 触发事件(insert | update | delete)

```bash
# 创建触发器
CREATE TRIGGER 触发器名
  BEFORE 或 AFTER # 触发时间
  INSERT 或UPDATE 或 DELETE # 监视事件
  ON 表名 # 监视地点
  FOR EACH ROW #在mysql中必须写，行级触发器，在oracle可以不写，表示语句级触发器
  BEGIN # 开始触发
    sql语句1 sql语句2 ......
  END # 结束触发


# 查看触发器
SHOW TRIGGERS;

# 删除触发器
DROP TRIGGER 触发器名;
```



监控insert行为时，引用变量用new，监控delete行为时，引用变量用old

```bash
# (1)订单表插入数据时，触发商品表对应的数据修改的触发器。
CREATE TRIGGER tr1
AFTER INSERT
ON orders
FOR EACH ROW
BEGIN
  UPDATE goods SET num=num-new.much WHERE gid=new.gid;
END;

# 改进版，当购买数量大于库存数量是，默认为库存数量,防止爆仓。
CREATE TRIGGER tr4
BEFORE INSERT
ON orders
FOR EACH ROW
BEGIN
  DECLARE rnum SMALLINT UNSIGNED DEFAULT 0;
  SELECT num INTO rnum FROM goods WHERE gid=new.gid;
  IF new.much>rnum THEN
    SET new.much = rnum;
  END IF;

  UPDATE goods SET num=num-new.much WHERE gid=new.gid;
END;


# (2)订单表删除数据时，触发商品表对应的数据修改的触发器（实际中订单只能失效，不能删除）。
CREATE TRIGGER tr2
AFTER DELETE
ON orders
FOR EACH ROW
BEGIN
  UPDATE goods SET num=num+old.much WHERE gid=old.gid;
END;
```



监控update行为时，引用变量update前用old，update后用new。

```bash
# (3)修改订单表数据时，触发商品表对应的数据修改的触发器。
CREATE TRIGGER tr3
BEFORE UPDATE
ON orders
FOR EACH ROW
BEGIN
  UPDATE goods SET num=num+old.much-new.much WHERE gid=new.gid;
END;
```

# 2. 存储过程

把若干条sql语句封装起来并起个名字，在过程中把数据存储到数据库中。

存储过程使用：

```bash
# 创建存储过程
CREATE PROCEDURE 名称()
  BEGIN
    # sql语句
  END;

# 调用存储过程
CALL 存储过程名字();

# 查看存储过程
SHOW PROCEDURE STATUS;

# 删除存储过程
DROP PROCEDURE 存储过程名字;
```



存储过程是可以编程的，意味着可以使用变量、表达式、控制结构来完成复杂的功能。

## **(1) 声明变量**

```bash
# 语法
DECLARE 变量名 变量类型 [default 默认值]
  # 注意：声明变量必须在begin和end之间声明。
  # 变量可以参与sql语句的运算，SET 变量名 := 表达式

# 示例
CREATE PROCEDURE test1()
  BEGIN
    DECLARE leng INT DEFAULT 0;
    DECLARE widch INT DEFAULT 0;
    SET leng := 5;
    SET widch := 6;
    SELECT leng*widch;
  END;
```



## **(2) 参数**

参数分为in、 out、 inout类型。in表示输入类型，out表示输出类型，inout表示输入输出类型。

```bash
# (1)in和out类型
CREATE PROCEDURE cuArea(in r INT, OUT area INT)
  BEGIN
    SET area:=0;# 如果输出area参与运算时必须设置area的初始值，因为null参与运算的值都为null
    SET area := 3.14*r*r;
  END;

# 调用：
CALL cuArea(10,@area);
SELECT @area;# 结果：314

# (2)inout类型
CREATE PROCEDURE add_1(INOUT v INT)
  BEGIN
    SET v := v + 1;
  END;

# 定义变量和调用
SET @v := 1;
CALL add_1(@v);
SELECT @v;  # 结果：2
```



## **(3) if条件控制结构**

```bash
# 语法
IF 条件1 THEN
ELSEIF条件2 THEN
......
ELSE
END IF;
# 其中ELSEIF和ELSE可以没有。

# 使用示例
CREATE PROCEDURE compare(v1 INT,v2 INT)
  BEGIN
    IF v1>v2 THEN
      SELECT concat(v1,'大于',v2);
    ELSEIF v1<v2 THEN
      SELECT concat(v1,'小于',v2);
    ELSE
      SELECT concat(v1,'等于',v2);
    END IF;
  END;
```



## **(4) case选择控制结构**

```bash
# 语法
CASE 变量
  WHEN 值 THEN 表达式;
  ......
  ELSE 不满足条件最后的默认结果;
END CASE;
# 注：else 可以省略。

# 使用示例
CREATE PROCEDURE cs()
  BEGIN
    DECLARE v INT;
    SET v := floor(rand()*10);
    CASE v
      WHEN 0 THEN SELECT '星期日';
      WHEN 1 THEN SELECT '星期一';
      WHEN 2 THEN SELECT '星期二';
      WHEN 3 THEN SELECT '星期三';
      WHEN 4 THEN SELECT '星期四';
      WHEN 5 THEN SELECT '星期五';
      WHEN 6 THEN SELECT '星期六';
      ELSE SELECT 'unknown day';
    END CASE;
  END;
```



## **(5) while循环结构**

```bash
# 语法
WHILE 条件 DO
  执行语句
END WHILE;
# 注：避免死循环

# 使用示例
CREATE PROCEDURE cusum (v INT)
  BEGIN
    DECLARE s INT DEFAULT 0;
    WHILE v>0 DO
      SET s := s + v;
      SET v := v-1;
    END WHILE;
    SELECT s;
  END;

# 调用：
CALL cusum(100);  # 结果：5050
```



## **(6) repeat循环结构**

```bash
# 语法
REPEAT
  执行语句......
UNTIL 条件 END REPEAT;

# 使用示例
CREATE PROCEDURE cuSum2(v INT)
  BEGIN
    DECLARE sum INT DEFAULT 0;
    REPEAT
      SET sum := sum+v;
      SET v:=v-1;
    UNTIL v<=0 END REPEAT;
    SELECT sum;
  END;

# 调用
CALL cuSum2(100); # 结果：5050
```

# 3. 游标

一条sql的select语句取出对应的n条资源，取出资源的接口(句柄)就是游标，沿着游标，每次只取出一行，取出的行可以任意的逻辑控制了，而select没有这种功能。

```bash
# 声明游标
DECLARE 游标名 CURSOR FOR select语句;
# 设置触发边界标志
DECLARE EXIT HANDLER FOR NOT FOUND 表达式;
# 打开游标
OPEN 游标名;
# 取值
FETCH游标名INTO 变量1, 变量2 ......;
# 关闭游标
CLOSE 游标名;

# 用循环读取游标数据，结束条件是判断是否去到最后一条数据(事先计算出来的总数)。

# 使用示例
CREATE PROCEDURE cursor1()
  BEGIN
    DECLARE tmp_name VARCHAR(20);
    DECLARE tme_num INT;
    DECLARE cnt INT;
    DECLARE i INT DEFAULT 0;

    DECLARE get_goods CURSOR FOR SELECT name,num FROM goods;
    OPEN get_goods;
    SELECT count(*) INTO cnt FROM goods;
    WHILE i<cnt DO
      SET i:=i+1;
      FETCH get_goods INTO tmp_name,tme_num;# 取一条数据
      SELECT tmp_name,tme_num;
    END WHILE;

    CLOSE get_goods;
  END;

# 调用
CALL cursor1();
```



在mysql的cursor中可以用declare exit或continue handler for not fond来操作越界标志。类似于js中的事件，当读取游标完毕则触发该事件。其中exit和continue的区别是是否执行后面的sql语句。

```bash
# (1)触发越界后执行exit，不执行后面的sql语句
CREATE PROCEDURE cursor2()
  BEGIN
    DECLARE tmp_name VARCHAR(20);
    DECLARE tme_num INT;
    DECLARE isEnd BOOL DEFAULT FALSE;

    DECLARE get_goods CURSOR FOR SELECT name,num FROM goods;
    DECLARE EXIT HANDLER FOR NOT FOUND SET isEnd:=TRUE ;# 设置触发边界标志
    OPEN get_goods;

    WHILE !isEnd DO
      FETCH get_goods INTO tmp_name,tme_num;# 取一条数据
      SELECT tmp_name,tme_num;
    END WHILE;

    CLOSE get_goods;
  END;

# 调用
CALL cursor2();

# (2)触发越界后执行continue，继续后面的sql语句。
CREATE PROCEDURE cursor3()
  BEGIN
    DECLARE tmp_name VARCHAR(20);
    DECLARE tme_num INT;
    DECLARE isEnd BOOL DEFAULT FALSE;

    DECLARE get_goods CURSOR FOR SELECT name,num FROM goods;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET isEnd:=TRUE ;# 设置触发边界标志
    OPEN get_goods;

    FETCH get_goods INTO tmp_name,tme_num;# 进入循环前先取一条数据
    WHILE !isEnd DO
      SELECT tmp_name,tme_num;
      FETCH get_goods INTO tmp_name,tme_num;# 取一条数据
    END WHILE;

    CLOSE get_goods;
  END;

# 调用
CALL cursor3();
```