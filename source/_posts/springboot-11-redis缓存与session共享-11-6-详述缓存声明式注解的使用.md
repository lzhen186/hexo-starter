---
title: SpringBoot 11.redis缓存与session共享 11.6.详述缓存声明式注解的使用
date: 2022-07-03T09:11:33.041Z
tags: [springboot]
---
# 一、缓存注解说明：

`@Cacheable` 通常应用到读取数据的方法上，如查找方法：先从缓存中读取，如果没有再调用方法获取数据，然后把数据查询结果添加到缓存中。如果缓存中查找到数据，被注解的方法将不会执行。

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200429154434.png)

`@CachePut`通常应用于保存和修改方法配置，能够根据方法的请求参数对其结果进行缓存，和 @Cacheable 不同的是，它每次都会触发被注解方法的调用。

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200429154505.png)

`@CachEvict` 通常应用于删除方法配置，能够根据一定的条件对缓存进行清空。可以清除一条或多条缓存。

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200429154630.png)

Caching为组合注解（非常有用），请看下文。

# 二、缓存注解常用策略

查询分两种：一种是查回来一个集合，一种是查一个对象。
那么，我们随便增删改一个对象，都有可能导致集合缓存与数据库数据不一致。
通常情况下，我们该怎么做才能做到一致性？**只要发生删改查，就把集合类缓存销毁**
**对于查询方法：**
`@Cacheable(value=“obj”)` 或 `@Cacheable(value=“objList”)`
**对于修改和新增方法：**

```java
@Caching(evict = {@CacheEvict(cacheNames = "objList",allEntries = true)},
                  put={@CachePut(cacheNames = "obj",key = "#id")})
```

**对于删除方法：**

```java
@Caching(evict = {@CacheEvict(cacheNames = "objList",allEntries = true),
                  @CacheEvict(cacheNames = "obj",key = "#id")})
```

> 在实际的生产环境中，没有一定之规，哪种注解必须用在哪种方法上，`@CachEvict` 注解通常也用于更新方法上。数据的缓存策略，要根据资源的使用方式，做出合理的缓存策略规划。保证缓存与业务数据库的数据一致性。并做好测试，没有一定之规。