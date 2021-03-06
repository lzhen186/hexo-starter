---
title: SpringBoot 02.RESTful接口实现及测试 2.2.1.Spring常用注解的讲解
date: 2022-07-02T09:29:33.191Z
tags: [springboot]
---
# 一、常用注解回顾

## 1.1 `@RequestBody`与`@ResponseBody`

```java
//注意并不要求@RequestBody与@ResponseBody成对使用。
public @ResponseBody  AjaxResponse saveArticle(@RequestBody ArticleVO article)
```

如上代码所示：

- `@RequestBody`修饰请求参数，注解用于接收HTTP的body，默认是使用JSON的格式
- `@ResponseBody`修饰返回值，注解用于在HTTP的body中携带响应数据，默认是使用JSON的格式。如果不加该注解，spring响应字符串类型，是跳转到模板页面或JSP页面的开发模式。说白了：加上这个注解你开发的是一个数据接口，不加这个注解你开发的是一个页面跳转控制器。

那么我们有一个问题：如果我们想接收或XML数据该怎么办？我们想响应excel的数据格式该怎么办？

## 1.2. `@RequestMapping`注解

`@RequestMapping`注解是所有常用注解中，最有看点的一个注解，用于标注HTTP服务端点。它的很多属性对于丰富我们的应用开发方式方法，都有很重要的作用。如：

- `value`： 应用请求端点，最核心的属性，用于标志请求处理方法的唯一性；
- `method`： HTTP协议的method类型， 如：GET、POST、PUT、DELETE等；
- `consumes`： HTTP协议请求内容的数据类型（Content-Type），例如`application/json`, `text/html`;
- `produces`: HTTP协议响应内容的数据类型。下文会详细讲解。
- `params`： HTTP请求中必须包含某些参数值的时候，才允许被注解标注的方法处理请求。
- `headers`： HTTP请求中必须包含某些指定的header值，才允许被注解标注的方法处理请求。

```java
@RequestMapping(value = "/article", method = POST)
@PostMapping(value = "/article")
```

上面代码中两种写法起到的是一样的效果，也就是`PostMapping`等同于`@RequestMapping`的method等于POST。同理：`@GetMapping`、`@PutMapping`、`@DeleteMapping`也都是简写的方式。

## 1.3. `@RestController`与`@Controller`

`@Controller`注解是开发中最常使用的注解，它的作用有两层含义：

- 一是告诉Spring，被该注解标注的类是一个Spring的Bean，需要被注入到Spring的上下文环境中。
- 二是该类里面所有被`RequestMapping`标注的注解都是HTTP服务端点。

`@RestController`相当于 `@Controller`和`@ResponseBody`结合。它有两层含义：

- 一是作为Controller的作用，将控制器类注入到Spring上下文环境，该类`RequestMapping`标注方法为HTTP服务端点。
- 二是作为`ResponseBody`的作用，请求响应默认使用的序列化方式是JSON，而不是跳转到`jsp`或模板页面。

## 1.4. `@PathVariable` 与`@RequestParam`

`PathVariable`用于URI上的{参数}，如下方法用于删除一篇文章，其中id为文章id。如：我们的请求URL为“/article/1”,那么将匹配`DeleteMapping`并且`PathVariable`接收参数id=1。而`RequestParam`用于接收普通表单方式或者ajax模拟表单提交的参数数据。

```java
@DeleteMapping("/article/{id}")
public @ResponseBody AjaxResponse deleteArticle(@PathVariable Long id) {

@PostMapping("/article")
public @ResponseBody AjaxResponse deleteArticle(@RequestParam Long id) {
```

# 二、接收复杂嵌套对象参数

有一些朋友可能还无法理解`RequestBody`注解存在的真正意义，表单数据提交用`RequestParam`就好了，为什么还要搞出来一个`RequestBody`注解呢？`RequestBody`注解的真正意义在于能够使用对象或者嵌套对象接收前端数据*。

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200416133342.png)

仔细看上面的代码，是一个`paramData`对象里面包含了一个`bestFriend`对象。这种数据结构使用`RequestParam`就无法接收了，`RequestParam`只能接收平面的、一对一的参数。像上文中这种数据结构的参数，就需要我们在java服务端定义两个类，一个类是`ParamData`，一个类是`BestFriend`.

```java
public class ParamData {
    private String name;
    private int id;
    private String phone;
    private BestFriend bestFriend;
    
    public static class BestFriend {
        private String address;
        private String sex;
    }
}
```

- 注意上面代码中省略了GET、SET方法等必要的java plain model元素。
- 注意成员变量名称一定要和JSON属性名称对应上。
- 注意接收不同类型的参数，使用不同的成员变量类型

完成以上动作，我们就可以使用`@RequestBody ParamData paramData`，一次性的接收以上所有的复杂嵌套对象参数了，参数对象的所有属性都将被赋值。

