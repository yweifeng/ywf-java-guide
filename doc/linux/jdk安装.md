# jdk安装

## 1.1  检查一下系统中的jdk版本 

```shell
[root@rocketmq-nameserver1 local]# clear
[root@rocketmq-nameserver1 local]# java -version
openjdk version "1.8.0_232"
OpenJDK Runtime Environment (build 1.8.0_232-b09)
OpenJDK 64-Bit Server VM (build 25.232-b09, mixed mode)
```



## 1.2  检测jdk安装包 

```shell
[root@rocketmq-nameserver1 local]# rpm -qa | grep java
tzdata-java-2018c-1.el7.noarch
python-javapackages-3.4.1-11.el7.noarch
java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64
java-1.8.0-openjdk-devel-1.8.0.232.b09-0.el7_7.x86_64
javapackages-tools-3.4.1-11.el7.noarch
java-1.8.0-openjdk-headless-1.8.0.232.b09-0.el7_7.x86_64
```

## 1.3  卸载openjdk 

```shell
rpm -e --nodeps tzdata-java-2018c-1.el7.noarch
rpm -e --nodeps java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64
rpm -e --nodeps java-1.8.0-openjdk-devel-1.8.0.232.b09-0.el7_7.x86_64
rpm -e --nodeps javapackages-tools-3.4.1-11.el7.noarch
rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.232.b09-0.el7_7.x86_64

#或者
yum remove *openjdk*
```



## 1.4 安装新的jdk

```shell
# 方式一：
访问：https://www.oracle.com/java/technologies/javase-jdk8-downloads.html

#方式二：下载jdk（比较慢）
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz"

#解压

```



## 1.5 配置环境变量

vim /etc/profile

```bash
JAVA_HOME=/opt/jdk/jdk1.8.0_141
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
```



## 1.6 重启配置信息并校验

```shell
source /etc/profile
java -version
```

