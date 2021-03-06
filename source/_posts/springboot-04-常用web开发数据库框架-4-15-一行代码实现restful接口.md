---
title: SpringBoot 04.常用web开发数据库框架 4.15.一行代码实现RESTFul接口
date: 2022-07-03T02:41:37.125Z
tags: [springboot]
---
# 一、介绍spring data rest

Spring Data REST是基于Spring Data的repository之上，可以把 repository **自动**输出为REST资源，目前支持Spring Data JPA、Spring Data MongoDB、Spring Data Neo4j、Spring Data GemFire、Spring Data Cassandra的 repository **自动**转换成REST服务。注意是**自动**。

# 二、实现rest接口的最快方式

```xml
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-data-rest</artifactId> 
</dependency>
```

```java
@RepositoryRestResource(collectionResourceRel = "article",path="articles")
public interface ArticleDao extends MongoRepository<Article, String> {

}
```



参数：

1. collectionResourceRel 对应的是mongodb数据库资源文档的名称
2. path是Rest接口资源的基础路径
   如：GET /articles/{id}
   DELETE /articles/{id}

就简单的这样一个实现，Spring Data Rest就可以基于article资源，生成一套GET、PUT、POST、DELETE的增删改查的REST接口。