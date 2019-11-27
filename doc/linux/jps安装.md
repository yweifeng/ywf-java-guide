### 安装jps

```shell
[root@namenode ~]# jps
bash: jps: command not found...
[root@namenode ~]# find / -name jps
find: ‘/run/user/1001/gvfs’: Permission denied
[root@namenode ~]# rpm -qa |grep -i jdk
java-1.8.0-openjdk-headless-1.8.0.65-3.b17.el7.x86_64
java-1.8.0-openjdk-1.8.0.65-3.b17.el7.x86_64
java-1.7.0-openjdk-1.7.0.91-2.6.2.3.el7.x86_64
java-1.7.0-openjdk-headless-1.7.0.91-2.6.2.3.el7.x86_64

[root@namenode ~]# yum list *openjdk-devel*

需要安装openjdk-devel包
[root@namenode ~]# yum install java-1.8.0-openjdk-devel.x86_64
[root@namenode ~]# which jps
/usr/bin/jps

[root@namenode ~]# jps
12995 Jps
10985 ResourceManager
11179 NodeManager
10061 NameNode
10301 DataNode
10655 SecondaryNameNode
————————————————
版权声明：本文为CSDN博主「Leshami」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/leshami/article/details/78562642
```

