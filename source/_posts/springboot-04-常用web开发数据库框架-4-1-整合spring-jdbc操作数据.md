---
title: SpringBoot 04.常用web开发数据库框架 4.1.整合Spring JDBC操作数据
date: 2022-07-03T01:13:36.202Z
tags: [springboot]
---
# 一、jdbc简介

JDBC（Java DataBase Connectivity,java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。JDBC提供了一种基准，据此可以构建更高级的工具和接口，使数据库开发人员能够编写数据库应用程序，

# 二、使用jdbc操作数据库的步骤

直接在 Java 程序中使用 JDBC 比较复杂，需要 7 步才能完成数据库的操作：

1. 加载数据库驱动
2. 建立数据库连接
3. 创建数据库操作对象
4. 定义操作的 SQL 语句
5. 执行数据库操作
6. 获取并操作结果集
7. 关闭对象，回收资源

关键代码如下：

```java
try {

    // 1、加载数据库驱动
    Class.forName(driver);

    // 2、获取数据库连接
    conn = DriverManager.getConnection(url, username, password);

    // 3、获取数据库操作对象
    stmt = conn.createStatement();

    // 4、定义操作的 SQL 语句
    String sql = "select * from user where id = 6";

    // 5、执行数据库操作
    rs = stmt.executeQuery(sql);

    // 6、获取并操作结果集
    while (rs.next()) {

    // 解析结果集

    }

} catch (Exception e) {
    // 日志信息
} finally {
    // 7、关闭资源
}
```

通过上面的示例可以看出直接使用 JDBC 来操作数据库比较复杂。为此，Spring Boot 针对 JDBC 的使用提供了对应的 Starter 包：spring-boot-starter-jdbc，它其实就是在 Spring JDBC 上做了进一步的封装，方便在 Spring Boot 生态中更好的使用 JDBC。

# 三、 将Spring JDBC集成到Spring boot项目

第一步：引入maven依赖包，包括spring JDBC和MySQL驱动。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

第二步：修改application.yml，增加数据库连接、用户名、密码相关的配置。driver-class-name请根据自己使用的数据库和数据库版本准确填写。

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/testdb?useUnicode=true&characterEncoding=utf-8
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
```

# 四、  spring boot jdbc 基础代码

spring jdbc集成完毕之后，我们来写代码做一个基本的测试。首先我们新建一张测试表article

```sql
CREATE TABLE `article` (
	`id` INT(11) NOT NULL AUTO_INCREMENT,
	`author` VARCHAR(32) NOT NULL,
	`title` VARCHAR(32) NOT NULL,
	`content` VARCHAR(512) NOT NULL,
	`create_time` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
	PRIMARY KEY (`id`)
)
COMMENT='文章'
ENGINE=InnoDB;
```

DAO层代码:

- jdbcTemplate.update适合于insert 、update和delete操作；
- jdbcTemplate.queryForObject用于查询单条记录返回结果
- jdbcTemplate.query用于查询结果列表
- BeanPropertyRowMapper可以将数据库字段的值向对象映射，满足驼峰标识也可以自动映射。如:数据库create_time字段映射到createTime属性。

```java
@Repository
public class ArticleJDBCDAO {

    @Resource
    private JdbcTemplate jdbcTemplate;

    //保存文章
    public void save(Article article) {
        //jdbcTemplate.update适合于insert 、update和delete操作；
        jdbcTemplate.update("INSERT INTO article(author, title,content,create_time) values(?, ?, ?, ?)",
                article.getAuthor(),
                article.getTitle(),
                article.getContent(),
                article.getCreateTime());

    }

    //删除文章
    public void deleteById(Long id) {
        //jdbcTemplate.update适合于insert 、update和delete操作；
        jdbcTemplate.update("DELETE FROM article WHERE id = ?",new Object[]{id});

    }

    //更新文章
    public void updateById(Article article) {
        //jdbcTemplate.update适合于insert 、update和delete操作；
        jdbcTemplate.update("UPDATE article SET author = ?, title = ? ,content = ?,create_time = ? WHERE id = ?",
                article.getAuthor(),
                article.getTitle(),
                article.getContent(),
                article.getCreateTime(),
                article.getId());

    }

    //根据id查找文章
    public Article findById(Long id) {
        //queryForObject用于查询单条记录返回结果
        return (Article) jdbcTemplate.queryForObject("SELECT * FROM article WHERE id=?", new Object[]{id}, new BeanPropertyRowMapper(Article.class));
    }

    //查询所有
    public List<Article> findAll(){
        //query用于查询结果列表
        return (List<Article>) jdbcTemplate.query("SELECT * FROM article ",  new BeanPropertyRowMapper(Article.class));
    }


}
```

service层接口:

```java
public interface ArticleRestService {

    public Article saveArticle(Article article);

    public void deleteArticle(Long id);

    public void updateArticle(Article article);

    public Article getArticle(Long id);

    public List<Article> getAll();
}
```

service层接口实现

```java
@Slf4j
@Service
public class ArticleRestJDBCService implements ArticleRestService{

    @Resource
    private
    ArticleJDBCDAO articleJDBCDAO;

    @Transactional
    public Article saveArticle( Article article) {
        articleJDBCDAO.save(article);
        //int a = 2/0；  //人为制造一个异常，用于测试事务
        return article;
    }

    public void deleteArticle(Long id){
        articleJDBCDAO.deleteById(id);
    }

    public void updateArticle(Article article){
        articleJDBCDAO.updateById(article);
    }

    public Article getArticle(Long id){
        return articleJDBCDAO.findById(id);
    }

    public List<Article> getAll(){
        return articleJDBCDAO.findAll();
    }
}
```

重点测试一下事务的回滚。在saveArticle方法上使用了@Trasactional注解，该注解基本功能为事务管理，保证saveArticle方法一旦有异常，所有的数据库操作就回滚。