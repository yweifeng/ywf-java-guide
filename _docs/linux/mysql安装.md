---
title: mysql安装
category: linux
order: 3
---



```shell
#下载并安装mysql yum

wget https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm

rpm -ivh mysql57-community-release-el7-9.noarch.rpm

yum install mysql-server

#mysql配置
#启动mysql 
[root@localhost /]# systemctl start mysqld.service
#查看mysql状态
[root@localhost /]# systemctl status mysqld.service
#查找root密码 
[root@localhost /]# grep "password" /var/log/mysqld.log 
2019-11-13T15:27:05.345051Z 1 [Note] A temporary password is generated for root@localhost: hoe8rTf!50u.
#登录mysql修改密码
[root@localhost /]# mysql -uroot -p
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '!QAZ2wsx';

#密码太简单策略修改
mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | OFF    |
| validate_password_dictionary_file    |        |
| validate_password_length             | 8      |
| validate_password_mixed_case_count   | 1      |
| validate_password_number_count       | 1      |
| validate_password_policy             | MEDIUM |
| validate_password_special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.00 sec)


#修改密码策略
mysql> set global validate_password_policy=0;
mysql> set global validate_password_length=1;

#设置远程访问权限
mysql> grant all privileges on *.* to 'root' @'%' identified by '!QAZ2wsx';

#卸载yum自动更新
[root@localhost /]# yum -y remove mysql57-community-release-el7-9.noarch
```



### 解决centos7使用yum安装mysql 下载速度慢的问题

- 备份系统自带yum源配置文件/etc/yum.repos.d/CentOS-Base.repo

```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

- 下载ailiyun的yum源配置文件到/etc/yum.repos.d/

```shell
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

- 运行yum makecache生成缓存

```shell
yum makecache
```

- 这时候再更新系统就会看到以下mirrors.aliyun.com信息

```shell
yum -y update
```

