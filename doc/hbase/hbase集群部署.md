# hbase集群部署

### 环境配置

| 描述    | IP地址          | hosts |
| ------- | --------------- | ----- |
| master  | 192.168.111.128 | node1 |
| slaver1 | 192.168.111.129 | node2 |
| slaver2 | 192.168.111.130 | node3 |

| 软件信息  | 版本      |
| --------- | --------- |
| 操作系统  | centos7.0 |
| jdk       | 1.8       |
| zookeeper | 3.4.12    |
| hbase     | 2.2.2     |

- [zookeeper集群部署手册]([https://yweifeng.github.io/ywf-java-guide/doc/zookeeper/zookeeper%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2.html](https://yweifeng.github.io/ywf-java-guide/doc/zookeeper/zookeeper集群部署.html))



### 下载并安装

- [下载地址](https://hbase.apache.org/downloads.html)

```shell
# 解压
tar -zxvf hbase-2.2.2-bin.tar.gz
# 重命名
mv hbase-2.2.2 hbase
# 设置环境变量
vim /etc/profile

export HBASE_HOME=/opt/hbase
export PATH=${HBASE_HOME}/bin:$PATH
# 重启生效环境变量
source /etc/profile
```



### 修改配置信息

- 修改**reigionservers**

```shell
cd /opt/hbase/conf
vim regionservers

node1
node2
node3
```

- 修改 **hbase-site.xml**

```xml
<configuration>
  <property>
    <name>hbase.rootdir</name>
    <value>hdfs://master:9000/hbase</value>
  </property>
  <property>
     <name>hbase.cluster.distributed</name>
     <value>true</value>
  </property>
  <property>
     <name>hbase.master.port</name>
     <value>16000</value>
  </property>
   <property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/opt/zookeeper/data</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>node1,node2,node3</value>
  </property>
  <property>
    <name>hbase.zookeeper.property.clientPort</name>
    <value>2181</value>
  </property>
</configuration>
```

- 修改**hbase-env.sh**

```properties
# HBASE_MANAGES_ZK=false 表示，hbase和大家公用一个zookeeper集群，而不是自己管理集群。
export HBASE_MANAGES_ZK=false
```

