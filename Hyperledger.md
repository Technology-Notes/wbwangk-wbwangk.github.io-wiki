## Hyperledger概述
### 什么是区块链
#### 一个分布式账本
所有交易数据被保存到账本中。账本中的记录只能增加，不能修改。所有节点(Peer)都保存了一份交易数据的全部。每间隔一定时间或交易量达到一定数量，形成一个区块(Block)。区块一旦形成就不能修改。多个区块顺序链接在一起形成区块链。  
#### 智能合约
为了支持信息的一致性更新，区块链网络使用智能合约来提供对账本的受控访问。  
智能合约不仅封装信息，并使其在整个网络中保持简单的关键机制，还可以写入智能合约以允许参与者自动执行交易的某些方面。  
例如，可以写一个智能合约来规定运输项目的成本，这个成本取决于何时到达。根据双方同意并写入账本的条款，当收到物料时，适当的资金自动转账。  
智能合约的实现，在Hyperldedger中叫链码(chaincode)。  
#### 共识
在整个网络中保持账本交易同步的过程 - 确保账本只有在交易得到适当的参与者批准时才会更新，而且当账本更新时，它们以相同的顺序更新相同的交易 - 被称为共识。  
现在，将区块链视为一个共享的，复制的交易系统就足够了，该系统通过智能合约进行更新，并通过一个称为共识的协作过程保持一致的同步。
### 区块链有什么用
传统信息系统，参与者的私有被用来更新他们的私有账本。而区块链系统，是共享程序来更新共享账本。共享程序指出智能合约(链码)，因为智能合约的实现程序源码也要在区块链网络中被传播和复制，并保存在每个参与者节点。  

### Hyperledger Projects
把一段时间内生成的信息（包括数据或代码）打包成一个区块，盖上时间 戳，与上一个区块衔接在一起，每下一个区块的页首都包含了上一个区块的索引数据，然后再在本页中写入新的信息，从而形成新的区块，首尾相连，最终形成了区块链。  
5个主项目：Cello、Fabric(Golang)、SawtoothLake(Python)、Iroha、Blockchain Explorer  
#### Fabric
区块链技术的⼀个实现  
Golang  
#### STL-SawtoothLake
⾼度模块化的分布式账本平台 
Python  
• PoET consensus  
• Transaction Families  
• Scalability  
#### Iroha
轻量级的分布式账本， 侧重于移动  
C++  
• C++ environment  
• Mobile and Web application Support  
• Sumeragi consensus  
#### Blockchain Explorer
展⽰和查询区块链块、事务和相关数据的Web应⽤  
Node.js，UI to interact with ledger，Under-development  
• Web UI to explorer a blockchain  
• Single-Page	Application  
#### Cello
BaaS的⼯具集，帮助创建、管理、终⽌区块链  
Blockchain as a Service  
Python,JavaScript  

### Fabric
#### 成员管理(Membership)
会员注册、⾝份保护、 内容保密、交易审计功能，以保证平台访问的安全性。  
#### 区块服务（BlockChain&Transaction)
负责节点间的共识管理、账本的分布式计算、账本的存储以及节点间的P2P协议功能的实现，是区块链的核⼼组成部分，为区块链的主体功能提供了底层⽀撑。  
#### ChainCode
chaincode即智能合约(smart contract)。该模块为ChainCode提供安全的部署、运⾏的环境。  

#### Event
贯穿于其他各个组件中间，为各个组件间的异步通信提供了技术实现。  


### shared ledger
账本提供了一个所有状态变化的历史记录。它可以是一个业务的系统记录。
**区块账本(Block ledger)**  
 - 基于文件系统，只能添加新区块  
 - 区块保存在所有提交者节点，部分可选子集保存在排序服务节点  
**状态账本**  
 - 世界状态(账本)保存了智能合约数据的当前值  
   如vehicleOwner=Daisy  
 - 开发者通过链码API访问隐藏起来的KVS(key-value数据库)  
   如GetState(), PutState(), GetStateByRange()等等  
 - 保存在所有提交者节点  
