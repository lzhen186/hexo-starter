---
title: SpringBoot 04.常用web开发数据库框架 4.7.Spring data JPA的多数据源实现
date: 2022-07-03T01:59:02.141Z
tags: [springboot]
---
# 一、主流的多数据源支持方式

1. 将数据源对象作为参数，传递到调用方法内部，这种方式增加额外的编码。
2. 将Repository操作接口分包存放，Spring扫描不同的包，自动注入不同的数据源。这种方式实现简单，也是一种“约定大于配置”思想的典型应用。本文将以这种方式实现JPA的多数据源支持
3. 使用Spring AOP面向切面编程，然后在持久层接口方法上面加注解，不同的注解使用表示使用不同的数据源。在此暂不做介绍。

# 二、修改`application.yml`配置多数据源

```java
spring:
  datasource:
    primary:
        jdbc-url: jdbc:mysql://localhost:3306/testdb?useUnicode=true&characterEncoding=utf-8
        username: root
        password: 123456
        driver-class-name: com.mysql.jdbc.Driver
    secondary:
        jdbc-url: jdbc:mysql://localhost:3306/testdb?useUnicode=true&characterEncoding=utf-8
        username: root
        password: 123456
        driver-class-name: com.mysql.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update
    database: mysql
    show-sql: true
```

# 三、 两组数据持久化接口及实体类,放到不同的package里面

1. 将4.6章节中使用到的Article.java及ArticleRepository.java放到`club.krislin.bootlaunch.jpa.testdb`下面
2. 然后写另外一套操作接口放到`club.krislin.bootlaunch.jpa.testdb2`下面，如下：

```java
@Data
@Entity
public class Message {
    
    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String content;

}
public interface MessageRepository extends JpaRepository<Message,Long> {

}
```

# 四、多数据源支持

数据源`DataSource`的Bean对象创建并注入Spring上下文，分别对应`application.yml`里面的两套数据源配置

```java
@Configuration
public class JPADataSourceConfig {

    @Primary
    @Bean(name = "primaryDataSource")
    @Qualifier("primaryDataSource")
    @ConfigurationProperties(prefix="spring.datasource.primary")    //结合application.yml的配置
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

  

    @Bean(name = "secondaryDataSource")
    @Qualifier("secondaryDataSource")
    @ConfigurationProperties(prefix="spring.datasource.secondary")   //结合application.yml的配置
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

配置实体扫描以及事务管理,注意看@Primary和带注释的地方

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        entityManagerFactoryRef="entityManagerFactoryPrimary",
        transactionManagerRef="transactionManagerPrimary",
        basePackages= { "club.krislin.bootlaunch.jpa.testdb" }) //设置Repository所在位置
public class JPAPrimaryConfig {

 
    @Resource
    @Qualifier("primaryDataSource")
    private DataSource primaryDataSource;        //primary数据源注入

    @Primary
    @Bean(name = "entityManagerPrimary")        //primary实体管理器
    public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
        return entityManagerFactoryPrimary(builder).getObject().createEntityManager();
    }

  

    @Primary
    @Bean(name = "entityManagerFactoryPrimary")    //primary实体工厂
    public LocalContainerEntityManagerFactoryBean entityManagerFactoryPrimary (EntityManagerFactoryBuilder builder) {

        return builder
                .dataSource(primaryDataSource)
                .properties(getVendorProperties())
                .packages("club.krislin.bootlaunch.jpa.testdb")     //设置实体类所在位置
                .persistenceUnit("primaryPersistenceUnit")
                .build();
    }

  

    @Resource
    private JpaProperties jpaProperties;

    private Map getVendorProperties() {
        return jpaProperties.getHibernateProperties(new HibernateSettings());
    }

  

    @Primary
    @Bean(name = "transactionManagerPrimary")         //primary事务管理器
    public PlatformTransactionManager transactionManagerPrimary(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(entityManagerFactoryPrimary(builder).getObject());
    }
}
```

上面的代码将扫描`club.krislin.bootlaunch.jpa.testdb`下面的实体对象和Repository，并使用primary数据源。仿造这段代码再写一套`club.krislin.bootlaunch.jpa.testdb2`的配置使用secondary数据源。代码如下：

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        entityManagerFactoryRef="entityManagerFactorySecondary",
        transactionManagerRef="transactionManagerSecondary",
        basePackages= { "club.krislin.bootlaunch.jpa.testdb2" }) //设置Repository所在位置
public class JPASecondaryConfig {

    @Resource
    @Qualifier("secondaryDataSource")
    private DataSource secondaryDataSource;

    @Bean(name = "entityManagerSecondary")
    public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
        return entityManagerFactorySecondary(builder).getObject().createEntityManager();

    }

  

    @Bean(name = "entityManagerFactorySecondary")
    public LocalContainerEntityManagerFactoryBean entityManagerFactorySecondary (EntityManagerFactoryBuilder builder) {

        return builder
                .dataSource(secondaryDataSource)
                .properties(getVendorProperties())
                .packages("club.krislin.bootlaunch.jpa.testdb2") //设置实体类所在位置
                .persistenceUnit("secondaryPersistenceUnit")
                .build();
    }

    @Resource
    private JpaProperties jpaProperties;

    private Map getVendorProperties() {
        return jpaProperties.getHibernateProperties(new HibernateSettings());
    }

    @Bean(name = "transactionManagerSecondary")
    PlatformTransactionManager transactionManagerSecondary(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(entityManagerFactorySecondary(builder).getObject());

    }

}
```

# 五、测试

```java
//先构造一个Article对象article，这个操作针对testdb
articleRepository.save(article);
//在构造一个Message对象message，这个操作针对testdb2
messageRepository.save(message);
```

如果article数据能正确插入`testdb`的`article`表，message数据能正确的插入testdb2的message表，则JPA的多数据源实现正确。