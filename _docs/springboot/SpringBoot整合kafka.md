---
title: Springboot集成kafka
category: springboot
order: 16
---

### pom.xml引入依赖

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```



### application.yaml配置参数

```yaml
server:
  port: 9001
spring:
  application:
    name: guide-kafka
  kafka:
    bootstrap-servers: 192.168.172.128:9092,192.168.172.129:9092,192.168.172.130:9092
    producer:
      #      如果 acks=0，生产者在成功写入消息之前不会等待任何来自服务器的响应。也就是说，如果当中出现了问题，导致服务器没有收到消息，那么生产者就无从得知，消息也就丢失了。不过，因为生产者不需要等待服务器的响应，所以它可以以网络能够支持的最大速度发送消息，从而达到很高的吞吐量。
      #      如果 acks=1，只要集群的首领节点收到消息，生产者就会收到一个来自服务器的成功响应。如果消息无法到达首领节点（比如首领节点崩溃，新的首领还没有被选举出来），生产者会收到一个错误响应，为了避免数据丢失，生产者会重发消息。不过，如果一个没有收到消息的节点成为新首领，消息还是会丢失。这个时候的吞吐量取决于使用的是同步发送还是异步发送。如果让发送客户端等待服务器的响应（通过调用 Future 对象的 get() 方法），显然会增加延迟（在网络上传输一个来回的延迟）。如果客户端使用回调，延迟问题就可以得到缓解，不过吞吐量还是会受发送中消息数量的限制（比如，生产者在收到服务器响应之前可以发送多少个消息）。
      #      如果 acks=all，只有当所有参与复制的节点全部收到消息时，生产者才会收到一个来自服务器的成功响应。这种模式是最安全的，它可以保证不止一个服务器收到消息，就算有服务器发生崩溃，整个集群仍然可以运行。不过，它的延迟比 acks=1 时更高，因为我们要等待不只一个服务器节点接收消息。
      acks: all
      retries: 3
      #      当有多个消息需要被发送到同一个分区时，生产者会把它们放在同一个批次里。
      #      该参数指定了一个批次可以使用的内存大小，按照字节数计算（而不是消息个数）。
      #      当批次被填满，批次里的所有消息会被发送出去。不过生产者并不一定都会等到批次被填满才发送，
      #      半满的批次，甚至只包含一个消息的批次也有可能被发送。所以就算把批次大小设置得很大，也不会造成延迟，
      #      只是会占用更多的内存而已。但如果设置得太小，因为生产者需要更频繁地发送消息，会增加一些额外的开销。
      batch-size: 16384
      #      该参数用来设置生产者内存缓冲区的大小，生产者用它缓冲要发送到服务器的消息。
      #      如果应用程序发送消息的速度超过发送到服务器的速度，会导致生产者空间不足。
      #      这个时候， send() 方法调用要么被阻塞，要么抛出异常，取决于如何设置 block.on.buffer.full 参数
      #      （在 0.9.0.0 版本里被替换成了 max.block.ms，表示在抛出异常之前可以阻塞一段时间）。
      buffer-memory: 33554432
      #      该参数指定了生产者在发送批次之前等待更多消息加入批次的时间。KafkaProducer 会在批次填满或 linger.ms 达到上限时把批次发送出去。
      #      默认情况下，只要有可用的线程，就算批次里只有一个消息，生产者也会把消息发送出去。
      linger-size: 0
    consumer:
      group-id: ywf-kafka-guide
      auto-offset-reset: latest
      enable-auto-commit:  false
      auto-commit-interval: 1000
      max-poll-records: 5
```



### KafkaConfiguration.java

```java
package com.ywf.guide.kafka.config;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.config.KafkaListenerContainerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.kafka.listener.ContainerProperties;

import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableKafka
public class KafkaConfiguration {

    @Value("${spring.kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Value("${spring.kafka.consumer.enable-auto-commit}")
    private Boolean autoCommit;

    @Value("${spring.kafka.consumer.auto-commit-interval}")
    private Integer autoCommitInterval;

    @Value("${spring.kafka.consumer.group-id}")
    private String groupId;

    @Value("${spring.kafka.consumer.max-poll-records}")
    private Integer maxPollRecords;

    @Value("${spring.kafka.consumer.auto-offset-reset}")
    private String autoOffsetReset;

    @Value("${spring.kafka.producer.retries}")
    private Integer retries;

    @Value("${spring.kafka.producer.batch-size}")
    private Integer batchSize;

    @Value("${spring.kafka.producer.linger-size}")
    private Integer lingerSize;

    @Value("${spring.kafka.producer.buffer-memory}")
    private Integer bufferMemory;

    @Value("${spring.kafka.producer.acks}")
    private String acks;

    /**
     * 生产者配置信息
     */
    @Bean
    public Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.ACKS_CONFIG, acks);
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ProducerConfig.RETRIES_CONFIG, retries);
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, batchSize);
        props.put(ProducerConfig.LINGER_MS_CONFIG, lingerSize);
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, bufferMemory);
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return props;
    }

    /**
     * 生产者工厂
     */
    @Bean
    public ProducerFactory<String, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }

    /**
     * 生产者模板
     */
    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }

    /**
     * 消费者配置信息
     */
    @Bean
    public Map<String, Object> consumerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.GROUP_ID_CONFIG, groupId);
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, autoOffsetReset);
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, maxPollRecords);
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, autoCommitInterval);
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, autoCommit);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return props;
    }

    /**
     * 消费者批量工程
     */
    @Bean
    public KafkaListenerContainerFactory<?> batchFactory() {
        ConcurrentKafkaListenerContainerFactory<Integer, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(new DefaultKafkaConsumerFactory<>(consumerConfigs()));
        //设置为批量消费，每个批次数量在Kafka配置参数中设置ConsumerConfig.MAX_POLL_RECORDS_CONFIG
        factory.setBatchListener(true);
        // 配置多线程
        factory.setConcurrency(5);

        // 关闭自动提交后设置
        // AckMode 如下:
        //    RECORD :当listener一读到消息，就提交offset
        //    BATCH : poll() 函数读取到的所有消息,就提交offset
        //    TIME : 当超过设置的ackTime ，即提交Offset
        //    COUNT ：当超过设置的COUNT，即提交Offset
        //    COUNT_TIME ：TIME和COUNT两个条件都满足，提交offset
        //    MANUAL ： Acknowledgment.acknowledge()即提交Offset，和Batch类似
        //    MANUAL_IMMEDIATE： Acknowledgment.acknowledge()被调用即提交Offset
        if (!autoCommit) {
            factory.getContainerProperties().setAckMode(ContainerProperties.AckMode.MANUAL_IMMEDIATE);
        }
        return factory;
    }
}

