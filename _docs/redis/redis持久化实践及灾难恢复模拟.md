---
title: redis持久化实践及灾难恢复模拟
category: redis
order: 7
---



### 对Redis持久化的探讨与理解

目前Redis持久化的方式有两种： RDB 和 AOF

首先，我们应该明确持久化的数据有什么用，答案是用于重启后的数据恢复。
Redis是一个内存数据库，无论是RDB还是AOF，都只是其保证数据恢复的措施。
所以Redis在利用RDB和AOF进行恢复的时候，都会读取RDB或AOF文件，重新加载到内存中。

RDB就是Snapshot快照存储，是默认的持久化方式。
可理解为半持久化模式，即按照一定的策略周期性的将数据保存到磁盘。
对应产生的数据文件为dump.rdb，通过配置文件中的save参数来定义快照的周期。
下面是默认的快照设置：

```
save 900 1    #当有一条Keys数据被改变时，900秒刷新到Disk一次
save 300 10   #当有10条Keys数据被改变时，300秒刷新到Disk一次
save 60 10000 #当有10000条Keys数据被改变时，60秒刷新到Disk一次
```

Redis的RDB文件不会坏掉，因为其写操作是在一个新进程中进行的。
当生成一个新的RDB文件时，Redis生成的子进程会先将数据写到一个临时文件中，然后通过原子性rename系统调用将临时文件重命名为RDB文件。
这样在任何时候出现故障，Redis的RDB文件都总是可用的。

同时，Redis的RDB文件也是Redis主从同步内部实现中的一环。
第一次Slave向Master同步的实现是：
Slave向Master发出同步请求，Master先dump出rdb文件，然后将rdb文件全量传输给slave，然后Master把缓存的命令转发给Slave，初次同步完成。
第二次以及以后的同步实现是：
Master将变量的快照直接实时依次发送给各个Slave。
但不管什么原因导致Slave和Master断开重连都会重复以上两个步骤的过程。
Redis的主从复制是建立在内存快照的持久化基础上的，只要有Slave就一定会有内存快照发生。

可以很明显的看到，RDB有它的不足，就是一旦数据库出现问题，那么我们的RDB文件中保存的数据并不是全新的。
从上次RDB文件生成到Redis停机这段时间的数据全部丢掉了。

