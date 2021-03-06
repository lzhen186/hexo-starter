---
title: SpringBoot 04.常用web开发数据库框架 4.14.整合Spring data mongodb操作数据
date: 2022-07-03T02:38:20.458Z
tags: [springboot]
---


# 一、mongodb简介

`MongoDB`（来自于英文单词“Humongous”，中文含义为“庞大”）是可以应用于各种规模的企业、各个行业以及各类应用程序的开源数据库。作为一个适用于敏捷开发的数据库，`MongoDB`的数据模式可以随着应用程序的发展而灵活地更新。与此同时，它也为开发人员 提供了传统数据库的功能：二级索引，完整的查询系统以及严格一致性等等。 `MongoDB`能够使企业更加具有敏捷性和可扩展性，各种规模的企业都可以通过使用MongoDB来创建新的应用，提高与客户之间的工作效率，加快产品上市时间，以及降低企业成本。

**`MongoDB`是专为可扩展性，高性能和高可用性而设计的数据库。** 它可以从单服务器部署扩展到大型、复杂的多数据中心架构。利用内存计算的优势，`MongoDB`能够提供高性能的数据读写操作。 `MongoDB`的本地复制和自动故障转移功能使您的应用程序具有企业级的可靠性和操作灵活性。

简单来说，`MongoDB`是一个基于**分布式文件存储的数据库**，它是一个介于关系数据库和非关系数据库之间的产品，其主要目标是在键/值存储方式（提供了高性能和高度伸缩性）和传统的RDBMS系统（具有丰富的功能）之间架起一座桥梁，它集两者的优势于一身。

`MongoDB`支持的数据结构非常松散，是类似`json`的**bson格式**，因此可以存储比较复杂的数据类型，也因为他的存储格式也使得它所存储的数据在Nodejs程序应用中使用非常流畅。

**传统的关系数据库一般由数据库（database）、表（table）、记录（record）三个层次概念组成，MongoDB是由数据库（database）、集合（collection）、文档对象（document）三个层次组成。MongoDB对于关系型数据库里的表，但是集合中没有列、行和关系概念，这体现了模式自由的特点。**

`MongoDB`中的**一条记录就是一个文档**，是一个数据结构，由字段和值对组成。MongoDB文档与JSON对象类似。字段的值有可能包括其它文档、数组以及文档数组。MongoDB支持OS X、Linux及Windows等操作系统，并提供了Python，PHP，Ruby，Java及C++语言的驱动程序，社区中也提供了对Erlang及.NET等平台的驱动程序。

# 二、集成spring data mongodb

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

application.yml配置连接

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://root:123456@localhost:27017/testdb
```

注意，这里填写格式

```
    单机模式：mongodb://name:pwd@ip:port/database
    集群模式：mongodb://name:pwd@ip1:port1,ip2:port2/database
```

在项目入口启动类上面加一个注解。开启Mongodb审计功能.

```java
@EnableMongoAuditing
```

# 三、实现CURD

创建实体

```java
@Document(collection="article")//集合名
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class Article  implements Serializable {

    private static final long serialVersionUID = -8985545025018238754L;

    @Id
    private String id;

    @Indexed
    private String author;
    private String title;
    @Field("msgContent")
    private String content;

    @CreatedDate
    private Date createTime;
    private List<Reader> reader;


}
```

这里注意：

1. 一定要实现Serializable 接口，否则在序列化的时候会报错。
2. @Document(collection=“article”) 表示：操作的集合为:`article`。
3. 另外，针对`@CreatedDate`注解，也和之前的`jpa`用法一样，创建时会自动赋值，需要在启动类中添加`@EnableMongoAuditing`注解使其生效！
4. 可使用`@Field`注解，可指定存储的键值名称，默认就是类字段名。如设置`@Field("msgContent")`后，效果如下：
   ![img](https://box.kancloud.cn/67690650ce4b515eda7e167e081c6ccc_734x220.png)
5. `@Id`主键，不可重复，自带索引，可以在定义的列名上标注，需要自己生成并维护不重复的约束。如果自己不设置@Id主键，mongo会自动生成一个唯一主键，并且插入时效率远高于自己设置主键。
6. `@Indexed`声明该字段需要加索引，加索引后以该字段为条件检索将大大提高速度。
   唯一索引的话是@Indexed(unique = true)。

数据库操作的Dao

```java
public interface ArticleDao extends MongoRepository<Article,String> {
        //支持关键字查询，和JPA的用法一样
        Article findByAuthor(String author);
}
```

还可以使用使用mongoTemplate来完成数据操作，如:

```java
        @Autowired
        MongoTemplate mongoTemplate;

        //使用 save和insert都可以进行插入
        //区别：当存在"_id"时
        //insert 插入已经存在的id时 会异常
        //save 则会进行更新
        //简单来说 save 就是不存在插入 存在更新
        mongoTemplate.insert(msg);
        mongoTemplate.save(msg);


        //查找 article根据Criteria 改造查询条件
        Query query = new Query(Criteria.where("author").is("zimug"));
        Article article = mongoTemplate.find(query, Article.class);
   
```

更多用法参考:https://docs.spring.io/spring-data/mongodb/docs/2.1.9.RELEASE/reference/html/