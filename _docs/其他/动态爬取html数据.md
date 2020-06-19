---
title: 定时任务+多线程+xpath爬取网页
category: 其他
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
        <!-- https://mvnrepository.com/artifact/cn.wanghaomiao/JsoupXpath -->
        <dependency>
            <groupId>cn.wanghaomiao</groupId>
            <artifactId>JsoupXpath</artifactId>
            <version>2.3.2</version>
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
  htmls:
    - name: "上海台风研究所"
      url: "http://www2.sti.org.cn/index.php?m=content&c=index&a=lists&catid=82&page={0}"
      href: "//ul[@class='problem_ol']//ul[@class='problem_ol']//li//h2//a"
      host: "http://www2.sti.org.cn"
      title: "//div[@class='problem']//h1"
      content: "//div[@class='problem_conts']"
      time: "//div[@class='problem']//h3//span"
      param:
        - min: 1
          max: 4
      handle-class: "com.crawling.handle.html.NewsHandleImpl"
      cron: "0/30 * * * * *"

    - name: "中国天气网"
      url: "http://news.weather.com.cn"
      href: "//div[@class='newcard'][1]/p/a"
      title: "//p[@class='articleTittle'][1]"
      content: "//div[@class='articleBody'][1]"
      time: "//div[@class='articleTimeSizeleft']//span[1]"
      source: "//div[@class='articleTimeSizeleft']//span[2]/a"
      handle-class: "com.crawling.handle.html.NewsHandleImpl"
      cron: "0/30 * * * * *"

    - name: "中国天气网"
      url: "http://so.mnw.cn/cse/search?entry=1&s=6497631107033654394&q=2020%E5%8F%B0%E9%A3%8E"
      href: "//div[@id='results']//div//h3//a"
      title: "//div[@class='iw ic']//div[@class='l']//h1"
      content: "//div[@class='icontent']"
      time: "//div[@class='iw ic']//div[@class='l']//div//div[@class='il']//span[2]"
      source: "//div[@class='iw ic']//div[@class='l']//div//div[@class='il']//span[1]"
      handle-class: "com.crawling.handle.html.NewsHandleImpl"
      cron: "0/30 * * * * *"

    - name: "中央气象台"
      url: "http://www.nmc.cn/publish/typhoon/typhoon_new.html"
      title: "//div[@class='title']"
      content: "//div[@id='text']//div[@class='writing']"
      time: "//div[@class='ctitle']//span[2]"
      source: "//div[@class='ctitle']//span[1]"
      handle-class: "com.crawling.handle.html.NewsHandleImpl"
      cron: "0/30 * * * * *"

    - name: "福建闽南网"
      url: "http://so.mnw.cn/cse/search?entry=1&s=6497631107033654394&q=2020%E5%8F%B0%E9%A3%8E"
      href: "//div[@id='results']//div//h3//a"
      title: "//div[@class='iw ic']//div[@class='l']//h1[1]"
      content: "//div[@class='icontent'][1]"
      time: "//div[@class='iw ic']//div[@class='l']//div//div[@class='il']//span[2]"
      source: "//div[@class='iw ic']//div[@class='l']//div//div[@class='il']//span[1]"
      handle-class: "com.crawling.handle.html.NewsHandleImpl"
      cron: "0/30 * * * * *"

    - name: "中国气象科普网"
      url: "http://www.qxkp.net/zhfy/tffy/index.shtml"
      host: "http://www.qxkp.net/"
      href: "//div[@class='list_R pdR26']//ul//li//div[@class='fl']//a"
      title: "//div[@class='w986']//div[1]"
      content: "//div[@id='BodyLabel'][1]"
      time: "//div[@class='titleInfo'][1]"
      source: "//div[@class='titleInfo'][1]"
      reg:
        time: "发布时间：(.*)(&nbsp;&nbsp;&nbsp;&nbsp;来源：)"
        source: "来源：(.*)(&nbsp;&nbsp;&nbsp;&nbsp;)"
      handle-class: "com.crawling.handle.html.NewsHandleImpl"
      cron: "0/30 * * * * *"

    - name: "福建省气象局"
      url: "http://fj.cma.gov.cn/xwzx/qxyw/qxxw/fj/index_{0}.htm"
      href: "//div[@class='problem']//ul[@class='problem_ol']//ul//li//h2//a"
      title: "//div[@class='main pad-30']//div[@class='xl-nr clearflx']//div[@class='zf-xl-tit mar-B30 clearflx']//h3"
      content: "//div[@class='main pad-30']//div[@class='xl-nr clearflx']//div[@id='detailCon']"
      time: "//div[@class='main pad-30']//div[@class='xl-nr clearflx']//div[@class='zf-xl-tit mar-B30 clearflx']//h5"
      source: "//div[@class='main pad-30']//div[@class='xl-nr clearflx']//div[@class='zf-xl-tit mar-B30 clearflx']//h5"
      img: "//table//tr//td[@class='bt_content']//div//div//p//img"
      param:
        - min: 0
          max: 2
      handle-class: "com.crawling.handle.html.NewsHandleImpl"
      cron: "0/30 * * * * *"

