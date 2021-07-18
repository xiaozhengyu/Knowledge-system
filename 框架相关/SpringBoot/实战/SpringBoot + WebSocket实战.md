# SpringBoot + WebSocket实战

参考博客：https://www.cnblogs.com/boboooo/p/13589467.html

WebSocket，干什么用的？我们有了HTTP，为什么还要用WebSocket？很多同学都会有这样的疑问。我们先来看一个场景，大家的手机里都有微信，在微信中，只要有新的消息，这个联系人的前面就会有一个红点，这个需求要怎么实现呢？大家思考3秒钟。哈哈，最简单，最笨的方法就行客户端轮询，在微信的客户端每隔一段时间（比如：1s或者2s），向服务端发送一个请求，查询是否有新的消息，如果有消息就显示红点。这种方法是不是太笨了呢？每次都要客户端去发起请求，难道就不能从服务端发起请求吗？这样客户端不就省事了吗。再看看股票软件，每个股票的当前价格都是实时的，这我们怎么做，每个一秒请求后台查询当前股票的价格吗？这样效率也太低了吧，而且时效性也很低。这就需要我们今天的主角WebSocket去实现了。

## 什么是WebSocket

WebSocket协议，[RFC 6455](https://tools.ietf.org/html/rfc6455)这个大家有兴趣可以看看，太深，太底层。它是通过一个TCP连接，在客服端与服务端之间建立的一个全双工、双向的通信渠道。它是一个不同于HTTP的TCP协议，但是它通过HTTP工作。它的默认端口也是80和443，和HTTP是一样的。

一个WebSocket的交互开始于一个HTTP请求，这是一个握手请求，这个请求中包含一个Upgrade请求头，具体如下：

```
GET /spring-websocket-portfolio/portfolio HTTP/1.1
Host: localhost:8080
Upgrade: websocket 
Connection: Upgrade 
Sec-WebSocket-Key: Uc9l9TMkWGbHFD2qnFHltg==
Sec-WebSocket-Protocol: v10.stomp, v11.stomp
Sec-WebSocket-Version: 13
Origin: http://localhost:8080
```

我们看到的第3行和第4行就是这个特殊的请求头，既然包含了这个特殊的请求头，那么请求就要升级，升级成WebSocket请求。这个握手请求的响应也比较特殊，它的成功状态码是101，而不是HTTP的200，如下：

```
HTTP/1.1 101 Switching Protocols 
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: 1qVdfYHU9hPOl4JYYNXF623Gzn0=
Sec-WebSocket-Protocol: v10.stomp
```

在这次成功的握手请求以后，在客户端和服务端之间的socket被打开，客户端和服务端可以进行消息的发送和接收。

## 代码实现

### 1.依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

-   spring-boot-starter-websocket，这是我们今天的主角，我们WebSocket的实现都依赖于这个jar包；
-   spring-boot-starter-thymeleaf，这只是起个辅助作用，在项目中要写个页面；

### 2.WebSocketHandler

好了，基础工作准备好了，下面进入最核心的代码，先写个WebSocketHandler，这个是用于在服务端接收和返回消息使用的。如下：

```java
import lombok.extern.log4j.Log4j2;
import org.springframework.stereotype.Component;
import org.springframework.web.socket.*;
import org.springframework.web.socket.handler.TextWebSocketHandler;

import java.util.Date;

/**
 * WebSocket处理器
 *
 * @author xiao
 * @date 2021年7月18日时间
 */
@Log4j2
@Component
public class MyHandler extends TextWebSocketHandler {

    @Override
    public void handleMessage(WebSocketSession session, WebSocketMessage<?> message) throws Exception {
        super.handleMessage(session, message);
    }

    @Override
    protected void handleBinaryMessage(WebSocketSession session, BinaryMessage message) {
        super.handleBinaryMessage(session, message);
    }

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        log.info("接收到来自客户端的消息：{}", message.getPayload());

        // 主动向客户端推送消息
        for (int i = 0; i < 10; i++) {
            session.sendMessage(new TextMessage(new Date() + " hello client !"));
            Thread.sleep(2000);
        }

        super.handleTextMessage(session, message);
    }

    @Override
    protected void handlePongMessage(WebSocketSession session, PongMessage message) throws Exception {
        super.handlePongMessage(session, message);
    }

    @Override
    public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
        super.handleTransportError(session, exception);
    }

    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        log.info("连接已确认");
        super.afterConnectionEstablished(session);
    }

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        log.info("连接已关闭");
        super.afterConnectionClosed(session, status);
    }
}
```

-   我们创建一个MyHandler类，继承TextWebSocketHandler类，这个类主要是处理文本的，当然也可以继承其他的类，比如：处理二进制的BinaryWebSocketHandler；

-   然后，我们实现handleTextMessage方法，这个方法有两个参数，WebSocketSession和TextMessage，TextMessage是接收客户端发来的消息。WebSocketSession用于设置WebSocket会话和向客户端发送消息；

-   在具体的方法实现中，我们调用TextMessage的getPayload方法，可以取出客户端发送的消息；

-   最后我们通过WebSocketSession的sendMessage方法向客户端发送消息，这里进行10次循环，每次循环我们间隔2秒；

-   继承体系如下：

    ![image-20210718122934053](markdown/SpringBoot + WebSocket实战.assets/image-20210718122934053.png)

### 3.配置

好了，到这里最核心的处理接收消息的方法我们已经写好了，然后我们将这个handler指定一个URL，如下：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

/**
 * WebSocket配置类
 *
 * @author xiao
 * @date 2021年7月18日时间
 */
@EnableWebSocket
@Configuration
public class WebSocketConfig implements WebSocketConfigurer {

    private final MyHandler myHandler;

    @Autowired
    public WebSocketConfig(MyHandler myHandler) {
        this.myHandler = myHandler;
    }

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry webSocketHandlerRegistry) {
        webSocketHandlerRegistry.addHandler(myHandler, "/websocket").setAllowedOriginPatterns("*");
    }

}
```

-   首先，我们写一个WebSocket的配置类WebSocketConfig去实现WebSocketConfigurer接口；
-   由于这是一个配置类，所以在类上加上注解@Configuration，同时因为要做WebSocket的配置，还要加上@EnableWebSocket这个注解；
-   这个类要实现注册WebSocketHandler的方法registerWebSocketHandlers，在这里，我们将前面写的Handler映射到/websocket这个URL；

到这里，WebSocket的服务端的内容就写好了，接下来，我们再写个简单的页面，在页面中，使用js进行socket的调用，具体页面内容如下：

```html
<body>
    <div id="msg"></div>
    <script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js"></script>
    <script>
        var socket = new WebSocket("ws://localhost:8080/websocket");
        socket.onopen = function() {
            socket.send("hello server !");
            console.log("数据发送中...");
        };
        socket.onmessage = function (evt){
            var received_msg = evt.data;
            $("#msg").append(received_msg+'<br>');
            console.log("数据已接收..."+received_msg);
        };

        socket.onclose = function(){
            // 关闭 websocket
            console.log("连接已关闭...");
        };
    </script>
