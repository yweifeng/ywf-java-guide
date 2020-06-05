---
title: mycat 分库分表
category: mysql
order: 4
---



### 配置schema.xml

```xml

<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

        <schema name="mycat_ywf" checkSQLschema="false" sqlMaxLimit="100">
                <table name="user" dataNode="dn1,dn2" primaryKey="unid" rule="ywf_rule"></table>
        </schema>
        <dataNode name="dn1" dataHost="localhost1" database="ywf" />
        <dataNode name="dn2" dataHost="localhost2" database="ywf" />
        <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
                          writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
                <writeHost host="hostM1" url="192.168.111.129:3306" user="root"
                                   password="!QAZ2wsx">
                        <!-- can have multi read hosts -->
                        <readHost host="hostS2" url="192.168.111.128:3306" user="root" password="!QAZ2wsx" />
                </writeHost>
        </dataHost>
        <dataHost name="localhost2" maxCon="1000" minCon="10" balance="0"
                          writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
                <writeHost host="hostS1" url="192.168.111.128:3306" user="root"
                                   password="!QAZ2wsx">
                </writeHost>
        </dataHost>

</mycat:schema>

```



### 配置rule.xml

```xml
<!--添加rule规则-->
<tableRule name="ywf_rule">
    <rule>
        <columns>unid</columns>
        <algorithm>mod-long</algorithm>
    </rule>
</tableRule>

<!-- 修改集群节点数量 -->
<function name="mod-long" class="io.mycat.route.function.PartitionByMod">
    <!-- how many data nodes -->
    <property name="count">2</property>
</function>
```



### 配置server.xml

```xml
#修改sequnceHandlerType为0  读取sequence_conf.properties
 <property name="sequnceHandlerType">0</property>
```



### 验证

```shell
#登录管理窗口
[root@localhost bin]# mysql -uroot -p -h127.0.0.1 -P3310 mycat_ywf;

#插入数据模拟
insert into user(unid,name) values(next value for MYCATSEQ_GLOBAL,'ywf01');
insert into user(unid,name) values(next value for MYCATSEQ_GLOBAL,'ywf02');
insert into user(unid,name) values(next value for MYCATSEQ_GLOBAL,'ywf03');
insert into user(unid,name) values(next value for MYCATSEQ_GLOBAL,'ywf04');

```

