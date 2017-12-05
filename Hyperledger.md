## Hyperledger概述
### 什么是区块链
#### 一个分布式账本
所有事务数据被保存到账本中。账本中的记录只能增加，不能修改。所有节点(Peer)都保存了事务数据的完整副本。每间隔一定时间或交易量达到一定数量，形成一个区块(Block)。区块一旦形成就不能修改。多个区块顺序链接在一起形成区块链。  
（把一段时间内生成的信息（包括数据或代码）打包成一个区块，盖上时间戳，与上一个区块衔接在一起，每下一个区块的页首都包含了上一个区块的索引数据，然后再在本页中写入新的信息，从而形成新的区块，首尾相连，最终形成了区块链。）  
事务日志保存到区块链中（有点像数据库日志），账本的当前状态体现在世界状态中。世界状态保存在NoSQL数据库中，如Leveldb、Couchdb。  
#### 智能合约(smart contract)
在Hyperldedger Fabric中链码(chaincode)和智能合约(smart contract)是一个意思。  
链码是封装用于创建和修改资产的业务逻辑和事务处理指令的软件。当外部应用程序需要与账本交互时，它需要调用链码。在大多数情况下，链码仅与账本的的数据库组件-世界状态（例如查询）交互，而不是与事务日志(区块链)交互。  
链码可以用几种编程语言来实现。当前支持的链式代码语言是Go和node.js，支持未来发行版中的Java和其他语言。  
Hyperledger Fabric支持私有网络（使用通道）。每个节点（参与者或称成员）可以加入多个通道，通道内的节点同享同一个账本。  
具有正确权限的用户可以轻松地安装和实例化通道的链式代码，并查看是谁在他们参与的渠道中，恰当授权的用户可以根据所建立的区块链网络的策略调用链码，创建新的通道，甚至更新通道的访问权限。  
#### 共识
在整个网络中保持账本交易同步的过程 - 确保账本只有在交易得到适当的参与者批准时才会更新，而且当账本更新时，它们以相同的顺序更新相同的交易 - 被称为共识。  
现在，将区块链视为一个共享的，复制的事务系统就足够了，该系统通过智能合约进行更新，并通过一个称为共识的协作过程保持一致的同步。  
事务必须按照发生的顺序写入账本。共识算法有很多。例如，PBFT（Practical Byzantine Fault Tolerance）可以提供文件副本相互通信的机制，以保持每个副本的一致性，即使在发生损坏的情况下也是如此。  
Hyperledger Fabric被设计为允许网络发起者（指通道建立者？）选择最能代表参与者之间存在关系的共识机制。与隐私一样，还有一系列需求。从具有高度结构化的关系网络到更加对等的网络。  
我们将更多地了解Hyperledger Fabric共识机制，目前包括SOLO和Kafka，并且很快将扩展到SBFT（简化的拜占庭容错）。  

### Hyperledger Projects
5个主项目：Cello、Fabric(Golang)、SawtoothLake(Python)、Iroha、Blockchain Explorer  
- Fabric  
    区块链技术的⼀个实现(Golang)  
- STL-SawtoothLake  
    ⾼度模块化的分布式账本平台（Python）  
    PoET consensus、Transaction Families、Scalability    
- Iroha  
    轻量级的分布式账本， 侧重于移动(C++)
- Blockchain Explorer
    展⽰和查询区块链块、事务和相关数据的Web应⽤(node.js)    
    UI to interact with ledger，Under-development  
- Cello
    BaaS的⼯具集，帮助创建、管理、终⽌区块链(Blockchain as a Service)  

### Fabric
Hyperledger Fabric网络的成员通过成员资格服务提供商（MSP）注册，而不是一个允许未知身份参与网络的开放式无权限系统（需要证明工作证明的协议来验证交易和保护网络）。
Hyperledger Fabric还提供了创建**通道**的能力，允许一组参与者创建单独的交易账本。用通道来控制访问账本的权限。账本可以加入通道，参与者也可以加入通道。对于一个通道，里面的参与者可以“看到”里面的账本。  
#### 成员管理(Membership)
会员注册、⾝份保护、 内容保密、交易审计功能，以保证平台访问的安全性。  
在Hyperledger Fabric中，每个参与者（客户端，peer，oderer）都属于某个组织。  
组织拥有CA，为其成员（客户端，peer，oderer）提供证书，以便向其他人或其他组织认证自己。  
交易方持有多种类型的证书，交易不同环节将使用如下这些类型的证书：  
- 注册证书(Enrollment Cert)。用于验证使用者身份，永不过期
- 事务证书(Transaction Cert)。每个交易时生成，用于交易的签名  
- 通信证书(TLS-Cert)，用于通信通道加密，永不过期  

#### 区块服务（BlockChain&Transaction)
负责节点间的共识管理、账本的分布式计算、账本的存储以及节点间的P2P协议功能的实现，是区块链的核⼼组成部分，为区块链的主体功能提供了底层⽀撑。  
#### 链码(ChainCode)
chaincode即智能合约(smart contract)。该模块为链码提供安全的部署、运⾏的环境。  
#### Event
贯穿于其他各个组件中间，为各个组件间的异步通信提供了技术实现。  
### 账本
由两个组件组成：世界状态和事务日志。  
账本提供了一个所有状态变化的历史记录。它可以是一个业务的系统记录。每个Hyperledger Fabric网络的参与者都拥有账本的副本（网络指通道）。  
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
 - 保存所有链码事务的顺序历史记录。索引保存在KVS，开发者通过链码API访问  
 - 保存在所有提交者节点  
 
## 开发环境(vagrant)
(/e/vagrant10/ambari-agrant/fabric/devenv)   
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
$ curl -sSL https://goo.gl/6wtTN5 | bash -s 1.1.0-preview
```
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
上述脚本还下载了几个工具到`/opt/fabric-samples/bin`目录下，它们是cryptogen,configtxgen,configtxlator,peer。  
为了访问它们方便，需要修改PATH参数。编辑`/etc/profile`，在文件的最后添加内容：
```
export PATH=/opt/fabric-samples/bin:$PATH
```
为了保存VM当前状态，回到vagrant命令行下执行`vagrant snapshot save a3`  

### 启动首个网络(first-network)
本节遵循hyperledger官方文档“[Building Your First Network](http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html)  
```
cd /opt/fabric-samples/first-network
./byfn.sh -m up
```
上述命令会编译Golang链码镜像，和启动相应的容器。  
#### 启动网络的过程
```
Generate certificates using cryptogen tool 
Generating Orderer Genesis block
Generating channel configuration transaction 'channel.tx'
Generating anchor peer update for Org1MSP
Generating anchor peer update for Org2MSP  
Channel "mychannel" is created successfully =====================
PEER0 joined on the channel "mychannel" =====================
PEER1 joined on the channel "mychannel" =====================
PEER2 joined on the channel "mychannel" =====================
PEER3 joined on the channel "mychannel" =====================
Anchor peers for org "Org1MSP" on "mychannel" is updated successfully =====================
Anchor peers for org "Org2MSP" on "mychannel" is updated successfully =====================
Chaincode is installed on remote peer PEER0 =====================
Chaincode is installed on remote peer PEER2 =====================
Chaincode Instantiation on PEER2 on channel 'mychannel' is successful =====================
Querying on PEER0 on channel 'mychannel'... =====================
Invoke transaction on PEER0 on channel 'mychannel' is successful =====================
Chaincode is installed on remote peer PEER3 =====================
Querying on PEER3 on channel 'mychannel'... =====================
========= All GOOD, BYFN execution completed ===========
 _____   _   _   ____
