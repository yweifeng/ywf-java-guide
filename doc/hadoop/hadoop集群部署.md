# 1.hadoop快速入门

```
http://hadoop.apache.org/docs/r1.0.4/cn/quickstart.html
```

#节点目录

HDFS daemon：  NameNode, SecondaryNameNode, DataNode

YARN damones： ResourceManager, NodeManager, WebAppProxy

MapReduce Job History Server



## 1.1 集群部署搭建

### 1.1.1 环境配置

| host    | IP              | 角色                                   |
| ------- | --------------- | -------------------------------------- |
| master  | 192.168.111.128 | NameNode、DataNode                     |
| slaver1 | 192.168.111.129 | DataNode、ResourceManager、NodeManager |
| slaver2 | 192.168.111.130 | DataNode、SecondaryNameNode            |



### 1.1.2 下载并安装hadoop

```shell
# 方式一:
下载地址: https://hadoop.apache.org/releases.html

# 方式二：下载
wget http://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.9.2/hadoop-2.9.2.tar.gz

# 解压
tar -zxvf hadoop-2.9.2.tar.gz 

# 重命名
mv hadoop-2.9.2 hadoop

# 拷贝到其他服务器
scp -r hadoop root@192.168.111.128:/opt
scp -r hadoop root@192.168.111.130:/opt

```



### 1.1.3  **在各节点上设置主机名及创建hadoop组和用户** 

配置hadoop-master、hadoop-slave1、hadoop-slave2 hosts文件

```shell
[root@hadoop-master ~]#  vi /etc/hosts
192.168.111.128 hadoop-master
192.168.111.129 hadoop-slave1
192.168.111.130 hadoop-slave2

# scp拷贝 hosts到其他服务器
[root@hadoop-master ~]# scp /etc/hosts root@hadoop-slave1:/etc/
[root@hadoop-master ~]# scp /etc/hosts root@hadoop-slave2:/etc/

#注意：修改hosts中，是立即生效的，无需source或者. 
```



创建hadoop组和用户

```shell
[root@hadoop-master ~]# groupadd hadoop 
[root@hadoop-master ~]# useradd -d /usr/hadoop -g hadoop -m hadoop
[root@hadoop-master ~]# passwd hadoop

# 设置权限
[hadoop@hadoop-master ~]$ su root
密码：
[root@hadoop-master ~]# chown -R hadoop:hadoop /opt/*
```



### 1.1.4  在各节点上设置SSH无密码登录 

```shell
[hadoop@hadoop-master ~]$ su hadoop
[hadoop@hadoop-master ~]$ pwd
/usr/hadoop
[hadoop@hadoop-master ~]$ ssh-keygen -t rsa -P ""
# 查看生成的文件
[hadoop@hadoop-master ~]$ cd .ssh
[hadoop@hadoop-master .ssh]$ ls
authorized_keys  id_rsa  id_rsa.pub  known_hosts
```



### 1.1.5  **本机无密钥登录** 

```shell
[hadoop@hadoop-master .ssh]$ cat id_rsa.pub >> authorized_keys
[hadoop@hadoop-master .ssh]$ cd ~
[hadoop@hadoop-master ~]$ chmod 700 .ssh
[hadoop@hadoop-master ~]$ chmod 600 .ssh/*

# 验证
[hadoop@hadoop-master .ssh]$ ssh hadoop-master
Last login: Wed Dec 18 19:21:28 2019 from hadoop-master

```



### 1.1.6  master与其他节点无密钥登录

```shell
#执行cd ~.ssh发现ssh目录找不到
这是因为没有用root用户ssh登录过，执行一下ssh操作就会自动生成这个文件夹了
执行 ssh 主机名
然后按照步骤输入yes  密码就可以了
```



```shell
#从master中把authorized_keys分发到各个结点上
[hadoop@hadoop-master ~]$ scp /usr/hadoop/.ssh/authorized_keys hadoop@hadoop-slave1:/usr/hadoop/.ssh

[hadoop@hadoop-master ~]$ scp /usr/hadoop/.ssh/authorized_keys hadoop@hadoop-slave2:/usr/hadoop/.ssh

#然后在各个节点对authorized_keys执行(一定要执行该步，否则会报错)：chmod 600 authorized_keys
#保证.ssh 700，.ssh/authorized_keys 600权限

[hadoop@localhost ~]# chmod -R 700 .ssh
[hadoop@localhost ~]# cd .ssh
[hadoop@localhost .ssh]# chmod 600 authorized_keys

# 在hadoop-master分发公钥，分别分发给三台主机。
[hadoop@hadoop-master ~]# ssh-copy-id hadoop-master
[hadoop@hadoop-master ~]# ssh-copy-id hadoop-slave1
[hadoop@hadoop-master ~]# ssh-copy-id hadoop-slave2
```



