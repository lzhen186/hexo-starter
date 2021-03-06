---
title: SpringBoot 02.RESTful接口实现及测试 2.5.1. 使用Swagger2构建API文档
date: 2022-07-02T11:29:02.093Z
tags: [springboot]
---
# 一、为什么要发布API接口文档

当下很多公司都采取前后端分离的开发模式，前端和后端的工作由不同的工程师完成。在这种开发模式下，维护一份及时更新且完整的API 文档将会极大的提高我们的工作效率。传统意义上的文档都是后端开发人员使用word编写的，相信大家也都知道这种方式很难保证文档的及时性，这种文档久而久之也就会失去其参考意义，反而还会加大我们的沟通成本。而 Swagger 给我们提供了一个全新的维护 API 文档的方式，下面我们就来了解一下它的优点：

- 代码变，文档变。只需要少量的注解，Swagger 就可以根据代码自动生成 API 文档，很好的保证了文档的时效性。
- 跨语言性，支持 40 多种语言。
- Swagger UI 呈现出来的是一份可交互式的 API 文档，我们可以直接在文档页面尝试 API 的调用，省去了准备复杂的调用参数的过程。
- 还可以将文档规范导入相关的工具（例如 SoapUI）, 这些工具将会为我们自动地创建自动化测试。

# 二、整合swagger2生成文档

首先通过maven坐标引入swagger相关的类库。

```xml
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.6.1</version>
</dependency>

<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.6.1</version>
</dependency>
```

然后通过java Config对Swagger2进行配置。

```java
@Configuration
@EnableSwagger2 
public class Swagger2 {

    private ApiInfo apiInfo() {
		return new ApiInfoBuilder()
				.title("springboot利用swagger构建api文档")
				.description("简单优雅的restfun风格")
				.termsOfServiceUrl("http://www.krislin.club")
				.version("1.0")
				.build();
    }

	@Bean
	public Docket createRestApi() {
		return new Docket(DocumentationType.SWAGGER_2)
                        .apiInfo(apiInfo())
                        .select()
                        //扫描basePackage包下面的“/rest/”路径下的内容作为接口文档构建的目标
                        .apis(RequestHandlerSelectors.basePackage("club.krislin.bootlaunch"))
                        .paths(PathSelectors.regex("/rest/.*"))
                        .build();
	}
	
	
}
```

- `@EnableSwagger2` 注解表示开启`SwaggerAPI`文档相关的功能
- 在`apiInfo`方法中配置接口文档的title(标题)、描述、`termsOfServiceUrl`（服务协议）、版本等相关信息
- 在`createRestApi`方法中，`basePackage`表示扫描哪个package下面的Controller类作为API接口文档内容范围
- 在`createRestApi`方法中，paths表示哪一个请求路径下控制器映射方法，作为API接口文档内容范围

集成完成之后，做一下访问验证：http://localhost:8888/swagger-ui.html，如下：

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200416193906.png)
swagger不仅提供了静态的接口文档的展示，还提供了执行接口方法测试的功能。在下图中填入接口对应的参数，点击“try it out"就可以实现接口请求的发送与响应结果的展示。
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200416194054.png)

# 三、书写swagger注解

通常情况下Controller类及方法书写了swagger注解，就不需要写java注释了。一个成熟的团队，前端人员根据英文方法的名称和参数名称就能知道方法的作用，前提是代码开发者认真的为接口及参数起英文名。通过团队内推广RESTful接口的设计原则和良好的统一的交互规范（本专栏之前已经介绍过），就能知道响应结果的含义。这也是一种“约定大于配置”的体现。

当然，如果你的团队没有“约定“，那么就需要“配置”来做文档说明。我通常把这个过程叫做“为接口功能添加注释”。写法如下：

```java
@ApiOperation(value = "添加文章", notes = "添加新的文章", tags = "Article",httpMethod = "POST")
@ApiImplicitParams({
        @ApiImplicitParam(name = "title", value = "文章标题", required = true, dataType = "String"),
        @ApiImplicitParam(name = "content", value = "文章内容", required = true, dataType = "String"),
        @ApiImplicitParam(name = "author", value = "文章作者", required = true, dataType = "String")
})
@ApiResponses({
        @ApiResponse(code=200,message="成功",response=AjaxResponse.class),
})
@PostMapping("/article")
public @ResponseBody  AjaxResponse saveArticle(
        @RequestParam(value="title") String title,  //参数1
        @RequestParam(value="content") String content,//参数2
        @RequestParam(value="author") String author,//参数3
) {
```

swagger注释添加完成之后，接口文档变成如下的样子（含有中文说明）：
![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img//20200416194139.png)

swagger注解详细介绍:

```
@Api：用在Controller控制器类上
     属性tags="说明该类的功能及作用"

@ApiOperation：用在Controller控制器类的请求的方法上
    value="说明方法的用途、作用"
    notes="方法的备注说明"

@ApiImplicitParams：用在请求的方法上，表示一组参数说明
    @ApiImplicitParam：请求方法中参数的说明
        name：参数名
        value：参数的汉字说明、解释、用途
        required：参数是否必须传，布尔类型
        paramType：参数的类型，即参数存储位置或提交方式
            · header --> Http的Header携带的参数的获取：@RequestHeader
            · query --> 请求参数的获取：@RequestParam   （如上面的例子）
            · path（用于restful接口）--> 请求参数的获取：@PathVariable
            · body（不常用）
            · form（不常用）    
        dataType：参数类型，默认String，其它值dataType="Integer"       
        defaultValue：参数的默认值

@ApiResponses：用在控制器的请求的方法上，对方法的响应结果进行描述
    @ApiResponse：用于表达一个响应信息
        code：数字，例如400
        message：信息，例如"请求参数没填好"
        response：响应结果封装类，如上例子中的AjaxResponse.class

@ApiModel：value=“通常用在描述@RequestBody和@ResponseBody注解修饰的接收参数或响应参数实体类”
    @ApiModelProperty：value="实体类属性的描述"
```

# 四、生产环境下如何禁用swagger2

我们的文档通常是在团队内部观看及使用的，不希望发布到生产环境让用户看到。

- 禁用方法1：使用注解`@Profile({"dev","test"})` 表示在开发或测试环境开启，而在生产关闭。
- 禁用方法2：使用注解`@ConditionalOnProperty(name = "swagger.enable", havingValue = "true")` 然后在测试配置或者开发配置中 添加 `swagger.enable = true` 即可开启，生产环境不填则默认关闭Swagger.

更多使用细节可以参考：
https://www.ibm.com/developerworks/cn/java/j-using-swagger-in-a-spring-boot-project/index.html