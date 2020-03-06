# kafka集群

## 集群部署

### 服务器环境

centos7 、zookeeper-3.4.12、kafka_2.11-2.0.0

| 主机名 | ip地址          | kafka broke.id | myid |
| ------ | --------------- | -------------- | ---- |
| master | 192.168.111.140 | 1              | 1    |
| slave1 | 192.168.111.141 | 2              | 2    |
| slave2 | 192.168.111.142 | 3              | 3    |

**前置部署**

- [zookeeper集群部署手册](https://yweifeng.github.io/ywf-java-guide/doc/zookeeper/zookeeper集群部署.html)



### kafka集群搭建

#### 下载 kafka安装包   [kafka_2.11-2.0.0 .tgz](https://links.jianshu.com/go?to=https%3A%2F%2Fpan.baidu.com%2Fs%2F1Flc6qthv6p1Dqq7mEISyIA)



#### 解压并设置环境变量

```shell
# 解压
tar -zxvf kafka_2.11-2.0.0 .tgz

# 重命名
mv kafka_2.11-2.0.0 kafka

# 设置环境变量
vim /etc/profile
export KAFKA_HOME=/opt/kafka
PATH=${KAFKA_HOME}/bin:$PATH

# 重启环境变量
source /etc/profile

# 修改hosts
source /etc/hosts
192.168.111.140 master
192.168.111.141 slave1
192.168.111.142 slave2
```

#### 修改配置文件 server.properties 

```shell
vim /opt/kafka/config/server.properties
# 修改broker.id
broker.id=1
# 分別修改 listeners    listeners=PLAINTEXT://主机名:9092
listeners=PLAINTEXT://master:9092

# 修改 zookeeper连接地址
zookeeper.connect=master:2181,slave1:2181,slave2:2181
```

#### 拷贝到其他服务器

```shell
# 拷贝kafka到slave1和slave2
scp -r kafka root@slave1:/opt
# 拷贝/etc/hosts到slave1和slave2
scp hosts root@slave1:/etc
# 拷贝/etc/profile到slave1和slave2
scp /etc/profile root@slave1:/etc
# 重启环境变量
source /etc/profile
# 修改server.properties broker.id
master: broker.id=1
slave1: broker.id=2
slave2: broker.id=3
```

#### 启动kafka

```shell
# 各服务器单独启动kafka
bin/kafka-server-start.sh -daemon config/server.properties 
```

#### 创建并查看topic

```shell
# 选择一台服务器创建topic  topic命名不要用_   partition不要用1个
kafka-topics.sh --create --zookeeper master:2181,slave1:2181,slave2:2181 --replication-factor 3 --partitions 3 --topic topic-ywf

# 查看topic
kafka-topics.sh --describe --zookeeper master:2181,slave1:2181,slave2:2181 --topic topic-ywf
```



### kafka监控搭建

#### 安装[kafka Tool](http://www.kafkatool.com/download2/kafkatool_64bit.exe)

