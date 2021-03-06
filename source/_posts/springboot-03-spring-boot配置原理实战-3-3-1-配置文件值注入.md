---
title: SpringBoot 03.spring boot配置原理实战 3.3.1.配置文件值注入
date: 2022-07-03T00:21:40.791Z
tags: [springboot]
---
# Pet和Person类

## Pet.java

```java
package club.krislin.dao;

/**
 * @Package club.krislin.dao
 * @ClassName Pet
 * @Description TODO
 * @Date 20/5/23 13:10
 * @Author krislin
 * @Version V1.0
 */
public class Pet {
    private String name;
    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Pet{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

## Person.java

```java
package club.krislin.dao;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.List;
import java.util.Map;

/**
 * @Package club.krislin.dao
 * @ClassName Person
 * @Description TODO
 * @Date 20/5/23 13:14
 * @Author krislin
 * @Version V1.0
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;
    private Character gender;
    private Integer age;
    private boolean boss;
    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Pet pet;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Character getGender() {
        return gender;
    }

    public void setGender(Character gender) {
        this.gender = gender;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public boolean isBoss() {
        return boss;
    }

    public void setBoss(boolean boss) {
        this.boss = boss;
    }

    public Date getBirth() {
        return birth;
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    public Map<String, Object> getMaps() {
        return maps;
    }

    public void setMaps(Map<String, Object> maps) {
        this.maps = maps;
    }

    public List<Object> getLists() {
        return lists;
    }

    public void setLists(List<Object> lists) {
        this.lists = lists;
    }

    public Pet getPet() {
        return pet;
    }

    public void setPet(Pet pet) {
        this.pet = pet;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", gender=" + gender +
                ", age=" + age +
                ", boss=" + boss +
                ", birth=" + birth +
                ", maps=" + maps +
                ", lists=" + lists +
                ", pet=" + pet +
                '}';
    }
}
```



提示：

![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200523131654.png)

需要导入配置文件处理器，以后编写配置就有提示了

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

# **配置文件**

```yaml
person:
  name: 张三
  gender: 男
  age: 36
  boss: true
  birth: 1982/10/1
  maps: {k1: v1,k2: v2}
  lists:
    - apple
    - peach
    - banana
  pet:
    name: 小狗
    age: 12
```

# **测试**

```java
package club.krislin;

import club.krislin.dao.Person;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class HelloWorldApplicationTests {
    @Autowired
    private Person person;

    @Test
    void contextLoads() {
        System.out.println(person);
    }

}
```

# properties

上面yaml对应的properties配置文件写法

```properties
person.name=李四
person.age=34
person.birth=1986/09/12
person.boss=true
person.gender=女
person.lists=cat,dog,pig
person.maps.k1=v1
person.maps.k2=v2
person.pet.name="小黑"
person.pet.age=10
```

> 测试，发现中文会乱码，而且char类型还会抛出Failed to bind properties under 'person.gender' to java.lang.Character异常

## 中文乱码解决方法

在设置中找到`File Encodings`，将配置文件字符集改为`UTF-8`，并勾选：

-  `Transparent native-to-ascii conversion`

  ![](https://gitee.com/krislin_zhao/IMGcloud/raw/master/img/20200523133116.png)

> yaml和properties配置文件同时存在，properties配置文件的内容会覆盖yaml配置文件的内容