# elasticsearch整合Ik分词

## IK简介

ElasticSearch（以下简称ES）默认的分词器是标准分词器Standard，如果直接使用在处理中文内容的搜索时，中文词语被分成了一个一个的汉字，因此引入中文分词器IK就能解决这个问题，同时用户可以配置自己的扩展字典、远程扩展字典等。

## IK安装

github 下载与es相同版本的zip安装包[https://github.com/medcl/elasticsearch-analysis-ik/releases](https://github.com/medcl/elasticsearch-analysis-ik/releases)

```shell
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.7.1/elasticsearch-analysis-ik-6.7.1.zip
```

```shell
cd /opt/elk/es/plugins
# 创建目录 analysis-ik
mkdir analysis-ik
cd analysis-ik
# 解压到/opt/elk/es/plugins 目录
unzip elasticsearch-analysis-ik-6.7.1.zip
```

**scp到其他服务器**

```shell
scp -r /opt/elk/es/plugins/analysis-ik root@192.168.111.129:/opt/elk/es/plugins
scp -r /opt/elk/es/plugins/analysis-ik root@192.168.111.130:/opt/elk/es/plugins
```

**重启es集群**

```shell
su es
cd /opt/elk/es/bin
./elasticsearch -d
```

## 验证IK分词

ik插件提供了两种分词模式：**ik_max_word**和**ik_smart**：

**ik_max_word**：会将文本做最细粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国，中华人民，中华，华人，人民共和国，人民，人民，共和国，共和，和，国国，国歌”，会穷尽各种可能的组合;

**ik_smart**：会做最粗粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国，国歌”。 



使用**kibana**测试

