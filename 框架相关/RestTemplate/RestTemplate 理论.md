# RestTemplate 理论

<(￣︶￣)↗[参考文章](https://www.jianshu.com/p/0fd5f3f64137)

<(￣︶￣)↗[RestTemplate 官方文档](https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/web/client/RestTemplate.html)

---

RestTemplate 是 Spring 提供的一个用于调用 RESTful 接口的便捷模板类。它提供了很多便捷访问 HTTP 接口的方法，大大提高了开发者的编写效率。

| HTTP method | RestTemplate methods                                         |
| :---------: | ------------------------------------------------------------ |
|   DELETE    | [`delete(java.lang.String, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/web/client/RestTemplate.html#delete(java.lang.String, java.lang.Object...)) |
|     GET     | [`getForObject(java.lang.String, java.lang.Class, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/web/client/RestTemplate.html#getForObject(java.lang.String, java.lang.Class, java.lang.Object...)) |
|             | [`getForEntity(java.lang.String, java.lang.Class, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/web/client/RestTemplate.html#getForEntity(java.lang.String, java.lang.Class, java.lang.Object...)) |
|    HEAD     | [`headForHeaders(java.lang.String, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/web/client/RestTemplate.html#headForHeaders(java.lang.String, java.lang.Object...)) |
|   OPTIONS   | [`optionsForAllow(java.lang.String, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/web/client/RestTemplate.html#optionsForAllow(java.lang.String, java.lang.Object...)) |
|    POST     | [`postForLocation(java.lang.String, java.lang.Object, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/web/client/RestTemplate.html#postForLocation(java.lang.String, java.lang.Object, java.lang.Object...)) |
|             | [`postForObject(java.lang.String, java.lang.Object, java.lang.Class, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/web/client/RestTemplate.html#postForObject(java.lang.String, java.lang.Object, java.lang.Class, java.lang.Object...)) |
|     PUT     | [`put(java.lang.String, java.lang.Object, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/web/client/RestTemplate.html#put(java.lang.String, java.lang.Object, java.lang.Object...)) |
|     any     | [`exchange(java.lang.String, org.springframework.http.HttpMethod, org.springframework.http.HttpEntity, java.lang.Class, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/web/client/RestTemplate.html#exchange(java.lang.String, org.springframework.http.HttpMethod, org.springframework.http.HttpEntity, java.lang.Class, java.lang.Object...)) |
|             | [`execute(java.lang.String,  org.springframework.http.HttpMethod,  org.springframework.web.client.RequestCallback,  org.springframework.web.client.ResponseExtractor, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/3.0.x/javadoc-api/org/springframework/web/client/RestTemplate.html#execute(java.lang.String, org.springframework.http.HttpMethod, org.springframework.web.client.RequestCallback, org.springframework.web.client.ResponseExtractor, java.lang.Object...)) |

