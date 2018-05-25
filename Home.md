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
#### 以太坊监控
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

比特币的交易主要是由两部分的数据组成的，一部分为”基础交易数据”，负责记录交易比特币的来源和去处；另一部分为“见证数据” 即签名脚本(sigature script)，目的是为了证明交易请求是否真实可信。每一笔交易都对应一串 64 位的十六进制哈希交易ID(TxID)标识。TxID 的计算方法使得任何人可以对交易做微小的改动，不会改变交易的内容，但是会改变TxID，这就是所谓的第三方延展性。隔离认证的目的之一就是为了解决交易延展性问题， Mt.GoX 当年的比特币60 多万的比特币被盗就是利用交易延展性。隔离认证的想法就是把签名脚本从交易中单独那出炉放在一个新的数据结中，而且在审计区块大小的时候并不审计签名这部分的体积，所以隔离认证不仅可以扩容，而且在同样体积下的区块下也能放进去更多的交易，但是问题就是验证交易的时候需要单独验证签名部分的数据结构，会增加验证耗费，如果矿工不配合的话，那么问题自然也就出现了 。

### 以太坊：什么是ERC20标准？
https://www.jianshu.com/p/a5158fbfaeb9

### 侧链：为什么研究者认为它解决了谜题中的重要一环
http://www.8btc.com/sidechains-solved-key-piece-puzzle

### 薛蛮子: 区块链改变生产关系
http://www.8btc.com/xue-man-zi-says-blockchain-equip-everyone-with-ablity-to-create-bat
区块链精神的认识： 首先是信任的规模化。其次是代码即共识，第三个是赋能到个体。

### Chrome插件开发
https://www.cnblogs.com/liuxianan/p/chrome-plugin-develop.html

### 解读：智能合约、以太坊、ICO
https://www.kaspersky.com.cn/blog/ethereum-ico/8756/

### Plasma
http://www.ceoclub.cc/blockchain/20170826/761.html
 昨天，Vitalik Buterin和Joseph Poon发布了Plasma项目的工作草案。
是一个以太坊的子链/侧链方案

### Algorand
http://www.8btc.com/sivio-micali-algorand  
图灵奖得主Sivio Micali的Algorand区块链协议简介  
随机投票的共识算法  

### 从零构建基于以太坊（Ethereum）钱包Parity联盟链
http://www.8btc.com/ethereum-parity  

### ConsenSys的BTC Relay，被认为是第一个侧链
http://www.8btc.com/relay-connecting-ethereum-with-bitcoin  

### Regis是一个平台，它能够构建、部署和管理在以太坊区块链分散登记
http://www.8btc.com/regis-decentralized-registry  

### uPort基于区块链的身份验证
https://www.jianshu.com/p/12a2454440bf
项目官网:uport.me

### 以太坊上的去中心化自治组织管理应用：Aragon白皮书（中文）
Aragon Network: 去中心化的价值交换基础设施  
http://ethfans.org/posts/aragon-network-whitepaper  

### Merkle Patricia Tree 梅克尔帕特里夏树（MPT）规范
http://me.tryblockchain.org/Ethereum-MerklePatriciaTree.html  

### 以太坊开发入门，完整入门
http://blog.csdn.net/uxiAD7442KMy1X86DtM3/article/details/79385479  
涉及Merkle树、IPFS、Gelem、Mist、Status、MetaMask、Parity等，很全面

### Filecoin：一种去中心化的存储网络
http://chainx.org/paper/index/index/id/13.html

### ENS 以太坊名称解析
ENS注册地址：https://registrar.ens.domains  
介绍文章：http://8btc.com/thread-93086-1-1.html  

### 比特币交易（Transaction）的输入与输出
http://blog.csdn.net/taifei/article/details/73735289

### www.provenance.org 一个产品溯源网站
基于以太坊

### Ethereum explained...[to your mother]
https://ethereum.stackexchange.com/questions/45/how-would-i-explain-ethereum-to-a-non-technical-friend

### 星火节点计划Windows接入文档
https://article.pchome.net/content-2045443.html
国内以太坊节点数量不足300个，而且国内特殊的网络原因也会影响到国内用户同步区块数据。为了国内用户更加流畅地同步区块，EthFans 发起星火节点计划。

