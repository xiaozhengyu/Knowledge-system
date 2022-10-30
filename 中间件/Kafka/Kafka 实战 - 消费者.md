# 消费者

[TOC]

## 单个消费者-订阅主题

### 代码

```java
package com.xzy.consumer.a;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;

/**
 * 说明：独立消费者-订阅主题
 *
 * @author xzy
 * @date 2022/10/30  19:21
 * @see com.xzy.producer.b.SystemPartitionerProducer
 */
public class SingleConsumerAndTopicConsume {
    private final static Logger LOGGER = LoggerFactory.getLogger(SingleConsumerAndTopicConsume.class);

    public static void main(String[] args) {
        consumeTest();
    }

    private static void consumeTest() {
        // 1.配置
        Properties properties = new Properties();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka1:9093,kafka2:9094,kafka3:9095");
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "xzyConsumerGroup");

        // 2.创建消费者
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);

        // 3.订阅 Topic（Note：可以订阅多个主题）
        List<String> topics = new ArrayList<>();
        topics.add("myTopic123");
        consumer.subscribe(topics);

        // 4.消费数据
        LOGGER.info("接收 myTopic123 分区所有 Partition 的消息 ...");
        while (true) {
            Duration timeout = Duration.ofSeconds(3000);
            ConsumerRecords<String, String> consumerRecords = consumer.poll(timeout);
            for (ConsumerRecord<String, String> consumerRecord : consumerRecords) {
                LOGGER.info("消费数据：{}", consumerRecord);
            }
        }
    }
}
```

### 测试

```
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 689, CreateTime = 1667134331711, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 690, CreateTime = 1667134331733, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 691, CreateTime = 1667134331749, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 692, CreateTime = 1667134332018, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 4, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 693, CreateTime = 1667134332037, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 4, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 694, CreateTime = 1667134332068, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 6, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 695, CreateTime = 1667134332089, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 6, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 696, CreateTime = 1667134332295, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 697, CreateTime = 1667134332363, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 698, CreateTime = 1667134332410, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 699, CreateTime = 1667134332682, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 700, CreateTime = 1667134332737, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 138, CreateTime = 1667134331770, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 139, CreateTime = 1667134331795, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 140, CreateTime = 1667134331817, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 141, CreateTime = 1667134331839, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 0, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 142, CreateTime = 1667134331862, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 0, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 143, CreateTime = 1667134331922, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 2, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 144, CreateTime = 1667134331945, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 2, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 145, CreateTime = 1667134331969, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 3, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 146, CreateTime = 1667134331993, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 3, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 147, CreateTime = 1667134332147, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 9, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 148, CreateTime = 1667134332172, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 9, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 149, CreateTime = 1667134332197, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 150, CreateTime = 1667134332231, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 151, CreateTime = 1667134332269, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 152, CreateTime = 1667134332326, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 153, CreateTime = 1667134332384, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 154, CreateTime = 1667134332444, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 155, CreateTime = 1667134332483, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 156, CreateTime = 1667134332524, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 157, CreateTime = 1667134332566, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 158, CreateTime = 1667134332606, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 159, CreateTime = 1667134332652, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 160, CreateTime = 1667134332705, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 161, CreateTime = 1667134332760, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 162, CreateTime = 1667134332782, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 207, CreateTime = 1667134331586, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 208, CreateTime = 1667134331646, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 209, CreateTime = 1667134331679, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 210, CreateTime = 1667134331885, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 1, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 211, CreateTime = 1667134331916, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 1, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 212, CreateTime = 1667134332055, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 5, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 213, CreateTime = 1667134332062, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 5, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 214, CreateTime = 1667134332110, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 7, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 215, CreateTime = 1667134332118, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 7, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 216, CreateTime = 1667134332127, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 8, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 217, CreateTime = 1667134332136, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 8, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 218, CreateTime = 1667134332222, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 219, CreateTime = 1667134332258, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 220, CreateTime = 1667134332315, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 221, CreateTime = 1667134332353, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 222, CreateTime = 1667134332432, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 223, CreateTime = 1667134332471, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 224, CreateTime = 1667134332511, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 225, CreateTime = 1667134332553, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 226, CreateTime = 1667134332593, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 227, CreateTime = 1667134332638, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndTopicConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 228, CreateTime = 1667134332767, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)

```

