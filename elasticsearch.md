## elasticsearch基本功能
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

## elasticsearch的中文分词插件IK
github上有个很火的中文分词插件IK([地址](https://github.com/medcl/elasticsearch-analysis-ik))，支持中文分词。所谓中文分词，指可以把`软件集团`分词为`软件`和`集团`，而不是按字分为`软`、`件`、`集`、`团`四个。  
下面的测试参考了[这个](http://www.cnblogs.com/phpshen/p/6085274.html)网上的文章。  
### elasticsearch安装
本节介绍用yum安装elasticsearch，并注册为服务。  
```
$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
为elasticsearch创建repo文件(/etc/yum.repos.d/elasticsearch.repo)，内容如下：
```
[elasticsearch-5.x]
name=Elasticsearch repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
然后用yum安装elasticsearch，并注册为服务和启动服务：
```
$ yum -y install elasticsearch
$ chkconfig --add elasticsearch
$ service elasticsearch start
```
elasticsearch默认监听9200端口，可以用curl测试一下：
```
$ curl localhost:9200
{
  "name" : "c261V4t",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "_yEGBQj3TEK6gure_Qsb3Q",
  "version" : {
    "number" : "5.6.1",
    "build_hash" : "667b497",
    "build_date" : "2017-09-14T19:22:05.189Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```
可以看到刚刚安装elasticsearch的版本号是5.6.1。IK插件的版本必须和elasticsearch的版本号一样，这是个很容易被忽略的问题。如果两者版本号不一样，elasticsearch的启动会失败。  
### IK插件的安装
有两种方式获得IK插件：下载java源码自己构建，或直接下载构建好的插件。  
自己构建插件需要的软件有git、jdk、maven：
```
$ cd /opt
$ git clone https://github.com/medcl/elasticsearch-analysis-ik
$ git checkout tags/v5.6.1
```
如果你的elasticsearch的版本号不是v5.6.1，那么一定要用`git checkout`将版本号切换到你的elasticsearch的版本号。  
然后进行构建：
```
$ cd /opt/elasticsearch-analysis-ik
$ mvn clean
$ mvn compile
$ mvn package
```
耗时较长（几十分钟）。构建的结果是`target/releases/elasticsearch-analysis-ik-5.6.1.zip`。  
需要把构建出来的插件复制到elasticsearch目录下。可以用下列命令寻找elasticsearch的安装目录：
```
$ whereis elasticsearch
elasticsearch: /etc/elasticsearch /usr/share/elasticsearch
```
`/usr/share/elasticsearch`就是elasticsearch的安装目录，插件要安装到`/usr/share/elasticsearch/plugins`目录下。只有通过yum安装的软件包才可以用`whereis`命令来定位，当然象第一章手工部署的elasticsearch本来就知道它的位置。  
下面把插件的zip文件复制到elasticsearch的插件目录下，并解压：
```
$ cd /opt/elasticsearch-analysis-ik
$ cp target/releases/elasticsearch-analysis-ik-5.6.1.zip  /usr/share/elasticsearch/plugins
$ cd /usr/share/elasticsearch/plugins
$ unzip elasticsearch-analysis-ik-5.6.1.zip
$ mv elasticsearch ik
```
可能跟配置有关，在我的测试环境下，unzip后自动创建了一个叫`elasticsearch`的目录，把它改名为`ik`。不管怎么说，插件目录大约这样：
```
$ ls -l /usr/share/elasticsearch/plugins/ik
-rw-r--r--. 1 root root 263965 Sep 20 06:15 commons-codec-1.9.jar
-rw-r--r--. 1 root root  61829 Sep 20 06:15 commons-logging-1.2.jar
drwxr-xr-x. 2 root root   4096 Sep 20 07:09 config
-rw-r--r--. 1 root root  51393 Sep 20 07:21 elasticsearch-analysis-ik-5.6.1.jar
-rw-r--r--. 1 root root 736658 Sep 20 06:15 httpclient-4.5.2.jar
-rw-r--r--. 1 root root 326724 Sep 20 06:15 httpcore-4.4.4.jar
-rw-r--r--. 1 root root   2666 Sep 20 07:21 plugin-descriptor.properties
```
除了自己用maven构建插件，还可以到这个地址去下载：https://github.com/medcl/elasticsearch-analysis-ik/releases。  
需要注意的是下载的包的版本号必须与elasticsearch的版本一致。在https://github.com/medcl/elasticsearch-analysis-ik首页上提到的`elasticsearch-plugin install`安装插件的办法经测试不行。    

### IK插件的测试
重启elasticsearch服务：
```
$ service elasticsearch restart
$ service elasticsearch status
```
如果有错误，可以查看elasticsearch日志，路径一般是`/var/log/elasticsearch/elasticsearch.log`。安装IK插件后，在日志文件的最后可以看到：
```
[2017-09-20T07:25:52,374][INFO ][o.e.p.PluginsService     ] [c261V4t] loaded plugin [analysis-ik]
```
在第一章中是使用`./elasticsearch`直接运行的elasticsearch，上述日志信息被直接打印到屏幕上了。  

下面对IK插件的功能进行测试。首先创建一个新的`m8`索引。第一章的`books`索引貌似不能直接用。
```
$ curl -XPUT 'http://localhost:9200/m8'
$ curl -XPUT localhost:9200/m8 -d '{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "ik" : {
                    "tokenizer" : "ik_smart"
                }
            }
        }
    },
    "mappings" : {
        "logs" : {
            "dynamic" : true,
            "properties" : {
                "message" : {
                    "type" : "string",
                    "analyzer" : "ik_smart"
                }
            }
        }
    }
}'
```
刚刚创建的m8下虽然没有添加任何索引，但可以用来测试分词功能：
```
$ curl 'http://localhost:9200/m8/_analyze?pretty&analyzer=ik_smart' -d '南京市长江大桥'
{
  "tokens" : [
    {
      "token" : "南京市",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "长江大桥",
      "start_offset" : 3,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 1
    }
  ]
}
```
IK支持的分词方式有两种：  
- ik_max_word: 会将文本做最细粒度的拆分  
- ik_smart: 会做最粗粒度的拆分  

下面是`ik_max_word`的测试：
```
$ curl 'http://localhost:9200/m8/_analyze?pretty&analyzer=ik_max_word' -d '南京市长江大桥'
{
  "tokens" : [
    {
      "token" : "南京市",
      "start_offset" : 0,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "南京",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "市长",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "长江大桥",
      "start_offset" : 3,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "长江",
      "start_offset" : 3,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "大桥",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 5
    }
  ]
}
```
### IK的搜索测试
安装https://github.com/medcl/elasticsearch-analysis-ik首页的测试数据：
1.创建索引，`index`已经被占用了，只好用`index2`：
```
$ curl -XPUT http://localhost:9200/index2
```
2.创建映射
```
$ curl -XPOST http://localhost:9200/index2/fulltext/_mapping -d'
{
        "properties": {
            "content": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_max_word"
            }
        }
    
}'
```
3.为一些文档创建索引
```
$ curl -XPOST http://localhost:9200/index2/fulltext/1 -d'
{"content":"美国留给伊拉克的是个烂摊子吗"}`
$ curl -XPOST http://localhost:9200/index2/fulltext/2 -d'
{"content":"公安部：各地校车将享最高路权"}'
$ curl -XPOST http://localhost:9200/index2/fulltext/3 -d'
{"content":"中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"}'
$ curl -XPOST http://localhost:9200/index2/fulltext/4 -d'
{"content":"中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"}'
```
4.使用highlighting查询
```
$ curl -XPOST http://localhost:9200/index2/fulltext/_search  -d'
{
    "query" : { "match" : { "content" : "中国" }},
    "highlight" : {
        "pre_tags" : ["<tag1>", "<tag2>"],
        "post_tags" : ["</tag1>", "</tag2>"],
        "fields" : {
            "content" : {}
        }
    }
}'
```
结果：
```
{"took":131,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":2,"max_score":0.6099695,"hits":[{"_index":"index2","_type":"fulltext","_id":"4","_score":0.6099695,"_source":
{"content":"中国驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"}
,"highlight":{"content":["<tag1>中国</tag1>驻洛杉矶领事馆遭亚裔男子枪击 嫌犯已自首"]}},{"_index":"index2","_type":"fulltext","_id":"3","_score":0.27179778,"_source":
{"content":"中韩渔警冲突调查：韩警平均每天扣1艘中国渔船"}
,"highlight":{"content":["中韩渔警冲突调查：韩警平均每天扣1艘<tag1>中国</tag1>渔船"]}}]}}
```
注意上面查询结果中的高亮标签`<tag1>中国</tag1>`。前台js可以针对`<tag1>`标签中内容加上类似黄色背景的高亮效果。  

#### IK的自定义词典
[原文](https://www.2cto.com/database/201611/560699.html)
来到IK插件的配置文件目录，该目录下有很多.dic文件，是IK的字典文件。在这个目录下创建`ik`目录，然后把当前目录下的`IKAnalyzer.cfg.xml`复制到新建的ik目录下，编辑`IKAnalyzer.cfg.xml`，添加远程字典：
```
$ cd /usr/share/elasticsearch/plugins/ik/config
$ mkdir ik
$ cp IKAnalyzer.cfg.xml ik
$ cat ik/IKAnalyzer.cfg.xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 -->
        <entry key="ext_dict"></entry>
         <!--用户可以在这里配置自己的扩展停止词字典-->
        <entry key="ext_stopwords"></entry>
        <!--用户可以在这里配置远程扩展字典 -->
        <entry key="remote_ext_dict">https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/test2.php</entry>
        <!--用户可以在这里配置远程扩展停止词字典-->
        <!-- <entry key="remote_ext_stopwords"></entry> -->
