# 握手时出错403
浏览器控制台错误提示
```
Error during WebSocket handshake: Unexpected response code: 403
```
## 配置示例
- websocket 注册
```
@Override
public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
    registry.addHandler(socketHandler, "/test").addInterceptors(new SystemInfoSocketHandshakeInterceptor());
    registry.addHandler(socketHandler, "/sockjs/test").addInterceptors(new SystemInfoSocketHandshakeInterceptor())
            .withSockJS();
}
```

- 握手拦截器
```java
@Configuration
public class SystemInfoSocketHandshakeInterceptor implements HandshakeInterceptor {

    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object
            > attributes) {
        log.info("socket beforeHandshake..");
        if (request instanceof ServletServerHttpRequest) {
            ServletServerHttpRequest servletRequest = (ServletServerHttpRequest) request;
            HttpSession session = servletRequest.getServletRequest().getSession(false);
            // 业务处理
        }
        return true;
    }

    @Override
    public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception exception) {
        log.info("socket beforeHandshake..");
    }
}
```
- 客户端连接
```js
var wsServer = "ws://127.0.0.1:8080";
var webSocket;
if ('WebSocket' in window || 'MozWebSocket' in window) {
    webSocket = new WebSocket(wsServer + "/test");
} else {
    webSocket = new SockJS(wsServer + "/sockjs/test");
}

webSocket.onerror = function (event) {
    console.log("websockt连接错误")
};
```

## 分析
- 连接失败
- 访问拦截
- nginx代理
- 跨域

## 解决

### 连接问题
如果是连接上面的问题，一般看一下配置是否正确

注意版本问题

### 访问拦截
在拦截器中，解除拦截
```
@Bean
public WebMvcConfigurer corsConfigurer() {
    return new WebMvcConfigurer() {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/**").allowedOrigins("*");
        }
    };
}
```

### nginx代理
在nginx中配置
`proxy_set_header Upgrade $http_upgrade` 
`proxy_set_header Connection "upgrade"`
`proxy_set_header Host $host`
三个配置

完整配置如下:
```conf
location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

### 跨域
添加`setAllowedOrigins("*")`解决跨域问题
```
@Override
public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
    registry.addHandler(socketHandler, "/test")
            .addInterceptors(new SystemInfoSocketHandshakeInterceptor())
            .setAllowedOrigins("*");
    registry.addHandler(socketHandler, "/sockjs/test")
            .addInterceptors(new SystemInfoSocketHandshakeInterceptor())
            .setAllowedOrigins("*")
            .withSockJS();
}
```

## 资料相关
- [Spring WebSockets](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket)

---
收录时间: 2021/02/03

<Vssue :title="$title" />