| ____| | \ | | |  _ \
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```
停止网络：
```
./byfn.sh -m down
```
### 理解Fabric网络
应用通过API调用智能合约。智能合约托管在网络中，靠名称和版本号识别。例如，智能合约容器的名称是`dev-peer0.org1.example.com-fabcar-1.0`，其中`fabcar`是智能合约名称，`1.0`是智能合约版本号，而`dev-peer0.org1.example.com`是peer名称。  
API可以把SDK访问。SDK封装了应用与智能合约通信的接口，如查询或接收账本更新。这些API使用几个不同的网络地址，接收一些输入参数。智能合约由peer管理员安装，然后按照链码的策略被实例化到通道中。智能合约的实例化流程与普通调用的事务流程相同，背书、排序、验证、提交，之后才能与链码容器交互（智能合约实例化就是链码容器启动）。   
#### 查询
查询是最简单的调用：一个请求和响应。最常见的查询是向状态数据库查询一个key的当前值(`GetState`)。然而，[链码shim接口](https://github.com/hyperledger/fabric/blob/release/core/chaincode/shim/interfaces.go)允许不同的Get请求，如`GetHistoryForKey`或`GetCreator`。  
创建查询需要指定一个peer、一个链码、一个通道和一系列输入(如key)和一个可用的链码函数，然后通过API`chain.queryByChaincode`发送查询到peer。相应的响应值会返回给应用客户端。  
#### 更新
账本更新开始于应用创建一个事务提议。类似于查询，创建事务请求需要指定一个peer、链码、通道、函数和一系列输入。程序之后会调用API`channel.SendTransactionProposal`发送事务提议到peer寻求背书。  
网络(也就是背书peer(可能多个))会返回一个提议响应，应用使用该响应来创建和签署事务请求。通过调用API`channel.sendTransaction`，这个事务请求被发送到排序服务。排序服务将事务捆绑入一个区块，并将它发送到通道中的所有peer以求验证(Fabcar网络只有一个peer和一个通道)。  
最后应用使用两个事件处理器API：用`eh.setPeerAddr`连接到peer的事件监听者端口，用`eh.registerTxEvent`和一个特定事务ID去注册事件。`eh.registerTxEvent`API使应用可以收到事务结果通知（就是验证是否通过）。  
事务流程图示参考本文的[共识过程](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E5%85%B1%E8%AF%86%E8%BF%87%E7%A8%8B)一节。  
关于事务流程的更多细节参考[Transaction Flow](http://hyperledger-fabric.readthedocs.io/en/latest/txflow.html)。  
开始链码编程参考[Chaincode for Developers](http://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html)。  
更多背书策略参考[Endorsement policies](http://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html)。  
更多fabric架构信息参考[Architecture Explained](http://hyperledger-fabric.readthedocs.io/en/latest/arch-deep-dive.html)。  
## 编写首个应用
在实践本章前，请按[启动首个网络(first-network)](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E5%90%AF%E5%8A%A8%E9%A6%96%E4%B8%AA%E7%BD%91%E7%BB%9Cfirst-network)一章的描述准备好环境。  
本章原文是官网的[Writing Your First Application](http://hyperledger-fabric.readthedocs.io/en/latest/write_first_app.html)  
本章的工作目录是`/opt/fabric-samples/fabcar`。  
在本章中，首先访问CA生成注册证书(ECert)，然后利用生成的身份(用户对象)查询和更新账本。  
### 建立开发环境
可以通过执行下列命令停止“first-network”一章中的各种容器：
```
$ cd /opt/fabric-samples/fabcar
$ ../first-network/byfn.sh -m down
```
或使用docker命令删除所有容器(可以与上面的命令混合使用)：
```
$ docker rm -f $(docker ps -aq)
```
删除所有缓存网络：
```
$ docker network prune
```
#### 安装客户端和启动网络
`/opt/fabric-samples/fabcar`目录下有个`package.json`文件，定义了示范应用的依赖包，通过执行下列命令安装依赖（与java的maven类似）：
```
$ npm install
```
使用脚本`startFabric.sh`启动网络。这个命令会启动几个Fabric实体和一个智能合约容器(链码)。
```
$ ./startFabric.sh
$ docker ps
```
用上述docker命令可以看到启动的容器清单。  
### 理解Fabric网络
参考章节[理解Fabric网络]()  
### 注册用户
#### 注册管理员用户
前面的`startFabric.sh`命令会启动一个CA容器，可以通过下列命令查看CA容器的日志：
```
$ docker logs -f ca.example.com
```
执行下列命令来注册admin用户：
```
$ node enrollAdmin.js
```
上述代码会向创建目录`hfc-key-store`，创建私钥，向CA发送证书签名请求(CSR)，然后把返回的CA签名证书(eCert)存放在`hfc-key-store`目录。  
#### 注册用户user1
admin用户是运维人员用的管理员证书。在一般的业务场景下，应用程序使用“普通用户”(非管理员)来访问Fabric网络。需要说明的是，向Fabric网络注册普通用户需要使用管理员身份，如现在就是用admin身份来注册`user1`用户。  
```
$ node registerUser.js
```
与注册admin用户类似，代码会创建user1用户私钥，向CA发送证书签名请求(CSR)，并将返回的CA签名证书(eCert)存放在`hfc-key-store`目录。  
### 查询账本
用`docker ps`命令可以看到一个`hyperledger/fabric-couchdb`的容器，这是一个保存“世界状态”的key-value数据库(couchdb)。查询账本实际上就是在查询key-value数据库中的数据（数据库变化日志保存在区块链中）。查询参数一般是一个或几个key，也可以用json串当参数进行复杂查询。  
演示查询的代码文件是`query.js`，其中可以看到下面的代码，说明应用使用`user1`作为签名实体。
```
fabric_client.getUserContext('user1', true);
```
user1的身份证明材料已经放在了`hfc-key-store`目录下，我们简单地告诉应用去获取身份。  
```
$ node query.js
```
会返回一个json串，里面是全部10辆车的信息。  
观察一下`query.js`源码。在应用的初始化区段，定义了几个变量，如通道名称、证书库地址和网络端点。
```js
var channel = fabric_client.newChannel('mychannel');
var peer = fabric_client.newPeer('grpc://localhost:7051');
channel.addPeer(peer);

