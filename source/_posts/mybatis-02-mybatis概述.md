---
title: MyBatis 02.mybatis概述
date: 2022-07-01T04:40:52.011Z
tags: [mybatis]
---
# 1. mybatis 介绍

**[MyBatis 3.4 参考文档中文版](https://www.bookstack.cn/read/MyBatis-zh-3.4/typeHandlers)**

* mybatis是一个持久层的框架，是apache下的顶级项目。

  mybatis托管到goolecode下，再后来托管到github下(https://github.com/mybatis/mybatis-3/releases)。
* mybatis让程序将主要精力放在sql上，通过mybatis提供的映射方式，自由灵活生成（半自动化，大部分需要程序员编写sql）满足需要sql语句。
* mybatis可以将向 preparedStatement中的输入参数自动进行输入映射，将查询结果集灵活映射成java对象。（输出映射）

# 2. 框架原理

mybatis框架

![](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimage.codes51.com%2FArticle%2Fimage%2F20170710%2F20170710181330_4531.png&refer=http%3A%2F%2Fimage.codes51.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1659243137&t=337ca4f4486c54381385d7e2238e166b)

# 3. mybatis框架执行过程

1、配置mybatis的配置文件，SqlMapConfig.xml（名称不固定）

2、通过配置文件，加载mybatis运行环境，创建SqlSessionFactory会话工厂(SqlSessionFactory在实际使用时按单例方式)

3、通过SqlSessionFactory创建SqlSession。SqlSession是一个面向用户接口（提供操作数据库方法），实现对象是线程不安全的，建议sqlSession应用场合在方法体内。

4、调用sqlSession的方法去操作数据。如果需要提交事务，需要执行SqlSession的commit()方法。

5、释放资源，关闭SqlSession

# 4. mybatis开发dao的方法

1.原始dao 的方法

* 需要程序员编写dao接口和实现类
* 需要在dao实现类中注入一个SqlSessionFactory工厂

  2.mapper代理开发方法（建议使用）

只需要程序员编写mapper接口（就是dao接口）。
程序员在编写mapper.xml(映射文件)和mapper.java需要遵循一个开发规范：

* mapper.xml中namespace就是mapper.java的类全路径。
* mapper.xml中statement的id和mapper.java中方法名一致。
* mapper.xml中statement的parameterType指定输入参数的类型和mapper.java的方法输入参数类型一致
* mapper.xml中statement的resultType指定输出结果的类型和mapper.java的方法返回值类型一致。

SqlMapConfig.xml配置文件：可以配置properties属性、别名、mapper加载。

# 5. 输入映射和输出映射

* 输入映射：

  * parameterType：指定输入参数类型可以简单类型、pojo、hashmap。
  * 对于综合查询，建议parameterType使用包装的pojo，有利于系统扩展。
* 输出映射：
  		- resultType：查询到的列名和resultType指定的pojo的属性名一致，才能映射成功。
  		- reusltMap：可以通过resultMap 完成一些高级映射。如果查询到的列名和映射的pojo的属性名不一致时，通过resultMap设置列名和属性名之间的对应关系（映射关系）。可以完成映射。
  			- 高级映射：
  				将关联查询的列映射到一个pojo属性中。（一对一）
  				将关联查询的列映射到一个List<pojo>中。（一对多）

# 6. 动态sql

* 动态sql：（重点）

  * if判断（掌握）
  * where
  * foreach
  * sql片段（掌握）