```



### CrawConfiguration.java

```java
package com.crawling.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;

@Component
@ConfigurationProperties(prefix = "craws")
public class CrawConfiguration {
    private List<Map<String, Object>> htmls;

    public List<Map<String, Object>> getHtmls() {
        return htmls;
    }

    public void setHtmls(List<Map<String, Object>> htmls) {
        this.htmls = htmls;
    }

}

```



### HtmlCrawHandle.java

```java
package com.crawling.handle;

import java.util.List;
import java.util.Map;

public interface HtmlCrawHandle {

    /**
     * 数据处理
     * @param result
     */
    void handle(List<Map<String, String>> result);
}

```



### NewsHandleImpl.java

```java
package com.crawling.handle.html;

import com.crawling.handle.HtmlCrawHandle;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * 新闻类数据处理
 */
public class NewsHandleImpl implements HtmlCrawHandle {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @Override
    public void handle(List<Map<String, String>> data) {
        logger.info("爬取结果：" + data);
        // 数据处理， 如图片存入oss等操作
        for (Map<String, String> map : data) {
            // 文章内容处理
            String content = convertContent(map.get("content"));
        }
        // TODO 保存入库
    }

    /**
     * 网页内容处理
     * @param content
     * @return
     */
    private String convertContent(String content) {
        String newContent = "";
        // 替换图片，上传到OSS
        replaceImg(content);
        // TODO 过滤iframe标签
        return newContent;
    }

    /**
     * 将图片上传到OSS,并替换图片地址
     * @param content
     */
    private void replaceImg(String content) {
        List<String> imageList = replaceHtmlTag(content, "img", "src");
        if (imageList.size() > 0) {
            logger.info("要替换的图片列表为：" + imageList);
            for (String image : imageList) {
                // 上传到OSS 并替换content image
            }
        }
    }

    /**
     * 替换指定标签的属性和值
     * @param str 需要处理的字符串
     * @param tag 标签名称
     * @param tagAttrib 要替换的标签属性值
     * @return
     */
    public List<String> replaceHtmlTag(String str, String tag, String tagAttrib) {
        List<String> tagList = new ArrayList<>();
        String regxpForTag = "<\\s*" + tag + "\\s+([^>]*)\\s*" ;
        String regxpForTagAttrib = tagAttrib + "=\\s*\"([^\"]+)\"" ;
        Pattern patternForTag = Pattern.compile (regxpForTag,Pattern. CASE_INSENSITIVE);
        Pattern patternForAttrib = Pattern.compile (regxpForTagAttrib,Pattern. CASE_INSENSITIVE);
        Matcher matcherForTag = patternForTag.matcher(str);
        StringBuffer sb = new StringBuffer();
        boolean result = matcherForTag.find();
        while (result) {
            Matcher matcherForAttrib = patternForAttrib.matcher(matcherForTag.group(1));
            if (matcherForAttrib.find()) {
                String attributeStr = matcherForAttrib.group(1);
                tagList.add(attributeStr);
            }
            result = matcherForTag.find();
        }
        matcherForTag.appendTail(sb);
        return tagList;
    }

}