var member_user = null;
var store_path = path.join(__dirname, 'hfc-key-store');
console.log('Store path:'+store_path);
var tx_id = null;
```
下面是执行查询的代码：
```js
// queryCar chaincode function - requires 1 argument, ex: args: ['CAR4'],
// queryAllCars chaincode function - requires no arguments , ex: args: [''],
const request = {
  //targets : --- letting this default to the peers assigned to the channel
  chaincodeId: 'fabcar',
  fcn: 'queryAllCars',
  args: ['']
};
```
应用执行时，它调用了peer的`fabcar`链码，执行其中的`queryAllCars`函数。 
如果想看看有哪些链码函数可以调用，可以打开文件`../chaincode/fabcar/go/fabcar.go`来看。可以看到下列函数可以调用：`initLedger`, `queryCar`, `queryAllCars`, `createCar`和`changeCarOwner`。  
下图图示了应用、智能合约和账本的关系：  
![](http://hyperledger-fabric.readthedocs.io/en/latest/_images/RunningtheSample.png)  
下面示范一下调用链码的`queryCar`函数来查询某一辆车的信息。将`query.js`中的`queryAllCars`函数换成`quieryCar`，参数key是`CAR4`。
```js
const request = {
  //targets : --- letting this default to the peers assigned to the channel
  chaincodeId: 'fabcar',
  fcn: 'queryCar',
  args: ['CAR4']
};
```
保存`query.js`，并执行，可以看到返回了`CAR4`的车辆信息：
```
$ node query.js
{"colour":"black","make":"Tesla","model":"S","owner":"Adriana"}
```
#### 更新账本
更新账本的过程：先提议，后背书，结果返回到应用，然后发送给排序节点，再写到各个peer的账本。  
![](http://hyperledger-fabric.readthedocs.io/en/latest/_images/UpdatingtheLedger.png)  
js文件`invoke.js`是更新账本的程序示范。首先可以看到程序中构造请求的代码：
```js
var request = {
  //targets: let default to the peer assigned to the client
  chaincodeId: 'fabcar',
  fcn: 'createCar',
  args: ['CAR10', 'Chevy', 'Volt', 'Red', 'Nick'],
  chainId: 'mychannel',
  txId: tx_id
};
```
在代码中通过调用链码`fabcar`的`createCar`函数，试图创建一个10号车(key是`CAR10`)。执行`invoke.js`：
```
$ node invoke.js
Store path:/opt/fabric-samples/fabcar/hfc-key-store
Successfully loaded user1 from persistence
Assigning transaction_id:  1adb925d24db20816bcfc97f0216f3b094f3291778af775c1d07d0d3179a3031
Transaction proposal was good
Successfully sent Proposal and received ProposalResponse: Status - 200, message - "OK"
info: [EventHub.js]: _connect - options {"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1}
The transaction has been committed on peer localhost:7053
Send transaction promise and event listener promise have completed
Successfully sent transaction to the orderer.
Successfully committed the change to the ledger by the peer
```
从上面可以看到关于`ProposalResponse`的终端输出和promise(一个异步调用标准)。上文的`The transaction has been committed on peer localhost:7053`表示事务已经成功写到peer。下面可以回到`query.js`，将查询条件由`CAR4`改成`CAR10`：
```js
const request = {
  //targets : --- letting this default to the peers assigned to the channel
  chaincodeId: 'fabcar',
  fcn: 'queryCar',
  args: ['CAR10']
};
```
重新执行`query.js`可以查询到最新添加的10号车的信息：
```
$ node query.js
Response is  {"colour":"Red","make":"Chevy","model":"Volt","owner":"Nick"}
```
下面重新修改代码`invoke.js`。将函数`createCar`改成`changeCarOwner`，目的是把10号车的车主改成`Dave`：
```js
var request = {
  //targets: let default to the peer assigned to the client
  chaincodeId: 'fabcar',
  fcn: 'changeCarOwner',
  args: ['CAR10', 'Dave'],
  chainId: 'mychannel',
  txId: tx_id
};
```
重新执行`invoke.js`和`query.js`，可以看到车主信息被从`Nick`改成了`Dave`。
```
$ node invoke.js
$ node query.js
Response is  {"colour":"Red","make":"Chevy","model":"Volt","owner":"Dave"}
```
本章主要讲应用开发，后面的章节会讲链码的开发。  

## 链码教程
[官方原文](http://hyperledger-fabric.readthedocs.io/en/latest/chaincode.html)  
链码是一个程序，用Go、Node.js编写(未来会支持其他语言，如Java)，实现了一个规定的接口。链码运行在安全的Docker容器中，隔离于背书peer过程。链码通过应用提交的事务来初始化和管理账本状态。  
链码处理网络成员都同意的业务逻辑，所以它可以被认为是“智能合约”。链码创建的状态是不能直接被其他链码访问的（scoped）。但在同一个网络内（一般指通道？），通过适当的授权，一个链码可以调用其它链码而从访问它的状态。  
我们提供了观看链码的两个不同视角，一个面向开发链码应用的开发者，另一个面向区块链网络的管理者，他会利用Hyperledger Fabric API去安装、实例化和更新链码，但不会涉及链码应用开发（[链码教程:链码运维](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E9%93%BE%E7%A0%81%E6%95%99%E7%A8%8B%E9%93%BE%E7%A0%81%E8%BF%90%E7%BB%B4)）。  

## 链码教程:链码开发
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/chaincode4ade.html)    
每个链码程序都必须实现`Chaincode`[接口](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#Chaincode)。下面是Go语言的`Chaincode`接口：
```go
type Chaincode interface {
    // Init is called during Instantiate transaction after the chaincode container
    // has been established for the first time, allowing the chaincode to
    // initialize its internal data
    Init(stub ChaincodeStubInterface) pb.Response

    // Invoke is called to update or query the ledger in a proposal transaction.
    // Updated state variables are not committed to the ledger until the
    // transaction is committed.
    Invoke(stub ChaincodeStubInterface) pb.Response
}
```
Fabric通过调用这些约定的函数来运行事务。在响应中通过调用这些方法来接收事务。当链码收到`instantiate`或`upgrade`事务时，`Init`方法会被调用，使链码可以执行必要的初始化，包括应用状态初始化。在响应中收到`invoke`事务时`Invoke`方法会被调用，使链码可以处理事务提议(proposal)。  
链码的[“shim”](https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim)API中的另一个接口是`ChaincodeStub`[接口](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub))。它用于访问和修改账本，以及链码间的调用。  
在这个教程中，我们将示范使用这些API实现一个简单链码应用，来管理简单“资产”。   

### 简单资产链码
我们的应用是一个基本示范链码，用来在账本中创建资产(键值对)。  
假定`GOPATH=/opt/gopath`，下面创建示范代码的工作目录：
```
$ mkdir -p $GOPATH/src/sacc && cd $GOPATH/src/sacc
$ touch sacc.go
```
touch命令创建了一个叫sacc.go的空文件。下面描写如何向sacc.go中添加代码。    
首先用go的import语句添加链码的必要依赖，shim包和[peer protobuf](http://godoc.org/github.com/hyperledger/fabric/protos/peer)包。然后，增加一个`SimpleAsset`结构作为链码shim函数的接收者。
```go
package main

import (
    "fmt"

    "github.com/hyperledger/fabric/core/chaincode/shim"
    "github.com/hyperledger/fabric/protos/peer"
)

// SimpleAsset implements a simple chaincode to manage an asset
type SimpleAsset struct {
}
```
实现`Init`函数：
```go
// Init is called during chaincode instantiation to initialize any data.
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {

}
```
#### 初始化
注意，链码的程序版本更新也会调用`Init`函数。（在Fabric中链码程序更新也是通过事务，同不同事务一样，所以会调用`Init`函数。）  

下面，我们调用[ChaincodeStubInterface.GetStringArgs](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.GetStringArgs)函数取得调用`Init`的参数，并进行验证。在我们案例中，仅仅验证一下参数是否为键值对。  
```go
// Init is called during chaincode instantiation to initialize any
// data. Note that chaincode upgrade also calls this function to reset
// or to migrate data, so be careful to avoid a scenario where you
// inadvertently clobber your ledger's data!
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
  // Get the args from the transaction proposal
  args := stub.GetStringArgs()
  if len(args) != 2 {
    return shim.Error("Incorrect arguments. Expecting a key and a value")
  }
}
```
调用被验证后，我们将初始状态保存到账本。为此，我们用传入的键值对当参数，调用[ChaincodeStubInterface.PutState](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.PutState)。假如一切正常，返回一个peer.Response对象来表明初始化成功。  
```go
// Init is called during chaincode instantiation to initialize any
// data. Note that chaincode upgrade also calls this function to reset
// or to migrate data, so be careful to avoid a scenario where you
// inadvertently clobber your ledger's data!
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
  // Get the args from the transaction proposal
  args := stub.GetStringArgs()
  if len(args) != 2 {
    return shim.Error("Incorrect arguments. Expecting a key and a value")
  }

  // Set up any variables or assets here by calling stub.PutState()

  // We store the key and the value on the ledger
  err := stub.PutState(args[0], []byte(args[1]))
  if err != nil {
    return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
  }
  return shim.Success(nil)
}
```
#### 调用链码
首先，添加Invoke函数：
```go
// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The 'set'
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {

}
```
就像上面的`Init`函数，我们需要从`ChaincodeStubInterface`中取出参数。`Invoke`函数的参数将会是链码应用的函数名。在我们的案例中，应用只有两个函数：`set`和`get`，用来设置资产值或取资产的当前状态。我们首先调用[ChaincodeStubInterface.GetFunctionAndParameters](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.GetFunctionAndParameters)来获取链码应用的函数名和参数。   
```go
// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // Extract the function and args from the transaction proposal
    fn, args := stub.GetFunctionAndParameters()

}
```
然后，我们验证函数名是`set`或`get`，调用这些链码应用函数，通过`shim.Success`或`shim.Error`返回相应的响应，它们会将响应串行化为gRPC protobuf消息。  
```go
// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // Extract the function and args from the transaction proposal
    fn, args := stub.GetFunctionAndParameters()

    var result string
    var err error
    if fn == "set" {
            result, err = set(stub, args)
    } else {
            result, err = get(stub, args)
    }
    if err != nil {
            return shim.Error(err.Error())
    }

    // Return the result as success payload
    return shim.Success([]byte(result))
}
```
#### 实现链码应用
正如上面指出的那样，我们的链码应用需要实现两个函数，它们会被`Invoke`函数调用。为了访问账本状态，我们会用到链码shim API的[ChaincodeStubInterface.PutState](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.PutState)和[ChaincodeStubInterface.GetState](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#ChaincodeStub.GetState)函数。  
```go
// Set stores the asset (both key and value) on the ledger. If the key exists,
// it will override the value with the new one
func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 2 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
    }

    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
            return "", fmt.Errorf("Failed to set asset: %s", args[0])
    }
    return args[1], nil
}

// Get returns the value of the specified asset key
func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 1 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key")
    }

    value, err := stub.GetState(args[0])
    if err != nil {
            return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
    }
    if value == nil {
            return "", fmt.Errorf("Asset not found: %s", args[0])
    }
    return string(value), nil
}
```
#### 放在一起
最后，需要添加`main`函数，它会调用[shim.Start](http://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#Start)函数。下面是链码的完整源码：  
```go
package main

import (
    "fmt"

    "github.com/hyperledger/fabric/core/chaincode/shim"
    "github.com/hyperledger/fabric/protos/peer"
)

// SimpleAsset implements a simple chaincode to manage an asset
type SimpleAsset struct {
}

// Init is called during chaincode instantiation to initialize any
// data. Note that chaincode upgrade also calls this function to reset
// or to migrate data.
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
    // Get the args from the transaction proposal
    args := stub.GetStringArgs()
    if len(args) != 2 {
            return shim.Error("Incorrect arguments. Expecting a key and a value")
    }

    // Set up any variables or assets here by calling stub.PutState()

    // We store the key and the value on the ledger
    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
            return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
    }
    return shim.Success(nil)
}

// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // Extract the function and args from the transaction proposal
    fn, args := stub.GetFunctionAndParameters()

    var result string
    var err error
    if fn == "set" {
            result, err = set(stub, args)
    } else { // assume 'get' even if fn is nil
            result, err = get(stub, args)
    }
    if err != nil {
            return shim.Error(err.Error())
    }

    // Return the result as success payload
    return shim.Success([]byte(result))
}

// Set stores the asset (both key and value) on the ledger. If the key exists,
// it will override the value with the new one
func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 2 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
    }

    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
            return "", fmt.Errorf("Failed to set asset: %s", args[0])
    }
    return args[1], nil
}

// Get returns the value of the specified asset key
func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
    if len(args) != 1 {
            return "", fmt.Errorf("Incorrect arguments. Expecting a key")
    }

    value, err := stub.GetState(args[0])
    if err != nil {
            return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
    }
    if value == nil {
            return "", fmt.Errorf("Asset not found: %s", args[0])
    }
    return string(value), nil
}

