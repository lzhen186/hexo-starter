---
title: SpringBoot 13.服务器推送技术 13.3.双向实时通信websocket
date: 2022-07-03T14:27:18.836Z
tags: [springboot]
---
# 一、整合websocket

```xml
<!-- 引入websocket依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

开启websocket功能

```java
@Configuration
public class WebSocketConfig {
 
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

# 二、websocket 用法实验

![](https://cdn.jsdelivr.net/gh/krislinzhao/IMGcloud/img/20200501171402.png)
WebSocketServer本节内容的核心代码，websocket服务端代码

1. @ServerEndpoint(value = "/ws/asset")表示websocket的接口服务地址
2. @OnOpen注解的方法，为连接建立成功时调用的方法
3. @OnClose注解的方法，为连接关闭调用的方法
4. @OnMessage注解的方法，为收到客户端消息后调用的方法
5. @OnError注解的方法，为出现异常时调用的方法

```java
/**
 * WebSocket服务端示例
 */
@Component
@Slf4j
@ServerEndpoint(value = "/ws/asset")
public class WebSocketServer {  
  
    private static final AtomicInteger OnlineCount = new AtomicInteger(0);
    // concurrent包的线程安全Set，用来存放每个客户端对应的Session对象。  
    private static CopyOnWriteArraySet<Session> SessionSet = new CopyOnWriteArraySet<>();
  
  
    /** 
     * 连接建立成功调用的方法 
     */  
    @OnOpen
    public void onOpen(Session session) throws IOException {
        SessionSet.add(session);   
        int cnt = OnlineCount.incrementAndGet(); // 在线数加1  
        log.info("有连接加入，当前连接数为：{}", cnt);  
        SendMessage(session, "连接成功");  
    }  
  
    /** 
     * 连接关闭调用的方法 
     */  
    @OnClose
    public void onClose(Session session) {  
        SessionSet.remove(session);  
        int cnt = OnlineCount.decrementAndGet();  
        log.info("有连接关闭，当前连接数为：{}", cnt);  
    }  
  
    /** 
     * 收到客户端消息后调用的方法
     * @param message 客户端发送过来的消息
     */  
    @OnMessage
    public void onMessage(String message, Session session) throws IOException {
        log.info("来自客户端的消息：{}",message);  
        SendMessage(session, "收到消息，消息内容："+message);
    }  
  
    /** 
     * 出现错误
     */  
    public void onError(Session session, Throwable error) {  
        log.error("发生错误：{}，Session ID： {}",error.getMessage(),session.getId());
    }  
  
    /** 
     * 发送消息，实践表明，每次浏览器刷新，session会发生变化。 
     * @param session  session
     * @param message  消息
     */  
    private static void SendMessage(Session session, String message) throws IOException {

        session.getBasicRemote().sendText(String.format("%s (From Server，Session ID=%s)",message,session.getId()));

    }  
  
    /** 
     * 群发消息 
     * @param message  消息
     */  
    public static void BroadCastInfo(String message) throws IOException {
        for (Session session : SessionSet) {  
            if(session.isOpen()){  
                SendMessage(session, message);  
            }  
        }  
    }  
  
    /** 
     * 指定Session发送消息
     * @param sessionId sessionId
     * @param message  消息
     */  
    public static void SendMessage(String sessionId,String message) throws IOException {  
        Session session = null;  
        for (Session s : SessionSet) {  
            if(s.getId().equals(sessionId)){  
                session = s;  
                break;  
            }  
        }  
        if(session!=null){  
            SendMessage(session, message);  
        } else{
            log.warn("没有找到你指定ID的会话：{}",sessionId);  
        }  
    }  
      
} 
```

客户端代码，做几次实验，自然明了代码的意思，先不要看代码，先看效果。
public/wstest/html

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>websocket测试</title>
    <style type="text/css">
        h3,h4{
            text-align:center;
        }
    </style>
</head>
<body>

<h3>WebSocket测试，在<span style="color:red">控制台</span>查看测试信息输出！</h3>
<h4>
    <br>
    http://localhost:8888/api/ws/sendOne?message=单发消息内容&id=none
    <br>
    http://localhost:8888/api/ws/sendAll?message=群发消息内容
</h4>

<h3>请输入要发送给服务器端的消息：</h3><br/>

<input id="text" type="text" />
<button onclick="sendToServer()">发送服务器消息</button>
<button onclick="closeWebSocket()">关闭连接</button>
<br>信息:<span id="message"></span>


<script type="text/javascript">
    var socket;
    if (typeof (WebSocket) == "undefined") {
        console.log("遗憾：您的浏览器不支持WebSocket");
    } else {
        socket = new WebSocket("ws://localhost:8888/ws/asset");
        //连接打开事件
        socket.onopen = function() {
            console.log("Socket 已打开");
            socket.send("消息发送测试(From Client)");
        };
        //收到消息事件
        socket.onmessage = function(msg) {
            document.getElementById('message').innerHTML += msg.data + '<br/>';
        };
        //连接关闭事件
        socket.onclose = function() {
            console.log("Socket已关闭");
        };
        //发生了错误事件
        socket.onerror = function() {
            alert("Socket发生了错误");
        }

        //窗口关闭时，关闭连接
        window.unload=function() {
            socket.close();
        };
    }

    //关闭连接
    function closeWebSocket(){
        socket.close();
    }

    //发送消息给服务器
    function sendToServer(){
        var message = document.getElementById('text').value;
        socket.send(message);
    }
</script>

</body>
</html>
```

### 测试页面

http://localhost:8888/wstest.html

# 三、服务端广播与指定session消息发送

```java
@RestController
@RequestMapping("/api/ws")
public class WebSocketController {  
  

    /** 
     * 群发消息内容 
     * @param message 消息内容
     */
    @RequestMapping(value="/sendAll", method=RequestMethod.GET)
    AjaxResponse sendAllMessage(@RequestParam String message){
        try {  
            WebSocketServer.BroadCastInfo(message);
        } catch (IOException e) {
            throw new CustomException(CustomExceptionType.SYSTEM_ERROR,"群发消息失败");
        }  
        return AjaxResponse.success();
    }  

    /** 
     * 指定会话ID发消息 
     * @param message 消息内容 
     * @param id 连接会话ID
     */
    @RequestMapping(value="/sendOne", method=RequestMethod.GET)
    AjaxResponse sendOneMessage(@RequestParam String message,@RequestParam String id){
        try {  
            WebSocketServer.SendMessage(id,message);  
        } catch (IOException e) {
            throw new CustomException(CustomExceptionType.SYSTEM_ERROR,"指定会话ID发消息失败");
        }  
        return AjaxResponse.success();
    }  
}  
```