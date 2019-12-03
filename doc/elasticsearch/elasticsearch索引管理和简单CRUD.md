<!-- TOC -->

- [elasticsearch索引管理和简单CRUD](#elasticsearch索引管理和简单crud)
    - [创建索引](#创建索引)
    - [修改索引副本数量](#修改索引副本数量)
    - [查询索引分片信息](#查询索引分片信息)
    - [查看索引mapping](#查看索引mapping)
    - [新建索引mapping](#新建索引mapping)
    - [简单的CRUD](#简单的crud)
        - [新增数据](#新增数据)
        - [查询数据](#查询数据)
        - [修改数据](#修改数据)
            - [**方式一：全量替换**](#方式一全量替换)
            - [**方式二：局部修改**](#方式二局部修改)
        - [删除数据](#删除数据)

<!-- /TOC -->
# elasticsearch索引管理和简单CRUD

## 创建索引

创建 school索引，设置3个**主分片**（number_of_shards），每个主分片有3个**副本**（number_of_replicas）

```
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

```
PUT /goods/_settings
{
	"number_of_replicas": 2
}
```



## 查询索引分片信息

```
GET /goods/_search_shards
```

**返回值**

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

```
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

```
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

```
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



### 查询数据

```
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

```
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

```
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

```
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