// main function starts up the chaincode in the container during instantiate
func main() {
    if err := shim.Start(new(SimpleAsset)); err != nil {
            fmt.Printf("Error starting SimpleAsset chaincode: %s", err)
    }
}
```
#### 构建链码
编译链码：
```
$ go get -u --tags nopkcs11 github.com/hyperledger/fabric/core/chaincode/shim
$ go build --tags nopkcs11
```
执行上述`go get`时碰到问题：
```
# cd /opt/gopath/src/github.com/hyperledger/fabric; git pull --ff-only
error: Your local changes to the following files would be overwritten by merge:
        docs/source/kafka.rst
Please, commit your changes or stash them before you can merge.
```
解决办法是：
```
$ cd /opt/gopath/src/github.com/hyperledger/fabric;
$ git reset --hard
```
上述命令的含义是放弃本地的修改，以网上版本为准。  
回到原来的目录，重新执行`go get`，问题解决。  
#### 使用开发模式测试
通常链码由peer启动和维护。但在“开发模式”下，链码由用户构建和启动。在快速代码/构建/运行/调试周期转换的链码开发阶段，此模式非常有用。  
我们利用预生成的排序器和通道工件启动“开发模式”，获得一个示范开发网络。用户可以直接跳到编译链码过程和驱动调用。  

### 调试与测试
需要启动3个终端窗口。  
#### 窗口1：启动网络
```
$ cd /opt/fabric-samples/chaincode-docker-devmode
$ docker-compose -f docker-compose-simple.yaml up
```
上面的命令用`SingleSampleMSPSolo`排序器profile启动网络，并以开发模式启动peer。还启动了两个附加容器，一个是链码环境，另一个是与链码交互的CLI。创建和加入通道的命令被嵌入到CLI容器中，从而我们可以直接调用链码。  
#### 窗口2：构建和启动链码
用下列命令进入`chaincode`容器内的bash环境:
```
$ docker exec -it chaincode bash
root@d2629980e76b:/opt/gopath/src/chaincode#
```
虽然看上去与宿主机类似，其实已经在`chaincode`容器内。下面执行的都是该容器内的命令。下面编译你的链码：  
```
$ cd sacc
$ go build
```
运行链码：
```
$ CORE_PEER_ADDRESS=peer:7051 CORE_CHAINCODE_ID_NAME=mycc:0 ./sacc
```
上述链码随peer启动，链码日志表示它成功注册到peer。注意，现阶段的链码还没有与通道关联。这个会在后续的步骤中通过实例化命令做到。  
#### 窗口3：使用链码
虽然你处于`--peer-chaincodedev`模式，你仍然需要安装链码，因此生命周期系统链码可以像通常一样执行检查。将来这个需求可能在`--peer-chaincodedev`模式下移除。  
我们利用CLI容器执行这些调用：
```
$ docker exec -it cli bash
$ peer chaincode install -p chaincodedev/chaincode/sacc -n mycc -v 0
$ peer chaincode instantiate -n mycc -v 0 -c '{"Args":["a","10"]}' -C myc
```
现在发出一个调用将`a`的值改成`20`。
```
$ peer chaincode invoke -n mycc -c '{"Args":["set", "a", "20"]}' -C myc
```
最后，查询`a`，可以看到值是`20`。
```
$ peer chaincode query -n mycc -c '{"Args":["query","a"]}' -C myc
```
#### 测试新链码
默认我们仅挂载`sacc`。然而，你通过将它们添加到`chaincode`子目录来测试不同的链码（需要重启网络）。你可以在`chaincode`容器中访问它们。  

#### 链码加密
在某些情况下，需要对key关联的全部值或部分值进行加密。例如，当一个人的社会安全号码或地址在写入账本时，可能不希望这些数据以明文形式出现。链码加密利用[实体扩展](https://github.com/hyperledger/fabric/tree/master/core/chaincode/shim/ext/entities)，它内部包装了BCCSP，支持加密和椭圆曲线数字签名。例如，为了加密，链码的调用者通过临时字段传入加密密钥。然后可以将相同的密钥用于随后的查询操作，从而允许对加密的状态值进行解密。

对于更多详细信息和示范，看`fabric/examples`目录下的[Encc Example](https://github.com/hyperledger/fabric/tree/master/examples/chaincode/go/enccc_example)。特别注意utils.go 助手程序。该实用程序加载链码代码API和实体扩展，并构建一个新的函数类（例如`encryptAndPutState`＆`getStateAndDecrypt`），以便示例加密链码利用。因此，链码现在可以和基本的shim API结合起来，使`Get`和`Put`可以添加解密和加密功能。  

## 链码教程:链码运维
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/chaincode4noah.html)  
链码是一个程序，用Go、Node.js编写(未来会支持其他语言，如Java)，实现了一个规定的接口。链码运行在安全的Docker容器中，隔离于背书peer过程。链码通过应用提交的事务来初始化和管理账本状态。  
链码处理网络成员都同意的业务逻辑，所以它可以被认为是“智能合约”。链码创建的状态是不能直接被其他链码访问的（scoped）。但在同一个网络内（一般指通道？），通过适当的授权，一个链码可以调用其它链码而从访问它的状态。  
本章假定了一个叫诺亚的运维工程师，通过他的视角关注链码。根据诺亚的喜好，我们专注于链码的全生命周期维护，即包装、安装、实例化和升级链码的过程。  
### 链码生命周期
Hyperledger Fabric API允许与区块链网络中的不同节点(peer、orderer和MSP)交互，它还允许在背书peer节点上打包、安装、实例化和升级链码。Hyperledger Fabric各语言SDK对Hyperledger Fabric API进行抽象以利于应用开发，所以它可以用于管理链码生命周期。此外，Hyperledger Fabric API还可以通过CLI直接访问，这在本章我们会用到。  
我们提供了四个命令去管理链码生命周期：`package`、`install`、`instantiate`和`upgrade`。在未来的版本中，我们正考虑增加`stop`和`start`事务去禁用和重新启用链码，而不用实际卸载它。在链码被成功安装和实例化后，链码是活跃状态(运行中)，可以通过 `invoke`事务处理事务。链码可以在安装后多次升级（版本更新）。  
### 打包
链码包由三部分组成：
- 链码，就象在`ChaincodeDeploymentSpec`(简称CDS)中定义的。CDS通过code和其它属性(如名称和版本)来定义链码包  
- 一个可选的实例化策略，这有时被称为背书策略  
- 链码“拥有者”（实体）的一组数字签名  

签名用于以下目的：  
- 建立链码的所有权  
- 允许验证包裹的内容  
- 允许检测包裹篡改  
链码实例化事务的创建者需要通过链码的实例化策略的验证。  

#### 建包
有两个方法对链码打包，复杂的和简单的。当链码具有多个拥有者时，它需要被多个身份签名。这需要我们首先建立一个签名的链码包(`SignedCDS`)，然后发给其他拥有者进行签名。这是一个复杂流程。
简化流程是，当你部署的SignedCDS只有一个签名，而且签名者就是安装事务的发起者。（即安装一个自己签名的包到自己的peer）  
先讲复杂流程。  
创建一个签名的链码包，适用下列命令：  
```
$ peer chaincode package -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -v 0 -s -S -i "AND('OrgA.admin')" ccpack.out
```
`-s`选项表示创建一个多拥有者签名的包，如果不加就简单创建一个纯CDS。当指定了`-s`选项，如果有其他拥有者需要签名，则`-S` 选项必须设置。否则，会创建一个仅包含实例化策略的SignedCDS。  
`-S`选项使处理流程使用 `core.yaml`文件中`localMspid`属性下定义的MSP身份对包进行签名。  
`-S`选项使可选的。但如果一个包没有签名，它就不能被其他拥有者使用`signpackage`命令进行签名。  
`-i`选项用于为链码指定实例化策略。实例化策略与背书策略的格式相同，都是指定哪些身份可以实例化这个链码。在上面的例子中，只有`OrgA`的管理员(admin)可以实例化这个链码。如果没有设置策略，会使用默认策略，则只允许peer的MSP的管理员身份去实例化链码。  
#### 包签名
一个创建时签名的链码包可以被移交给其他拥有者查看和签名。流程支持out-of-band对链码包签名。  

[ChaincodeDeploymentSpec](https://github.com/hyperledger/fabric/blob/master/protos/peer/chaincode.proto#L78)可以选择被集体拥有者签名，而从创建一个[SignedChaincodeDeploymentSpec](https://github.com/hyperledger/fabric/blob/master/protos/peer/signed_cc_dep_spec.proto#L26)(或叫SignedCDS)。SignedCDS包含3个元素：  
 1. CDS包含的链码源码、名称和版本号。  
 2. 一个链码的实例化策略，表述为背书策略。  
 3. 链码拥有者列表，通过[背书](https://github.com/hyperledger/fabric/blob/master/protos/peer/proposal_response.proto#L111)定义。    

*【注意】：当链码在一些通道实例化时，这个背书策略通过out-of-band确定MSP身份。如果实例化策略没有指定，默认策略是通道的任何MSP管理员。*  

每个拥有者都对`ChaincodeDeploymentSpec`进行背书，背书方法是对CDS与拥有者身份（如证书）的组合结果进行签名(算法：sign(ProposalResponse.payload + endorser))。  
一个链码拥有者使用下面的命令对以前创建的签名包进行签名：
```
$ peer chaincode signpackage ccpack.out signedccpack.out
```
`ccpack.out`和`signedccpack.out`分别是输入包和输出包。`signedccpack.out`中包含了一个对包的新增签名，签名使用了本地MSP。  

#### 安装链码
安装(`install`)事务按规定格式对链码的源码进行打包，这个格式称为`ChaincodeDeploymentSpec`（或称CDS），该事务将链码安装在将来要运行它的peer节点上。  

*【注意】：你必须将链码安装在要运行链码的通道的每个背书peer节点上。*  

当`install`API简单给予了一个`ChaincodeDeploymentSpec`，它将使用默认实例化策略和包含一个空的拥有者列表。  

*【注意】：为了保证链码逻辑对网络上的其他成员保密，链码只安装在链码拥有者的背书peer节点上（可能存在一个或多个拥有者）。哪些没有链码的成员，不能是链码事务的背书者；也就是说，他们不能执行链码。然而，他们仍然可以验证和提交事务到账本。*  

为了安装链码，发送一个[SignedProposal](https://github.com/hyperledger/fabric/blob/master/protos/peer/proposal.proto#L104)到`lifecycle system chaincode`(LSCC)(LSCC会在[系统链码](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E7%B3%BB%E7%BB%9F%E9%93%BE%E7%A0%81)一节中描述)。例如，使用CLI安装**sacc**示范链码（前文在“链码教程:链码开发-[调试与测试](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E8%B0%83%E8%AF%95%E4%B8%8E%E6%B5%8B%E8%AF%95)”一节中描述过）的命令如下：
```
$ peer chaincode install -p chaincodedev/chaincode/sacc -n mycc -v 0
```
CLI内部为**sacc**创建一个`SignedChaincodeDeploymentSpec`，并发送它到本地peer，peer调用LSCC上的`Install`方法。`-p`选项指定了链码的路径，它必须位于用户`GOPATH`的源码树上，如`$GOPATH/src/sacc`。[CLI](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#cli)一节有这个命令选项的详细描述。  
请注意，为了安装在peer上，SignedProposal的签名必须来自peer的本地MSP管理员之一。  

#### 实例化
`instantiate`事务调用`lifecycle System Chaincode`(LSCC)在一个通道上创建和实例化某个链码。这是一个链码-通道绑定过程：一个链码可以绑定到任意数量的通道，独立和互不依赖地运行在每个通道上。换句话说，无论链码在多少个其他通道上安装和实例化，对于提交事务的通道状态是隔离的。  
`instantiate`事务的创建者必须满足包含在SignedCDS中的链码实例化策略，必须还是通道的写入者（这是通道创建时的配置之一）。这对于通道安全很重要，可以阻止恶意实体部署链码和欺骗成员执行非绑定通道的链码。  
例如，回想一下，默认实例化策略是任何通道MSP管理员，因此链码实例化事务的创建者必须是通道管理员的成员。交易提议到达背书者时，会根据实例化策略验证创建者的签名。在提交到账本之前，在交易验证期间再次执行此操作。  
实例化事务还为通道上的链码建立了背书策略。背书策略描述了事务结果可以被通道成员接受的证据需求。  
例如，使用CLI实例化**sacc**链码和用`john`和`0`初始化状态，命令如下：
```
$ peer chaincode instantiate -n sacc -v 1.0 -c '{"Args":["john","0"]}' -P "OR ('Org1.member','Org2.member')"
```
*【注意】上面的背书策略(CLI使用波兰语表示法)，所有的**sacc**事务需要一个Org1成员或Org2成员的背书。就是说，为了事务生效，Org1或Org2需要对调用(Invoke)**sacc**的执行结果签名。*   
实例化成功后，通道中的链码进入活动状态，准备好处理任意[ENDORSER_TRANSACTION](https://github.com/hyperledger/fabric/blob/master/protos/common/common.proto#L42)类型的事务提议。当事务到达背书peer时，它们会被并发处理。  
#### 版本更新
链码可以在任何时间更新版本，版本是SignedCDS的组成部分。SignedCDS的其它部分，如拥有者和实例化策略是可选项。然而，链码名称必须相同，否则它会被视为完全不同的链码。  
版本更新前，链码的新版本必须已经在背书者节点上安装。更新是一个类似于实例化的事务，它绑定新版本的链码到通道。绑定链码旧版本的通道仍然运行旧版本。话句话说，`upgrade`事务仅影响提交了更新事务的通道。  

*【注意】，由于链码的多个版本可能同时有效，更新过程不会自动删除就版本，因此用户必须临时管理它。*  

更新事务还是与`instantiate`事务由细微的不同：`upgrade`事务检查当前链码实例化策略，不是新策略(如果指定了策略)。这确保了只有在当前实例化策略中存在的成员才可以更新链码。  

*【注意】，在更新时，链码的`Init`函数将被调用去执行相关数据更新或重新初始化，所以链码更新时要小心避免重置状态。*  

#### 停止和启动
注意`stop`和`start`生命周期事务还没有被实现。然而，你可以手工停止链码，办法是从每个背书者peer删除链码容器和SingedCDS包。在每个运行背书peer节点的主机或虚机上删除链码容器，然后删除SignedCDS。  
```
(注意，为了从peer节点删除CDS，你需要先进入peer节点的容器。我们提供了干这个的工具脚本)
$ docker rm -f <container id>
$ rm /var/hyperledger/production/chaincodes/<ccname>:<ccversion>
```
停止在用于以受控方式进行升级的工作流程中是有用的，其中链码可以在发布升级之前在所有peer的信道上停止。  
#### CLI

*【注意】：我们正在评估是否发布平台专属Hyperledger Fabric peer二进制包。在此之前，你可以在一个docker容器中简单调用命令。*  

为了显示当前可用的CLI命令，在运行中的`fabric-peer`Docker容器中执行下列命令：
```
$ docker run -it hyperledger/fabric-peer bash
(peer chaincode --help)
```
它将显示类似的以下输出：
```
Usage:
  peer chaincode [command]

