---
title: redis+lua生成分布式锁
category: lua
order: 2
---

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.ywf</groupId>
    <artifactId>redis-lua</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>redis-lua</name>
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
            <artifactId>spring-boot-starter-web</artifactId>
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
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
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

### application.yml

```properties
spring:
  application:
    name: redis-cluster
  redis:
    cluster:
      nodes: node1:6379,node1:6380,node2:6379,node2:6380,node3:6379,node3:6380
      max-redirects: 6
redis:
  timeout: 10000 #客户端超时时间单位是毫秒 默认是2000
  maxIdle: 300 #最大空闲数
  maxTotal: 1000 #控制一个pool可分配多少个jedis实例,用来替换上面的redis.maxActive,如果是jedis 2.4以后用该属性
  maxWaitMillis: 1000 #最大建立连接等待时间。如果超过此时间将接到异常。设为-1表示无限制。
  minEvictableIdleTimeMillis: 300000 #连接的最小空闲时间 默认1800000毫秒(30分钟)
  numTestsPerEvictionRun: 1024 #每次释放连接的最大数目,默认3
  timeBetweenEvictionRunsMillis: 30000 #逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1
  testOnBorrow: true #是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个
  testWhileIdle: true #在空闲时检查有效性, 默认false
server:
  port: 8080
```

### RedisClusterConfig

```java
package com.ywf.redislua.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisClusterConfiguration;
import org.springframework.data.redis.connection.RedisNode;
import org.springframework.data.redis.connection.RedisPassword;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.JdkSerializationRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import redis.clients.jedis.JedisPoolConfig;

import java.util.HashSet;
import java.util.Set;

/**
 * @Author:ywf
 */

@Configuration
public class RedisClusterConfig {
    @Value("${spring.redis.cluster.nodes}")
    private String clusterNodes;
    @Value("${spring.redis.cluster.max-redirects}")
    private int maxRedirects;
    @Value("${redis.timeout}")
    private int timeout;
    @Value("${redis.maxIdle}")
    private int maxIdle;
    @Value("${redis.maxTotal}")
    private int maxTotal;
    @Value("${redis.maxWaitMillis}")
    private int maxWaitMillis;
    @Value("${redis.minEvictableIdleTimeMillis}")
    private int minEvictableIdleTimeMillis;
    @Value("${redis.numTestsPerEvictionRun}")
    private int numTestsPerEvictionRun;
    @Value("${redis.timeBetweenEvictionRunsMillis}")
    private int timeBetweenEvictionRunsMillis;
    @Value("${redis.testOnBorrow}")
    private boolean testOnBorrow;
    @Value("${redis.testWhileIdle}")
    private boolean testWhileIdle;
    @Bean
    public JedisPoolConfig getJedisPoolConfig() {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        // 最大空闲数
        jedisPoolConfig.setMaxIdle(maxIdle);
        // 连接池的最大数据库连接数
        jedisPoolConfig.setMaxTotal(maxTotal);
        // 最大建立连接等待时间
        jedisPoolConfig.setMaxWaitMillis(maxWaitMillis);
        // 逐出连接的最小空闲时间 默认1800000毫秒(30分钟)
        jedisPoolConfig.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis);
        // 每次逐出检查时 逐出的最大数目 如果为负数就是 : 1/abs(n), 默认3
        jedisPoolConfig.setNumTestsPerEvictionRun(numTestsPerEvictionRun);
        // 逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1
        jedisPoolConfig.setTimeBetweenEvictionRunsMillis(timeBetweenEvictionRunsMillis);
        // 是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个
        jedisPoolConfig.setTestOnBorrow(testOnBorrow);
        // 在空闲时检查有效性, 默认false
        jedisPoolConfig.setTestWhileIdle(testWhileIdle);
        return jedisPoolConfig;
    }

    /**
     * Redis集群的配置
     * @return RedisClusterConfiguration
     * @throws
     */
    @Bean
    public RedisClusterConfiguration redisClusterConfiguration(){
        RedisClusterConfiguration redisClusterConfiguration = new RedisClusterConfiguration();
        //Set<RedisNode> clusterNodes
        String[] serverArray = clusterNodes.split(",");
        Set<RedisNode> nodes = new HashSet<RedisNode>();
        for(String ipPort:serverArray){
            String[] ipAndPort = ipPort.split(":");
            nodes.add(new RedisNode(ipAndPort[0].trim(),Integer.valueOf(ipAndPort[1])));
        }
        redisClusterConfiguration.setClusterNodes(nodes);
        redisClusterConfiguration.setMaxRedirects(maxRedirects);
//        redisClusterConfiguration.setPassword(RedisPassword.of(password));
        return redisClusterConfiguration;
    }

    /**
     * @param
     * @return
     * @Description:redis连接工厂类
     * @date 2018/10/25 19:45
     */
    @Bean
    public JedisConnectionFactory jedisConnectionFactory() {
        //集群模式
        JedisConnectionFactory  factory = new JedisConnectionFactory(redisClusterConfiguration(),getJedisPoolConfig());
        factory.setDatabase(0);
        factory.setTimeout(timeout);
        factory.setUsePool(true);
        return factory;
    }

    /**
     * 实例化 RedisTemplate 对象
     *
     * @return
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        initDomainRedisTemplate(redisTemplate);
        return redisTemplate;
    }

    /**
     * 设置数据存入 redis 的序列化方式,并开启事务
     * 使用默认的序列化会导致key乱码
     *
     */
    private void initDomainRedisTemplate(RedisTemplate<String, Object> redisTemplate) {
        //如果不配置Serializer，那么存储的时候缺省使用String，如果用User类型存储，那么会提示错误User can't cast to String！
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        //这个地方有一个问题，这种序列化器会将value序列化成对象存储进redis中，如果
        //你想取出value，然后进行自增的话，这种序列化器是不可以的，因为对象不能自增；
        //需要改成StringRedisSerializer序列化器。
        redisTemplate.setValueSerializer(new JdkSerializationRedisSerializer());
        redisTemplate.setEnableTransactionSupport(false);
        redisTemplate.setConnectionFactory(jedisConnectionFactory());
    }
}
```

