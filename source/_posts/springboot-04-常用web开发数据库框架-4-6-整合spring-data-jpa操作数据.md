---
title: SpringBoot 04.常用web开发数据库框架 4.6.整合Spring Data JPA操作数据
date: 2022-07-03T01:47:23.424Z
tags: [springboot]
---
# 一、 Sping Data JPA 简介

**Spring Data JPA** 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套 JPA 应用框架，底层使用了 Hibernate 的 JPA 技术实现，可使开发者用极简的代码即可实现对数据的访问和操作。它提供了包括增删改查等在内的常用功能，且易于扩展！学习并使用 Spring Data JPA 可以极大提高开发效率！

# 二、 将Spring Data JPA集成到Spring Boot

第一步：引入maven依赖包，包括Spring Data JPA和Mysql的驱动

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
 <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

第二步：修改application.yml，配置好数据库连接和jpa的相关配置

```java
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/testdb?useUnicode=true&characterEncoding=utf-8
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update
    database: mysql
    show-sql: true
```

`spring.jpa.properties.hibernate.hbm2ddl.auto`是hibernate的配置属性，其主要作用是：自动根据实体类的定义创建、更新、验证数据库表结构。该参数的几种配置如下：

- `create`：每次加载hibernate时都会删除上一次的生成的表，然后根据你的model类再重新来生成新表，哪怕两次没有任何改变也要这样执行，这就是导致数据库表数据丢失的一个重要原因。
- `create-drop`：每次加载hibernate时根据model类生成表，但是sessionFactory一关闭,表就自动删除。
- `update`：最常用的属性，第一次加载hibernate时根据model类会自动建立起表的结构（前提是先建立好数据库），以后加载hibernate时根据model类自动更新表结构，即使表结构改变了但表中的行仍然存在不会删除以前的行。要注意的是当部署到服务器后，表结构是不会被马上建立起来的，是要等应用第一次运行起来后才会。
- `validate`：每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。

# 三、 基础核心用法

我们来实现一个简单的使用JPA操作数据库的例子。

## 3.1.实体Model类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity
@Table(name="article")
public class Article {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false,length = 32)
    private String author;

    @Column(nullable = false, unique = true,length = 32)
    private String title;

    @Column(length = 512)
    private String content;

    private Date createTime;
}
```

- @Entity 表示这个类是一个实体类，接受JPA控制管理，对应数据库中的一个表
- @Table 指定这个类对应数据库中的表名。如果这个类名和数据库表名符合驼峰及下划线规则，可以省略这个注解。如FlowType类名对应表名flow_type。
- @Id 指定这个字段为表的主键
- @GeneratedValue(strategy=GenerationType.IDENTITY) 指定主键的生成方式，一般主键为自增的话，就采用GenerationType.IDENTITY的生成方式
- @Column 注解针对一个字段，对应表中的一列。nullable = false表示数据库字段不能为空, unique = true表示数据库字段不能有重复值,length = 32表示数据库字段最大程度为32.

关于更多注解的详细用法，请参考：[# Hibernate Annotations 参考文档](https://docs.jboss.org/hibernate/annotations/3.4/reference/zh_cn/html_single/#entity)

## 3.2.数据操作接口

```java
public interface ArticleRepository extends JpaRepository<Article,Long> {
}
```

XxxRepository继承 JpaRepository<T,ID>为我们提供了各种针对单表的数据操作方法：增删改查。只要你不是完全英语小白，通过调用接口的方法名称就能知道方法是做什么操作的。

## 3.3.service层接口:

```java
public interface ArticleRestService {

     ArticleVO saveArticle(ArticleVO article);

     void deleteArticle(Long id);

     void updateArticle(ArticleVO article);

     ArticleVO getArticle(Long id);

     List<ArticleVO> getAll();
}
```

## 3.4.service层接口实现

```java
@Service
public class ArticleJPARestService implements  ArticleRestService  {

    //将JPA仓库对象注入
    @Resource
    private ArticleRepository articleRepository;

    @Resource
    private Mapper dozerMapper;

    public ArticleVO saveArticle( ArticleVO article) {

        Article articlePO = dozerMapper.map(article,Article.class);
        articleRepository.save(articlePO);    //保存一个对象到数据库，insert

        return  article;
    }

    @Override
    public void deleteArticle(Long id) {
        articleRepository.deleteById(id);   //根据id删除1条数据库记录
    }

    @Override
    public void updateArticle(ArticleVO article) {
        Article articlePO = dozerMapper.map(article,Article.class);
        articleRepository.save(articlePO);   //更新一个对象到数据库，仍然使用save方法
    }

    @Override
    public ArticleVO getArticle(Long id) {
        Optional<Article> article = articleRepository.findById(id);  //根据id查找一条数据
        return dozerMapper.map(article.get(),ArticleVO.class);
    }

