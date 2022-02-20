# RestTemplate 配置、拓展

## 1、处理请求头



（1）如果是发送 post、put 请求，需要设置请求头，可以在调用方法时，利用第二个参数传入 HttpEntity 对象，HttpEntity 可以用于设置请求头信息，例如：

```java
HttpHeaders requestHeaders = new HttpHeaders();
requestHeaders.set("MyRequestHeader", "MyValue");
HttpEntity requestEntity = new HttpEntity(requestHeaders);

Book book = restTemplate.postForObject("http://127.0.0.1:8080/getbook", requestEntity, Book.class);
```

以`postForObject()`方法为例，其第二个参数接收 Object 类型的数据，如传入的是 HttpEntity，则使用它作为整个请求实体，如果传入的是其它 Object 类型，则将 Object 参数作为 request body，新建一个 HttpEntity 作为请求实体

```java
private HttpEntityRequestCallback(Object requestBody, Type responseType) {
    super(responseType);
    //如果是HttpEntity类型的，直接作为请求实体赋值给this.requestEntity
    if (requestBody instanceof HttpEntity) {
        this.requestEntity = (HttpEntity<?>) requestBody;
    }
    //如果requestBody不是HttpEntity类型，且不为空，以Object参数作为request body，并新建HttpEntity
    else if (requestBody != null) {
        this.requestEntity = new HttpEntity<Object>(requestBody);
    }
    else {
        this.requestEntity = HttpEntity.EMPTY;
    }
}
```



