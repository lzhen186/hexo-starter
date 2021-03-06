---
title: MyBatis 05.配置文件
date: 2022-07-01T05:17:39.298Z
tags: [mybatis]
---
# 1. SqlMapConfig.xml中配置的内容和顺序

- properties（属性）
- settings（全局配置参数）
- **typeAliases（类型别名）**
- typeHandlers（类型处理器）
- *objectFactory（对象工厂）*
- *plugins（插件）*
- environments（环境集合属性对象）
  - environment（环境子属性对象）
     - transactionManager（事务管理）
     - dataSource（数据源）
- **mappers（映射器）**


(注：粗体是重点，斜体不常用)


# 2. properties（属性）
  * 将数据库连接的参数单独配置在，db.properties中，只需要在SqlMapConfig.xml中加载db.properties的属性值。在SqlMapConfig.xml中就不需要对数据库连接参数硬编码。
  * 好处：方便对参数进行统一管理，其它xml可以引用该db.properties
  * 特性： MyBatis 将按照下面的顺序来加载属性：
    * 在 properties 元素体内定义的属性首先被读取。
    * 然后会读取properties 元素中resource或 url 加载的属性，它会覆盖已读取的同名属性。
    * 最后读取parameterType传递的属性，它会覆盖已读取的同名属性。
  * 建议：
    * 不要在SqlMapConfig.xml的properties元素体内添加任何属性值，只将属性值定义在properties文件中。
    
    * 在properties文件中定义属性名要有一定的特殊性，如XXXXX.XXXXX.XXXX
    
      ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601130131.jpg)
    
      
    
      ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601130213.jpg)


# 3. settings（全局配置参数）
mybatis框架在运行时可以调整一些运行参数。比如：开启二级缓存、开启延迟加载。。全局参数将会影响mybatis的运行行为。具体如下：
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200601130256.jpg)

# 4. typeAliases（类型别名）(重点)
  * 单个定义
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601130401.jpg)
  * 批量定义（常用）
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601130434.jpg)
    这样在其他地方就可以使用，例如：
    ![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200601130558.jpg)
* mybatis默认支持的别名
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200601130711.jpg)

# 5. typeHandlers（类型处理器）
  * mybatis中通过typeHandlers完成jdbc类型和java类型的转换。通常情况下，mybatis提供的类型处理器满足日常需要，不需要自定义.
  * mybatis支持的类型处理器 
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601130804.jpg)

# 6. objectFactory（对象工厂）

# 7. plugins（插件）

# 8. environments（环境集合属性对象）
  * environment（环境子属性对象）
      * transactionManager（事务管理）
      * dataSource（数据源）

# 9. mappers（映射器）
  * 通过resource
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601131109.jpg)
  * 通过class
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601131140.jpg)
  * 通过package(推荐使用)
    ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200601131232.jpg)

