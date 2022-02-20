# RestTemplate 自定义

RestTemplate 自定义主要有三种方法，具体取决于希望自定义应用的范围

1.  **类范围**。为了尽量缩小自定义的范围，在类中注入自动配置的`RestTemplateBuilder`，然后根据需求调用它的配置方法，每次调用配置方法都会 new RestTemplateBuilder() 并返回，所以对 RestTemplateBuilder 的配置只会影响由它创建的 RestTemplate

2.  **应用范围**。可以使用`RestTemplateCustomizer`来自定义应用范围的的 RestTemplate，所有注册到 Spring 容器的`RestTemplateCustomizer`都会自动生效。如下，通过 RestTemplateCustomizer 设置连接池

    ```reasonml
    @Bean
        public RestTemplateCustomizer restTemplateCustomizer(){
            return new RestTemplateCustomizer(){
                @Override
                public void customize(RestTemplate restTemplate) {
                    HttpClientBuilder httpClientBuilder = HttpClientBuilder.create();
    
                    //创建连接管理器
                    PoolingHttpClientConnectionManager poolingHttpClientConnectionManager 
                                             = new PoolingHttpClientConnectionManager();
                    poolingHttpClientConnectionManager.setMaxTotal(100);
                    poolingHttpClientConnectionManager.setDefaultMaxPerRoute(20);
                    httpClientBuilder.setConnectionManager(poolingHttpClientConnectionManager);
    
                    //创建httpClient
                    HttpClient httpClient = httpClientBuilder.build();
    
                    //创建HttpComponentsClientHttpRequestFactory
                    HttpComponentsClientHttpRequestFactory httpComponentsClientHttpRequestFactory 
                                                 = new HttpComponentsClientHttpRequestFactory(httpClient);
                    httpComponentsClientHttpRequestFactory.setConnectTimeout(10 * 1000);
                    httpComponentsClientHttpRequestFactory.setReadTimeout(60 * 1000);
                    httpComponentsClientHttpRequestFactory.setConnectionRequestTimeout(20 * 1000);
    
                    restTemplate.setRequestFactory(httpComponentsClientHttpRequestFactory);
                }
            };
        }
    ```

3.  最后，最极端的（也是很少使用的）选项是创建你自己的`RestTemplateBuilder` bean。这将关闭`RestTemplateBuilder`的自动配置，并阻止使用任何`RestTemplateCustomizer` bean