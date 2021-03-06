---
title: SpringBoot 04.常用web开发数据库框架 4.2 Spring JDBC多数据源的实现
date: 2022-07-03T01:20:28.310Z
tags: [springboot]
---
# 一、配置多个数据源

application.yml配置2个数据源，第一个叫做primary，第二个叫做secondary。注意两个数据源连接的是不同的库，testdb和testdb2.

```java
spring:
  datasource:
    primary:
      jdbc-url: jdbc:mysql://localhost:3306/testdb?useUnicode=true&characterEncoding=utf-8
      username: root
      password: 123456
      driver-class-name: com.mysql.jdbc.Driver
    secondary:
      jdbc-url: jdbc:mysql://localhost:3306/testdb2?useUnicode=true&characterEncoding=utf-8
      username: root
      password: 123456
      driver-class-name: com.mysql.jdbc.Driver
```

# 二、通过Java Config将数据源注入到Spring上下文。

- primaryJdbcTemplate使用primaryDataSource数据源操作数据库testdb。
- secondaryJdbcTemplate使用secondaryDataSource数据源操作数据库testdb2。

```java
@Configuration
public class DataSourceConfig {
    @Primary
    @Bean(name = "primaryDataSource")
    @Qualifier("primaryDataSource")
    @ConfigurationProperties(prefix="spring.datasource.primary")
    public DataSource primaryDataSource() {
            return DataSourceBuilder.create().build();
    }

    @Bean(name = "secondaryDataSource")
    @Qualifier("secondaryDataSource")
    @ConfigurationProperties(prefix="spring.datasource.secondary")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name="primaryJdbcTemplate")
    public JdbcTemplate primaryJdbcTemplate (
        @Qualifier("primaryDataSource") DataSource dataSource ) {
        return new JdbcTemplate(dataSource);
    }

    @Bean(name="secondaryJdbcTemplate")
    public JdbcTemplate secondaryJdbcTemplate(
            @Qualifier("secondaryDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
```

# 三、ArticleJDBCDAO改造

- 在上一节的代码的基础上改造
- 注入primaryJdbcTemplate作为默认的数据库操作对象。
- 将jdbcTemplate作为参数传入ArticleJDBCDAO的方法，不同的template操作不同的库。

```java
@Repository
public class ArticleJDBCDAO {

    @Resource
    private JdbcTemplate primaryJdbcTemplate;


    //以保存文章为例，其他的照做
    public void save(Article article,JdbcTemplate jdbcTemplate ) {
        if(jdbcTemplate == null){
            jdbcTemplate= primaryJdbcTemplate;
        }

        //jdbcTemplate.update适合于insert 、update和delete操作；
        jdbcTemplate.update("INSERT INTO article(author, title,content,create_time) values(?, ?, ?, ?)",
                article.getAuthor(),
                article.getTitle(),
                article.getContent(),
                article.getCreateTime());
    
    }


}
```

# 四、测试同时向两个数据库保存数据

在src/test/java目录下，加入如下单元测试类，并进行测试。正常情况下，在testdb和testdb2数据库的article表，将分别插入一条数据。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringJdbcTest {

    @Resource
    private ArticleJDBCDAO articleJDBCDAO;
    @Resource
    private JdbcTemplate primaryJdbcTemplate;
    @Resource
    private JdbcTemplate secondaryJdbcTemplate;


    @Test
    public void testJdbc() {
        articleJDBCDAO.save(
                Article.builder()
                .author("zimug").title("primaryJdbcTemplate").content("ceshi").createTime(new Date())
                .build(),
                primaryJdbcTemplate);
        articleJDBCDAO.save(
                Article.builder()
               .author("zimug").title("secondaryJdbcTemplate").content("ceshi").createTime(new Date())
               .build(),
                secondaryJdbcTemplate);
    }


}
```