Available Commands:
  install     Package the specified chaincode into a deployment spec and save it on the peer's path.
  instantiate Deploy the specified chaincode to the network.
  invoke      Invoke the specified chaincode.
  list        Get the instantiated chaincodes on a channel or installed chaincodes on a peer.
  package     Package the specified chaincode into a deployment spec.
  query       Query using the specified chaincode.
  signpackage Sign the specified chaincode package
  upgrade     Upgrade chaincode.

Flags:
    --cafile string      Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
-h, --help               help for chaincode
-o, --orderer string     Ordering service endpoint
    --tls                Use TLS when communicating with the orderer endpoint
    --transient string   Transient map of arguments in JSON encoding
```
为了方便在脚本式应用中使用，`peer`命令在失败事件中总是产生非零的返回码。  
链码命令的例子：  
```
peer chaincode install -n mycc -v 0 -p path/to/my/chaincode/v0
peer chaincode instantiate -n mycc -v 0 -c '{"Args":["a", "b", "c"]}' -C mychannel
peer chaincode install -n mycc -v 1 -p path/to/my/chaincode/v1
peer chaincode upgrade -n mycc -v 1 -c '{"Args":["d", "e", "f"]}' -C mychannel
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","e"]}'
peer chaincode invoke -o orderer.example.com:7050  --tls --cafile $ORDERER_CA -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}'
```
### 系统链码
系统链码与普通链码具有相同的编程模型，知识它运行在peer进程中，而不是在隔离的容器中。因此，系统链码构建在peer可执行文件中，它不会遵循上述同样的生命周期。特别是，安装、实例化和版本更新不会用在系统链码上。  
系统链码的目的是减少peer和链码间gRPC通信成本，和管理灵活性的折中。例如，系统链码只能与peer程序一起更新。它必须以一套固定参数注册，且不能有背书策略或背书策略函数。  
Hyperledger Fabric中使用系统链代码来实现许多系统行为，以便系统集成商可以根据需要替换或修改它们。  
当前系统链码的列表：  
1. [LSCC](https://github.com/hyperledger/fabric/tree/master/core/scc/lscc) 生命周期系统链码，处理上述的生命周期请求。  
2. [CSCC](https://github.com/hyperledger/fabric/tree/master/core/scc/cscc) 配置系统链码，处理peer端的通道配置。  
3. [QSCC](https://github.com/hyperledger/fabric/tree/master/core/scc/qscc) 查询系统链码，提供账本查询API，例如获取区块和交易。  
4. [ESCC](https://github.com/hyperledger/fabric/tree/master/core/scc/escc) 背书系统链码，通过签署交易提议响应来处理背书。  
5. [VSCC](https://github.com/hyperledger/fabric/tree/master/core/scc/vscc) 验证系统链码，处理事务验证，包括检查背书策略和多版本并发控制。  

更改或覆盖这些系统链码要小心，特别是LSCC、ESCC和VSCC，因为它们处在主事务的运行路径上。值得注意的是，VSCC将区块将提交到账本之前的验证，通道中的所有peer计算相同的验证以避免账本分歧（非确定性）是很重要的。因此，如果修改或更换VSCC，需要特别小心。  


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

## 会员服务提供者(MSP)
### MSP配置
MSP的默认实现中，需要指定一组参数，用于身份(证书)验证和签名验证。这些参数在[RFC5280](http://www.ietf.org/rfc/rfc5280.txt)中描述。包括：
- 一些自签名(X.509)证书，构成了信任根(root of trust)  
- 一些X.509中间CA证书  
- 一些X.509证书，代表MSP的管理员，管理员证书由某个信任根签署  
- 一些这个MSP的OU证书  
- 一些撤销证书(CRL)  
- 一些自签名证书(X.509)，用于TLS
- 一些X.509证书，表示中间TLS CA证书

### 怎样生成MSP证书和密钥?
1. 使用[Openssl](https://www.openssl.org/)。需要强调的是Fabric不支持RSA密钥证书。  
2. 使用`cryptogen`工具，在[快速开始](http://hyperledger-fabric.readthedocs.io/en/latest/getting_started.html)中解释过。  
3. 使用[Hyperledger Fabric CA](http://hyperledger-fabric-ca.readthedocs.io/en/latest/)也可以生成配置MSP的密钥和证书。  

### 在peer&oderer端建立MSP
为peer或oderer建立一个本地MSP，管理员需要创建一个文件夹(如`$MY_PATH/mspconfig`)，下面包含6个子文件夹和一个文件：  
1. 一个`admincerts`文件夹，包含几个PEM文件，每个文件对应一个管理员证书  
2. 一个`cacerts`文件夹，包含几个PEM文件，每个文件对应到一个根CA证书
3. (可选)一个`intermediatecerts`文件夹，包含几个PEM文件，每个文件对应一个中间CA证书  
4. (可选)一个`confi.yaml`文件，包含所考虑的OU的信息  
5. (可选)一个`crls`文件夹，包含了撤销证书列表(CRL)  
6. 一个`keystore`文件夹，包含一个PEM文件，是节点的签名密钥，不支持RSA密钥  
7. 一个`signcerts`文件夹，包含一个PEM文件，是节点的X.509证书  
8. (可选)一个`tlscacerts`文件夹，包含几个PEM文件，每个对应到一个TLS根CA证书  
9. (可选)一个`tlsintermediatecerts`文件夹，包含几个PEM文件，每个对应一个中间TLS CA证书   

在节点的配置文件中(对peer是core.yaml，对orderer是orderer.yaml)，需要指定到mspconfig文件夹的路径，和节点MSP的身份(Id)。到msconfig的路径是相对于FABRIC_CFG_PATH，对于peer由参数`mspConfigPath`的值定义，对于orderer由参数`LocalMSPDir`值定义。节点MSP的身份，对于peer由参数`localMspId`的值定义，对于orderer由参数`LocalMSPID`的值定义。  
这些变量可以被环境变量覆盖，对于peer节点由前缀为CORE的环境变量覆盖(如CORE_PEER_LOCALMSPID)，对于orderer节点由前缀为ORDERER的环境变量覆盖(如ORDERER_GENERAL_LOCALMSPID)。  
注意，对于建立orderer，需要生成和提供系统通道的创世区块(genesis block)到orderer节点。MSP配置对这个区块的需求在下一节讲到。  
重新配置一个“本地”MSP只能手工进行，需要peer和orderer进程重启。在后续版本中，我们的目标是提供在线/动态重新配置（即使用一个用节点管理系统链码来避免停止节点）。

### 通道MSP建立
在创建系统时，需要指定出现在网络中的所有MSP的验证参数，并且包括在系统通道的创世区块中。回想一下前文讲到的组成MSP身份的MSP验证参数，信任证书的根、中间CA和管理员证书，还有OU规范和CRL。系统创世区块被提供给orderer（在orderer的创建阶段），允许它们认证通道创建请求。如果区块包括两个相同身份的MSP，oderer会拒绝，使网络自举失败。  
对于应用通道，通道的创世区块包含了管理通道的MSP验证组件。我们强调这是应用的责任：确保在指示其一个或多个peer加入通道之前，通道的创世区块(或最新的配置区块)中包含了正确的MSP配置信息。  
在使用`configtxgen`工具启动一个通道时，需要配置通道MSP，办法是将MSP的验证参数包含在`mspconfig`文件夹，有在`configtx.yaml`的相应章节设置文件夹路径。  
重新配置通道的MSP，包括MSP的CA更新CRL公告，通过MSP的管理员证书之一的所有者创建`config_update`对象来实现。管理员管理的客户端应用会将这个更新广播到MSP出现的通道中。  

### 最佳实践
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
6. CA和TLS CA
  MSP身份的根CA河MSP TLS证书的根CA(和相关中间CA)需要放在不同的文件夹。这避免不同类型证书的冲突。虽然不禁止在MSP身份和TLS证书之间复用同一个CA，但最佳实践建议避免在生产系统中这样做。  

## 通道配置(configtx)
Hyperledger Fabric区块链网络的共享配置被保存在一个集合配置事务中，每个通道一个。每个配置事务通常通过configtx短名称引用。  
通道配置有如下重要属性：
1. 版本(**Versioned**)：配置的所有元素都有一个关联的版本，每次修改都会更新。此外，每次更新配置都会收到一个顺序号。  
2. 权限(**Permissioned**)：配置的每个元素有一个关联的策略，用于管理是否允许对该元素进行修改。任何拥有以前configtx副本（并且没有其他信息）的人都可以根据这些策略验证新配置的有效性。
3. 层次(**Hierarchical**)：根配置组包含子组，每个层次组都有关联的值和策略。这些政策可以利用层次结构从下层策略中推导出一个层次的策略。
解剖一个配置
### 排序系统通道配置
排序系统通道需要定义排序参数，和创建通道的合伙人。对一个排序服务必须有一个排序系统通道，它是创建的第一个通道（指自举时）。推荐不要在排序系统通道的创世配置中定义应用段，但可以在测试时这样做。注意，对排序系统通道具有读权限的成员可以看到所有通道的创建，所以这个通道的访问权限需要严格控制。  

### 应用通道配置
通道的应用配置设计用于应用类型的事务。它的定义类似于：
```
&ConfigGroup{
    Groups: map<string, *ConfigGroup> {
        "Application":&ConfigGroup{
            Groups:map<String, *ConfigGroup> {
                {{org_name}}:&ConfigGroup{
                    Values:map<string, *ConfigValue>{
                        "MSP":msp.MSPConfig,
                        "AnchorPeers":peer.AnchorPeers,
                    },
                },
            },
        },
    },
}
```
### 通道创建
当排序节点收到一个不存在的通道的`CONFIG_UPDATE`，排序节点就假定这是个通道创建请求，然后执行下列操作：
1. 排序节点对通道创建请求的合伙人身份进行验证。它通过查看顶层group的`Consortium`值来验证。  
2. 
## 通道配置(configtxgen)

## 背书策略
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html)  
背书策略是用来指导peer如何决定事务是否得到适当的赞同。当peer收到一个事务时，作为事务验证流程的一部分，它调用与事务的链码相关联的VSCC(验证系统链码)来确定事务的有效性。回忆一下，一个事务包含来自任何背书peer的一个或多个背书。VSCC的任务是做出以下决定：
- 所有的背书是有效的（也就是通过预期的消息它们来自有效证书的有效签名）  
- 有恰当数量的签名  
- 背书来自预期的源  

背书策略是指定第二和第三点的一种方式。  

### CLI中的背书策略语法
(本节中的身份都是principal)
在CLI中，策略用一种简单的语言来表达，即基于身份的布尔表达式。  
一个身份通过MSP描述，它的任务是验证签名者身份和签名者角色存在于MSP中。当前，支持两种角色，**member**(成员)和**admin**(管理员)。身份用`MSP`.`ROLE`描述，这里`MSP`是MSP ID(必填)，`ROLE`是`member`或`admin`二选一。验证身份(principal)的例子是`Org0.admin`(`Org0`MSP的任何管理员)或`Org1.member`(`Org1`MSP的任何成员)。  
这个语言的语法是：  
```
EXPR(E[, E...])
```
这里`EXPR`是`AND`或`OR`，表示两个布尔表达式。`E`可以是一个身份或另一个嵌入的`EXPR`。（运算符在前面，这被称为波兰语表示法）   

例如：  
- `AND('Org1.member', 'Org2.member', 'Org3.member')` 要求三位身份分别签名  
- `OR('Org1.member', 'Org2.member')` 两身份中任何一个签名都可以  
- `OR('Org1.member', AND('Org2.member', 'Org3.member'))` 要求`Org1` MSP成员的一个签名，或`Org2`MSP成员和`Org3`MSP成员的分别签名  
### 定义链码的背书策略
使用这个语言，链码部署者可以为某个链码指定特定策略。注意，默认策略是需要`DEFAULT`MSP成员的一个签名。用CLI实例化链码时，没有指定策略就使用默认策略。  
实例化时用`-P`选项指定策略。例如：
```
$ peer chaincode instantiate -C <channelid> -n mycc -P "AND('Org1.member', 'Org2.member')"
```
这个命令部署链码`mycc`，使用的策略是`AND('Org1.member', 'Org2.member')`，即同时需要Org1和Org2的成员签署事务。  

# 架构
## 架构解释
Hyperledger Fabric架构具有以下优点：  
- **Chaincode信任灵活性**。该架构将链码（区块链应用）信任假设与排序信任假定分开。换句话说，排序服务可以由一组节点（orderer）提供，并容忍的一些节点的失效或欺诈；并且对于每个链码，背书者可能不同。  
- **可扩展性**。由于背书节点只负责特定链码，与排序节点无关，系统可以比通过相同节点完成这些功能更好地扩展。具体而言，当不同的链码指定无关的背书节点时，这引入了背书节点之间的分区机制从而允许链码的并行运行（背书）。  
- **保密**。该架构便于部署对于其事务的内容和状态更新具有机密性要求的链码。
- **共识模块化**。该体系结构是模块化的，并允许插件式的共识（即排序服务）实现。  
### 1.系统架构
区块链是由多个彼此通信的节点组成的分布式系统。区块链运行程序称为链码，保存状态和总帐数据，并执行事务。链码是链码调用事务操作的核心元素。事务必须被“背书”，只有背书的事务可能会被提交，并对状态产生影响。管理功能和参数存在一个或多个特殊的链码，统称为系统链码。  
#### 1.1 事务
事务可能有两种类型：
- **部署事务**创建新的链码并将一个程序作为参数。当部署事务成功执行时，链码就被安装在区块链上。  
- **调用(Invoke)事务**在先前部署的链码的上下文中执行操作。调用事务是指链码及其提供的某个函数。链码会执行指定的函数 - 这可能涉及修改相应的状态，并返回一个输出。  
如后面所述，部署事务是调用事务的特殊情况，其中创建新链的部署事务相当于系统链码上的调用事务。  
**备注**： 这个文件目前假设一个事务要么创建新的链码，要么调用已经部署的链码提供的操作。本文档尚未描述：a）对查询（只读）事务（包含在v1中）的优化，b）对交叉链码事务（post-v1特性）的支持。  
#### 1.2 区块链数据结构
**1.2.1状态**  
区块链的最新状态（或简称为状态）被建模为版本化的键值库（KVS），其中键名称和值是任意的blob。这些条目由运行在区块链上的链码（应用程序）通过`put`和`get` KVS来操纵。状态被持久化存储，状态的更新被日志记录。请注意，采用版本化的KVS作为状态模型，实现可以使用实际的KVS，还可以使用RDBMS或任何其他解决方案。  
**1.2.2账本**  
账本提供了一个在系统运行过程中发生的所有成功状态变化（我们称为有效事务）和不成功的改变状态尝试（我们称为无效事务）的可验证历史。  
账本是由排序服务（参见1.3.3节）构建，是一个由事务区块（有效或无效）组成的有序哈希链。哈希链定义了账本中的总的区块顺序，每个区块都包含一个完全有序的事务数组。这会在所有事务中强加顺序。  
账本保存在所有peer节点，并可保存在部分排序节点(可选)。在排序节点上下文中，我们称账本为`OrdererLedger`，然而在peer上下文中，我们称账本为`PeerLedger`。`PeerLedger`与`OrdererLedger`的区别在于，peer在本地维护一个位掩码(bitmask)，将有效的事务从无效的事务中分离出来（更多细节见第XX章）。  
像第XX节（v1之后的功能）中所述的那样，peer可能会删除`PeerLedger`。排序节点维护`OrdererLedger`是为了容错性和`PeerLedger`的可用性，并且可以决定在什么时候修剪(prune)它，前提是排序服务的属性（参见第1.3.3节）得到维护。  

账本允许peer重放所有事务的历史记录并重建状态。因此，如1.2.1节所述的状态是可选的数据结构。  
#### 1.3节点
节点是区块链的通信实体。一个“节点”只是一个逻辑功能，也就是不同类型的多个节点可以在同一个物理服务器上运行。重要的是节点如何分组在“信任域”中，并与控制它们的逻辑实体相关联。  
存在三种类型的节点：  
1. **客户端**或**提交客户端**：它发出实际调用事务到背书节点，广播提议事务到排序服务。  
2. **Peer**：它提交事务、维护状态和一个账本副本(参看1.2节)。此外，peer还可以具有背书者角色。  
3. **排序服务节点**或**orderer**：运行通信服务的节点，实现交付担保，如原子或全部排序广播。  

**1.3.1客户端** 
客户代表代表最终用户行事的实体。它必须连接到peer与区块链进行通信。客户端可以连接到其选择的任何peer端。客户端创建并调用事务。  

如第2节详细描述的那样，客户端同时与peer和排序服务通信。  

**1.3.2 Peer**  
peer以区块的形式从排序服务接收顺序的状态更新，并维护状态和账本。  

peer还可以担当**背书peer**的特殊角色(或称**endorser**)。背书peer的特殊功能是针对特定的链码进行的，并且包含在提交前的背书事务中。每个链码可以指定一个背书策略，策略可以指向一组背书peer。该策略为有效的事务背书定义了必要和充分的条件（通常是一组背书签名），如后面的第2和第3节所述。有一种部署新链码的特殊部署事务，它的（部署）背书策略是指定为系统链码的背书政策。  

**1.3.3排序服务节点(Orderer)**  
排序服务(orderer)是一个通信架构，它提供投递担保。排序服务可以用不同的方式实现：从中心式服务（例如在开发和测试中使用）到针对不同网络和节点失效模型的分布式协议。

排序服务为客户和peer提供共享的*通信通道*，为包含事务的消息提供广播服务。客户端连接到该通道，并可以在该通道上广播消息，然后传送给所有peer。该通道支持所有消息的原子交付，也就是全排序交付和（特定实现）可靠性的消息通信。换句话说，通道向所有连接的peer输出相同的消息，并以相同的逻辑顺序输出到所有peer。这种原子通信保证也被称为*全序广播*、*原子广播*或分布式系统下的*共识*。通信消息是包含在区块链状态中的候选事务。  

**分区(排序服务通道)**。排序服务可以支持多通道，类似发布/订阅（pub / sub）消息系统的主题。客户端可以连接到给定的通道，然后可以发送消息并获取到达的消息。通道可以被认为是分区 - 连接到一个通道的客户端不知道其他通道的存在，但客户端可能连接到多个通道。尽管Hyperledger Fabric中包含的一些排序服务实现支持多个通道，但为了简化表示，在本文档的其余部分，我们假设排序服务包含单个通道/主题。  

**排序服务API**。peer通过排序服务提供的接口连接到排序服务提供的通道。排序服务API包含两个基本操作（很常见的*异步事件*）：  
- `broadcast(blob)`: 客户端调用这个广播二进制消息`blob`到通道。这在BFT上下文中也叫做`request(blob)`，当发送请求到一个服务时。  
- `deliver(seqno, prevhash, blob)`：(略)

**账本和区块信息**。账本(参看1.2.2节)包含排序服务输出的所有数据。简而言之，它是一系列`deliver(seqno, prevhash, blob)`事件，根据前面所述的`prevhash`计算形成一个哈希链。  
大多数情况下，出于效率原因，不输出单个事务（blob），排序服务将对blob进行分组(批处理)，并在一个`deliver`事件输出区块。在这种情况下，排序服务必须强制和传达每个块内的blob的确定性排序。区块中的blob数量可以由排序服务实现动态地选择。  

下面为便于表述，我们定义排序服务属性（本小节的其余部分），并解释事务背书的工作流程。假定一个blob产生一个`deliver`事件。一个区块对应一组顺序的`deliver`事件（每个blob一个事件）。区块本身也对应一个`deliver`事件，依靠排序服务，多个区块顺序排列组成区块链。  

### 2.事务背书的基本流程
#### 2.1 客户端创建一个事务并将它发送到选择的背书peers  
为了调用一个事务，客户端发送一个`PROPOSE`消息给它所选择的一组背书peer（可能不是同一时间 - 见2.1.2节和2.3节）。对于给定的`chaincodeID`客户端根据背书策略(看第3节)可以获得一组背书peer。例如，根据给定`chiancodeID`事务可以发送给所有背书peer。也就是说，一些背书人可能会离线，其他人可能会反对并选择不赞成事务。发起客户端利用目前可用的背书节点尝试满足策略表达式的要求。  

接下来，我们首先详细描述PROPOSE消息格式，然后讨论提交客户端和背书人之间可能的交互模式。  
**2.1.1 PROPOSE消息格式**  
**2.1.2 消息模式**  

#### 2.2 背书peer模拟一个事务并生成一个背书签名  
从客户端收到`<PROPOSE,tx,[anchor]>`消息后，背书peer`epID`首先验证客户端签名`clientSig`，然后模拟一个事务。如果客户端指定了`anchor`，则背书peer模拟事务的方法是，在本地KVS中读取与版本号`anchor`相匹配的keys。  
模拟一个事务包括背书peer临时性执行一个事务(`txPayload`)（调用事务中`chaincodeID`指定的链码）和背书peer本地保存的状态副本。（用状态副本和临时事务可以得到一个临时的新状态）  
作为执行的结果，背书peer计算*读版本依赖*(`readset`)和*状态更新*(`writeset`)，在数据库语言中也叫*MVCC+postimage info*。  
回想一下状态由键值对组成。所有的键值对条目是版本化的，也就是每个条目都包含有序的版本信息，当通过一个key更新库中的值时版本号会增长。peer解释(模拟执行)事务，记录链码访问的键值对，但不会真的更新状态。进一步来说：
- 在背书peer执行事务前，给定状态`s`，对于事务读取的每个key`k`，键值对`(k,s(k).version)`被添加到`readset`。  
- 此外，对于每个key`k`事务更新为新值`v'`，键值对`(k,v')`被添加到`writeset`。或者，`v'`可能是以前值(`s(k).value`)的新值的增量。  