AOF(Append-Only File)比RDB方式有更好的持久化性。
由于在使用AOF持久化方式时，Redis会将每一个收到的写命令都通过Write函数追加到文件中，类似于[MySQL](http://lib.csdn.net/base/14)的binlog。
当Redis重启是会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。
对应的设置参数为：
$ vim /opt/redis/etc/redis_6379.conf

```
appendonly yes       #启用AOF持久化方式
appendfilename appendonly.aof #AOF文件的名称，默认为appendonly.aof
# appendfsync always #每次收到写命令就立即强制写入磁盘，是最有保证的完全的持久化，但速度也是最慢的，一般不推荐使用。
appendfsync everysec #每秒钟强制写入磁盘一次，在性能和持久化方面做了很好的折中，是受推荐的方式。
# appendfsync no     #完全依赖OS的写入，一般为30秒左右一次，性能最好但是持久化最没有保证，不被推荐。
```

AOF的完全持久化方式同时也带来了另一个问题，持久化文件会变得越来越大。
比如我们调用INCR test命令100次，文件中就必须保存全部的100条命令，但其实99条都是多余的。
因为要恢复数据库的状态其实文件中保存一条SET test 100就够了。
为了压缩AOF的持久化文件，Redis提供了bgrewriteaof命令。
收到此命令后Redis将使用与快照类似的方式将内存中的数据以命令的方式保存到临时文件中，最后替换原来的文件，以此来实现控制AOF文件的增长。
由于是模拟快照的过程，因此在重写AOF文件时并没有读取旧的AOF文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的AOF文件。
对应的设置参数为:
$ vim /opt/redis/etc/redis_6379.conf

```
no-appendfsync-on-rewrite yes   #在日志重写时，不进行命令追加操作，而只是将其放在缓冲区里，避免与命令的追加造成DISK IO上的冲突。
auto-aof-rewrite-percentage 100 #当前AOF文件大小是上次日志重写得到AOF文件大小的二倍时，自动启动新的日志重写过程。
auto-aof-rewrite-min-size 64mb  #当前AOF文件启动新的日志重写过程的最小值，避免刚刚启动Reids时由于文件尺寸较小导致频繁的重写。
```

到底选择什么呢？下面是来自官方的建议：
通常，如果你要想提供很高的数据保障性，那么建议你同时使用两种持久化方式。
如果你可以接受灾难带来的几分钟的数据丢失，那么你可以仅使用RDB。
很多用户仅使用了AOF，但是我们建议，既然RDB可以时不时的给数据做个完整的快照，并且提供更快的重启，所以最好还是也使用RDB。
因此，我们希望可以在未来（长远计划）统一AOF和RDB成一种持久化模式。

在数据恢复方面：
RDB的启动时间会更短，原因有两个：
一是RDB文件中每一条数据只有一条记录，不会像AOF日志那样可能有一条数据的多次操作记录。所以每条数据只需要写一次就行了。
另一个原因是RDB文件的存储格式和Redis数据在内存中的编码格式是一致的，不需要再进行数据编码工作，所以在CPU消耗上要远小于AOF日志的加载。



### 灾难恢复模拟

既然持久化的数据的作用是用于重启后的数据恢复，那么我们就非常有必要进行一次这样的灾难恢复模拟了。
据称如果数据要做持久化又想保证稳定性，则建议留空一半的物理内存。因为在进行快照的时候，fork出来进行dump操作的子进程会占用与父进程一样的内存，真正的copy-on-write，对性能的影响和内存的耗用都是比较大的。
目前，通常的设计思路是**利用Replication机制来弥补aof、snapshot性能上的不足，达到了数据可持久化**。
即Master上Snapshot和AOF都不做，来保证Master的读写性能，而Slave上则同时开启Snapshot和AOF来进行持久化，保证数据的安全性。



环境:

**master**: 192.168.172.128: 6379

**slave**: 192.168.172.129:6380



首先，修改Master上的如下配置：

```
#save 900 1 #禁用Snapshot
#save 300 10
#save 60 10000

appendonly no #禁用AOF
```

接着，修改Slave上的如下配置：

```
save 900 1 #启用Snapshot
save 300 10
save 60 10000

appendonly yes #启用AOF
appendfilename appendonly.aof #AOF文件的名称
# appendfsync always
appendfsync everysec #每秒钟强制写入磁盘一次
# appendfsync no  

no-appendfsync-on-rewrite yes   #在日志重写时，不进行命令追加操作
auto-aof-rewrite-percentage 100 #自动启动新的日志重写过程
auto-aof-rewrite-min-size 64mb  #启动新的日志重写过程的最小值
```



启动master与slave

```shell
# master
./redis-server /opt/redis/redis-cluster/6379/redis-6379.conf

# slave
./redis-server /opt/redis/redis-cluster/6380/redis-6380.conf
```



启动完成后在Master中确认未启动Snapshot参数

```shell
[root@master src]# ./redis-cli -c -h master -p 6379
master:6379> CONFIG GET save
1) "save"
2) ""
```



脚本生成25条数据模拟



```shell
cd /opt/redis/redis-cluster/6379/
vim redis-cli-generate.temp.sh
#!/bin/bash

REDISCLI="redis-cli -c -h master -p 6379 SET"
ID=1

while(($ID<50001))
do
  INSTANCE_NAME="i-2-$ID-VM"
  UUID=`cat /proc/sys/kernel/random/uuid`
  PRIVATE_IP_ADDRESS=10.`echo "$RANDOM % 255 + 1" | bc`.`echo "$RANDOM % 255 + 1" | bc`.`echo "$RANDOM % 255 + 1" | bc`\
  CREATED=`date "+%Y-%m-%d %H:%M:%S"`

  $REDISCLI vm_instance:$ID:instance_name "$INSTANCE_NAME"
  $REDISCLI vm_instance:$ID:uuid "$UUID"
  $REDISCLI vm_instance:$ID:private_ip_address "$PRIVATE_IP_ADDRESS"
  $REDISCLI vm_instance:$ID:created "$CREATED"

  $REDISCLI vm_instance:$INSTANCE_NAME:id "$ID"

  ID=$(($ID+1))
done

# 授权
chmod 777 ./redis-cli-generate.temp.sh
# 执行
./redis-cli-generate.temp.sh

# 查看master和slave的rdb文件
[root@master ~]# cd /opt/redis/redis-cluster/6379/
[root@master 6379]# ll
总用量 76
-rw-r--r--. 1 root root    77 5月  22 09:58 appendonly.aof
-rw-r--r--. 1 root root   176 5月  22 13:58 dump.rdb
-rw-r--r--. 1 root root  1121 5月  22 13:58 nodes.conf
-rw-r--r--. 1 root root 57802 5月  22 13:55 redis-6379.conf

[root@slave1 src]# cd ../redis-cluster/6380/
[root@slave1 6380]# ll
总用量 1540
-rw-r--r--. 1 root root 951906 5月  22 14:14 appendonly.aof
-rw-r--r--. 1 root root 462160 5月  22 14:13 dump.rdb
-rw-r--r--. 1 root root   1117 5月  22 13:31 nodes.conf
-rw-r--r--. 1 root root  57798 5月  22 13:56 redis-6380.conf

# master只在一次做slave创建，后面不随数据增大而增大
# slave aof和rdb都在增大

# 查看当前节点数量
master:6379> dbsize
(integer) 43069

# 查看详情:
master:6379> info
# Memory
used_memory:7343736
used_memory_human:7.00M
used_memory_rss:8876032
used_memory_rss_human:8.46M
used_memory_peak:7352008
used_memory_peak_human:7.01M
used_memory_peak_perc:99.89%
used_memory_overhead:3998696
used_memory_startup:1443208
used_memory_dataset:3345040
used_memory_dataset_perc:56.69%
total_system_memory:1019629568
total_system_memory_human:972.39M
used_memory_lua:37888
used_memory_lua_human:37.00K
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
mem_fragmentation_ratio:1.21
mem_allocator:jemalloc-4.0.3
active_defrag_running:0
lazyfree_pending_objects:0


# slave 执行save
[root@slave1 6380]# redis-cli -c -h slave1 -p 6380
slave1:6380> save
OK
# 拷贝aop和rdb文件
cp appendonly.aof appendonly.0520.aof
cp dump.rdb dump.0520.rdb

# master 执行flushall模拟数据丢失
./redis-cli -c -h master -p 6379
slave1:6379> flushall

# kill master和 slave，

# 将aop和rdb覆盖
cp appendonly.0520.aof appendonly.aof
cp dump.0520.rdb dump.rdb

# 将aop和rdb拷贝到master
scp dump.0520.rdb root@master:/opt/redis/redis-cluster/6379/dump.rdb
scp appendonly.0520.aof root@master:/opt/redis/redis-cluster/6379/appendonly.aof

# 重启master slave
# master
./redis-server /opt/redis/redis-cluster/6379/redis-6379.conf

# slave
./redis-server /opt/redis/redis-cluster/6380/redis-6380.conf
# 查看dbsize
./redis-cli -c -h master -p 6379
slave1:6380> dbsize
(integer) 48973


```

