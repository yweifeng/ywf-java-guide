---
title: redis企业级数据备份方案
category: redis
order: 7
---



### 企业级的持久化的配置策略

> 在实际生产环境，RDB 和 AOF 一定都要打开，RDB 和 AOF 的配置需要根据业务的数据量决定



### 企业级的数据备份方案

- 写 crontab 定时调度脚本做数据备份
- 每小时都 copy 一份 rdb 的备份，到一个目录中去，仅仅保留最近48小时的备份
- 每天都保留一份当日的 rdb 的备份，到一个目录中去，仅仅保留最近1个月的备份
- 每次 copy 备份的时候，都把太旧的备份给删了
- 每天晚上将当前服务器上所有的数据备份，发送一份到远程的云服务上去



#### 按小时备份

redis_rdb_copy_hourly.sh

```shell
#!/bin/sh 
cur_date=`date +%Y%m%d%k`
rm -rf /usr/local/redis/snapshotting/$cur_date
mkdir /usr/local/redis/snapshotting/$cur_date
cp /var/redis/6379/dump.rdb /usr/local/redis/snapshotting/$cur_date
del_date=`date -d -48hour +%Y%m%d%k`
rm -rf /usr/local/redis/snapshotting/$del_date
```



每小时 copy 一次备份，删除48小时前的数据。

```shell
crontab -e
```

```shell
0 * * * * sh /usr/local/redis/copy/redis_rdb_copy_hourly.sh
```



#### 按天备份

redis_rdb_copy_daily.sh

```shell
#!/bin/sh 
cur_date=`date +%Y%m%d`
rm -rf /usr/local/redis/snapshotting/$cur_date
mkdir /usr/local/redis/snapshotting/$cur_date
cp /var/redis/6379/dump.rdb /usr/local/redis/snapshotting/$cur_date
del_date=`date -d -1month +%Y%m%d`
rm -rf /usr/local/redis/snapshotting/$del_date
```



每天 copy 一次备份，删除一个月前的数据。

```shell
crontab -e
0 * * * * sh /usr/local/redis/copy/redis_rdb_copy_hourly.sh
0 0 * * * sh /usr/local/redis/copy/redis_rdb_copy_daily.sh
```



### 数据恢复方案

1. 如果是 Redis 进程挂掉，那么重启 Redis 进程即可，直接基于 AOF 日志文件恢复数据；
2. 如果是 Redis 进程所在机器挂掉，那么重启机器后，尝试重启 Redis 进程，尝试直接基于 AOF 日志文件进行数据恢复；
3. 如果 Redis 当前最新的 AOF 和 RDB 文件出现了丢失/损坏，那么可以尝试基于该机器上当前的某个最新的 RDB 数据副本进行数据恢复；



恢复步骤参考如下：

- 停止 Redis
- 在 Redis 配置文件中关闭 AOF 持久化配置
- 拷贝云服务上最新的 RDB 备份数据到 /var/redis/6379 文件夹下
- 重启 Redis，确认数据恢复
- 直接在命令行热修改 Redis 配置，config set appendonly yes
- 确认在 /var/redis/6379 文件夹下生成 AOF 持久化文件 appendonly.aof
- 停止 Redis
- 在 Redis 配置文件中打开 AOF 持久化配置
- 重启 Redis，确认数据情况



1. 如果当前机器上的所有RDB文件全部损坏，那么从远程的云服务上拉取最新的RDB快照回来恢复数据
2. 如果是发现有重大的数据错误，比如某个小时上线的程序一下子将数据全部污染了，数据全错了，那么可以选择某个更早的时间点，对数据进行恢复



举个例子，12点上线了代码，发现代码有 bug，导致代码生成的所有的缓存数据全部错了，找到一份11点的 rdb 的冷备，然后按照上面的步骤，去恢复到11点的数据，就可以了。