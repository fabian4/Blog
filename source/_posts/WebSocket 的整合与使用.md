---
title: WebSocket 的整合与使用
date: 2020/11/10
description: WebSocket 的整合与使用
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/QmFG1Tc5YxgwbUe.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/QmFG1Tc5YxgwbUe.jpg'
categories:
  - Spring
tags:
  - SpringBoot
  - WebSocket
abbrlink: 45659
---

# WebSocket 的整合与使用

## 一、WebSocket 介绍

> 初次接触 WebSocket 的人，都会问同样的问题：我们已经有了 HTTP 协议，为什么还需要另一个协议？它能带来什么好处？
>
> 答案很简单，因为 HTTP 协议有一个缺陷：通信只能由客户端发起。
>
> **HTTP 协议做不到服务器主动向客户端推送信息**

HTTP这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。我们只能使用**轮询**：每隔一段时候，就发出一个询问，了解服务器有没有新的信息。最典型的场景就是聊天室。**轮询**的效率低，非常浪费资源（因为必须不停连接，或者 HTTP 连接始终打开）。因此，工程师们一直在思考，有没有更好的方法。WebSocket 就是这样发明的。

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。

WebSocket 其他特点包括：

- 建立在 TCP 协议之上，服务器端的实现比较容易。
- 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
- 数据格式比较轻量，性能开销小，通信高效。
- 可以发送文本，也可以发送二进制数据。
- 没有同源限制，客户端可以与任意服务器通信。
- 协议标识符是`ws`（如果加密，则为`wss`），服务器网址就是 URL。

## 二、整合 SpringBoot 项目

### 1. 导入 pom 依赖

~~~xml
 <!-- Spring boot websocket -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
~~~

### 2. 添加 WebSocket 配置类

~~~Java
@Configuration
public class WebSocketConfig {

    // 初始到 Spring 容器
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
~~~

### 3. 配置 WebSocket

~~~java
@Component
@ServerEndpoint("/webSocket/{sid}")
public class WebSocketServer {

    private static final Logger log = LoggerFactory.getLogger(WebSocketServer.class);

    /**
     * concurrent包的线程安全Set，用来存放每个客户端对应的MyWebSocket对象。
     */
    private static CopyOnWriteArraySet<WebSocketServer> webSocketSet = new CopyOnWriteArraySet<>();

    /**
     * 与某个客户端的连接会话，需要通过它来给客户端发送数据
     */
    private Session session;

    /**
     * 接收sid
     */
    private String sid="";
    /**
     * 连接建立成功调用的方法
     * */
    @OnOpen
    public void onOpen(Session session,@PathParam("sid") String sid) throws IOException {
        this.session = session;
        log.info("客户端上线：{}", sid);
        // 发送测试信息
        session.getBasicRemote().sendText("test");
        System.out.println(Arrays.toString(webSocketSet.toArray()));
        //如果存在就先删除一个，防止重复推送消息
        webSocketSet.removeIf(webSocket -> webSocket.sid.equals(sid));
        webSocketSet.add(this);
        this.sid=sid;
    }

    /**
     * 连接关闭调用的方法
     */
    @OnClose
    public void onClose() {
        webSocketSet.remove(this);
    }

    /**
     * 收到客户端消息后调用的方法
     * @param message 客户端发送过来的消息
     */
    @OnMessage
    public void onMessage(String message, Session session) {
        log.info("收到来"+sid+"的信息:"+message);
    }

    @OnError
    public void onError(Session session, Throwable error) {
        log.error("发生错误");
        error.printStackTrace();
    }
    
    /**
     * 实现服务器主动推送
     */
    private void sendMessage(String message) throws IOException {
        log.info("推送消息到"+sid+"，推送内容:"+ message);
        this.session.getBasicRemote().sendText(message);
    }


    /**
     * 群发自定义消息
     * */
    public static void sendInfo(String message,@PathParam("sid") String sid) {
        for (WebSocketServer item : webSocketSet) {
            try {
                //这里可以设定只推送给这个sid的，为null则全部推送
                if(sid==null) {
                    item.sendMessage(message);
                }else if(item.sid.equals(sid)){
                    item.sendMessage(message);
                }
            } catch (IOException ignored) { }
        }
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        WebSocketServer that = (WebSocketServer) o;
        return Objects.equals(session, that.session) &&
                Objects.equals(sid, that.sid);
    }

    @Override
    public int hashCode() {
        return Objects.hash(session, sid);
    }
}
~~~

## 三、 WebSocket 通信测试

启动项目后，我们的 通信地址是 ws://127.0.0.1:8000/webSocket/你的客户机id

我们找一个 webSocket 通信测试网站 http://www.easyswoole.com/wstool.html

连接我们的通信地址就能收到消息了

![image-20201210180624727](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/cC3M76aKG5eAUpy.png)

我们在 测试网站 测试我们是 Socket 通信时，服务器也可以接受到信息

![image-20201210180915972](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/W5RsZwDxkbEPFa6.png)