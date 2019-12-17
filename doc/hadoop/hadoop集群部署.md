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

| 服务器环境 | centos7         |                             |
| ---------- | --------------- | --------------------------- |
| Master     | 192.168.111.129 | NameNode、DataNode          |
| slaver1    | 192.168.111.128 | DataNode、ResourceManager   |
| slaver2    | 192.168.111.130 | DataNode、SecondaryNameNode |



### 1.1.2 下载并安装hadoop

```shell
# 下载
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
192.168.111.129 hadoop-master
192.168.111.128 hadoop-slave1
192.168.111.130 hadoop-slave2

# scp拷贝 hosts到其他服务器
[root@rocketmq-nameserver1 ~]# scp /etc/hosts root@hadoop-slave1:/etc/
[root@rocketmq-nameserver1 ~]# scp /etc/hosts root@hadoop-slave2:/etc/

#注意：修改hosts中，是立即生效的，无需source或者. 
```



创建hadoop组和用户

```shell
[root@rocketmq-nameserver1 ~]# groupadd hadoop 
[root@rocketmq-nameserver1 ~]# useradd -d /usr/hadoop -g hadoop -m hadoop
[root@rocketmq-nameserver1 ~]# passwd hadoop
```



### 1.1.4  在各节点上设置SSH无密码登录 

最终达到目的：即在master:节点执行 ssh hadoop@hadoop-slave1不需要密码，此处只需配置master访问slave1免密。

```shell
[hadoop@hadoop-master ~]$ su - hadoop
[hadoop@hadoop-master ~]$ pwd
/usr/hadoop
[hadoop@hadoop-master ~]$ cd .ssh
[hadoop@hadoop-master .ssh]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/usr/hadoop/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /usr/hadoop/.ssh/id_rsa.
Your public key has been saved in /usr/hadoop/.ssh/id_rsa.pub.
The key fingerprint is:
11:b2:23:8c:e7:32:1d:4c:2f:00:32:1a:15:43:bb:de hadoop@hadoop-master
The key's randomart image is:
+--[ RSA 2048]----+
|=+*.. . .        |
|oo O . o .       |
|. o B + .        |
|   = + . .       |
|  + o   S        |
| . +             |
|  . E            |
|                 |
|                 |
+-----------------+
[hadoop@hadoop-master .ssh]$ 
[hadoop@hadoop-master .ssh]$ cp id_rsa.pub authorized_keys
[hadoop@hadoop-master .ssh]$ ll
total 16
-rwx------. 1 hadoop hadoop 1230 Jan 31 23:27 authorized_keys
-rwx------. 1 hadoop hadoop 1675 Feb 23 19:07 id_rsa
-rwx------. 1 hadoop hadoop  402 Feb 23 19:07 id_rsa.pub
-rwx------. 1 hadoop hadoop  874 Feb 13 19:40 known_hosts
[hadoop@hadoop-master .ssh]$

```



### 1.1.5  **本机无密钥登录** 

```shell
[hadoop@hadoop-master ~]$ pwd
/usr/hadoop
[hadoop@hadoop-master ~]$ chmod -R 700 .ssh
[hadoop@hadoop-master ~]$ cd .ssh
[hadoop@hadoop-master .ssh]$ chmod 600 authorized_keys
[hadoop@hadoop-master .ssh]$ ll
total 16
-rwx------. 1 hadoop hadoop 1230 Jan 31 23:27 authorized_keys
-rwx------. 1 hadoop hadoop 1679 Jan 31 23:26 id_rsa
-rwx------. 1 hadoop hadoop  410 Jan 31 23:26 id_rsa.pub
-rwx------. 1 hadoop hadoop  874 Feb 13 19:40 known_hosts

# 验证
ssh hadoop@hadoop-master	
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
[hadoop@rocketmq-nameserver1 ~]$ scp /usr/hadoop/.ssh/authorized_keys hadoop@hadoop-slave1:/usr/hadoop/.ssh

[hadoop@rocketmq-nameserver1 ~]$ scp /usr/hadoop/.ssh/authorized_keys hadoop@hadoop-slave2:/usr/hadoop/.ssh

#然后在各个节点对authorized_keys执行(一定要执行该步，否则会报错)：chmod 600 authorized_keys
#保证.ssh 700，.ssh/authorized_keys 600权限

[root@localhost /]# cd usr/hadoop/
[root@localhost ~]# chmod -R 700 .ssh
[root@localhost hadoop]# cd .ssh/
[root@localhost .ssh]# chmod 600 authorized_keys

# 在hadoop-master分发公钥，分别分发给三台主机。
[root@rocketmq-nameserver1 hadoop]# ssh-copy-id hadoop-master
[root@rocketmq-nameserver1 hadoop]# ssh-copy-id hadoop-slave1
[root@rocketmq-nameserver1 hadoop]# ssh-copy-id hadoop-slave2
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
[root@rocketmq-nameserver2 hadoop]# vim hadoop-env.sh 
[root@rocketmq-nameserver2 hadoop]# vim mapred-env.sh 
[root@rocketmq-nameserver2 hadoop]# vim yarn-env.sh 

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
[root@rocketmq-nameserver2 hadoop]# cp mapred-site.xml.template mapred-site.xml
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



### 1.1.13 格式化NameNode

注意：

如果需要重新格式化NameNode,需要先将原来NameNode和DataNode下的文件全部删除，不然会报错，NameNode和DataNode所在目录是在core-site.xml中hadoop.tmp.dir、dfs.namenode.name.dir、dfs.datanode.data.dir属性配置的。

```shell
# 只能执行一次 相当于初始化操作
[root@rocketmq-nameserver1 data]# hdfs namenode -format
```



### 1.1.14 启动hdfs和yarn进程

```shell
#hadoop-master
[root@rocketmq-nameserver1 hadoop]# sbin/start-dfs.sh 

#hadoop-slave1
[root@rocketmq-nameserver1 hadoop]# sbin/start-yarn.sh
```



### 1.1.15 浏览器访问

#####  http://192.168.111.129:50070

#####  [http://192.168.111.128:8088](http://192.168.111.128:8088/) 

### 1.1.16 测试wordcount

```shell
# 创建目录
[root@rocketmq-nameserver1 hadoop]#  hdfs dfs -mkdir -p /wordcountdemo/input 

# 上传文件
[root@rocketmq-nameserver1 hadoop]# hdfs dfs -put /opt/1.txt /wordcountdemo/input
[root@rocketmq-nameserver1 hadoop]# hdfs dfs -put /opt/2.txt /wordcountdemo/input

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

