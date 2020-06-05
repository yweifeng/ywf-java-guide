---
title: mycat 读写分离
category: mysql
order: 3
---



### 下载和安装mycat

```shell
#下载mycat
http://dl.mycat.io/1.6.5/ 
选择Mycat-server-1.6.5-release-20180122220033-linux.tar.gz

#解压
```



### 修改schema.xml 和 server.xml

schema.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

        <schema name="mycat_ywf" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">
        </schema>
        <dataNode name="dn1" dataHost="localhost1" database="ywf" />
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
</mycat:schema>
```

server.xml

```xml
<user name="root" defaultAccount="true">
    <property name="password">!QAZ2wsx</property>
    <property name="schemas">mycat_ywf</property>
</user>
```



### 修改log级别

```
进入mycat/conf
 vim log4j2.xml
 <asyncRoot level="info" includeLocation="true"> 
 改为   
 <asyncRoot level="trace" includeLocation="true">
```



### 启动mycat

```shell
[root@localhost bin]# ./mycat start
# 查看日志
./mycat console
```



### 验证

```shell
[root@localhost bin]# mysql -uroot -p -h127.0.0.1 -P3310 mycat_ywf;
mysql> show databases;
+-----------+
| DATABASE  |
+-----------+
| mycat_ywf |
+-----------+
1 row in set (0.00 sec)

```

