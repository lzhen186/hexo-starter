---
title: SpringBoot 11.redis缓存与session共享 11.7.个性化自定义缓存到期时间
date: 2022-07-03T09:15:00.052Z
tags: [springboot]
---
# 一、redis 缓存配置

在 application.yml指定 spring.cache.type=redis。

```properties
spring.cache.type=
spring.cache.redis.cache-null-values=true      # Allow caching null values.
spring.cache.redis.key-prefix=                        # Key prefix.
spring.cache.redis.time-to-live=                      # 缓存到期时间，默认不主动删除永远不到期
spring.cache.redis.use-key-prefix=true            # Whether to use the key prefix when writing to Redis.
```

# 二、自定义缓存到期时间

由于redis缓存设置的到期时间是统一的，没有办法根据缓存名称(value属性)分别设置缓存到期的时间，所以我们进行一个简单的改造。

```java
@Data
@Configuration
@Component
@ConfigurationProperties(prefix = "caching")
public class RedisConfig {

    @Bean
    public RedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate redisTemplate = new RedisTemplate();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        //序列化重点在这四行代码
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);

        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }



    //自定义redisCacheManager
    @Bean
    public RedisCacheManager redisCacheManager(RedisTemplate redisTemplate) {
        RedisCacheWriter redisCacheWriter = RedisCacheWriter.nonLockingRedisCacheWriter(redisTemplate.getConnectionFactory());

        RedisCacheManager redisCacheManager = new RedisCacheManager(redisCacheWriter,
                this.buildRedisCacheConfigurationWithTTL(redisTemplate,RedisCacheConfiguration.defaultCacheConfig().getTtl().getSeconds()),  //默认的redis缓存配置
                this.getRedisCacheConfigurationMap(redisTemplate)); //针对每一个cache做个性化缓存配置

        return  redisCacheManager;
    }

    //配置注入，key是缓存名称，value是缓存有效期
    private Map<String,Long> ttlmap;

    //根据ttlmap的属性装配结果，个性化RedisCacheConfiguration
    private Map<String, RedisCacheConfiguration> getRedisCacheConfigurationMap(RedisTemplate redisTemplate) {
        Map<String, RedisCacheConfiguration> redisCacheConfigurationMap = new HashMap<>();

        for(Map.Entry<String, Long> entry : ttlmap.entrySet()){
            String cacheName = entry.getKey();
            Long ttl = entry.getValue();
            redisCacheConfigurationMap.put(cacheName,this.buildRedisCacheConfigurationWithTTL(redisTemplate,ttl));
        }

        return redisCacheConfigurationMap;
    }


    //让缓存的序列化方式使用redisTemplate.getValueSerializer()，并为每一个缓存分别设置ttl
    private RedisCacheConfiguration buildRedisCacheConfigurationWithTTL(RedisTemplate redisTemplate,Long ttl){
        return  RedisCacheConfiguration.defaultCacheConfig()
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(redisTemplate.getValueSerializer()))
                .entryTtl(Duration.ofSeconds(ttl));
    }

}
```

# 三、自定义配置实现缓存失效时间个性化

```yaml
caching:
  ttlmap:
    article: 10
    articleAll: 20
```