### 1.1.7 设置hadoop环境变量

```shell
#修改配置文件
vim /etc/profile

#set hadoop
HADOOP_HOME=/opt/hadoop
PATH=$JAVA_HOME/bin:$PATH/bin:$HADOOP_HOME/bin
export JAVA_HOME  CLASSPATH PATH HADOOP_HOME

#重启配置
source /etc/profile

#校验
hadoop version
```



### 1.1.8 修改hadoop-env.sh 、mapred-env.sh 、yarn-env.sh  设置JAVA_HOME地址

```shell
[root@hadoop-master hadoop]# vim hadoop-env.sh 
[root@hadoop-master hadoop]# vim mapred-env.sh 
[root@hadoop-master hadoop]# vim yarn-env.sh 
```



### 1.1.9  配置core-site.xml  

fs.defaultFS为NameNode的地址。

hadoop.tmp.dir为hadoop临时目录的地址，默认情况下，NameNode和DataNode的数据文件都会存在这个目录下的对应子目录下。应该保证此目录是存在的，如果不存在，先创建。

```xml
<configuration>
 <property>
   <name>fs.defaultFS</name>
   <value>hdfs://hadoop-master:8020</value>
 </property>
 <property>
   <name>hadoop.tmp.dir</name>
   <value>/opt/hadoop/data/tmp</value>
 </property>
</configuration>
```



### 1.1.10  配置hdfs-site.xml 

 dfs.namenode.secondary.http-address是指定secondaryNameNode的http访问地址和端口号，因为在规划中，我们将hadoop-slave2规划为SecondaryNameNode服务器。 

```xml
<configuration>
<property>
   <name>dfs.namenode.secondary.http-address</name>
   <value>hadoop-slave2:50090</value>
 </property>
</configuration>
```



### 1.1.11  配置yarn-site.xml 

根据规划yarn.resourcemanager.hostname这个指定resourcemanager服务器指向hadoop-slave1。

yarn.log-aggregation-enable是配置是否启用日志聚集功能。

yarn.log-aggregation.retain-seconds是配置聚集的日志在HDFS上最多保存多长时间。

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop-slave1</value>
    </property>
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
    <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>106800</value>
    </property>
</configuration>
```



### 1.1.12  配置mapred-site.xml 

复制mapred-site.xml.template 为  mapred-site.xml

```shell
[root@hadoop-master hadoop]# cp mapred-site.xml.template mapred-site.xml
```

配置mapred-site.xml

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>hadoop-slave2:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>hadoop-slave2:19888</value>
    </property>
</configuration>
```

### 1.1.13 配置slaves (master节点)

```bash
hadoop-master
hadoop-slave1
hadoop-slave2
```



### 1.1.14 格式化NameNode

注意：只在master节点使用

如果需要重新格式化NameNode,需要先将原来NameNode和DataNode下的文件全部删除，不然会报错，NameNode和DataNode所在目录是在core-site.xml中hadoop.tmp.dir、dfs.namenode.name.dir、dfs.datanode.data.dir属性配置的。

```shell
# 只能执行一次 相当于初始化操作
[root@hadoop-master data]# hdfs namenode -format
```



### 1.1.15 启动hdfs和yarn进程

```shell
#hadoop-master
[root@hadoop-master hadoop]# sbin/start-dfs.sh 

#hadoop-slave1
[root@hadoop-master hadoop]# sbin/start-yarn.sh
```



### 1.1.16 浏览器访问

#####  [http://192.168.111.128:50070](http://192.168.111.128:50070)

#####  [http://192.168.111.129:8088](http://192.168.111.129:8088/) 

### 1.1.17 测试wordcount

```shell
# 创建目录
[root@hadoop-master hadoop]#  hdfs dfs -mkdir -p /wordcountdemo/input 

# 上传文件
[root@hadoop-master hadoop]# hdfs dfs -put /opt/1.txt /wordcountdemo/input
[root@hadoop-master hadoop]# hdfs dfs -put /opt/2.txt /wordcountdemo/input

#运行WordCount MapReduce Job

```



## 2.1 java 调用hdfs

### 2.1.1 建立maven项目，引入依赖

```xml
<properties>
    <hadoop.version>2.9.2</hadoop.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
        <artifactId>hadoop-client</artifactId>
        <version>${hadoop.version}</version>
    </dependency>
</dependencies>
```



### 2.1.2拷贝core-site.xml、hdfs-site.xml和log4j.properties到resources



### 2.1.3运行出现 以下异常解决方法

```java
java.io.FileNotFoundException: java.io.FileNotFoundException: HADOOP_HOME and hadoop.home.dir are unset. 
```



```
git 下载 winuntil
https://github.com/steveloughran/winutils
```

