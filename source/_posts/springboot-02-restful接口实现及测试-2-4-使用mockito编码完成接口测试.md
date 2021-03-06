---
title: SpringBoot 02.RESTful接口实现及测试 2.4.使用Mockito编码完成接口测试
date: 2022-07-02T11:11:26.245Z
tags: [springboot]
---
# 一、介绍Mock

## 1.1什么是Mock?

在面向对象程序设计中，模拟对象（英语：mock object，也译作模仿对象）是以可控的方式模拟真实对象行为的**假的对象**。比如:对象B依赖于对象A，但是A代码还没写是一个空类空方法不能用，我们来mock一个假的A来完成测试。

## 1.2 为什么要使用Mock?

> 在单元测试中，模拟对象可以模拟复杂的、真实的对象的行为， 如果真实的对象无法放入单元测试中，使用模拟对象就很有帮助。

在下面的情形，可能需要使用模拟对象来代替真实对象：

- 真实对象的行为是不确定的（例如，当前的时间或当前的温度）；
- 真实对象很难搭建起来；
- 真实对象的行为很难触发（例如，网络错误）；
- 真实对象速度很慢（例如，一个完整的数据库，在测试之前可能需要初始化）；
- 真实的对象是用户界面，或包括用户界面在内；
- 真实的对象使用了回调机制；
- 真实对象可能还不存在（例如，其他程序员还为完成工作）；
- 真实对象可能包含不能用作测试的信息（高度保密信息等）和方法。

## 1.3 Mockito

Mockito是GitHub上使用最广泛的Mock框架,并与JUnit结合使用.Mockito框架可以创建和配置mock对象.使用Mockito简化了具有外部依赖的类的测试开发!

# 二、模拟网络请求进行测试

在开始书写测试代码之前，我们先回顾一下JUnit常用的测试注解。在junit4和junit5中，注解的写法有些许变化。

| junit4         | junit5        | 特点                                                   |
| :------------- | :------------ | :----------------------------------------------------- |
| `@Test`        | `@Test`       | 声明一个测试方法                                       |
| `@BeforeClass` | `@BeforeAll`  | 在当前类的所有测试方法之前执行。注解在【静态方法】上   |
| `@AfterClass`  | `@AfterAll`   | 在当前类中的所有测试方法之后执行。注解在【静态方法】上 |
| `@Before`      | `@BeforeEach` | 在每个测试方法之前执行。注解在【非静态方法】上         |
| `@After`       | `@AfterEach`  | 在每个测试方法之后执行。注解在【非静态方法】           |



```java
//@Transactional
@Slf4j
@SpringBootTest
public class ArticleRestControllerTest {

    //mock对象
    private MockMvc mockMvc;

    //mock对象初始化
    @Before
    public void setUp() {
        mockMvc = MockMvcBuilders.standaloneSetup(new ArticleRestController()).build();
    }

    //测试方法
    @Test
    public void saveArticle() throws Exception {

        String article = "{\n" +
                "    \"id\": 1,\n" +
                "    \"author\": \"krislin\",\n" +
                "    \"title\": \"spring boot学习\",\n" +
                "    \"content\": \"c\",\n" +
                "    \"createTime\": \"2017-07-16 05:23:34\",\n" +
                "    \"reader\":[{\"name\":\"krislin\",\"age\":18},{\"name\":\"kobe\",\"age\":37}]\n" +
                "}";
        MvcResult result = mockMvc.perform(
                MockMvcRequestBuilders.request(HttpMethod.POST, "/rest/article")
                .contentType("application/json").content(article))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.jsonPath("$.data.author").value("zimug"))
                .andExpect(MockMvcResultMatchers.jsonPath("$.data.reader[0].age").value(18))
                .andDo(print())
                .andReturn();

        log.info(result.getResponse().getContentAsString());

    }
}
```

## 2.1 `@SpringBootTest` 注解说明:

是用来创建Spring的上下文`ApplicationContext`，保证测试在上下文环境里运行。单独使用`@SpringBootTest`不会启动servlet容器。所以**只是使用`SpringBootTest` 注解时不可以使用`@Resource`和`@Autowired`等注解进行bean的依赖注入**。（准确的说是可以使用，但被注解的bean为null）。

## 2.2 @Transactional

可以使单元测试进行事务回滚，以保证数据库表中没有因测试造成的垃圾数据，因此保证单元测试可以反复执行；
但是不建议这么做，使用该注解会破坏测试真实性。请参考这篇文章详细理解：

## 2.3 `MockMvc`对象有以下几个基本的方法:

- `perform `: 执行一个`RequestBuilder`请求，会自动执行`SpringMVC`的流程并映射到相应的控制器Controller执行处理。
- `contentType`：发送请求内容的序列化的格式，`"application/json"`表示JSON数据格式
- `andExpect`: 添加`RequsetMatcher`验证规则，验证控制器执行完成后结果是否正确，或者说是结果是否与我们期望（Expect）的一致。
- `andDo`: 添加`ResultHandler`结果处理器，比如调试时打印结果到控制台
- `andReturn`: 最后返回相应的`MvcResult`,然后进行自定义验证/进行下一步的异步处理

# 三、真实servlet容器环境下的测试

换一种写法：看看有没有什么区别。在测试类上面额外加上这样两个注解，并且`mockMvc`对象使用@Resource自动注入，删掉`Before`注解及`setUp`函数。

```java
@RunWith(SpringRunner.class)
@AutoConfigureMockMvc
```

