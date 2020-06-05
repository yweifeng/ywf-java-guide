---
title: logstash介绍和部署
category: logstash
order: 1
---



## logstash介绍

Logstash使用管道方式进行日志的搜集处理和输出。Logstash管道有两个必需的元素，输入和输出，以及一个可选元素过滤器。输入插件从数据源那里消费数据，过滤器插件根据你的期望修改数据，输出插件将数据写入目的地。在logstash中，包括了三个阶段:输入input --> 处理filter（不是必须的） --> 输出output

![img](../../../images/logstash/l2.png)



### 输入

logstash主要支持的数据输入方式为：标准输入、文件输入、TCP输入、syslog输入、http_poller抓取、kafka消息队列输入等。

### 过滤器

数据从源传输到存储库的过程中，Logstash 过滤器能够解析各个事件，识别已命名的字段以构建结构，并将它们转换成通用格式，以便更轻松、更快速地分析和实现商业价值。Logstash 能够动态地转换和解析数据，不受格式或复杂度的影响。

目前，logstash过滤器支持的功能有：date时间处理、GROK正则捕获、dissect解析、GeoIP地址查询、Json编解码、metrics数据修改、splite拆分事件、交叉日志合并等。

### 输出

目前，logstash主要支持的输出方式：输出到Elasticsearch、发送email、调用系统命令执行、保存成文件、报警发送到Nagios、标准输出stdout、TCP发送数据、输出到HDFS。



## 下载安装