如果客户端在`PROPOSE`消息中指定了`anchor`，则客户端指定的`anchor`必须等于背书peer模拟事务时产生的`readset`。  
然后，peer将内部`tran-proposal`（即`tx`）转发作为其背书事务逻辑的组成部分，称为**背书逻辑**。默认情况下，peer中的背书逻辑接受`tran-proposal`并简单地签名`tran-proposal`。然而，背书逻辑可能解释任意函数，例如，用`tran-proposal`和`tx`作为输入与遗留系统交互来判断是否批准事务。  
如果背书逻辑决定对一个事务进行背书，它发送`<TRANSACTION-ENDORSED, tid, tran-proposal,epSig>`消息到发起客户端(`tx.clientID`)，这里：
- `tran-proposal := (epID,tid,chaincodeID,txContentBlob,readset,writeset)`，`txContentBlob`是链码/事务指定信息。  
- `epSig`是背书peer在`tran-proposal`上的签名  

另外，如果背书逻辑拒绝对事务签名，背书界面会发送一个`(TRANSACTION-INVALID, tid, REJECTED)`消息到发起客户端。  
注意，背书节点在这一步不会修改状态，因事务模拟而生成的更新不会影响状态。  

#### 2.3 发起客户端收集事务背书并广播到排序服务
发起客户端等待，直到它收到“足够的”消息和签名(`TRANSACTION-ENDORSED, tid, *, *`)，得出事务提议被背书的结论。正如2.1.2节所讨论的，这可能涉及一个或多个与背书人交互的往返。  