```



### HtmlCrawThread.java

```java
package com.crawling.thread;

import com.crawling.utils.HtmlCrawUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.lang.reflect.Method;
import java.util.List;
import java.util.Map;

public class HtmlCrawThread extends Thread {
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    private Map<String, Object> map;

    public HtmlCrawThread(Map<String, Object> map) {
        this.map = map;
    }

    @Override
    public void run() {
        Object classObj = map.get("handle-class");
        if (classObj != null) {
            String handleClass = (String) classObj;
            try {
                List<Map<String, String>> result = HtmlCrawUtil.crawGet(map);
                Class<?> clazz = Class.forName(handleClass);
                Method handle = clazz.getDeclaredMethod("handle", List.class);
                handle.invoke(clazz.newInstance(), result);
            } catch (Exception e) {{
                e.printStackTrace();
            }}
        } else {
            logger.info("没有定义handle-class");
        }

    }
}

```



### HtmlCrawUtil.java

```java
package com.crawling.utils;

import org.apache.commons.lang.StringEscapeUtils;
import org.htmlcleaner.XPatherException;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.seimicrawler.xpath.JXDocument;
import org.seimicrawler.xpath.JXNode;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.util.StringUtils;

import java.io.IOException;
import java.util.*;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * html解析爬取
 */
public class HtmlCrawUtil {
    private static Logger logger = LoggerFactory.getLogger(HtmlCrawUtil.class);

    public static List<Map<String, String>> crawGet(Map<String, Object> html) throws IOException {
        List<Map<String, String>> result;
        logger.info("开始爬取 " + html.get("name"));
        if (null != html.get("href")) {
            result = crawGetDetail(html);
        } else {
            result = crawGetByHtml(html);
        }
        convert(html, result);
        logger.info("爬取结束 ");
        return result;
    }

    /**
     * 数据处理
     *
     * @param html
     * @param list
     */
    private static void convert(Map<String, Object> html, List<Map<String, String>> list) {
        // 正则表达式处理
        Object object = html.get("reg");
        if (null != object) {
            for (Map<String, String> resultMap : list) {
                Map regMap = (Map) object;
                Iterator<Map.Entry<String, String>> it = regMap.entrySet().iterator();
                while (it.hasNext()) {
                    Map.Entry<String, String> entry = it.next();
                    Pattern pattern = Pattern.compile(entry.getValue());   //正则匹配
                    Matcher matcher = pattern.matcher(resultMap.get(entry.getKey()).toString());
                    while (matcher.find()) {
                        resultMap.put(entry.getKey(), matcher.group(1));
                    }
                }
            }
        }
    }

    /**
     * 当前网页包含超链接，标题、来源、作者等信息
     *
     * @param html
     * @return
     * @throws IOException
     * @throws XPatherException
     */
    private static List<Map<String, String>> crawGetByHtml(Map<String, Object> html) {
        List<Map<String, String>> result = new ArrayList<>();
        List<String> urls = getUrls(html);
        for (String url : urls) {
            try {
                long startTime = System.currentTimeMillis();
                Document doc = Jsoup.connect(url).get();
                JXDocument jxDocument = JXDocument.create(doc);
                Iterator<Map.Entry<String, Object>> it = html.entrySet().iterator();
                Map resultMap = new HashMap();
                while (it.hasNext()) {
                    Map.Entry<String, Object> entry = it.next();
                    if (entry.getValue().toString().startsWith("//")) {
                        List<JXNode> elNodes = jxDocument.selN(entry.getValue().toString());
                        if (elNodes != null && elNodes.size() > 0) {
                            if (entry.getKey().equals("content")) {
                                resultMap.put(entry.getKey(),elNodes.get(0).asElement().html());
                            } else {
                                resultMap.put(entry.getKey(),elNodes.get(0).asElement().text());
                            }
                        }
                    }
                }
                if (resultMap.size() > 0) {
                    result.add(resultMap);
                }
                long endTime = System.currentTimeMillis();
                logger.info("爬取" + url + "耗时 " + (endTime - startTime) + "ms");
            } catch (Exception e) {
                logger.error(e.getMessage(), e.getCause());
            }
        }
        return result;
    }

