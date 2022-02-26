# Hystrix 实战 - 服务熔断

### 主启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
@EnableHystrix
@EnableCircuitBreaker
public class PaymentApplication {
    public static void main(String[] args) {
        SpringApplication.run(PaymentApplication.class, args);
    }
}
```

### 服务熔断

影响 Hystrix 断路器的 3 个重要参数：

-   时间窗口：断路器工作时需要统计接口工作情况，而统计的时间范围就是时间窗口，默认为 10 秒
-   请求总数阈值：在时间窗口内，请求总数必须超过阈值断路器才有资格熔断，默认为 20，这意味着如果 10 秒内请求的次数不足 20 个，即使所有请求都超时或失败，熔断依然不会触发。
-   错误百分比阈值：在时间窗口内，如果请求总数阈值已经达到，如果错误请求的占比超过该阈值，熔断就会触发

Hystrix 断路器工作的大概流程：

1.   时间窗口内，请求次数达到阈值，失败百分比达到阈值——触发熔断，再次接到请求时直接执行降级方法

2.   一段时间后（默认为 5 秒），尝试放行一个请求去执行原逻辑，如果请求成功，则结束熔断，如果请求失败，则继续熔断

     

```java
@HystrixCommand(fallbackMethod = "planB",commandProperties = {
        @HystrixProperty(name = "circuitBreaker.enabled",value = "true"),
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold",value = "10"), 
        @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds",value = "10000"),
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage",value = "60"),
})
public String testBreaker(){
    ...
}

private String planB(){...}
```

根据上述配置，触发熔断有两种条件：

1.   10000 毫秒内，请求次数超过 10
2.   10000 毫秒内，请求失败了超过 60%