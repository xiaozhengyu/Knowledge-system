# Spring 事件机制

>   参考文章：https://www.jianshu.com/p/bab2da26f282



[toc]



## 理论说明

在 Spring 中有一个事件机制，该机制基于`观察者模式`实现。通过 Spring 的事件机制可以达到以下目的：

-   应用模块间的解耦
-   对同一种事件可以根据需要定义多种处理方式
-   对主线应用不干扰，是一个极佳的开闭原则实践



Spring 事件机制相关的类：

-   ApplicationEvent：自定义事件的时候需要实现这个抽象类

    ![image-20221009095724936](markdown/Spring实战 - 事件机制.assets/image-20221009095724936.png)

    

-   ApplicationEventPublisher：Spring 的 ApplicationContext 默认实现了这个接口，用于<u>在上下文内发布事件</u>

    ![image-20221009095803588](markdown/Spring实战 - 事件机制.assets/image-20221009095803588.png)

-   ApplicationListener：自定义事件监听器的时候需要实现这个接口

    ![image-20221009095828311](markdown/Spring实战 - 事件机制.assets/image-20221009095828311.png)



>   **观察者模式：**观察者模式建立一种对象与对象之间的依赖关系。当一个对象（称之为：<u>观察目标</u>）发生改变时，它将主动通知其他对象（称之为：<u>观察者</u>），这些被通知对象将做出相应的反应。一个观察目标可以有多个观察者，而这些观察者之间相互独立，可以根据需要进行增减，这就使得系统更加易于扩展。



## 实战演示

自定义事件：

```java
package com.xzy;

import lombok.Getter;
import lombok.ToString;
import org.springframework.context.ApplicationEvent;

/**
 * 自定义事件
 *
 * @author xzy.xiao
 * @date 2022/10/8  18:57
 */
@Getter
@ToString
public class UserEvent extends ApplicationEvent {

    // ==================== constant ====================

    public static final int ET_INSERT = 1;
    public static final int ET_DELETE = 2;
    public static final int ET_UPDATE = 3;

    // ==================== field ====================

    /**
     * 事件类型
     */
    private final int eventType;
    /**
     * 发起事件的用户
     */
    private final User eventUser;
    /**
     * 事件创建时间
     */
    private final Long createTime;

    // ==================== constructor ====================

    public UserEvent(int eventType, User eventUser) {
        super(eventUser);
        this.eventType = eventType;
        this.eventUser = eventUser;
        this.createTime = System.currentTimeMillis();
    }
}
```

自定义事件发布器：

```java
package com.xzy;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.ApplicationContext;

/**
 * {@link UserEvent} 事件发布器
 *
 * @author xzy.xiao
 * @date 2022/10/8  19:09
 */
public class UserEventPublisher {
    private static final Logger logger = LoggerFactory.getLogger(UserEventPublisher.class);

    public static void sendInsertEvent(ApplicationContext applicationContext) {
        UserEvent userEvent = new UserEvent(UserEvent.ET_INSERT, new User(1000L, "张三"));
        logger.info("发布事件：{}", userEvent);
        applicationContext.publishEvent(userEvent);
    }

    public static void sendDeleteEvent(ApplicationContext applicationContext) {
        UserEvent userEvent = new UserEvent(UserEvent.ET_DELETE, new User(1000L, "张三"));
        logger.info("发布事件：{}", userEvent);
        applicationContext.publishEvent(userEvent);
    }

    public static void sendUpdateEvent(ApplicationContext applicationContext) {
        UserEvent userEvent = new UserEvent(UserEvent.ET_UPDATE, new User(1000L, "张三"));
        logger.info("发布事件：{}", userEvent);
        applicationContext.publishEvent(userEvent);
    }
}
```

自定义监听器：

```java
package com.xzy;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.ApplicationListener;

/**
 * {@link UserEvent} 事件监听器
 *
 * @author xzy.xiao
 * @date 2022/10/8  19:09
 */
public class UserEventListener implements ApplicationListener<UserEvent> {
    Logger logger = LoggerFactory.getLogger(UserEventListener.class);

    /**
     * Handle an application event.
     *
     * @param event the event to respond to
     */
    @Override
    public void onApplicationEvent(UserEvent event) {
        logger.info("监听到事件：{}", event);
        // TODO:
    }
}
```

演示效果：

```java
public static void main(String[] args) throws Exception {
    // 创建容器
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();

    // 注册事件监听器
    applicationContext.registerBean("userEventListener1", UserEventListener.class); // Note：事件会被所有相关监听器监听到
    applicationContext.registerBean("userEventListener2", UserEventListener.class);
    applicationContext.refresh();

    // 发布事件
    for (int i = 0; i < 3; i++) {
        UserEventPublisher.sendInsertEvent(applicationContext);
        UserEventPublisher.sendDeleteEvent(applicationContext);
        UserEventPublisher.sendUpdateEvent(applicationContext);

        Thread.sleep(1000);
    }

}
```

![image-20221009101742151](markdown/Spring实战 - 事件机制.assets/image-20221009101742151.png)

-   一个事件可以有多个监听器，各个监听器相互独立
-   事件必须发布在监听器所在的上下文