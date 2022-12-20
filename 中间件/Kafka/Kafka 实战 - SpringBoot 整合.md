# SpringBoot 整合 Kafka

[TOC]

---

Maven：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns = "http://maven.apache.org/POM/4.0.0"
         xmlns:xsi = "http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation = "http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>org.xzy</groupId>
    <artifactId>kafka-demo</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    
    <modules>
        <module>producer</module>
        <module>consumer</module>
    </modules>
    
    <properties>
        <!--环境参数：开发、编译-->
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <build.plugins.plugin.version>2.3.7.RELEASE</build.plugins.plugin.version>
        <maven.compiler.plugin>3.8.1</maven.compiler.plugin>
    </properties>
    
    <dependencyManagement>
        <dependencies>
            <!--lombok-->
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>1.16.18</version>
            </dependency>
            <!--web-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
                <version>2.3.4.RELEASE</version>
            </dependency>
            <!--kafka-->
            <dependency>
                <groupId>org.springframework.kafka</groupId>
                <artifactId>spring-kafka</artifactId>
                <version>2.5.6.RELEASE</version>
            </dependency>
            <!--test-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <version>2.3.4.RELEASE</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${build.plugins.plugin.version}</version>
                <configuration>
                    <fork>true</fork>
                    <addResources>true</addResources>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>${maven.compiler.plugin}</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```



## 一、生产者

### 配置

SpringBoot 配置 Kafka 的方式有两种：

1.   配置文件

     ```properties
     spring:
       application:
         name: hello-kafka
       kafka:
         listener:
           #设置是否批量消费，默认 single（单条），batch（批量）
           type: single
         # 集群地址
         bootstrap-servers: kafka1:9093,kafka2:9094,kafka3:9095
         # === 生产者配置 ===
         producer:
           # 重试次数
           retries: 3
           # 应答级别
           # acks=0 把消息发送到kafka就认为发送成功
           # acks=1 把消息发送到kafka leader分区，并且写入磁盘就认为发送成功
           # acks=all 把消息发送到kafka leader分区，并且leader分区的副本follower对消息进行了同步就任务发送成功
           acks: all
           # 批量处理的最大大小 单位 byte
           batch-size: 4096
           # 发送延时,当生产端积累的消息达到batch-size或接收到消息linger.ms后,生产者就会将消息提交给kafka
           buffer-memory: 33554432
           # 客户端ID
           client-id: hello-kafka
           # Key 序列化类
           key-serializer: org.apache.kafka.common.serialization.StringSerializer
           # Value 序列化类
           value-serializer: org.apache.kafka.common.serialization.StringSerializer
           # 消息压缩：none、lz4、gzip、snappy，默认为 none。
           compression-type: gzip
           properties:
             partitioner:
               #指定自定义分区器
               class: top.zysite.hello.kafka.partitioner.MyPartitioner
             linger:
               # 发送延时,当生产端积累的消息达到batch-size或接收到消息linger.ms后,生产者就会将消息提交给kafka
               ms: 1000
             max:
               block:
                 # KafkaProducer.send() 和 partitionsFor() 方法的最长阻塞时间 单位 ms
                 ms: 6000
         # === 消费者配置 ===
         consumer:
           # 默认消费者组
           group-id: testGroup
           # 自动提交 offset 默认 true
           enable-auto-commit: false
           # 自动提交的频率 单位 ms
           auto-commit-interval: 1000
           # 批量消费最大数量
           max-poll-records: 100
           # Key 反序列化类
           key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
           # Value 反序列化类
           value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
           # 当kafka中没有初始offset或offset超出范围时将自动重置offset
           # earliest:重置为分区中最小的offset
           # latest:重置为分区中最新的offset(消费分区中新产生的数据)
           # none:只要有一个分区不存在已提交的offset,就抛出异常
           auto-offset-reset: latest
           properties:
             interceptor:
               classes: top.zysite.hello.kafka.interceptor.MyConsumerInterceptor
             session:
               timeout:
                 # session超时，超过这个时间consumer没有发送心跳,就会触发rebalance操作
                 ms: 120000
             request:
               timeout:
                 # 请求超时
                 ms: 120000
     ```

2.   配置类

     ```java
     package com.xzy.config;
     
     import org.apache.kafka.clients.producer.ProducerConfig;
     import org.springframework.context.annotation.Bean;
     import org.springframework.context.annotation.Configuration;
     import org.springframework.kafka.annotation.EnableKafka;
     import org.springframework.kafka.core.DefaultKafkaProducerFactory;
     import org.springframework.kafka.core.KafkaTemplate;
     
     import java.util.HashMap;
     import java.util.Map;
     
     /**
      * Kafka 生产者配置类
      *
      * @author xzy.xiao
      * @date 2022/11/9  20:48
      */
     @EnableKafka
     @Configuration
     public class KafkaProducerConfig {
     
         public DefaultKafkaProducerFactory<String, String> producerFactory() {
             Map<String, Object> props = new HashMap<>();
             // kafka 集群地址
             props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka1:9093,kafka2:9094,kafka3:9095");
             // 重试次数
             props.put(ProducerConfig.RETRIES_CONFIG, 3);
             // 应答级别
             // acks=0 把消息发送到kafka就认为发送成功
             // acks=1 把消息发送到kafka leader分区，并且写入磁盘就认为发送成功
             // acks=all 把消息发送到kafka leader分区，并且leader分区的副本follower对消息进行了同步就任务发送成功
             props.put(ProducerConfig.ACKS_CONFIG, "all");
             // KafkaProducer.send() 和 partitionsFor() 方法的最长阻塞时间 单位 ms
             props.put(ProducerConfig.MAX_BLOCK_MS_CONFIG, 6000);
             // 批量处理的最大大小 单位 byte
             props.put(ProducerConfig.BATCH_SIZE_CONFIG, 4096);
             // 发送延时,当生产端积累的消息达到batch-size或接收到消息linger.ms后,生产者就会将消息提交给kafka
             props.put(ProducerConfig.LINGER_MS_CONFIG, 1000);
             // 生产者可用缓冲区的最大值 单位 byte
             props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);
             // 每条消息最大的大小
             props.put(ProducerConfig.MAX_REQUEST_SIZE_CONFIG, 1048576);
             // Key 序列化方式
             props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, org.apache.kafka.common.serialization.StringSerializer.class);
             // Value 序列化方式
             props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, org.apache.kafka.common.serialization.StringSerializer.class);
             // 消息压缩：none、lz4、gzip、snappy，默认为 none。
             props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "gzip");
             // 自定义分区器
             // props.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, MyPartitioner.class.getName());
             return new DefaultKafkaProducerFactory<>(props);
         }
     
         /**
          * @return 不包含事务的 Template
          */
         @Bean("kafkaTemplate")
         public KafkaTemplate<String, String> kafkaTemplate() {
             return new KafkaTemplate<>(producerFactory());
         }
     
         /**
          * @return 包含事务的 Template
          */
         @Bean("kafkaTemplateWithTransaction")
         public KafkaTemplate<String, String> kafkaTemplateWithTransaction() {
             // 设置事务Id前缀
             DefaultKafkaProducerFactory<String, String> defaultKafkaProducerFactory = producerFactory();
             defaultKafkaProducerFactory.setTransactionIdPrefix("tx");
     
             return new KafkaTemplate<>(defaultKafkaProducerFactory);
         }
     }
     ```

### 生产消息

```java
package com.xzy.service;

