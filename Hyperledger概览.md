## Hyperledger概述
### 什么是区块链
#### 一个分布式账本
所有交易数据被保存到账本中。账本中的记录只能增加，不能修改。所有peer都保存了交易数据的完整副本。每间隔一定时间或交易量达到一定数量，形成一个区块(Block)。区块一旦形成就不能修改。多个区块顺序链接在一起形成区块链。  

交易日志保存到区块链中（有点像数据库日志），账本的当前状态体现在全局状态中。全局状态保存在NoSQL数据库中，如Leveldb、Couchdb。  
#### 智能合约(smart contract)
在Hyperldedger Fabric中链码(chaincode)和智能合约(smart contract)是一个意思。  
链码是封装用于创建和修改资产的业务逻辑和交易处理指令的软件。当外部应用程序需要与账本交互时，它需要调用链码。在大多数情况下，链码仅与账本的的数据库组件-全局状态（例如查询）交互，而不是与交易日志(区块链)交互。  
链码可以用几种编程语言来实现。当前支持的链式代码语言是Go和node.js，支持未来发行版中的Java和其他语言。  
Hyperledger Fabric支持私有网络（使用通道）。每个peer（参与者或称成员）可以加入多个通道，通道内的peer同享同一个账本。  
具有正确权限的用户可以轻松地安装和实例化通道的链码，并查看是谁在他们参与的渠道中，恰当授权的用户可以根据所建立的区块链网络的策略调用链码，创建新的通道，甚至更新通道的访问权限。  
#### 共识
在整个网络中保持账本交易同步的过程 - 确保账本只有在交易得到适当的参与者批准时才会更新，而且当账本更新时，它们以相同的顺序更新相同的交易 - 被称为共识。  
将共识定义为三阶段流程：背书，排序和确认。
现在，将区块链视为一个共享的，复制的交易系统就足够了，该系统通过智能合约进行更新，并通过一个称为共识的协作过程保持一致的同步。  
交易必须按照发生的顺序写入账本。共识算法有很多。例如，PBFT（Practical Byzantine Fault Tolerance）可以提供文件副本相互通信的机制，以保持每个副本的一致性，即使在发生损坏的情况下也是如此。  
Hyperledger Fabric被设计为允许网络发起者（指通道建立者）选择最能代表参与者之间存在关系的共识机制。与隐私一样，还有一系列需求。从具有高度结构化的关系网络到更加对等的网络。  
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
    展⽰和查询区块链块、交易和相关数据的Web应⽤(node.js)    
    UI to interact with ledger，Under-development  
- Cello
    BaaS的⼯具集，帮助创建、管理、终⽌区块链(Blockchain as a Service)  

### Fabric
Hyperledger Fabric网络的成员是通过成员服务提供商（MSP）注册的，而不是一个允许未知身份参与网络的开放式无权限系统（需要证明工作证明的协议来验证交易和保护网络）。
Hyperledger Fabric还提供了创建**通道**的能力，允许一组参与者创建单独的交易账本。用通道来控制访问账本的权限。账本可以加入通道，参与者也可以加入通道。对于一个通道，里面的参与者可以“看到”里面的账本。  
#### 成员管理(Membership)
成员注册、⾝份保护、 内容保密、交易审计功能，以保证平台访问的安全性。  
在Hyperledger Fabric中，每个参与者（客户端，peer，oderer）都属于某个组织。  
组织拥有CA，为其成员（客户端，peer，oderer）提供证书，以便向其他人或其他组织认证自己。  
交易方持有多种类型的证书，交易不同环节将使用如下这些类型的证书：  
- 登记证书(Enrollment Cert)。用于验证使用者身份，永不过期
- 交易证书(Transaction Cert)。每个交易开始时生成，用于交易的签名  
- 通信证书(TLS-Cert)，用于通信连接加密，永不过期  

#### 区块服务（BlockChain&Transaction)
负责peer间的共识管理、账本的分布式计算、账本的存储以及peer间的P2P协议功能的实现，是区块链的核⼼组成部分，为区块链的主体功能提供了底层⽀撑。  
#### 链码(ChainCode)
链码即智能合约(smart contract)。该模块为链码提供安全的部署、运⾏的环境。  
#### Event
贯穿于其他各个组件中间，为各个组件间的异步通信提供了技术实现。  
### 账本
由两个组件组成：全局状态和交易日志。  
账本提供了一个所有状态变化的历史记录。它可以是一个业务的系统记录。每个Hyperledger Fabric网络的参与者都拥有账本的副本（网络指通道）。  
**区块账本(Block ledger)**  
 - 基于文件系统，只能添加新区块  
 - 区块保存在所有提交者peer，部分可选子集保存在排序服务节点  
