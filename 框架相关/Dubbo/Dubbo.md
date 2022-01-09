# Apache Dubbo

[Apache Dubbo 是一款高性能、轻量级的开源服务框架](https://dubbo.apache.org/zh/)

## 简介

Apache Dubbo 是一款微服务开发框架，它提供了 RPC通信 与 微服务治理 两大关键能力。这意味着，使用 Dubbo 开发的微服务，将具备相互之间的远程发现与通信能力， 同时利用 Dubbo 提供的丰富服务治理能力，可以实现诸如服务发现、负载均衡、流量调度等服务治理诉求。同时 Dubbo 是高度可扩展的，用户几乎可以在任意功能点去定制自己的实现，以改变框架的默认行为来满足自己的业务需求。

Apache Dubbo 提供了六大核心能力：

1.   面向接口代理的高性能<font color = red>RPC调用</font>

     提供高性能的基于代理的远程调用能力，服务以接口为粒度，为开发者屏蔽远程调用底层细节。

2.   智能<font color = red>负载均衡</font>

     内置多种负载均衡策略，智能感知下游节点健康状况，显著减少调用延迟，提高系统吞吐量。

3.   服务自动<font color = red>注册与发现</font>

     支持多种注册中心服务，服务实例上下线实时感知。

4.   高度<font color = red>可扩展</font>能力

     遵循微内核+插件的设计原则，所有核心能力如 Protocol、Transport、Serialization 被设计为扩展点，平等对待内置实现和第三方实现。

5.   运行期<font color = red>流量调度</font>

     内置条件、脚本等路由策略，通过配置不同的路由规则，轻松实现灰度发布，同机房优先等功能。

6.   可视化的服务<font color = red>治理与运维</font>

     提供丰富服务治理、运维工具：随时查询服务元数据、服务健康状态及调用统计，实时下发路由策略、调整配置参数。

（[→ 更详细的官方说明](https://dubbo.apache.org/zh/docs/introduction/#what-dubbo3-is)）

## 实战

[→ Dubbo - 实战](Dubbo - 实战.md)

## 原理

[→ Dubbo - 原理](Dubbo - 原理.md)