**历史账本**  
 - 保存所有链码事务的顺序历史记录  
   如updateOwner(from=John, to=Anthony); updateOwner (from=Anthony, to=Daisy);等等  
 - 索引保存在KVS，开发者通过链码API访问  
   如GetHistoryForKey()  
 - 保存在所有提交者节点  

### 隐私与安全
交易方持有多种类型的证书，交易不同环节将使用如下这些类型的证书：  
• E-Cert（Enrollment Cert）  
  – 长期持有，携带或可以追溯使用者信息  
  – 用于身份认证  
• T-Cert（Transaction Cert）  
  – 每个交易时生成，用于交易的签名  
• TLS-Cert，长期持有，主要用于SSL/TLS通讯  

### 什么是Hyperledger结构
Hyperledger Fabric网络的成员通过成员资格服务提供商（MSP）注册，而不是一个允许未知身份参与网络的开放式无权限系统（需要证明工作证明的协议来验证交易和保护网络）。
Hyperledger Fabric还提供了创建**通道**的能力，允许一组参与者创建单独的交易账本。用通道来控制访问账本的权限。账本可以加入通道，参与者也可以加入通道。对于一个通道，里面的参与者可以“看到”里面的账本。  
#### 共享账本
Hyperledger Fabric有一个账本子系统，由两个组件组成：世界状态和事务日志。每个Hyperledger Fabric网络的参与者都拥有账本的副本（网络指通道？）。  
事务日志保存到区块链中（有点像数据库日志），账本的当前状态体现在世界状态中。世界状态保存在NoSQL数据库中，如Leveldb、Couchdb。  
#### 智能合约
Chaincode，也称为智能合同，是封装用于创建和修改资产的业务逻辑和事务处理指令的软件。当外部应用程序需要与账本交互时，它需要调用链码。在大多数情况下，链码仅与账本的的数据库组件-世界状态（例如查询）交互，而不是与交易日志(区块链)交互。  
Chaincode可以用几种编程语言来实现。当前支持的链式代码语言是Go和node.js，支持未来发行版中的Java和其他语言。  
Hyperledger Fabric支持私有网络（使用通道）。每个节点（参与者或称成员）可以加入多个通道，通道内的节点可以共享账本。  
具有正确权限的用户可以轻松地安装和实例化通道的链式代码，并查看是谁在他们参与的渠道中，恰当授权的用户可以根据所建立的区块链网络的策略调用链码，创建新的通道，甚至更新通道的访问权限。  
#### 共识
交易必须按照发生的顺序写入账本。共识算法有很多。例如，PBFT（Practical Byzantine Fault Tolerance）可以提供文件副本相互通信的机制，以保持每个副本的一致性，即使在发生损坏的情况下也是如此。  
Hyperledger Fabric被设计为允许网络发起者（指通道建立者？）选择最能代表参与者之间存在关系的共识机制。与隐私一样，还有一系列需求。从具有高度结构化的关系网络到更加对等的网络。  
我们将更多地了解Hyperledger Fabric共识机制，目前包括SOLO和Kafka，并且很快将扩展到SBFT（简化的拜占庭容错）。  

