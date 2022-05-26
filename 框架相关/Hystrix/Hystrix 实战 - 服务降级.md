# Hystrix 实战 - 服务降级

[toc]

依赖版本管理：

```xml
<!--参数管理-->
<properties>
    <!--环境参数：开发、编译-->
    <java.version>1.8</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <build.plugins.plugin.version>2.3.7.RELEASE</build.plugins.plugin.version>
    <maven.compiler.plugin>3.8.1</maven.compiler.plugin>
    <!--依赖参数：版本-->
    <spring.boot.version>2.2.2.RELEASE</spring.boot.version>
    <spring.cloud.version>Hoxton.RELEASE</spring.cloud.version>
    <spring.cloud.alibaba.version>2.1.0.RELEASE</spring.cloud.alibaba.version>
    <junit.version>4.12</junit.version>
    <log4j.version>1.2.17</log4j.version>
    <lombok.version>1.16.18</lombok.version>
    <mysql.version>8.0.25</mysql.version>
    <druid.spring.boot.version>1.1.13</druid.spring.boot.version>
    <mybatis.spring.boot.version>2.2.0</mybatis.spring.boot.version>
    <lombok.version>1.18.20</lombok.version>
    <zookeeper.discovery.version>3.0.4</zookeeper.discovery.version>
    <starter.openfeign.version>2.2.9.RELEASE</starter.openfeign.version>
    <starter.hystrix.version>1.4.7.RELEASE</starter.hystrix.version>
</properties>

<!--依赖管理：锁定版本-->
<dependencyManagement>
    <dependencies>
        <!--spring boot-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring.boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!--spring cloud-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring.cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!--spring cloud alibaba-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>${spring.cloud.alibaba.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>${mysql.version}</version>
        </dependency>
        <!--druid-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>${druid.spring.boot.version}</version>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>${mybatis.spring.boot.version}</version>
        </dependency>
        <!--spring-boot-configuration-processor-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <version>${spring.boot.version}</version>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
        </dependency>
        <!--zookeeper-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <version>${zookeeper.discovery.version}</version>
        </dependency>
        <!--openFeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>${starter.openfeign.version}</version>
        </dependency>
        <!--hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>${starter.hystrix.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## 1、服务提供方

### 依赖文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns = "http://maven.apache.org/POM/4.0.0"
         xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation = "http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-cloud-demo-2020</artifactId>
        <groupId>org.xzy</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    
    <modelVersion>4.0.0</modelVersion>
    <artifactId>cloud-service-user</artifactId>
    <description>用户服务</description>
    
    <dependencies>
        <!--other module-->
        <dependency>
            <groupId>org.xzy</groupId>
            <artifactId>cloud-api-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--spring-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--spring-boot-tools-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <!--spring-boot-configuration-processor-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <!--eureka-client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!--hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
    </dependencies>
</project>
```

注意：spring-cloud-starter-netflix-hystrix

### 配置文件

```yaml
server:
  port: 8001

spring:
  application:
    name: cloud-user-service

eureka:
  instance:
    instance-id: cloud-user-service8001
    prefer-ip-address: true
  client:
    register-with-eureka: true # 是否向注册中心注册自己
    fetch-registry: true # 是否需要检索服务
    service-url:
      defaultZone: http://eureka-register1:7001/eureka/,http://eureka-register2:7002/eureka/,http://eureka-register3:7003/eureka/
```

### 主启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableHystrix // Mark
public class UserApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
    }
}
```

注意：@EnableHystrix

### 业务代码

```java
@RestController
@RequestMapping("/user/delay")
@Slf4j
public class DelayController {
    @Autowired
    private final DelayService delayService;

    @GetMapping("/delay") // 没有配置服务降级的接口
    public MessageBox<String> delayRpo() {
        return delayService.delayRpo();
    }

    @GetMapping("/delay_with_fallback") // 配置了服务降级的接口
    public MessageBox<String> delayRpoWithFallback() {
        return delayService.delayRpoWithFallback();
    }
}
```

```java
@Service
@Slf4j
@DefaultProperties(defaultFallback = "defaultFallback")
public class DelayServiceImpl implements DelayService {
    @Value("${server.port}")
    private String serverPort;

    /**
     * 直接躺平：慢慢处理 + 抛出异常
     */
    @Override
    public MessageBox<String> delayRpo() {
        // 模拟：出现异常
        if (Math.random() > 0.7) { // 30%概率的出现异常
            log.info("抛出异常");
            throw new IllegalArgumentException("随机异常");
        }

        // 模拟：处理缓慢
        int delayMilliseconds = (int) (Math.random() * 10000); // 随机处理时间 0~10000ms
        try {
            log.info("执行时间：{}ms", delayMilliseconds);
            TimeUnit.MILLISECONDS.sleep(delayMilliseconds);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        log.info("正常处理...");
        String msg = "Server port：" + serverPort + " UUID：" + UUID.randomUUID().toString() + " Current thread：" + Thread.currentThread();
        return MessageBox.ok(msg);
    }

    /**
     * 服务降级：如果处理超时或出现异常，执行 ”Plan B“
     */
    @Override
    @HystrixCommand(fallbackMethod = "planB", commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000") // 超时时间
    })
    public MessageBox<String> delayRpoWithFallback() {
        return delayRpo();
    }

    /*----------fallback method----------*/

