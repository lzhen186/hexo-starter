---
title: "SpringBoot 11.redis缓存与session共享 11.3.使用redisTemplate操作数据  "
date: 2022-07-03T08:57:27.113Z
tags: [springboot]
---
# 一、spring-data-redis简介

Spring Boot 提供了对 Redis 集成的组件包：spring-boot-starter-data-redis，它依赖于 spring-data-redis 和 lettuce。Spring Boot 1.0 默认使用的是 Jedis 客户端，2.0 替换成了 Lettuce，但如果你从 Spring Boot 1.5.X 切换过来，几乎感受不大差异，这是因为 spring-boot-starter-data-redis 为我们隔离了其中的差异性。

1. Lettuce：是一个可伸缩线程安全的 Redis 客户端，多个线程可以共享同一个 RedisConnection，它利用优秀 Netty NIO 框架来高效地管理多个连接。
2. Spring Data：是 Spring 框架中的一个主要项目，目的是为了简化构建基于 Spring 框架应用的数据访问，包括非关系数据库、Map-Reduce 框架、云数据服务等，另外也包含对关系数据库的访问支持。
3. Spring Data Redis：是 Spring Data 项目中的一个主要模块，实现了对 Redis 客户端 API 的高度封装，使对 Redis 的操作更加便捷。

# 二、整合spring data redis

引入依赖包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

引入 commons-pool2 是因为 Lettuce 需要使用 commons-pool2 创建 Redis 连接池。

application 配置

```yaml
spring:
  redis:
    database: 0 # Redis 数据库索引（默认为 0）
    host: 192.168.161.3 # Redis 服务器地址
    port: 6379 # Redis 服务器连接端口
    password: 123456 # Redis 服务器连接密码（默认为空）
    lettuce:
      pool:
        max-active: 8 # 连接池最大连接数（使用负值表示没有限制） 默认 8
        max-wait: -1 # 连接池最大阻塞等待时间（使用负值表示没有限制） 默认 -1
        max-idle: 8 # 连接池中的最大空闲连接 默认 8
        min-idle: 0 # 连接池中的最小空闲连接 默认 0
```

# 三、redis模板封装类

### redis模板封装类

```java
redisTemplate.opsForValue();//操作字符串
redisTemplate.opsForHash();//操作 hash
redisTemplate.opsForList();//操作 list
redisTemplate.opsForSet();//操作 set
redisTemplate.opsForZSet();//操作有序 set
```

StringRedisTemplate 与 RedisTemplate 的封装的 Reids 操作要比我们第二节讲的自己调用 Jedis 的 API 的方式更优雅了一步。

### redis模板封装类的两种注入方式

```java
public class Example {

  // 注入RedisTemplate，更通用
  @Autowired
  private RedisTemplate<String, String> template;

  // 如：注入RedisTemplate的子类ListOperations，只能操作List
  @Resource
  private ListOperations<String, String> listOps;

}
```

而 StringRedisTemplate 与 RedisTemplate 对应 API 详细请查看此文档 [Spring Data Redis 官方操作手册](https://docs.spring.io/spring-data/redis/docs/2.0.2.RELEASE/reference/html/#redis:template) ，这里不是本节的重点介绍对象了，不再赘述。

# 四、代码测试

### 准备数据对象

```java
@Data
public class Person implements Serializable {

    private static final long serialVersionUID = -8985545025228238754L;

    String id;
    String firstname;
    String lastname;
    Address address;

    public Person() {
    }

    public Person(String firstname, String lastname) {
        this.firstname = firstname;
        this.lastname = lastname;
    }
}
```

```java
@Data
public class Address implements Serializable {

    private static final long serialVersionUID = -8985545025228238771L;

    String city;
    String country;

    public Address() {
    }

    public Address(String city, String country) {
        this.city = city;
        this.country = country;
    }
}
```



### 使用redisTemplate

```java
@SpringBootTest
public class RedisConfigTest {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Autowired
    private RedisTemplate redisTemplate;

    @Resource(name = "redisTemplate")
    private ValueOperations<String,Object> valueOperations;

    @Resource(name = "redisTemplate")
    private HashOperations<String, String, Object> hashOperations;

    @Resource(name = "redisTemplate")
    private ListOperations<String, Object> listOperations;

    @Resource(name = "redisTemplate")
    private SetOperations<String, Object> setOperations;

    @Resource(name = "redisTemplate")
    private ZSetOperations<String, Object> zSetOperations;

    @Test
    public void testValueObj(){
        Person person = new Person("kobe","bryant");
        person.setAddress(new Address("武汉","中国"));
        //10秒后数据消失
        valueOperations.set("player:1",person,20, TimeUnit.SECONDS);

        Person getBack = (Person) valueOperations.get("player:1");
        System.out.println(getBack);
    }

    @Test
    public void testSetOperation() throws Exception{
        Person person = new Person("kobe","bryant");
        Person person2 = new Person("curry","stephen");
        setOperations.add("playerset",person,person2);
        Set<Object> result = setOperations.members("playerset");
        System.out.println(result);
    }

    @Test
    public void HashOperations() throws Exception{
        Person person = new Person("kobe","bryant");
        hashOperations.put("hash:player","firstname",person.getFirstname());
        hashOperations.put("hash:player","lastname",person.getLastname());
        hashOperations.put("hash:player","address",person.getAddress());
        System.out.println(hashOperations.get("hash:player","firstname"));
    }

    @Test
    public void  ListOperations() throws Exception{

        listOperations.leftPush("list:player",new Person("kobe","bryant"));
        listOperations.leftPush("list:player",new Person("Jordan","Mikel"));
        listOperations.leftPush("list:player",new Person("curry","stephen"));

        System.out.println(listOperations.leftPop("list:player"));
    }
}
```

# 五、解决key-value乱码问题

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200429152232.png)

```java
@Configuration
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

        //重点在这四行代码
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);

        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
}
```

乱码问题的症结在于对象的序列化问题，RedisTemplate默认使用的是JdkSerializationRedisSerializer，StringRedisTemplate默认使用的是StringRedisSerializer。

Spring Data JPA为我们提供了下面的Serializer：GenericToStringSerializer、Jackson2JsonRedisSerializer、JacksonJsonRedisSerializer、JdkSerializationRedisSerializer、OxmSerializer、StringRedisSerializer。

序列化方式对比：

- JdkSerializationRedisSerializer: 使用JDK提供的序列化功能。 优点是反序列化时不需要提供类型信息(class)，但缺点是需要实现Serializable接口，还有序列化后的结果非常庞大，是JSON格式的5倍左右，这样就会消耗redis服务器的大量内存。而且是以二进制形式保存，自然人无法理解。
- Jackson2JsonRedisSerializer： 使用Jackson库将对象序列化为JSON字符串。优点是速度快，序列化后的字符串短小精悍，不需要实现Serializable接口。似乎没啥缺点。
- StringRedisSerializer序列化之后的结果，自然人也是可以理解，但是value只能是String类型，不能是Object。