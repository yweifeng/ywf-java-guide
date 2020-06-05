---
title: redis集群添加节点及slot分配
category: redis
order: 2
---

### 添加设置配置文件

```shell
# redis-cluster 下添加一个master和一个slave节点
cp -r /opt/redis/redis-cluster/6379 /opt/redis/redis-cluster/6380
cp -r /opt/redis/redis-cluster/6379 /opt/redis/redis-cluster/6381

# 修改配置文件
mv redis-6379.conf redis-6381.conf
mv redis-6379.conf redis-6382.conf

# 修改2个配置文件
vim /opt/redis/redis-cluster/6380/redis-6380.conf
:%s/6379/6380/g
:wq
# 启动节点
cd /opt/redis/src
./redis-server /opt/redis/redis-cluster/6381/redis-6381.conf
./redis-server /opt/redis/redis-cluster/6382/redis-6382.conf
# 启动后登录6381查看节点情况：
./redis-cli -p 6381 -h master -c
cluster nodes
```



### 添加master节点

#### 集群新增节点

```shell
# 向集群中添加节点6381，注意一定要保证节点里面没有添加过任何数据，不然添加会报错。
# 第一个节点为新增的节点  第二个节点为集群中的节点
[root@master src]# ./redis-trib.rb add-node 192.168.172.128:6381  192.168.172.128:6379
>>> Adding node 192.168.172.128:6381 to cluster 192.168.172.128:6379
>>> Performing Cluster Check (using node 192.168.172.128:6379)
M: 035218ed678107f601fcd74a35153fd2f88e9e26 192.168.172.128:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: a09e1d78620d4d8ac02ab47c5f9548608960cdc5 192.168.172.130:6379
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: 2e84949867da404ce5ef9d584c316f0acb1c53b9 192.168.172.129:6380
   slots: (0 slots) slave
   replicates 035218ed678107f601fcd74a35153fd2f88e9e26
S: adc211d7f014dfdba019b6596e820c2a507db93a 192.168.172.128:6380
   slots: (0 slots) slave
   replicates 45fc5fbd200fdbf679f25b4575008818dd163e0f
M: 45fc5fbd200fdbf679f25b4575008818dd163e0f 192.168.172.129:6379
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 2e85e8c7dd73f55128e1c956c5ffd49b0a1a9cfb 192.168.172.130:6380
   slots: (0 slots) slave
   replicates a09e1d78620d4d8ac02ab47c5f9548608960cdc5
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
>>> Send CLUSTER MEET to node 192.168.172.128:6381 to make it join the cluster.
[OK] New node added correctly.
```

#### 检查完整性

```shell
# 检查节点完整性
[root@master src]# ./redis-trib.rb check master:6379
>>> Performing Cluster Check (using node master:6379)
M: 035218ed678107f601fcd74a35153fd2f88e9e26 master:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: a09e1d78620d4d8ac02ab47c5f9548608960cdc5 192.168.172.130:6379
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: d1a2368a7d225d8168864285791c1274c29bb0fb 192.168.172.128:6381
   slots: (0 slots) master
   0 additional replica(s)
S: 2e84949867da404ce5ef9d584c316f0acb1c53b9 192.168.172.129:6380
   slots: (0 slots) slave
   replicates 035218ed678107f601fcd74a35153fd2f88e9e26
S: adc211d7f014dfdba019b6596e820c2a507db93a 192.168.172.128:6380
   slots: (0 slots) slave
   replicates 45fc5fbd200fdbf679f25b4575008818dd163e0f
M: 45fc5fbd200fdbf679f25b4575008818dd163e0f 192.168.172.129:6379
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 2e85e8c7dd73f55128e1c956c5ffd49b0a1a9cfb 192.168.172.130:6380
   slots: (0 slots) slave
   replicates a09e1d78620d4d8ac02ab47c5f9548608960cdc5
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```



### 分配虚拟槽

