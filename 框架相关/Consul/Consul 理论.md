# Consul 理论

## 1. 是什么？

<(￣︶￣)↗[Consul 官方说明](https://www.consul.io/docs/intro)

## 2. 能干嘛？

The key features of Consul are:

-   **Service Discovery（服务发现）**: Clients of Consul can register a service, such as `api` or `mysql`, and other clients can use Consul to discover providers of a given service. Using either DNS or HTTP, applications can easily find the services they depend upon.
-   **Health Checking（健康监测）**: Consul clients can provide any number of health checks, either associated with a given service ("is the webserver returning 200 OK"), or with the local node ("is memory utilization below 90%"). This information can be used by an operator to monitor cluster health, and it is used by the service discovery components to route traffic away from unhealthy hosts.
-   **KV Store（k-v数据存储）**: Applications can make use of Consul's hierarchical key/value store for any number of purposes, including dynamic configuration, feature flagging, coordination, leader election, and more. The simple HTTP API makes it easy to use.
-   **Secure Service Communication（安全服务通信）**: Consul can generate and distribute TLS certificates for services to establish mutual TLS connections. [Intentions](https://www.consul.io/docs/connect/intentions) can be used to define which services are allowed to communicate. Service segmentation can be easily managed with intentions that can be changed in real time instead of using complex network topologies and static firewall rules.
-   **Multi Datacenter（多数据中心）**: Consul supports multiple datacenters out of the box. This means users of Consul do not have to worry about building additional layers of abstraction to grow to multiple regions.

## 3. 怎么用？

<(￣︶￣)↗[Spring Cloud Consul 中文教程](https://www.springcloud.cc/spring-cloud-consul.html)