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

### awesome rest
https://github.com/marmelab/awesome-rest

### vagrant配置端口转发
```
Vagrant.configure("2") do |config|
  config.vm.network "forwarded_port", guest: 80, host: 8080
end
```
### openstack all in one
在(https://app.vagrantup.com/boxes/search)中搜索openstack发现了stackinabox/openstack这个vagrant box。它号称将这个openstack装入了这一个盒子中，操作系统是ubuntu16。

### Blockchain Demo
https://anders.com/blockchain/

### openssl sha256
```
$ openssl dgst -sha256
dddd(stdin)= 5bf8aa57fc5a6bc547decf1cc6db63f10deb55a3c6c5df497d631fb3d95e1abf
```
dddd是要进行hash的文本，之后是组合键ctrl+d，而且要多按几次。dddd之后不要回车，否则回车也会被加入到hash输入中。
### html2pdf
http://www.pdfonfly.com/

### docker远程管理
centos:`/etc/sysconfig/docker`，ubunut:`/etc/default/docker`:
```
DOCKER_OPTS="$DOCKER_OPTS -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375 --api-cors-header='*'"
```
### PDF2MD
http://pdf2md.morethan.io/

### 供应链可追溯性/防伪
https://wiki.hyperledger.org/_media/groups/requirements/hyperledger_-_supply_chain_traceability-_anti_counterfeiting.pdf

### Tutorial: Hyperledger Fabric v1.1 – Create a Development Business Network on zLinux
https://github.com/CATechnologies/blockchain-tutorials/wiki/Tutorial:-Hyperledger-Fabric-v1.1-%E2%80%93-Create-a-Development-Business-Network-on-zLinux#toc9

### git proxy
```
git config --global https.proxy http://127.0.0.1:1080
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --global --unset all.proxy
```
### npm install 对目录没有权限
```
$ npm install --unsafe-perm --verbose -g composer-cli
```
### VM proxy
当VM需要代理才能访问互联网时，在VM内执行：
```
$ export http_proxy=<宿主机ip:代理端口号>
$ export https_proxy=<宿主机ip:代理端口号>
```
### fabric 翻译项目
https://lists.hyperledger.org/pipermail/hyperledger-twg-china/2017-November/000419.html

### mkdoc
https://github.com/HyperledgerCN/hyperledgerDocs/blob/master/docs/index.md  
先[安装pip](http://pip.readthedocs.io/en/latest/installing/)  
```
pip install mkdocs  
pip install python-markdown-math
git clone https://github.com/HyperledgerCN/hyperledgerDocs
cd hyperledgerDocs
mkdocs build
mkdocs serve  --dev-addr 192.168.16.103:8000
```
用浏览器访问192.168.16.103:8000即可。build时曾发现缺了mdx_math扩展，安装python-markdown-math后解决。