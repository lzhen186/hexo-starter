---
title: SpringBoot 03.spring boot配置原理实战 3.2.详解YAML语法及占位符语法
date: 2022-07-03T00:17:21.140Z
tags: [springboot]
---
# 1. YAML

YAML（YAML Ain't Markup Language）

 YAML A Markup Language：是一个标记语言

 YAML isn't Markup Language：不是一个标记语言；

标记语言：

 以前的配置文件；大多都使用的是 **xxxx.xml**文件；

 YAML：**以数据为中心**，比json、xml等更适合做配置文件；

# 2. YAML语法

## 2.1 基本语法

k: (空格)v : 表示一堆键值对(空格必须有)

以`空格`的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的

次等级的前面是空格，不能使用制表符(tab)

冒号之后如果有值，那么冒号和值之间至少有一个空格，不能紧贴着

```yaml
server:
	port: 8080
	path: /hello
```

## 2.2 值得写法

### 1. 字面量：普通的值（数字，字符串，布尔）

```yaml
k: v
```

字符串默认不用加上单引号或者双引号；

`""`：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思

*eg：* name: "zhangsan \n lisi"：输出；zhangsan 换行 lisi

`''`：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据

*eg：* name: ‘zhangsan \n lisi’：输出；zhangsan \n lisi

### 2. 对象、Map（属性和值）

`k: v`在下一行来写对象的属性和值的关系；注意缩进

1. ```yaml
   person:
     name: 张三
     gender: 男
     age: 22Copy to clipboardErrorCopied
   ```

2. 行内写法

   ```yaml
   person: {name: 张三,gender: 男,age: 22}
   ```

### 3. 数组（List、Set）

1. ```yaml
   fruits: 
     - 苹果
     - 桃子
     - 香蕉Copy to clipboardErrorCopied
   ```

2. 行内写法

   ```yaml
   fruits: [苹果,桃子,香蕉]
   ```

# 3. 简单示例

## 一、设计一个YAML数据结构

```
#    1. 一个家庭有爸爸、妈妈、孩子。
#    2. 这个家庭有一个名字（family-name）叫做“happy family”
#    3. 爸爸有名字(name)和年龄（age）两个属性
#    4. 妈妈有两个别名
#    5. 孩子除了名字(name)和年龄（age）两个属性，还有一个friends的集合
#    6. 每个friend有两个属性：hobby(爱好)和性别(sex)
```

上面的数据结构用yaml该如何表示呢？

```yaml
family:
  family-name: "happy family"
  father:
    name: zimug
    age: 18
  mother:
    alias:
      - lovely
      - ailice
  child:
    name: zimug
    age: 5
    friends:
      - hobby: football
        sex:  male
      - hobby: basketball
        sex: female
```

或者是friends的部分写成

```yaml
 friends:
      - {hobby: football,sex:  male},
      - {hobby: basketball,sex: female}
```

### 规则1：字符串的单引号与双引号

- 双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思，如：
  ​ name: “zhangsan \n lisi”：输出；zhangsan 换行 lisi
- 单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据，如：
  ​ name: ‘zhangsan \n lisi’：输出；zhangsan \n lisi

### 规则2：支持松散的语法

```
family-name = familyName = family_name
```

## 二、配置文件占位符

Spring Boot配置文件支持占位符，一些用法如下

### 2.1 随机数占位符

```yaml
${random.value}
${random.int}
${random.long}
${random.int(10)}
${random.int[1024,65536]}
```

### 2.2 默认值

占位符获取之前配置的值，如果没有可以是用“冒号”指定默认值
格式例如，xxxxx.yyyy是属性层级及名称，如果该属性不存在，冒号后面填写默认值

```yaml
${xxxxx.yyyy:默认值}
```