---
title: SpringBoot 04.常用web开发数据库框架 4.11.Spring mybatis的多数据源实现
date: 2022-07-03T02:23:34.882Z
tags: [springboot]
---
# 一、修改application.yml为双数据源

在application.yml配置双数据源，第一个数据源访问testdb库，第二个数据源访问testdb2库

```java
spring:
  datasource:
    primary:
      url: jdbc:mysql://localhost:3306/testdb?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
      username: root
      password: 123456
      driver-class-name: com.mysql.cj.jdbc.Driver
    secondary:
      url: jdbc:mysql://localhost:3306/testdb2?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
      username: root
      password: 123456
      driver-class-name: com.mysql.cj.jdbc.Driver
```

# 二、主数据源配置

去掉SpringBoot程序主入口上的@MapperScan注解，将注解移到下面的MyBatis专用配置类上方。 DataSource数据源、SqlSessionFactory、TransactionManager事务管理器、SqlSessionTemplate依据不同的数据源分别配置。第一组是primary，第二组是secondary。

```java
@Configuration
@MapperScan(basePackages = "club.krislin.testdb",    //数据源primary-testdb库接口存放目录
        sqlSessionTemplateRef = "primarySqlSessionTemplate")
public class PrimaryDataSourceConfig {

    @Bean(name = "primaryDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.primary")   //数据源primary配置
    @Primary
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "primarySqlSessionFactory")
    @Primary
    public SqlSessionFactory testSqlSessionFactory(
                        @Qualifier("primaryDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
         //设置XML文件存放位置，如果参考上一篇Mybatis最佳实践，将xml和java放在同一目录下，这里不用配置
        //bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/test1/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "primaryTransactionManager")
    @Primary
    public DataSourceTransactionManager testTransactionManager(
                        @Qualifier("primaryDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "primarySqlSessionTemplate")
    @Primary
    public SqlSessionTemplate testSqlSessionTemplate(
                        @Qualifier("primarySqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

# 三、第二个数据源配置

参照primary配置，书写第二份secondary数据源配置的代码。

```java
@Configuration
@MapperScan(basePackages = "club.krislin.testdb2",     //注意这里testdb2目录
        sqlSessionTemplateRef = "secondarySqlSessionTemplate")
public class SecondaryDataSourceConfig {

    @Bean(name = "secondaryDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.secondary")    //注意这里secondary配置
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "secondarySqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory(
                        @Qualifier("secondaryDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        //bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mybatis/mapper/test1/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "secondaryTransactionManager")
    public DataSourceTransactionManager testTransactionManager(
                        @Qualifier("secondaryDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "secondarySqlSessionTemplate")
    public SqlSessionTemplate testSqlSessionTemplate(
                        @Qualifier("secondarySqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }

}
```

# 四、 测试用例

将自动生成的代码（自己写Mapper和实体类也可以），分别存放于testdb和testdb2两个文件夹
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200425100739.png)

测试代码：

```java
@SpringBootTest
public class DoubleDataSourcesTest {
    @Autowired
    ArticleDao articleDao;

    @Autowired
    MessageDao messageDao;

    @Test
    public void ArticleTest(){
        Article article = Article.builder()
                .author("james")
                .content("如何打篮球")
                .title("篮球")
                .createTime(new Date())
                .build();

        articleDao.insert(article);

        List<Article> articles = articleDao.queryAllByLimit(0, 5);
        for (Article article1:articles){
            System.out.println(article1);
        }
    }

    @Test
    public void MessageTest(){
        Message message = Message.builder()
                .author("kobe")
                .content("凌晨四点半的洛杉矶")
                .title("奋斗")
                .createTime(new Date())
                .build();
        messageDao.insert(message);

        List<Message> messages = messageDao.queryAllByLimit(0, 5);
        for (Message message1:messages){
            System.out.println(message1);
        }
    }
}

```