-   myTopic123 所有分区的数据都被接收



## 单个消费者-订阅分区

### 代码

```java
package com.xzy.consumer.a;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;

/**
 * 说明：独立消费者-订阅分区
 *
 * @author xzy
 * @date 2022/10/30  19:22
 * @see com.xzy.producer.b.SystemPartitionerProducer
 */
public class SingleConsumerAndPartitionConsume {
    private final static Logger LOGGER = LoggerFactory.getLogger(SingleConsumerAndPartitionConsume.class);

    public static void main(String[] args) {
        consumeTest();
    }

    private static void consumeTest() {
        // 1.配置
        Properties properties = new Properties();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka1:9093,kafka2:9094,kafka3:9095");
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "xzyConsumerGroup");

        // 2.创建消费者
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);

        // 3.订阅 Partition
        List<TopicPartition> topicPartitions = new ArrayList<>();
        topicPartitions.add(new TopicPartition("myTopic123", 0));
        consumer.assign(topicPartitions);

        // 4.消费数据
        LOGGER.info("接收 myTopic123 分区第 0 Partition 的消息 ...");
        while (true) {
            Duration timeout = Duration.ofSeconds(3000);
            ConsumerRecords<String, String> consumerRecords = consumer.poll(timeout);
            for (ConsumerRecord<String, String> consumerRecord : consumerRecords) {
                LOGGER.info("消费数据：{}", consumerRecord);
            }
        }
    }
}
```

### 测试

```
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 229, CreateTime = 1667134419122, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 230, CreateTime = 1667134419173, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 231, CreateTime = 1667134419196, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 232, CreateTime = 1667134419356, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 1, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 233, CreateTime = 1667134419378, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 1, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 234, CreateTime = 1667134419515, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 5, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 235, CreateTime = 1667134419544, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 5, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 236, CreateTime = 1667134419632, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 7, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 237, CreateTime = 1667134419661, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 7, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 238, CreateTime = 1667134419692, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 8, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 239, CreateTime = 1667134419721, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 8, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 240, CreateTime = 1667134419776, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 241, CreateTime = 1667134419817, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 242, CreateTime = 1667134419893, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 243, CreateTime = 1667134419968, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 244, CreateTime = 1667134420031, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 245, CreateTime = 1667134420090, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 246, CreateTime = 1667134420139, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 247, CreateTime = 1667134420207, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 248, CreateTime = 1667134420277, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 249, CreateTime = 1667134420333, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 250, CreateTime = 1667134420395, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 251, CreateTime = 1667134420467, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[main] INFO com.xzy.consumer.a.SingleConsumerAndPartitionConsume - 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 252, CreateTime = 1667134420560, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)

```

-   只接收指定分区的数据



## 消费者组-订阅分区

### 代码

```java
package com.xzy.consumer.b;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;

/**
 * 说明：验证同一个消费者组中的消费者只能消费不同的Partition
 *
 * @author xzy
 * @date 2022/10/30  20:03
 * @see com.xzy.producer.b.SystemPartitionerProducer
 */
public class ConsumerGroup {
    private final static Logger LOGGER = LoggerFactory.getLogger(ConsumerGroup.class);

    public void consumeTest(String consumerName) {

        // 1.配置
        Properties properties = new Properties();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka1:9093,kafka2:9094,kafka3:9095");
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "xzyConsumerGroup");

        // 2.创建消费者
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);

        // 3.订阅 Partition
        List<String> topics = new ArrayList<>();
        topics.add("myTopic123");
        consumer.subscribe(topics);

        // 4.消费数据
        LOGGER.info("{} 开始订阅 myTopic123 ...", consumerName);
        while (true) {
            Duration timeout = Duration.ofSeconds(3000);
            ConsumerRecords<String, String> consumerRecords = consumer.poll(timeout);
            for (ConsumerRecord<String, String> consumerRecord : consumerRecords) {
                LOGGER.info("{} 消费数据：{}", consumerName, consumerRecord);
            }
        }
    }
}
```