**状态账本**  
 - 全局状态(账本)保存了智能合约数据的当前值  
   如vehicleOwner=Daisy  
 - 开发者通过链码API访问隐藏起来的KVS(key-value数据库)  
   如GetState(), PutState(), GetStateByRange()等等  
 - 保存在所有提交者peer  
**历史账本**  
 - 保存所有链码交易的顺序历史记录。索引保存在KVS，开发者通过链码API访问  
 - 保存在所有提交者peer  
 
## Fabric文档
[英文官方文档](https://hyperledger-fabric.readthedocs.io/en/latest/)   
[中文官方文档](https://hyperledgercn.github.io/hyperledgerDocs/)  

[自己的Fabric文档](https://wbwangk.github.io/hyperledgerDocs/)  

Composer项目在Fabric的基础上增加了REST API、web UI等能力。这是自己翻译的[Composer文档](https://wbwangk.github.io/ComposerDocs/)。

## Hyperledger项目介绍
#### 
[Quilt](https://github.com/hyperledger/quilt)Hyperledger Quilt - An implementation of the Interledger Protocol [https://interledger.org](https://interledger.org/)  
[一个Quilt的PPT](https://chicagopaymentssymposium.org/wp-content/uploads/2016/10/101216-visions-interoperability-thomas.pdf)   
Quit遵循了[ISO 20022](http://wiki.mbalib.com/wiki/ISO20022)。  
[stackoverflow上的相关回答](https://stackoverflow.com/questions/46203765/interledger-connector-for-hyperledger)   

### Hyperledger Burrow
[Hyperledger Burrow](https://github.com/hyperledger/burrow)是一个可授权的智能合约机器；在2017年4月成为Hyperledger旗下的第四个分布式账本平台。它最初是由Monax开发并提供给Hyperledger的。

Burrow提供了一个模块化区块链客户端，它带有一个根据以太坊虚拟机（EVM）规范构建的可授权的智能合约解释器。Burrow为Hyperledger的全面工作提供了一个非常决定性的、聚焦于智能合同的区块链设计。Burrow的用户可以通过使用智能合约和基于“原生安全”的权限层来获得一个访问控制层。

Burrow的主要组成如下：

- 共识引擎，负责维护节点之间的网络堆栈，和供应用引擎使用的交易排序服务。

- 应用区块链接口（“ABCI”）为共识引擎和应用引擎提供了连接的接口规范。

- 智能合约应用引擎为应用程序构建者提供强大的确定性智能合约引擎，用于运行复杂的工业流程。

- 网关为系统集成和用户界面提供编程接口。

### Hyperledger Indy
Hyperledger Indy是一个专门用于分布式账本去中心身份。它提供工具、库和可重复使用的组件，用于创建和使用以区块链或其他分布式账本为基础的独立数字身份，以便跨管理域、应用程序和任何其他“孤岛”进行互操作。

由于分布式账在事后不能进行修改，基于账本的身份用例需要仔细考虑基本组件，包括性能、规模、信任模型和隐私。特别是，Privacy by Design和隐私保护技术对于全球范围内可以进行关联的公共身份识别账户至关重要。

出于所有这些原因，Hyperledger Indy开发了去中心身份的规范、术语和设计模式，并为这些概念提供了一个实现，可供在Hyperledger联盟内部和外部使用。



## 备忘
[Hyperledger Composer](https://hyperledger.org/projects/composer)  
[Hyperledger Composer教程](https://developer.ibm.com/courses/all/ibm-blockchain-foundation-developer/?course=begin#11982)，含composer的安装。  
[另一个官方文档](http://fabrictestdocs.readthedocs.io/en/latest/glossary.html)  

在学习超级账本(Hyperlegder)过程中，还编写了另外三个相关wiki文档：

- [Hyperledger_Composer](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger_Composer)

- [Fabric](https://github.com/wbwangk/wbwangk.github.io/wiki/Fabric)

- [Fabric笔记](https://github.com/wbwangk/wbwangk.github.io/wiki/Fabric%E7%AC%94%E8%AE%B0)

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

