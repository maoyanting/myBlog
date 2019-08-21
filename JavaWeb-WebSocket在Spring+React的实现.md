---
title: JavaWeb-WebSocket在Spring+React的实现
date: 2018-01-05 11:27:46
categories: 
- JavaWeb
tags:
- Spring
- WebSocket
---


## 背景
**先说一下别的轮询方式：**
ajax轮询：让浏览器隔个几秒就发送一次请求，询问服务器是否有新信息。
long poll：浏览器发起连接后，如果没消息，就一直不返回Response给浏览器。直到有消息才返回，返回完之后，浏览器再次建立连接，周而复始。

ajax轮询 需要服务器有很快的处理速度和资源。
long poll 需要有很高的并发。

**再说webSocket**，先必须记住，**webSocket是一个协议**
我们使用的应该说是webSocket API

WebSocket是基于TCP的独立的协议。
和HTTP的唯一关联就是HTTP服务器需要发送一个“Upgrade”请求，即101 Switching Protocol到HTTP服务器，然后由服务器进行协议转换

//简单来说，就是我是从TCP那边继承过来的，干活需要依靠HTTP先帮我连接上
## 前端部分：
**最简单的实现：**

```
const ws = new WebSocket("ws://echo.websocket.org/ws");

ws.onopen = function(evt) {
  console.log("Connection open ...");
  ws.send("Hello WebSockets!");
};

ws.onmessage = function(evt) {
  console.log( "Received Message: " + evt.data);
  ws.close();
};

ws.onclose = function(evt) {
  console.log("Connection closed.");
};
```
## 后端部分
先放[官方文档](https://docs.spring.io/spring/docs/4.3.12.RELEASE/spring-framework-reference/htmlsingle/#websocket)(我用的是spring 4.3.13.RELEASE)

其实基本上就4四个部分：
1. 添加依赖
2. Handler类
3. 拦截器
4. 配置

### 添加依赖
```
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-websocket</artifactId>
      <version>4.3.13.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.apache.tomcat.embed</groupId>
      <artifactId>tomcat-embed-websocket</artifactId>
      <version>8.0.23</version>
      <scope>provided</scope>
    </dependency>
```

### Handler类
 Handler类一般implementing `WebSocketHandler` 或者extending `TextWebSocketHandler` or `BinaryWebSocketHandler`

```
public interface WebSocketHandler {
	void afterConnectionEstablished(WebSocketSession session) throws Exception;
	void handleMessage(WebSocketSession session, WebSocketMessage<?> message) throws Exception;
	void handleTransportError(WebSocketSession session, Throwable exception) throws Exception;
	void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception;
	boolean supportsPartialMessages();
}
```



### 拦截器
这里你可以选择:
#### 不实现自己的拦截器

使用spring给你的`HttpSessionHandshakeInterceptor`

Spring官方文档是有写：

> easiest way to customize the initial HTTP WebSocket handshake request is through a HandshakeInterceptor, which exposes "before" and "after" the handshake methods. Such an interceptor can be used to preclude the handshake or to make any attributes available to the WebSocketSession.

通俗的说

普通request的session和webSocket的`WebSocketSession`是不一样的。

`HttpSessionHandshakeInterceptor`会帮你把session中的attributes拿出来，放到`WebSocketSession`里面去

#### 实现自己的拦截器

```
@Component
public class WebSocketHandshakeInterceptor extends BaseController implements HandshakeInterceptor {
    private static final Logger logger = LoggerFactory.getLogger(WebSocketHandshakeInterceptor.class);

    @Override
    public boolean beforeHandshake(ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Map<String, Object> map) throws Exception {
        if (serverHttpRequest instanceof ServletServerHttpRequest) {
            ServletServerHttpRequest servletRequest = (ServletServerHttpRequest) serverHttpRequest;
            HttpSession session = servletRequest.getServletRequest().getSession(false);
            User user = getSessionUser(servletRequest.getServletRequest());
            if (session != null) {
                map.put(WEBSOCKET_USERID, user.getUserId());
            }else {
                logger.error("session为空");
                return false;
            }
        }
        return true;
    }

    @Override
    public void afterHandshake(ServerHttpRequest serverHttpRequest, ServerHttpResponse serverHttpResponse, WebSocketHandler webSocketHandler, Exception e) {

    }
}
```

其中，beforeHandshake()方法，在调用 handler 前调用。
常用来注册用户信息，绑定 WebSocketSession，在 handler 里根据用户信息获取WebSocketSession发送消息。
注意：⚠️需要把这个拦截器注册让spring识别

//这里获取的user是之前在用户登录的时候就把user放入了session中。
继承的controller里面有getSessionUser方法。

### 配置
#### 编写 `WebSocketConfig` 配置类，实现 `WebSocketConfigurer`接口
```
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {
    @Autowired
    private SystemWebSocketHandler systemWebSocketHandler;
    @Autowired
    private WebSocketHandshakeInterceptor webSocketHandshakeInterceptor;

    /**
     * 注册实现类，设置访问WebSocket的地址
     * 注册拦截器
     *
     * @param webSocketHandlerRegistry
     */
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry webSocketHandlerRegistry) {
        webSocketHandlerRegistry.addHandler(systemWebSocketHandler, "/ws")
                .addInterceptors(webSocketHandshakeInterceptor).setAllowedOrigins("http://localhost:8000");
    }
}
```


实现` WebSocketConfigurer` 接口，重写 `registerWebSocketHandlers` 方法，这是一个核心实现方法，配置 `websocket` 入口，允许访问的域、注册 Handler、SockJs 支持和拦截器。
##### addHandler()
注册和路由的功能，当客户端发起 websocket 连接，把 /path 交给对应的 handler 处理，而不实现具体的业务逻辑，可以理解为收集和任务分发中心。
##### addInterceptors()
顾名思义就是为 handler 添加拦截器，
可以用spring给的`HttpSessionHandshakeInterceptor`，
```
addInterceptors(new HttpSessionHandshakeInterceptor())
```
也可以如上文，使用自己写的拦截器
##### setAllowedOrigins(String[] domains)
允许指定的域名或IP(含端口号)建立长连接，如果只允许自家域名访问，这里轻松设置。如果不限时使用"*"号，如果指定了域名，则必须要以http或https开头。
//我这里是因为我的前端项目通过代理访问后端的接口所以需要设置。

#### 通过 XML 配置文件添加配置
```
<bean id="chatHandler" class="com.websocket.SystemWebSocketHandler"/>
<websocket:handlers>
    <!-- 配置消息处理bean和路径的映射关系 -->
    <websocket:mapping path="/ws" handler="chatHandler"/>
    <!-- 配置握手拦截器 -->
    <websocket:handshake-interceptors>
        <bean class="com.websocket.WebSocketHandshakeInterceptor"/>
    </websocket:handshake-interceptors>
    <!-- 开启sockjs，去掉则关闭sockjs -->
    <websocket:sockjs/>
</websocket:handlers>
<!-- 配置websocket消息的最大缓冲区长度，可以不配 -->
<bean class="org.springframework.web.socket.server.standard.ServletServerContainerFactoryBean">
    <property name="maxTextMessageBufferSize" value="8192"/>
    <property name="maxBinaryMessageBufferSize" value="8192"/>
</bean>
```

## 开启和关闭webSocket
### 1 开启
当运行到前端
```
const ws = new WebSocket("ws://echo.websocket.org/ws");
```

这一行代码的时候，前端就发起和后端的webSocket连接，
会先触发拦截器
后端连接上后，会调用`afterConnectionEstablished`里面的方法
前端会调用`.open`方法
至此，webSocket开启成功。
### 2 运行
**前端---->后端**
前端调用`.send()`方法发送信息
后端调用`handleMessage`方法接收并消息
**后端---->前端**
后端调用`webSocketSession.sendMessage(message)`方法发送信息
前端调用`.onmessage()`方法接收信息
### 3 关闭
前端调用`.close()`方法