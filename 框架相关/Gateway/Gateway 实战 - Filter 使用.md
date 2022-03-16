# Filter 使用

[toc]



<(￣︶￣)↗[Spring 官方文档](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#gatewayfilter-factories)



## 是什么？

“Route filters allow the modification of the incoming HTTP request or outgoing HTTP response in some manner.” 路由过滤器可用于对 HTTP 请求以及 HTTP 响应进行修改。

## 怎么用？

Spring Cloud Gateway 中的路由过滤器可以分为两种：

1.   GatewayFilter：仅对某个 Route 起作用
2.   GlobalFilter：对全局所有 Route 起作用

对于 GatewayFilter 和 GlobalFilter，Gateway 内置了多种实现提供使用者选择，具体内容可见官方文档：[GatewayFilter Factories](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#gatewayfilter-factories)，[Global Filters](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.1.RELEASE/reference/html/#global-filters)



下面是通过自定义 GlobalFilter 实现全局日志的示例：

```java
@Slf4j
public class LogGatewayFilter implements GlobalFilter, Ordered {
    /**
     * Process the Web request and (optionally) delegate to the next {@code WebFilter}
     * through the given {@link GatewayFilterChain}.
     *
     * @param exchange the current server exchange
     * @param chain    provides a way to delegate to the next filter
     * @return {@code Mono<Void>} to indicate when request processing is complete
     */
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("my log gateway filter...");
        // return exchange.getResponse().setComplete(); // 直接返回
        return chain.filter(exchange); // 交给后面的filter继续处理
    }

    /**
     * Get the order value of this object.
     * <p>Higher values are interpreted as lower priority. As a consequence,
     * the object with the lowest value has the highest priority (somewhat
     * analogous to Servlet {@code load-on-startup} values).
     * <p>Same order values will result in arbitrary sort positions for the
     * affected objects.
     *
     * @return the order value
     * @see #HIGHEST_PRECEDENCE
     * @see #LOWEST_PRECEDENCE
     */
    @Override
    public int getOrder() {
        return 0;
    }
}
```

```java
@SpringBootConfiguration
public class GatewayConfig {
    /**
     * 自定义过滤器
     */
    @Bean
    public LogGatewayFilter logGatewayFilter() {
        return new LogGatewayFilter();
    }
}
```
