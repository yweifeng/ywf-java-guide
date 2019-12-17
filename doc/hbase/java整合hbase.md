# java整合hbase

### 环境

- **Hbase**: 2.1.1
- **Hadoop**: 3.1.1
- **springboot**: 2.2.2.RELEASE



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
    <artifactId>hbase</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>hbase</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <hbase.version>2.1.1</hbase.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
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
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>${hbase.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>javax.servlet</groupId>
                    <artifactId>servlet-api</artifactId>
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



### application.yml

```properties
hbase:
  config:
    hbase.zookeeper.quorum: wdnode31,wdnode32,wdnode33
    hbase.zookeeper.port: 2181
    hbase.zookeeper.znode: /hbase-unsecure
    hbase.client.keyvalue.maxsize: 1572864000
```



### HbaseProperties.java

```java
package com.ywf.hbase.config;

import org.springframework.boot.context.properties.ConfigurationProperties;

import java.util.Map;

/**
 * @Author:ywf
 */
@ConfigurationProperties(prefix = "hbase")
public class HbaseProperties {

    private Map<String, String> config;
    public Map<String, String> getConfig() {
        return config;
    }
    public void setConfig(Map<String, String> config) {
        this.config = config;
    }
}
```



### HbaseConfig.java

```java
package com.ywf.hbase.config;

import org.apache.hadoop.hbase.HBaseConfiguration;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Configuration;

import java.util.Map;
import java.util.Set;

/**
 * @Author:ywf
 */
@Configuration
@EnableConfigurationProperties(HbaseProperties.class)
public class HbaseConfig {
    private final HbaseProperties properties;

    public HbaseConfig(HbaseProperties properties) {
        this.properties = properties;
    }

    public org.apache.hadoop.conf.Configuration configuration() {
        org.apache.hadoop.conf.Configuration configuration = HBaseConfiguration.create();
        Map<String, String> config = properties.getConfig();
        Set<String> keySet = config.keySet();
        for (String key : keySet) {
            configuration.set(key, config.get(key));
        }
        return configuration;
    }
}
```



### 测试类HbaseApplicationTests.java

```java
package com.ywf.hbase;

import com.ywf.hbase.client.HbaseClient;
import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.io.IOException;

@SpringBootTest
@RunWith(SpringJUnit4ClassRunner.class)
class HbaseApplicationTests {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    private final static String TABLE = "quick-hbase-table";
    private final static String TABLE_FAM_1 = "quick";
    private final static String TABLE_FAM_2 = "hbase";

    @Autowired
    private HbaseClient hBaseClient;

    @Test
    public void createTable() throws IOException {
        hBaseClient.createTable(TABLE, TABLE_FAM_1, TABLE_FAM_2);
    }

    /**
     * 向TABLE中插入一条记录，列族quick下，写了两个key，speed和感觉feel，列族hbase下插入三个key 动作action、时间time和用户user，类似一个日志
     */
    @Test
    public void insertOrUpdate() throws IOException {
        hBaseClient.insertOrUpdate(TABLE, "1", TABLE_FAM_1, "speed", "1km/h");
        hBaseClient.insertOrUpdate(TABLE, "1", TABLE_FAM_1, "feel", "better");
        hBaseClient.insertOrUpdate(TABLE, "1", TABLE_FAM_2, "action", "create table");
        hBaseClient.insertOrUpdate(TABLE, "1", TABLE_FAM_2, "time", "2019年07月20日17:52:53");
        hBaseClient.insertOrUpdate(TABLE, "1", TABLE_FAM_2, "user", "admin");
        /**
         * shell 结果
         * hbase(main):007:0> scan 'quick-hbase-table'
         * ROW                                          COLUMN+CELL
         *  1                                           column=hbase:action, timestamp=1563616496366, value=create table
         *  1                                           column=hbase:time, timestamp=1563616496379, value=2019\xE5\xB9\xB407\xE6\x9C\x8820\xE6\x97\xA517:52:53
         *  1                                           column=hbase:user, timestamp=1563616496384, value=admin
         *  1                                           column=quick:feel, timestamp=1563616496362, value=better
         *  1                                           column=quick:speed, timestamp=1563616496353, value=1km/h
         * 1 row(s)
         */
        hBaseClient.insertOrUpdate(TABLE, "2", TABLE_FAM_2, "user", "admin");

    }

    @Test
    public void deleteRow() throws IOException {
        hBaseClient.deleteRow(TABLE, "2");
    }


    @Test
    public void deleteColumnFamily() throws IOException {
        hBaseClient.deleteColumnFamily(TABLE, "1", TABLE_FAM_2);
    }

    @Test
    public void deleteColumn() throws IOException {
        hBaseClient.deleteColumn(TABLE, "1", TABLE_FAM_2, "action");
    }

    @Test
    public void deleteTable() throws IOException {
        hBaseClient.deleteTable(TABLE);
    }

    @Test
    public void getValue() {
        String result = hBaseClient.getValue(TABLE, "1", TABLE_FAM_2, "time");
        logger.info(result);
    }

    @Test
    public void selectOneRow() throws IOException {
        hBaseClient.selectOneRow(TABLE, "1");
    }

    @Test
    public void scanTable() throws IOException {
        hBaseClient.scanTable(TABLE, "{FILTER=>\"PrefixFilter('2019')\"");
    }

    @Test
    public void tableExists() throws IOException {
        logger.info("table exists " + hBaseClient.tableExists(TABLE));
    }

}
```

- 项目地址:[https://github.com/yweifeng/ywf-hbase](https://github.com/yweifeng/ywf-hbase)



- 运行时报错：

```
java.io.IOException: Could not locate executable F:\soft\hadoop-3.0.0\bin\bin\winutils.exe in the Hadoop binaries.
```



- 解决方案：

  - 下载[winutil.exe](resource/winutil.exe)
  - 新建环境变量

  ```
  HADOOP_HOME=wintuil.exe存放的地址
  
  配置path 
    %HADOOP_HOME%\bin
  ```
  
  - 重启电脑运行

