---
title: MongoDB 03.mongoDB增删改查操作
date: 2022-07-02T00:06:27.670Z
tags: [mongodb]
---
# 1. 库和集合的基础命令

## 1. 库级命令

```bash
# 查看所有数据库
show dbs # 或使用mysql命令show databases;

# 切换到指定数据库
use 库名

# 创建数据库(mongodb不提供命令)
# mongodb的数据库是隐式创建的(创建表时自动创建)，就算没有指定的库，也可以使用use命令。

# 删除数据库
db.dropDatabase()
```

## 2. 表(集合)级命令

```bash
# 显示所有表
show collections; # 或mysql命令show tables;

# 创建表
db.createCollection('user')

# 删除表
db.user.drop()
```

# 2. insert插入数据

## 1. 增加一个文档

```bash
# 不指定id
db.user.insert({name:'张三',age:22})

# 指定id
db.user.insert({_id:10001,name:'李四',age:23})
```



## 2. 一次增加多个文档

```bash
db.user.insert([{name:'张三',age:24},{_id:10002,name:'李四',age:25},{name:'王五',age:26}])
```

# 3. remove删除数据

语法： 

```bash
db.collection.remove(查询表达式,选项)
```

查询表达式: 类似mysql的where条件，不加条件会删除表所有数据。 选项: 是否删除一行，{justOne:true/false}，默认是false

```bash
# 示例
db.user.remove({_id:10001})
db.user.remove({age:24},{ justOne:true})
```

注：实际应用中一般不使用删除，使用一个字段来标记删除。

# 4. update修改数据

语法：

> ```bash
> db.collection.updae(查询表达式,新值,选项)
> ```

错误示例：

> ```bash
> db.user.update({name:‘张三’},{age:33})
> ```

注意：这种方式是新文档直接替换了旧文档，必须指定赋值表达式，常用表达式有：

| 表达式       | 说明                     |
| :----------- | :----------------------- |
| $set         | 设置字段新的值           |
| $unset       | 删除指定的列             |
| $inc         | 增长                     |
| $rename      | 重命名该列               |
| $setOnInsert | 当upsert时，设置字段的值 |

## 1. 赋值表达式

```bash
# $set设置字段新值(如果字段不存在则新增加该列)
db.user.update({name:'张三'},{$set:{age:33}})

# $inc自增字段值
db.user.update({name:'张三'},{$inc:{age:1}})

# $rename修改字段名
db.user.update({name:'张三'},{$rename:{sex:'gender'}})

# $unset删除指定列
db.user.update({name:'张三'},{$unset:{phone:1}})

# setOnInsert当upsert为true时，添加附加字段
db.user.update({name:'赵六'},{$setOnInsert:{age:25,gender:'male'}},{upsert:true})

# 多种赋值表达可以一次执行
db.collection.updae(查询表达式,{
{$set:{...}},
{$unset:{...}},
{$inc:{...}},
{$rename:{...}}
})
```

## 2. 修改选项

选项包括upsert和multi

- upsert：没有匹配的行，是否直接插入该行，默认为false。
- multi：查询表达式匹配中多行，是否修改多行，默认false(只修改一行)

```bash
# 示例
db.user.update({name:'李四'},{$set:{age:44}},{upsert:true})
db.user.update({gender:'male'},{$set:{age:55}},{multi:true})
```

# 5. query查询

```bash
# 语法：db.collection.find(查询表达式,查询的列)
db.user.find() # 查询所有

# 查询匹配条件的所有列
db.user.find({name:'张三'})

# 查询匹配条件的指定列({列名:1,列名:1,...})
db.user.find({name:'张三'},{age:1})

# 不查询id列
db.user.find({name:'张三'},{_id:0,age:1})
```

## 1. 字段值查询

```bash
# 查询主键为32的商品
db.goods.find({goods_id:32})

# 子文档查询(指向子文档的key是字符串)
db.stu.find({'score.yuwen':75})

# 数组元素查询(指向子文档的key是字符串)
db.stu.find({'hobby.2':'football'})
```

## 2. 范围查询

```bash
# $ne 不等于
# 查询不属第3栏目的所有商品
db.goods.find({cat_id:{$ne:3}})

# $gt 大于
# 查询高于3000元的商品
db.goods.find({shop_price:{$gt:3000}})

# $gte 大于等于
# 查询高于或等于3000元的商品
db.goods.find({shop_price:{$gte: 3000}})

# $lt 小于
# 查询低于500元的商品
db.goods.find({shop_price:{$lt:500}})

# $lte 小于等于
# 查询低于500元的商品
db.goods.find({shop_price:{$lte:500}})
```

## 3. 集合查询

```bash
# $in 在集合，(当查询字段是数组时，不需完全匹配也会命中该行)
# 查询栏目为4和11的商品
db.goods.find({cat_id:{$in:[4,11]}})

# $nin 不在集合
# 查询栏目不是3和4的商品
db.goods.find({cat_id:{$nin:[3,4]}})

# $all完全匹配，(当查询字段是数组时，必须完全匹配才命中该行)
db.goods.find({cat_id:{$all:[4,11]}})
```

## 4. 逻辑查询

```bash
# $and 逻辑与，必须都满足条件
# 查询价格为100到500的商品
db.goods.find({$and:[{shop_price:{$gt:100}},{shop_price:{$lt:500}}]})

# $or 逻辑或，至少满足一个条件
# 查询价格小于100或大于3000的商品
db.goods.find({$or:[{shop_price:{$lt:100}},{shop_price:{$gt:3000}}]})

# $nor 与非逻辑
# 查询不在栏目3，并且价格不小1000的商品
db.goods.find({$nor:[{cat_id:3},{shop_price:{$lt:1000}}]})
```

## 5. 元素运算符查询

```bash
# $mod 求余
# 查询年龄对10求余为0的用户
db.goods.find({age:{$mod:[10,0]}})

# $exist 查询列是否存在
# 查询有电话属性的用户
db.user.find({phone:{$exists:1}})

# $type 查询属性为指定类型的文档
# 查询为浮点型的文档
db.user.find({phone:{$type:1}})

# 查询为字符串型的文档
db.user.find({phone:{$type:2}})
```

## 6. 其他常用查询

```bash
# 统计行数
db.goods.count()
db.goods.find({cat_id:3}).count()

# 查询每页数据
    # 按指定属性排序，1表示升序，-1表示降序
    sort({属性:1|-1})
    # 跳过行数
    skip(num)
    # 限制取多少行
    limit(num)

# 页码page，每页行数n，则skip数=(page-1)*n
db.goods.find({cat_id:3}).sort({shop_price:-1}).skip(0).limit(10)
```

