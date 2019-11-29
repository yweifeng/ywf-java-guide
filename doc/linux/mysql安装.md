# mysql安装

```shell
#下载并安装mysql yum

wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm

yum -y install mysql57-community-release-el7-10.noarch.rpm

yum -y install mysql-community-server

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
[root@localhost /]# yum -y remove mysql57-community-release-el7-10.noarch
```
