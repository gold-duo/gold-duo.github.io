---
title: 0407Elasticsearch
date: 2023-09-17 22:00
categories: [python,网络爬虫开发实战]
tags: [python,网络爬虫开发实战,Elasticsearch,spider] 
---

ElasticSearch (p159)是一个分布式的，高性能，高可用的，可伸缩的搜索和分析系统

## ElasticSearch特点
- (1)可以作为大型分布式集群(数百台服务器)技术，处理 PB 级的数据，服务大公司；也可以运行在单机上服务于小公司
- (2)Elasticsearch 不是什么新技术，主要是将全文检索、数据分析以及分布式技术，合并在了一起，才形成了独一无二的 ES：lucene (全文检索)，商用的数据分析软件，分布式数据库
- (3)对用户而言，是开箱即用的，非常简单，作为中小型应用，直接 3 分钟部署一下 ES，就可以作为生产环境的系统来使用了，此时的场景是数据量不大，操作不是太复杂
- (4)数据库的功能面对很多领域是不够用的(事务，还有各种联机事务型的操作)；特殊的功能，比如全文检索，同义词处理，相关度排名，复杂数据分析，海量数据的近实时处理，Elasticsearch 作为传统数据库的一个补充，提供了数据库所不能提供的很多功能。

## 安装：
- 直接 [下载](https://www.elastic.co/cn/downloads/elasticsearch)压缩包解压，运行 `bin/elasticsearch`启动。浏览器打开`http://localhost:9200/`验证是否正常（显示json信息）

```python
#运行浏览器控制台出现"received plaintext http traffic on an https channel, closing connection Netty4HttpChannel{localAddress=/[0:0:0:0:0:0:0:1]:9200"
# 打开 config/elasticsearch.yml 关闭SSL认证,将 xpack.security.enabled 设置成 false
```
- 安装pyton库:`pip3 install elasticsearch`

## Elasticssearch的概念

概念|描述
---|---
节点(Node)|单个Elasticssearch实例称为节点
集群(Cluster)|一组节点构成集群
索引(index)|Elasticssearch管理的顶层单位就叫做索引(相当于MySQL、MongoDB数据库的概念)。ES查找数据直接查找索引
~~类型(Type)~~|~~类似MySQL的表、MongoDB中的集合~~( 7.X版本中去除type的概念)
文档(documnet)|索引里的单条记录称为文档(类似MySQL中的行)
字段(field)|多个字段组成一个文档(类似MySQL中的字段)


### 用关系型数据库的类比Elasticssearch:

关系型数据库| DB|表(Tables)|行(Rows)|列(Columns)
 ---|---|---|---|---
Elasticssearch|索引(indices)|~~类型(Types)~~|文档(Documents)|字段(Fields)

## 使用
### 链接Elasticssearch

```python
from elasticsearch import Elasticsearch
es = Elasticsearch("httpss://localhost:9200", verify_certs=True)#强制SSL验证,注意"httpss".httpss://[user:pwd^]hostname:port
```

### CRUD

操作|实例|描述|其他
---|---|---|---
创建index|es.<span style="color:red">indices.create</span>(index='news',ignore=400)||如果创建失败返回的`status=400`表示已经存在，ignore=400则忽略改错误否则抛异常
删除index|es.<span style="color:red">indices.delete</span>(index='news',ignore={400,404})||<span style='white-space:nowrap;'>404不存在（http code套进来了？）</span>
插入数据|js={'title':'gold-duo','url':'https://github.com/gold-duo/notes'}<br/>es.<span style="color:red">create</span>(index='news',id=1,body=js)<br/><br/>es.<span style="color:red">index</span>(index='news', body=data)|id唯一标识(无则自动生成)，body代表文档的内容|es.create调用的是es.index
更新数据|es.<span style="color:red">update</span>(index='news',doc=new_data,id=1)</font><br/>es.index(index='news', body=new_data, id=1)||如上es.index既可插入又可更新，所以当数据不存在则更新
查询数据|dsl={'query': {'match': {'title': '高考 圆梦'} } }<br/>es.search(index='news',body=dsl)|返回json:<br/>hits.total.value:匹配条数<br/>hits.max_score:最大匹配分数<br/>hits.hits:匹配的数据|全文检索，所以这里设置的'高考 圆梦'，就是包含'高考'或者'圆梦'的数据<br/>查询[DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
获取数据|es.get(index="news",  id=1)|直接返回id=1的数据|

### 设置全文检索分词器(p164)

```python
# 1.安装分词插件（注意对应elasticsearch版本的插件版本）
#   elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.11.3/elasticsearch-analysis-ik-8.11.3.zip

# 2.重启 elasticsearch

# 3.设置index的mapping
mapping = {
    'properties': {
        'title': {
            'type': 'text',                 #字段类型
            'analyzer': 'ik_max_word',      #分词器
            'search_analyzer': 'ik_max_word'#搜索分词器，不指定默认用英文分词
        }
    }
}
es.indices.put_mapping(index='news', body=mapping)
```