import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.producer.RecordMetadata;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaOperations;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.stereotype.Service;
import org.springframework.util.concurrent.ListenableFutureCallback;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

/**
 * 说明：Kafka 生产者
 *
 * @author xzy
 * @date 2022/11/13  15:42
 */
@Slf4j
@Service
public class KafkaProducerService {
    private final KafkaTemplate<String, String> kafkaTemplate;
    private final KafkaTemplate<String, String> kafkaTemplateWithTransaction;

    @Autowired
    public KafkaProducerService(KafkaTemplate<String, String> kafkaTemplate, KafkaTemplate<String, String> kafkaTemplateWithTransaction) {
        this.kafkaTemplate = kafkaTemplate;
        this.kafkaTemplateWithTransaction = kafkaTemplateWithTransaction;
    }

    /**
     * 发送消息（同步）
     *
     * @param topic   主题
     * @param message 消息
     */
    public void sendMessageSync(String topic, String message) {
        log.info("sendMessageSync => topic：{}, message：{}", topic, message);
        kafkaTemplate.send(topic, message);
    }

    /**
     * 发送消息（同步）
     *
     * @param topic    主题
     * @param message  消息
     * @param timeout  超时时间
     * @param timeUnit 时间单位
     * @throws ExecutionException   -
     * @throws InterruptedException -
     * @throws TimeoutException     发送超时
     */
    public void sendMessageSync(String topic, String message, long timeout, TimeUnit timeUnit) throws ExecutionException, InterruptedException, TimeoutException {
        log.info("sendMessageSync => topic：{}, message：{}, timeout：{}({})", topic, message, timeout, timeUnit);
        kafkaTemplate.send(topic, message).get(timeout, timeUnit);
    }

