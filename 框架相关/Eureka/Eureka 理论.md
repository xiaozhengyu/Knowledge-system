# Eureka 理论

[toc]

## 服务治理

RPC 框架中服务间的依赖关系往往比较复杂，因此使用单独的组件专门对服务进行治理

-   服务注册
-   服务发现



## 服务注册 & 服务发现

如果把 Eureka 看作婚姻中介，那么张三将自己的信息挂到中介的过程就相当于服务注册，张三从中介获取其他人的信息就相当于服务发现。



![image-20220210134350740](markdown/Eureka 理论.assets/image-20220210134350740.png)

<center>图左：Eureka 架构   图右：Dubbo架构</center>

|     别名     |                            IP                             |
| :----------: | :-------------------------------------------------------: |
| user_service | xxx.xxx.xxx.xxx<br />xxx.xxx.xxx.xxx<br />xxx.xxx.xxx.xxx |



在服务注册和服务发现过程中，有一个注册中心的角色。服务提供者将自身消息（服务地址、通讯方式、别名…）注册到注册中心。服务消费者根据服务提供者的别名到注册中心查询服务提供者的具体信息（服务地址、通讯方式…），然后在本地完成 RPC 调用。

## EurekaServer & EurekaClient

Euraka 采用的是 CS 架构：

-   Eureka Server 作为服务注册中心，负责对外提供服务注册和服务查询的功能。运维人员可以通过 Eureka Server 监视各个服务节点的是否正常运行。Eureka Client 可以在 Eureka Server 登记自己的信息，也可以查询其他 Client 的信息。

-   Eureka Client 作为客户端，负责将自己的信息（如别名、地址）上报到注册中心，并定时发送心跳（默认每30秒发送一次），Eureka Server 凭据心跳判断服务的健康状态，如果在多个周期内没有接收到心跳，Eureka Service 会将服务节点从服务注册表移除。