# 三、Http数据转换的原理

大家现在使用JSON都比较普遍了，其方便易用、表达能力强，是绝大部分数据接口式应用的首选。那么如何响应其他的类型的数据？其中的判别原理又是什么？下面就来给大家介绍一下：
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200416134205.png)

- 当一个HTTP请求到达时是一个`InputStream`，通过`HttpMessageConverter`转换为java对象，从而进行参数接收。
- 当对一个HTTP请求进行响应时，我们首先输出的是一个java对象，然后由`HttpMessageConverter`转换为`OutputStream`输出。

当我们在Spring Boot应用中集成了`jackson`的类库之后，如下的一些`HttpMessageConverter`将会被加载。

| 实现类                                 | 功能说明                                                     |
| :------------------------------------- | :----------------------------------------------------------- |
| `StringHttpMessageConverter`           | 将请求信息转为字符串                                         |
| `FormHttpMessageConverter`             | 将表单数据读取到`MultiValueMap`中                            |
| `XmlAwareFormHttpMessageConverter`     | 扩展与`FormHttpMessageConverter`，如果部分表单属性是XML数据，可用该转换器进行读取 |
| `ResourceHttpMessageConverter`         | 读写`org.springframework.core.io.Resource`对象               |
| `BufferedImageHttpMessageConverter`    | 读写`BufferedImage`对象                                      |
| `ByteArrayHttpMessageConverter`        | 读写二进制数据                                               |
| `SourceHttpMessageConverter`           | 读写`java.xml.transform.Source`类型的对象                    |
| `MarshallingHttpMessageConverter`      | 通过Spring的`org.springframework,xml.Marshaller`和`Unmarshaller`读写XML消息 |
| `Jaxb2RootElementHttpMessageConverter` | 通过JAXB2读写XML消息，将请求消息转换为标注的`XmlRootElement`和`XmlType`连接的类中 |
| `MappingJacksonHttpMessageConverter`   | 利用Jackson开源包的`ObjectMapper`读写JSON数据                |
| `RssChannelHttpMessageConverter`       | 读写RSS种子消息                                              |
| `AtomFeedHttpMessageConverter`         | 和`RssChannelHttpMessageConverter`能够读写RSS种子消息        |

根据HTTP协议的Accept和Content-Type属性，以及参数数据类型来判别使用哪一种`HttpMessageConverter`。当使用`RequestBody`或`ResponseBody`时，再结合前端发送的Accept数据类型，会自动判定优先使用`MappingJacksonHttpMessageConverter`作为数据转换器。但是，不仅JSON可以表达对象数据类型，XML也可以。如果我们希望使用XML格式该怎么告知Spring呢，那就要使用到produces属性了。

```java
@GetMapping(value ="/demo",produces = MediaType.APPLICATION_XML_VALUE)
```

这里我们明确的告知了返回的数据类型是`xml`，就会使用`Jaxb2RootElementHttpMessageConverter`作为默认的数据转换器。当然实现XML数据响应比JSON还会更复杂一些，还需要结合`@XmlRootElement`、`@XmlElement`等注解实体类来使用。同理consumes属性你是不是也会用了呢。

# 四、自定义`HttpMessageConverter`

其实绝大多数的数据格式都不需要我们自定义`HttpMessageConverter`，都有第三方类库可以帮助我们实现(包括下文代码中的Excel格式)。但有的时候，有些数据的输出格式并没有类似于Jackson这种类库帮助我们处理，需要我们自定义数据格式。该怎么做?下面代码只是帮助我们理解的一个例子，不要用于生产：

```java
@Service
public class TeamToXlsConverter extends AbstractHttpMessageConverter<Team> {

    private static final MediaType EXCEL_TYPE = MediaType.valueOf("application/vnd.ms-excel");

    TeamToXlsConverter() {
        super(EXCEL_TYPE);
    }

    @Override
    protected Team readInternal(final Class<? extends Team> clazz, final HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
        return null;
    }

    @Override
    protected boolean supports(final Class<?> clazz) {
        return (Team.class == clazz);
    }

    @Override
    protected void writeInternal(final Team team, final HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
        try (final Workbook workbook = new HSSFWorkbook()) {
            final Sheet sheet = workbook.createSheet();
            int rowNo = 0;
            for (final TeamMember member : team.getMembers()) {
                final Row row = sheet.createRow(rowNo++);
                row.createCell(0)
                   .setCellValue(member.getName());
            }
            workbook.write(outputMessage.getBody());
        }
    }
}
```

- 实现`AbstractHttpMessageConverter`接口
- 指定该转换器是针对哪种数据格式的？如上文代码中的"application/vnd.ms-excel"
- 指定该转换器针对那些对象数据类型？如上文代码中的supports函数
- 使用`writeInternal`对数据进行输出处理，上例中是输出为Excel格式。