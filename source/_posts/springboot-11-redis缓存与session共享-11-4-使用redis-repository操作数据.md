---
title: SpringBoot 11.redis缓存与session共享 11.4.使用Redis Repository操作数据
date: 2022-07-03T09:04:36.967Z
tags: [springboot]
---
# 一、整体操作hash对象

上一节我们操作hash对象的时候是一个属性一个属性设置的，那我们有没有办法将对象一次性hash入库呢？第一种方式我们可以使用`Jackson2HashMapper`

```java
@Resource(name="redisTemplate")
private HashOperations<String, String, Object> jacksonHashOperations;
private HashMapper<Object, String, Object> jackson2HashMapper = new Jackson2HashMapper(false);
@Test
public void testHashPutAll(){

    Person person = new Person("kobe","bryant");
    person.setId("1");
    person.setAddress(new Address("南京","中国"));
    //将对象以hash的形式放入数据库
    Map<String,Object> mappedHash = jackson2HashMapper.toHash(person);
    jacksonHashOperations.putAll("player" + person.getId(), mappedHash);

    //将对象从数据库取出来
    Map<String,Object> loadedHash = jacksonHashOperations.entries("player" + person.getId());
    Object map = jackson2HashMapper.fromHash(loadedHash);
    Person getback = new ObjectMapper().convertValue(map,Person.class);
    Assert.assertEquals(person.getFirstname(),getback.getFirstname());
}
```

# 二、第二种方法redis repository

```java
@RedisHash("people")
public class Person {
  @Id
  String id;
  
  //其他和上一节代码一样

}
public interface PersonRepository extends CrudRepository<Person, String> {
 // 继承CrudRepository，获取基本的CRUD操作
}
```

在项目入口方法上加上注解@EnableRedisRepositories

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class RedisRepositoryTest {

    @Autowired
    PersonRepository personRepository;

    @Test
    public void test(){

        Person rand = new Person("kris", "lin");
        rand.setAddress(new Address("杭州", "中国"));
        personRepository.save(rand);
        Optional<Person> op = personRepository.findById(rand.getId());
        Person p2 = op.get();
        personRepository.count();
        personRepository.delete(rand);

    }

}
```