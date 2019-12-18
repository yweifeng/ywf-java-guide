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



### HbaseClient.java

```java
package com.ywf.hbase.client;

import com.ywf.hbase.config.HbaseConfig;
import org.apache.commons.lang3.StringUtils;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CompareOperator;
import org.apache.hadoop.hbase.TableExistsException;
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
import java.util.NavigableMap;

/**
 * @Author:ywf
 */
@Component
public class HbaseClient {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @Autowired
    private HbaseConfig config;

    private static Connection connection = null;
    private static Admin admin = null;

    @PostConstruct
    public void init() {
        if (connection != null) {
            return;
        }
        try {
            connection = ConnectionFactory.createConnection(config.configuration());
            admin = connection.getAdmin();
        } catch (IOException e) {
            logger.error("HBase create connection failed: {}", e);
        }
    }

    /**
     * create 'tableName','[Column Family 1]','[Column Family 2]'
     * @param tableName
     * @param columnFamilies 列族名
     * @throws IOException
     */
    public void createTable(String tableName, String... columnFamilies) throws IOException {
        TableName name = TableName.valueOf(tableName);
        boolean isExists = this.tableExists(tableName);
        if (isExists) {
            throw new TableExistsException(tableName + "is exists!");
        }
        TableDescriptorBuilder descriptorBuilder = TableDescriptorBuilder.newBuilder(name);
        List<ColumnFamilyDescriptor> columnFamilyList = new ArrayList<>();
        for (String columnFamily : columnFamilies) {
            ColumnFamilyDescriptor columnFamilyDescriptor = ColumnFamilyDescriptorBuilder
                    .newBuilder(columnFamily.getBytes()).build();
            columnFamilyList.add(columnFamilyDescriptor);
        }
        descriptorBuilder.setColumnFamilies(columnFamilyList);
        TableDescriptor tableDescriptor = descriptorBuilder.build();
        admin.createTable(tableDescriptor);
    }

    /**
     * put <tableName>,<rowKey>,<family:column>,<value>,<timestamp>
     * @param tableName
     * @param rowKey
     * @param columnFamily
     * @param column
     * @param value
     * @throws IOException
     */
    public void insertOrUpdate(String tableName, String rowKey, String columnFamily, String column, String value)
            throws IOException {
        this.insertOrUpdate(tableName, rowKey, columnFamily, new String[]{column}, new String[]{value});
    }

    /**
     * put <tableName>,<rowKey>,<family:column>,<value>,<timestamp>
     * @param tableName
     * @param rowKey
     * @param columnFamily
     * @param columns
     * @param values
     * @throws IOException
     */
    public void insertOrUpdate(String tableName, String rowKey, String columnFamily, String[] columns, String[] values)
            throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Put put = new Put(Bytes.toBytes(rowKey));
        for (int i = 0; i < columns.length; i++) {
            put.addColumn(Bytes.toBytes(columnFamily), Bytes.toBytes(columns[i]), Bytes.toBytes(values[i]));
            table.put(put);
        }
    }

    /**
     * @param tableName
     * @param rowKey
     * @throws IOException
     */
    public void deleteRow(String tableName, String rowKey) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Delete delete = new Delete(rowKey.getBytes());
        table.delete(delete);
    }

    /**
     * @param tableName
     * @param rowKey
     * @param columnFamily
     * @throws IOException
     */
    public void deleteColumnFamily(String tableName, String rowKey, String columnFamily) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Delete delete = new Delete(rowKey.getBytes());
        delete.addFamily(Bytes.toBytes(columnFamily));
        table.delete(delete);
    }

    /**
     * delete 'tableName','rowKey','columnFamily:column'
     * @param tableName
     * @param rowKey
     * @param columnFamily
     * @param column
     * @throws IOException
     */
    public void deleteColumn(String tableName, String rowKey, String columnFamily, String column) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Delete delete = new Delete(rowKey.getBytes());
        delete.addColumn(Bytes.toBytes(columnFamily), Bytes.toBytes(column));
        table.delete(delete);
    }

    /**
     * disable 'tableName' 之后 drop 'tableName'
     * @param tableName
     * @throws IOException
     */
    public void deleteTable(String tableName) throws IOException {
        boolean isExists = this.tableExists(tableName);
        if (!isExists) {
            return;
        }
        TableName name = TableName.valueOf(tableName);
        admin.disableTable(name);
        admin.deleteTable(name);
    }

    /**
     * get 'tableName','rowkey','family:column'
     * @param tableName
     * @param rowkey
     * @param family
     * @param column
     * @return
     */
    public String getValue(String tableName, String rowkey, String family, String column) {
        Table table = null;
        String value = "";
        if (StringUtils.isBlank(tableName) || StringUtils.isBlank(family) || StringUtils.isBlank(rowkey) || StringUtils
                .isBlank(column)) {
            return null;
        }
        try {
            table = connection.getTable(TableName.valueOf(tableName));
            Get g = new Get(rowkey.getBytes());
            g.addColumn(family.getBytes(), column.getBytes());
            Result result = table.get(g);
            List<Cell> ceList = result.listCells();
            if (ceList != null && ceList.size() > 0) {
                for (Cell cell : ceList) {
                    value = Bytes.toString(cell.getValueArray(), cell.getValueOffset(), cell.getValueLength());
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                table.close();
                connection.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return value;
    }

    /**
     * get 'tableName','rowKey'
     * @param tableName
     * @param rowKey
     * @return
     * @throws IOException
     */
    public String selectOneRow(String tableName, String rowKey) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Get get = new Get(rowKey.getBytes());
        Result result = table.get(get);
        NavigableMap<byte[], NavigableMap<byte[], NavigableMap<Long, byte[]>>> map = result.getMap();
        for (Cell cell : result.rawCells()) {
            String row = Bytes.toString(cell.getRowArray());
            String columnFamily = Bytes.toString(cell.getFamilyArray());
            String column = Bytes.toString(cell.getQualifierArray());
            String value = Bytes.toString(cell.getValueArray());
            // 可以通过反射封装成对象(列名和Java属性保持一致)
            logger.info(row);
            logger.info(columnFamily);
            logger.info(column);
            logger.info(value);
        }
        return null;
    }

    /**
     * scan 't1',{FILTER=>"PrefixFilter('2015')"}
     * @param tableName
     * @param rowKeyFilter
     * @return
     * @throws IOException
     */
    public String scanTable(String tableName, String rowKeyFilter) throws IOException {
        Table table = connection.getTable(TableName.valueOf(tableName));
        Scan scan = new Scan();
        if (!StringUtils.isEmpty(rowKeyFilter)) {
            RowFilter rowFilter = new RowFilter(CompareOperator.EQUAL, new SubstringComparator(rowKeyFilter));
            scan.setFilter(rowFilter);
        }
        ResultScanner scanner = table.getScanner(scan);
        try {
            for (Result result : scanner) {
                logger.info(Bytes.toString(result.getRow()));
                for (Cell cell : result.rawCells()) {
                    logger.info(cell.toString());
                }
            }
        } finally {
            if (scanner != null) {
                scanner.close();
            }
        }
        return null;
    }


    /**
     * 判断表是否已经存在，这里使用间接的方式来实现
     *
     * admin.tableExists() 会报NoSuchColumnFamilyException， 有人说是hbase-client版本问题
     * @param tableName
     * @return
     * @throws IOException
     */
    public boolean tableExists(String tableName) throws IOException {
        TableName[] tableNames = admin.listTableNames();
        if (tableNames != null && tableNames.length > 0) {
            for (int i = 0; i < tableNames.length; i++) {
                if (tableName.equals(tableNames[i].getNameAsString())) {
                    return true;
                }
            }
        }
        return false;
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

  - 下载[winutils.exe](resource/bin/winutils.exe)
  - 新建环境变量

  ```
  HADOOP_HOME=wintuils.exe存放的地址
  
  配置path 
    %HADOOP_HOME%\bin
  ```
  
  - 重启电脑运行

