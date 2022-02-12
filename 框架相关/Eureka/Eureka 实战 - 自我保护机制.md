# 自我保护模式

<(￣︶￣)↗[什么是 Eureka 的自我保护机制？](./Eureka 理论.md)

[toc]

## 1、禁用自我保护机制

```yaml
eureka:
  server:
     enable-self-preservation: false        # 禁用自我保护机制
     eviction-interval-timer-in-ms: 3000    # 清理失效节点的时间间隔
```

Eureka 自我保护机制默认为打开状态，建议生产环境打开此配置。

## 2、客户端心跳时间

```yaml
eureka:
  instance:
    lease-expiration-duration-in-seconds: 90
    lease-renewal-interval-in-seconds: 30
```

-   lease-expiration-duration-in-seconds：Eureka Server 至上一次收到 Eureka Client 的心跳之后，等待下一次心跳的超时时间，在这个时间内若没收到下一次心跳，则将移除该 instance
-   lease-renewal-interval-in-seconds：Eureka Client 想 Eureka Server 发送心跳的频率。

如果 lease-expiration-duration-in-seconds 设置的太大，请求发送到 instance 的时候，instance 可能已经挂了；如果 lease-expiration-duration-in-seconds 设置的太小，instance 可能因为临时的网络抖动而被 Eureka Service 移除。lease-expiration-duration-in-seconds 的值至少应该大于 lease-renewal-interval-in-seconds