没加上这两个注解之前，日志中是不会打印这个`SpringBoot`的flag的。

## 3.1 `@RunWith`注解

- `RunWith`方法为我们构造了一个的Servlet容器运行运行环境，并在此环境下测试。然而为什么要构建servlet容器？因为使用了依赖注入，注入了`MockMvc`对象，而在上一个例子里面是我们自己new的。
- 而`@AutoConfigureMockMvc`注解，该注解表示 `MockMvc`由spring容器构建，你只负责注入之后用就可以了。这种写法是为了让测试在Spring容器环境下执行。
- Spring的容器环境是什么呢？比如常见的 Service、Dao 都是Spring容器里的bean，装载到容器里面都可以使用`@Resource`和`@Autowired`来注入引用。

简单的说：**如果你单元测试代码使用了依赖注入就加上`@RunWith`，如果你不是手动`new MockMvc`对象就加上`@AutoConfigureMockMvc`**

# 四、轻量级测试

在`RunWith`的`AutoConfigureMockMvc`注解的共同作用下，启动了`SpringMVC`的运行容器，并且把项目中所有的@Bean全部都注入进来。把所有的bean都注入进来是不是很臃肿？这样会拖慢单元测试的效率。如果我只是想测试一下控制层Controller，怎么办？或者说我只想具体到测试一下`ArticleRestController`，怎么办？要把应用中所有的bean都注入么？有没有轻量级的解决方案？一定是有的。

```java
@RunWith(SpringRunner.class)
@WebMvcTest(ArticleRestController.class)
public class WebMockTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private ArticleRestService service;

    @Test
    public void saveArticle() throws Exception {
        String article = "{\n" +
                "    \"id\": 1,\n" +
                "    \"author\": \"zimug\",\n" +
                "    \"title\": \"手摸手教你开发spring boot\",\n" +
                "    \"content\": \"c\",\n" +
                "    \"createTime\": \"2017-07-16 05:23:34\",\n" +
                "    \"reader\":[{\"name\":\"zimug\",\"age\":18},{\"name\":\"kobe\",\"age\":37}]\n" +
                "}";
    
    
        ObjectMapper objectMapper = new ObjectMapper();
        Article articleObj = objectMapper.readValue(article,Article.class);
    
        //打桩
        when(articleRestService.saveArticle(articleObj)).thenReturn("ok");
    
    
        MvcResult result = mockMvc.perform(MockMvcRequestBuilders.request(HttpMethod.POST, "/rest/article")
                .contentType("application/json").content(article))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.jsonPath("$.data.author").value("zimug"))
                .andExpect(MockMvcResultMatchers.jsonPath("$.data.reader[0].age").value(18))
                .andDo(print())
                .andReturn();
    
        log.info(result.getResponse().getContentAsString());
    
    }
}
```

## 4.1 `@SpringBootTest`与`@WebMvcTest`区别

- `@SpringBootTest`注解告诉`SpringBoot`去寻找一个主配置类(例如带有`@SpringBootApplication`的配置类)，并使用它来启动Spring应用程序上下文。`SpringBootTest`加载完整的应用程序并注入所有可能的bean，因此速度会很慢。
- `@WebMvcTest`注解主要用于controller层测试，只覆盖应用程序的controller层，HTTP请求和响应是Mock出来的，因此不会创建真正的网络连接。`WebMvcTest`要快得多，因为我们只加载了应用程序的一小部分。

## 4.2 `@MockBean`

如果我们使用了`WebMvcTest`，只加载了Controller层的bean，那么Controller所依赖的Service没有被加载进来怎么办？ 我们可以用`MockBean`伪造模拟一个Service 。大家注意上文代码中，打了一个桩

```java
when(articleRestService.saveArticle(articleObj)).thenReturn("ok");
```

也就是告诉测试用例程序，当你调用`articleRestService.saveArticle(articleObj)`方法的时候，不要去真的调用这个方法，直接返回一个结果（“ok”）就好了。`articleRestService`就是那个不能被测试或者不能测试的类，需要我们来模拟Mock一下。

# 五、`MockMvc`更多的用法总结

```java
//模拟GET请求：
mockMvc.perform(MockMvcRequestBuilders.get("/user/{id}", userId));

//模拟Post请求：
mockMvc.perform(MockMvcRequestBuilders.post("uri", parameters));

//模拟文件上传：
mockMvc.perform(MockMvcRequestBuilders.multipart("uri").file("fileName", "file".getBytes("UTF-8")));


//模拟session和cookie：
mockMvc.perform(MockMvcRequestBuilders.get("uri").sessionAttr("name", "value"));
mockMvc.perform(MockMvcRequestBuilders.get("uri").cookie(new Cookie("name", "value")));

//设置HTTP Header：
mockMvc.perform(MockMvcRequestBuilders
                        .get("uri", parameters)
                        .contentType("application/x-www-form-urlencoded")
                        .accept("application/json")
                        .header("", ""));
```

# 六、为什么要写代码做测试？使用接口测试工具Postman很方便啊

- 因为在做系统的自动化持续集成的时候，会要求自动的做单元测试，只有所有的单元测试都跑通了，才能打包构建。比如：使用maven。这里重点是**自动化**，所以postman这种工具很难插入到持续集成的自动化流程中去。所以需要我们自己写代码完成单元测试。
- 另外写代码测试能模拟出更多复杂的测试场景，类似Postman工具只能完成简单的接口测试。