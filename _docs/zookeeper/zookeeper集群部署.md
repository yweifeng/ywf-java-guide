---
title: zookeeper集群部署
category: zookeeper
order: 1
---

#### 下载zookeeper安装包 [zookeeper-3.4.12.tar](https://links.jianshu.com/go?to=https%3A%2F%2Fpan.baidu.com%2Fs%2F1uvbXJvjqOetpUB8Y7aT05Q)

#### 解压并设置环境变量

```shell
# 解压
tar -zxvf zookeeper-3.4.12.tar

# 重命名
mv zookeeper-3.4.12 zookeeper

# 修改环境变量
vim etc/profile
#set zookeeper environment
export ZK_HOME=/opt/zookeeper
export PATH=$ZK_HOME/bin:$PATH

# 重启环境变量
source /etc/profile
```



### 设置主机名和配置hosts

| IP              | hostname |
| --------------- | -------- |
| 192.168.172.128 | master   |
| 192.168.172.129 | slave1   |
| 192.168.172.130 | slave2   |

- 修改etc/host文件
- 修改etc/hostname 修改机器名



#### 修改配置文件

```shell
# 拷贝zoo_sample.cfg 并重命名为zoo.cfg
cd opt/zookeeper/conf
cp zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg

# 修改配置文件
#修改数据文件夹路径
dataDir=/opt/zookeeper/data
#在文件末尾添加
server.1=master:2888:3888
server.2=slave1:2888:3888
server.3=slave2:2888:3888
```

#### 创建数据文件夹

```shell
cd opt/zookeeper
mkdir data
```

#### 创建myid文件

```shell
# 192.168.111.128
echo 1 >> /opt/zookeeper/data/myid
# 192.168.111.129
echo 2 >> /opt/zookeeper/data/myid
# 192.168.111.130
echo 3 >> /opt/zookeeper/data/myid
```

#### 启动zookeeper

```shell
# 每个机子单独启动zookeeper
[root@hadoop-slave1 zookeeper]# cd bin/
[root@hadoop-slave1 bin]# ./zkServer.sh start

# 查看启动结果
[root@hadoop-slave1 bin]# zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Error contacting service. It is probably not running.

# cat zookeeper.out分析原因
2019-11-25 10:41:24,690 [myid:1] - WARN  [QuorumPeer[myid=1]/0:0:0:0:0:0:0:0:2181:QuorumCnxManager@584] - Cannot open channel to 2 at election address /192.168.111.129:3888
java.net.ConnectException: 拒绝连接 (Connection refused)

#解决方法 
1、scp 拷贝 zookeeper到3台服务器。
2、修改myid。
3、scp 拷贝/etc/profile 到3太服务器。
4、source /etc/profile
5、三台服务器./zkServer.sh start启动  ./zkServer.sh status

#192.168.111.128
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Mode: follower

#192.168.111.129
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Mode: follower

#192.168.111.130
ZooKeeper JMX enabled by default
Using config: /opt/zookeeper/bin/../conf/zoo.cfg
Mode: leader
```