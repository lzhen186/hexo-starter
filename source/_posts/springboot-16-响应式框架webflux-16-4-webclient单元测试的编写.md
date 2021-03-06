---
title: SpringBoot 16.响应式框架webflux 16.4.webclient单元测试的编写
date: 2022-07-04T01:01:37.913Z
tags: [springboot]
---
# 一、使用@SpringBootTest测试

@SpringBootTest也可用于测试 Spring WebFlux 控制器，它会将整个应用启动，并注入WebTestClient ，CityWebFluxController以及它所依赖的CityHandler.

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class CityWebFluxControllerTest {

    @Autowired
    private WebTestClient webClient;

    private static Map<String, City> cityMap = new HashMap<>();

    @BeforeClass
    public static void setup() throws Exception {
        City bj= new City();
        bj.setId(1L);
        bj.setProvinceId(2L);
        bj.setCityName("BJ");
        bj.setDescription("welcome to beijing");
        cityMap.put("BJ", bj);
    }

    @Test
    public void testSave() throws Exception {

        City expectCity = webClient.post().uri("/city")
                .contentType(MediaType.APPLICATION_JSON)
                .body(BodyInserters.fromObject(cityMap.get("BJ")))
                .exchange()
                .expectStatus().isOk()
                .expectBody(City.class).returnResult().getResponseBody();

        Assert.assertNotNull(expectCity);
        Assert.assertEquals(expectCity.getId(), cityMap.get("BJ").getId());
        Assert.assertEquals(expectCity.getCityName(), cityMap.get("BJ").getCityName());
    }

}
```

WebTestClient.post() 方法构造了 POST 测试请求，并使用 uri 指定路由。
expectStatus() 用于验证返回状态是否为 ok()，即 200 返回码。
expectBody(City.class) 用于验证返回对象体是为 City 对象，并利用 returnResult 获取对象。
Assert 是以前我们常用的断言方法验证测试结果。

# 二、使用@WebFluxTest和Mockito测试

@WebFluxTest 注入了 WebTestClient 对象，只用于测试 WebFlux 控制器@Controller，好处是快速，并不会将所有的Component都注入到 容器。
所以Controller层依赖的CityHandler我们需要自己Mock。如果忘记了请回看第二章的内容。

```java
@RunWith(SpringRunner.class)
@WebFluxTest
public class CityWebFluxControllerTest {

    @Autowired
    private WebTestClient webTestClient;

    @MockBean
    private CityHandler cityHandler;

    private static Map<String, City> cityMap = new HashMap<>();

    @BeforeClass
    public static void setup() throws Exception {
        City bj= new City();
        bj.setId(1L);
        bj.setProvinceId(2L);
        bj.setCityName("BJ");
        bj.setDescription("welcome to beijing");
        cityMap.put("BJ", bj);
    }

    @Test
    public void testSave() throws Exception {

        Mockito.when(this.cityHandler.save(cityMap.get("BJ")))
                .thenReturn(Mono.create(cityMonoSink -> cityMonoSink.success(cityMap.get("BJ"))));

        City expectCity = webTestClient.post().uri("/city")
                .contentType(MediaType.APPLICATION_JSON)
                .body(BodyInserters.fromObject(cityMap.get("BJ")))
                .exchange()
                .expectStatus().isOk()
                .expectBody(City.class).returnResult().getResponseBody();



        Assert.assertNotNull(expectCity);
        Assert.assertEquals(expectCity.getId(), cityMap.get("BJ").getId());
        Assert.assertEquals(expectCity.getCityName(), cityMap.get("BJ").getCityName());
    }

}
```
