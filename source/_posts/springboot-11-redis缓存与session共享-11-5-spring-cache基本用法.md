---
title: SpringBoot 11.redis缓存与session共享 11.5.spring cache基本用法
date: 2022-07-03T09:07:40.330Z
tags: [springboot]
---
# 一、为什么要做缓存

- 提升性能

  绝大多数情况下，select 是出现性能问题最大的地方。一方面，select 会有很多像 join、group、order、like 等这样丰富的语义，而这些语义是非常耗性能的；另一方面，大多 数应用都是读多写少，所以加剧了慢查询的问题。

  分布式系统中远程调用也会耗很多性能，因为有网络开销，会导致整体的响应时间下降。为了挽救这样的性能开销，在业务允许的情况（不需要太实时的数据）下，使用缓存是非常必要的事情。

- 缓解数据库压力

  当用户请求增多时，数据库的压力将大大增加，通过缓存能够大大降低数据库的压力。

# 二、常用缓存操作流程

- **失效**：应用程序先从 cache 取数据，没有得到，则从数据库中取数据，成功后，放到缓存中。
- **命中**：应用程序从 cache 中取数据，取到后返回。
- **更新**：先把数据存到数据库中，成功后，再让缓存**失效或更新**。缓存操作失败，数据库事务回滚。
- **删除**: 先从数据库里面删掉，再从缓存里面删掉。缓存操作失败，数据库事务回滚。

# 三、spring cache

第一步：pom.xml 添加 Spring Boot 的 jar 依赖：

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

第二步：添加入口启动类 `@EnableCaching`注解开启 Caching，实例如下。

```java
@EnableCaching
```

# 四、在`ArticleRestController`类上实现一个简单的例子

缓存对象有几个非常需要注意的点：

1. 必须实现无参的构造函数
2. 需要实现Serializable 接口和定义`serialVersionUID`（因为缓存需要使用JDK的方式序列化和反序列化），这种方法不好。因为我们要修改所有的实体类，实现Serializable接口。我们完全可以修改为使用JSON序列化与反序列化的方式，可读性更强，体积更小，速度更快。

```java
@Cacheable(value="article")
@GetMapping( "/article/{id}")
public @ResponseBody  AjaxResponse getArticle(@PathVariable Long id) {
```

第一次访问走数据库，第二次访问就走缓存了，可以下断点试一下。

# 五、如何改变缓存的序列化方式

```java
@Configuration
public class RedisConfig {
   //这个函数是上一节的内容
   @Bean
   public RedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory) {
       RedisTemplate redisTemplate = new RedisTemplate();
       redisTemplate.setConnectionFactory(redisConnectionFactory);
       Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

       ObjectMapper objectMapper = new ObjectMapper();
       objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
       objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

       jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

       //重点在这四行代码
       redisTemplate.setKeySerializer(new StringRedisSerializer());
       redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
       redisTemplate.setHashKeySerializer(new StringRedisSerializer());
       redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);

       redisTemplate.afterPropertiesSet();
       return redisTemplate;
   }

    //本节的重点配置，让缓存的序列化方式使用redisTemplate.getValueSerializer()
   @Bean
   public RedisCacheManager redisCacheManager(RedisTemplate redisTemplate) {
       RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(redisTemplate.getConnectionFactory());
       RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
               .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(redisTemplate.getValueSerializer()));
       return new RedisCacheManager(redisCacheWriter, redisCacheConfiguration);
   }
}
```