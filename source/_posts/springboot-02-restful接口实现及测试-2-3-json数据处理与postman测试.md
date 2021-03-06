---
title: SpringBoot 02.RESTful接口实现及测试 2.3 JSON数据处理与PostMan测试
date: 2022-07-02T10:58:44.510Z
tags: [springboot]
---
# 一、 FastJSON、Gson和Jackson对比

**开源的Jackson**：SpringBoot默认是使用Jackson作为JSON数据格式处理的类库，Jackson在各方面都比较优秀，所以不建议将Jackson替换为Gson或fastjson。

**Google的Gson**：Gson是Google为满足内部需求开发的JSON数据处理类库，其核心结构非常简单，toJson与fromJson两个转换函数实现对象与JSON数据的转换，

**阿里巴巴的FastJson**：Fastjson是阿里巴巴开源的JSON数据处理类库，其主要特点是序列化速度快。当并发数据量越大的时候，越能体现出fastjson的优势。JSON处理类库时快并不是唯一需要考虑的因素，与数据库或磁盘IO相比，JSON数据序列化与反序列化的这点时间还不足以对软件性能产生比较大的影响。

**性能比较**：笔者看多很多的关于这三个类库的性能测试（截止2019年11月20日），总结如下：

- 序列化过程性能：fastjson >= jackson > Gson，Gson在数据并发量较大时会与其他二者有较明显差距。
- 反序列化性能：三者几乎不相上下，Gson略好一点。

**fastjson为人诟病的问题：**：虽然fastjson速度上有一定的优势，但是其为了追求速度，很大程度放弃了JSON的规范性。因此还时不时的在有些版本中暴露安全问题。

# 二、在Spring中注解方法使用Jackson

不建议将Jackson替换为Gson或fastjson。jackson的常用注解的使用方法。

> 什么叫序列化与反序列化？说白了就是把对象转成可传输、可存储的格式（json、xml、二进制、甚至自定义格式）叫做序列化。反序列化顾名思义。

## 常用注解

这些注解通常用于标注java实体类或实体类的属性。

- @JsonPropertyOrder(value={"pname1","pname2"}) 改变子属性在JSON序列化中的默认定义的顺序。如：param1在先，param2在后。
- @JsonIgnore 排除某个属性不做序列化与反序列化
- @JsonProperty(anotherName) 为某个属性换一个名称，体现在JSON数据里面
- @JsonInclude(JsonInclude.Include.NON_NULL) 排除为空的元素不做序列化反序列化
- @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8") 指定日期类型的属性格式

```java
@JsonPropertyOrder(value={"content","title"})  
public class Article {

    @JsonIgnore
    private Long id;

    @JsonProperty("auther")
    private String author;
    private String title;
    private String content;

    @JsonInclude(JsonInclude.Include.NON_NULL)
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date createTime;
    private List<Reader> reader;

}
```

上文代码中对应的JSON数据格式可以为：

```json
{
    auther :"",
    content:"",
    title:"",
    createTime:"2019-10-20 12:12:12",
    reader:[{"name":"zimug","age":18},{"name":"kobe","age":37}]
}
```

- 因为定义了JsonPropertyOrder，content在先，title在后
- 因为定义了JsonIgnore，id属性被忽略
- 因为定义了JsonProperty，author属性变为auther
- 因为定义了JsonInclude和JsonFormat，createTime不要为空，并且格式为 "yyyy-MM-dd HH:mm:ss"

通常会对日期类型转换，进行全局配置，而不是在每一个java bean里面配置

```yaml
spring: 
    jackson:
        date-format: yyyy-MM-dd HH:mm:ss
        time-zone: GMT+8
```

# 三、手动数据转换

除了在spring框架内实现自动的前后端JSON数据与java对象的转换，我们还可以使用jackson自己写代码进行转换。

```java
//jackson的ObjectMapper 转换对象
ObjectMapper mapper = new ObjectMapper();
//将某个java对象转换为JSON字符串
String jsonStr = mapper.writeValueAsString(javaObj);
//将jsonStr转换为Ademo类的对象
Ademo ademo = mapper.readValue(jsonStr, Ademo.class);
```

当JSON字符串代表的对象的字段多于类定义的字段时，使用readValue会抛出UnrecognizedPropertyException异常，在类的定义处加上@JsonIgnoreProperties(ignoreUnknown = true)可以解决这个问题。

# 四、Postman测试

下面让我们结合postman对REST接口和Jackson做一下测试吧。Postman是接口测试过程中经常使用到的工具。
测试使用数据:

```json
{
    "id": 1,
    "author": "krislin",
    "title": "spring boot学习",
    "content": "c",
    "createTime": "",
}
```

下面以测试新增文章的接口为例：

- 测试的接口服务端点为“/rest/article”
- 服务端点支持的HTTP方法为POST
- 使用Http协议的body传输JSON数据，对应Controller应该使用RequestBody进行数据参数接收
- 点击Send进行接口数据的发送

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200416162708.png)

