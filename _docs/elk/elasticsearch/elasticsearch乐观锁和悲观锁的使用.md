---
title: es乐观锁和悲观锁的使用
category: elasticsearch
order: 11
---

<!-- TOC -->

- [乐观锁](#乐观锁)
  - [内部version](#内部version)
  - [外部version](#外部version)
- [悲观锁](#悲观锁)
  - [全局锁](#全局锁)
  - [document锁](#document锁)
  - [共享锁与排它锁](#共享锁与排它锁)

<!-- /TOC -->

> 多个线程去同时访问es中的一份数据，然后各自去修改之后更新到es，由于线程的先后顺序不同，可能会导致后续的修改覆盖掉之前的修改，显然一些场景下我们是不允许发生这种并发冲突的问题，例如电商库存的修改等



### 乐观锁

#### 内部version

在es内部第次一创建document的时候，它的**_version**默认会是1，之后进行的删除和修改的操作_version都会增加1。可以看到删除一个document之后，再进行同一个id的document添加操作，版本号是加1而不是初始化为1，从而可以说明document并不是正真地被物理删除，它的一些版本号信息一样会存在，而是会在某个时刻一起被清除。

在es后台，有很多类似于replica同步的请求，这些请求都是异步多线程的，对于多个修改请求是乱序的，因此会使用_version乐观锁来控制这种并发的请求处理。当后续的修改请求先到达，对应修改成功之后_version会加1，然后检测到之前的修改到达会直接丢弃掉该请求；而当后续的修改请求按照正常顺序到达则会正常修改然后_version在前一次修改后的基础上加1（此时_version可能就是3，会基于之前修改后的状态）。



#### 外部version

**?version=1&version_type=external**

**内部version和外部version的区别**

- 对于内在_version=1，只有在后续请求满足?_version=1的时候才能够更新成功；_

- 对于外部_version=1，只有在后续请求满足?_version>1才能够修改成功。



### 悲观锁

#### 全局锁

通过doc来进行对整个index上锁

一个线程进行操作之前创建一个锁，例如：

```bash
PUT /lockindex/locktype/global/_create
{}
```

同时如果有另一个线程要进行相关更新操作，那么同样执行上述代码是会报错。在上述线程执行完DELETE对应doc之后，该线程就可以重新获取到doc的锁从而执行自己的一些列操作。

**这种方式，操作很简单，但是锁住了整个index，导致整个系统的并发能力很低**。



#### document锁

粒度更细的锁, 需要通过脚本来实现：

```bash
POST /lockindex/lock/1/_update
{
  "upsert": { "process_id": 123 },
  "script": "if ( ctx._source.process_id != process_id ) { assert false }; ctx.op = 'noop';"
  "params": {
    "process_id": 123
  }
}
```

**process_id**：很重要，会在lock中，设置对对应的doc加锁的进程的id，这样其他进程过来的时候，才知道，这条数据已经被别人给锁了 
**assert false**：不是当前进程加锁的话，则抛出异常 
**ctx.op=’noop’**：不做任何修改 
**params**：里面有个process_id，是你的要执行增删改操作的进程的唯一id

对于同一个process_id的进程是都可以来修改doc，但是用不同的process_id去修改已经上锁的其他process_id是会assert false抛错。



#### 共享锁与排它锁

**共享锁：**数据是共享的，多个线程可以获取同一个数据的共享锁，然后对这个数据执行读操作 
**排它锁：**只能有一个线程获取排它锁，然后执行更新操作

共享锁与排他锁是互斥的特性，如果有一个线程想要去修改一个数据，也就是获取一个排它锁，此时需要等待其他所有的共享锁先释放掉才能够进行操作，反之亦然。

首先添加共享锁，其他线程也可以来读取数据：

```bash
judge-lock-2.groovy: if (ctx._source.lock_type == 'exclusive') { assert false }; ctx._source.lock_count++

POST /lockindex/lock/1/_update 
{
  "upsert": { 
    "lock_type":  "shared",
    "lock_count": 1
  },
  "script": {
    "lang": "groovy",
    "file": "judge-lock-2"
  }
}
```

如果其他线程也需要获取共享锁，那么执行上述同样的代码即可，最终只是lock_count加1了：

```bash
GET /lockindex/lock/1

{
  "_index": "lockindex",
  "_type": "lock",
  "_id": "1",
  "_version": 3,
  "found": true,
  "_source": {
    "lock_type": "shared",
    "lock_count": 3
  }
}
```

当添加排他锁的时候：

```bash
PUT /lockindex/lock/1/_create
{ "lock_type": "exclusive" }
```

则会报错

对共享锁进行解锁：

```bash
POST /lockindex/lock/1/_update
{
  "script": {
    "lang": "groovy",
    "file": "unlock-shared"
  }
}
```


添加过多少个共享锁，对应的执行解锁操作相应次数即可完全解锁。每次解锁lock_count对应减1，当为0的时候就将/lockindex/lock/1删除

对应的解除排它锁：

```bash
DELETE /lockindex/lock/1
```

