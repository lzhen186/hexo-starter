---
title: SpringBoot 12.整合分布式文件系统fastdfs 12.3.开发一个自定义fastdfs-starter
date: 2022-07-03T14:12:48.567Z
tags: [springboot]
---
# 一、主要实现的功能

1. 实现FastDFSClientUtil及properties的自动装配（即：如何开发一个自定义的spring-boot-starter）
2. 加入连接线程池管理（非重点）
   ![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200501135940.png)

**[krislin-fastdfs-spring-boot-start Github地址](https://github.com/krislinzhao/krislin-fastdfs-spring-boot-starter)**

# 二、实现FastDFSClientUtil及properties的自动装配

实际上就是要自己实现一个starter

第一步：maven的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>club.krislin.spring</groupId>
    <artifactId>krislin-fastdfs-spring-boot-starter</artifactId>
    <version>1.0.0</version>
    <name>krislin-fastdfs-spring-boot-starter</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>net.oschina.zcx7878</groupId>
            <artifactId>fastdfs-client-java</artifactId>
            <version>1.27.0.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>2.0</version>
        </dependency>
    </dependencies>

</project>
```

注意其中 spring-boot-configuration-processor 的作用是编译时生成spring-configuration-metadata.json， 此文件主要给IDE使用，用于提示使用。如在intellij idea中，当配置此jar相关配置属性在application.yml， 你可以用ctlr+鼠标左键，IDE会跳转到你配置此属性的类中。
fastdfs-client-java和commons-pool2先不用管他们两个是实现fastdfs功能及连接池的，与自动装配无关

第二步：fastdfs属性类

```java
@ConfigurationProperties(prefix = "krislin.fastdfs")
public class FastDFSProperties {

    private Integer connect_timeout = 5;
    private Integer network_timeout = 30;
    private String charset = "UTF-8";
    private List<String> tracker_server = new ArrayList<>();
    private Integer max_total;
    private Boolean http_anti_steal_token = false;
    private String http_secret_key = "";
    private Integer http_tracker_http_port = 8987;
    //下面这个实际上不是fastdfs的属性，为了方便实用自定义属性，表示访问nginx的http地址
    private String httpserver;

    public Integer getHttp_tracker_http_port() {
        return http_tracker_http_port;
    }

    public void setHttp_tracker_http_port(Integer http_tracker_http_port) {
        this.http_tracker_http_port = http_tracker_http_port;
    }

    public Boolean getHttp_anti_steal_token() {
        return http_anti_steal_token;
    }

    public void setHttp_anti_steal_token(Boolean http_anti_steal_token) {
        this.http_anti_steal_token = http_anti_steal_token;
    }

    public String getHttp_secret_key() {
        return http_secret_key;
    }

    public void setHttp_secret_key(String http_secret_key) {
        this.http_secret_key = http_secret_key;
    }

    public Integer getMax_total() {
        return max_total;
    }

    public void setMax_total(Integer max_total) {
        this.max_total = max_total;
    }

    public String getHttpserver() {
        return httpserver;
    }

    public void setHttpserver(String httpserver) {
        this.httpserver = httpserver;
    }

    public List<String> getTracker_server() {
        return tracker_server;
    }

    public void setTracker_server(List<String> tracker_server) {
        this.tracker_server = tracker_server;
    }

    public Integer getConnect_timeout() {
        return connect_timeout;
    }

    public void setConnect_timeout(Integer connect_timeout) {
        this.connect_timeout = connect_timeout;
    }

    public Integer getNetwork_timeout() {
        return network_timeout;
    }

    public void setNetwork_timeout(Integer network_timeout) {
        this.network_timeout = network_timeout;
    }

    public String getCharset() {
        return charset;
    }

    public void setCharset(String charset) {
        this.charset = charset;
    }
}
```

第三步：自动装配配置类

```java
/**
 * 实现最终目标把FastDFSClientUtil自动注入Spring，提供外部使用
* 使用方式 @Resource、 @Autowired
 */
@Configuration
//当classpath下面有这三个类才做自动装配
@ConditionalOnClass(value = {FastDFSClientFactory.class,FastDFSClientPool.class,FastDFSClientUtil.class,})
//@EnableConfigurationProperties 相当于把使用 @ConfigurationProperties的类注入。
@EnableConfigurationProperties(FastDFSProperties.class)
public class AutoConfigure {

    private final FastDFSProperties properties;

    @Autowired
    public AutoConfigure(FastDFSProperties properties) {
        this.properties = properties;
    }

    @Bean
    FastDFSClientPool fastDFSClientPool(){
        return new FastDFSClientPool(properties);
    }

    /**
     * 当没有FastDFSClientUtil，就把FastDFSClientUtil作为Bean注入Spring
     */
    @Bean
    @ConditionalOnMissingBean
    FastDFSClientUtil fastDFSClientUtil (){
        return  new FastDFSClientUtil(properties);
    }

}
```

最后一步，在resources/META-INF/下创建spring.factories文件，内容供参考下面：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
club.krislin.spring.fastdfs.AutoConfigure
```

如果有多个自动配置类，用逗号分隔换行即可。

# 四、测试自动装配的结果

application.yml

```yaml
krislin:
    fastdfs:
      httpserver: http://192.168.1.91:80/
      connect_timeout: 5
      network_timeout: 30
      charset: UTF-8
      tracker_server:
        - 192.168.1.91:22122
      max_total: 50
      http_anti_steal_token:false
      http_secret_key: 
```

# 五、总结下Starter的工作原理:

```
Spring Boot在启动时扫描项目所依赖的JAR包，寻找包含spring.factories文件的JAR包
根据spring.factories配置加载AutoConfigure类
根据 @Conditional 注解的条件，进行自动配置并将Bean注入Spring Context
```