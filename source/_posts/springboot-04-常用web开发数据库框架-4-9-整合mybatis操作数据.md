---
title: SpringBoot 04.常用web开发数据库框架 4.9.整合Mybatis操作数据
date: 2022-07-03T02:08:07.150Z
tags: [springboot]
---
# 一、整合Mybatis

第一步：引入maven依赖包，包括mybatis相关依赖包和mysql驱动包。

```xml
      <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!-- alibaba的druid数据库连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.20</version>
        </dependency>
```

第二步：保证application.yml里面有数据库连接的配置。并配置mybatis的xml文件存放位置，下文配置的xml文件目录位置是resources/mapper。

```yaml
server:
  port: 8888

spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
  datasource:
    url: jdbc:mysql://localhost:3306/testdb?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
    # 使用druid数据源
    type: com.alibaba.druid.pool.DruidDataSource
    filters: stat
    maxActive: 20
    initialSize: 1
    maxWait: 60000
    minIdle: 1
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: select 'x'
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    maxOpenPreparedStatements: 20

## 该配置节点为独立的节点，有很多同学容易将这个配置放在spring的节点下，导致配置无法被识别
mybatis:
  mapper-locations: classpath:mapper/*.xml  #注意：一定要对应mapper映射xml文件的所在路径
  type-aliases-package: club.krislin.entity  # 注意：对应实体类的路径
```

第三步：配置Mybatis的Mapper(dao)的扫描路径

```java
@SpringBootApplication
@MapperScan(basePackages = {"club.krislin.dao"})
public class BootLaunchApplication {

    public static void main(String[] args) {
        SpringApplication.run(BootLaunchApplication.class, args);
    }

}
```

# 二、安装mybatis generator插件

这个插件将帮助我们根据数据库表结构生成Mybatis操作接口及实体类定义等内容。能极大的方便我们开发，减少手写代码量。
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200424123821.png)

[插件怎么安装，请点这里？](https://github.com/kmaster/better-mybatis-generator)
[插件生成的代码详细用法？点这里](https://www.kancloud.cn/hanxt/springboot2/1177661)

# 三、增删改查实现代码

Service层接口

```java
public interface ArticleRestService {

     ArticleVO saveArticle(ArticleVO article);

     void deleteArticle(Long id);

     void updateArticle(ArticleVO article);

     ArticleVO getArticle(Long id);

     List<ArticleVO> getAll();public interface IArticleService {

    /**
     * 通过ID查询单条数据
     *
     * @param id 主键
     * @return 实例对象
     */
    Article queryById(Integer id);

    /**
     * 查询多条数据
     *
     * @param offset 查询起始位置
     * @param limit 查询条数
     * @return 对象列表
     */
    List<Article> queryAllByLimit(int offset, int limit);

    /**
     * 新增数据
     *
     * @param article 实例对象
     * @return 实例对象
     */
    Article insert(Article article);

    /**
     * 修改数据
     *
     * @param article 实例对象
     * @return 实例对象
     */
    Article update(Article article);

    /**
     * 通过主键删除数据
     *
     * @param id 主键
     * @return 是否成功
     */
    boolean deleteById(Integer id);
}
```

Service接口实现

```java
@Service("articleService")
public class ArticleServiceImpl implements IArticleService {
    @Resource
    private ArticleDao articleDao;

    /**
     * 通过ID查询单条数据
     *
     * @param id 主键
     * @return 实例对象
     */
    @Override
    public Article queryById(Integer id) {
        return this.articleDao.queryById(id);
    }

    /**
     * 查询多条数据
     *
     * @param offset 查询起始位置
     * @param limit 查询条数
     * @return 对象列表
     */
    @Override
    public List<Article> queryAllByLimit(int offset, int limit) {
        return this.articleDao.queryAllByLimit(offset, limit);
    }

    /**
     * 新增数据
     *
     * @param article 实例对象
     * @return 实例对象
     */
    @Override
    public Article insert(Article article) {
        this.articleDao.insert(article);
        return article;
    }

    /**
     * 修改数据
     *
     * @param article 实例对象
     * @return 实例对象
     */
    @Override
    public Article update(Article article) {
        this.articleDao.update(article);
        return this.queryById(article.getId());
    }

    /**
     * 通过主键删除数据
     *
     * @param id 主键
     * @return 是否成功
     */
    @Override
    public boolean deleteById(Integer id) {
        return this.articleDao.deleteById(id) > 0;
    }
}
```

# 四、测试

ArticleTest测试类

```java
@SpringBootTest
public class ArticleTest {
    @Autowired
    IArticleService articleService;

    @Test
    public void test(){
        Article article = articleService.queryById(1);
        System.out.println(article);
    }
}
```