```



### CommonKafkaProducer.java

```java
package com.ywf.guide.kafka.producer;

public interface CommonKafkaProducer {

    /**
     * 发送同步消息
     *
     * @param topic
     * @param msg
     */
    void sendSyncMsg(String topic, String msg);

    /**
     * 发送异步消息
     *
     * @param topic
     * @param msg
     */
    void sendAsyncMsg(String topic, String msg);
}

```



### CommonProducerImpl.java

```java
package com.ywf.guide.kafka.producer.impl;

import com.ywf.guide.kafka.producer.CommonKafkaProducer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.stereotype.Component;
import org.springframework.util.concurrent.ListenableFuture;
import org.springframework.util.concurrent.ListenableFutureCallback;

import java.util.concurrent.ExecutionException;

@Component
public class CommonProducerImpl implements CommonKafkaProducer {

    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @Override
    public void sendSyncMsg(String topic, String msg) {
        logger.info("开始发送同步消息");
        try {
            kafkaTemplate.send(topic, msg).get();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        logger.info("发送同步消息成功");
    }

    @Override
    public void sendAsyncMsg(String topic, String msg) {
        logger.info("发送信息");
        ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send(topic, msg);
        future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
            @Override
            public void onSuccess(SendResult<String, String> result) {
                logger.info("消费成功" + System.currentTimeMillis());
            }

            @Override
            public void onFailure(Throwable ex) {
                logger.info("消费失败");
                ex.getStackTrace();
            }
        });
    }
}

```



### CommonKafkaConsumer.java

```java
package com.ywf.guide.kafka.consumer;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.support.Acknowledgment;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public class CommonKafkaConsumer {

    private Logger logger = LoggerFactory.getLogger(this.getClass());
    private final static String TOPIC_NAME = "topic-ywf-test-10";

    @KafkaListener(topics = TOPIC_NAME, containerFactory = "batchFactory")
    public void batchConsumer(List<ConsumerRecord<String, String>> records, Acknowledgment acknowledgment) {
        logger.info(Thread.currentThread().getName() + "  {}", records.size());
        acknowledgment.acknowledge();
    }
}

```



### 测试类 KafkaApplicationTests.java

```java
package com.ywf.guide.kafka;

import com.ywf.guide.kafka.producer.CommonKafkaProducer;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest(classes = KafkaApplication.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class KafkaApplicationTests {
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    private final static int MSG_SIZE = 100000;

    private final static String TOPIC_NAME = "topic-ywf-test-10";

    @Test
    void contextLoads() {
    }

    @Autowired
    private CommonKafkaProducer commonKafkaProducer;


    /**
     * 测试消息同步发送
     */
    @Test
    public void testSendSyncMsg() {
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < MSG_SIZE; i++) {
            String msg = "hello " + i;
            commonKafkaProducer.sendSyncMsg(TOPIC_NAME, msg);
        }
        long endTime = System.currentTimeMillis();
        logger.info("run " + (endTime - startTime) + " ms");
    }

    /**
     * 测试消息异步发送
     */
    @Test
    public void testSendAsyncMsg() throws InterruptedException {
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < MSG_SIZE; i++) {
            String msg = "hello " + i;
            commonKafkaProducer.sendAsyncMsg(TOPIC_NAME, msg);
            Thread.sleep(1);
        }
        long endTime = System.currentTimeMillis();
        logger.info("run " + (endTime - startTime) + " ms");
    }

}

```



## 同步异步ACK生产数据测试结果

```shell
# kafka生产数据同步异步ACK性能测试
	一万条数据测试
		同步生产:
			ACK 
				all  9590 ms
				1   8728 ms
				0   4675 ms
				
		异步生产：
			ACK 
				all  732 ms
				1   641 ms
				0   688 ms
				
		
	十万条数据测试
		同步生产:
			ACK 
				all  71409 ms
				1   88170 ms
				0   22646 ms
				
		异步生产：
			ACK 
				all  2282 ms
				1   2523 ms
				0   3155 ms

```

