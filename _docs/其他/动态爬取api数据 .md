---
title: 定时任务+多线程爬取api
category: 其他
order: 3
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
        <version>2.3.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.ws</groupId>
    <artifactId>crawling</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>crawling</name>
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
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.6</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.56</version>
        </dependency>
        <dependency>
            <groupId>commons-lang</groupId>
            <artifactId>commons-lang</artifactId>
            <version>2.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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
server:
  port: 10001
craws:
  apis:
    - name: "福州24小时历史天气"
      url: "http://service.envicloud.cn:8082/v2/weatherhistory/EXDMMTU5MJI5NDKXNJU2NG==/101230101"
      handle-class: "com.crawling.handle.api.WeatherHistoryHandleImpl"
      type: "GET"
      cron: "0/5 * * * * *"

    - name: "福州天气预报"
      url: "http://service.envicloud.cn:8082/v2/weatherforecast"
      handle-class: "com.crawling.handle.api.WeatherHistoryHandleImpl"
      type: "POST"
      header:
        Content-type: "application/json; charset=utf-8"
        Accept: "application/json"
      data:
        accesskey: "EXDMMTU5MJI5NDKXNJU2NG=="
        citycodes:
          - "101230101"
      cron: "0/5 * * * * *"
```



### CrawConfiguration.java

```java
package com.crawling.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;

@Component
@ConfigurationProperties(prefix = "craws")
public class CrawConfiguration {

    private List<Map<String, Object>> apis = new ArrayList<>();

    public List<Map<String, Object>> getApis() {
        return apis;
    }

    public void setApis(List<Map<String, Object>> apis) {
        this.apis = apis;
    }
}

```



### APICrawHandle.java

```java
package com.crawling.handle;

public interface APICrawHandle {

    /**
     * 数据处理
     * @param result
     */
    void handle(String result);
}

```



### WeatherHistoryHandleImpl.java

```java
package com.crawling.handle.api;

import com.crawling.handle.APICrawHandle;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * 福州天气预报处理类
 */
public class WeatherHistoryHandleImpl implements APICrawHandle {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @Override
    public void handle(String result) {
        logger.info("爬取结果：" + result);
    }

}

```



### APICrawThread.java

```java
package com.crawling.thread;

import com.alibaba.fastjson.JSONObject;
import com.crawling.utils.APICrawUtil;
import com.crawling.utils.HtmlCrawUtil;
import org.apache.commons.lang3.StringUtils;
import org.apache.http.Header;
import org.apache.http.message.BasicHeader;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class APICrawThread extends Thread {
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    private Map<String, Object> map;

    public APICrawThread(Map<String, Object> map) {
        this.map = map;
    }

    @Override
    public void run() {
        Object classObj = map.get("handle-class");
        if (classObj != null) {
            String handleClass = (String) classObj;
            try {
                Object typeObj = map.get("type");
                String type = "GET";
                if (null != typeObj) {
                    type = (String)typeObj;
                }
                Header[] headers = getHeader(map.get("header"));
                String result = "";
                if (type.equalsIgnoreCase("GET")) {// GET请求
                    result = APICrawUtil.crawGet(map.get("url").toString(), headers);
                } else {
                    Map dataMap = (Map) map.get("data");
                    convertMap(dataMap);
                    JSONObject data = new JSONObject(dataMap);
                    result = APICrawUtil.crawPost(map.get("url").toString(), data, headers);
                }
                Class<?> clazz = Class.forName(handleClass);
                Method handle = clazz.getDeclaredMethod("handle", String.class);
                handle.invoke(clazz.newInstance(), result);
            } catch (Exception e) {{
                e.printStackTrace();
            }}
        } else {
            logger.info("没有定义handle-class");
        }
    }

    /**
     * 获取header
     * @param header
     * @return
     */
    private Header[] getHeader(Object header) {
        if (header != null) {
            Map headerMap = (Map) header;
            Header[] headers = new BasicHeader[headerMap.size()];
            Iterator<Map.Entry<String, String>> it = headerMap.entrySet().iterator();
            int index = 0;
            while (it.hasNext()) {
                Map.Entry<String, String> entry = it.next();
                headers[index++] = new BasicHeader(entry.getKey(), entry.getValue());
            }
            return headers;
        }
        return null;
    }

    private void convertMap(Map dataMap) {
        // 处理对象里面是数组被解析成对象的问题
        Iterator<Map.Entry<String, String>> it = dataMap.entrySet().iterator();
        while (it.hasNext()) {
            Map.Entry<String, String> entry = it.next();
            Object valObj = entry.getValue();
            if (valObj instanceof Map) {
                Map valMap = (Map) valObj;
                if (valMap.get("0") != null) {
                    List<Object> list = new ArrayList<>();
                    Iterator<Map.Entry<String, Object>> subIt = valMap.entrySet().iterator();
                    while (subIt.hasNext()) {
                        Map.Entry<String, Object> subEntry = subIt.next();
                        list.add(subEntry.getValue());
                    }
                    dataMap.put(entry.getKey(), list);
                }
            }
        }
    }
}