“足够的”的确切数量取决于链码背书策略（另见第3节）。如果背书策略得到满足，事务就获得背书; 注意它还没有提交。来自背书peer的签名`TRANSACTION-ENDORSED`消息集合将建立一个背书的事务，称为*背书*(`endorsement`)。

如果发起客户端没有收集到事务的背书，则放弃该事务，并选择稍后重试。  

对于含有有效背书的事务，我们现在开始使用排序服务。发起客户端使用`broadcast(blob)`调用排序服务，这里`blob=endorsement`。如果客户端没有直接调用排序服务的能力，可以通过自己选择的peer代理它的广播。这样的peer必须被客户信任：不会从`endorsement`删除任何消息，除非事务被认为是无效的。但是请注意，代理peer可能不会编造有效的endorsement。  

#### 2.4 排序服务
当一个事件`deliver(seqno, prevhash, blob)`发生，并且peer已经应用了blob的序列号低于`seqno`的所有的状态更新，peer做下面这些：
- 它根据`blob.tran-proposal.chaincodeID`指向的链码策略检查`blob.endorsement`看是否有效。  
- 在一个典型的情况下，它也验证依赖关系（blob.endorsement.tran-proposal.readset）同时没有被违反。在更复杂的用例中，背书的`tran-proposal`字段可能有所不同，在这种情况下，背书策略（第3部分）指定了状态如何演变。  
（省略一些）

