---
title: SpringBoot 17.应用程序监控管理 17.1.Actuator应用监控快速入门
date: 2022-07-04T01:08:27.763Z
tags: [springboot]
---
# 一、Spring Boot Actuator简介

Spring Boot作为构建微服务节点的方案，一定要提供全面而且细致的监控指标，使微服务更易于管理！微服务不同于单体应用，微服务的每个服务节点都单独部署，独立运行，大型的微服务项目甚至有成百上千个服务节点。这就为我们进行系统监控与运维提出了挑战。为了应对这个挑战，其中最重要的工作之一就是：微服务节点能够合理的暴露服务的相关监控指标，用以对服务进行健康检查、监控管理，从而进行合理的流量规划与安排系统运维工作！
                                         ![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200507123616.png)

Spring Boot Actuator 可以监控我们的应用程序，收集流量和数据库状态、健康状态等监控指标。在生产环境中，我们可以方便的通过HTTP请求获取应用的状态信息，以JSON格式数据响应。
						                   ![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200507123713.png)

**Spring Boot Actuator监控端点的分类**

- 静态配置类：主要是一些静态配置信息，比如： Spring Bean 加载信息、yml 或properties配置信息、环境变量信息、请求接口关系映射信息等；
- 动态指标类：主要用于展现程序运行期状态，例如内存堆栈信息、请求链信息、健康指标信息等；
- 操作控制类：主要是shutdown功能，用户可以远程发送HTTP请求，从而关闭监控功能。

# 二、Actuator开启与配置

在Spring Boot2.x项目中开启Actuator非常简单，只需要引入如下的mavn坐标即可。

```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
```

Spring Boot Actuator启用之后默认开放了两个端点的访问：

- `/actuator/health`用以监控应用状态。返回值是应用状态信息，包含四种状态DOWN(应用不正常), OUT_OF_SERVICE(服务不可用),UP(状态正常), UNKNOWN(状态未知)。如果服务状态正常，我们访问`http:/lhost:port/actuator/health`得到如下响应信息：

```
{
    "status" : "UP"
}
```

- `/actuator/info` 用来响应应用相关信息，默认为空。可以根据我们自己的需要，向服务调用者暴露相关信息。如下所示,配置属性可以随意起名，但都要挂在info下面：

```
info.app-name=spring-boot-actuator-demo
info.description=spring-boot-actuator-demo indexs monitor 
```

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200507123907.png)

如果我们希望开放更多的监控端点给服务调用者，需要配置：**开放部分监控端点**，端点名称用逗号分隔

```properties
management.endpoints.web.exposure.exclude=beans,env
```

**开放所有监控端点:**

```properties
management.endpoints.web.exposure.include=*
```

星号在YAML配置文件中中有特殊的含义，所以使用YAML配置文件一定要加引号，如下所示：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

# 三、常用监控端点说明

注意下图中的服务启用，不等于对外开放访问。对外开放访问的服务端点一定要先开启服务。如果服务不是默认开启的，使用如下方式开启:

```properties
# shutdown是服务端点名称，可以替换
management.endpoint.shutdown.enabled=true
```

| ID             | 描述                                                         | 服务是否默认启用 |
| :------------- | :----------------------------------------------------------- | :--------------- |
| auditevents    | 应用程序的审计事件相关信息                                   | Yes              |
| beans          | 应用中所有Spring Beans的完整列表                             | Yes              |
| conditions     | （configuration and auto-configuration classes）的状态及它们被应用或未被应用的原因 | Yes              |
| configprops    | `@ConfigurationProperties`的集合列表                         | Yes              |
| env            | Spring的 `ConfigurableEnvironment`的属性                     | Yes              |
| flyway         | flyway 数据库迁移路径，如果有的话                            | Yes              |
| liquibase      | Liquibase数据库迁移路径，如果有的话                          | Yes              |
| metrics        | 应用的`metrics`指标信息                                      | Yes              |
| mappings       | 所有`@RequestMapping`路径的集合列表                          | Yes              |
| scheduledtasks | 应用程序中的`计划任务`                                       | Yes              |
| sessions       | 允许从Spring会话支持的会话存储中检索和删除(retrieval and deletion)用户会话。使用Spring Session对反应性Web应用程序的支持时不可用。 | Yes              |
| shutdown       | 允许应用以优雅的方式关闭（默认情况下不启用）                 | No               |
| threaddump     | 执行一个线程dump                                             | Yes              |

如果使用web应用(Spring MVC, Spring WebFlux, 或者 Jersey)，你还可以使用以下端点：

| ID         | 描述                                                         | 默认启用 |
| :--------- | :----------------------------------------------------------- | :------- |
| heapdump   | 返回一个GZip压缩的`hprof`堆dump文件                          | Yes      |
| jolokia    | 通过HTTP暴露`JMX beans`（当Jolokia在类路径上时，WebFlux不可用） | Yes      |
| logfile    | 返回`日志文件内容`（如果设置了logging.file或logging.path属性的话），支持使用HTTP **Range**头接收日志文件内容的部分信息 | Yes      |
| prometheus | 以可以被Prometheus服务器抓取的格式显示`metrics`信息          | Yes      |
