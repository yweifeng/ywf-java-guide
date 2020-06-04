<!-- TOC -->

- [elasticsearch使用bulk批量增删改和mget批量查询](#elasticsearch使用bulk批量增删改和mget批量查询)
    - [bulk批量增删改](#bulk批量增删改)
    - [bulk size最佳大小](#bulk-size最佳大小)
    - [mget批量查询](#mget批量查询)

<!-- /TOC -->
# elasticsearch使用bulk批量增删改和mget批量查询



## bulk批量增删改

bulk api对json的语法，有严格的要求，每个json串不能换行，只能放一行，同时一个json串和一个json串之间，必须有一个换行**

- bulk每一个操作要两个json串（delete语法除外），语法如下：

```json
{"action":"{metadata}"}
{"data"}
```

- 举例，比如你现在要创建一个文档，放bulk里面，看起来会是这样子的

```json
{"index": {"_index": "test_index", "_type": "test_type", "_id": "1"}}
{"test_field1": "test1", "test_field2": "test2"}
```

- 有哪些类型的操作可以执行呢？
   （1）delete：删除一个文档，只需要一个json串
   （2）create：PUT /index/type/id/_create，强制创建，如果原本存在则会报错
   （3）index：普通的put操作，可以是创建文档，也可以是全量替换文档
   （4）update：执行的partial update操作
- 增删改操作

```bash
#操作
POST /_bulk
{"delete":{"_index":"test_index","_type":"test_type","_id":11}}
{"create":{"_index":"test_index","_type":"test_type","_id":3}}
{"test_field":"test3"}
{"index":{"_index":"test_index","_type":"test_type","_id":4}}
{"test_field":"test4"}
{"index":{"_index":"test_index","_type":"test_type","_id":4}}
{"test_field":"replaced test2"}
{"update":{"_index":"test_index","_type":"test_type","_id":1}}
{"doc":{"test_field2":"2"}}
```

**返回值**

```json
{
  "took" : 618,
  "errors" : true,
  "items" : [
    {
      "delete" : {
        "_index" : "test_index",
        "_type" : "test_type",
        "_id" : "11",
        "_version" : 1,
        "result" : "not_found",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 404
      }
    },
    {
      "create" : {
        "_index" : "test_index",
        "_type" : "test_type",
        "_id" : "3",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "test_index",
        "_type" : "test_type",
        "_id" : "4",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "test_index",
        "_type" : "test_type",
        "_id" : "4",
        "_version" : 2,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 200
      }
    },
    {
      "update" : {
        "_index" : "test_index",
        "_type" : "test_type",
        "_id" : "1",
        "status" : 404,
        "error" : {
          "type" : "document_missing_exception",
          "reason" : "[test_type][1]: document missing",
          "index_uuid" : "hnqCuLeHSg-wT44DrqVoHg",
          "shard" : "3",
          "index" : "test_index"
        }
      }
    }
  ]
}
```

> bulk操作中，任意一个操作失败，是不会影响其他的操作的，但是在返回结果里，会告诉你异常日志

- 如果查询的document是一个index下的不同type中的话 和 如果查询的数据都在同一个index下的同一个type下 的语法变换与 _mget 批量查找类似



## bulk size最佳大小

> bulk request会加载到内存里，如果太大的话，性能反而会下降，因此需要反复尝试一个最佳的bulk size。一般从1000条数据开始，尝试逐渐增加。另外，如果看大小的话，最好是在515MB之间。



## mget批量查询

- 批量查询的好处

> 比如说要查询100条数据，那么就要发送100次网络请求，这个开销是很大的。如果进行批量查询的话，查询100条数据，就只要发送1次网络请求，网络请求的性能开销缩减100倍。
>  -一条一条查询

```undefined
GET /test_index/test_type/1
GET /test_index/test_type/2
```

**批量查询**
 （1）查询的数据在不同的index下的

```bash
GET /_mget
{
  "docs":
  [
    {
      "_index":"test_index",
      "_type":"test_type",
      "_id": 1
    },
    {
      "_index":"test_index",
      "_type":"test_type",
      "_id": 2
    }
  ]
}
```

（2）如果查询的document是一个index下的不同type中的话

```bash
GET /test_index/_mget
{
  "docs":
  [
    {
       "_type":"test_type",
      "_id": 1
    },
    {
       "_type":"test_type",
      "_id": 2
    }
  ]
}
```

（3）如果查询的数据都在同一个index下的同一个type下

```bash
#操作
GET /test_index/test_type/_mget
{
  "ids":[1,2]
}
```

mget的重要性

- 可以说mget是很重要的，一般来说，在进行查询的时候，如果一次性要查询多条数据的话，那么一定要用batch批量操作的api尽可能减少网络开销次数，可能可以将性能提升数倍，甚至数十倍，非常非常之重要。