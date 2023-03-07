# Spring 源码 - 事件机制

[TOC]



## ApplicationEventMulticaster

>   `Multicast`，译作广播、群播、多播。在计算机网络中，Multicast 指的是将消息同时传递给一组目标地址。



### 接口说明

从功能的角度出发，ApplicationEventMulticaster 中的方法可以分为两类：**管理监听器**、**管理事件**

```java
public interface ApplicationEventMulticaster {

    // ========== 管理监听器 ==========
    
   /**
    * Add a listener to be notified of all events.
    * @param listener the listener to add
    */
   void addApplicationListener(ApplicationListener<?> listener);

   /**
    * Add a listener bean to be notified of all events.
    * @param listenerBeanName the name of the listener bean to add
    */
   void addApplicationListenerBean(String listenerBeanName);

   /**
    * Remove a listener from the notification list.
    * @param listener the listener to remove
    */
   void removeApplicationListener(ApplicationListener<?> listener);

   /**
    * Remove a listener bean from the notification list.
    * @param listenerBeanName the name of the listener bean to remove
    */
   void removeApplicationListenerBean(String listenerBeanName);

   /**
    * Remove all listeners registered with this multicaster.
    * <p>After a remove call, the multicaster will perform no action
    * on event notification until new listeners are registered.
    */
   void removeAllListeners();

    
    // ========== 管理事件 ==========
    
    
   /**
    * Multicast the given application event to appropriate listeners.
    * <p>Consider using {@link #multicastEvent(ApplicationEvent, ResolvableType)}
    * if possible as it provides better support for generics-based events.
    * @param event the event to multicast
    */
   void multicastEvent(ApplicationEvent event);

   /**
    * Multicast the given application event to appropriate listeners.
    * <p>If the {@code eventType} is {@code null}, a default type is built
    * based on the {@code event} instance.
    * @param event the event to multicast
    * @param eventType the type of event (can be {@code null})
    * @since 4.2
    */
   void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType);

}
```

ApplicationEventMulticaster 内定义了两个方法分别用于管理<u>普通事件</u>和<u>泛型事件</u>：

1.   **普通事件**

     普通事件：

     ```java
     public class XxxEvent extends ApplicationEvent {}
     ```

     普通事件监听器：

     ```java
     // 基于接口
     @Component
     public class XxxEventListener implements ApplicationListener<XxxEvent> {...}
     
     // 基于注解
     @EventListener
     public void onXxxEvent(XxxEvent xxxEvent) {}
     ```

2.   **泛型事件**

     泛型事件：

     ```java
     public class XxxEvent<T> extends ApplicationEvent implements ResolvableTypeProvider {
         // Return the {@link ResolvableType} describing this instance (or {@code null} if some sort of default should be applied instead).
         @Override
         public ResolvableType getResolvableType() {...} 
     }
     ```

     泛型事件监听器：

     ```java
     // 基于接口
     @Component
     public class XxxEventListener implements ApplicationListener<XxxEvent<XxxType>> {...}
     
     // 基于注解
     @EventListener
     public void onXxxEvent(XxxEvent<XxxType> xxxEvent) {}
     ```



### 接口实现

>   Q：关于实现 ApplicationEventMulticaster 接口你有什么想法、思路？
>
>   A：实现接口的关键是如何维护 Listener 与 Event 之间的对应关系，或者说如何判断一个 Event 应该分发给哪些 Listener。除此之外，才需要考虑各种并发问题、查找效率问题。
>



### 接口应用



通常 ApplicationEventPublisher 接口的实现类会借助 ApplicationEventMulticaster 的实例来实现监听器以及事件的管理（合成复用原则），典型的例子是 ApplicationContent 的实现类：

```java
// ApplicationContent 接口继承了 ApplicationEventPublisher 接口
public interface ApplicationContext extends ... ApplicationEventPublisher {}
```

```java
// ApplicationContent 接口的抽象类借助 ApplicationEventMulticaster 实例来实现事件监听器的管理以及事件的发布
public abstract class AbstractApplicationContext ... implements ConfigurableApplicationContext {
    ...
    
    /** Helper class used in event publishing. */
	@Nullable
	private ApplicationEventMulticaster applicationEventMulticaster;
    
    ...
}
```

