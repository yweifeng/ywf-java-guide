---
title: java整合hbase
category: hbase
order: 3
---



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
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
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
    hbase.zookeeper.quorum: master,slave1,slave2
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



### HbaseClient.java

```java
package com.ywf.hbase.util;

import com.ywf.hbase.config.HbaseConfig;
import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.hbase.CompareOperator;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.filter.RowFilter;
import org.apache.hadoop.hbase.filter.SubstringComparator;
import org.apache.hadoop.hbase.util.Bytes;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * @Author:ywf
 */
@Component
public class HbaseClient {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @Autowired
    private HbaseConfig hbaseConfig;
    private Admin admin;
    private Connection connection;

    @PostConstruct
    public void init() {
        try {
            connection = ConnectionFactory.createConnection(hbaseConfig.configuration());
            admin = connection.getAdmin();
        } catch (IOException e) {
            logger.error("hbase连接异常:" + e.getMessage(), e.getCause());
            e.printStackTrace();
        }
    }

    /**
     * 创建表
     * shell:
     *      create 'tableName','[Column Family1]','[Column Family2]'
     * @param tableName 表名
     * @param columnFamilies 列族
     */
    public void createTable(String tableName, String... columnFamilies) throws IOException {
        TableName name = TableName.valueOf(tableName);
        boolean exists = tableExists(tableName);
        if (!exists) {
            // 表描述器创建者
            TableDescriptorBuilder tableDescriptorBuilder = TableDescriptorBuilder.newBuilder(name);
            List<ColumnFamilyDescriptor> columnFamilyDescriptorList = new ArrayList<>();
            // 循环列族
            for (String columnFamily : columnFamilies) {
                // 列族描述器
                ColumnFamilyDescriptor columnFamilyDescriptor =
                        ColumnFamilyDescriptorBuilder.newBuilder(columnFamily.getBytes()).build();
                columnFamilyDescriptorList.add(columnFamilyDescriptor);
            }
            tableDescriptorBuilder.setColumnFamilies(columnFamilyDescriptorList);
            // 创建表描述器
            TableDescriptor tableDescriptor = tableDescriptorBuilder.build();
            // 创建表
            admin.createTable(tableDescriptor);
            logger.info(String.format("table[%s]创建成功", tableName));
        } else {
            logger.info(String.format("table[%s]已存在", tableName));
        }
    }

    /**
     * 删除表
     * shell:
     *      disable 'tableName'
     *      delete 'tableName'
     * @param tableName 表名
     * @return 是否删除成功
     */
    public boolean deleteTable(String tableName) throws IOException {
        if (!tableExists(tableName)) {
            logger.info(String.format("要删除的table[%s]不存在", tableName));
            return false;
        } else {
            TableName name = TableName.valueOf(tableName);
            admin.disableTable(name);
            admin.deleteTable(name);
            return true;
        }
    }

    /**
     * 判断表是否存在
     * shell:
     *      exists 'tableName'
     * @param tableName 表名
     * @return 表名是否存在
     */
    public boolean tableExists(String tableName) throws IOException {
        return admin.tableExists(TableName.valueOf(tableName));
    }

    /**
     * 添加或者更新数据
     * shell:
     *      put 'tableName', 'rowKey', 'columnFamily:columnName', 'columnValue'
     * @param tableName 表名
     * @param rowKey 行键
     * @param columnFamily 列族
     * @param columnNames 字段名称数组
     * @param columnValues 字段值数组
     * @return
     */
    public void insertOrUpdate(String tableName, String rowKey, String columnFamily, String[] columnNames, String[] columnValues) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Put put = new Put(Bytes.toBytes(rowKey));
        for (int i = 0; i < columnNames.length; i++) {
            put.addColumn(Bytes.toBytes(columnFamily), Bytes.toBytes(columnNames[i]), Bytes.toBytes(columnValues[i]));
        }
        table.put(put);
    }

    /**
     * 删除一行
     * shell:
     *      delete 'tableName', 'rowKey'
     * @param tableName 表名
     * @param rowKey 行键
     */
    public void deleteRow(String tableName, String rowKey) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Delete delete = new Delete(Bytes.toBytes(rowKey));
        table.delete(delete);
    }

    /**
     * 删除一行列族
     * shell:
     *      delete 'tableName', 'rowKey', 'columnFamily'
     * @param tableName 表名
     * @param rowKey 行键
     * @param columnFamily 列族
     */
    public void deleteColumnFamily(String tableName, String rowKey, String columnFamily) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Delete delete = new Delete(Bytes.toBytes(rowKey));
        delete.addFamily(Bytes.toBytes(columnFamily));
        table.delete(delete);
    }

    /**
     * 删除column
     * shell:
     *      delete 'tableName','rowKey','columnFamily:columnName'
     * @param tableName 表名
     * @param rowKey 行键
     * @param columnFamily 列族
     * @param columnName 列名
     */
    public void deleteColumn(String tableName, String rowKey, String columnFamily, String columnName) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Delete delete = new Delete(Bytes.toBytes(rowKey));
        delete.addColumn(Bytes.toBytes(columnFamily), Bytes.toBytes(columnName));
        table.delete(delete);
    }

    /**
     * 获取指定表指定行中指定列族的所有列的数据信息
     * shell:
     *      get 'tableName', 'rowKey', 'columnFamily'
     * @param tableName 表名
     * @param rowKey 行键
     * @param columnFamily 列族
     * @return
     */
    public Result selectOneRow(String tableName, String rowKey, String columnFamily) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Get get = new Get(Bytes.toBytes(rowKey));
        get.addFamily(Bytes.toBytes(columnFamily));
        return table.get(get);
    }

    /**
     * 获取指定表指定行中指定列族指定列名的数据信息
     * shell:
     *      get 'tableName','rowKey','columnFamily:columnName','columnFamily:columnName'
     * @param tableName 表名
     * @param rowKey 行键
     * @param columnFamily 列族
     * @param columnNames 列名
     * @return
     */
    public Result getColumnValue(String tableName, String rowKey, String columnFamily, String... columnNames) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Get get = new Get(Bytes.toBytes(rowKey));
        for (String columnName : columnNames) {
            get.addColumn(Bytes.toBytes(columnFamily), Bytes.toBytes(columnName));
        }
        return table.get(get);
    }


    /**
     * 指定过滤条件进行全表扫描
     * shell:
     *      scan 'tableName'
     * @param tableName 表名
     * @return
     */
    public ResultScanner scanTable(String tableName) throws IOException {
        return scanTable(tableName, null);
    }


    /**
     * 指定过滤条件进行全表扫描
     * shell:
     *      scan 'tableName', {FILTER=>"PrefixFilter(100)"}
     * @param tableName 表名
     * @param rowKeyFilter 过滤条件
     * @return
     */
    public ResultScanner scanTable(String tableName, String rowKeyFilter) throws IOException {
        return scanTable(tableName, rowKeyFilter, null, null);
    }

    /**
     * 指定过滤条件进行全表扫描
     * shell:
     *      scan 'tableName', {FILTER=>"PrefixFilter(100)"}
     * @param tableName 表名
     * @param rowKeyFilter 过滤条件
     * @param minStamp 最小时间戳
     * @param maxStamp 最大时间戳
     * @return
     */
    public ResultScanner scanTable(String tableName, String rowKeyFilter, Long minStamp, Long maxStamp) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Scan scan = new Scan();
        if (StringUtils.isNotBlank(rowKeyFilter)) {
            // 添加行键过滤条件
            RowFilter rowFilter = new RowFilter(CompareOperator.EQUAL, new SubstringComparator(rowKeyFilter));
            scan.setFilter(rowFilter);
        }

        if (null != minStamp && null != maxStamp) {
            scan.setTimeRange(minStamp, maxStamp);
        }

        return table.getScanner(scan);
    }
}
```