#### 组织
在Hyperledger Fabric中，每个参与者（客户端，peer，oderer）都属于某个组织。  
组织拥有CA，为其成员（客户端，peer，oderer）提供证书，以便向其他人或其他组织认证自己。  
它还提供了一种将参与者聚集在一起的简单方法，以便定义跨多个客户端，peer和oderer的访问控制规则，而不必为每个参与者单独定义。组织还可以通过发送CRL到组织参与的每个通道，来撤消其成员。  



 
(/e/vagrant10/ambari-agrant/fabric/devenv)  
## vagrant VM
fabric官方库提供了一个Vagrantfile，是个ubuntu16的环境，供开发调试用。如何想在一个空linux下手工安装，可以参考[Fabri Getting Started](http://hyperledger-fabric.readthedocs.io/en/latest/getting_started.html)。   
  
在windows下启动git bash，然后克隆fabric库：
```
$ cd /e/vagrant10/ambari-vagrant
$ git clone https://github.com/hyperledger/fabric.git
$ cd fabric/devenv
$ vagrant up
$ vagrant ssh
```
虚拟机启动过程中会自动执行一个`setup.sh`脚本进行初始化。有时会半途执行失败。可以进入linux后手工执行脚本：
```
$ cd /hyperledger/devenv
$ ./setup.sh
```
这样，你就拥有了一个fabric开发环境。(snap a2)  

通过分析`setup.sh`，也可以看到手工安装docker-compose、golang的办法：
#### 安装docker-compose
```
$ curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```
#### 安装go lang
`setup.sh`中提供的是从`golang.org`下载安装包，下面描述的是从另一个网站下载安装包。
```
$ cd /opt
$ wget https://redirector.gvt1.com/edgedl/go/go1.9.2.linux-amd64.tar.gz
$ tar -C /usr/local -xzf go1.9.2.linux-amd64.tar.gz
```
在文件`/etc/profile`的最后添加：
```
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/opt/gopath
```
使环境变量生效后验证go版本：
```
$ source /etc/profile
$ go version
go version go1.9.2 linux/amd64
```
### Hyperledger Fabric Samples
[Hyperledger官方GETTING STARTED](https://hyperledger-fabric.readthedocs.io/en/latest/samples.html)  
```
$ cd /opt
$ git clone -b master https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples
$ curl -sSL https://goo.gl/fMh2s3 | bash
```
如果执行后无反应，这是因为该链接被 墙。设置proxy:
```
$ export https_proxy=http://10.180.36.75:25378
$ export http_proxy=http://10.180.36.75:25378
```
25378是宿主机的翻&墙软件端口号，ip是宿主机ip。  

脚本会自动下载了很多docker镜像:
```
hyperledger/fabric-ca          latest                 2736904862db        4 weeks ago         218MB
hyperledger/fabric-ca          x86_64-1.1.0-preview   2736904862db        4 weeks ago         218MB
hyperledger/fabric-tools       latest                 6051774928a6        4 weeks ago         1.33GB
hyperledger/fabric-tools       x86_64-1.0.4           6051774928a6        4 weeks ago         1.33GB
hyperledger/fabric-couchdb     latest                 cf24b91dfeb1        4 weeks ago         1.5GB
hyperledger/fabric-couchdb     x86_64-1.0.4           cf24b91dfeb1        4 weeks ago         1.5GB
hyperledger/fabric-kafka       latest                 7a9d6f3c4a7c        4 weeks ago         1.29GB
hyperledger/fabric-kafka       x86_64-1.0.4           7a9d6f3c4a7c        4 weeks ago         1.29GB
hyperledger/fabric-zookeeper   latest                 53c4a0d95fd4        4 weeks ago         1.3GB
hyperledger/fabric-zookeeper   x86_64-1.0.4           53c4a0d95fd4        4 weeks ago         1.3GB
hyperledger/fabric-orderer     latest                 b17741e7b036        4 weeks ago         151MB
hyperledger/fabric-orderer     x86_64-1.0.4           b17741e7b036        4 weeks ago         151MB
hyperledger/fabric-peer        latest                 1ce935adc397        4 weeks ago         154MB
hyperledger/fabric-peer        x86_64-1.0.4           1ce935adc397        4 weeks ago         154MB
hyperledger/fabric-javaenv     latest                 a517b70135c7        4 weeks ago         1.41GB
hyperledger/fabric-javaenv     x86_64-1.0.4           a517b70135c7        4 weeks ago         1.41GB
hyperledger/fabric-ccenv       latest                 856061b1fed7        4 weeks ago         1.28GB
hyperledger/fabric-ccenv       x86_64-1.0.4           856061b1fed7        4 weeks ago         1.28GB
```
(snap a3)  
进入目录`/opt/fabric-samples/first-network`，这里就是hyperledger官方文档“[Building Your First Network](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html)”描述的工作目录。  
可以象文件中描写的那样执行`./byfn.sh --help`看看帮助文档。  
### 启动网络
```
cd /opt/fabric-samples/first-network
./byfn.sh -m up
```
上述命令会编译Golang链码镜像，和启动相应的容器。Go是默认的链码语言，但也支持Node.js链码。如果你想用node链码运行本教程，象下面这样执行：
```sh
# we use the -l flag to specify the chaincode language
# forgoing the -l flag will default to Golang

./byfn.sh -m up -l node
```
停止网络：
```
./byfn.sh -m down
```
### 密钥生成器
我们用`cryptogen`工具为不同的网络实体生成密码学文件。这些证书表达身份，对实体间通信和交易认证进行签名和验证。  
Cryptogen的配置文件是`crypto-config.yaml`，该文件包括网络拓扑，允许我们为组织以及属于组织的组件生成一系列证书和密钥。每个组织都会分配一个根证书(`ca-cert`)，该证书绑定特殊组件(peer和orderer)到组织。  
配置文件中有个`count`变量，我们用它来指定组织下的peer数量，在我们示例中，每个组织下有两个peer。我们在本文不会详述[X509证书和PKI](https://en.wikipedia.org/wiki/Public_key_infrastructure)。  
在`crypto-config.yaml`文件中，注意`OrdererOrgs`之下的“Name”, “Domain” and “Specs”参数。网络实体的命名约定是：`{{.Hostname}}.{{.Domain}}`。例如，排序节点的名称是`orderer.example.com`，这关联了一个MSP ID `Orderer`，关于MSP的更多细节参考[Membership Service Providers (MSP)](http://hyperledger-fabric.readthedocs.io/en/latest/msp.html)文档。  
执行`cryptogen`后，生成的证书和密钥保存到了目录`crypto-config`下。  
#### 配置交易生成器
工具`configtxgen`用于生成四个配置工件：
- 排序器(orderer)`genesis block`  
- 通道`configuration transaction`  
- 两个`anchor peer transactions`，每个组织生成一个  
关于`configtxgen`更多细节参考[Channel Configuration (configtxgen)](http://hyperledger-fabric.readthedocs.io/en/latest/configtxgen.html)。  

## Membership Service Providers (MSP)
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/msp.html)  
#### 组织与MSP映射
1. 一个组织映射为一个MSP  
  其他可能：组织下的部门都映射到一个MSP；多个组织共用同一个MSP   
2. 组织下有多个部门(OU)，各部门访问不同的通道  
  有两种可能的选择：  
  1) 定义一个MSP容纳组织的所有成员  
     配置MSP由一系列的根证书、中间证书和管理员证书组成，成员identity包含他所属的部门(OU)。策略将使用这一OU，这些策略包括对通道的读写策略，或对链码的背书策略。  
  2) 为每个OU定义一个MSP  
3. 隔离同一组织下的peer  
  在这一需求下，各peer为自己背书？  
4. 管理员和CA证书  
  将管理员证书和CA证书分离更安全。  
5. CA和TLS CA
  MSP identity 根CA和MSP TLS 证书根CA需要放在不同目录下。

## 备忘
### 共识过程
[出处](https://developer.ibm.com/courses/all/ibm-blockchain-foundation-developer/?course=begin#12034)   
- Committing Peer  
  Maintains Ledger and state. Commits Transactions. May hold smart contract(chaincode).  
- Endorsing Peer  
  Specialized committing peer that receives a transaction proposal for endorsement, responds granting or denying endorsement. Must hold smart contract.  
- Ordering Nodes(service)  
  Approves the inclusion of transaction blocks into the ledger and communicates with committing and endorsing peer nodes. Does not hold smart contract. Does not hold ledger.  
step 1/7 - propose transaction:  
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction.jpg)  
step 2/7 - execute transaction:  
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction2.jpg)
step 3/7 - propose response:  
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction3.jpg)  
step 4/7 - order transaction:  
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction4.jpg)  
step 5/7 - deliver transaction: 
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction5.jpg)  
step 6/7 - validate transaction: 
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction6.jpg)  
step 7/7 - notify transaction: 
![](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/images/fabric_transaction7.jpg)  

Transactions typically follow a seven-step process to become a block on the chain.
## 备忘
[Hyperledger Composer](https://hyperledger.org/projects/composer)  
[Hyperledger Composer教程](https://developer.ibm.com/courses/all/ibm-blockchain-foundation-developer/?course=begin#11982)，含composer的安装。  