# 服务发现

服务发现，即获取注册中心中记录的服务的信息：有哪些服务？服务有哪些实例？实例的详细信息？



[toc]

## 1、配置文件

```yaml
eureka:
  instance:
    instance-id: cloud-payment-service8001
    prefer-ip-address: true
  client:
    register-with-eureka: true # 是否向注册中心注册自己
    fetch-registry: true # 是否需要检索服务
    service-url:
      defaultZone: http://eureka-register1:7001/eureka/,http://eureka-register2:7002/eureka/,http://eureka-register3:7003/eureka/
```

注意：eureka.client.fetch-registry

## 2、主启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
public class PaymentApplication {
    public static void main(String[] args) {
        SpringApplication.run(PaymentApplication.class, args);
    }
}
```

注意：@EnableDiscoveryClient

## 3、业务类

```java
@Autowired
private final DiscoveryClient discoveryClient;

MessageBox<Map<String, List<Map<String, String>>>> getInstanceInfo() {
    // 服务
    List<String> serviceList = discoveryClient.getServices();
    Map<String, List<Map<String, String>>> service2instance = new HashMap<>(serviceList.size());
    for (String service : serviceList) {
        // 实例
        List<ServiceInstance> instanceList = discoveryClient.getInstances(service);
        List<Map<String, String>> instanceInfoList = new ArrayList<>(instanceList.size());
        for (ServiceInstance instance : instanceList) {
            // 实例信息
            Map<String, String> instanceInfo = new HashMap<>(4);
            instanceInfo.put("instanceId", instance.getInstanceId());
            instanceInfo.put("instanceHost", instance.getHost());
            instanceInfo.put("instancePort", instance.getPort() + "");
            instanceInfo.put("uri", instance.getUri().toString());
            instanceInfoList.add(instanceInfo);
        }
        service2instance.put(service, instanceInfoList);
    }
    return MessageBox.ok(service2instance);
}
```

注意：org.springframework.cloud.client.discovery.DiscoveryClient

## 4、测试

调用方法，返回信息：

```json
{
  "status": 1,
  "message": "成功",
  "ok": true,
  "data": {
    "cloud-payment-service": [
      {
        "instancePort": "8001",
        "instanceId": "cloud-payment-service8001",
        "uri": "http://169.254.4.157:8001",
        "instanceHost": "169.254.4.157"
      }
    ]
  },
  "fail": false
}
```

