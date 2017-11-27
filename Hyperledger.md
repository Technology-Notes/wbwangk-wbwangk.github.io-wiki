(/e/vagrant10/ambari-agrant/centos7.3 c7303)
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
ChainCode的集成平台，为ChainCode提供安全的部署、运⾏的环境。  
#### Event
贯穿于其他各个组件中间，为各个组件间的异步通信提供了技术实现。  

### Fabric 0.6 运行时架构
保证区块链的私有性，机
密性，可审计性
• 可拔插的共识框架  
• PBFT,	SIEVE(proto),NOOPS  
• Chaincode 运⾏环境(Go,Java WIP)  
• Docker container (user-cc)  
• In peer process (system-cc)  
• Client Node.js SDK  
• REST APIs  
• Basic CLI  

获取Compose⽂件:  
```
$ git clone h1ps://github.com/yeasy/docker-compose-files
```
fabric安装参考: [Fabri Getting Started](http://hyperledger-fabric.readthedocs.io/en/latest/getting_started.html)  
安装docker-compose引擎：
```
$ curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```
可以到docker-compose的[发布页面](https://github.com/docker/compose/releases)看最新的版本号。
#### go lang
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

### 启动 hyperledger
```
$ cd /opt
$ git clone https://github.com/yeasy/docker-compose-files.git
$ cd docker-compose-files/hyperledger_fabric/latest
$ make start
```
花费几个小时，下载了数个docker镜像，然后查看docker镜像：
```
$ docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             SIZE
yeasy/hyperledger-fabric-peer   1.0.4               5e7484dcb7d6        5 days ago          1.02GB
hyperledger/fabric-tools        x86_64-1.0.4        6051774928a6        3 weeks ago         1.33GB
hyperledger/fabric-orderer      x86_64-1.0.4        b17741e7b036        3 weeks ago         151MB
```
查看docker容器：
```
$ docker ps
CONTAINER ID        IMAGE                                   COMMAND                  CREATED             STATUS                          PORTS               NAMES
758aeb009e33        yeasy/hyperledger-fabric-peer:1.0.4     "peer node start"        21 hours ago        Restarting (1) 12 seconds ago                       peer0.org2.example.com
d15a5bda0c0c        yeasy/hyperledger-fabric-peer:1.0.4     "peer node start"        21 hours ago        Restarting (1) 12 seconds ago                       peer1.org2.example.com
44f425a9309d        yeasy/hyperledger-fabric-peer:1.0.4     "peer node start"        21 hours ago        Restarting (1) 12 seconds ago                       peer1.org1.example.com
1b639f5b35ac        hyperledger/fabric-tools:x86_64-1.0.4   "bash -c 'cd /tmp; s…"   21 hours ago        Up 35 minutes                                       fabric-cli
d52d739b2122        yeasy/hyperledger-fabric-peer:1.0.4     "peer node start"        21 hours ago        Restarting (1) 11 seconds ago                       peer0.org1.example.com
```
### Chaincode
### shared ledger
Ledger provides a verifiable history of all successful state changes. It is THE system of record for a
business. Business will have multiple ledgers for multiple business networks in which they participate. The
ledger is SHARED, REPLICATED and PERMISSIONED.
 Transaction – an asset transfer onto or off the ledger
John gives a car to Anthony (simple)
 Contract – conditions for transaction to occur
If Anthony pays John money, then car passes from John to Anthony (simple)
If car won't start, funds do not pass to John (as decided by third party arbitrator) (more complex)

Block ledger
 - File system based, only new Blocks appending
 - Blocks are stored on all committers and optional subset of ordering service nodes
State ledger
 - World/Ledger state holds current value of smart contract data
 e.g. vehicleOwner=Daisy
 - KVS hidden from developer by chaincode APIs
 e.g. GetState(), PutState(), GetStateByRange(), etc…
 - Stored on all committers
History ledger
 - Holding historic sequence of all chain code transactions
 e.g. updateOwner(from=John, to=Anthony); updateOwner (from=Anthony, to=Daisy);etc
 - Index stored in KVS and hidden from developer by chaincode APIs
 e.g. GetHistoryForKey()
 - Stored on all committ

### 共识机制
#### Nodes and roles
- **Peer**: Commits transacUons, maintains ledger and state 
- **Endorsing peer**: Specialised peer that receives a transaction proposal for endorsement, responds granting or denying endorsement 
- **Ordering peer**: Approves the inclusion of transaction blocks into the ledger and communicates with peer and endorsing peer nodes

### 隐私与安全
交易方持有多种类型的证书，交易不同环节将使用如下这些类型的证书：  
• E-Cert（Enrollment Cert）  
  – 长期持有，携带或可以追溯使用者信息  
  – 用于身份认证  
• T-Cert（Transaction Cert）  
  – 每个交易时生成，用于交易的签名  
• TLS-Cert，长期持有，主要用于SSL/TLS通讯  

## Hyperledger Fabric Samples
[Hyperledger官方GETTING STARTED](https://hyperledger-fabric.readthedocs.io/en/latest/samples.html)  
```
$ cd /opt
$ git clone -b master https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples
$ curl -sSL https://goo.gl/fMh2s3 | bash
```
如果执行后无反应，也可以下载`https://goo.gl/fMh2s3`，自己创建脚本文件，手工执行。执行后，在`bin`目录下多了几个可执行文件：
```
cryptogen,configtxgen,configtxlator, peer
```
然后，自动下载了很多docker镜像。  