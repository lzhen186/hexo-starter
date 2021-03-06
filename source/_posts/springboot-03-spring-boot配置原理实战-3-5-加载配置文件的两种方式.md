---
title: SpringBoot 03.spring boot配置原理实战 3.5.加载配置文件的两种方式
date: 2022-07-03T00:31:09.748Z
tags: [springboot]
---
# 一、为什么要学加载外部配置文件

有一些老的项目的jar包并未主动的去与spring boot 融合，如果使用这些jar包就得使用他们自己的配置文件。如果我们的应用也需要这些配置，就不要同样的配置写两份了（一份放在`application.yml`，一份放在自定义的配置文件），这种还不如多个配置文件就好了。所以我们需要学习一下如何加载外部自定义配置文件。

# 二、方式一：使用`@PropertySource`加载自定义`yml`或`properties`文件

我们新建一个配置文件`family.yml`，把YAML数据结构放里面

```yaml
#    1. 一个家庭有爸爸、妈妈、孩子。
#    2. 这个家庭有一个名字（family-name）叫做“happy family”
#    3. 爸爸有名字(name)和年龄（age）两个属性
#    4. 妈妈有两个别名
#    5. 孩子除了名字(name)和年龄（age）两个属性，还有一个friends的集合
#    6. 每个friend有两个属性：hobby(爱好)和性别(sex)

family:
  family-name: "happy family"
  father:
    name: zimug
    age: 18
  mother:
    alias:
      - lovely
      - ailice
  child:
    name: zimug2
    age: 5
    friends:
      - hobby: football
        sex:  male
      - hobby: basketball
        sex: female
```

因为`@PropertySource`默认不支持读取YAML格式外部配置文件，所以我们继承`DefaultPropertySourceFactory` ，然后对它的`createPropertySource`进行一下修改

```java
public class MixPropertySourceFactory extends DefaultPropertySourceFactory {

  @Override
  public PropertySource<?> createPropertySource(String name, EncodedResource resource) throws IOException {
    String sourceName = name != null ? name : resource.getResource().getFilename();
    if (!resource.getResource().exists()) {
      return new PropertiesPropertySource(sourceName, new Properties());
    } else if (sourceName.endsWith(".yml") || sourceName.endsWith(".yaml")) {
      Properties propertiesFromYaml = loadYml(resource);
      return new PropertiesPropertySource(sourceName, propertiesFromYaml);
    } else {
      return super.createPropertySource(name, resource);
    }
  }

  private Properties loadYml(EncodedResource resource) throws IOException {
    YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();
    factory.setResources(resource.getResource());
    factory.afterPropertiesSet();
    return factory.getObject();
  }
}
```

然后基于上一节的代码，在Family类的上面加上如下注解即可。

```java
@PropertySource(value = {"classpath:family.yml"}, factory = MixPropertySourceFactory.class)
public class Family {
```

如果是读取`properties`配置文件，加`@PropertySource(value = {"classpath:family.properties"})`即可。不需要定义`MixPropertySourceFactory`。

# 三、方式二：使用`@ImportResource`加载Spring的xml配置文件

在没有注解的时代，spring的相关配置都是通过xml来完成的，如：beans.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="testBeanService" class="club.krislin.bootlaunch.service.TestBeanService"></bean>
</beans>
```

然后建一个空类，`club.krislin.bootlaunch.service.TestBeanService`。
测试用例:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class ImportResourceTests {

    @Autowired
    private ConfigurableApplicationContext  ioc;

    @Test
    public void testHelloService() {
        //测试Spring上下文环境中是否有testBeanService这样一个bean，有的话表示xml配置文件生效
        boolean testBeanService= ioc.containsBean("testBeanService");
        System.out.println(testBeanService);
    }
}
```

因为此时还没使用`@ImportResource`加载`beans.xml`，所以输出`false`.在启动类上加

```java
@ImportResource(locations = {"classpath:beans.xml"})
```

再试一下测试用例，输出:`true`

> 注意！这个注解是放在主入口函数的类上，而不是测试类上

SpringBoot推荐使用全注解的方式给容器中添加组件：

1. 配置类`@Configuration`---------->Spring配置文件
2. 使用`@Bean`给容器中添加组件

```java
/**
 *
 * Configuration：指明当前类是一个配置类；就是来替代之前的Spring配置文件
 */
@Configuration
public class BeanConfiguration {

    /**
     *相当于在配置文件中用<bean><bean/>标签添加组件
     * 将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
     */
    @Bean
    public Pet myPet() {
        Pet pet = new Pet();
        pet.setName("嘟嘟");
        pet.setAge(3);
        return pet;
    }
}
```