</properties>
```
test2.php是放在github上的一个文件，内容是：
```
$s = <<<'EOF'
陈港生
元楼
蓝瘦
EOF;
header('Last-Modified: '.gmdate('D, d M Y H:i:s', time()).' GMT', true, 200);
header('ETag: "5816f349-19"');
echo $s;
```
重启elasticsearch，然后测试一下：
```
$ service elasticsearch restart
$ curl -XGET 'http://localhost:9200/_analyze?pretty&analyzer=ik_max_word' -d '
成龙原名陈港生'
{
  "tokens" : [
    {
      "token" : "成龙",
      "start_offset" : 1,
      "end_offset" : 3,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "原名",
      "start_offset" : 3,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "陈港生",
      "start_offset" : 5,
      "end_offset" : 8,
      "type" : "CN_WORD",
      "position" : 2
    }
  ]
}
```
`陈港生`本来不是一个词，增加了自定义远程字典后背elasticsearch当成一个词了。自定义的远程词典每分钟被IK加载一次，即支持动态更新词典而不用重启elasticsearch服务。  

## 其它
[拼音分词器](http://blog.csdn.net/napoay/article/details/53907921)、[Elasticsearch服务器开发（第2版）.pdf](http://wtdown.2cto.com/ware/E-book/2016512/Elasticsearch_14.5MB.rar)    