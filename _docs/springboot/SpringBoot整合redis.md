---
title: SpringBoot整合redis
category: springboot
category-order: 1
order: 14
---



目前java操作redis的客户端有jedis跟Lettuce。在springboot1.x系列中，其中使用的是jedis,但是到了springboot2.x其中使用的是Lettuce。 因为我们的版本是springboot2.x系列，所以今天使用的是Lettuce。
关于jedis跟lettuce的区别：

Lettuce 和 Jedis 的定位都是Redis的client，所以他们当然可以直接连接redis server。
Jedis在实现上是直接连接的redis server，如果在多线程环境下是非线程安全的，这个时候只有使用连接池，为每个Jedis实例增加物理连接
Lettuce的连接是基于Netty的，连接实例（StatefulRedisConnection）可以在多个线程间并发访问，应为StatefulRedisConnection是线程安全的，所以一个连接实例（StatefulRedisConnection）就可以满足多线程环境下的并发访问，当然这个也是可伸缩的设计，一个连接实例不够的情况也可以按需增加连接实例。



### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.0.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.ywf.guide</groupId>
    <artifactId>redis</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>redis</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.dyuproject.protostuff/protostuff-core -->
        <dependency>
            <groupId>com.dyuproject.protostuff</groupId>
            <artifactId>protostuff-core</artifactId>
            <version>1.0.8</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.dyuproject.protostuff/protostuff-runtime -->
        <dependency>
            <groupId>com.dyuproject.protostuff</groupId>
            <artifactId>protostuff-runtime</artifactId>
            <version>1.0.8</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```



### application.yaml

```yaml
spring:
  redis:
    database: 0
    cluster:
      nodes: 192.168.172.128:6379,192.168.172.128:6380,192.168.172.128:6381,192.168.172.128:6382,192.168.172.129:6379,192.168.172.129:6380,192.168.172.130:6379,192.168.172.130:6380
      max-redirects: 3 # 获取失败 最大重定向次数
      timeout: 6000
    lettuce:
      pool:
        max-active: 1000  #连接池最大连接数（使用负值表示没有限制）
        max-idle: 10 # 连接池中的最大空闲连接
        min-idle: 5 # 连接池中的最小空闲连接
        max-wait: -1 # 连接池最大阻塞等待时间（使用负值表示没有限制）
```



### RedisConfiguration.java

```java
package com.ywf.guide.redis.config;

import com.ywf.guide.redis.util.ProtostuffSerializer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import javax.annotation.Resource;
import java.io.Serializable;

@Configuration
public class RedisConfiguration {
    @Resource
    private LettuceConnectionFactory myLettuceConnectionFactory;

    @Bean
    public RedisTemplate<String, Serializable> redisTemplate() {
        RedisTemplate<String, Serializable> template = new RedisTemplate<>();
        template.setKeySerializer(new StringRedisSerializer());
//        template.setValueSerializer( new JdkSerializationRedisSerializer());
//        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        template.setValueSerializer( new ProtostuffSerializer());
        template.setConnectionFactory(myLettuceConnectionFactory);
        return template;
    }
}

```



### RedisFactoryConfig.java

```java
package com.ywf.guide.redis.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.MapPropertySource;
import org.springframework.data.redis.connection.RedisClusterConfiguration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;

import java.util.HashMap;
import java.util.Map;

@Configuration
public class RedisFactoryConfig {

    @Value("${spring.redis.cluster.nodes}")
    private String nodes;

    @Value("${spring.redis.cluster.timeout}")
    private String timeout;

    @Value("${spring.redis.cluster.max-redirects}")
    private String maxRedirects;

    @Bean
    public RedisConnectionFactory myLettuceConnectionFactory() {
        Map<String, Object> source = new HashMap<String, Object>();
        source.put("spring.redis.cluster.nodes", nodes);
        source.put("spring.redis.cluster.timeout", timeout);
        source.put("spring.redis.cluster.max-redirects", maxRedirects);
        RedisClusterConfiguration redisClusterConfiguration;
        redisClusterConfiguration = new RedisClusterConfiguration(new MapPropertySource("RedisClusterConfiguration", source));
        return new LettuceConnectionFactory(redisClusterConfiguration);
    }
}

