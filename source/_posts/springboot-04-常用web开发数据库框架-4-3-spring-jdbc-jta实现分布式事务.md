---
title: SpringBoot 04.常用web开发数据库框架 4.3.Spring JDBC JTA实现分布式事务
date: 2022-07-03T01:25:18.808Z
tags: [springboot]
---
# 一、举例说明分布式事务

在上一节代码的的Service层做一下测试，人为制造一个被除数为0的异常。然后对该服务对应的Controller方法发送请求。（postman）

```java
@Resource
private JdbcTemplate primaryJdbcTemplate;
@Resource
private JdbcTemplate secondaryJdbcTemplate;

@Transactional
public Article saveArticle( Article article) {
    articleJDBCDAO.save(article,primaryJdbcTemplate);
    articleJDBCDAO.save(article,secondaryJdbcTemplate);
    int a = 2/0;    //人为制造一个被除数为0的异常
    return article;
}
```

secondaryJdbcTemplate的数据插入数据成功，primaryJdbcTemplate的数据插入数据失败。数据库事务不能跨连接, 当然也就不能跨数据源，更不能跨库。一旦出现跨连接的情况，也就成了分布式事务，事务就不能单纯依赖于数据库去处理。我们这一节的实现方式，是通过JTA来实现。

# 二、通过整合JTA实现分布式事务

> 分布式事务就是跨数据库连接的事务。一个事务的数据库操作，要么都成功，要么都失败回滚。后面我们专门做一个章节讲解事务与分布式事务。

## 2.1 引入maven依赖包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jta-atomikos</artifactId>
</dependency>
```

## 2.2 修改application配置文件

双数据源配置。删掉原有数据库连接配置.

```yaml
primarydb:
  uniqueResourceName: primary
  xaDataSourceClassName: com.mysql.jdbc.jdbc2.optional.MysqlXADataSource
  xaProperties:
    url: jdbc:mysql://localhost:3306/testdb?useUnicode=true&characterEncoding=utf-8&useSSL=false
    user: test
    password: 4rfv$RFV
  exclusiveConnectionMode: true
  minPoolSize: 3
  maxPoolSize: 10
  testQuery: SELECT 1 from dual #由于采用HikiriCP，用于检测数据库连接是否存活。

secondarydb:
  uniqueResourceName: secondary
  xaDataSourceClassName: com.mysql.jdbc.jdbc2.optional.MysqlXADataSource
  xaProperties:
    url: jdbc:mysql://localhost:3306/testdb2?useUnicode=true&characterEncoding=utf-8&useSSL=false
    user: test
    password: 4rfv$RFV
  exclusiveConnectionMode: true
  minPoolSize: 3
  maxPoolSize: 10
  testQuery: SELECT 1 from dual #由于采用HikiriCP，用于检测数据库连接是否存活。
```

MysqlXADataSource的解释:根据jdbc 4.0规范(12.2)：XA数据源生成能够在全局/分布式事务中使用的XA连接。如果需要跨多个数据库或JMS调用的事务，则可能需要此类连接。您可以在此处找到有关概念的明确说明：http://www.theserverside.com/discussions/thread.tss?thread_id=21385#95346

如果不使用分布式事务，则在声明驱动程序时无需指定xa-datasource-class。这个xa-datasource-class是专门为分布式事务准备的。

## 2.3 数据源配置

下面是数据源bean的配置，将上面配置文件中的属性加载到Spring Bean里面。也就是将配置参数应用到我们的双数据库数据源实例对象中。

```java
@Configuration
public class DataSourceConfig {
     //jta数据源primarydb
    @Bean(initMethod="init", destroyMethod="close", name="primaryDataSource")
    @Primary
    @ConfigurationProperties(prefix = "primarydb")
    public DataSource primaryDataSource() {
         //这里是关键，返回的是AtomikosDataSourceBean，所有的配置属性也都是注入到这个类里面
        return new AtomikosDataSourceBean();
    }

    //jta数据源secondarydb
    @Bean(initMethod="init", destroyMethod="close", name="secondaryDataSource")
    @ConfigurationProperties(prefix = "secondarydb")
    public DataSource secondaryDataSource()  {
        return new AtomikosDataSourceBean();
    }

    //primaryJdbcTemplate使用primaryDataSource数据源
    @Bean
    public JdbcTemplate primaryJdbcTemplate(
                    @Qualifier("primaryDataSource") DataSource primaryDataSource) {
        return new JdbcTemplate(primaryDataSource);
    }

    //secondaryJdbcTemplate使用secondaryDataSource数据源
    @Bean
    public JdbcTemplate secondaryJdbcTemplate(
                    @Qualifier("secondaryDataSource") DataSource secondaryDataSource) {
        return new JdbcTemplate(secondaryDataSource);
    }
}
```

## 2.4 事务管理器配置

负责协调多个JTA数据源实现事务机制。固定写法，不用纠结UserTransaction 、TransactionManager 、JtaTransactionManager 都是什么。或者说：都是帮我们实现JTA分布式事务的事务管理器。

```java
@Configuration
public class TransactionManagerConfig {

    @Bean
    public UserTransaction userTransaction() throws SystemException {
        UserTransactionImp userTransactionImp = new UserTransactionImp();
        userTransactionImp.setTransactionTimeout(10000);
        return userTransactionImp;
    }

    @Bean(name = "atomikosTransactionManager", initMethod = "init", destroyMethod = "close")
    public TransactionManager atomikosTransactionManager() throws Throwable {
        UserTransactionManager userTransactionManager = new UserTransactionManager();
        userTransactionManager.setForceShutdown(false);
        return userTransactionManager;
    }

    @Bean(name = "transactionManager")
    @DependsOn({ "userTransaction", "atomikosTransactionManager" })
    public PlatformTransactionManager transactionManager() throws Throwable {
        UserTransaction userTransaction = userTransaction();

        JtaTransactionManager manager = new JtaTransactionManager(userTransaction, atomikosTransactionManager());
        return manager;
    }
}
```

这时候，再测试一下文首中的测试用例：人为制造一个被除数为0的异常，异常抛出，两个数据库实例中的article表将都无法插入数据。符合事务的要求：正常情况数据操作都成功，异常情况数据操作都失败回滚。

# 三、JTA实现分布式事务的优缺点

优点： 能够支持分布式事务
缺点： 性能开销大，不适合用于高并发场景