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
https://wiki.hyperledger.org/groups/twgc/team_ie

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

### hyperledger wiki
https://wiki.hyperledger.org/projects/fabric

### Deploying to a multi-organization Hyperledger Fabric
https://hyperledger.github.io/composer/unstable/tutorials/deploy-to-fabric-multi-org.html

### 跨链
[quilt](https://github.com/hyperledger/quilt)Hyperledger Quilt - An implementation of the Interledger Protocol  

### 区块链案例
Everledger：钻石追溯
Ascribe：艺术品、论文等版权保护
mediachain：创意保护
monegraph：编辑、博客等内容所有者保护作品
Chronicled.com: 供应链追溯

### 共识机制，可编程的“利益”转移规则
[一本书](http://bitcoin-on-nodejs.ebookchain.org/1-%E4%BA%86%E8%A7%A3%E5%8A%A0%E5%AF%86%E8%B4%A7%E5%B8%81/7-%E5%85%B1%E8%AF%86%E6%9C%BA%E5%88%B6%EF%BC%8C%E5%8F%AF%E7%BC%96%E7%A8%8B%E7%9A%84%E2%80%9C%E5%88%A9%E7%9B%8A%E2%80%9D%E8%BD%AC%E7%A7%BB%E8%A7%84%E5%88%99.html)
POW,POS,DPOS


### composer-rest-server认证数据持久化
[链接](https://github.com/hyperledger/composer/blob/master/packages/composer-website/jekylldocs/integrating/deploying-the-rest-server.md)  

### passport-ldap
[1](https://stackoverflow.com/questions/45734046/using-passport-http-on-hyperledger-composer-rest-api)  
```
  export COMPOSER_PROVIDERS='{
  "basic": {
    "provider": "basic",
    "module": "passport-http",
    "clientID": "REPLACE_WITH_CLIENT_ID",
    "clientSecret": "REPLACE_WITH_CLIENT_SECRET",
    "authPath": "/auth/local",
    "callbackURL": "/auth/local/callback",
    "successRedirect": "/",
    "failureRedirect": "/login" 
     }
    }'
```
[2](https://github.com/hyperledger/composer/issues/2065)  
这个网页讲到调用的URL。  
```
export COMPOSER_PROVIDERS='{ "ldap": { "provider": "ldap", "authScheme": "ldap", "module": "passport-ldapauth", "authPath": "/auth/ldap", "successRedirect": "/", "failureRedirect": "/failure", "session": false, "json": true, "profileAttributesFromLDAP": { "login": "uid", "username": "uid", "displayName": "givenName", "email": "mail", "externalId": "uid" }, "server": { "url": "ldap://ldap.xxx.com:389", "bindDn": "uid=aldred,ou=Users,o=xxx,dc=xxx,dc=com", "bindCredentials": "caveman", "searchBase": "ou=Users,o=xxx,dc=xxx,dc=com", "searchFilter": "(uid={{username}})" } } }'
```

### docker.sock permission denied
```
sudo usermod -a -G docker $USER
```
当前用户需要重新登录，然后执行docker ps就正常了。

#### p2p ssh
https://github.com/nobonobo/ssh-p2p

### 数字资产保护
https://www.ascribe.io
文档：C:\Users\wbwang\Desktop\区块链\oreilly-blockchain-by-melanie-swan.pdf

### blockchain demo
https://github.com/anders94/blockchain-demo

[![](https://camo.githubusercontent.com/a9e81eef2ba0dfa94a66c2200569cb0c00bf3eb2/68747470733a2f2f696d672e796f75747562652e636f6d2f76692f5f3136306f4d7a626c59382f302e6a7067)](https://www.youtube.com/watch?v=_160oMzblY8)

### Token

Token是一种数字化的价值载体，是权益证明。加密的，可流通。[1](https://www.jianshu.com/p/981689784579)

区块链的 token被广泛认识，归功于以太坊及其订立的 ERC20标准。基于这个标准，任何人都可以在以太坊上发行自定义的 token，这个token可以代表任何权益和价值。现在用 token来作为代币权益证明进行ICO是一个普遍的做法。...今天我想提出一个建议，就是把 token 翻译成为“通证”，意思是就是“可流通的加密数字权益证明”。... “通证”三要素：第一是数字权益证明；第二是加密；第三是可流通。...有必要放在区块链上的程序，目前看就是涉及价值交换、权益管理之类的应用，或者说的更直白一点，就是涉及通证的应用。...通证启发和鼓励大家把各种权益证明，比如门票、积分、合同、证书、点卡、证券、权限、资质等等全部拿出来通证化（tokenization），放到区块链上流转，放到市场上交易，让市场自动发现其价格，同时在现实经济生活中可以消费、可以验证，是可以用的东西，这是紧贴实体经济的。[2](http://blog.csdn.net/myan/article/details/78712506)。

### Enigma
Enigma被认为是一个保护用户隐私的项目，因为内容不再上传到云服务商手里。麻省理工的一个项目，用于文件加密分享。与迅雷玩客币机制类似，奖励给帮其他用户存储的人。用比特币支付。  
Enigma系统将数据分解成碎片，然后使用一些巧妙的数学方法对这些数据进行掩盖。单独从每一碎片获知数据是不可能的，你不可能重新获得有关的原始数据。  

### 有用的blockchain资源（Blockchain-stuff）
https://github.com/Xel/Blockchain-stuff

### blockchain场景
参考了[Blockchain-stuff](https://github.com/Xel/Blockchain-stuff)

#### LBRY
LBRY是一个标准，虽然提供了参考实现。
https://github.com/lbryio/lbry：LBRY is a free, open, and community-run digital marketplace.
LBRY是第一个由市场参与者控制的数字市场。去中心的内容分享，支持付费获取内容，社区控制。
保护隐私vs恐怖分子的隐私；开车旅行很奇妙vs每天撞死几百万人；
和BitTorrent相比：发布和购买行为被记录；侵权内容的URL可以被更改（原理不明）；
不收取租金：没有必要给予45％的YouTube或30％的苹果。

#### artlery
art-backed加密货币(CLIO)，致力于艺术家分享作品并获得收益。

#### ascribe.io
可以上传文件，获得保护。Lock in attribution, securely share and trace where your digital work spreads.

#### bitmark
BITMARK GIVES YOU THE TOOLS TO TURN YOUR DIGITAL ASSETS INTO CRYPTO PROPERTY AND ENTER THE DATA ECONOMY.
bitmark交给你一个工具，将你的数字资产变成加密财产并进入数字经济。
(也是上传文件，生成数字指纹)
bitmark依赖IFTTT进行数字资产的共享、分发、交易。

#### binded
Copyright made simple。The easy way to protect your images, free forever.

#### everledger.io
钻石保真。Everledger成功的背后一半核心是区块链技术，另外一半则是宝石指纹技术。

### EthList：Ethereum Reading List
https://github.com/Scanate/EthList

### 区块链主流开源技术体系介绍
http://www.8btc.com/elwingao-blockchain-6

### 解决区块链三大问题的利器
http://www.8btc.com/elwingao-blockchain-7  
[同态加密](https://www.zhihu.com/question/27645858)  

### 量子链Qtum
量子链Qtum是中国社区原创的区块链公链。Qtum通过价值传输协议（Value Transfer Protocol）来实现点对点的价值转移，并根据此协议，构建一个支持多个行业的（金融、物联网、供应链、社交游戏等）去中心化的应用开发平台（DAPP Platform）。

量子链的定位是，做一个符合行业监管的区块链去中心化应用开发平台。[1](http://www.8btc.com/elwingao-blockchain-5)  

### 区块链应用开发入门
[1](http://www.8btc.com/elwingao-blockchain-1)
####  比特币
1. 基于Blockchain.info的API进行开发  
2. 采用Docker容器来快速安装和配置私有节点的比特币测试网络(bitcoin-testnet)作为开发试验环境  

### p2p网络原理
https://github.com/FactomProject/factomd/tree/5ea4f88d9019885aa89d933f4b8987fccbae1987/p2p

### 区块链笔记
http://book.8btc.com/books/1/master_bitcoin/_book/

一枚电子货币（an electronic coin）是这样的一串数字签名：每一位所有者通过对前一次交易和下一位拥有者的公钥(Public key) 签署一个随机散列的数字签名，并将这个签名附加在这枚电子货币的末尾，电子货币就发送给了下一位所有者。而收款人通过对签名进行检验，就能够验证该链条的所有者。
#### REST查询比特币钱包的例子
[1](http://book.8btc.com/books/1/master_bitcoin/_book/2/2.html)
查找Alice的比特币地址所有的未消费的输出:
```
$ curl https://blockchain.info/unspent?active=1Cdid9KFAaatwczBwBttQcwXYCpvK8h7FK
{
  "unspent_outputs": [
    {
      "tx_hash":"186f9f998a5...2836dd734d2804fe65fa35779",
      "tx_index":104810202,
      "tx_output_n":0,
      "script":"76a9147f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a888ac",
      "value":10000000,
      "value_hex":"00989680",
      "confirmations":0
    }
  ]
}
```
[点击查看Joe和Alice间的交易信息](https://blockchain.info/tx/7957a35fe64f80d234d76d83a2a8f1a0d8149a41d81de548f0a65a8a999f6f18)

#### 交易输出是个脚本
Alice的交易输出会包含一个脚本，这个脚本说 “这个输出谁能拿出一个签名和Bob的公开地址匹配上，就支付给谁”
#### Merkle树
Merkle树是一种哈希二叉树，它是一种用作快速归纳和校验大规模数据完整性的数据结构。  
H~A~ = SHA256(SHA256(交易A))  
交易HK由4个哈希值（在图7-5中由蓝色标注）HL、HIJ、HMNOP和HABCDEFGH组成的路径证明其存在。  
![](http://book.8btc.com/books/1/master_bitcoin/_book/7/Fig705.png)

### 以太坊官网文档中文版
[文档](http://book.8btc.com/books/6/ethereum/_book/)  
以太坊官网：https://ethereum.org/  


[1](http://book.8btc.com/books/6/ethereum/_book/ethereum-account-management.html)如果我们把以太坊限制为只有外部账户，只允许外部账户之间进行交易，我们就会进入到"代币"系统，"代币"系统不如比特币本身有力，只能用于转移以太币。

#### go客户端
[1](https://ethereum.github.io/go-ethereum/install/)
```
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum
```

EthStats.net是以太坊网络实时数据的仪表板  

#### geth使用
新建账号和看清单：
```
geth account new
geth account list
```
进入控制台：
```
geth console 2>> file_to_log_output
> eth.accounts
["0xa654c3043227997abf38e16e2cb071b480dfe603"]
> net.listening
true
> net.peerCount
4
```
#### 以太坊浏览器
https://ethstats.net/

#### 智能合约Solidity编程语言
http://www.tryblockchain.org/

#### 以太坊HelloWorld学习
https://github.com/littleredhat1997/Ethereum

#### 七节以太坊教程
本教程一共有七节课，主要是通过Nodejs来开发基于以太坊的Dapp应用教学，功能简单，容易上手，为你打开一个除了truffle的另一个应用开发模式。  

### IBM区块链平台
https://console.bluemix.net/docs/services/blockchain/index.html

### linux看硬盘空间
`df -h`看整个VM的存储使用情况，单位是字节；`du .`查看某目录占用的空间大小，单位是k。

### 手把手教你发行自己的以太坊ERC20 Token
[1](http://blog.csdn.net/sinat_34070003/article/details/79107181)
安装chrome插件metamask，连接以太币测试网Ropsten Test Net，领了4个ETH。  

#### 以太坊浏览器
https://ropsten.etherscan.io/

### 杨海勇做的composer-playground多租户POC
https://note.youdao.com/share/?id=428fa61d01b21264f68067b56b995246&type=note#/

### webpack 网页依赖管理和打包工具
[官方快速开始](https://webpack.js.org/guides/getting-started/)  

###  智能合约中存在的3种最常见的误解
http://blog.csdn.net/wo541075754/article/details/79061040

### Swarm vs IPFS
https://ethereum.stackexchange.com/questions/2138/what-is-the-difference-between-swarm-and-ipfs

### 区块链向Web开发人员解释，第1部分：理论
https://marmelab.com/blog/2016/04/28/blockchain-for-web-developers-the-theory.html  
在编写《区块链入门》时引用了该文章的文字和图片  

### 隔离见证 (Segregated Witness)
https://www.zhihu.com/question/58567061  

而隔离见证 (Segregated Witness，以下简称SW) ，是由比特币核心开发员Pieter Wuille 在2015年12月於香港提出的软分叉非常巧妙地彻底解决了这个问题（在交易发出後，确认前的交易ID可以被任意更改）。SW用户在交易时，会把比特币传送到有别於传统的地址。当要使用这些比特币的时候，其签署 (即见证)并不会记录为交易ID的一部份，而是另外处理。也就是说，交易ID完全是由交易状态 (即结馀的进出) 决定，不受见证部份影响。