### cosmos vs polkadot   跨链

### polkadot中文白皮书
https://wenku.baidu.com/view/348dee67590216fc700abb68a98271fe910eaf3b.html

### Ethereum Private chain Explorers 

https://ethereum.stackexchange.com/questions/13667/ethereum-private-chain-explorers  

Hosted Testnet (Morden) Explorers:

https://morden.ether.camp

http://testnet.etherscan.io

Open source projects:

https://github.com/gobitfly/etherchain-light

https://github.com/etherparty/explorer

https://github.com/maran/ethereum-blockchain-explorer

### 以太坊的一个POA侧链
https://poa.network/

### 权威证明PoA（Proof of Authority）
[参考1](https://www.zhihu.com/question/267204915)  
所谓权威证明（PoA），就是使用一组所谓的“权限” 来允许人们在区块链上创建新的节点并确保区块链的安全。以太坊测试网（Kovan）便是采用PoA算法。在PoA中，验证者（validator）是整个共识机制的关键。验证者不需要昂贵的显卡(如POW)，也不需要足够的资产(如POS)，但他必须具有已知的，并且已获得验证的身份。验证者通过放置这个身份来获得担保网络的权利，从而换取区块奖励。若是验证者在整个过程中有恶意行为，或与其他验证者勾结。那通过链上管理可以移除和替换恶意行为者。现有的法律反欺诈保障会被用于整个网络的参与者免受验证者的恶意行为。

**POA 网络（POA Network）：**好了，终于到了我们的正题，什么是POA网络？ POA网络便是建立在PoA共识之上的底层区块链。在POA网络中，每个验证者（validator）都必须在美国境内拥有公证许可。人们需要通过POA网络身份DApps进行身份验证，包括住址证明和无犯罪记录证明等。然后进行所谓的启动仪式（initiation ceremony）来获得密钥。通过的验证者便可以担当起保护网络的安全的重任，并获得回报。 POA网络作为一个底层链，其上的每一个新的专用链都可以使用相同的验证器，或者拥有自己的一套验证器以及其他任何类型的可验证许可证。

![img](https://pic2.zhimg.com/80/v2-f778c545d345c2237ea3e8b26d6b8088_hd.jpg)

通俗的理解就是，以前我们交易需要一群互不认识的人来拼算力，艰难的计算一道复杂的数学题，从而争出个输赢。而现在，我们只需要有一个信得过的人（至少得到了他们团队的认可）做担保，便可快速通过交易。而这个人也会因为为这笔交易做担保，而获得报酬。要是这个担保人使坏怎么办？没关系，其他的可信担保人看着呢，他要是敢使坏，我踢你出局。而要是担保人故意为难你呢？没关系，现有的法律可以保证你的合法权益（注意，数字资产丢失不在我国法律保护范围）。附上一个他们网络中一位验证者对PoA机制的视频介绍链接：[https://youtu.be/5RumGukS8pw](https://link.zhihu.com/?target=https%3A//youtu.be/5RumGukS8pw)最后还需要强调一下，POA网络除了共识机制以外，和以太坊网络完全一样。也就是说你在以太坊网络上想干的事都可以在POA网络上干，什么智能合约啦，发行新币啦，建立自己的DApps啦，都没有问题。

**如何评价POA网络：**说了那么多，上面都是答非所问。现在来谈谈我对POA网络的看法。POA网络确实是对以太坊网络现如今诸多不合理问题的一个解决方案。他相对于以太坊网络有很好的兼容，同时具有极大的优势。1. 官方宣称**5s**就能打包一个区块，而我们的大以太受阻塞时，可能一个小时后，你还收不到你的代币。2. 不需要挖矿，节能环保3. 整个网络，验证者互相监督，随时可以投票加入新的验证者或者剔出不合格验证者4. 所有的验证者都跟网络签署了协议，若谁要想在POA上弄个新的分叉出来（POC？），那不行，这是要走基本法的5. 高度可扩展性和高度兼容性，智能合约也信手拈来，让其成为优化的ETH2.0

![img](https://pic3.zhimg.com/80/v2-ab4700a716093b2c183036c7d8d2ee8a_hd.jpg)

优点固然多，解决方案也巧妙。但是不知道大家有没有和我同样的疑问，这样的机制还是一个去中心的网络吗？在智能合约作为信任的基础上，POA网络将人与人由合约建立其的信任，转变成人与人对网络中验证者的信任。而将对验证者的信任，转变成对那些对验证者身份进行审核的人的信任。最终转变的结果是，**我相信你们选出来的验证者绝大多数都是好人，且这之中大多数人互相不认识，不会串通起来为所欲为**。解决区块链问题往往难以得到两全，有时候获得速度，可能就要丧失一些安全。这个“两全之策”背后，确无法让我停下思考，这样的网络真的安全吗？这样的网络真的可以托付起未来区块链改变互联网的重任吗？现如今区块链尚未成熟，区块拥堵问题，并没有完美无瑕的解答，不管是硬分叉，还是强制扩容，或是新兴的闪电网络，在尝试过程中PoA共识也是其中的一个方案。我们难以评判孰好孰坏，“实践是检验真理的唯一标准”，好不好，试了才知道。以上只是笔者自己的一些思考，有什么不足还请指教。谢谢！

(陈龙的回答)

权威证明机制Proof of Authority(POA) 是运用在区块链上的通过基于身份权益的治理机制来传递迅速交易的算法。在基于POA的网络中，交易和区块被已知的账户即见证人验证。见证人运行允许他们把交易打包进区块的软件。这个过程是自动化的并不需要见证人时刻的监控电脑。
**POA与工作证明机制(POW)和权益证明机制(POA)的区别**
POW和POS共识都使用了挖矿的机制，而POA不是
POS机制使用了一个机制，选择有最高权益的参与者作为见证人，这个机制假设最大的权益者是被 确认交易被处理所激励的。POW在交易被确认之前通过验证工作已经被完成来运行
同时，POA使用了个人身份作为单独的权威证明去验证，这意味着整个过程不需要挖矿
**POA与代理权益机制的区别(DPoS)**
DPOS通过使用见证人产生区块来运行。见证人是被利益相关者选举出来的，选举方法是一票一个见证人一份权益。然而，在POA中，权威的指定是事先钦定的，这意味着不会由于不公平的权益(Stake)产生偏见和不公平的过程。在POA中，见证人需要用他们自己的身份通过Dapps正式验证，并且在公开的领域他们的身份信息对每一个人都是透明可见的。
**POA的优势**
POA背后的理念是个人争取去成为见证人的权利，因此他们有保持自己原有地位的激励。通过把身份和声誉绑定在一起，见证人被激励去验证交易和维护网络安全，他们不会希望把自己的身份和一个不好的声誉联系在一起。这个机制被认为比普通的权益证明机制（POS）更安全

### 揭秘18年顶级区块链基础设施，POA有望成为第2个以太坊，投前必读
https://baijiahao.baidu.com/s?id=1594747466218367000&wfr=spider&for=pc  
POS: 代表：未来币以及转型之后的以太坊。  
DPOS: 代表：EOS。  
POA: 代表： POA Network （https://poa.network/）

### 盘点国内最热门的五款区块链项目：量子链、红烧肉、共享链、公信宝、NEO
http://app.myzaker.com/news/article.php?pk=5a311c581bc8e0b55a000001  

### 可参考的区块链白皮书
[Qtum量子链v0.6](https://s.qtum.site/white-papers/%5B2016-08-01%5DQuantum-White-Paper-v0.6-CN.pdf)(新版是英文的)  
[Bytom比原链](http://bytom.io/BytomWhitePaperV1.0.pdf)  
[公信宝区块链技术和应用白皮书(V2.0.0)](https://github.com/gxchain/whitepaper/blob/master/zh/whitepaper.md)  
[iCubeChain 白皮书：超级智能自金融网络](https://github.com/wbwangk/wbwangk.github.io/raw/master/Docs/iCubeChain%20%E7%99%BD%E7%9A%AE%E4%B9%A6%EF%BC%9A%E8%B6%85%E7%BA%A7%E6%99%BA%E8%83%BD%E8%87%AA%E9%87%91%E8%9E%8D%E7%BD%91%E7%BB%9C%20.pdf)  
[中钞可信登记开放平台白皮书](https://github.com/wbwangk/wbwangk.github.io/raw/master/Docs/%E4%B8%AD%E9%92%9E%E5%8F%AF%E4%BF%A1%E7%99%BB%E8%AE%B0%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0%E7%99%BD%E7%9A%AE%E4%B9%A6.pdf)  

### 区块链的预言机oracle机制解释
http://www.sohu.com/a/215929325_686941  

### Setup your first Parity PoA network in a few commands
https://github.com/dstarcev/parity-poa-playground  
自带了[ethstats](https://github.com/cubedro/eth-netstats)面版。  
有3个authority节点和3个member节点构成的集群。安装在u1601上。  

### Kovan基于POA共识的以太坊测试网络
http://www.8btc.com/parity-pushes-new-ethereum-testnet

### 当区块链延伸到版权保护 这些问题值得深思
http://mp.weixin.qq.com/s/AFdVIDx7gVLPeYtVnmOR9w

### 溯源链
https://www.tacchain.io/Index/index_c.html

### sybil attack （女巫攻击）
http://blog.csdn.net/qq_35056292/article/details/60480292  

### 加密货币市值
https://coinmarketcap.com/zh/all/views/all/  
排名144位的是POA Network

### 【Chainge之供应链金融】系列二：区块链3.0：分道扬镳，说散就散
http://www.8btc.com/178811-2

### solidity变量的一个教程
http://wiki.jikexueyuan.com/project/solidity-zh/types.html

### 以太坊中文文档（riversyang翻译）
https://github.com/riversyang/homestead-guide-cn

### 以太坊轻钱包或钱包托管
https://github.com/ConsenSys/eth-lightwallet

### Proof of Existence 
https://proofofexistence.com/
A web application to prove the existence of documents using the blockchain

### The blockchain for Smart Media Tokens (SMTs) and decentralized applications. https://steem.io
https://github.com/steemit/steem

### “比特币是Wei Dai在网络朋克中所提到的B-Money构想和尼克·萨博提出的Bitgold的具体实现。”
http://www.8btc.com/bitcoin-lovers

### CryptoPunk
https://github.com/larvalabs/cryptopunks  
Collectible 8-bit characters on the Ethereum blockchain. http://www.larvalabs.com/cryptopunks

### 令牌是什么
https://github.com/ethereumbook/ethereumbook/blob/develop/tokens.asciidoc

### ethereumbook中文版
https://github.com/AMTCOIN/ethereumbook_zh

### 开源的在线钱包
https://github.com/MyCryptoHQ/MyCrypto   

### 以太坊实现的零售商店
https://github.com/brakmic/BlockchainStore

### awesome-ethereum
https://github.com/btomashvili/awesome-ethereum

### 基于以太坊的声誉
https://github.com/Iudex/Iudex
用户需要绑定外部账号，如微博，然后通过 http://oraclize.it/ 抓取数据，进行计算

### 以太坊节点数、客户端版本、节点分布等
https://ethernodes.org/network/1  
Parity：4916，Geth：9527， 总数：16326  

Total16326 (100%)  
United States5751 (35.23%)  
China2376 (14.55%)  
Canada923 (5.65%)   
Germany843 (5.16%  

### 以太坊分片的中文讲座
https://www.youtube.com/watch?v=GhuWWShfqBI

### 拼图游戏（收币、补天）源码
https://github.com/beidan/Puzzle

### 以太坊基金会声明反对EME作为W3C推荐标准
https://blog.ethereum.org/2017/08/21/statement-objecting-w3c-publishing-eme-recommendation/  
EME是加密媒体扩展的缩写

### 中国将建区块链国家标准，最快2019年完成
http://www.8btc.com/blockchain_standard_2019

### 沈寓实：“互链网”时代即将到来，区块链是中国的重大机遇
http://www.8btc.com/blockchain_security_chance  
网络空间世界观——两重世界与五域重构  
信息互联网向价值互联网转变  
区块链技术构建网络空间超互联未来  

### zkSNARK介绍
https://blog.ethereum.org/2016/12/05/zksnarks-in-a-nutshell/  
zkSNARK的可能性令人印象深刻，您可以验证计算的正确性，而无需执行它们，您甚至不会知道执行了什么 - 只是它正确地完成了。  
解释RSA、零知识证明等。  

### 世界银行权威点评比特币、以太坊、瑞波币的弊端
http://www.8btc.com/world-bank-digital-currency  
挖矿难度分析： 几乎相当于整个孟加拉国的耗电量。平均用于处理一笔交易的电费(约20美元)，可以为大约5户高收入国家的家庭供电一天。（截至2018年春季，矿工每次交易获得的总报酬略低于100美元。）   
2018年3月各矿池提供网络区块数据表  
Fizzy是保险公司AXA的一个参数保险应用程序(https:/fizzy.axa/)  

### 比特币是有限游戏，还是无限游戏？
http://www.8btc.com/201805161324  

### 大咖超链接丨尹振涛：虚拟货币的监管首先要明确它是资产
http://www.8btc.com/chaolianji-yinzhengtao

### 通货膨胀，交易费用和货币政策的加密货币政策
https://blog.ethereum.org/2016/07/27/inflation-transaction-fees-cryptocurrency-monetary-policy/  

### Scalability and Asynchronous Programming
https://docs.google.com/presentation/d/1CjD0W4l4-CwHKUvfF5Vlps76fKLEC6pIwu1a_kC_YRQ/edit#slide=id.p  
一个讲以太坊扩展性和异步编程的PPT

### 以太坊15年的入门三教程
[以太坊实践第3部分：如何在区块链上建立自己的透明银行](https://blog.ethereum.org/2015/12/07/ethereum-in-practice-part-3-how-to-build-your-own-transparent-bank-on-the-blockchain/)  
[第二篇文章介绍了如何生成由这些令牌控制的数字民主](https://blog.ethereum.org/2015/12/04/ethereum-in-practice-part-2-how-to-build-a-better-democracy-in-under-a-100-lines-of-code/)  
[第一篇文章中，我们详细介绍了如何创建一个令牌](https://blog.ethereum.org/2015/12/03/how-to-build-your-own-cryptocurrency/)  

第一篇中有Token的示范代码(不是ERC20)，比较容易读懂。该token表示股东在某个组织中的股份，最高是100%。
第二篇中是一个投票，股东的股份值代表了投票的权重，可以投票给某个地址转账以太币。
第三篇演示了投票执行合约的某个函数。

### 从零到一
http://www.doc88.com/p-0993271886880.html  http://vdisk.weibo.com/s/GZDboi2fBIag  

### Futarchy简介(预测市场)
https://blog.ethereum.org/2014/08/21/introduction-futarchy/

### 您可能没有想过的保证金和预测市场的应用
https://blog.ethereum.org/page/10/  

### 以太坊中的梅克尔(Merkle)树
https://blog.ethereum.org/2015/11/15/merkling-in-ethereum/

### 预测市场Augur
http://www.8btc.com/augur59107

### 代币分类框架：一套正确认识 token 的思维工具
https://ethfans.org/posts/the-token-classification-framework  
从五个维度对 token 进行分类  

### 引介 | 区块链上的自主身份
https://ethfans.org/posts/31973

### 王永利：区块链发展，要跳出“比特币区块链”范式
http://www.8btc.com/wang-yong-li-0523

### “电商版以太坊”CyberMiles：我们的构想与未知
https://www.leiphone.com/news/201804/05YG1SHXasUA1Slz.html

### 工信部于佳宁：“区块链+”：金融支持实体经济的新路径
http://www.8btc.com/yujianing-2

### 科普 | 为什么使用提款（Withdrawal）模式？
https://ethfans.org/posts/why-use-the-withdrawal-pattern

### 科普 | 现代经济中的货币，Part-3
https://ethfans.org/posts/money-in-modern-economy-part-3

### 观点 | 重新发明 Google：代币就是新时代的“超链接”
https://ethfans.org/posts/building-google-for-the-economic-web-on-the-ethereum-blockchain  
有web1.0、web2.0、web3.0的对比表格

### 科普 | 小跑进入以太坊，Part-3
https://ethfans.org/posts/getting-up-to-speed-on-ethereum-part-3
预言机、预测市场、稳定币、 基本注意力代币（BAT）、如何发布列表、查找和筛选列表，如何在社区内部管理声誉

### EthFans精选 | 以太坊入门清单
https://ethfans.org/posts/1099

### PPT | 以太坊台北分片研讨会：安全模式机制设计
https://ethfans.org/posts/taipei-sharding-workshop-security-models-in-mechanism-design