---
title: SpringBoot 13.服务器推送技术 13.2.服务端推送事件SSE
date: 2022-07-03T14:24:39.246Z
tags: [springboot]
---
# 一、模拟网络支付场景

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200501170746.png)

我们下面用SSE的方式来模拟一下，时序图中三条黑色实线的请求操作。
**注意：在返回最终支付结果的操作，实现了服务端向客户端的事件推送**

# 二、模拟实现

ssetest.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>SSE</title>
</head>
<body>
    <div id = "message"></div>
    <script>
        if (window.EventSource) {
            var source = new EventSource('orderpay');
            innerHTML = '';
            source.addEventListener('message', function(e) {
                innerHTML += e.data + "<br/>";
                document.getElementById("message").innerHTML = innerHTML;
            });

            source.addEventListener('open', function(e) {
                console.log("连接打开.");
            }, false);

            // 响应finish事件，主动关闭EventSource
            source.addEventListener('finish', function(e) {
                console.log("数据接收完毕，关闭EventSource");
                source.close();
                console.log(e);
            }, false);

            source.addEventListener('error', function(e) {
                if (e.readyState == EventSource.CLOSED) {
                    console.log("连接关闭");
                } else {
                    console.log(e);
                }
            }, false);
        } else {
            console.log("你的浏览器不支持SSE");
        }
    </script>

</body>
</html>
```

```java
@Controller
@RequestMapping("sse")
public class SSEControler {

    public static final ConcurrentHashMap<Long,SseEmitter> sseEmitters = new ConcurrentHashMap<>();


    @GetMapping("/test")
    public String ssetest(){
        return "ssetest";
    }

    @GetMapping("/orderpay")
    public SseEmitter orderpay(){
        Long payRecordId = 1L;
        //设置默认的超时时间60秒
        final SseEmitter emitter = new SseEmitter(60 * 1000L);
        try {
            System.out.println("连接建立成功");
            //TODO 这里可以做一些订单保存的操作
            sseEmitters.put(payRecordId,emitter);
        }catch (Exception e){
            emitter.completeWithError(e);
        }

        return emitter;
    }


    @GetMapping("/payback")
    public @ResponseBody String payback (){

        SseEmitter emitter = sseEmitters.get(1L);
        try {
            emitter.send("支付成功");

            System.out.println("发送finish事件");
            emitter.send(SseEmitter.event().name("finish").id("6666").data("哈哈"));
            System.out.println("调用complete");
            emitter.complete();
        } catch (IOException e) {
            emitter.completeWithError(e);
        }
        return "ok";
    }
}
```



对连接超时异常进行全局处理

```java
@ExceptionHandler(AsyncRequestTimeoutException.class)
@ResponseBody
public String handleAsyncRequestTimeoutException(AsyncRequestTimeoutException e) {
    return SseEmitter.event().data("timeout!!").build().stream()
            .map(d -> d.getData().toString())
            .collect(Collectors.joining());
}
```

# 三、访问测试

**http://localhost:8888/sse/test**

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200501171045.png)

**http://localhost:8888/sse/payback**

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200501171226.png)
![img](https://box.kancloud.cn/2276fddae2895f9a7b57abfd1c303e89_1452x329.png)