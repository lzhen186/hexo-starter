---
title: SpringBoot 03.spring boot配置原理实战 3.3.2.配置文件值注入的两种方式
date: 2022-07-03T00:24:58.475Z
tags: [springboot]
---
# 一、使用@Value获取配置值

```java
@Data
@Component
public class Family {

    @Value("${family.family-name}")
    private String familyName;

}
```

# 二、使用@ConfigurationProperties获取配置值

下面是用于接收yml配置的实体java类，根据yml的嵌套结构，写出来对应的java类:

```java
@Data
@Component
@ConfigurationProperties(prefix = "family")
public class Family {

    //@Value("${family.family-name}")
    private String familyName;
    private Father father;
    private Mother mother;
    private Child child;

}
@Data
public class Father {
    private String name;
    private Integer age;
}
@Data
public class Mother {
    private String[] alias;
}
@Data
public class Child {
    private String name;
    private Integer age;
    private List<Friend> friends;
}
@Data
public class Friend {
    private String hobby;
    private String sex;
}
```

# 三、测试用例

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class CustomYamlTest {

    @Autowired
     Family family;

    @Test
    public void hello(){
        System.out.println(family.toString());
    }
}
```

测试结果:

```bash
Family(familyName=happy family, father=Father(name=zimug, age=18), 
mother=Mother(alias=[lovely, ailice]), child=Child(name=zimug2, age=5, 
friends=[Friend(hobby=football, sex=male), Friend(hobby=basketball, sex=female)]))
```

# 四、比较一下二者

|                          | @ConfigurationProperties | @Value             |
| :----------------------- | :----------------------- | :----------------- |
| 功能                     | 批量注入属性到java类     | 一个个属性指定注入 |
| 松散语法绑定             | 支持                     | 不支持             |
| SpEL                     | 不支持                   | 支持               |
| 复杂数据类型(对象、数组) | 支持                     | 不支持             |
| JSR303数据校验           | 支持                     | 不支持             |