    /**
     * 发送消息并获取结果
     *
     * @param topic   主题
     * @param message 消息
     */
    public void sendMessageAndGetResult(String topic, String message) throws ExecutionException, InterruptedException {
        log.info("sendMessageSync => topic：{}, message：{}", topic, message);
        SendResult<String, String> sendResult = kafkaTemplate.send(topic, message).get();
        RecordMetadata recordMetadata = sendResult.getRecordMetadata();
        log.info("SendResult => partition：{}", recordMetadata.partition());
    }

    /**
     * 发送消息（同步）
     *
     * @param topic   主题
     * @param key     kafka根据key进行hash，决定message存到哪个partition
     * @param message 消息
     */
    public void sendMessageSync(String topic, String key, String message) throws ExecutionException, InterruptedException {
        log.info("sendMessageSync => topic：{}, key：{}, message：{}", topic, key, message);
        SendResult<String, String> sendResult = kafkaTemplate.send(topic, key, message).get();
        RecordMetadata recordMetadata = sendResult.getRecordMetadata();
        log.info("SendResult => partition：{}", recordMetadata.partition());
    }

    /**
     * 发送消息（异步）
     *
     * @param topic   主题
     * @param message 消息
     */
    public void sendMessageAsync(String topic, String message) {
        log.info("sendMessageAsync => topic：{}, message：{}", topic, message);
        kafkaTemplate.send(topic, message)
                .addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
                    @Override
                    public void onFailure(Throwable ex) {
                        log.error("sendMessageAsync failure! => topic：{}, message：{}", topic, message);
                    }

                    @Override
                    public void onSuccess(SendResult<String, String> result) {
                        log.info("sendMessageAsync success! => topic：{}, message：{}", topic, message);
                    }
                });
    }

    /**
     * 以事务方式发送消息
     *
     * @param topic   主题
     * @param message 消息
     */
    public void sendMessageInTransaction(String topic, String message) {
        kafkaTemplateWithTransaction.executeInTransaction(new KafkaOperations.OperationsCallback<String, String, Object>() {
            @Override
            public Object doInOperations(KafkaOperations<String, String> operations) {
                operations.send(topic, message);
                throw new RuntimeException("exception"); // 异常会中断事务，消息并不会发送出去
            }
        });
    }
}
```

#### 测试

```java
package com.xzy.service;


import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.concurrent.ExecutionException;

/**
 * 说明：
 *
 * @author xzy
 * @date 2022/11/13  16:50
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class KafkaProducerServiceTest {

    private final String TEST_TOPIC = "myTopic123";

    @Autowired
    private KafkaProducerService kafkaProducerService;

    /**
     * 同步的消息发送
     */
    @Test
    public void sendMessageSync() {
        for (int i = 0; i < 10; i++) {
            String msg = "msg-" + System.currentTimeMillis();
            kafkaProducerService.sendMessageSync(TEST_TOPIC, msg);
        }
    }

    /**
     * 指定key的消息发送
     */
    @Test
    public void sendMessageSync1() throws ExecutionException, InterruptedException {
        for (int i = 0; i < 100; i++) {
            for (int j = 0; j < 3; j++) {
                String key = "key-" + i;
                String msg = "msg-" + System.currentTimeMillis();
                kafkaProducerService.sendMessageSync(TEST_TOPIC, key, msg);
                // 从控制台打印的信息可以看出，设置了相同的key的message会发到相同的partition
            }
        }
    }

    /**
     * 异步的消息发送
     */
    @Test
    public void sendMessageAsync() throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            String msg = "msg-" + System.currentTimeMillis();
            kafkaProducerService.sendMessageAsync(TEST_TOPIC, msg);
        }

        // 等待消息发送完成！
        Thread.sleep(5000);
    }

    /**
     * 带有事务的消息发送
     */
    @Test
    public void sendMessageInTransaction() {
        kafkaProducerService.sendMessageInTransaction(TEST_TOPIC, "msg");
    }
}
```

## 二、消费者

### 配置

```java
package com.xzy.config;