    public MessageBox<String> planB() {
        log.info("处理失败，出现异常或超时，进行服务降级...");
        String msg = "Server port：" + serverPort + " UUID：" + UUID.randomUUID().toString() + " Current thread：" + Thread.currentThread() + " (Plan B：下行服务)";
        return MessageBox.ok(msg);
    }

    public MessageBox<String> defaultFallback(){
        log.info("指定默认的服务降级方法");
        String msg = "Server port：" + serverPort + " UUID：" + UUID.randomUUID().toString() + " Current thread：" + Thread.currentThread() + " (defaultFallback：下行服务)";
        return MessageBox.ok(msg);
    }
}
```

在上述 ServiceImpl 中，delayRpo() 是一个可能处理时间很长，也可能抛出异常的方法。在上述 Controller 中，/delay 接口直接调用了 delayRpo() 方法，因此服务消费方调用 /deley 接口时，可能响应超时，也可能直接获取到 delayRpo() 抛出的异常信息。

delayRpoWithFallback() 在 delayRpo() 的基础上利用 Hystrix  配置了服务降级方法，按照配置，当 delayRpoWithFallback() 的处理时间超过 3000 毫秒，或者抛出了异常，系统将执行 planB()，并以 planB() 的响应作为 delayRpoWithFallback() 响应作为返回。因此，服务消费方调用 /delay_with_fallback 接口时，总是能在 3000 毫秒内获取到正常的响应。

根据配置 @DefaultProperties(defaultFallback = "defaultFallback") 的配置，如果使用 @HystrixCommand 时没有指定降级方法，那么以 defaultFallback() 作为降级方法。因此可以使用 <font color = red>@DefaultProperties</font> 配置全局通用的降级方法，以避免为每个方法创建降级方法。

## 2、服务消费方

### 依赖文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns = "http://maven.apache.org/POM/4.0.0"
         xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation = "http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-cloud-demo-2020</artifactId>
        <groupId>org.xzy</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    
    <modelVersion>4.0.0</modelVersion>
    <artifactId>cloud-service-payment</artifactId>
    <description>支付服务</description>
    
    <dependencies>
        <!--other module-->
        <dependency>
            <groupId>org.xzy</groupId>
            <artifactId>cloud-api-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--spring-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--spring-boot-tools-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <!--spring-boot-configuration-processor-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <!--eureka-client-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!--ribbon-->
        <!--spring-cloud-starter-netflix-eureka-client已经包含了ribbon，所以可以不用额外引入-->
        <!--openFeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
    </dependencies>

</project>
```

### 配置文件

```yaml
server:
  port: 9001

spring:
  application:
    name: cloud-payment-service
  main:
    allow-bean-definition-overriding: true

eureka:
  instance:
    instance-id: cloud-payment-service9001
    prefer-ip-address: true
  client:
    register-with-eureka: true # 是否向注册中心注册自己
    fetch-registry: true # 是否需要检索服务
    service-url:
      defaultZone: http://eureka-register1:7001/eureka/,http://eureka-register2:7002/eureka/,http://eureka-register3:7003/eureka/

feign:
  httpclient:
    connection-timeout: 10000 # 连接超时，默认2秒
  hystrix:
    enabled: true # 服务降级

ribbon:
  ReadTimeout: 60000
  ConnectTimeout: 60000

logging:
  level:
    com.xzy.feign: debug
```

注意：由于我们使用 OpenFeign 进行接口调用，因此需要配置 <font color = red>feign.hystrix.enabled</font> = true

### 主启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient
@EnableFeignClients
@EnableHystrix // Mark
public class PaymentApplication {
    public static void main(String[] args) {
        SpringApplication.run(PaymentApplication.class, args);
    }
}
```

### 业务代码

```java
@Service
@FeignClient(value = "cloud-user-service",fallback = UserServiceFeignFallback.class)
public interface UserServiceFeign {

    @GetMapping("/user/delay/delay") // 调用没有设置服务降级的远程接口
    MessageBox<String> delayRpo();

    @GetMapping("/user/delay/delay_with_fallback") // 调用设置了服务降级的远程接口
    MessageBox<String> delayRpoWithFallback();
}
```

```java
@Slf4j
@Service
public class UserServiceFeignFallback implements UserServiceFeign {

    @Value("${server.port}")
    private String serverPort;

    @Override
    public MessageBox<String> delayRpo() {
        log.info("delayRpo() 处理失败，出现异常或超时，进行服务降级...");
        String msg = "Server port：" + serverPort + " UUID：" + UUID.randomUUID().toString() + " Current thread：" + Thread.currentThread() + " (Plan B：上行服务)";
        return MessageBox.ok(msg);
    }

    @Override
    public MessageBox<String> delayRpoWithFallback() {
        log.info("delayRpoWithFallback() 处理失败，出现异常或超时，进行服务降级...");
        String msg = "Server port：" + serverPort + " UUID：" + UUID.randomUUID().toString() + " Current thread：" + Thread.currentThread() + " (Plan B：上行服务)";
        return MessageBox.ok(msg);
    }
}
```

以往使用 Feign 的时候是不需要实现 Feign 接口的，但是当 Feign 与 Hystrix 配合一起使用时，可以为 Feign 配置实现类，<u>实现类中的每一个方法都相当于远程接口的本地降级方法</u>。当然，这种方式是非必须的，如果只需要为个别方法配置服务降级，可以在 Controller 那里使用 @HystrixCommand 进行配置。