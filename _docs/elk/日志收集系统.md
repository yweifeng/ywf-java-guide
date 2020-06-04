# springboot aop + elk + kafka 搭建 日志收集系统

## 准备环境

**centos 7 、 elk版本 6.7.1、 jdk 1.8**

| 服务器          | kafka 端口 | zookeeper 端口 | elasticsearch | kibana | logstash |
| --------------- | ---------- | -------------- | ------------- | ------ | -------- |
| 192.168.111.128 | 9092       | 2181           | 9200          | 5601   | 9600     |
| 192.168.111.129 | 9092       | 2181           | 9200          |        |          |
| 192.168.111.130 | 9092       | 2181           | 9200          |        |          |

### 启动zookeeper

```shell
# 3台服务器都启动
cd /opt/zookeeper
bin/zkServer.sh start
```

### 启动kafka

```shell
# 3台服务器都启动
cd /opt/kafak
bin/kafka-server-start.sh -daemon config/server.properties 
```

### 启动elasticsearch

```shell
su es
cd /opt/elk/es
bin/elasticsearch -d
```

### 启动kibana

```shell
# 启动192.168.111.128
cd /opt/elk/kibana
nohup bin/kibana &
```

### 配置logstash

**新增配置信息**

```shell
vim config/ywf-system-log.cfg


input {
    kafka {
        bootstrap_servers => "192.168.111.128:9092,192.168.111.129:9092,192.168.111.130:9092"
        group_id => "ywf-system-log-group"
        auto_offset_reset => "latest"
        consumer_threads => 5
        decorate_events => true
        topics => ["ywf-system-log"]
    }
}

output {
    elasticsearch {
        hosts => ["192.168.111.128:9200", "192.168.111.129:9200", "192.168.111.130:9200"]
        index => "ywf-system-log"
    }
}
```

### 启动logstash

```shell
bin/logstash -f config/ywf-system-log.cfg
```

### kafka创建topic

```shell
kafka-topics.sh --create --zookeeper 192.168.111.128:2181,192.168.111.129:2181,192.168.111.130:2181 --replication-factor 3 --partitions 3 --topic ywf-system-log
```



## springboot 项目搭建

**本次系统使用添加多个切点的方式，也可以改为切面的方式。**

### pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.ywf</groupId>
    <artifactId>ywf-elk-log</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>ywf-elk-log</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.1.26</version>
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
server:
  port: 8008

spring:
  application:
    name: ywf-elk-log
  kafka:
    producer:
      acks: all #acks:消息的确认机制，默认值是0， acks=0：如果设置为0，生产者不会等待kafka的响应。 acks=1：这个配置意味着kafka会把这条消息写到本地日志文件中，但是不会等待集群中其他机器的成功响应。 acks=all：这个配置意味着leader会等待所有的follower同步完成。这个确保消息不会丢失，除非kafka集群中所有机器挂掉。这是最强的可用性保证。
      retries: 3 #发送失败重试次数，配置为大于0的值的话，客户端会在消息发送失败时重新发送。
      batch-size: 16384 #当多条消息需要发送到同一个分区时，生产者会尝试合并网络请求。这会提高client和生产者的效率。
      buffer-memory: 33554432 #即32MB的批处理缓冲区
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      bootstrap-servers: 192.168.111.128:9092,192.168.111.129:9092,192.168.111.130:9092 #如果kafka启动错误，打开debug级别日志，出现Can't resolve address: flink:9092 的错误
logging:
  file: ywf-elk-log.log
  level:
    #    root: debug #开启dubug级别
    com.kafka: debug
```

### 启动类

```java
package com.ywf.ywfelklog;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class YwfElkLogApplication {

    public static void main(String[] args) {
        SpringApplication.run(YwfElkLogApplication.class, args);
    }

}
```

### 日志注解

```java
package com.ywf.ywfelklog.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * @Author:ywf
 * 日志注解
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface SystemLog {
}
```

### 切面类

```java
package com.ywf.ywfelklog.aop;

import com.alibaba.fastjson.JSONObject;
import com.ywf.ywfelklog.util.WebUtil;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.util.Enumeration;
import java.util.concurrent.ExecutionException;

/**
 * @Author:ywf
 */

