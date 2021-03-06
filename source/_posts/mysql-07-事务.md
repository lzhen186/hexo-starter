---
title: MySQL 07.事务
date: 2022-07-01T08:25:21.240Z
tags: [mysql]
---
- [1. 事务特性(ACID)](#1-事务特性acid)
- [2. 设置事务级别](#2-设置事务级别)

# 1. 事务特性(ACID)

- 原子性(Atomicity）：是指某几句sql的影响，要么都发生，要么都不发生。
- 一致性(Consistency)：事务前后的数据，保持业务上的合理一致。
- 隔离性(Isolation)：在事务进行过程中，其他事务看不到此事务的任何效果。
- 持久性(Durability)：事务一旦发生，不能取消，只能通过补偿性事务来抵消效果。

事务与引擎：myisam引擎不支持事务，innodb和BDB引擎支持。

一个完整的事务过程：

- (1) 启动事务：START TRANSACTION;
- (2) sql执行：增删改查，如果出错，回滚ROLLBACK
- (3) 结束事务：COMMIT(提交)或ROLLBACK(取消);

注意：commit一旦发生之后，rollback无法回滚。

```bash
# 示例
START TRANSACTION;
UPDATE account SET balance=balance+1000 WHERE name='曹操';
UPDATE account SET balance=balance-1000 WHERE name='刘备';
COMMIT;
```

# 2. 设置事务级别

set session transaction isolation level [read uncommitted | read committed | repeatable read | serializable]

- read uncommitted：可以读未提交的事务内容，称为”脏读”，破坏了事务的隔离性，一般不用。
- read commited：在一个事务进行过程中，读不到另一个进行事务的操作，但是可以读到另一个结束事务的操作影响，一般不用。
- repeatable read：在一个事务过程中，所有信息都来自事务开始那一瞬间的信息，不受其他已提交事务的影响，大多数的系统，用此隔离级别，建议使用。
- serializeable：串行化，把所有事务进行编号，按顺序一个一个来执行，也就取消了事务冲突的可能，隔离级别最高，但事务相互等待的时间会比较长，在实际中使用也不是很多。