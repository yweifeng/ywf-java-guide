<!-- TOC -->

- [elasticsearch多种搜索方式](#elasticsearch多种搜索方式)
    - [**bulk模拟数据**](#bulk模拟数据)
    - [query string search](#query-string-search)
    - [query DSL](#query-dsl)
    - [filter DSL](#filter-dsl)
    - [**query 和 filter的区别**](#query-和-filter的区别)
    - [query filter 组合使用](#query-filter-组合使用)
    - [full-text search（全文检索）](#full-text-search全文检索)
        - [提高查询精度](#提高查询精度)
        - [控制查询精度](#控制查询精度)
    - [phrase search（短语搜索）](#phrase-search短语搜索)
    - [highlight search（高亮搜索结果）](#highlight-search高亮搜索结果)

<!-- /TOC -->
# elasticsearch多种搜索方式

## **bulk模拟数据**

使用**bulk**批量导入6条数据

```bash
POST /_bulk
{"index":{"_index":"goods","_type":"product","_id":"1"}}
{"name":"heimei toothpaste","price":20.0,"tags":["toothpaste"]}
{"index":{"_index":"goods","_type":"product","_id":"2"}}
{"name":"gaolujie toothpaste","price":12.0,"tags":["toothpaste"]}
{"index":{"_index":"goods","_type":"product","_id":"3"}}
{"name":"lining shoes","price":100.0,"tags":["shoes"]}
{"index":{"_index":"goods","_type":"product","_id":"4"}}
{"name":"anta shoes","price":120.0,"tags":["shoes"]}
{"index":{"_index":"goods","_type":"product","_id":"5"}}
{"name":"adidas shoes","price":200.0,"tags":["sport","foreign shoes"]}
{"index":{"_index":"goods","_type":"product","_id":"6"}}
{"name":"nike shoes","price":300.0,"tags":["sport","foreign shoes"]}

```



## query string search

**搜索全部商品**

```bash
GET /goods/product/_search
```

返回值

```json
{
  // 耗费了50毫秒
  "took" : 50,
  // 无超时
  "timed_out" : false,
  // 数据拆分了5个分片，请求会发到所有的primary shard(或者是replicas shard)
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    // 匹配到6条记录
    "total" : 6,
    // 最大匹配度
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "5",
        "_score" : 1.0,
        "_source" : {
          "name" : "adidas shoes",
          "price" : 200.0,
          "tags" : [
            "sport",
            "foreign shoes"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "gaolujie toothpaste",
          "price" : 12.0,
          "tags" : [
            "toothpaste"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "4",
        "_score" : 1.0,
        "_source" : {
          "name" : "anta shoes",
          "price" : 120.0,
          "tags" : [
            "shoes"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "6",
        "_score" : 1.0,
        "_source" : {
          "name" : "nike shoes",
          "price" : 300.0,
          "tags" : [
            "sport",
            "foreign shoes"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "heimei toothpaste",
          "price" : 20.0,
          "tags" : [
            "toothpaste"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "name" : "lining shoes",
          "price" : 100.0,
          "tags" : [
            "shoes"
          ]
        }
      }
    ]
  }
}
```



**搜索商品名称中包含shoes的商品，而且按照售价降序排序**

```bash
GET /goods/product/_search?q=name:shoes&sort=price:desc
```

**返回结果**

```json
{
  "took" : 149,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 4,
    "max_score" : null,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "6",
        "_score" : null,
        "_source" : {
          "name" : "nike shoes",
          "price" : 300.0,
          "tags" : [
            "sport",
            "foreign shoes"
          ]
        },
        "sort" : [
          300.0
        ]
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "5",
        "_score" : null,
        "_source" : {
          "name" : "adidas shoes",
          "price" : 200.0,
          "tags" : [
            "sport",
            "foreign shoes"
          ]
        },
        "sort" : [
          200.0
        ]
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "4",
        "_score" : null,
        "_source" : {
          "name" : "anta shoes",
          "price" : 120.0,
          "tags" : [
            "shoes"
          ]
        },
        "sort" : [
          120.0
        ]
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "3",
        "_score" : null,
        "_source" : {
          "name" : "lining shoes",
          "price" : 100.0,
          "tags" : [
            "shoes"
          ]
        },
        "sort" : [
          100.0
        ]
      }
    ]
  }
}
```



**总结：**

```
适用于临时的在命令行使用一些工具，比如curl，快速的发出请求，来检索想要的信息；但是如果查询请求很复杂，是很难去构建的
在生产环境中，几乎很少使用query string search
```



## query DSL

DSL：Domain Specified Language，特定领域的语言
http request body：请求体，可以用json的格式来构建查询语法，比较方便，可以构建各种复杂的语法，比query string search强大



**查询所有的商品**

```bash
GET /goods/product/_search
{
	"query":{
		"match_all":{}
	}
}
```



**搜索商品名称中包含shoes的商品，而且按照售价降序排序**

```bash
GET /goods/product/_search
{
	"query": {
		"match": {
			"name": "shoes"
		}
	},
	"sort": [{
		"price": "desc"
	}]
}
```



**查询所有商品，按商品价格降序,每页显示3条**

```bash
GET /goods/product/_search
{
	"query": {
		"match_all": {}
	},
	"sort":[{
		"price": "desc"
	}],
	"from": 0,
	"size": 3
}
```



**查询所有商品，只展示商品名称和价格**

```bash
GET /goods/product/_search
{
	"query": {
		"match_all": {}
	},
	"_source": ["name","price"]
}
```

**返回值**

```json
{
  "took" : 95,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 6,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "5",
        "_score" : 1.0,
        "_source" : {
          "price" : 200.0,
          "name" : "adidas shoes"
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "price" : 12.0,
          "name" : "gaolujie toothpaste"
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "4",
        "_score" : 1.0,
        "_source" : {
          "price" : 120.0,
          "name" : "anta shoes"
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "6",
        "_score" : 1.0,
        "_source" : {
          "price" : 300.0,
          "name" : "nike shoes"
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "price" : 20.0,
          "name" : "heimei toothpaste"
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "price" : 100.0,
          "name" : "lining shoes"
        }
      }
    ]
  }
}
```



## filter DSL

**搜索且售价大于100元的商品**

```bash
GET /goods/product/_search
{
	"query": {
		"bool": {
            "filter": {
                "range": {
                    "price": {
                        "gt": 100
                    }
                }
            }
		}
	}
}
```

**返回结果**

```json
{
  "took" : 24,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 3,
    "max_score" : 0.0,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "5",
        "_score" : 0.0,
        "_source" : {
          "name" : "adidas shoes",
          "price" : 200.0,
          "tags" : [
            "sport",
            "foreign shoes"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "4",
        "_score" : 0.0,
        "_source" : {
          "name" : "anta shoes",
          "price" : 120.0,
          "tags" : [
            "shoes"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "6",
        "_score" : 0.0,
        "_source" : {
          "name" : "nike shoes",
          "price" : 300.0,
          "tags" : [
            "sport",
            "foreign shoes"
          ]
        }
      }
    ]
  }
}
```



## **query 和 filter的区别**

```
query filter在性能上对比：filter是不计算相关性的，同时可以cache。因此，filter速度要快于query。

query与filter区别如下：
1. query是要相关性评分的，filter不要；
2. query结果无法缓存，filter可以。

所以，选择参考：
1. 全文搜索、评分排序，使用query；
2. 是非过滤，精确匹配，使用filter。
```



## query filter 组合使用

**搜索且售价大于100元的商品，并且名字包含shoes，按商品价格降序排序**

```bash
GET /goods/product/_search
{
	"query":{
		"bool": {
			"must": {
				"match": {
					"name": "shoes"
				}
			},
			"filter": {
				"range": {
					"price": {
						"gt": 100
					}
				}
			}
		}
	},
	"sort": [{
	  "price":"desc"
	}]
}
```



## full-text search（全文检索）

**全文检索名称包含lining shoes的商品**

```bash
GET /goods/product/_search
{
	"query": {
		"match": {
			"name": "lining shoes"
		}
	}
}
```

**等价于**

```bash
GET /goods/product/_search
{
	"query": {
        "bool": {
            "should": [
                {
                    "term": {
                        "name": "lining"
                    }
                }, {
                    "term": {
                        "name": "shoes"
                    }
                }
            ]
        }
	}
}
```

**返回结果**

```json
{
  "took" : 40,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 4,
    "max_score" : 0.5753642,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "3",
        "_score" : 0.5753642,
        "_source" : {
          "name" : "lining shoes",
          "price" : 100.0,
          "tags" : [
            "shoes"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "4",
        "_score" : 0.47000363,
        "_source" : {
          "name" : "anta shoes",
          "price" : 120.0,
          "tags" : [
            "shoes"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "6",
        "_score" : 0.47000363,
        "_source" : {
          "name" : "nike shoes",
          "price" : 300.0,
          "tags" : [
            "sport",
            "foreign shoes"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "5",
        "_score" : 0.2876821,
        "_source" : {
          "name" : "adidas shoes",
          "price" : 200.0,
          "tags" : [
            "sport",
            "foreign shoes"
          ]
        }
      }
    ]
  }
}
```



### 提高查询精度

match查询接受一个operator参数，该参数的默认值是"**or**"。你可以将它改变为"**and**"来要求所有的词条都需要被匹配：

```bash
GET /goods/product/_search
{
	"query": {
		"match": {
			"name": {
				"query": "lining shoes",
				"operator": "and"
			}
		}
	}
}
```

**等价于**

```bash
GET /goods/product/_search
{
	"query": {
		"bool": {
			"must": [{
					"term": {
						"name": "lining"
					}
				},
				{
					"term": {
						"name": "shoes"
					}
				}
			]
		}
	}
}
```

**返回值**

```json
{
  "took" : 5,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.5753642,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "3",
        "_score" : 0.5753642,
        "_source" : {
          "name" : "lining shoes",
          "price" : 100.0,
          "tags" : [
            "shoes"
          ]
        }
      }
    ]
  }
}
```



### 控制查询精度

match查询支持**minimum_should_match**参数，它能够让你指定有多少词条必须被匹配才会让该文档被当做一个相关的文档。尽管你能够指定一个词条的绝对数量，但是通常指定一个百分比会更有意义，因为你无法控制用户会输入多少个词条

```bash
GET /goods/product/_search
{
	"query": {
		"match": {
			"name": {
				"query": "lining shoes",
				"minimum_should_match": "100%"
			}
		}
	}
}
```



## phrase search（短语搜索）

phrase search，要求输入的搜索串，必须在指定的字段文本中，完全包含一模一样的，才可以算匹配，才能作为结果返回

**查询商品名称包含 anta shoes 的商品**

```bash
GET /goods/product/_search
{
	"query": {
		"match_phrase": {
			"name": "anta shoes"
		}
	}
}
```

**返回值**

```json
{
  "took" : 30,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 1.4508328,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "4",
        "_score" : 1.4508328,
        "_source" : {
          "name" : "anta shoes",
          "price" : 120.0,
          "tags" : [
            "shoes"
          ]
        }
      }
    ]
  }
}
```



## highlight search（高亮搜索结果）

**查询商品名字包含shoes的商品，高亮显示**

```bash
GET /goods/product/_search
{
	"query": {
		"match": {
			"name": "shoes"
		}
	},
	"highlight": {
		"fields":{
			"name": {}
		}
	}
}
```

**返回值**

```json
{
  "took" : 258,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 4,
    "max_score" : 0.47000363,
    "hits" : [
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "4",
        "_score" : 0.47000363,
        "_source" : {
          "name" : "anta shoes",
          "price" : 120.0,
          "tags" : [
            "shoes"
          ]
        },
        "highlight" : {
          "name" : [
            "anta <em>shoes</em>"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "6",
        "_score" : 0.47000363,
        "_source" : {
          "name" : "nike shoes",
          "price" : 300.0,
          "tags" : [
            "sport",
            "foreign shoes"
          ]
        },
        "highlight" : {
          "name" : [
            "nike <em>shoes</em>"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "5",
        "_score" : 0.2876821,
        "_source" : {
          "name" : "adidas shoes",
          "price" : 200.0,
          "tags" : [
            "sport",
            "foreign shoes"
          ]
        },
        "highlight" : {
          "name" : [
            "adidas <em>shoes</em>"
          ]
        }
      },
      {
        "_index" : "goods",
        "_type" : "product",
        "_id" : "3",
        "_score" : 0.2876821,
        "_source" : {
          "name" : "lining shoes",
          "price" : 100.0,
          "tags" : [
            "shoes"
          ]
        },
        "highlight" : {
          "name" : [
            "lining <em>shoes</em>"
          ]
        }
      }
    ]
  }
}
```

