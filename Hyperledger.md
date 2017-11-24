(/e/vagrant10/ambari-agrant/centos7.3 c7303)
### Hyperledger Projects
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
$ curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
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
Peer:	Commits	transacUons,	maintains	ledger	and	state	
Endorsing	peer:	Specialised	peer	that	receives	a	transacUon	
proposal	for	endorsement,	responds	granUng	or	denying	
endorsement	
Ordering	peer:	Approves	the	inclusion	of	transacUon	blocks	
into	the	ledger	and	communicates	with	peer	and	endorsing	
peer	nodes