（2）如果是其它HTTP方法调用要设置请求头，可以使用exchange()方法，可以参考 [官方示例](https://docs.spring.io/spring/docs/4.3.9.RELEASE/spring-framework-reference/html/remoting.html#rest-template-headers)

```java
HttpHeaders requestHeaders = new HttpHeaders();
requestHeaders.set("MyRequestHeader", "MyValue");
HttpEntity requestEntity = new HttpEntity(requestHeaders);

HttpEntity<String> response = template.exchange(
        "http://example.com/hotels/{hotel}",
        HttpMethod.GET, requestEntity, String.class, "42");

String responseHeader = response.getHeaders().getFirst("MyResponseHeader");
String body = response.getBody();
```

总之，设置 request header 信息，需要找到对应的 restTemplate 方法中可以使用 HttpEntity 作为参数的，提前设置好请求头信息

注意：HttpEntity 有4个构造方法：无参构造、只设置请求 body、只设置 headers、既设置headers又设置body

```java
/**
 * Create a new, empty {@code HttpEntity}.
 */
protected HttpEntity() {
  this(null, null);
}

/**
 * Create a new {@code HttpEntity} with the given body and no headers.
 * @param body the entity body
 */
public HttpEntity(T body) {
  this(body, null);
}

/**
 * Create a new {@code HttpEntity} with the given headers and no body.
 * @param headers the entity headers
 */
public HttpEntity(MultiValueMap<String, String> headers) {
  this(null, headers);
}

/**
 * Create a new {@code HttpEntity} with the given body and headers.
 * @param body the entity body
 * @param headers the entity headers
 */
public HttpEntity(T body, MultiValueMap<String, String> headers) {
  this.body = body;
  HttpHeaders tempHeaders = new HttpHeaders();
  if (headers != null) {
      tempHeaders.putAll(headers);
  }
  this.headers = HttpHeaders.readOnlyHttpHeaders(tempHeaders);
}
```

## 2、处理响应头

使用 RestTemplate 中`xxxForEntity()`的方法，会返回 ResponseEntity，可以从中获取到响应状态码，响应头和 body 等信息：

```java
HttpHeaders requestHeaders = new HttpHeaders();
requestHeaders.set("MyRequestHeader", "MyValue");
HttpEntity requestEntity = new HttpEntity(requestHeaders);

HttpEntity<String> response = template.exchange(
        "http://example.com/hotels/{hotel}",
        HttpMethod.GET, requestEntity, String.class, "42");

//response相关信息
String responseHeader = response.getHeaders().getFirst("MyResponseHeader");
String body = response.getBody();
```

## 3、ClientHttpRequestFactory

ClientHttpRequestFactory 是 Spring 定义的一个接口，用于生产 org.springframework.http.client.ClientHttpRequest 对象，RestTemplate只是模板类，抽象了很多调用方法，而底层真正使用何种框架发送 HTTP 请求是通过 ClientHttpRequestFactory 指定的。

```java
/**
 * Factory for {@link ClientHttpRequest} objects.
 * Requests are created by the {@link #createRequest(URI, HttpMethod)} method.
 * ClientHttpRequest对象的工厂
 *
 * @author Arjen Poutsma
 * @since 3.0
 */
public interface ClientHttpRequestFactory {

    /**
     * Create a new {@link ClientHttpRequest} for the specified URI and HTTP method.
     * <p>The returned request can be written to, and then executed by calling
     * {@link ClientHttpRequest#execute()}.
     * 使用指定的URI和HTTP方法新建一个ClientHttpRequest对象
     * 可以修改返回的request，并通过ClientHttpRequest的execute()方法执行调用
     * 即调用的逻辑也被Spring封装到ClientHttpRequest中
     * 
     * @param uri the URI to create a request for
     * @param httpMethod the HTTP method to execute
     * @return the created request
     * @throws IOException in case of I/O errors
     */
    ClientHttpRequest createRequest(URI uri, HttpMethod httpMethod) throws IOException;

}
```

RestTemplate 可以在构造时设置 ClientHttpRequestFactory，也可以通过 setRequestFactory() 方法设置：

```java
构造方法设置：
/**
 * Create a new instance of the {@link RestTemplate} based on the given {@link ClientHttpRequestFactory}.
 * @param requestFactory HTTP request factory to use
 * @see org.springframework.http.client.SimpleClientHttpRequestFactory
 * @see org.springframework.http.client.HttpComponentsClientHttpRequestFactory
 */
public RestTemplate(ClientHttpRequestFactory requestFactory) {
    this();
    setRequestFactory(requestFactory);
}
```

可以看到上面注释中已经给出了Spring的两种ClientHttpRequestFactory的实现类`SimpleClientHttpRequestFactory`和`HttpComponentsClientHttpRequestFactory`

### SimpleClientHttpRequestFactory

如果什么都不设置，RestTemplate 默认使用的是 SimpleClientHttpRequestFactory，其内部使用的是 jdk 的java.net.HttpURLConnection 创建底层连接，<font color = red>默认是没有连接池的</font>，connectTimeout 和 readTimeout 都是 **-1**，即<font color = red>没有超时时间</font>。

```java
public class SimpleClientHttpRequestFactory implements ClientHttpRequestFactory, AsyncClientHttpRequestFactory {
    。。。。。。
        
    private int connectTimeout = -1;
    private int readTimeout = -1;
    
    //创建Request
    @Override
    public ClientHttpRequest createRequest(URI uri, HttpMethod httpMethod) throws IOException {
        HttpURLConnection connection = openConnection(uri.toURL(), this.proxy);
        prepareConnection(connection, httpMethod.name());

         //bufferRequestBody默认为true
        if (this.bufferRequestBody) {
            return new SimpleBufferingClientHttpRequest(connection, this.outputStreaming);
        }
        else {
            return new SimpleStreamingClientHttpRequest(connection, this.chunkSize, this.outputStreaming);
        }
    }
    
    
    /**
     * Opens and returns a connection to the given URL.
     * 打开并返回一个指定URL的连接
     * <p>The default implementation uses the given {@linkplain #setProxy(java.net.Proxy) proxy} -
     * if any - to open a connection.
     * @param url the URL to open a connection to
     * @param proxy the proxy to use, may be {@code null}
     * @return the opened connection  返回类型为 java.net.HttpURLConnection
     * @throws IOException in case of I/O errors
     */
    protected HttpURLConnection openConnection(URL url, Proxy proxy) throws IOException {
        URLConnection urlConnection = (proxy != null ? url.openConnection(proxy) : url.openConnection());
        if (!HttpURLConnection.class.isInstance(urlConnection)) {
            throw new IllegalStateException("HttpURLConnection required for [" + url + "] but got: " + urlConnection);
        }
        return (HttpURLConnection) urlConnection;
    }
    
    
    /**
     * Template method for preparing the given {@link HttpURLConnection}.
     * <p>The default implementation prepares the connection for input and output, and sets the HTTP method.
     * @param connection the connection to prepare
     * @param httpMethod the HTTP request method ({@code GET}, {@code POST}, etc.)
     * @throws IOException in case of I/O errors
     */
    protected void prepareConnection(HttpURLConnection connection, String httpMethod) throws IOException {
         //如果connectTimeout大于等于0，设置连接超时时间
        if (this.connectTimeout >= 0) {
            connection.setConnectTimeout(this.connectTimeout);
        }
         //如果readTimeout大于等于0，设置读超时时间
        if (this.readTimeout >= 0) {
            connection.setReadTimeout(this.readTimeout);
        }

        connection.setDoInput(true);

        if ("GET".equals(httpMethod)) {
            connection.setInstanceFollowRedirects(true);
        }
        else {
            connection.setInstanceFollowRedirects(false);
        }

        if ("POST".equals(httpMethod) || "PUT".equals(httpMethod) ||
                "PATCH".equals(httpMethod) || "DELETE".equals(httpMethod)) {
            connection.setDoOutput(true);
        }
        else {
            connection.setDoOutput(false);
        }

        connection.setRequestMethod(httpMethod);
    }
    
    。。。。。。
}
```

### HttpComponentsClientHttpRequestFactory

HttpComponentsClientHttpRequestFactory 底层使用 Apache HttpClient 创建请求，访问远程的 Http 服务，<u>可以使用一个已经配置好的 HttpClient 实例创建 HttpComponentsClientHttpRequestFactory 请求工厂，HttpClient 实例中可以配置连接池和证书等信息</u>

1.   添加HttpClient依赖

     ```xml
     <dependency>
         <groupId>org.apache.httpcomponents</groupId>
         <artifactId>httpclient</artifactId>
         <version>x.x.x</version>   <!-- springboot项目不用指定 -->
     </dependency>
     ```

2.   设置超时时间

     设置超时时间，可以直接使用 Spring 的底层基于 HttpClient 的 HttpComponentsClientHttpRequestFactory，此处设置的是 ClientHttpRequestFactory 级别的全局超时时间

     ```java
     @Configuration  
     public class RestTemplateConfig {  
       
         @Bean  
         public RestTemplate restTemplate() {  
             return new RestTemplate(clientHttpRequestFactory());  
         }  
       
         @Bean 
         private ClientHttpRequestFactory clientHttpRequestFactory() {  
             HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();  
             factory.setConnectTimeout(30 * 1000);  //连接超时时间
             factory.setReadTimeout(60 * 1000);  //读超时时间
             return factory;  
         }  
     } 
     ```

     **注意：**如果通过一个 HttpClient 实例创建 HttpComponentsClientHttpRequestFactory，并通过 HttpClient 指定了 DefaultRequestConfig，设置了 connectTimeout、readTimeout 等，在实际执行请求创建 request 时会与HttpComponentsClientHttpRequestFactory 的配置合并，connectTimeout、socketTimeout、connectionRequestTimeout  以 HttpComponentsClientHttpRequestFactory 的配置为准

     ```kotlin
     HttpComponentsClientHttpRequestFactory：
     /**
      * Merge the given {@link HttpClient}-level {@link RequestConfig} with
      * the factory-level {@link RequestConfig}, if necessary.
      * @param clientConfig the config held by the current    httpClient级别的requestConfig配置
      * @return the merged request config
      * (may be {@code null} if the given client config is {@code null})
      * @since 4.2
      */
     protected RequestConfig mergeRequestConfig(RequestConfig clientConfig) {
         if (this.requestConfig == null) {  // nothing to merge
             return clientConfig;
         }
     
         RequestConfig.Builder builder = RequestConfig.copy(clientConfig);
         int connectTimeout = this.requestConfig.getConnectTimeout();  //HttpComponentsClientHttpRequestFactory级别的配置
         if (connectTimeout >= 0) {
             builder.setConnectTimeout(connectTimeout);
         }
         int connectionRequestTimeout = this.requestConfig.getConnectionRequestTimeout();
         if (connectionRequestTimeout >= 0) {
             builder.setConnectionRequestTimeout(connectionRequestTimeout);
         }
         int socketTimeout = this.requestConfig.getSocketTimeout();
         if (socketTimeout >= 0) {
             builder.setSocketTimeout(socketTimeout);
         }
         return builder.build();
     }
     ```

     上例中虽然没有指定http连接池，但**  HttpComponentsClientHttpRequestFactory无参构造会创建一个HttpClient，并默认使用了连接池配置，MaxTotal=10，DefaultMaxPerRoute=5 **，具体如下：

     ```java
     HttpComponentsClientHttpRequestFactory：
     /**
      * Create a new instance of the {@code HttpComponentsClientHttpRequestFactory}
      * with a default {@link HttpClient}.
      */
     public HttpComponentsClientHttpRequestFactory() {
         this(HttpClients.createSystem());
     }
     
     
     HttpClients：
     /**
      * Creates {@link CloseableHttpClient} instance with default
      * configuration based on system properties.
      * 创建CloseableHttpClient实例使用基于system properties的默认配置
      */
     public static CloseableHttpClient createSystem() {
         return HttpClientBuilder.create().useSystemProperties().build();
     }
     
     
     HttpClientBuilder：
     /**
      * Use system properties when creating and configuring default
      * implementations.
      */
     public final HttpClientBuilder useSystemProperties() {
         this.systemProperties = true;  //设置systemProperties为true
         return this;
     }
     
     public CloseableHttpClient build() {
         HttpClientConnectionManager connManagerCopy = this.connManager; //没有设置，为null
         if (connManagerCopy == null) {
             。。。。。。
             //创建连接池管理器PoolingHttpClientConnectionManager
             @SuppressWarnings("resource")
             final PoolingHttpClientConnectionManager poolingmgr = new PoolingHttpClientConnectionManager(
                     RegistryBuilder.<ConnectionSocketFactory>create()
                         .register("http", PlainConnectionSocketFactory.getSocketFactory())
                         .register("https", sslSocketFactoryCopy)
                         .build(),
                     null,
                     null,
                     dnsResolver,
                     connTimeToLive,
                     connTimeToLiveTimeUnit != null ? connTimeToLiveTimeUnit : TimeUnit.MILLISECONDS);
             if (defaultSocketConfig != null) {
                 poolingmgr.setDefaultSocketConfig(defaultSocketConfig);
             }
             if (defaultConnectionConfig != null) {
                 poolingmgr.setDefaultConnectionConfig(defaultConnectionConfig);
             }
             //由于是HttpClientBuilder.create().useSystemProperties().build(),systemProperties为true
             if (systemProperties) {
                 String s = System.getProperty("http.keepAlive", "true");  //http.keepAlive默认值为true
                 if ("true".equalsIgnoreCase(s)) {
                     s = System.getProperty("http.maxConnections", "5");  //默认值为5
                     final int max = Integer.parseInt(s);
                     poolingmgr.setDefaultMaxPerRoute(max);  //DefaultMaxPerRoute=5
                     poolingmgr.setMaxTotal(2 * max);  //MaxTotal=10
                 }
             }
             if (maxConnTotal > 0) {
                 poolingmgr.setMaxTotal(maxConnTotal);
             }
             if (maxConnPerRoute > 0) {
                 poolingmgr.setDefaultMaxPerRoute(maxConnPerRoute);
             }
             connManagerCopy = poolingmgr;
         }
     }
     ```

3.   配置连接池

     ```java
     @Configuration  
     public class RestTemplateConfig {  
       
         @Bean  
         public RestTemplate restTemplate() {  
             return new RestTemplate(clientHttpRequestFactory());  
         }  
       
         @Bean
         public HttpComponentsClientHttpRequestFactory clientHttpRequestFactory() {
             try {
                 HttpClientBuilder httpClientBuilder = HttpClientBuilder.create();
     
                 //开始设置连接池
                 PoolingHttpClientConnectionManager poolingHttpClientConnectionManager 
                                                         = new PoolingHttpClientConnectionManager();
                 poolingHttpClientConnectionManager.setMaxTotal(100);  //最大连接数
                 poolingHttpClientConnectionManager.setDefaultMaxPerRoute(20);  //同路由并发数
                 httpClientBuilder.setConnectionManager(poolingHttpClientConnectionManager);
     
                 HttpClient httpClient = httpClientBuilder.build();
                 // httpClient连接配置
                 HttpComponentsClientHttpRequestFactory clientHttpRequestFactory 
                                                         = new HttpComponentsClientHttpRequestFactory(httpClient);
                 clientHttpRequestFactory.setConnectTimeout(30 * 1000);  //连接超时
                 clientHttpRequestFactory.setReadTimeout(60 * 1000);     //数据读取超时时间
                 clientHttpRequestFactory.setConnectionRequestTimeout(30 * 1000);  //连接不够用的等待时间
                 return clientHttpRequestFactory;
             }
             catch (Exception e) {
                 logger.error("初始化clientHttpRequestFactory出错", e);
             }
             return null;
         } 
     } 
     ```

## 4、自定义 MessageConverter

RestTemplate 的无参构造中默认会初始化很多 messageConverters，用于请求/响应中的消息转换

```java
/**
 * Create a new instance of the {@link RestTemplate} using default settings.
 * Default {@link HttpMessageConverter}s are initialized.
 * 使用默认配置创建一个RestTemplate实例
 * 默认的HttpMessageConverter集合被初始化
 */
public RestTemplate() {
    this.messageConverters.add(new ByteArrayHttpMessageConverter());
    this.messageConverters.add(new StringHttpMessageConverter());
    this.messageConverters.add(new ResourceHttpMessageConverter());
    this.messageConverters.add(new SourceHttpMessageConverter<Source>());
    this.messageConverters.add(new AllEncompassingFormHttpMessageConverter());

    if (romePresent) {
        this.messageConverters.add(new AtomFeedHttpMessageConverter());
        this.messageConverters.add(new RssChannelHttpMessageConverter());
    }

    if (jackson2XmlPresent) {
        this.messageConverters.add(new MappingJackson2XmlHttpMessageConverter());
    }
    else if (jaxb2Present) {
        this.messageConverters.add(new Jaxb2RootElementHttpMessageConverter());
    }

    /**
     * 如果类路径下包含com.fasterxml.jackson.databind.ObjectMapper 和 com.fasterxml.jackson.core.JsonGenerator
     * 使用jackson做http请求、响应的json转换
     */
    if (jackson2Present) {
        this.messageConverters.add(new MappingJackson2HttpMessageConverter());
    }
    else if (gsonPresent) {  //类路径下包含 com.google.gson.Gson
        this.messageConverters.add(new GsonHttpMessageConverter());
    }
}
```

Springboot 项目默认使用 jackson 做 json 转换

使用 fastjson 做 json 转换：

1.  引入 fastjson 依赖
2.  排除 jackson 的 HttpMessageConverter 转换器
3.  添加 fastjson 的转换器

排除 jackson 的 HttpMessageConverter 转换器有两种方式：

1.   类路径下去掉 jackson 的支持

     ```xml
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
         <exclusions>
             <exclusion>
                 <artifactId>jackson-databind</artifactId> 
                 <groupId>com.fasterxml.jackson.core</groupId>
             </exclusion>
         </exclusions>
     </dependency>
     ```

2.   在初始化配置 RestTemplate 时，去掉其默认的 MappingJackson2HttpMessageConverter

     ```java
     @Bean
     public RestTemplate restTemplate() {
         RestTemplate restTemplate = new RestTemplate();
         restTemplate.setRequestFactory(clientHttpRequestFactory());
     
         //restTemplate默认的HttpMessageConverter
         List<HttpMessageConverter<?>> messageConverters = restTemplate.getMessageConverters();
         List<HttpMessageConverter<?>> messageConvertersNew = new ArrayList<HttpMessageConverter<?>>();
         
         for(HttpMessageConverter httpMessageConverter : messageConverters){
             //跳过MappingJackson2HttpMessageConverter
             if (httpMessageConverter instanceof MappingJackson2HttpMessageConverter) continue;
     
             messageConvertersNew.add(httpMessageConverter);
         }
     
         //添加fastjson转换器
         messageConvertersNew.add(fastJsonHttpMessageConverter());
     
         return restTemplate;
     }
     
     @Bean
     public HttpMessageConverter fastJsonHttpMessageConverter() {
         //MediaType
         List<MediaType> mediaTypes = new ArrayList<>();
         mediaTypes.add(MediaType.APPLICATION_JSON_UTF8);
     
         //FastJsonConfig
         FastJsonConfig fastJsonConfig = new FastJsonConfig();
         fastJsonConfig.setSerializerFeatures(SerializerFeature.WriteMapNullValue,
                                              SerializerFeature.QuoteFieldNames);
     
         //创建FastJsonHttpMessageConverter4    Spring 4.2后使用
         FastJsonHttpMessageConverter4 fastJsonHttpMessageConverter = new FastJsonHttpMessageConverter4();
         fastJsonHttpMessageConverter.setSupportedMediaTypes(mediaTypes);
         fastJsonHttpMessageConverter.setFastJsonConfig(fastJsonConfig);
     
         return fastJsonHttpMessageConverter;
     }
     ```
