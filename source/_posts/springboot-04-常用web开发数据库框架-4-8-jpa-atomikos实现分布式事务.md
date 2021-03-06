---
title: SpringBoot 04.常用web开发数据库框架 4.8.JPA+atomikos实现分布式事务
date: 2022-07-03T02:03:00.959Z
tags: [springboot]
---
# 一、整合JTA(atomikos)

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jta-atomikos</artifactId>
</dependency>
```

在上一章节的配置基础上，把jdbc-url改成url，然后在spring前缀下面加（注意这里是jta配置，而不是jpa配置，jta表示分布式事务）:

```yaml
  jta:
    atomikos:
      datasource:
        max-pool-size: 20
        borrow-connection-timeout: 60
      connectionfactory:
        max-pool-size: 20
        borrow-connection-timeout: 60
```

# 二、配置多数据源分布式事务

删掉上一节的数据源配置及事务配置代码，加入如下代码。以下为固定代码不需修改:

```java
public class AtomikosJtaPlatform extends AbstractJtaPlatform {

    private static final long serialVersionUID = 1L;

    static TransactionManager transactionManager;
    static UserTransaction transaction;

    @Override
    protected TransactionManager locateTransactionManager() {
        return transactionManager;
    }

    @Override
    protected UserTransaction locateUserTransaction() {
        return transaction;
    }
}
```

事务管理器的配置，以下除了设置JPA特性的部分，为固定代码不需修改:

```java
@Configuration
@ComponentScan
@EnableTransactionManagement
public class JPAAtomikosTransactionConfig {

    @Bean
    public PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }

    //设置JPA特性
    @Bean
    public JpaVendorAdapter jpaVendorAdapter() {
        HibernateJpaVendorAdapter hibernateJpaVendorAdapter = new HibernateJpaVendorAdapter();
        //显示sql
        hibernateJpaVendorAdapter.setShowSql(true);
        //自动生成/更新表
        hibernateJpaVendorAdapter.setGenerateDdl(true);
        //设置数据库类型
        hibernateJpaVendorAdapter.setDatabase(Database.MYSQL);
        return hibernateJpaVendorAdapter;
    }

    @Bean(name = "userTransaction")
    public UserTransaction userTransaction() throws Throwable {
        UserTransactionImp userTransactionImp = new UserTransactionImp();
        userTransactionImp.setTransactionTimeout(10000);
        return userTransactionImp;
    }

    @Bean(name = "atomikosTransactionManager", initMethod = "init", destroyMethod = "close")
    public TransactionManager atomikosTransactionManager() throws Throwable {
        UserTransactionManager userTransactionManager = new UserTransactionManager();
        userTransactionManager.setForceShutdown(false);
        AtomikosJtaPlatform.transactionManager = userTransactionManager;
        return userTransactionManager;
    }

    @Bean(name = "transactionManager")
    @DependsOn({"userTransaction", "atomikosTransactionManager"})
    public PlatformTransactionManager transactionManager() throws Throwable {
        UserTransaction userTransaction = userTransaction();
        AtomikosJtaPlatform.transaction = userTransaction;
        TransactionManager atomikosTransactionManager = atomikosTransactionManager();
        return new JtaTransactionManager(userTransaction, atomikosTransactionManager);
    }

}
```

设置第一个数据库的primary数据源及实体扫描管理(扫描testdb目录)，实体管理器、数据源都要加上primary，以示区分:

```java
@Configuration
@DependsOn("transactionManager")
@EnableJpaRepositories(basePackages = "club.krislin.bootlaunch.jpa.testdb",  //注意这里
        entityManagerFactoryRef = "primaryEntityManager",
        transactionManagerRef = "transactionManager")
public class JPAPrimaryConfig {
    @Autowired
    private JpaVendorAdapter jpaVendorAdapter;

    //primary
    @Primary
    @Bean(name = "primaryDataSourceProperties")
    @Qualifier("primaryDataSourceProperties")
    @ConfigurationProperties(prefix = "spring.datasource.primary")     //注意这里
    public DataSourceProperties primaryDataSourceProperties() {
        return new DataSourceProperties();
    }

