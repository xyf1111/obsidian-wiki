---
title: "网络 08 - HTTP2与HTTP3及WebSocket"
date: 2026-06-11
tags: [网络]
---

# 网络 08 - HTTP/2、HTTP/3 与 WebSocket

## HTTP 协议演进

| 版本 | 年份 | 核心改进 |
|------|------|----------|
| HTTP/1.1 | 1997 | 持久连接、管道化、分块传输 |
| HTTP/2 | 2015 | 二进制分帧、多路复用、头部压缩、服务端推送 |
| HTTP/3 | 2022 | QUIC（基于 UDP）、0-RTT、无队头阻塞 |

## HTTP/1.1 的问题

1. **队头阻塞（HOL Blocking）** — 同一连接上请求必须按顺序返回
2. **头部冗余** — 每次请求发送相同的头部，浪费带宽
3. **连接数过多** — 浏览器为每个域名开 6 个连接

## HTTP/2 改进

### 二进制分帧

HTTP/1.1 是文本协议，HTTP/2 改为二进制帧，更高效解析。

### 多路复用

```
HTTP/1.1:  ┌──请求1──┐──请求2──┐──请求3──┐ （串行）
HTTP/2:   ┌─请求1─┐┌─请求2─┐┌─请求3─┐   （并发帧）
```

### 头部压缩（HPACK）

- 使用静态表 + 动态表 + Huffman 编码
- 减小头部体积 80%+

### 服务端推送（Server Push）

服务端可主动推送客户端可能需要的资源（如 HTML 中的 CSS/JS）。

## HTTP/3（QUIC）

### QUIC 协议

基于 UDP 实现的可靠传输协议。

| HTTP/2 | HTTP/3 |
|--------|--------|
| 基于 TCP | 基于 UDP（QUIC） |
| TCP 队头阻塞 | 无队头阻塞（丢包只影响单个流） |
| 1-3 RTT 连接 | 0-1 RTT 连接 |
| 内核态 TLS | 用户态加密 |

## WebSocket

### 什么是 WebSocket

基于 TCP 的**全双工通信**协议，浏览器与服务器可以实时双向推送数据。

### 建立过程

```
客户端 → HTTP 升级请求 → 服务端 → 101 切换协议 → 全双工通信
```

```http
GET /chat HTTP/1.1
Host: example.com
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
```

### 应用场景

| 场景 | 说明 |
|------|------|
| **实时聊天** | 即时消息推送 |
| **股票行情** | 价格实时更新 |
| **协作编辑** | 多人同时编辑文档 |
| **游戏** | 实时对战 |
| **通知推送** | 替代轮询 |

### HTTP 短轮询 vs WebSocket

| 对比 | 短轮询 | WebSocket |
|------|--------|-----------|
| 延迟 | 受轮询间隔影响 | 实时 |
| 带宽 | 每次建立 HTTP 连接 | 一次连接，持续通信 |
| 服务端压力 | 高（大量无效请求） | 低（事件触发） |

### JavaScript 客户端 API

```js
// 建立连接
const ws = new WebSocket('ws://localhost:8080/channel/echo');

// 接收消息
ws.addEventListener('message', (event) => {
    console.log('收到:', event.data);
});
// 或 ws.onmessage = (event) => { ... };

// 发送消息
ws.send('Hello, server!');

// 关闭连接
ws.close();

// 连接状态
if (ws.readyState === WebSocket.OPEN) { /* 连接打开 */ }

// 事件监听
ws.addEventListener('open', (e) => console.log('已连接'));
ws.addEventListener('close', (e) => console.log('已关闭'));
ws.addEventListener('error', (e) => console.error('异常:', e.error));
```

> WebSocket 没有跨域限制，可在任意页面控制台直接连接测试。

## Spring Boot WebSocket 实战

Spring Boot 整合 WebSocket 有两种方式：Jakarta EE 规范（`@ServerEndpoint`）和 Spring WebSocket 模块。

### 方式一：Jakarta EE @ServerEndpoint（推荐）

**添加依赖：**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