```



### APICrawUtil.java

```java
package com.crawling.utils;

import com.alibaba.fastjson.JSONObject;
import org.apache.http.Header;
import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

import java.io.IOException;
import java.nio.charset.StandardCharsets;

/**
 * API接口爬取
 */
public class APICrawUtil {

    /**
     * GET请求
     *
     * @param url
     * @return
     * @throws Exception
     */
    public static String crawGet(String url, Header... headers) throws IOException {
        CloseableHttpClient client = HttpClients.createDefault();
        HttpGet httpGet = new HttpGet(url);
        if (headers != null) {
            httpGet.setHeaders(headers);
        }
        CloseableHttpResponse Response = client.execute(httpGet);
        HttpEntity entity = Response.getEntity();
        String content = EntityUtils.toString(entity, StandardCharsets.UTF_8);
        Response.close();
        client.close();
        return content;
    }

    /**
     * POST请求
     *
     * @param url
     * @param data
     * @param headers
     * @return
     * @throws IOException
     */
    public static String crawPost(String url, JSONObject data, Header... headers) throws IOException {
        HttpPost httpPost = new HttpPost(url);
        if (headers != null) {
            httpPost.setHeaders(headers);
        }
        if (data != null && data.size() > 0) {
            httpPost.setEntity((new StringEntity(data.toString(), StandardCharsets.UTF_8)));
        }
        CloseableHttpClient client = HttpClients.createDefault();
        CloseableHttpResponse response = client.execute(httpPost);
        HttpEntity httpEntity = response.getEntity();
        String content = EntityUtils.toString(httpEntity, StandardCharsets.UTF_8);
        response.close();
        client.close();
        return content;
    }

}

```



### CrawlingApplication.java

```java
package com.crawling;

import com.crawling.config.CrawConfiguration;
import com.crawling.thread.APICrawThread;
import com.crawling.thread.HtmlCrawThread;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;
import org.springframework.scheduling.support.CronTrigger;

import javax.annotation.PostConstruct;
import java.util.Map;

@SpringBootApplication
public class CrawlingApplication {
    @Autowired
    private ThreadPoolTaskScheduler threadPoolTaskScheduler;

    @Bean
    public ThreadPoolTaskScheduler threadPoolTaskScheduler() {
        ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
        // 设置池大小，不然只有单个执行
        threadPoolTaskScheduler.setPoolSize(100);
        return threadPoolTaskScheduler;
    }
    public static void main(String[] args) {
        SpringApplication.run(CrawlingApplication.class, args);
    }

    @Autowired
    private CrawConfiguration crawConfiguration;

    @PostConstruct
    public void init() {
        // 启动API爬取
        for (Map<String, Object> api : crawConfiguration.getApis()) {
            threadPoolTaskScheduler.schedule(new APICrawThread(api), new CronTrigger(api.get("cron").toString()));
        }
    }
}

```

