# hbase集群部署

### 环境配置

| IP地址          | hosts                 |
| --------------- | --------------------- |
| 192.168.111.128 | node1、 hadoop-slave1 |
| 192.168.111.129 | node2、 hadoop-master |
| 192.168.111.130 | node3、 hadoop-slave2 |

### 版本信息

| 软件信息  | 版本      |
| --------- | --------- |
| 操作系统  | centos7.0 |
| jdk       | 1.8       |
| zookeeper | 3.4.12    |
| hbase     | 2.2.2     |

- [zookeeper集群部署手册]([https://yweifeng.github.io/ywf-java-guide/doc/zookeeper/zookeeper%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2.html](https://yweifeng.github.io/ywf-java-guide/doc/zookeeper/zookeeper集群部署.html))

- [hadoop集群部署手册]([https://yweifeng.github.io/ywf-java-guide/doc/hadoop/hadoop%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2.html](https://yweifeng.github.io/ywf-java-guide/doc/hadoop/hadoop集群部署.html))



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



### 修改hosts

```shell
vim /etc/hosts
# 添加以下信息
192.168.111.128 node1
192.168.111.129 node2
192.168.111.130 node3

192.168.111.129 hadoop-master
192.168.111.128 hadoop-slave1
192.168.111.130 hadoop-slave2
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
    <value>hdfs://hadoop-master:9000/hbase</value>
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

- 拷贝hadoop的配置文件hdfs-site.xml 到hbase的配置文件目录

```shell
cp /opt/hadoop/etc/hadoop/hdfs-site.xml /opt/hbase/conf/
```



### 拷贝到其他服务器

```shell
# 拷贝hbase
scp -r /opt/hbase root@node2:/opt/
scp -r /opt/hbase root@node3:/opt/

# 拷贝/etc/hosts
scp /etc/hosts root@node2:/etc/
scp /etc/hosts root@node3:/etc/

# 拷贝/etc/profile
scp /etc/profile root@node2:/etc/
scp /etc/profile root@node3:/etc/
	
# 每个服务器重启/etc/profile
source /etc/profile
```



### 保持集群服务器时间同步

```shell
# 3台服务器都执行
ntpdate -u 0.uk.pool.ntp.org
```



### 启动

- 启动zookeeper集群

```shell
cd /opt/zookeeper/bin
./zkServer.sh start
```



- 启动dfs集群

```shell
cd /opt/hadoop/sbin
./start-dfs.sh
```



- 启动hbase集群

```shell
cd /opt/hbase/bin
./start-hbase.sh
```

