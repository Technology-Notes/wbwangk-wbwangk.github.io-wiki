电商大数据平台[官方博客](https://imaidata.github.io/blog)

github wiki中自动添加目录的chrome插件: [Google Chrome / Chromium](https://chrome.google.com/webstore/detail/github-toc/nalkpgbfaadkpckoadhlkihofnbhfhek)
### yum手工下载软件包
[参考](http://www.360doc.com/content/14/0306/23/13084517_358376529.shtml)  
### Elasticsearch文档
[elasticsearch中文权威指南.pdf](http://wtdown.2cto.com/ware/E-book/2016512/Elasticsearch_14.5MB.rar)
[ElasticSearchManual.pdf](http://116.224.87.15/file3.data.weipan.cn/22935107/c31179b32e8e4a82a96502904b5b1626051f2162?ip=1505807323,58.56.96.29&ssig=tks4zViV4k&Expires=1505807923&KID=sae,l30zoo1wmz&fn=ElasticSearchManual.pdf&skiprd=2&se_ip_debug=58.56.96.29&corp=2&from=1221134&wsiphost=local)
### http_load
[home](https://acme.com/software/http_load/)  
```
$ cd /opt
$ wget https://acme.com/software/http_load/http_load-09Mar2016.tar.gz
$ tar xzvf http_load-09Mar2016.tar.gz
$ cd  http_load-09Mar2016
$ make
$ make install
```
编辑一个文本文件，如t.tt，内容：
```
http://localhost:8080/
```
然后运行
```
$ http_load -p 10 -s 5 t.tt
94720 fetches, 10 max parallel, 1.8944e+06 bytes, in 5 seconds
20 mean bytes/connection
18944 fetches/sec, 378880 bytes/sec
msecs/connect: 0.0958101 mean, 0.772 max, 0.014 min
msecs/first-response: 0.293987 mean, 0.988 max, 0.232 min
HTTP response codes:
  code 200 -- 94720
```
`-p`是并发用户数，'-s'是执行时长。  

### What is HMAC Authentication and why is it useful?
[here](https://www.wolfe.id.au/2012/10/20/what-is-hmac-authentication-and-why-is-it-useful/)  

### java base64
[原文](http://www.importnew.com/14961.html)  
```
// 编码
String asB64 = java.util.Base64.getEncoder().encodeToString("some string".getBytes("utf-8"));
// 解码
byte[] asBytes = java.util.Base64.getDecoder().decode("c29tZSBzdHJpbmc=");
// URL编码
String urlEncoded = java.util.Base64.getUrlEncoder().encodeToString("subjects?abcd".getBytes("utf-8"));
```
### Confluent REST Proxy for Kafka
https://github.com/confluentinc/kafka-rest

### 浏览器中的sql笔记本 Franchise
https://franchise.cloud

### A collection of useful resources for building RESTful HTTP+JSON APIs.
https://github.com/yosriady/api-development-tools

### awesome useful-java-links
https://github.com/Vedenin/useful-java-links

### Awesome_APIs 包括国内API服务清单
https://github.com/TonnyL/Awesome_APIs