### RedisLockUtil

```java
package com.ywf.redislua.util;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.ClassPathResource;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import org.springframework.scripting.support.ResourceScriptSource;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import java.util.Collections;
import java.util.concurrent.TimeUnit;

/**
 * @Author:ywf
 */
@Component
public class RedisLockUtil {
    private final static String RESULT_SUCCESS = "1";
    private Logger logger = LoggerFactory.getLogger(RedisLockUtil.class);
    @Autowired
    private RedisTemplate redisTemplate;
    /**
     * lua lock脚本
     */
    private DefaultRedisScript lockScript;
    /**
     * lua unlock脚本
     */
    private DefaultRedisScript unLockScript;
    /**
     * redis String 序列化脚本
     */
    private StringRedisSerializer stringRedisSerializer;

    /**
     * @PostConstruct
     * @PostConstruct 顺序
     * 构造方法 => @Autowired => @PostConstruct => init
     */
    @PostConstruct
    public void init() {
        stringRedisSerializer = new StringRedisSerializer();
        lockScript = new DefaultRedisScript();
        lockScript.setResultType(String.class);
        unLockScript = new DefaultRedisScript();
        unLockScript.setResultType(String.class);
        // 加载脚本
        lockScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("lua/lock.lua")));
        unLockScript.setScriptSource(new ResourceScriptSource(new ClassPathResource("lua/unLock.lua")));
    }

    /**
     * redis加分布式锁
     *
     * @param key        redis锁的key值
     * @param requestId  请求的id,随机生成，不重复，防止解锁别人产生的锁
     * @param expireTime 超时时间
     * @param retryTimes 重试次数
     * @return 是否枷锁成功
     */
    public boolean lock(String key, String requestId, long expireTime, int retryTimes) {
        if (retryTimes < 1) {
            retryTimes = 1;
        }
        int count = 0;
        try {
            while (true) {
                /**
                 * RedisScript<T> redisScript,
                 * RedisSerializer<?> redisSerializer,
                 * RedisSerializer<T> redisSerializer,
                 * List<K> keyList,
                 * Object... obj..
                 */
                String result = (String) redisTemplate.execute(lockScript, stringRedisSerializer, stringRedisSerializer,
                        Collections.singletonList(key), requestId, String.valueOf(expireTime));
                System.out.println(key + " lock result = " + result);

                // 判断是否成功加锁
                if (RESULT_SUCCESS.equals(result)) {
                    logger.info("对 key = [" + key + "], requestId = [" + requestId + "]获取锁成功，第 " + count + "次加锁");
                    return true;
                } else {
                    //加锁失败 判断是否重试
                    if (count++ >= retryTimes) {
                        logger.info("对 key = [" + key + "], requestId = [" + requestId + "]获取锁失败");
                        return false;
                    } else {
                        logger.info("获取锁失败，尝试对 key = [" + key + "], requestId = [" + requestId + "],第 " + count + "次加锁");
                        // 重试
                        // 休眠200ms
                        TimeUnit.MILLISECONDS.sleep(200);
                        continue;
                    }
                }
            }
        } catch (Exception ex) {
            logger.error(ex.getMessage(), ex.getCause());
            ex.printStackTrace();
        }
        return false;
    }

    /**
     * 解锁 放在finally
     *
     * @param key       解锁 redis的key值
     * @param requestId 不重复的值，防止解锁了别人加的锁
     * @return
     */
    public boolean unLock(String key, String requestId) {
        String result = (String) redisTemplate.execute(unLockScript, stringRedisSerializer, stringRedisSerializer,
                Collections.singletonList(key), requestId);
        logger.info(key + " unlock result = " + result);
        return RESULT_SUCCESS.equals(result);
    }
}
```

