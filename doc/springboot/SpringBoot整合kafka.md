
## Springboot集成kafka

### 简单实现

#### 配置host

```
windows: C:\Windows\System32\drivers\etc\hosts
```

#### 建立项目，修改pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ywf</groupId>
    <artifactId>kafka</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.2.RELEASE</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>

        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.8.2</version>
        </dependency>

    </dependencies>
</project>
```

#### 建立传输实体类

```java
package com.ywf;

import lombok.Data;

import java.util.Date;

@Data
public class Message {
    private Long id;    //id

    private String msg; //消息

    private Date sendTime;  //时间戳

}

```

#### 创建KafkaSender

```java
package com.ywf;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Component;

import java.util.Date;
import java.util.UUID;

@Component
@Slf4j
public class KafkaSender {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    private Gson gson = new GsonBuilder().create();

    //发送消息方法
    public void send() {
        Message message = new Message();
        Long id = System.currentTimeMillis();
        message.setId(id);
        message.setMsg(UUID.randomUUID().toString());
        message.setSendTime(new Date());
        log.info("+++++++++++++++++++++  message = {}", gson.toJson(message));
        kafkaTemplate.send("topic-ywf", String.valueOf(id), gson.toJson(message));
    }
}
```

#### 创建KafkaReceiver

```java
package com.ywf;

import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

import java.util.Optional;

@Component
@Slf4j
public class KafkaReceiver {
    @KafkaListener(topics = {"topic-ywf"})
    public void listen(ConsumerRecord<?, ?> record) {

        Optional<?> kafkaMessage = Optional.ofNullable(record.value());

        if (kafkaMessage.isPresent()) {

            Object message = kafkaMessage.get();

            log.info("----------------- record =" + record);
            log.info("------------------ message =" + message);
        }

    }
}
```

#### 创建KafkaApplication，并启动

```java
package com.ywf;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class KafkaApplication {
    public static void main(String[] args) {

        ConfigurableApplicationContext context = SpringApplication.run(KafkaApplication.class, args);

        KafkaSender sender = context.getBean(KafkaSender.class);

        for (int i = 0; i < 3; i++) {
            //调用消息发送类中的消息发送方法
            sender.send();

            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```



### 同步、异步生产消息

#### 建立项目，修改pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ywf</groupId>
    <artifactId>async-kafka</artifactId>
    <version>1.0-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.2.RELEASE</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.1.9.RELEASE</version>
            <scope>compile</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

#### 建立application.yml

```properties
server:
  port: 8007

spring:
  application:
    name: kafkaDemo
  kafka:
    producer:
      acks: all #acks:消息的确认机制，默认值是0， acks=0：如果设置为0，生产者不会等待kafka的响应。 acks=1：这个配置意味着kafka会把这条消息写到本地日志文件中，但是不会等待集群中其他机器的成功响应。 acks=all：这个配置意味着leader会等待所有的follower同步完成。这个确保消息不会丢失，除非kafka集群中所有机器挂掉。这是最强的可用性保证。
      retries: 0 #发送失败重试次数，配置为大于0的值的话，客户端会在消息发送失败时重新发送。
      batch-size: 16384 #当多条消息需要发送到同一个分区时，生产者会尝试合并网络请求。这会提高client和生产者的效率。
      buffer-memory: 33554432 #即32MB的批处理缓冲区
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      bootstrap-servers: 192.168.111.128:9092,192.168.111.129:9092,192.168.111.130:9092 #如果kafka启动错误，打开debug级别日志，出现Can't resolve address: flink:9092 的错误，需要在 windows下修改IP映射即可， C:\Windows\System32\drivers\etc\hosts, 192.168.234.128 flink。
    consumer:
      group-id: test
      auto-offset-reset: latest #（1）earliest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费；（2）latest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据 ；（3）none：topic各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常
      enable-auto-commit:  true  #如果为true，消费者的偏移量将在后台定期提交。
      auto-commit-interval: 1000 #消费者偏移自动提交给Kafka的频率 （以毫秒为单位），默认值为5000
      max-poll-records: 5 #一次拉起的条数
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      bootstrap-servers: 192.168.111.128:9092,192.168.111.129:9092,192.168.111.130:9092
logging:
  file: kafkaDemo.log
  level:
    #    root: debug #开启dubug级别
    com.kafka: debug
```

#### 创建启动类

```java
package com.ywf;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class KafkaApplication {
    public static void main(String[] args) {
        SpringApplication.run(KafkaApplication.class, args);
    }
}
```

#### 创建生产接口

```java
package com.ywf;

import java.util.concurrent.ExecutionException;

public interface HelloProducerService {
    public void sendSyncHello(String helloQueue, String message) throws InterruptedException, ExecutionException;

    public void sendAsyncHello(String helloQueue, String message);
}
```

#### 创建生产实现类

```java
package com.ywf;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.stereotype.Service;
import org.springframework.util.concurrent.ListenableFuture;
import org.springframework.util.concurrent.ListenableFutureCallback;

import java.util.concurrent.ExecutionException;


@Service
public class HelloProducerServiceImpl implements HelloProducerService {

    private Logger logger = LoggerFactory.getLogger(HelloProducerServiceImpl.class);

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @Override
    public void sendSyncHello(String helloQueue, String message) throws InterruptedException, ExecutionException {
        logger.debug("发送信息");
        try {
            kafkaTemplate.send(helloQueue, message).get();
            Thread.sleep(1000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        logger.debug("消费成功" + System.currentTimeMillis());
    }

    @Override
    public void sendAsyncHello(String helloQueue, String message) {
        logger.debug("发送信息");
        ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send(helloQueue, message);
        future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
            @Override
            public void onSuccess(SendResult<String, String> result) {
                try {
                    Thread.sleep(1000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                logger.debug("消费成功" + System.currentTimeMillis());
            }

            @Override
            public void onFailure(Throwable ex) {
                logger.debug("消费失败");
                ex.getStackTrace();
            }
        });
    }
}
```

#### 创建消费类

```java
package com.ywf;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

@Component
public class HelloConsumerService {
    private Logger logger = LoggerFactory.getLogger(HelloConsumerService.class);

    @KafkaListener(topics = {"app-sync-log","app-async-log"})
    public void receive(String message){
        logger.info("------hello：消费者处理消息------"+message);
        System.out.println("消费完成"+System.currentTimeMillis()+"ms");
        logger.debug(message);
    }
}
```

#### 创建测试类

```java
import com.ywf.HelloProducerService;
import com.ywf.KafkaApplication;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import java.util.UUID;
import java.util.concurrent.ExecutionException;

/**
 * @author Administrator
 *
 */
@RunWith(SpringRunner.class)
@SpringBootTest(classes = KafkaApplication.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@EnableTransactionManagement //如果mybatis中service实现类中加入事务注解，需要此处添加该注解
@EnableAutoConfiguration
public class KafkaCase {

    @Autowired
    private HelloProducerService helloProducerService;

    @Test
    public void sendSyncTest() throws InterruptedException, ExecutionException {
        for (int i = 0; i < 1; i++) {
            String message = UUID.randomUUID().toString();
            System.out.println("发送消息:"+i);
            helloProducerService.sendSyncHello("app-sync-log", message);
            System.out.println("发送完成"+System.currentTimeMillis()+"ms");
        }
    }

    @Test
    public void sendAsyncTest() {
        for (int i = 0; i < 1; i++) {
            String message = UUID.randomUUID().toString();
            System.out.println("发送消息:"+i);
            helloProducerService.sendAsyncHello("app-async-log", message+0000+i);
            System.out.println("发送完成"+System.currentTimeMillis()+"ms");
        }
    }
}
```

