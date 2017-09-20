### 安装
[官方参考](https://www.elastic.co/guide/en/elasticsearch/reference/current/_installation.html)  
安装环境是centos7.3(c7303,/e/vagrant10):  
```
$ cd /opt
$ curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.1.tar.gz
$ tar -xvf elasticsearch-5.6.1.tar.gz
$ sudo useradd elastic -d /home/elastic
$ sudo chown -R elastic:elastic /opt/elasticsearch-5.6.1
$ sudo su - elastic
$ cd /opt
$ cd elasticsearch-5.6.1/bin
$ ./elasticsearch
```
elasticsearch不能用root运行，所以先创建用户elastic。  
elasticsearch的REST API规则是：`<host>:<port>/<index>/<type>/<id>?<参数>`。  

### 索引创建
#### 新增或更新索引(指定索引id)
```
$ curl -XPUT 'localhost:9200/books/es/1' -d '{"title":"ES Server", "published": 2013}'
{"_index":"books","_type":"es","_id":"1","_version":2,"_shards":{"total":2,"successful":1,"failed":0},"created":false}
$ curl -XPUT 'localhost:9200/books/es/1' -d '{"title":"ES Server2", "published": 2013}'
{"_index":"books","_type":"es","_id":"1","_version":3,"_shards":{"total":2,"successful":1,"failed":0},"created":false}
```
两次PUT方法都指定了索引id，第二次调用会更新第一次的内容。
#### 新增索引(自动id)
```
$ curl -XPOST 'localhost:9200/books/es' -d '{"title":"ES Server3", "published": 2013}'
{"_index":"books","_type":"es","_id":"AV6ZtiQ4wIGFABcqZVLx","_version":1,"_shards":{"total":2,"successful":1,"failed":0},"created":true}
$ curl -XPUT 'localhost:9200/books/es/AV6ZtiQ4wIGFABcqZVLx' -d '{"title":"ES Server4", "published": 2013}'
{"_index":"books","_type":"es","_id":"AV6ZtiQ4wIGFABcqZVLx","_version":2,"_shards":{"total":2,"successful":1,"failed":0},"created":false}
```
自动产生id需要使用**POST**方法。第二次调用更新索引内容，使用了POST方法。

#### 删除索引
```
$ curl -XDELETE 'localhost:9200/books/es/AV6ZtiQ4wIGFABcqZVLx'
{"found":true,"_index":"books","_type":"es","_id":"AV6ZtiQ4wIGFABcqZVLx","_version":3,"_shards":{"total":2,"successful":1,"failed":0}}
```
### 搜索
创建测试数据：
```
$ curl -XPOST 'localhost:9200/books/solr' -d '{"title":"软件集团", "published": 2013}'
$ curl -XPOST 'localhost:9200/books/solr' -d '{"title":"集团软件", "published": 2013}'
$ curl -XPOST 'localhost:9200/books/solr' -d '{"title":"软 件 集 团", "published": 2014}'
$ curl -XPOST 'localhost:9200/books/solr' -d '{"title":"软件集团SBG", "published": 2014}'
```
搜索所有数据：
```
$ curl -XGET 'localhost:9200/books/_search?pretty'
```
为了搜索汉字，需要利用CURL的`--data-urlencode`参数：
```
$ curl -G -v "http://localhost:9200/books/_search?pretty" --data-urlencode "q=title:软件集团"
> GET /books/_search?pretty&q=title%3A%E8%BD%AF%E4%BB%B6%E9%9B%86%E5%9B%A2 HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:9200
> Accept: */*
{
  "took" : 18,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 4,
    "max_score" : 1.7378285,
    "hits" : [
      {
        "_index" : "books",
        "_type" : "solr",
        "_id" : "AV6crBYj4-tVne1_q8tf",
        "_score" : 1.7378285,
        "_source" : {
          "title" : "软件集团",
          "published" : 2013
        }
      },
      {
        "_index" : "books",
        "_type" : "solr",
        "_id" : "AV6crC984-tVne1_q8tg",
        "_score" : 1.7378285,
        "_source" : {
          "title" : "集团软件",
          "published" : 2013
        }
      },
      {
        "_index" : "books",
        "_type" : "solr",
        "_id" : "AV6crEZJ4-tVne1_q8th",
        "_score" : 1.1507283,
        "_source" : {
          "title" : "软 件 集 团",
          "published" : 2014
        }
      },
      {
        "_index" : "books",
        "_type" : "solr",
        "_id" : "AV6crFtz4-tVne1_q8ti",
        "_score" : 1.1299736,
        "_source" : {
          "title" : "软件集团SBG",
          "published" : 2014
        }
      }
    ]
  }
}
```
_score是匹配程度得分，搜索`软件集团`的_score总结如下：  

score  |  文档  
------- | -------  
1.7378285 | 软件集团  
1.7378285 | 集团软件  
1.1507283 | 软 件 集 团  
1.1299736 | 软件集团SBG  

从上面的结果看，Elasticsearch的默认分词器(tokenizer)将每个汉字视为一个单词。即使在汉字之间加上空格，不影响搜索结果。