    @Primary
    @Bean(name = "primaryDataSource", initMethod = "init", destroyMethod = "close")
    @ConfigurationProperties(prefix = "spring.datasource.primary")
    public DataSource primaryDataSource() throws SQLException {
        MysqlXADataSource mysqlXaDataSource = new MysqlXADataSource();
        mysqlXaDataSource.setUrl(primaryDataSourceProperties().getUrl());
        mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);
        mysqlXaDataSource.setPassword(primaryDataSourceProperties().getPassword());
        mysqlXaDataSource.setUser(primaryDataSourceProperties().getUsername());
        AtomikosDataSourceBean xaDataSource = new AtomikosDataSourceBean();
        xaDataSource.setXaDataSource(mysqlXaDataSource);
        xaDataSource.setUniqueResourceName("primary");
        xaDataSource.setBorrowConnectionTimeout(60);
        xaDataSource.setMaxPoolSize(20);
        return xaDataSource;
    }

    @Primary
    @Bean(name = "primaryEntityManager")
    @DependsOn("transactionManager")
    public LocalContainerEntityManagerFactoryBean primaryEntityManager() throws Throwable {

        HashMap<String, Object> properties = new HashMap<String, Object>();
        properties.put("hibernate.transaction.jta.platform", AtomikosJtaPlatform.class.getName());
        properties.put("javax.persistence.transactionType", "JTA");
        LocalContainerEntityManagerFactoryBean entityManager = new LocalContainerEntityManagerFactoryBean();
        entityManager.setJtaDataSource(primaryDataSource());
        entityManager.setJpaVendorAdapter(jpaVendorAdapter);
        //这里要修改成主数据源的扫描包
        entityManager.setPackagesToScan("club.krislin.bootlaunch.jpa.testdb");
        entityManager.setPersistenceUnitName("primaryPersistenceUnit");
        entityManager.setJpaPropertyMap(properties);
        return entityManager;
    }
}
```

设置第二个数据库的数据源及实体扫描管理(扫描testdb2目录):

```java
@Configuration
@DependsOn("transactionManager")
@EnableJpaRepositories(basePackages = "club.krislin.bootlaunch.jpa.testdb2",   //注意这里
        entityManagerFactoryRef = "secondaryEntityManager",
        transactionManagerRef = "transactionManager")
public class JPASecondaryConfig {
    @Autowired
    private JpaVendorAdapter jpaVendorAdapter;


    @Bean(name = "secondaryDataSourceProperties")
    @Qualifier("secondaryDataSourceProperties")
    @ConfigurationProperties(prefix = "spring.datasource.secondary")    //注意这里
    public DataSourceProperties masterDataSourceProperties() {
        return new DataSourceProperties();
    }


    @Bean(name = "secondaryDataSource", initMethod = "init", destroyMethod = "close")
    @ConfigurationProperties(prefix = "spring.datasource.secondary")
    public DataSource masterDataSource() throws SQLException {
        MysqlXADataSource mysqlXaDataSource = new MysqlXADataSource();
        mysqlXaDataSource.setUrl(masterDataSourceProperties().getUrl());
        mysqlXaDataSource.setPinGlobalTxToPhysicalConnection(true);
        mysqlXaDataSource.setPassword(masterDataSourceProperties().getPassword());
        mysqlXaDataSource.setUser(masterDataSourceProperties().getUsername());
        AtomikosDataSourceBean xaDataSource = new AtomikosDataSourceBean();
        xaDataSource.setXaDataSource(mysqlXaDataSource);
        xaDataSource.setUniqueResourceName("secondary");
        xaDataSource.setBorrowConnectionTimeout(60);
        xaDataSource.setMaxPoolSize(20);
        return xaDataSource;
    }

    @Bean(name = "secondaryEntityManager")
    @DependsOn("transactionManager")
    public LocalContainerEntityManagerFactoryBean masterEntityManager() throws Throwable {

        HashMap<String, Object> properties = new HashMap<String, Object>();
        properties.put("hibernate.transaction.jta.platform", AtomikosJtaPlatform.class.getName());
        properties.put("javax.persistence.transactionType", "JTA");
        LocalContainerEntityManagerFactoryBean entityManager = new LocalContainerEntityManagerFactoryBean();
        entityManager.setJtaDataSource(masterDataSource());
        entityManager.setJpaVendorAdapter(jpaVendorAdapter);
        //这里要修改成主数据源的扫描包
        entityManager.setPackagesToScan("club.krislin.bootlaunch.jpa.testdb2");
        entityManager.setPersistenceUnitName("secondaryPersistenceUnit");
        entityManager.setJpaPropertyMap(properties);
        return entityManager;
    }
}
```

大家看没看出来上面的代码有一个关键的点

- DataSourceProperties数据源配置、Datasource数据源、EntityManager实体管理器都是2套。分别是primary和secondary
- 实体和Repository的扫描目录也是2组，分别是testdb和testdb2
- 但是事务管理器只有一个，那就是transactionManager，是基于atomikos实现的。事务管理器只有一个，决定了不同的数据源使用同一个事务管理器，从而实现分布式事务。

# 三、测试一下

service里面分别向testdb插入article，testdb2插入message，数据插入都成功。人为制造一个被除数为0的异常，数据插入都失败。

```java
@Override
@Transactional
public ArticleVO saveArticle( ArticleVO article) {

    Article articlePO = dozerMapper.map(article,Article.class);
    articleRepository.save(articlePO);
    messageRepository.save(new Message(null,"krislin","爱学习"));
    //int a= 2/0;
    return  article;
}
```