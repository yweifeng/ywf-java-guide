# elasticsearch索引管理和简单CRUD

## 创建索引

创建 school索引，设置3个**主分片**（number_of_shards），每个主分片有3个**副本**（number_of_replicas）

```http
PUT /goods
{
	"settings": {
		"number_of_shards": 3,
		"number_of_replicas": 3
	}
}
```

- 主分片数量不能修改
- 副本数量可以修改

返回结果：

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "goods"
}
```



## 修改索引副本数量

```http
PUT /goods/_settings
{
	"number_of_replicas": 2
}
```



## 查询索引分片信息

```http
GET /goods/_search_shards
```

```json
{
  "nodes" : {
    "m-9_0VpRQRmFhNERW5Xzlg" : {
      "name" : "node-1",
      "ephemeral_id" : "NW22_BncRlSwuQdkDOXBVQ",
      "transport_address" : "192.168.111.128:9300",
      "attributes" : {
        "ml.machine_memory" : "1021931520",
        "xpack.installed" : "true",
        "ml.max_open_jobs" : "20",
        "ml.enabled" : "true"
      }
    },
    "mbk6euKWQOywsp2vjq1t_g" : {
      "name" : "node-2",
      "ephemeral_id" : "hqQ3GTkNRviVUSnhUmJMPQ",
      "transport_address" : "192.168.111.129:9300",
      "attributes" : {
        "ml.machine_memory" : "1021931520",
        "ml.max_open_jobs" : "20",
        "xpack.installed" : "true",
        "ml.enabled" : "true"
      }
    },
    "EreG9bpzQOKtolB13X2N9w" : {
      "name" : "node-3",
      "ephemeral_id" : "aQs53m-ISTqYDKy3D6A2uA",
      "transport_address" : "192.168.111.130:9300",
      "attributes" : {
        "ml.machine_memory" : "1021931520",
        "ml.max_open_jobs" : "20",
        "xpack.installed" : "true",
        "ml.enabled" : "true"
      }
    }
  },
  "indices" : {
    "goods" : { }
  },
  "shards" : [
    [
      {
        "state" : "STARTED",
        "primary" : false,
        "node" : "m-9_0VpRQRmFhNERW5Xzlg",
        "relocating_node" : null,
        "shard" : 0,
        "index" : "goods",
        "allocation_id" : {
          "id" : "BONEet9EQ1GszSAbgWZREA"
        }
      },
      {
        "state" : "STARTED",
        "primary" : true,
        "node" : "mbk6euKWQOywsp2vjq1t_g",
        "relocating_node" : null,
        "shard" : 0,
        "index" : "goods",
        "allocation_id" : {
          "id" : "ZLL5EBkTQdaOTJdWkYh3VQ"
        }
      },
      {
        "state" : "STARTED",
        "primary" : false,
        "node" : "EreG9bpzQOKtolB13X2N9w",
        "relocating_node" : null,
        "shard" : 0,
        "index" : "goods",
        "allocation_id" : {
          "id" : "sl9o9YcvSkm4RI5iYevh0w"
        }
      }
    ],
    [
      {
        "state" : "STARTED",
        "primary" : true,
        "node" : "EreG9bpzQOKtolB13X2N9w",
        "relocating_node" : null,
        "shard" : 1,
        "index" : "goods",
        "allocation_id" : {
          "id" : "CgQwG1KeQ5ugZ8mjsLj-mQ"
        }
      },
      {
        "state" : "STARTED",
        "primary" : false,
        "node" : "m-9_0VpRQRmFhNERW5Xzlg",
        "relocating_node" : null,
        "shard" : 1,
        "index" : "goods",
        "allocation_id" : {
          "id" : "wg0PWUOARk6NsMXqB3JfUw"
        }
      },
      {
        "state" : "STARTED",
        "primary" : false,
        "node" : "mbk6euKWQOywsp2vjq1t_g",
        "relocating_node" : null,
        "shard" : 1,
        "index" : "goods",
        "allocation_id" : {
          "id" : "xlfEUHXgSAmNy1ljM2UI-A"
        }
      }
    ],
    [
      {
        "state" : "STARTED",
        "primary" : false,
        "node" : "EreG9bpzQOKtolB13X2N9w",
        "relocating_node" : null,
        "shard" : 2,
        "index" : "goods",
        "allocation_id" : {
          "id" : "0TLKDM44TlqbuwjodP_uzA"
        }
      },
      {
        "state" : "STARTED",
        "primary" : true,
        "node" : "m-9_0VpRQRmFhNERW5Xzlg",
        "relocating_node" : null,
        "shard" : 2,
        "index" : "goods",
        "allocation_id" : {
          "id" : "WWgfzUCSQNaxl8QsAQG3jw"
        }
      },
      {
        "state" : "STARTED",
        "primary" : false,
        "node" : "mbk6euKWQOywsp2vjq1t_g",
        "relocating_node" : null,
        "shard" : 2,
        "index" : "goods",
        "allocation_id" : {
          "id" : "15tPaYkbQUu_a0Bt3aMoog"
        }
      }
    ]
  ]
}
```



## 查看索引mapping

```http
GET /goods/_mapping
```

**返回值**

```json
{
  "goods" : {
    "mappings" : { }
  }
}
```

- 新建的索引中，mapping是一个空集，所以我们就要创建这个index的mapping



## 新建索引mapping

```http
POST goods/product/_mapping?pretty
{
	"product": {
		"properties": {
			"title": {
				"type": "text",
				"store": "true"
			},
			"description": {
				"type": "text",
				"index": "false"
			},
			"price": {
				"type": "double"
			},
			"onSale": {
				"type": "boolean"
			},
			"type": {
				"type": "integer"
			},
			"createDate": {
				"type": "date"
			}
		}
	}
}
```

- 或者新增数据的时候，es会根据类型自动生成索引、默认分片和副本。



## 简单的CRUD

### 新增数据

```http
PUT /goods/product/1
{
	"title": "goods 001",
	"description": "this is a random desc ",
	"price": 22.6,
	"onSale": "true",
	"type": 2,
	"createDate": "2019-01-12"
}
```

**返回值**

```json
{
  "_index" : "goods",
  "_type" : "product",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```



## 查询数据

```http
GET /goods/product/1
```

**返回值**

```json
{
  "_index" : "goods",
  "_type" : "product",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "title" : "goods 001",
    "description" : "this is a random desc ",
    "price" : 22.6,
    "onSale" : "true",
    "type" : 2,
    "createDate" : "2019-01-12"
  }
}
```



### 修改数据

#### **方式一：全量替换**

```http
PUT /goods/product/1
{
	"title": "goods 001",
	"description": "this is a random desc ",
	"price": 22.6,
	"onSale": "false",
	"type": 2,
	"createDate": "2019-01-12"
}
```

**返回值**

```json
{
  "_index" : "goods",
  "_type" : "product",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```



#### **方式二：局部修改**

```http
POST /goods/product/1/_update
{
  "doc": {
    "onSale": "true"
  }
}
```

**返回值**

```json
{
  "_index" : "goods",
  "_type" : "product",
  "_id" : "1",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```



### 删除数据

```http
DELETE /goods/product/1
```

**返回值**

```json
{
  "_index" : "goods",
  "_type" : "product",
  "_id" : "1",
  "_version" : 4,
  "result" : "deleted",
  "_shards" : {
    "total" : 3,
    "successful" : 3,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}
```



## 刪除或者更新原理

```
如果是删除操作，commit的时候会生成一个 .del文件，里面将某个doc标识为 deleted状态，那么搜索的时候根据 .del文件就知道这个doc是否被删除了。

如果是更新操作，就是将原来的doc标识为deleted状态，然后重新写入一条数据。

buffer 每refresh一次，就会产生一个segment file，所以默认情况下是1秒钟一个segment file，这样下来segment file会越来越多，此时会定期执行merge。

每次merge的时候，会将多个segment file合并成一个，同时这里会将标识为 deleted的doc给物理删除掉，然后将新的segment file写入磁盘，这里会写一个

commit point，标识所有新的 segment file，然后打开segment file供搜索使用，同时删除旧的segment file。
```