# storm整合kafka

> SpringBoot aop + kafka + ELK 将用户访问接口通过spring aop将数据传输到kafka，通过logstash将数据存入es。
>
> 本次讲解 storm消费kafka数据

## pom.xml

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
    <artifactId>ywf-storm</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>ywf-storm</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <storm.version>1.2.2</storm.version>
        <kafka.version>1.1.0</kafka.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.logging.log4j</groupId>
                    <artifactId>log4j-to-slf4j</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>ch.qos.logback</groupId>
                    <artifactId>logback-classic</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- storm -->
        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>storm-core</artifactId>
            <version>${storm.version}</version>
            <!--在本地模式运行的时候需要把下面的给注释掉-->
            <!--            <scope>provided</scope>-->
        </dependency>

        <!-- JVM上的实时监控类库 -->
        <dependency>
            <groupId>com.codahale.metrics</groupId>
            <artifactId>metrics-core</artifactId>
            <version>3.0.2</version>
            <!--   <scope>provided</scope>-->
        </dependency>

        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>storm-kafka</artifactId>
            <version>${storm.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.11</artifactId>
            <version>${kafka.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>${kafka.version}</version>
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
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```



## SystemLogBolt

```java
package com.ywf.storm.bolt;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseRichBolt;
import org.apache.storm.tuple.Tuple;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @Author:ywf
 */
public class SystemLogBolt extends BaseRichBolt {
    private OutputCollector collector;

    /**
     * 记录每个用户接口访问量
     */
    private Map<String, Integer> userMethodMap = new ConcurrentHashMap<>();

    @Override
    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
        this.collector = outputCollector;
    }

    @Override
    public void execute(Tuple tuple) {
        // !!! 一定要用bytes 上游kafkaspout 传递过来的数据
        byte[] bytes = tuple.getBinaryByField("bytes");
        String value = new String(bytes);
        JSONObject jsonObject = JSON.parseObject(value);
        String paramsStr = jsonObject.getString("params");
        JSONObject params = JSONObject.parseObject(paramsStr);
        String username = params.getString("username");

        int count = userMethodMap.getOrDefault(username, 0) + 1;
        userMethodMap.put(username, count);

        System.out.println(userMethodMap);
        // ack
        collector.ack(tuple);
    }

    @Override
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {

    }
}
```



## LogKafkaTopology

```java
package com.ywf.storm;

import com.ywf.storm.bolt.SystemLogBolt;
import org.apache.storm.Config;
import org.apache.storm.LocalCluster;
import org.apache.storm.kafka.BrokerHosts;
import org.apache.storm.kafka.KafkaSpout;
import org.apache.storm.kafka.SpoutConfig;
import org.apache.storm.kafka.ZkHosts;
import org.apache.storm.topology.TopologyBuilder;

import java.util.UUID;

/**
 * kafka 日志kafka拓扑
 * @Author:ywf
 */
public class LogKafkaTopology {
    public static void main(String[] args) {
        TopologyBuilder topologyBuilder = new TopologyBuilder();
        String zkConnStr = "node1:2181,node2:2181,node3:2181";
        BrokerHosts brokerHosts = new ZkHosts(zkConnStr);
        String topicName = "ywf-system-log";
        String zkRoot = "/ywf-system-log";
        String id = UUID.randomUUID().toString();
        /**
         * BrokerHosts hosts zk集群地址
         * String topic, 主题
         * String zkRoot, zk主题地址
         * String id 唯一标识
         */
        SpoutConfig spoutConfig = new SpoutConfig(brokerHosts, topicName, zkRoot, id);
        KafkaSpout kafkaSpout = new KafkaSpout(spoutConfig);

        topologyBuilder.setSpout("kafkaSpout", kafkaSpout).setNumTasks(2);
        topologyBuilder.setBolt("systemLogBolt", new SystemLogBolt(),2).shuffleGrouping("kafkaSpout")
                .setNumTasks(2);

        Config config = new Config();
        config.setNumWorkers(2);
        config.setDebug(true);
        /**
         * 本地模式storm
         */
        LocalCluster cluster = new LocalCluster();
        cluster.submitTopology("logKafkaTopology", config, topologyBuilder.createTopology());
    }
}
```



## 本地启动应用