    /**
     * 弹出窗口页面包含超链接，标题、来源、作者、内容在新页面
     *
     * @param html
     * @return
     * @throws IOException
     * @throws XPatherException
     */
    private static List<Map<String, String>> crawGetDetail(Map<String, Object> html) throws IOException {
        List<Map<String, String>> result = new ArrayList<>();
        List<String> urls = getUrls(html);
        for (String url : urls) {
            JXDocument jxDocument = JXDocument.create(Jsoup.connect(url).get());
            List<JXNode> elNodes = jxDocument.selN(html.get("href").toString());
            Object host = html.get("host");
            for (JXNode node : elNodes) {
                String href = node.asElement().attributes().get("href");
                long startTime = System.currentTimeMillis();
                if (!StringUtils.isEmpty(host) && !href.contains(host.toString())) {
                    href = host + href;
                }
                try {
                    href = StringEscapeUtils.unescapeHtml(href);
                    jxDocument = JXDocument.create(Jsoup.connect(href).get());
                    Iterator<Map.Entry<String, Object>> it = html.entrySet().iterator();
                    Map resultMap = new HashMap();
                    while (it.hasNext()) {
                        Map.Entry<String, Object> entry = it.next();
                        try {
                            if (entry.getValue().toString().startsWith("//")) {
                                elNodes = jxDocument.selN(entry.getValue().toString());
                                if (elNodes != null && elNodes.size() > 0) {
                                    if (entry.getKey().equals("content")) {
                                        resultMap.put(entry.getKey(), elNodes.get(0).asElement().html());
                                    } else {
                                        resultMap.put(entry.getKey(), elNodes.get(0).asElement().text());
                                    }
                                }
                            }
                        } catch (Exception e) {
                            logger.error(e.getMessage(), e.getCause());
                        }
                    }
                    long endTime = System.currentTimeMillis();
                    logger.info("爬取" + href + "，耗时 " + (endTime - startTime) + "ms");
                    if (resultMap.size() > 0) {
                        result.add(resultMap);
                    }
                } catch (Exception e) {
                    logger.error(e.getMessage(), e.getCause());
                }
            }
        }
        return result;
    }

    /**
     * 获取完整URL,如拼接分页和特殊字段
     *
     * @param html
     * @return
     */
    private static List<String> getUrls(Map<String, Object> html) {
        List<String> urls = new ArrayList<>();
        Object paramObj = html.get("param");
        if (null != paramObj) {
            Map paramList = (Map) paramObj;
            Iterator<Map.Entry<String, Object>> it = paramList.entrySet().iterator();
            while (it.hasNext()) {
                Map.Entry<String, Object> entry = it.next();
                if (entry.getValue() instanceof Map) {
                    Map paramMap = (Map) entry.getValue();
                    Object minObj = paramMap.get("min"), maxObj = paramMap.get("max");
                    if (null != minObj && null != maxObj) {
                        int min = Integer.parseInt(minObj.toString()), max = Integer.parseInt(maxObj.toString());
                        for (int i = min; i <= max; i++) {
                            String url = html.get("url").toString().replace("{" + entry.getKey() + "}", String.valueOf(i));
                            urls.add(url);
                        }
                    }
                }
            }
        }
        if (urls.size() == 0) {
            urls.add(html.get("url").toString());
        }
        return urls;
    }
}

```



### CrawlingApplication.java

```java
package com.crawling;

import com.crawling.config.CrawConfiguration;
import com.crawling.thread.CrawThread;
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
        // 启动网页爬取
        for (Map<String, Object> html : crawConfiguration.getHtmls()) {
            threadPoolTaskScheduler.schedule(new CrawThread(html), new CronTrigger(html.get("cron").toString()));
        }
    }
}

```