### 3. 背书策略
#### 3.1 背书策略规范
一个**策略**，是对事务进行背书的条件。区块链peer拥有一套预先设定的背书策略，由安装特定链码的`deploy`事务引入。背书策略可以参数化，这些参数可以由`deploy`事务指定。  

为了保证区块链和安全属性，这套认可策略应该是一组经过验证的策略，确保有限的执行时间（终止），确定性，性能和安全保证。  

不允许动态增加背书策略，日后可以予以支持。  

#### 3.2 对背书策略的事务评估
只有通过策略背书，事务才被宣布有效。链码的调用事务首先必须获得链码策略的背书，否则将不会被提交。这是通过发起客户端和peer之间的交互来进行的，如第2节所述。  

形式上，背书策略是对背书的一个判断，并可能进一步陈述评估为“真”或“假”。对于部署事务，根据系统范围的策略（例如从系统链码）获得背书。

(省略一些)

### 共识过程(视频截图)
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
## 术语
#### 区块链网络
区块链网络至少由一个peer（负责批准和提交事务）、一个排序服务和成员服务组件（CA，分发和撤销代表用户身份和权限的加密证书）组成。  

#### Peer
peer是一个组件，负责执行事务和维护账本。peer有两种角色，endorser和committer。架构被设计成peer总是一个提交者，但不一定是背书者。peer在排序事务中不起作用。  

#### 成员
成员是在区块链网络中维护组件（Peers，Orderers和应用程序）的参与者（例如公司或组织）。成员通过其CA证书进行标识（即注册证书）。最终用户会利用会员的peer在特定的通道上执行事务操作。  

#### 事务
指被授权的最终用户对账本进行读/写的操作。有三种的事务类型 - 部署，调用和查询。  

#### 最终用户
最终用户是通过一组发布的API（即hfc SDK）与区块链交互的人。可以设置一个管理员用户，他通常将权限授予成员组件，而客户端用户通过管理员用户进行适当的身份验证，可以在各种通道上驱动链码应用程序（部署，调用，查询）。在自动执行事务的情况下，应用程序本身也可以被认为是最终用户。  

#### 排序服务
一个集中或非几种的服务，对区块中事务进行排序。您可以选择“排序”功能的不同实现方式 - 例如：简化和测试的“solo”，用于碰撞容错的Kafka，或用于拜占庭容错的sBFT/PBFT。您也可以开发自己的协议来插入服务。  

#### 共识
贯穿整个事务流程的一个更广泛的术语，用来按顺序生成一致性并确认按一组事务构组成区块的正确性。  

#### 排序者(Orderer)
构成排序服务的网络实体之一。一个排序服务节点（OSN）的集合，用来将事务排序进区块。在“独奏”的情况下，只需要一个OSN。事务被“广播”给排序者，然后作为区块“交付”到适当的通道。  

#### 背书者(Endorser)
一个特定的peer角色，背书peer负责模拟事务，并且防止不稳定或不确定的事务通过网络。事务以事务提议的形式发送给背书者。所有的背书peer同时也是committer peer（即他们写账本）。  

#### 提交者(Committer)  
一个特定的peer角色，负责将有效事务写入某通道的账本。一个peer可以同时充当背书者和提交者，当很多时候只是充当提交者。  

#### 引导(Bootstrap)
对等网络的引导，在这个对等网络中，参与者之间传播策略，系统链接代码和密码资料（证书），以及订购网络的引导。订购网络的启动必须在对等网络的引导之前，因为对等网络取决于订购服务的存在。网络只需要“自举”一次。

块
一批有序的交易，可能包含无效性质的交易，交付给同行进行验证和交付。



## 备忘
[Hyperledger Composer](https://hyperledger.org/projects/composer)  
[Hyperledger Composer教程](https://developer.ibm.com/courses/all/ibm-blockchain-foundation-developer/?course=begin#11982)，含composer的安装。  
[另一个官方文档](http://fabrictestdocs.readthedocs.io/en/latest/glossary.html)  
### 问题
#### VM翻墙问题
```
$ curl -sSL https://goo.gl/fMh2s3 | bash
```
如果执行后无反应，这是因为该链接被 墙。设置proxy:
```
$ export https_proxy=http://10.180.36.75:25378
$ export http_proxy=http://10.180.36.75:25378
```
25378是宿主机的翻&墙软件端口号，ip是宿主机ip。  