</body>
```

-   我们先写个div，在这个div中展示服务端返回内容；
-   引入jquery，主要进行div内容的操作；
-   在第二个script中，我们进行websocket的连接，注意，协议名称是ws，地址就是我们在WebSocketConfig中配置的地址；
-   接下来就是onopen，onmessage，onclose方法，分别对应着socket打开，接收服务端消息和socket关闭的方法。我们在onmessage方法中，接收到服务端的消息，将其添加到div当中。

最后，我们再给这个html页面写个controller映射，如下：

```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

/**
 * WebSocket控制器
 *
 * @author xiao
 * @date 2021年7月18日时间
 */
@Controller
public class SocketController {

    @RequestMapping("index")
    public String index() {
        return "index";
    }

}
```

项目结构：

![image-20210718123605559](markdown/SpringBoot + WebSocket实战.assets/image-20210718123605559.png)

yaml文件：

```yaml
spring:
  application:
    name: websocket
  thymeleaf:
    prefix: classpath:/templates/
logging:
  level:
    root: info
```

### 4.测试

启动后端：

![image-20210718123750213](markdown/SpringBoot + WebSocket实战.assets/image-20210718123750213.png)

浏览器访问：http://localhost:8080/index

![image-20210718123923775](markdown/SpringBoot + WebSocket实战.assets/image-20210718123923775.png)

![image-20210718123933275](markdown/SpringBoot + WebSocket实战.assets/image-20210718123933275.png)

![image-20210718124003249](markdown/SpringBoot + WebSocket实战.assets/image-20210718124003249.png)

关闭页面：

![image-20210718124025804](markdown/SpringBoot + WebSocket实战.assets/image-20210718124025804.png)