### 测试

xzyConsumerGroup  中有三个 Consumer，全都订阅 myTopic123：

```java
public static void main(String[] args) {
    new Thread(() -> new ConsumerGroup().consumeTest("BBB")).start();
    new Thread(() -> new ConsumerGroup().consumeTest("AAA")).start();
    new Thread(() -> new ConsumerGroup().consumeTest("CCC")).start();

    while (true) {
    }
}
```

发送消息：

```
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test1. 发送成功. 主题：myTopic123 分区：0
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test1. 发送成功. 主题：myTopic123 分区：0
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test1. 发送成功. 主题：myTopic123 分区：0
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test1. 发送成功. 主题：myTopic123 分区：1
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test1. 发送成功. 主题：myTopic123 分区：1
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test1. 发送成功. 主题：myTopic123 分区：1
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test1. 发送成功. 主题：myTopic123 分区：2
...
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test3. 发送成功. 主题：myTopic123 分区：0
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test3. 发送成功. 主题：myTopic123 分区：2
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test3. 发送成功. 主题：myTopic123 分区：0
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test3. 发送成功. 主题：myTopic123 分区：1
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test3. 发送成功. 主题：myTopic123 分区：0
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test3. 发送成功. 主题：myTopic123 分区：1
[kafka-producer-network-thread | producer-1] INFO com.xzy.producer.b.SystemPartitionerProducer - Test3. 发送成功. 主题：myTopic123 分区：2
```

消费者接收消息的情况：

```
[Thread-1] INFO com.xzy.consumer.b.ConsumerGroup - AAA 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 658, CreateTime = 1667133628958, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[Thread-0] INFO com.xzy.consumer.b.ConsumerGroup - BBB 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 96, CreateTime = 1667133629060, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 162, CreateTime = 1667133628901, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[Thread-0] INFO com.xzy.consumer.b.ConsumerGroup - BBB 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 97, CreateTime = 1667133629092, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[Thread-1] INFO com.xzy.consumer.b.ConsumerGroup - AAA 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 659, CreateTime = 1667133628994, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[Thread-0] INFO com.xzy.consumer.b.ConsumerGroup - BBB 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 98, CreateTime = 1667133629119, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 163, CreateTime = 1667133628935, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[Thread-0] INFO com.xzy.consumer.b.ConsumerGroup - BBB 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 99, CreateTime = 1667133629144, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 0, value = hello world!)
[Thread-1] INFO com.xzy.consumer.b.ConsumerGroup - AAA 消费数据：ConsumerRecord(topic = myTopic123, partition = 1, leaderEpoch = 0, offset = 660, CreateTime = 1667133629026, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[Thread-0] INFO com.xzy.consumer.b.ConsumerGroup - BBB 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 100, CreateTime = 1667133629171, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 0, value = hello world!)
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 164, CreateTime = 1667133628948, serialized key size = 0, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = , value = hello world!)
[Thread-0] INFO com.xzy.consumer.b.ConsumerGroup - BBB 消费数据：ConsumerRecord(topic = myTopic123, partition = 2, leaderEpoch = 0, offset = 101, CreateTime = 1667133629220, serialized key size = 1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = 2, value = hello world!)
...
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 196, CreateTime = 1667133689125, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 197, CreateTime = 1667133689200, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 198, CreateTime = 1667133689240, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 199, CreateTime = 1667133689282, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 200, CreateTime = 1667133689326, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 201, CreateTime = 1667133689383, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 202, CreateTime = 1667133689444, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 203, CreateTime = 1667133689492, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 204, CreateTime = 1667133689538, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 205, CreateTime = 1667133689592, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)
[Thread-2] INFO com.xzy.consumer.b.ConsumerGroup - CCC 消费数据：ConsumerRecord(topic = myTopic123, partition = 0, leaderEpoch = 0, offset = 206, CreateTime = 1667133689638, serialized key size = -1, serialized value size = 12, headers = RecordHeaders(headers = [], isReadOnly = false), key = null, value = hello world!)

```

-   可以看到，组内消费者处理不同分区的数据