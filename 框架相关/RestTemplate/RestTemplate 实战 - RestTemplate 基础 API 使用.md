# RestTemplate 基础 API 使用

[toc]

[官方接口文档](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html)

| HTTP method | RestTemplate methods                                         |
| ----------- | ------------------------------------------------------------ |
| DELETE      | [`delete(java.lang.String, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#delete-java.lang.String-java.lang.Object...-) |
| GET         | [`getForObject(java.lang.String, java.lang.Class, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#getForObject-java.lang.String-java.lang.Class-java.lang.Object...-) |
|             | [`getForEntity(java.lang.String, java.lang.Class, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#getForEntity-java.lang.String-java.lang.Class-java.lang.Object...-) |
| HEAD        | [`headForHeaders(java.lang.String, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#headForHeaders-java.lang.String-java.lang.Object...-) |
| OPTIONS     | [`optionsForAllow(java.lang.String, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#optionsForAllow-java.lang.String-java.lang.Object...-) |
| POST        | [`postForLocation(java.lang.String, java.lang.Object, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#postForLocation-java.lang.String-java.lang.Object-java.lang.Object...-) |
|             | [`postForObject(java.lang.String, java.lang.Object, java.lang.Class, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#postForObject-java.lang.String-java.lang.Object-java.lang.Class-java.lang.Object...-) |
| PUT         | [`put(java.lang.String, java.lang.Object, java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#put-java.lang.String-java.lang.Object-java.lang.Object...-) |
| any         | [`exchange(java.lang.String,  org.springframework.http.HttpMethod,  org.springframework.http.HttpEntity, java.lang.Class,  java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#exchange-java.lang.String-org.springframework.http.HttpMethod-org.springframework.http.HttpEntity-java.lang.Class-java.lang.Object...-) |
|             | [`execute(java.lang.String,  org.springframework.http.HttpMethod,  org.springframework.web.client.RequestCallback,  org.springframework.web.client.ResponseExtractor,  java.lang.Object...)`](https://docs.spring.io/spring-framework/docs/4.3.9.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html#execute-java.lang.String-org.springframework.http.HttpMethod-org.springframework.web.client.RequestCallback-org.springframework.web.client.ResponseExtractor-java.lang.Object...-) |

Note：

>   1.   RestTemplate 的方法命名遵循一定的规则：【what HTTP method is being invoked】 + 【what is returned】，例如：getForObject()、postForEntity()，get 表示 GET 请求，post 表示 POST 请求，Object 表示将 http 响应转换成指定的对象然后直接返回这个对象，Entity 表示需要将转换好的对象与其他响应信息一起封装成 HttpEntity 对象返回。
>
>        
>
>   2.   所有传递给这些方法以及从这些方法返回的对象都由 HttpMessageConverter 实例进行装换：
>
>        ```mermaid
>        graph LR
>        Object-->HttpMessageConverter-->HTTP
>        HTTP-->HttpMessageConverter-->Object
>        ```
>
>        
>
>   3.   exchange 和 execute 方法比其他方法的适用范围更广~(万金油)~



## GET 请求

### getForEntity() 方法

发送 GET 请求，返回 ResponseEntity

```java
/**
 * 参数1： String类型 或 URI类型的请求地址
 * 参数2： 指定返回的实体类型，class对象
 * 参数3： uri参数，可以是变长数组或map
 * 返回值：ResponseEntity<T>是Spring对HTTP响应的封装，包括了几个重要的元素，如响应码、contentType、contentLength、response header信息，response body信息等
 */
@Override
public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Object... uriVariables)
        throws RestClientException {
    RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
    ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
    return execute(url, HttpMethod.GET, requestCallback, responseExtractor, uriVariables);
}

@Override
public <T> ResponseEntity<T> getForEntity(String url, Class<T> responseType, Map<String, ?> uriVariables)
        throws RestClientException {
    RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
    ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
    return execute(url, HttpMethod.GET, requestCallback, responseExtractor, uriVariables);
}

@Override
public <T> ResponseEntity<T> getForEntity(URI url, Class<T> responseType) throws RestClientException {
    RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
    ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
    return execute(url, HttpMethod.GET, requestCallback, responseExtractor);
}
```

**举例：**

```java
ResponseEntity<Book> responseEntity = restTemplate
    .getForEntity("http://127.0.0.1:8080/getbook?bookname={1}", Book.class, "java");

Book book = responseEntity.getBody();  //响应体转换为Book类型
int statusCodeValue = responseEntity.getStatusCodeValue();  //响应状态码
HttpHeaders headers = responseEntity.getHeaders();  //响应头信息
```



### getForObject() 方法

发送 GET 请求，返回指定的 Object 类型

```java
/**
 * 参数1： String类型 或 URI类型的请求地址
 * 参数2： 指定返回的实体类型，class对象
 * 参数3： uri参数，可以是变长数组或map
 * 返回值：responseType指定的Object类型
 */
@Override
public <T> T getForObject(String url, Class<T> responseType, Object... uriVariables) throws RestClientException {
    RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
    HttpMessageConverterExtractor<T> responseExtractor =
            new HttpMessageConverterExtractor<T>(responseType, getMessageConverters(), logger);
    return execute(url, HttpMethod.GET, requestCallback, responseExtractor, uriVariables);
}

@Override
public <T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException {
    RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
    HttpMessageConverterExtractor<T> responseExtractor =
            new HttpMessageConverterExtractor<T>(responseType, getMessageConverters(), logger);
    return execute(url, HttpMethod.GET, requestCallback, responseExtractor, uriVariables);
}

@Override
public <T> T getForObject(URI url, Class<T> responseType) throws RestClientException {
    RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
    HttpMessageConverterExtractor<T> responseExtractor =
            new HttpMessageConverterExtractor<T>(responseType, getMessageConverters(), logger);
    return execute(url, HttpMethod.GET, requestCallback, responseExtractor);
}
```

举例：

```java
Book book = restTemplate
    .getForObject("http://127.0.0.1:8080/getbook?bookname={1}", Book.class, "java");
```



## POST 请求

### postForEntity() 方法

发送 POST 请求，返回 ResponseEntity

```java
/**
 * 参数1： String类型 或 URI类型的请求地址
 * 参数2： 请求body，可以是HttpEntity类型（可设置request header），或其它Object类型
 * 参数3： 指定返回的实体类型，class对象
 * 参数4： uri参数，可以是变长数组或map
 * 返回值：ResponseEntity<T>是Spring对HTTP响应的封装，包括了几个重要的元素，如响应码、contentType、contentLength、response header信息，response body信息等
 */
@Override
public <T> ResponseEntity<T> postForEntity(String url, Object request, Class<T> responseType, Object... uriVariables)  throws RestClientException {
    RequestCallback requestCallback = httpEntityCallback(request, responseType);
    ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
    return execute(url, HttpMethod.POST, requestCallback, responseExtractor, uriVariables);
}

@Override
public <T> ResponseEntity<T> postForEntity(String url, Object request, Class<T> responseType, Map<String, ?> uriVariables)  throws RestClientException {
    RequestCallback requestCallback = httpEntityCallback(request, responseType);
    ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
    return execute(url, HttpMethod.POST, requestCallback, responseExtractor, uriVariables);
}

@Override
public <T> ResponseEntity<T> postForEntity(URI url, Object request, Class<T> responseType) throws RestClientException {
    RequestCallback requestCallback = httpEntityCallback(request, responseType);
    ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
    return execute(url, HttpMethod.POST, requestCallback, responseExtractor);
}
```

**举例：**

```java
//参数是Book类型，返回值是ResponseEntity<Book>类型
ResponseEntity<Book> responseEntity = restTemplate
    .postForEntity("http://127.0.0.1:8080/updateBook", book, Book.class);

Book book = responseEntity.getBody();  //响应体转换为Book类型
int statusCodeValue = responseEntity.getStatusCodeValue();  //响应状态码
HttpHeaders headers = responseEntity.getHeaders();  //响应头信息
```



### postForObject() 方法

发送 POST 请求，返回指定的 Object 类型

```java
/**
 * 参数1： String类型 或 URI类型的请求地址
 * 参数2： 请求body，可以是HttpEntity类型（可设置request header），或其它Object类型
 * 参数3： 指定返回的实体类型，class对象
 * 参数4： uri参数，可以是变长数组或map
 * 返回值：responseType指定的Object类型
 */
@Override
public <T> T postForObject(String url, Object request, Class<T> responseType, Object... uriVariables)
        throws RestClientException {
    RequestCallback requestCallback = httpEntityCallback(request, responseType);
    HttpMessageConverterExtractor<T> responseExtractor =
            new HttpMessageConverterExtractor<T>(responseType, getMessageConverters(), logger);
    return execute(url, HttpMethod.POST, requestCallback, responseExtractor, uriVariables);
}

@Override
public <T> T postForObject(String url, Object request, Class<T> responseType, Map<String, ?> uriVariables)
        throws RestClientException {
    RequestCallback requestCallback = httpEntityCallback(request, responseType);
    HttpMessageConverterExtractor<T> responseExtractor =
            new HttpMessageConverterExtractor<T>(responseType, getMessageConverters(), logger);
    return execute(url, HttpMethod.POST, requestCallback, responseExtractor, uriVariables);
}

@Override
public <T> T postForObject(URI url, Object request, Class<T> responseType) throws RestClientException {
    RequestCallback requestCallback = httpEntityCallback(request, responseType);
    HttpMessageConverterExtractor<T> responseExtractor =
            new HttpMessageConverterExtractor<T>(responseType, getMessageConverters());
    return execute(url, HttpMethod.POST, requestCallback, responseExtractor);
}
```

**举例：**

```java
//参数是Book类型，返回值也是Book类型
Book book = restTemplate
    .postForObject("http://127.0.0.1:8080/updatebook", book, Book.class);
```



### exchange() 方法

-   可以支持多种 HTTP 方法，在参数中指定
-   可以在请求中增加 header 和 body 信息，返回类型是 ResponseEntity，可以从中获取响应的状态码，header 和 body 等信息

```java
HttpHeaders requestHeaders = new HttpHeaders();
requestHeaders.set("MyRequestHeader", "MyValue");
HttpEntity requestEntity = new HttpEntity(requestHeaders);

HttpEntity<String> response = template.exchange(
        "http://example.com/hotels/{hotel}",
        HttpMethod.GET,     //GET请求
        requestEntity,      //requestEntity，可以设置请求header、body
        String.class, "42");

String responseHeader = response.getHeaders().getFirst("MyResponseHeader");   //响应头信息
String body = response.getBody();
```
