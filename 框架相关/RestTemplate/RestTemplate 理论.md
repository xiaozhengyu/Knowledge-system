# RestTemplate 理论

<(￣︶￣)↗[官方文档](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/spring-framework-reference/html/remoting.html#rest-client-access)

RestTemplate 是 Spring 提供的用于客户端访问 RESTful 服务的核心类。它的概念类似于 Spring 中的其他模板类，例如 JdbcTemplate、JmsTemplate。

可以通过提供回调方法以及配置 HttpMessageConverter 来定制 RestTemplate 的行为。HttpMessageConverter 用于将对象封装到 HTTP 请求中，并将 HTTP 响应封装到对象。



在 Java 中，客户端访问 RESTful 服务的典型方法是通过 HttpClient，但是这种用法比较低级——需要自己封装请求，需要自己根据判断状态码，需要自己从响应中获取响应头以及响应体，甚至还需要自己做 JSON 转换：

```java
String uri = "http://example.com/hotels/1/bookings";
String request = // create booking request content

PostMethod post = new PostMethod(uri);
post.setRequestEntity(new StringRequestEntity(request));

httpClient.executeMethod(post);

if (HttpStatus.SC_CREATED == post.getStatusCode()) {
    Header location = post.getRequestHeader("Location");
    if (location != null) {
        System.out.println("Created new booking at :" + location.getValue());
    }
}
```

RestTemplate 将主要的 HTTP 请求进行封装，客户端可以直接调用封装好的方法，便捷的访问 RESTful 服务：

| DELETE           | [delete](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#delete(String, Object…)) |
| ---------------- | ------------------------------------------------------------ |
| GET              | [getForObject](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#getForObject(String, Class, Object…)) [getForEntity](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#getForEntity(String, Class, Object…)) |
| HEAD             | [headForHeaders(String url, String… uriVariables)](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#headForHeaders(String, Object…)) |
| OPTIONS          | [optionsForAllow(String url, String… uriVariables)](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#optionsForAllow(String, Object…)) |
| POST             | [postForLocation(String url, Object request, String… uriVariables)](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#postForLocation(String, Object, Object…)) [postForObject(String url, Object request, Class responseType, String… uriVariables)](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#postForObject(java.lang.String, java.lang.Object, java.lang.Class, java.lang.String…)) |
| PUT              | [put(String url, Object request, String…uriVariables)](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#put(String, Object, Object…)) |
| PATCH and others |                                                              |