**开发端点：**
```java
@ServerEndpoint(value = "/channel/echo")
public class EchoChannel {

    private Session session;

    @OnOpen
    public void onOpen(Session session, EndpointConfig config) {
        this.session = session;
    }

    @OnMessage
    public void onMessage(String message) throws IOException {
        if ("bye".equalsIgnoreCase(message)) {
            this.session.close(
                new CloseReason(CloseReason.CloseCodes.NORMAL_CLOSURE, "Bye"));
            return;
        }
        this.session.getAsyncRemote()
            .sendText("[" + Instant.now().toEpochMilli() + "] Hello " + message);
    }

    @OnClose
    public void onClose(CloseReason reason) { /* 断开处理 */ }

    @OnError
    public void onError(Throwable t) throws IOException {
        this.session.close(new CloseReason(
            CloseReason.CloseCodes.UNEXPECTED_CONDITION, t.getMessage()));
    }
}
```

**配置注册：**
```java
@Configuration
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

也可在端点上加 `@Component` 让 Spring 自动扫描注册。

### 注入 Bean 的注意事项

`@ServerEndpoint` 实例由 WebSocket 容器（Tomcat）创建，**不受 Spring 管理**，所以 `@Autowired` 注入会空指针。正确做法：

```java
@ServerEndpoint("/channel/echo")
@Component
public class EchoChannel implements ApplicationContextAware {

    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext ctx) {
        applicationContext = ctx;
    }

    private UserService userService;

    @OnOpen
    public void onOpen(Session session, EndpointConfig config) {
        this.session = session;
        // 连接创建时从 ApplicationContext 获取 Bean
        this.userService = applicationContext.getBean(UserService.class);
    }
}
```

`onOpen` 在整个连接生命周期只执行一次，不会带来通信性能损耗。

### 方式二：Spring WebSocket 模块

通过 `WebSocketHandler` 接口实现，可无缝整合 Spring Security 等模块，代码略（参考 Spring 官方文档）。

### WebSocket 连接状态码

| 状态码 | 含义 |
|--------|------|
| 1000 | 正常关闭（NORMAL_CLOSURE） |
| 1009 | 消息体超过 `@OnMessage` 的 `maxMessageSize` 限制 |
| 1011 | 服务端意外错误（UNEXPECTED_CONDITION） |

### 发送文件（服务端推送）

```java
// 服务端主动发送（基于 Session）
session.getAsyncRemote().sendText("消息内容");
session.getBasicRemote().sendText("同步消息");

// 群发管理（使用 CopyOnWriteArraySet 维护连接池）
private static CopyOnWriteArraySet<WebSocketServer> connections = new CopyOnWriteArraySet<>();
```

### 聊天室模型：点对点 vs 服务器中转

- **点对点直连**：每人与其他所有人各建一条连接，2000 人聊天室每人要建 1999 条连接，资源浪费严重，且无法统一控制用户、保证连接安全
- **服务器中转（聊天室基本模型）**：所有客户端连接同一服务器，消息统一发给服务器，由服务器决定转发/广播给哪些用户——每人只需一条连接

实现协议可选 HTTP 或 WebSocket；HTTP 单向、不够实时，实时聊天通常用 WebSocket。

> **WebSocket ≠ Socket**：Socket 不是协议，只是对 TCP/UDP 等协议的抽象封装接口，方便编程；WebSocket 是应用层协议，收发消息时只是**模拟**了 Socket 的实现。

### Socket.IO 聊天室实战（Node.js）

Socket.IO 是 Node.js 优秀的 WebSocket 封装库，一个 JS 语言同时实现前后端，官方提供聊天室 Demo（入门体验优于 Vertx/Netty 等 Java 方案）。

**服务端**（Express + Socket.IO）：监听事件（事件名可自定义），收到消息后广播给其他客户端：

```js
// 监听 chat message 事件
socket.on('chat message', (msg) => {
  // 收到消息后广播到其他客户端
  socket.broadcast.emit('chat message', msg);
});
```

**客户端**：点击发送按钮时触发事件，服务端广播后其余客户端监听插入 DOM：

```js
// 发送消息
socket.emit('chat message', '用户输入的消息');

// 接收并展示消息
socket.on('chat message', function(msg) {
  var item = document.createElement('li');
  item.textContent = msg;
  messages.appendChild(item);
  window.scrollTo(0, document.body.scrollHeight);
});
```

完整流程：Express 建 Node.js 服务 → 前端页面发送消息界面 → 前后端整合 Socket.IO → 发送/广播 → 接收方插入 DOM。可扩展到实时视频评论监控等场景；企业级聊天室仍需考虑鉴权、消息持久化、水平扩展等。