```



### User.java

```java
package com.ywf.guide.redis.entity;

import java.io.Serializable;

public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    private String id;
    private String name;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

```



### ProtostuffSerializer.java

```java
package com.ywf.guide.redis.util;

import com.dyuproject.protostuff.LinkedBuffer;
import com.dyuproject.protostuff.ProtostuffIOUtil;
import com.dyuproject.protostuff.Schema;
import com.dyuproject.protostuff.runtime.RuntimeSchema;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.SerializationException;

public class ProtostuffSerializer implements RedisSerializer<Object> {
    private boolean isEmpty(byte[] data) {
        return (data == null || data.length == 0);
    }
    private final Schema<ProtoWrapper> schema;
    private final ProtoWrapper wrapper;
    private final LinkedBuffer buffer;

    public ProtostuffSerializer() {
        this.wrapper = new ProtoWrapper();
        this.schema = RuntimeSchema.getSchema(ProtoWrapper.class);
        this.buffer = LinkedBuffer.allocate(LinkedBuffer.DEFAULT_BUFFER_SIZE);
    }

    @Override
    public byte[] serialize(Object t) throws SerializationException {
        if (t == null) {
            return new byte[0];
        }
        wrapper.data = t;
        try {
            return ProtostuffIOUtil.toByteArray(wrapper, schema, buffer);
        } finally {
            buffer.clear();
        }
    }
    @Override
    public Object deserialize(byte[] bytes) throws SerializationException {
        if (isEmpty(bytes)) {
            return null;
        }
        ProtoWrapper newMessage = schema.newMessage();
        ProtostuffIOUtil.mergeFrom(bytes, newMessage, schema);
        return newMessage.data;
    }

    private static class ProtoWrapper {
        public Object data;
    }
}

```



### RedisApplicationTests.java

```java
package com.ywf.guide.redis;

import com.ywf.guide.redis.entity.User;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.Cursor;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ScanOptions;

import java.util.HashSet;
import java.util.List;
import java.util.Set;

@SpringBootTest(classes = RedisApplication.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class RedisApplicationTests {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    private final static int SIZE = 100000;
    private final static String MATCH_KEY = "redis-cluster-test-proto-stuff:";

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    void contextLoads() {
    }

    /**
     * 模拟生成数据
     */
    @Test
    void testSet() {
        long startTime = System.currentTimeMillis();
        User u = new User();
        for (int i = 0; i < SIZE; i++) {
            u = new User();
            u.setId(String.valueOf(i));
            u.setName("ywf - " + i);
            redisTemplate.opsForValue().set(MATCH_KEY + i, u);
        }
        long endTime = System.currentTimeMillis();
        logger.info("运行了：" + (endTime - startTime) + "ms");
    }

    @Test
    void testScanAndMultiGet() {
        long startTime = System.currentTimeMillis();
        Set<String> keySet = (Set<String>) redisTemplate.execute((RedisCallback<Set<String>>) connection -> {
            Set<String> keysTmp = new HashSet<>();
            Cursor<byte[]> cursor = connection.scan(new ScanOptions.ScanOptionsBuilder().match(MATCH_KEY + "*").count(1000).build());
            while (cursor.hasNext()) {
                keysTmp.add(new String(cursor.next()));
            }
            return keysTmp;
        });

        List<User> userList = redisTemplate.opsForValue().multiGet(keySet);
        logger.info("keySet size = " + userList.size());
        User u = new User();
        long endTime = System.currentTimeMillis();
        logger.info("运行了：" + (endTime - startTime) + "ms");
    }
}

```



### 测试结果

|        | JdkSerializationRedisSerializer | GenericJackson2JsonRedisSerializer | ProtostuffSerializer |
| ------ | ------------------------------- | ---------------------------------- | -------------------- |
| 100    | 1022ms                          | 1049ms                             | 998ms                |
| 1000   | 2057ms                          | 1498ms                             | 1447ms               |
| 10000  | 6010ms                          | 4476ms                             | 4268ms               |
| 100000 | 37690ms                         | 30513ms                            | 30366ms              |



- 时间上比较jackson2json和protostuff速度最快
- 内存和磁盘占用率protostuff最小