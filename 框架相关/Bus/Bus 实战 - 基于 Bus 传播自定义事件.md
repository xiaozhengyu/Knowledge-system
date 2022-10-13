# 基于 Bus 传播自定义事件

>   官网文档：https://docs.spring.io/spring-cloud-bus/docs/current/reference/html/#broadcasting-your-own-events

## 实战

### 1. 自定义事件

```java
package com.xzy.event;

import lombok.Data;
import lombok.EqualsAndHashCode;
import org.springframework.cloud.bus.event.RemoteApplicationEvent;

/**
 * 自定义的远程事件
 *
 * @author xzy.xiao
 * @date 2022/10/13  11:16
 */
@Data
@EqualsAndHashCode(callSuper = true)
public class MineRemoteApplicationEvent extends RemoteApplicationEvent {

    public MineRemoteApplicationEvent() {
        // for serialization
    }

    public MineRemoteApplicationEvent(Object source, String originService, String destinationService) {
        super(source, originService, destinationService);
    }
}
```

>   Note：Bus 内部有一个监听 RemoteApplicationEvent 事件的监听器，它会将监听到的事件通过底层的 Stream 传播给其他服务，所以我们自定义的事件需要继承 RemoteApplicationEvent。
>
>   ```java
>   package org.springframework.cloud.bus;
>   ...
>   public class BusAutoConfiguration implements ApplicationEventPublisherAware {
>       
>   	@EventListener(classes = RemoteApplicationEvent.class)
>   	public void acceptLocal(RemoteApplicationEvent event) {
>   		if (this.serviceMatcher.isFromSelf(event)
>   				&& !(event instanceof AckRemoteApplicationEvent)) {
>   			this.cloudBusOutboundChannel.send(MessageBuilder.withPayload(event).build());
>   		}
>   	}    
>   }
>   ```

### 2. 监听事件

```java
package com.xzy.event;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.bus.event.RemoteApplicationEvent;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

/**
 * 自定义事件的监听器
 *
 * @author xzy.xiao
 * @date 2022/10/13  11:20
 */
@Component
public class MineEventListener {

    private final Logger log = LoggerFactory.getLogger(MineEventListener.class);
    private final ObjectMapper objectMapper = new ObjectMapper();

    @EventListener
    public void onRemoteApplicationEvent(RemoteApplicationEvent event) throws JsonProcessingException {
        log.info("接收事件：{}", objectMapper.writeValueAsString(event));
    }

    @EventListener
    public void onMineRemoteApplicationEvent(MineRemoteApplicationEvent event) throws JsonProcessingException {
        log.info("接收事件：{}", objectMapper.writeValueAsString(event));
    }
}
```

### 3. 发布事件

```java
package com.xzy.controller;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.xzy.event.MineRemoteApplicationEvent;
import com.xzy.msg.Message;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.bus.BusProperties;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author xzy.xiao
 * @date 2022/10/13  11:22
 */
@RestController
@RequestMapping(path = "/remote_event")
public class RemoteEventController {

    private final Logger log = LoggerFactory.getLogger(RemoteEventController.class);
    private final ObjectMapper objectMapper = new ObjectMapper();

    private final ApplicationEventPublisher applicationEventPublisher;
    private final BusProperties busProperties;

    @Autowired
    public RemoteEventController(ApplicationEventPublisher applicationEventPublisher, BusProperties busProperties) {
        this.applicationEventPublisher = applicationEventPublisher;
        this.busProperties = busProperties;
    }

    @PostMapping("/hello")
    public Message hello() throws JsonProcessingException {
        MineRemoteApplicationEvent mineRemoteApplicationEvent = new MineRemoteApplicationEvent("hello world!", busProperties.getId(), null);
        applicationEventPublisher.publishEvent(mineRemoteApplicationEvent);
        log.info("发布事件：{}", objectMapper.writeValueAsString(mineRemoteApplicationEvent));
        return Message.ok();
    }
}
```

### 4. 注册事件

```java
package com.xzy;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.bus.jackson.RemoteApplicationEventScan;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

/**
 * 用户服务
 *
 * @author xzy
 * @date 2022/1/10 10:34
 */
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
// Bus需要事先知道我们自定义的远程事件，否则反序列化的时候会生成UnknownRemoteApplicationEvent
@RemoteApplicationEventScan(basePackages = "com.xzy.event")
public class UserApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
    }
}
```

>   Note：Bus 通过 Stream 传播事件的时候需要把事件序列化，Bus 通过 Stream 接收事件的时候需要把事件反序列化。
>
>   ```mermaid
>   graph LR
>   
>   Service1[Service1]
>   Event1[Event]
>   Stream[Stream]
>   Event2[Event]
>   Service2[Service2]
>   Service3[Service3]
>   Service4[Service4]
>   
>   Service1-->Event1--序列化-->Stream--反序列化-->Event2
>   Event2-->Service2
>   Event2-->Service3
>   Event2-->Service4
>   ```
>
>   自定义的 RemoteApplicationEvent 需要先注册给 Bus，才能被正确的反序列化。

## 测试