@Component
@Aspect
public class LogAspect {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    /**
     * 方法执行之前保存操作日志
     * 本次使用多个切点自定义日志注解，不用切面
     *
     * @param joinPoint
     * @return
     * @throws Throwable
     */
    @Before("@annotation(com.ywf.ywfelklog.annotation.SystemLog)")
    public void Log(JoinPoint joinPoint) {
        System.out.println("do elk log ");

        // 获取请求参数
        JSONObject message = new JSONObject();
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();

        //获取方法参数
        Enumeration<String> eParams = request.getParameterNames();
        JSONObject params = new JSONObject();
        while (eParams.hasMoreElements()) {
            String key = eParams.nextElement();
            String value = request.getParameter(key);
            params.put(key, value);
        }
        String ip = WebUtil.getIpAddr(request);
        String requestURL = request.getRequestURL().toString();
        message.put("requestURL", requestURL);
        message.put("class", joinPoint.getTarget().getClass().getName());
        message.put("request_method", joinPoint.getSignature().getName());
        message.put("ip", ip);
        message.put("systemName", "ywf-elk-log");
        message.put("params", params.toJSONString());
        String msg = message.toJSONString();
        try {
            System.out.println("======= system log  ======");
            System.out.println("msg:" + msg);
            SendResult<String, String> result = kafkaTemplate.send("ywf-system-log", msg).get();
            System.out.println("发送日志到kafka, result= " + result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

### 控制层

```java
package com.ywf.ywfelklog.controller;

import com.ywf.ywfelklog.annotation.SystemLog;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

/**
 * @Author:ywf
 */
@RestController
public class LoginController {

    @RequestMapping("/login")
    @SystemLog
    public String login(@RequestParam String username, @RequestParam String pwd) {
        return "success";
    }

    @RequestMapping("/logout")
    @SystemLog
    public String logout(@RequestParam String username) {
        return "success";
    }
}
```

### 工具类

```java
package com.ywf.ywfelklog.util;

import javax.servlet.http.HttpServletRequest;

/**
 * @Author:ywf
 */
public class WebUtil {

    public static String getIpAddr(HttpServletRequest request) {
        String ipAddress = null;
        ipAddress = request.getHeader("x-forwarded-for");
        if (ipAddress == null || ipAddress.length() == 0
                || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("Proxy-Client-IP");
        }
        if (ipAddress == null || ipAddress.length() == 0
                || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ipAddress == null || ipAddress.length() == 0
                || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getRemoteAddr();
        }

        // 对于通过多个代理的情况，第一个IP为客户端真实IP,多个IP按照','分割
        if (ipAddress != null && ipAddress.length() > 15) { // "***.***.***.***".length()
            // = 15
            if (ipAddress.indexOf(",") > 0) {
                ipAddress = ipAddress.substring(0, ipAddress.indexOf(","));
            }
        }
        //或者这样也行,对于通过多个代理的情况，第一个IP为客户端真实IP,多个IP按照','分割
        //return ipAddress!=null&&!"".equals(ipAddress)?ipAddress.split(",")[0]:null;
        return ipAddress;
    }
}
```

### 启动应用，接口访问测试

- [http://localhost:8008/login?username=杨伟锋&pwd=123](http://localhost:8008/login?username=杨伟锋&pwd=123)

- [http://localhost:8008/login?username=王雅萍&pwd=520](http://localhost:8008/login?username=王雅萍&pwd=520)



## kibana验证

```bash
get /ywf-system-log/_search
```

**返回值**

```json
{
  "took" : 16,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "ywf-system-log",
        "_type" : "doc",
        "_id" : "0hrX0m4BF3dJvsS7qyjl",
        "_score" : 1.0,
        "_source" : {
          "@timestamp" : "2019-12-04T21:36:20.535Z",
          "message" : """{"class":"com.ywf.ywfelklog.controller.LoginController","ip":"0:0:0:0:0:0:0:1","params":"{\"pwd\":\"520\",\"username\":\"王雅萍\"}","requestURL":"http://localhost:8008/login","request_method":"login","systemName":"ywf-elk-log"}""",
          "@version" : "1"
        }
      },
      {
        "_index" : "ywf-system-log",
        "_type" : "doc",
        "_id" : "-fjW0m4Bu0jefXBr9-hS",
        "_score" : 1.0,
        "_source" : {
          "@timestamp" : "2019-12-04T21:35:32.861Z",
          "message" : """{"class":"com.ywf.ywfelklog.controller.LoginController","ip":"0:0:0:0:0:0:0:1","params":"{\"pwd\":\"123\",\"username\":\"杨伟锋\"}","requestURL":"http://localhost:8008/login","request_method":"login","systemName":"ywf-elk-log"}""",
          "@version" : "1"
        }
      }
    ]
  }
}
```









