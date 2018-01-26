[中文文档](https://wbwangk.github.io/hyperledgerDocs)  

## Fabric快速开始
本章《Fabric快速开始》的主要内容是按[中文文档](https://wbwangk.github.io/hyperledgerDocs)的流程搭建区块链环境，完成BYFN(first-network)的过程。

### 开发环境搭建(vagrant)
(宿主机：/e/vagrant9/ambari-agrant/fabric/devenv)   
fabric官方库提供了一个Vagrantfile，是个ubuntu16的环境，供开发调试用。可参考[Fabri Getting Started](http://hyperledger-fabric.readthedocs.io/en/latest/getting_started.html)。   
  
在宿主机下启动fabric开发环境VM：
```
$ git clone https://github.com/hyperledger/fabric.git
$ cd fabric/devenv && vagrant up && vagrant ssh
```
虚拟机启动过程中会自动执行一个`setup.sh`脚本进行初始化。有时会半途执行失败。可以进入linux后手工执行脚本：
```
$ cd /hyperledger/devenv && ./setup.sh
```
这样，你就拥有了一个fabric开发环境。(snap a3)  

通过分析`setup.sh`，也可以看到手工安装docker-compose、golang的办法：
#### 安装docker-compose
```
$ curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```
#### 安装go lang
`setup.sh`中提供的是从`golang.org`下载安装包，下面描述的是从另一个网站下载安装包。
```
$ cd /opt && wget https://redirector.gvt1.com/edgedl/go/go1.9.2.linux-amd64.tar.gz
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
#### Hyperledger Fabric Samples
[Hyperledger官方GETTING STARTED](https://hyperledger-fabric.readthedocs.io/en/latest/samples.html)  
```
$ cd /opt && git clone -b master https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples && curl -sSL https://goo.gl/6wtTN5 | bash -s 1.1.0-preview
```
脚本会自动下载了很多docker镜像，如ca、order、peer、ccenv、tools、couchdb、kafka、zookeeper、javaenv等。

脚本还下载了几个工具到`/opt/fabric-samples/bin`目录下，它们是`cryptogen`、`configtxgen`、`configtxlator`、`orderer`、`peer`。  

### MSP配置(cryptogen)
cryptogen工具根据配置文件`crypto-config.yaml`中配置，生成整个区块链所有成员的密码材料文件。
```
$ cd /opt/fabric-samples/first-network
$ cryptogen generate --config=./crypto-config.yaml
```
执行后，会在当前目录下创建一个`crypto-config`文件夹。`crypto-config`文件夹的结构如下：
```
crypto-config
    ordererOrganizations
        example.com
            ca
            msp
            orderers
            tlsca
            users
    peerOrganizations
        org1.example.com
            ca
            msp
            orderers
            tlsca
            users
        org2.example.com
            (同上)
```
上述目录结构是根据`crypto-config.yaml`的内容生成的。`crypto-config.yaml`的内容如下：
```yaml
OrdererOrgs:
  - Name: Orderer
    Domain: example.com
    Specs:
      - Hostname: orderer
PeerOrgs:
  - Name: Org1
    Domain: org1.example.com
  - Name: Org2
    Domain: org2.example.com
```
上述`crypto-config.yaml`是fabric-sample带的示范配置文件，它将Orderer和Peer两种节点的MSP都定义到了同一个文件中。而在实际的生产环境下，Orderer节点和Peer节点一般部署在不同的虚拟机中，则两种节点的`crypto-config.yaml`配置文件将有所不同。  

要搭建一个Fabric网络，首先应建立Orderer节点。而Orderer节点存在着一个系统区块链，上面保存了整个Fabric网络的基础信息。要创建一个区块链，先用`cryptogen`工具生成MSP密码文件，再用`configtxgen`工具生成创世区块。更具体的，应先生成Orderer节点的创世区块。  

### 生成初始区块(configtxgen)
有时把初始区块翻译成创世区块。

`configtxgen`运行时需要查找配置文件`configtx.yaml`，这个配置文件的位置是靠环境变量来指定的：
```
$ export FABRIC_CFG_PATH=/opt/fabric-samples/first-network
$ configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
```
执行后生成了一个“创世区块”文件`genesis.block`。如果用vi打开这个文件看看，发现里面有9个证书，应该是3个属于orderer，6个属于peer。  
`TwoOrgsOrdererGenesis`来自哪里？这就需要开发配置文件`configtx.yaml`来看看了：
```yaml
Profiles:
    TwoOrgsOrdererGenesis:
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
                    - *Org2
    TwoOrgsChannel:
        Consortium: SampleConsortium
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2
Organizations:
    - &OrdererOrg
        Name: OrdererOrg
        ID: OrdererMSP
        MSPDir: crypto-config/ordererOrganizations/example.com/msp

    - &Org1
        Name: Org1MSP
        ID: Org1MSP
        MSPDir: crypto-config/peerOrganizations/org1.example.com/msp

        AnchorPeers:
            - Host: peer0.org1.example.com
              Port: 7051

    - &Org2
        Name: Org2MSP
        ID: Org2MSP
        MSPDir: crypto-config/peerOrganizations/org2.example.com/msp

        AnchorPeers:
            - Host: peer0.org2.example.com
              Port: 7051
(略)
```
除了`profile`的定义`TwoOrgsOrdererGenesis`外，配置文件中另一个与`configtxgen`的执行有关的属性是一个目录：
```
MSPDir: crypto-config/ordererOrganizations/example.com/msp
```
这个目录中密码文件由`cryptogen`生成。如果之前没有用`cryptogen`生成这些文件，`configtxgen`执行时会提示这些上述目录不存在。

#### 用configtxgen查看区块信息
```
$ export FABRIC_CFG_PATH=/opt/bcnet
$ configtxgen -profile TwoOrgsOrdererGenesis -inspectBlock ./channel-artifacts/genesis.block
```
`configtxgen`会到FABRIC_CFG_PATH环境变量定义的目录下找`configtx.yaml`这个配置文件，并按配置文件中的定义先找`profile`，根据`profile`的值所对应的`MSPDir`下寻找密钥文件，然后利用密钥文件对创世区块进行解密并转换为json格式输出。

有的创世区块可以不加`-profile`参数就能正常显示，原因不明。  

#### 生成通道(configtxgen)
```
$ export FABRIC_CFG_PATH=/opt/fabric-samples/first-network
$ configtxgen -profile TwoOrgChannel -outputCreateChannelTx ./channel-artifacts/channel1.tx -channelID channel1
```

### peer加入通道(peer)
peer直接在宿主机上直接运行会报错，一般进入`cli`容器内执行:
```
$ docker exec -it cli bash
$$ peer --help
```

### 参考：共识过程(视频截图)
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
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/glossary.html)  

#### 锚点Peer
它是通道中的一个peer节点，其他peer可以发现它并与它通信。通道中的每个[成员](#成员)都一个锚点peer(或者多个锚点防范单点失效)，用于通道中属于不同成员的peer可以发现全部peer。  

#### 区块
一组有序的事务，通过哈希链接到通道中的前一个区块。

#### 链
账本的链式一个事务日志，组织成多个哈希值链接的区块。peer从排序服务接收事务区块，根据背书策略和并发违规将区块中的事务标记为生效和失效，而且在peer的文件系统中将区块追加到哈希链。  

#### 链码
链码是运行在一个账本上的软件，编码了资产和事务指令（业务逻辑）来修改资产。  

#### 通道
通道是一个私有区块链，它允许数据隔离和保密。通道的账本被通道中所有peer共享，与通道交互的事务参与方必须被通道正确地认证。通道被[配置区块](#配置区块)定义。  

#### 提交(Commitemnt)
通道中的每个[Peer](#Peer)生效事务的有序区块，然后提交(写/追加)区块到它的通道[账本](#账本)副本。Peer会标记每个区块的每个事务为生效或失效。  

#### 并发控制版本检查
这是一个保持通道内多个peer之间状态同步的方法。peer并行执行事务，在提交到账本前，peer用读检测数据在执行的时候没有被改变。如果事务的读数据在执行时和提交时之间被改变了，则发生了并发控制版本检查违规，事务在账本中被标记为失效，不会改变状态数据库的数据。  

#### 配置区块
为系统链(排序服务)或通道容纳成员定义和策略配置数据。任何对一个通道或全部网络(如成员离开或加入)的任何配置修改都导致一个新的配置区块追加到相应链中。配置区块包含创世区块，再加上delta。

#### 共识
贯穿整个事务流程的一个更广泛的术语，用来按顺序生成一致性并确认按一组事务构组成区块的正确性。  

#### 当前状态
账本的当前状态表示包含在它的链式事务日志中的所有密钥的最新值。针对包含在已处理区块中的每个有效事务，peer将最新值提交到账本的当前状态。由于当前状态代表通道已知的所有最新键值，因此有时称为世界状态。链码针对当前状态数据执行事务提议。  

#### 动态成员资格
Hyperledger Fabric在不影响这个网络的可靠性的前提下，支持增减会员、peer和排序服务节点。当业务关系调整和引各种原因需要增减实体时，动态成员资格是很需要的。   

#### 背书
指一个过程：特定peer节点执行一个链码事务，返回提议响应到客户端应用。提议响应包含链码执行相应信息、结果(read set和write set)、事件和一个签名（证明peer执行了链码）。链码应用程序遵照背书策略，其中规定了背书peer。  

#### 背书策略
定义了通道中的哪些peer节点必须执行附加在链码应用中的事务，和需要的响应(背书)组合。需要这样一个策略，定义一个事务至少得到几个peer的同意，或最少多少比例的peer同意，或全部peer的同意，这个策略会附加到一个特定链码应用上。策略可以根据应用程序和期望水平来反对不当行为（有意或无意）。一个事务在被提交peer标记为生效前，必须符合背书策略。对部署、实例化事务也需要一个明确的背书政策。  

#### Hyperledger Fabric CA
Hyperledger Fabric CA是向网络成员组织和它们的用户发放PKI证书的默认CA组件。CA向每个成员发放一个根证书(rootCert)，象每个授权用户发放一个登记证书(ECert)。  

#### 创世区块
初始化区块链网络或通道的配置区块，也作为链上的第一个区块。  

#### Gossip协议
gossip数据广播协议完成三个功能：1）管理peer发现和通道成员资格；2）在通道中所有peer之间广播账本数据；3）在通道中所有peer之间同步账本状态。更多细节参考[Gossip](https://hyperledger-fabric.readthedocs.io/en/latest/gossip.html)主题。  

#### 初始化
一个初始化链码应用的方法。  

#### 安装
将链码放置到peer的文件系统的过程。

#### 调用(invoke)
用于调用链码函数。一个客户端应用通过发送事务提议到一个peer来调用链码。peer会执行链码和返回一个背书提议响应到客户端应用。客户端应用收集足够的提议响应以便满足背书策略，然后递交事务结果以便排序、生效和提交。客户端应用程序也可以选择不去递交事务结果。例如查询账本，客户端应用程序通常不会递交这个只读事务，除非是为了审计的目的想把读账本的行为记录日志。调用包括通道id、调用的链码函数和参数数组。  

#### 领导peer
每个[成员](#成员)在它订阅的每个通道中都可以有多个peer。其中一个peer可以作为该通道的领导peer，代表成员与网络排序服务进行通信。排序服务“递送”区块到一个通道的领导peer(可能多个)，它(或它们)再分发区块到成员集群的其他peer。  

#### 账本
账本是一个通道链和被通道中所有peer共同维护的当前状态数据。  

#### 成员
一个在网络中拥有特定根证书的独立法律实体。象peer和应用客户端这样的网络组件会被关联到成员。  

#### 成员服务提供商
成员服务提供商（MSP）是指为客户端提供（匿名）凭证的系统的抽象组件，以及参与Hyperledger/fabric网络的peer。客户端使用这些凭证对其事务进行身份验证，peer使用这些凭据来验证事务处理结果（背书）。在与系统的事务处理组件紧密连接的同时，这个接口的目标是定义成员服务组件，这样可以顺利地插入替代的实现，而无需修改系统的事务处理组件的核心。  

#### 成员服务
成员服务在授权区块链网络上认证、授权和管理身份。在peer和orderer节点上运行的成员服务代码认证和授权区块链操作。它是个成员服务提供商抽象的一个基于PKI的实现。

#### 排序服务
一个将事务排序进入区块的节点集合。排序独立于peer过程而存在，并以先来先服务的原则为网络上的所有通道进行事务排序。
一个集中或非几种的服务，对区块中事务进行排序。您可以选择“排序”功能的不同实现方式 - 例如：简化和测试的“solo”，用于碰撞容错的Kafka，或用于拜占庭容错的sBFT/PBFT。您也可以开发自己的协议来插入服务。排序服务在开箱即用的SOLO和Kafa实现的之外支持插件式实现。排序服务一般绑定到整个网络；它存放了每个[成员](#成员)的密钥身份材料。    

#### Peer
peer是一个网络实体，负责维护账本和为了读写账本而运行链码容器。peer被成员拥有和维护。  

#### 策略
是背书、生效、链码管理和网络/通道管理的策略。  

#### 提议
针对通道中的某个peer发出的背书请求。提议要么是个实例化请求，要么是个调用（读或写）请求。

#### 查询
查询是一个链码调用，它读账本当前状态但不会写账本。链码函数可以对账本按key查询，或按一组key查询。由于查询不会修改账本状态，客户端应用程序通常不会为了排序、验证和提交而递交只读事务。偶尔，客户端应用选择为了排序、验证和提交而递交只读事务，例如如果客户端需要账本在某个时间点的当前状态的审计证明。  

#### SDK
Hyperledger Fabric客户端SDK是为了方便开发者编写和测试链码应用而提供的一个结构化库环境。通过标准接口，SDK是完全可配置和可扩展的。包括签名算法、日志框架和状态存储，组件可以轻松进出SDK。SDK为事务处理、成员服务、节点遍历和事件处理提供了API。提供了多种风格的SDK：Node.js、Java和Python。

#### 状态数据库
当前状态数据被保存一个状态数据库中，以便链码可以方便地读和查询。支持的数据库包括levelDB和couchDB。

#### 系统链
包含一个在系统级别定义网络的配置区块。系统链存在于排序服务中，与通道类似，具有包含以下信息的初始配置：参与者组织的根证书和排序服务节点、策略、OSN(排序服务节点)监听地址以及配置详细信息。对整个网络的任何改变（例如一个新的组织加入或新增一个排序节点）将导致新的配置区块被添加到系统链中。  
系统链可以被认为是通道或通道组的通用绑定。例如，一系列金融机构可以组成一个联盟（通过系统链表示），然后再根据联盟或业务议程创建通道。  

#### 事务
为了排序、验证和提交而被递交的调用或实例化结果。调用是一个为了从账本读或写数据的请求。实例化是一个为了在通道中启动和实例化链码的请求。应用客户端收集从背书peer返回的调用或实例化响应，包装结果集和背书为一个事务，事务为了排序、验证和提交而被递交。

#### 排序者(Orderer)
构成排序服务的网络实体之一。一个排序服务节点（OSN）的集合，用来将事务排序进区块。在“独奏”的情况下，只需要一个OSN。事务被“广播”给排序者，然后作为区块“交付”到适当的通道。  

#### 背书者(Endorser)
一个特定的peer角色，背书peer负责模拟事务，并且防止不稳定或不确定的事务通过网络。事务以事务提议的形式发送给背书者。所有的背书peer同时也是committer peer（即他们写账本）。  

#### 提交者(Committer)  
一个特定的peer角色，负责将有效事务写入某通道的账本。一个peer可以同时充当背书者和提交者，当很多时候只是充当提交者。  

## Hyperledger Fabric FAQ
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/Fabric-FAQ.html)
### 背书
问：在网络中多少个peer需要对一个事务进行背书？  
答：需要对一个事务进行背书的peer数量由链码部署时指定的背书策略来确定。 

问：一个应用客户端需要连接到所有peer吗？  
答：客户端需要连接到的peer数量，看链码的背书策略需要的peer数量。  

### 安装和访问控制
#### 数据隐私和访问控制
问：我怎么确保数据隐私？  
答：数据隐私有多个方面。首先，你可以用通道隔离你的网络，这里每个通道代表了参与者一个子集，允许看到数据的链码会被部署到这个通道。第二，在通道中，你可以通过可见性限制链码只能从一组背书节点获取数据。可见性设置会确定输入和输出链码数据是否包含在递交的事务中，而不仅仅是输出数据。第三，在调用链码前，你可以哈希或加密数据。如果你哈希数据，那么你需要提供一种方法来共享源数据。如果你加密数据，那么你需要提供共享解密密钥的方法。第四，通过将访问控制逻辑编码进链码，你可以限制数据访问你组织中的某些角色。第五，休息时的账本数据可以通过对peer文件系统的加密进行加密，在途数据通过TLS进行加密。 

问：oderer能否看到事务数据？  
答：不，orderer仅对事务排序，它们不会打开事务。如果你根本不希望数据流过orderer，而只关心输入数据，则可以使用可见性设置。可见性设置将确定输入和输出链码数据是否包含在递交的事务中，而不仅仅是输出数据。因此，输入数据只能被背书节点看到。如果你不想让orderer看链码输出，那么你可以在调用链码之前哈希或者加密数据。如果你哈希数据，那么你将需要提供一方法来共享源数据。如果您加密数据，那么您将需要提供共享解密密钥的方法。

### 应用端编程模型
#### 事务执行结果
问：应用客户端怎么知道一个事务的结果？  
答：背书者通过提议响应将事务模拟结果返回给客户端。如果存在多个背书者，客户端可以检查响应是否完全一致，并递交结果和背书进行排序和提交。最终，提交peer会记录事务生效或失效，并且客户端通过事件知道结果，即SDK使应用客户端可用。

#### 账本查询
问：我怎么查询账本数据？  
答：在链码中你可以通过key查询。key可以按范围查询，组合key查询等价于多参数查询。例如，一个组合key(拥有者、资产id)可以用于查询一个实体拥有的所有资产。这些基于key的查询可以用于对账本的只读查询，也可用于通过事务更新账本。

如果在链码中用JSON格式对资产数据建模，并使用CoutchDB作为状态数据，你就能对链码数据值进行复杂富文本查询（在链码使用CouchDB JSON查询语言）。应用客户端可以执行只读查询，但这些响应一般不会当作事务的一部分发往排序服务。

问：我怎么通过查询历史数据得知数据来源？  
答：链码API`GetHistoryForKey()`将返回一个key值的历史记录。

问：如何保证查询结果是正确的，特别是当被查询的peer可能正在恢复和赶上块处理？  
答：客户端可以查询多个peer，比较它们的区块高度，比较它们的查询结果，并倾向于更高的区块。

### 链码(智能合约和数字资产) 
问：Hyperledger Fabric是否支持智能合约逻辑？  
答：是的。我们叫这个功能为[链码](#链码)。这是我们对智能合约方法/算法的解释，附加功能。

链码是部署在网络上的程序代码，它在共识过程中被链确认者执行和确认。开发者可以利用链码开发业务合约、资源定义和共同管理的无中心化应用。

问：怎么创建业务合约？  
答；通常有两种方法开发业务合约：第一个方法是将单个合同编码进链码的独立实例；第二种方法，也是更有效的方法，使用链码创建无中心应用来管理一种或多种类型业务合约的生命周期，让最终用户在他们的应用中实例化合约实例。

问：怎样创建资产？  
答：用户可以使用链码(为了业务规则)和成员服务(为了数字令牌)来设计资产，以及管理它们的逻辑。

有大多区块链解决方案中有两种流行的方法定义资产：无状态UTXO模型，账户余额被编码进过去的事务记录中；和账户模型，账户余额保存在账本的状态存储空间中。

每种方法都有它自己的优势和缺点。区块链技术不会倾向于某一种。相反，我们的首要要求之一是确保两种方法都可以轻松实施。

问：编写链码支持哪些语言？  
答：链码可以用任何编程语言编写并在容器中执行。第一个完全支持的链式代码语言是Golang。

已经讨论了对其他语言的支持和模板语言的开发，更多的细节将在不久的将来发布。

也可以使用[Hyperledger Composer](https://hyperledger.github.io/composer/)构建Hyperledger Fabric应用程序。

问：Hyperledger Fabric是否有原生货币？  
答：没有。但是，如果您的区块链网络确实需要原生货币，则可以使用链码开发自己的原生货币。原生货币的一个常见属性是，每次在链上处理交易时，都会有一定数量的交易（定义货币将被调用的链码）。

### 最近发布的差异
问：作为v1.0.0版本的一部分，v0.6和v1.0之间有什么突出的区别？  
答：答：任何后续版本之间的差异与[发布说明](http://hyperledger-fabric.readthedocs.io/en/latest/releases.html)一起提供。由于Fabric是一个可插拔的模块化框架，您可以参考[设计文档](https://wiki.hyperleger.org/projects/fabric/design-docs)了解这些差异的更多信息。

问：哪里可以得到上面没有回答的技术问题的帮助？  
答：请使用[StackOverflow](https://stackoverflow.com/questions/tagged/hyperledger)。

## Fabric CA User’s Guide
[官方原文](http://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#table-of-contents)  
它提供的功能：
- 注册身份(identity)，或在用户注册时连接到LDAP  
- 发放注册证书(ECert)  
- 证书更新或撤销  

Hyperledger Fabric CA由一个服务器和一个客户端组件构成。  
下图展示了CA与Hyperledger Fabric整体架构的关系。  
![](http://hyperledger-fabric-ca.readthedocs.io/en/latest/_images/fabric-ca.png)  

有两种方式与Hyperledger Fabric CA交互：通过Hyperledger Fabric CA客户端或通过Fabric SDK。所有与Hyperledger Fabric CA服务器的通信是通过REST API。`fabric-ca/swagger/swagger-fabric-ca.json`是swagger格式的REST API文档，可以通过在线工具[http://editor2.swagger.io](http://editor2.swagger.io/)查看。  
可以搭建Hyperledger Fabric CA集群，在集群下，所有服务器共享同一个数据库，数据库中存有身份和证书。如果配置了LDAP，身份(identity)信息就保存在LDAP中，而不是数据库。  
一个服务器可以包含多个CA。每个CA要么是根CA，要么是中间CA。每个中间CA有一个父CA，父CA要么是根CA要么是另一个中间CA。  

[编写首个应用](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E7%BC%96%E5%86%99%E9%A6%96%E4%B8%AA%E5%BA%94%E7%94%A8)中通过命令启动了几个容器：
```
$ cd /opt/fabric-samples/fabcar
$ ./startFabric.sh
```
其中的`ca.example.com`就是一个CA容器，可以进入容器去看看($$表示在容器内的命令)：
```
$ docker exec -it ca.example.com bash
$$ whereis fabric-ca-server
fabric-ca-server: /usr/local/bin/fabric-ca-server
$$ ps -ef | grep fabric-ca-server
root         7     1  0 00:42 ?        00:00:00 fabric-ca-server start -b admin:adminpw -d
```
用`ps`命令查看CA的守护进程，可以看到它的启动命令是：
```
fabric-ca-server start -b admin:adminpw -d
```
`-b`参数指出了注册(enrollement)ID和密码。  
`fabric-ca-server-config.yaml`是Fabric CA的默认配置文件，可以通过find命令查找它的位置：  
```
$$ find / -name fabric-ca-server-config.yaml
/etc/hyperledger/fabric-ca-server/fabric-ca-server-config.yaml
```
#### CA相关环境变量
```
$$ env
FABRIC_CA_SERVER_CA_NAME=ca.example.com
FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/4239aa0dcd76daeeb8ba0cda701851d14504d31aad1b2ddddbac6a57365e497c_sk
FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem
FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
```
### Fabric CA CLI
命令帮助信息：[Server Command Line](http://hyperledger-fabric-ca.readthedocs.io/en/latest/servercli.html)、[Client Command Line](http://hyperledger-fabric-ca.readthedocs.io/en/latest/clientcli.html)  



## 备忘
### configtxgen
#### 生成orderer创世区块
```
$ export FABRIC_CFG_PATH=$PWD
$ configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
```
#### 创建通道
```
$ export CHANNEL_NAME=mychannel  
$ configtxgen -profile TwoOrgsChannel \
 -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
```
#### 创建锚点peer
```
$ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate \
./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
```
#### 获取组织配置材料
现在使用configtxgen工具输出JSON格式的Org3相关配置材料。作为开始的命令，告诉工具从当前目录下读取configtx.yaml。
```
$ export FABRIC_CFG_PATH=$PWD && ../../bin/configtxgen -printOrg Org3MSP > ../channel-artifacts/org3.json
```

### peer命令
#### 创建通道配置交易
```
$ export CHANNEL_NAME=mychannel
$ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
$ peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f \
./channel-artifacts/channel.tx --tls --cafile $ORDERER_CA
```
#### peer加入通道
```
$ CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
$ CORE_PEER_ADDRESS=peer0.org2.example.com:7051
$ CORE_PEER_LOCALMSPID="Org2MSP"
$ CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
$ peer channel join -b mychannel.block
```
#### 创建链码包
```
$ peer chaincode package -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -v 0 -s -S -i "AND('OrgA.admin')" ccpack.out
```
#### 对链码包签名
```
$ peer chaincode signpackage ccpack.out signedccpack.out
```
#### 安装链码
```
$ peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
```
#### 实例化链码
```
$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
```
#### 用链码查询
```
$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```
用系统链码查询：
```
An example GetBlockByNumber query with block number 1 on channel mychannel using QSCC is given below:
$ peer chaincode query -C "" -n qscc -c '{"Args":["GetBlockByNumber","mychannel","1"]}'
```
根据区块高度在通道mychannel上查询系统链码QSCC。返回是的该区块的内容[1](https://blockchain-fabric.blogspot.jp/2017/09/three-component-maintained-by-ledger-in.html)。
#### 用链码调用
```
$ peer chaincode invoke -o orderer.example.com:7050  --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'
```
#### 链码升级
```
$ peer chaincode upgrade -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -v 2.0 -c '{"Args":["init","a","90","b","210"]}' -P "OR ('Org1MSP.member','Org2MSP.member','Org3MSP.member')"
```
#### 取配置区块
```
$ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
$ export CHANNEL_NAME=mychannel
$ peer channel fetch config config_block.pb -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
$ peer channel fetch 0 mychannel.block -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
```
`0`表示获取创世区块。无参数表示获取最新区块。
#### 对交易签名
需要提前配置好4个环境变量，以便知道以哪个Admin的身份签名。
```
$ peer channel signconfigtx -f org3_update_in_envelope.pb
```
#### 更新呼叫
管理员签名会附加到这个呼叫上，所以不需要手工再次签署这个proto。
```
$ peer channel update -f org3_update_in_envelope.pb -c $CHANNEL_NAME -o orderer.example.com:7050 --tls --cafile $ORDERER_CA
```
#### 取最新区块到文件
```
$ peer channel fetch newest 1.block -o localhost:7050 -c composerchannel
```