```shell
# 为主节点6381分配虚拟槽
# 可以为任意的节点 在此登录的6379只是作为客户端去访问的
[root@master src]# ./redis-trib.rb reshard 192.168.172.128:6379
>>> Performing Cluster Check (using node 192.168.172.128:6379)
M: 035218ed678107f601fcd74a35153fd2f88e9e26 192.168.172.128:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: a09e1d78620d4d8ac02ab47c5f9548608960cdc5 192.168.172.130:6379
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
M: d1a2368a7d225d8168864285791c1274c29bb0fb 192.168.172.128:6381
   slots: (0 slots) master
   0 additional replica(s)
S: 2e84949867da404ce5ef9d584c316f0acb1c53b9 192.168.172.129:6380
   slots: (0 slots) slave
   replicates 035218ed678107f601fcd74a35153fd2f88e9e26
S: adc211d7f014dfdba019b6596e820c2a507db93a 192.168.172.128:6380
   slots: (0 slots) slave
   replicates 45fc5fbd200fdbf679f25b4575008818dd163e0f
M: 45fc5fbd200fdbf679f25b4575008818dd163e0f 192.168.172.129:6379
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 2e85e8c7dd73f55128e1c956c5ffd49b0a1a9cfb 192.168.172.130:6380
   slots: (0 slots) slave
   replicates a09e1d78620d4d8ac02ab47c5f9548608960cdc5
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 


# 由于我们现在4个master节点， 16384/4 = 4096

What is the receiving node ID?
# 输入 6381节点 ID
d1a2368a7d225d8168864285791c1274c29bb0fb

# 从哪些主节点抽取槽到新节点中：all为所有主节点，done：指定节点，在这里我们输入all,按enter继续：
Type 'all' 所有节点
Type 'done' 指定节点

# 输入yes后按enter开始分配


# 分配完成，检查完整性
[root@master src]#  ./redis-trib.rb check master:6379
>>> Performing Cluster Check (using node master:6379)
M: 035218ed678107f601fcd74a35153fd2f88e9e26 master:6379
   slots:1365-5460 (4096 slots) master
   1 additional replica(s)
M: a09e1d78620d4d8ac02ab47c5f9548608960cdc5 192.168.172.130:6379
   slots:12288-16383 (4096 slots) master
   1 additional replica(s)
M: d1a2368a7d225d8168864285791c1274c29bb0fb 192.168.172.128:6381
   # 分配了slots
   slots:0-1364,5461-6826,10923-12287 (4096 slots) master
   0 additional replica(s)
S: 2e84949867da404ce5ef9d584c316f0acb1c53b9 192.168.172.129:6380
   slots: (0 slots) slave
   replicates 035218ed678107f601fcd74a35153fd2f88e9e26
S: adc211d7f014dfdba019b6596e820c2a507db93a 192.168.172.128:6380
   slots: (0 slots) slave
   replicates 45fc5fbd200fdbf679f25b4575008818dd163e0f
M: 45fc5fbd200fdbf679f25b4575008818dd163e0f 192.168.172.129:6379
   slots:6827-10922 (4096 slots) master
   1 additional replica(s)
S: 2e85e8c7dd73f55128e1c956c5ffd49b0a1a9cfb 192.168.172.130:6380
   slots: (0 slots) slave
   replicates a09e1d78620d4d8ac02ab47c5f9548608960cdc5
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```



### 添加slave节点

```shell
# 添加从节点
./redis-trib.rb add-node 192.168.172.128:6382  192.168.172.128:6379

# 加入后是一个Master节点，而且没有拥有自己的slot槽。那么我们接下来要让它变成从节点。
./redis-cli -p 6382 -h 192.168.172.128

# 　使用CLUSTER REPLICATE 命令改变一个从节点的主节点
# 后面的字符串为节点6381的节点ID
cluster replicate d1a2368a7d225d8168864285791c1274c29bb0fb  

# 查看完整性
./redis-trib.rb check master:6379
```

