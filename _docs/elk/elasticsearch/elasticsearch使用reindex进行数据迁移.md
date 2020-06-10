---
title: es使用reindex进行数据迁移
category: elasticsearch
order: 10
---

<!-- TOC -->

- [应用背景](#应用背景)
- [Reindex](#reindex)
- [数据迁移步骤](#数据迁移步骤)
- [提升数据迁移效率](#提升数据迁移效率)
  - [1 ）提升批量写入大小值](#1-提升批量写入大小值)
  - [2 ）借助scroll的sliced提升写入效率](#2-借助scroll的sliced提升写入效率)

<!-- /TOC -->



### 应用背景

- 当你的数据量过大，而你的索引最初创建的分片数量不足，导致数据入库较慢的情况，此时需要扩大分片的数量，此时可以尝试使用Reindex。

- 当数据的mapping需要修改，但是大量的数据已经导入到索引中了，重新导入数据到新的索引太耗时；但是在ES中，一个字段的mapping在定义并且导入数据之后是不能再修改的。



### Reindex

ES提供了**_reindex**这个API。相对于我们重新导入数据肯定会快不少，实测速度大概是**bulk**导入数据的5-10倍。



### 数据迁移步骤

**1、创建新的索引(可以通过java程序也可以直接在head插件上创建)**

**2、复制数据**

```bash
POST _reindex
{
  "source": {
    "index": "old_index"
  },
  "dest": {
    "index": "new_index"
  }
}
```

但如果新的index中有数据，并且可能发生冲突，那么可以设置version_type`"version_type": "internal"`或者不设置，则Elasticsearch强制性的将文档转储到目标中。

```bash
POST _reindex
{
  "source": {
    "index": "old_index"
  },
  "dest": {
    "index": "new_index",
    "version_type": "internal"
  }
}
```



### 提升数据迁移效率

问题发现：

常规的如果我们只是进行少量的数据迁移利用普通的reindex就可以很好的达到要求，但是当我们发现我们需要迁移的数据量过大时，我们会发现reindex的速度会变得很慢。



**数据量几十个G的场景下，elasticsearch reindex速度太慢，从旧索引导数据到新索引，当前最佳方案是什么？**

原因分析：

reindex的核心做跨索引、跨集群的数据迁移。
慢的原因及优化思路无非包括：
  1）批量大小值可能太小。需要结合堆内存、线程池调整大小；
  2）reindex的底层是scroll实现，借助scroll并行优化方式，提升效率；
  3）跨索引、跨集群的核心是写入数据，考虑写入优化角度提升效率。

**可行方案**：

#### 1）提升批量写入大小值

默认情况下，_reindex使用1000进行批量操作，您可以在**source**中调整**batch_size**。

```bash
POST _reindex
{
  "source": {
    "index": "source",
    "size": 5000
  },
  "dest": {
    "index": "dest"
  }
}
```

**批量大小设置的依据**：

**1、使用批量索引请求以获得最佳性能。**

批量大小取决于数据、分析和集群配置，但一个好的起点是每批处理5-15 MB。

注意，这是物理大小。文档数量不是度量批量大小的好指标。例如，如果每批索引1000个文档:

1）每个1kb的1000个文档是1mb。

2）每个100kb的1000个文档是100 MB。

这些是完全不同的体积大小。

**2、逐步递增文档容量大小的方式调优。**

1）从大约5-15 MB的大容量开始，慢慢增加，直到你看不到性能的提升。然后开始增加批量写入的并发性(多线程等等)。

2）使用kibana、cerebro或iostat、top和ps等工具监视节点，以查看资源何时开始出现瓶颈。如果您开始接收EsRejectedExecutionException，您的集群就不能再跟上了:至少有一个资源达到了容量。

要么减少并发性，或者提供更多有限的资源(例如从机械硬盘切换到ssd固态硬盘)，要么添加更多节点。



#### 2）借助scroll的sliced提升写入效率

Reindex支持Sliced Scroll以并行化重建索引过程。 这种并行化可以提高效率，并提供一种方便的方法将请求分解为更小的部分。

sliced原理（from medcl）

1）用过Scroll接口吧，很慢？如果你数据量很大，用Scroll遍历数据那确实是接受不了，现在Scroll接口可以并发来进行数据遍历了。 
2）每个Scroll请求，可以分成多个Slice请求，可以理解为切片，各Slice独立并行，利用Scroll重建或者遍历要快很多倍。

slicing使用举例

slicing的设定分为两种方式：手动设置分片、自动设置分片。 
手动设置分片参见官网。 
自动设置分片如下：

```bash
POST _reindex?slices=5&refresh
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
```

**slices大小设置注意事项：**

1）slices大小的设置可以手动指定，或者设置slices设置为auto，auto的含义是：针对单索引，slices大小=分片数；针对多索引，slices=分片的最小值。
2）当slices的数量等于索引中的分片数量时，查询性能最高效。slices大小大于分片数，非但不会提升效率，反而会增加开销。
3）如果这个slices数字很大(例如500)，建议选择一个较低的数字，因为过大的slices 会影响性能。

效果

实践证明，比默认设置reindex速度能提升10倍+。