import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.consumer.Consumer;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.listener.ConcurrentMessageListenerContainer;
import org.springframework.kafka.listener.ConsumerAwareListenerErrorHandler;
import org.springframework.kafka.listener.ListenerExecutionFailedException;
import org.springframework.messaging.Message;

import java.util.HashMap;
import java.util.Map;

/**
 * Kafka 消费者配置类
 *
 * @author xzy.xiao
 * @date 2022/11/9  20:48
 */
@Slf4j
@EnableKafka
@Configuration
public class KafkaConsumerConfig {

    public ConsumerFactory<String, Object> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        //kafka集群地址
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka1:9093,kafka2:9094,kafka3:9095");
        //自动提交 offset 默认 true
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        //自动提交的频率 单位 ms
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, 1000);
        //批量消费最大数量
        props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 100);
        //消费者组
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "testGroup");
        //session超时，超过这个时间consumer没有发送心跳,就会触发rebalance操作
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 120000);
        //请求超时
        props.put(ConsumerConfig.REQUEST_TIMEOUT_MS_CONFIG, 120000);
        //Key 反序列化类
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, org.apache.kafka.common.serialization.StringDeserializer.class);
        //Value 反序列化类
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, org.apache.kafka.common.serialization.StringDeserializer.class);
        //当kafka中没有初始offset或offset超出范围时将自动重置offset
        //earliest:重置为分区中最小的offset
        //latest:重置为分区中最新的offset(消费分区中新产生的数据)
        //none:只要有一个分区不存在已提交的offset,就抛出异常
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "latest");
        //设置Consumer拦截器
        //props.put(ConsumerConfig.INTERCEPTOR_CLASSES_CONFIG, MyConsumerInterceptor.class.getName());
        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        //设置 consumerFactory
        factory.setConsumerFactory(consumerFactory());
        //设置是否开启批量监听
        factory.setBatchListener(false);
        //设置消费者组中的线程数量
        factory.setConcurrency(1);
        return factory;
    }

    /**
     * 消费异常处理器
     *
     * @return -
     */
    @Bean
    public ConsumerAwareListenerErrorHandler consumerAwareListenerErrorHandler() {
        return new ConsumerAwareListenerErrorHandler() {
            @Override
            public Object handleError(Message<?> message, ListenerExecutionFailedException exception, Consumer<?, ?> consumer) {
                log.error("consumer failed! message: {}, exceptionMsg: {}, groupId: {}", message, exception.getMessage(), exception.getGroupId());
                return null;
            }
        };
    }
}
```

### 消费消息

```java
package com.xzy.service;

import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * 说明：Kafka 消费者
 *
 * @author xzy
 * @date 2022/11/13  19:40
 */
@Slf4j
@Service
public class KafkaConsumerService {

    /**
     * 逐条消费
     */
    @KafkaListener(id = "consumeSingle", topics = {"myTopic123"})
    public void consumeSingle(String message) {
        log.info("consumeSingle => message：{}", message);
    }

    /*

        指定异常处理器：
        @KafkaListener(errorHandler = "consumerAwareListenerErrorHandler")

        指定分区：
        @KafkaListener(topicPartitions = {
                @TopicPartition(topic = "hello-batch1", partitions = {"0"})
        })

     */

    /**
     * 批量消费
     */
    @KafkaListener(id = "consumeBatch", topics = "hello-batch")
    public void consumeBatch(List<ConsumerRecord<String, String>> consumerRecordList) {
        int msgSize = consumerRecordList.size();
        log.info("consumeBatch => messageSize：{}", msgSize);
        for (int i = 0; i < msgSize; i++) {
            log.info("consumeBatch => message {}：{}", i, consumerRecordList.get(i));
        }
    }
}
```

#### 测试