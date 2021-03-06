---
title: SpringBoot 04.常用web开发数据库框架 4.12.mybatis+atomikos实现分布式事务
date: 2022-07-03T02:27:18.631Z
tags: [springboot]
---
# 一、整合jta-atomikos

首先需要引入jta的依赖包，注意是JTA(事务管理)，不是JPA。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jta-atomikos</artifactId>
</dependency>
```

双数据源配置。删掉原有其他的数据库连接配置.两个数据源的名称分别是：primary和secondary。分别访问testdb和testdb2数据库。另外注意：驱动类是MysqlXADataSource（支持分布式事务），而不是MysqlDataSource。

```yaml
# 分布式数据库的配置
primarydb:
  uniqueResourceName: primary
  xaDataSourceClassName: com.mysql.cj.jdbc.MysqlXADataSource
  xaProperties:
    url: jdbc:mysql://localhost:3306/testdb?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
    user: root
    password: 123456
  exclusiveConnectionMode: true
  minPoolSize: 3
  maxPoolSize: 10
  testQuery: SELECT 1 from dual #由于采用HikiriCP，用于检测数据库连接是否存活。

secondarydb:
  uniqueResourceName: secondary
  xaDataSourceClassName: com.mysql.cj.jdbc.MysqlXADataSource
  xaProperties:
    url: jdbc:mysql://localhost:3306/testdb2?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
    user: root
    password: 123456
  exclusiveConnectionMode: true
  minPoolSize: 3
  maxPoolSize: 10
  testQuery: SELECT 1 from dual #由于采用HikiriCP，用于检测数据库连接是否存活。

mybatis:
  mapper-locations: classpath:mapper/*.xml  # mapper的xml文件的位置
  type-aliases-package: club.krislin.test*  # mapper（dao）层的位置
```

# 二、配置多数据源及事务管理

数据源DataSource、SqlSessionFactory、SqlSessionTemplate、扫描路径，对于primarydb和secondarydb都是自己一套，需要分别配置。
数据源一：primarydb

```java
@Configuration
@EnableConfigurationProperties
@EnableAutoConfiguration
@MapperScan(basePackages = "club.krislin.testdb",    //注意这里
        sqlSessionTemplateRef = "primarySqlSessionTemplate")
public class PrimaryDataSourceJTAConfig {

    @Bean("primaryDataSource")
    @ConfigurationProperties(prefix = "primarydb")
    public DataSource primaryDataSource() {
        return new AtomikosDataSourceBean();
    }

    @Bean("primarySqlSessionFactory")
    public SqlSessionFactory primarySqlSessionFactory(@Qualifier("primaryDataSource") DataSource dataSource)
            throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        /**
         * 当Mapper和Mapper.xml我放在同一个文件夹时不用设资源路径
         * 没有放在同一个位置时要设置资源路径
         */
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/*.xml"));
        bean.setTypeAliasesPackage("club.krislin.testdb");  //这里需要修改为你的扫描类路径
        return bean.getObject();
    }

    @Bean("primarySqlSessionTemplate")
    public SqlSessionTemplate primarySqlSessionTemplate(
            @Qualifier("primarySqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

数据源二：secondarydb。参照数据源一，将primary修改为secondary再配置一组。

```java
@Configuration
@EnableConfigurationProperties
@EnableAutoConfiguration
@MapperScan(basePackages = "club.krislin.testdb2",   //注意这里
        sqlSessionTemplateRef = "secondarySqlSessionTemplate")
public class SecondaryDataSourceJTAConfig {

    @Bean("secondaryDataSource")
    @ConfigurationProperties(prefix = "secondarydb")
    @Primary
    public DataSource secondaryDataSource() {
        return new AtomikosDataSourceBean();
    }

    @Bean("secondarySqlSessionFactory")
    @Primary
    public SqlSessionFactory secondarySqlSessionFactory(@Qualifier("secondaryDataSource") DataSource dataSource)
            throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/*.xml"));
        bean.setTypeAliasesPackage("club.krislin.testdb2");   //注意这里
        return bean.getObject();
    }

    @Bean("secondarySqlSessionTemplate")
    @Primary
    public SqlSessionTemplate secondarySqlSessionTemplate(
            @Qualifier("secondarySqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

虽然我们将数据源及其相关配置分成了两组，但这两组数据源使用的事务管理器必须是同一个，这样才能实现分布式事务。下面是事务管理器的配置。固定代码，一点不用改，不要纠结。不需要问张三为什么叫张三，因为他爸爸就是这么给他起名的。

```java
@Configuration
@EnableTransactionManagement
public class XATransactionManagerConfig {

     //User事务
    @Bean(name = "userTransaction")
    public UserTransaction userTransaction() throws Throwable {
        UserTransactionImp userTransactionImp = new UserTransactionImp();
        userTransactionImp.setTransactionTimeout(10000);
        return userTransactionImp;
    }
    //分布式事务
    @Bean(name = "atomikosTransactionManager", initMethod = "init", destroyMethod = "close")
    public TransactionManager atomikosTransactionManager() throws Throwable {
        UserTransactionManager userTransactionManager = new UserTransactionManager();
        userTransactionManager.setForceShutdown(false);
        return userTransactionManager;
    }
    //事务管理器
    @Bean(name = "transactionManager")
    @DependsOn({ "userTransaction", "atomikosTransactionManager" })
    public PlatformTransactionManager transactionManager() throws Throwable {
        return new JtaTransactionManager(userTransaction(),atomikosTransactionManager());
    }

}
```

# 三、测试

将自动生成的代码，分别存放于testdb和testdb2两个文件夹
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200425110445.png)

测试代码

```java
@SpringBootTest
public class JTADataSourceTest {
    @Autowired
    ArticleDao articleDao;

    @Autowired
    MessageDao messageDao;

    @Test
    @Transactional
    public void ArticleTest() {
        Article article = Article.builder()
                .author("欧文")
                .content("如何运球")
                .title("篮球")
                .createTime(new Date())
                .build();

        articleDao.insert(article);

        Message message = Message.builder()
                .author("张宇")
                .content("凌晨四点半的大学")
                .title("奋斗")
                .createTime(new Date())
                .build();
        messageDao.insert(message);

        int a = 2 / 0;

    }
}
```

正常情况下，2组数据分别插入到testdb的article表和testdb2的message表。如果我们人为制造一个异常(如上面代码)，事务回滚，二者均无法插入数据。