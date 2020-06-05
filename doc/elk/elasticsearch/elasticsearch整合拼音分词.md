---
title: elasticsearch整合拼音分词
category: elasticsearch
order: 5
---



## 插件安装

github 下载与es相同版本的zip安装包[https://github.com/medcl/elasticsearch-analysis-pinyin/releases](https://github.com/medcl/elasticsearch-analysis-pinyin/releases?)

```shell
cd /opt/elk/es/plugins
# 创建目录 analysis-pinyin
mkdir analysis-pinyin
cd analysis-pinyin
# 解压到/opt/elk/es/plugins 目录
unzip elasticsearch-analysis-pinyin-6.7.1.zip
```

**scp到其他服务器**

```shell
scp -r /opt/elk/es/plugins/analysis-pinyin root@192.168.111.129:/opt/elk/es/plugins
scp -r /opt/elk/es/plugins/analysis-pinyin root@192.168.111.130:/opt/elk/es/plugins
```

**重启es集群**

```shell
su es
cd /opt/elk/es/bin
./elasticsearch -d
```



## 拼音分词 + IK分词混合使用

### 创建索引，配置两种分词器

```bash
PUT /blog/
{
	"settings":{
		"number_of_shards": 3,
		"number_of_replicas": 1,
		"analysis": {
            "analyzer": {
				"default":{
					"tokenizer":"ik_max_word"
				},				
                "pinyin_analyzer": {
                    "type": "custom",
                    "tokenizer": "my_pinyin",
                    "filter": ["word_delimiter"]
                }
            },
            "tokenizer": {
                "my_pinyin" : {
                      "type" : "pinyin",
                      "keep_first_letter":true,
                      "keep_separate_first_letter" : false,
                      "keep_full_pinyin" : true,
                      "keep_original" : false,
                      "limit_first_letter_length" : 16,
                      "lowercase" : true
                  }
            }
        }
	}
}
```



### **针对索引配置字段映射**

```bash
PUT /blog/_mapping/article
{    
	"properties" : {
		"title" : {
			"type" : "text",
			"analyzer" : "ik_max_word",
			"fields" : {
				"pinyin" : {
					"type" : "text",
					"term_vector" : "with_positions_offsets",
					"analyzer" : "pinyin_analyzer",
					"boost" : 10
				  }
			 }
		},      
		"content":{
		  "type":"text",
		  "analyzer" : "ik_max_word"
		},
		"author":{
		  "type":"text",
		  "analyzer" : "ik_max_word"
		},
		"keyword":{
		  "type":"text",
		  "analyzer" : "ik_max_word"
		},
		"createtime":{			
			"type":"date",			
			"format":"yyyy-MM-dd HH:mm:ss || yyyy-MM-dd || epoch_millis"	
		}
	}
}
```



### 检查自定义的词语分析器是否生效

```bash
GET /blog/_analyze
{
  "text": "刘德华",
  "analyzer": "pinyin_analyzer"
}
```

**返回值**

```json
{
  "tokens" : [
    {
      "token" : "liu",
      "start_offset" : 0,
      "end_offset" : 0,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "ldh",
      "start_offset" : 0,
      "end_offset" : 0,
      "type" : "word",
      "position" : 0
    },
    {
      "token" : "de",
      "start_offset" : 0,
      "end_offset" : 0,
      "type" : "word",
      "position" : 1
    },
    {
      "token" : "hua",
      "start_offset" : 0,
      "end_offset" : 0,
      "type" : "word",
      "position" : 2
    }
  ]
}
```



### 填充一些内容

```bash
PUT /blog/article/2
{
  "title": "王者荣耀好玩吗",
  "content": "王者荣耀不好玩",
  "author": "王军",
  "createtime": "2018-12-26",
  "keyword": "王者荣耀"
}
```



### 按照拼音搜索

```bash
GET /blog/article/_search
{
    "query":{
      "match":{
        "title.pinyin":"wang"
      }
    }
}
```

**返回值**

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 3.031859,
    "hits" : [
      {
        "_index" : "blog",
        "_type" : "article",
        "_id" : "2",
        "_score" : 3.031859,
        "_source" : {
          "title" : "王者荣耀好玩吗",
          "content" : "王者荣耀不好玩",
          "author" : "王军",
          "createtime" : "2018-12-26",
          "keyword" : "王者荣耀"
        }
      }
    ]
  }
}
```



### 按照中文名进行搜索

```bash
GET /blog/article/_search
{
  "query": {
    "match": {
      "title": "王者荣耀"
    }
  }
}
```



### 中文加拼音混合

```bash
GET /blog/article/_search
{
  "query": {
    "multi_match": {
      "type":"most_fields",
      "query":"王者rongyao",
      "fields":["title", "title.pinyin"]
    }
  }
}
```

**返回值**

```json
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 3,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 9.686445,
    "hits" : [
      {
        "_index" : "blog",
        "_type" : "article",
        "_id" : "2",
        "_score" : 9.686445,
        "_source" : {
          "title" : "王者荣耀好玩吗",
          "content" : "王者荣耀不好玩",
          "author" : "王军",
          "createtime" : "2018-12-26",
          "keyword" : "王者荣耀"
        }
      }
    ]
  }
}
```



### 通过 validate-query API 可以帮助理解查询是如何执行的

```bash
GET /blog/article/_validate/query?explain
{
  "query": {
    "multi_match": {
      "type":"most_fields",
      "query":"王者rongyao",
      "fields":["title", "title.pinyin"]
    }
  }
}
```

**返回值**

```json
{
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "valid" : true,
  "explanations" : [
    {
      "index" : "blog",
      "valid" : true,
      "explanation" : "+((title:王者 title:rongyao) | ((title.pinyin:wang)^10.0 (title.pinyin:zhe)^10.0 (title.pinyin:rong)^10.0 Synonym(title.pinyin:wzrongyao title.pinyin:yao)))~1.0 #*:*"
    }
  ]
}
```