### 实体类Student.java

```java
package com.ywf.hbase.entity;

/**
 * @Author:ywf
 */
public class Student {

    private String name;

    private String sex;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                '}';
    }
}
```



### 测试类HbaseApplicationTests.java

```java
package com.ywf.hbase;

import com.ywf.hbase.entity.Student;
import com.ywf.hbase.util.HbaseClient;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.client.ResultScanner;
import org.apache.hadoop.hbase.util.Bytes;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;
import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.List;

@SpringBootTest
class HbaseApplicationTests {
    private Logger log = LoggerFactory.getLogger(this.getClass());

    @Autowired
    private HbaseClient hbaseClient;
    private String tableName = "student";
    private String columnFamily = "info";
    private String rowKey = "1000";

    /**
     * 创建表
     */
    @Test
    void createTable() throws IOException {
        hbaseClient.createTable(tableName, columnFamily);
    }

    /**
     * 判断表是否存在
     */
    @Test
    void tableExists() throws IOException {
        hbaseClient.tableExists(tableName);
    }

    /**
     * 删除表
     */
    @Test
    void deleteTable() throws IOException {
        boolean res = hbaseClient.deleteTable(tableName);
        log.info(String.format("[%s]删除%s", tableName, res ? "成功": "失败"));
    }

    /**
     * 添加数据
     */
    @Test
    void insertOrUpdate() throws IOException {
        String[] columnNames = new String[] {"name", "sex"};
        String[] columnValues = new String[] {"zhangsan", "1"};
        hbaseClient.insertOrUpdate(tableName, rowKey, columnFamily, columnNames, columnValues);
    }

    /**
     * 删除一行
     */
    @Test
    void deleteRow() throws IOException {
        hbaseClient.deleteRow(tableName, rowKey);
    }

    /**
     * 删除列族
     */
    @Test
    void deleteColumnFamily() throws IOException {
        hbaseClient.deleteColumnFamily(tableName, rowKey, columnFamily);
    }

    /**
     * 删除column
     */
    @Test
    void deleteColumn() throws IOException {
        String columnName = "sex";
        hbaseClient.deleteColumn(tableName, rowKey, columnFamily, columnName);
    }

    /**
     * 获取指定表指定行中指定列族的所有列的数据信息
     */
    @Test
    void selectOneRow() throws IOException, NoSuchFieldException, IllegalAccessException {
        Result result = hbaseClient.selectOneRow(tableName, rowKey, columnFamily);
        Student student = new Student();
        for (Cell cell : result.rawCells()) {
            String columnName = Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength());
            String columnValue = Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
            // 反射到Java实体类
            Field f = student.getClass().getDeclaredField(columnName);
            f.setAccessible(true);
            f.set(student, columnValue);
        }
        log.info(student.toString());
    }

    /**
     * 获取指定表指定行中指定列族指定列名的数据信息
     */
    @Test
    void getColumnValue() throws IOException {
        Result result = hbaseClient.getColumnValue(tableName, rowKey, columnFamily, "name");
        for (Cell cell : result.rawCells()) {
            String columnName = Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength());
            String columnValue = Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
            log.info(columnName + "=" + columnValue);
        }
    }

    /**
     * 指定过滤条件进行全表扫描
     */
    @Test
    void scanTable() throws IOException, NoSuchFieldException, IllegalAccessException {
        ResultScanner results = hbaseClient.scanTable(tableName, "100");
        List<Student> studentList = new ArrayList<>();
        for (Result result : results) {
            Student student = new Student();
            for (Cell cell : result.rawCells()) {
                String columnName = Bytes.toString(cell.getQualifierArray(), cell.getQualifierOffset(), cell.getQualifierLength());
                String columnValue = Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                // 反射到Java实体类
                Field f = student.getClass().getDeclaredField(columnName);
                f.setAccessible(true);
                f.set(student, columnValue);
            }
            studentList.add(student);
        }
        log.info(studentList.toString());
    }

}

```

- 项目地址:[https://github.com/yweifeng/ywf-hbase](https://github.com/yweifeng/ywf-hbase)



- 运行时报错：

```
java.io.IOException: Could not locate executable F:\soft\hadoop-3.0.0\bin\bin\winutils.exe in the Hadoop binaries.
```



- 解决方案：

  - 下载[winutils.exe](resource/bin/winutils.exe)
  - 新建环境变量

  ```
  HADOOP_HOME=wintuils.exe存放的地址
  
  配置path 
    %HADOOP_HOME%\bin
  ```
  
  - 重启电脑运行

