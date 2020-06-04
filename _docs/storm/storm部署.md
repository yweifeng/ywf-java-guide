# Storm集群部署

> 环境介绍 jdk1.8、storm1.2.3 、zookeeper3.4.12 、centos7

| 服务器          | hosts       | storm角色          |
| --------------- | ----------- | ------------------ |
| 192.168.111.128 | storm1、zk1 | nimbus、supervisor |
| 192.168.111.129 | storm2、zk2 | nimbus、supervisor |
| 192.168.111.130 | storm3、zk3 | nimbus、supervisor |



## 配置hosts

```shell
vim /etc/hosts

192.168.111.128 storm1
192.168.111.129 storm2
192.168.111.130 storm3

192.168.111.128 zk1
192.168.111.129 zk2
192.168.111.130 zk3
```



## 下载安装

下载地址：[http://storm.apache.org/downloads.html](http://storm.apache.org/downloads.html)

```shell
cd /opt
# 解压缩
tar -zxvf apache-storm-1.2.3.tar.gz

# 重命名
mv apache-storm-1.2.3 storm

cd /opt/storm

mkdir storm-stage
```



## 修改配置

```shell
vim conf/storm.yml

#storm集群连接zookeeper集群的地址	
storm.zookeeper.servers:
     - "zk1"
     - "zk2"
     - "zk3"
# storm集群的slave节点列表
nimbus.seeds: ["storm1", "storm2", "storm3"]
# storm集群数据本地存放目录
storm.local.dir: "/opt/storm/storm-stage"
# worker jvm进程端口
supervisor.slots.ports:
     - 6700
     - 6701
     - 6702
     - 6703
```



## 安装storm python的脚本服务

```shell
yum install -y python-argparse
```



## 配置环境变量

```shell
vim /etc/profile

STORM_HOME=/opt/storm
PATH=$STORM_HOME/bin:$PATH
export PATH

# 重启配置
source etc/profile
```



## scp到其他服务器

```shell
# 拷贝storm
scp -r /opt/storm root@192.168.111.129:/opt
scp -r /opt/storm root@192.168.111.130:/opt

# 拷贝hosts
scp /etc/hosts root@192.168.111.129:/etc
scp /etc/hosts root@192.168.111.130:/etc

# 拷贝profile
scp /etc/profile root@192.168.111.129:/etc
scp /etc/profile root@192.168.111.130:/etc

# 登录192.168.111.129和192.168.111.130重启配置
source /etc/profile
```



## 启动storm

### 启动nimbus

```shell
# 192.168.111.128、192.168.111.129、192.168.111.130
storm nimbus &
```

### 启动supservisor

```shell
# 192.168.111.128、192.168.111.129、192.168.111.130
storm supervisor &
```

### 启动storm ui

```shell
# 192.168.111.128
storm ui &
```



## 浏览器访问

访问地址：[192.168.111.128:8080](192.168.111.128:8080)