### TestController

```java
package com.ywf.redislua.controller;

import com.ywf.redislua.util.RedisLockUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Author:ywf
 */

@RestController
public class TestController {

    @Autowired
    private RedisLockUtil redisLockUtil;

    /**
     * 加锁
     * @param key
     * @param requestId
     */
    @RequestMapping("/testRedisLock")
    public String testRedisLock(@RequestParam String key, @RequestParam String requestId) {
        if (redisLockUtil.lock(key, requestId, 60000, 3)) {
            return "lock success";
        }
        return "fail lock";
    }

    /**
     * 解锁
     * @param key
     * @param requestId
     * @return
     */
    @RequestMapping("/testRedisUnLock")
    public String testRedisUnLock(@RequestParam String key, @RequestParam String requestId) {
        if (redisLockUtil.unLock(key, requestId)) {
            return "unLock success";
        }
        return "fail unLock";
    }
}

```

### Lua脚本

#### lock.lua

```lua
-- SETNX 存在则加锁，不存在 不进行业务操作， 设置超时时间 防止死锁
if redis.call('set', KEYS[1], ARGV[1], 'NX', 'PX', ARGV[2]) then
    return '1'
else
    return '0'
end
```

#### unLock.lua

```lua
if redis.call('get', KEYS[1]) == ARGV[1] then
    return tostring(redis.call('del', KEYS[1]))
else
    return '0'
end
```



### 浏览器测试

第一次加锁

[http://localhost:8080/testRedisLock?key=order1&requestId=abc](http://localhost:8080/testRedisLock?key=order1&requestId=abc)

- lock success

再次申请加锁

[http://localhost:8080/testRedisLock?key=order1&requestId=abc](http://localhost:8080/testRedisLock?key=order1&requestId=abc)

- fail lock

控制台

```
2019-12-10 15:59:23.797  INFO 2648 --- [nio-8080-exec-2] com.ywf.redislua.util.RedisLockUtil      : 获取锁失败，尝试对 key = [order1], requestId = [abc],第 1次加锁
order1 lock result = 0
2019-12-10 15:59:24.005  INFO 2648 --- [nio-8080-exec-2] com.ywf.redislua.util.RedisLockUtil      : 获取锁失败，尝试对 key = [order1], requestId = [abc],第 2次加锁
order1 lock result = 0
2019-12-10 15:59:24.211  INFO 2648 --- [nio-8080-exec-2] com.ywf.redislua.util.RedisLockUtil      : 获取锁失败，尝试对 key = [order1], requestId = [abc],第 3次加锁
order1 lock result = 0
2019-12-10 15:59:24.417  INFO 2648 --- [nio-8080-exec-2] com.ywf.redislua.util.RedisLockUtil      : 对 key = [order1], requestId = [abc]获取锁失败
```



解锁请求

[localhost:8080/testRedisUnLock?key=order1&requestId=abc](localhost:8080/testRedisUnLock?key=order1&requestId=abc)

- unLock success



### redis分布式锁的缺点

```
1、集群环境下，对master节点申请了分布式锁，由于redis的主从同步是异步进行的，master在内存中写入了nx之后直接返回，客户端获取锁成功，此时master节点挂了，并且数据还没来得及同步，另一个节点被升级为master，这样其他的线程依然可以获取锁
2、每个方法超时时间的大小要根据具体的方法的执行时间来设置。
```