    @Override
    public List<ArticleVO> getAll() {
        List<Article> articleLis = articleRepository.findAll();  //查询article表的所有数据
        return DozerUtils.mapList(articleLis,ArticleVO.class);
    }
}
```

注意：虽然新增和修改都是使用的save方法，但是完成的功能是不一样的。当保存的对象有主键id的时候，save方法会根据id更新记录；当保存的对象没有主键id的时候，save方法会向数据库里面insert一条记录。

# 四、关键字查询接口

除了上文中JpaRepository为我们提供的增删改查的方法。我们还可以自定义方法，非常简单。把下面的方法名放到ArticleRepository 里面，它就自动为我们实现了通过author字段查找article表的所有数据。也就是说，我们使用了find(查找)关键字，JPA就自动将方法名为我们解析成数据库操作，太智能了。

```java
    //注意这个方法的名称，jPA会根据方法名自动生成SQL执行
    Article findByAuthor(String author);
```

其他具体的关键字，使用方法和生产成 SQL 如下表所示

| Keyword           | Sample                                  | JPQL snippet                                                 |
| :---------------- | :-------------------------------------- | :----------------------------------------------------------- |
| And               | findByLastnameAndFirstname              | … where x.lastname = ?1 and x.firstname = ?2                 |
| Or                | findByLastnameOrFirstname               | … where x.lastname = ?1 or x.firstname = ?2                  |
| Is,Equals         | findByFirstnameIs,findByFirstnameEquals | … where x.firstname = ?1                                     |
| Between           | findByStartDateBetween                  | … where x.startDate between ?1 and ?2                        |
| LessThan          | findByAgeLessThan                       | … where x.age < ?1                                           |
| LessThanEqual     | findByAgeLessThanEqual                  | … where x.age ⇐ ?1                                           |
| GreaterThan       | findByAgeGreaterThan                    | … where x.age > ?1                                           |
| GreaterThanEqual  | findByAgeGreaterThanEqual               | … where x.age >= ?1                                          |
| After             | findByStartDateAfter                    | … where x.startDate > ?1                                     |
| Before            | findByStartDateBefore                   | … where x.startDate < ?1                                     |
| IsNull            | findByAgeIsNull                         | … where x.age is null                                        |
| IsNotNull,NotNull | findByAge(Is)NotNull                    | … where x.age not null                                       |
| Like              | findByFirstnameLike                     | … where x.firstname like ?1                                  |
| NotLike           | findByFirstnameNotLike                  | … where x.firstname not like ?1                              |
| StartingWith      | findByFirstnameStartingWith             | … where x.firstname like ?1 (parameter bound with appended %) |
| EndingWith        | findByFirstnameEndingWith               | … where x.firstname like ?1 (parameter bound with prepended %) |
| Containing        | findByFirstnameContaining               | … where x.firstname like ?1 (parameter bound wrapped in %)   |
| OrderBy           | findByAgeOrderByLastnameDesc            | … where x.age = ?1 order by x.lastname desc                  |
| Not               | findByLastnameNot                       | … where x.lastname <> ?1                                     |
| In                | findByAgeIn(Collection ages)            | … where x.age in ?1                                          |
| NotIn             | findByAgeNotIn(Collection age)          | … where x.age not in ?1                                      |
| TRUE              | findByActiveTrue()                      | … where x.active = true                                      |
| FALSE             | findByActiveFalse()                     | … where x.active = false                                     |
| IgnoreCase        | findByFirstnameIgnoreCase               | … where UPPER(x.firstame) = UPPER(?1)                        |

可以看到我们这里没有任何类SQL语句就完成了两个条件查询方法。这就是Spring-data-jpa的一大特性：**通过解析方法名创建查询**。针对单表的数据查询简单到令人发指，怎么可以这么简单，照这个趋势发展，程序员早晚失业。

# 五、测试关键字查询

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class JPAKeyWordTest {

    @Resource
    private ArticleRepository articleRepository;
    
    @Test
    public void userTest() {
        Article article = articleRepository.findByAuthor("krislin");
        System.out.println(article);
    }

}
```

# 五、其他

Spring-data-jpa的能力远不止本文提到的这些，由于本文主要以整合介绍为主，对于Spring-data-jpa的使用只是介绍了常见的使用方式。本教程作为spring boot系列教程，并不能将spring data jpa的方方面面讲到，本文只会去讲最重要的部分，如果想更加深入的学习Spring data jpa。**其实，本文已经介绍了JPA最常用的用法中的80%，笔者不建议使用Query、NamedQuery、Specification、QueryDSL等，如果你用这些东西，还不如自己写SQL。** 当然，JPA的深度用户，也许会不同意我的说法，那么请参考下方文档进行更深入的学习：

> 建议参考: [Spring Data JPA 官方文档](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#reference)
> [JPA像Mybatis一样写SQL](http://www.thxopen.com/java/2018/09/19/use-spring-data-jpa-extra-to-write-sql-like-mybatis.html)