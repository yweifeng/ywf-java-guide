---
title: SpringCloud整合Hystrix
category: springcloud
order: 4
---

## Ribbon整合Hystrix

**pom.xml**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

**启动类@EnableHystrix**

```java
package com.ywf;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.hystrix.EnableHystrix;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableDiscoveryClient
@EnableHystrix
public class ServiceRibbonApp {
    public static void main(String[] args) {
        SpringApplication.run(ServiceRibbonApp.class, args);
    }

    @LoadBalanced
    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

**方法添加 @HystrixCommand(fallbackMethod = "hiError")**

```java
package com.ywf.controller;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

/**
 * @author ywf
 */
@RestController
public class RibbonController {

    @Autowired
    RestTemplate restTemplate;

    @GetMapping("/hi-ribbon")
    @HystrixCommand(fallbackMethod = "hiError")
    public String hiResttemplate(String name){
        return restTemplate.getForObject("http://nacos-provider/hi?name=" + name,String.class);
    }

    public String hiError(String name) {
        return "hi,"+name+",sorry,error!";
    }
}
```



## Feign整合Hystrix

**bootstrap.yml**

```properties
feign:
  hystrix:
    enabled: true
```

**新建NacosProviderClientHystrix**

```java
package com.ywf.hystrix;

import com.ywf.client.NacosProvicerClient;
import org.springframework.stereotype.Component;

/**
 * @Author:ywf
 */
@Component
public class NacosProviderClientHystrix implements NacosProvicerClient {
    @Override
    public String hi(String name) {
        return "error " +name;
    }
}
```

**@FeignClient添加注解**

```java
package com.ywf.client;

import com.ywf.hystrix.NacosProviderClientHystrix;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * @author ywf
 */
@FeignClient(value = "nacos-provider", fallback = NacosProviderClientHystrix.class)
public interface NacosProvicerClient {

    @RequestMapping(value = "/hi", method = RequestMethod.GET)
    String hi(@RequestParam(value = "name", defaultValue = "ywf", required = false) String name);
}
```

