---
title: SpringBoot 16.响应式框架webflux 16.3.webflux整合mongodb
date: 2022-07-04T00:37:29.982Z
tags: [springboot]
---
# 一、新增 POM 依赖与配置

在 pom.xml 配置新的依赖：

```xml
    <!-- Spring Boot 响应式 MongoDB 依赖 -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
    </dependency>
```

在 application.yml 配置下上面启动的 MongoDB 配置：

```yaml
spring: 
  data: 
    mongodb: 
      host: 192.168.161.3
      database: testdb   #数据库名
      port: 27017
      username: test      #用户名
      password: 123456
```

上面的配置在我实验过程中，有的时候回出现连接不上的问题，可以使用下面这个配置方式

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://test:123456@192.168.161.3:27017/testdb
```

# 二、代码小实验

使用上一节中用到的City实体类，加上@Id

```java
@Data
public class City {

    @Id
    private Long id;
    private Long provinceId;
    private String cityName;
    private String description;
}
```

注意： @Id 注解标记对应库表的主键或者唯一标识符。因为这个是我们的 PO，数据访问对象一一映射到数据存储。

MongoDB 数据访问层 CityRepository

```java
@Repository
public interface CityRepository extends ReactiveMongoRepository<City, Long> {
    Flux<City> findByCityName(String cityName);
}
```

CityRepository 接口只要继承 ReactiveMongoRepository 类即可，默认会提供很多实现，比如 CRUD 和列表查询参数相关的实现。ReactiveMongoRepository 接口默认实现了如下：

```java
    <S extends T> Mono<S> insert(S var1);

    <S extends T> Flux<S> insert(Iterable<S> var1);

    <S extends T> Flux<S> insert(Publisher<S> var1);

    <S extends T> Flux<S> findAll(Example<S> var1);

    <S extends T> Flux<S> findAll(Example<S> var1, Sort var2);
```

如图，ReactiveMongoRepository 的继承了ReactiveSortingRepository、ReactiveCrudRepository 实现了更多的常用的接口。
**支持关键字推断**

| 关键字  | 方法命名             |
| :------ | :------------------- |
| And     | findByNameAndPwd     |
| Or      | findByNameOrSex      |
| Id      | findById             |
| Between | findByIdBetween      |
| Like    | findByNameLike       |
| NotLike | findByNameNotLike    |
| OrderBy | findByIdOrderByXDesc |
| Not     | findByNameNot        |

常用案例，代码如下：

```java
    Flux<Person> findByLastname(String lastname);

    @Query("{ 'firstname': ?0, 'lastname': ?1}")
    Mono<Person> findByFirstnameAndLastname(String firstname, String lastname);

    // Accept parameter inside a reactive type for deferred execution
    Flux<Person> findByLastname(Mono<String> lastname);

    Mono<Person> findByFirstnameAndLastname(Mono<String> firstname, String lastname);
```

# 三、处理器类 Handler 和控制器类 Controller

```java
@Component
public class CityHandler {

    private final CityRepository cityRepository;

    @Resource
    private MongoTemplate mongoTemplate;

    @Autowired
    public CityHandler(CityRepository cityRepository) {
        this.cityRepository = cityRepository;
    }


    public Mono<City> save(City city) {
        return cityRepository.save(city);
    }

    public Mono<City> findCityById(Long id) {
        return cityRepository.findById(id);
    }

    public Flux<City> findAllCity() {

        return cityRepository.findAll();
    }

    public Flux<City> search(String cityName) {

        return cityRepository.findByCityName(cityName);
    }

    public Mono<City> modifyCity(City city) {

        return cityRepository.save(city);
    }

    public Mono<Long> deleteCity(Long id) {
        // 使用mongoTemplate来做删除,直接使用提供的删除方法不行
        Query query = Query.query(Criteria.where("id").is(id));
        mongoTemplate.remove(query, City.class);

        //cityRepository.deleteById(id);  这个方法无法删除数据
        return Mono.create(cityMonoSink -> cityMonoSink.success(id));
    }
}
```

在使用deleteById方法时候出现了问题，最好还是用mongoTemplate注入方法解决的。我感觉这个就是异步导致的bug，webflux可能还是不够稳定，问题较多。
不要对 Mono、Flux 陌生，把它当成对象即可。继续修改控制器类 Controller，代码如下：

```java
@RestController
@RequestMapping(value = "/citys")
public class CityWebFluxController {

    @Resource
    private CityHandler cityHandler;

    @GetMapping(value = "/{id}")
    public Mono<City> findCityById(@PathVariable("id") Long id) {
        return cityHandler.findCityById(id);
    }

    @GetMapping()
    public Flux<City> findAllCity() {
        return cityHandler.findAllCity();
    }

    @GetMapping(value = "/search/city")
    public Flux<City> search(@RequestParam("cityName") String cityName) {
        return cityHandler.search(cityName);
    }

    @PostMapping()
    public Mono<City> saveCity(@RequestBody City city) {
        return cityHandler.save(city);
    }

    @PutMapping()
    public Mono<City> modifyCity(@RequestBody City city) {
        return cityHandler.modifyCity(city);
    }

    @DeleteMapping(value = "/{id}")
    public Mono<Long> deleteCity(@PathVariable("id") Long id) {
        return cityHandler.deleteCity(id);
    }
}
```