下载地址：[https://www.elastic.co/cn/downloads/past-releases](https://www.elastic.co/cn/downloads/past-releases/)

```shell
# 解压
tar -zxvvf logstash-6.7.1.tar.gz
# 重命名
mv logstash-6.7.1 logstash
```



## 测试

### 简单测试

**新建logstatsh_test.conf**

vim config/logstash_test.conf

```bash
input {
    stdin {
    }
}
output {
    stdout {
    	codec => rubydebug {}
    }
}
```

执行测试

```shell
bin/logstash -f config/logstash_test.conf
```

返回值

```shell
Sending Logstash logs to /opt/elk/logstash/logs which is now configured via log4j2.properties
[2019-12-05T00:32:44,647][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modules or command line options are specified
[2019-12-05T00:32:44,694][INFO ][logstash.runner          ] Starting Logstash {"logstash.version"=>"6.7.1"}
[2019-12-05T00:32:59,798][INFO ][logstash.pipeline        ] Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>4, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>50}
[2019-12-05T00:33:00,114][INFO ][logstash.pipeline        ] Pipeline started successfully {:pipeline_id=>"main", :thread=>"#<Thread:0x19dc6069 run>"}
The stdin plugin is now waiting for input:
[2019-12-05T00:33:00,399][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
[2019-12-05T00:33:01,738][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
hello world
/opt/elk/logstash/vendor/bundle/jruby/2.5.0/gems/awesome_print-1.7.0/lib/awesome_print/formatters/base_formatter.rb:31: warning: constant ::Fixnum is deprecated
{
       "message" => "hello world",
          "host" => "hadoop-slave1",
      "@version" => "1",
    "@timestamp" => 2019-12-04T16:33:12.742Z
}
```



### 发送到ES

新建logstash_es_test.conf

```bash
vim config/logstash_es_test.conf
input{
    stdin{
    }
}
output{
    elasticsearch {
        hosts => ["192.168.111.128:9200","192.168.111.129:9200","192.168.111.130:9200"] 
        index => "logstash_test"
    }
    stdout{
        codec=>rubydebug
    }
}
```

执行测试

```shell
bin/logstash -f config/logstash_es_test.conf
```

返回值

```shell
Sending Logstash logs to /opt/elk/logstash/logs which is now configured via log4j2.properties
[2019-12-05T00:44:28,140][WARN ][logstash.config.source.multilocal] Ignoring the 'pipelines.yml' file because modules or command line options are specified
[2019-12-05T00:44:28,239][INFO ][logstash.runner          ] Starting Logstash {"logstash.version"=>"6.7.1"}

[2019-12-05T00:45:10,693][INFO ][logstash.pipeline        ] Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>4, "pipeline.batch.size"=>125, "pipeline.batch.delay"=>50}
[2019-12-05T00:45:17,182][INFO ][logstash.outputs.elasticsearch] Elasticsearch pool URLs updated {:changes=>{:removed=>[], :added=>[http://192.168.111.128:9200/, http://192.168.111.129:9200/, http://192.168.111.130:9200/]}}
[2019-12-05T00:45:18,525][WARN ][logstash.outputs.elasticsearch] Restored connection to ES instance {:url=>"http://192.168.111.128:9200/"}
[2019-12-05T00:45:19,707][INFO ][logstash.outputs.elasticsearch] ES Output version determined {:es_version=>6}
[2019-12-05T00:45:19,741][WARN ][logstash.outputs.elasticsearch] Detected a 6.x and above cluster: the `type` event field won't be used to determine the document _type {:es_version=>6}
[2019-12-05T00:45:19,879][WARN ][logstash.outputs.elasticsearch] Restored connection to ES instance {:url=>"http://192.168.111.129:9200/"}
[2019-12-05T00:45:20,156][WARN ][logstash.outputs.elasticsearch] Restored connection to ES instance {:url=>"http://192.168.111.130:9200/"}
[2019-12-05T00:45:20,420][INFO ][logstash.outputs.elasticsearch] New Elasticsearch output {:class=>"LogStash::Outputs::ElasticSearch", :hosts=>["//192.168.111.128:9200", "//192.168.111.129:9200", "//192.168.111.130:9200"]}
[2019-12-05T00:45:20,457][INFO ][logstash.outputs.elasticsearch] Using default mapping template
[2019-12-05T00:45:20,783][INFO ][logstash.outputs.elasticsearch] Attempting to install template {:manage_template=>{"template"=>"logstash-*", "version"=>60001, "settings"=>{"index.refresh_interval"=>"5s"}, "mappings"=>{"_default_"=>{"dynamic_templates"=>[{"message_field"=>{"path_match"=>"message", "match_mapping_type"=>"string", "mapping"=>{"type"=>"text", "norms"=>false}}}, {"string_fields"=>{"match"=>"*", "match_mapping_type"=>"string", "mapping"=>{"type"=>"text", "norms"=>false, "fields"=>{"keyword"=>{"type"=>"keyword", "ignore_above"=>256}}}}}], "properties"=>{"@timestamp"=>{"type"=>"date"}, "@version"=>{"type"=>"keyword"}, "geoip"=>{"dynamic"=>true, "properties"=>{"ip"=>{"type"=>"ip"}, "location"=>{"type"=>"geo_point"}, "latitude"=>{"type"=>"half_float"}, "longitude"=>{"type"=>"half_float"}}}}}}}}
[2019-12-05T00:45:21,205][INFO ][logstash.outputs.elasticsearch] Installing elasticsearch template to _template/logstash
[2019-12-05T00:45:21,285][INFO ][logstash.pipeline        ] Pipeline started successfully {:pipeline_id=>"main", :thread=>"#<Thread:0x779b87f run>"}
The stdin plugin is now waiting for input:
[2019-12-05T00:45:22,435][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
[2019-12-05T00:45:28,362][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
/opt/elk/logstash/vendor/bundle/jruby/2.5.0/gems/awesome_print-1.7.0/lib/awesome_print/formatters/base_formatter.rb:31: warning: constant ::Fixnum is deprecated
{
          "host" => "hadoop-slave1",
      "@version" => "1",
       "message" => "",
    "@timestamp" => 2019-12-04T16:45:22.432Z
}
```



**kibana验证ES**

```bash
GET /logstash_test/_search
```

**返回值**

```json
{
  "took" : 34,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "logstash_test",
        "_type" : "doc",
        "_id" : "9PHN0W4BmHOp3YslWX6L",
        "_score" : 1.0,
        "_source" : {
          "host" : "hadoop-slave1",
          "@version" : "1",
          "message" : "",
          "@timestamp" : "2019-12-04T16:45:22.432Z"
        }
      }
    ]
  }
}
```

