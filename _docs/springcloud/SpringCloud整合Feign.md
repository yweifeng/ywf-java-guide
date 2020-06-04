---
title: SpringCloud整合Feign(声明式调用)
category: springcloud
order: 3
---

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>ywf-springcloud</artifactId>
        <groupId>com.ywf</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>service-feign</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
    </dependencies>

</project>
```

**bootstrap.yml**

```properties
spring:
  application:
    name: service-feign
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
        # 命名空间
        namespace: c2efbc9a-03c3-46ab-9487-94b6e9e6fc53
  profiles:
    active: dev
server:
  port: 7001
```

**启动类**

```java
package com.ywf;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

/**
 * @author ywf
 */
@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
public class ServiceFeignApp {
    public static void main(String[] args) {
        SpringApplication.run(ServiceFeignApp.class, args);
    }
}
```

**FeignClient**

```java
package com.ywf.client;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * @author ywf
 */
@FeignClient("nacos-provider")
public interface NacosProvicerClient {

    @RequestMapping(value = "/hi", method = RequestMethod.GET)
    String hi(@RequestParam(value = "name", defaultValue = "ywf", required = false) String name);
}
```

**Controller**

```java
package com.ywf.controller;

import com.ywf.client.NacosProvicerClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
public class FeignController {
    @Autowired
    NacosProvicerClient nacosProvicerClient;

    @GetMapping("/hi-feign")
    public String hiFeign(){
        return nacosProvicerClient.hi("ywf");
    }
}
```

