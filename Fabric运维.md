[Hyperledger](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger)

## 成员服务提供者(MSP)
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/msp.html)  
本文提供了有关MSP的设置和最佳实践的详细信息。

成员服务提供商（MSP）是一个旨在提供成员维护体系结构抽象的组件。

特别是，MSP将发布和验证证书背后的所有密码学机制和协议以及用户认证抽象出来。MSP可以定义他们自己的身份概念，以及这些身份管理（身份验证）和认证（签名生成和验证）的规则。

Hyperledger Fabric区块链网络可以由一个或多个MSP管理。这提供了成员资格维护的模块化，以及跨不同成员标准和体系结构的互操作性。

在本文的其余部分中，我们将详细介绍由Hyperledger Fabric支持的MSP实现的设置，并讨论关于其使用的最佳实践。

### MSP配置
要创建一个MSP实例，需要在每个peer和orderer的本地指定其配置。

对于每个MSP，为了在网络中引用它，首先需要确定一个名称（如msp1，org2和org3.divA）。这是一个名称，通过名称一个MSP的成员规则可以表现为一个联盟、组织或组织单元(OU)在通道中被引用。这还被描述为MSP身份或MSP ID。MSP身份在每个MSP实例中必须唯一的。例如，如果两个MSP的实例具有相同的身份在系统通道创世区块中被检测到，oderer的建立会失败。

MSP的默认实现中，需要指定一组用于身份(证书)验证和签名验证的参数。这些参数在[RFC5280](http://www.ietf.org/rfc/rfc5280.txt)中描述。包括：
- 一个自签名(X.509)证书列表，构成了*信任根(root of trust)*  
- 一个表示中间CA的X.509证书列表；这些这证书应当被*信任根*的某个证书所证明(签署)；中间CA是可选参数  
- 一个X.509证书列表，证书验证路径应正确地指向某个信任根证书，它代表MSP的管理员；这些证书的拥有者被授权可以请求变更这个MSP的配置（如根CA、中间CA）  
- 一个组织单元(OU)列表，它们是应该被包含在X.509证书中这个MSP的有效成员；这是一个可选配置参数，例如可用于，多个组织利用同一个信任根或中间CA，它们为成员保留了一个OU字段。  
- 一个撤销证书(CRL)列表，每个对应了一个MSP CA(根或中间证书)；这是一个可选参数   
- 一个自签名证书(X.509)列表，组成TLS证书的信任根  
- 一个代表中间TLS CA的X.509证书列表；这些证书应当被TLS信任根的某个证书证明；中间CA是一个可选参数。    

对于这个MSP实例的有效身份需要满足以下条件：
- 它们是X.509格式证书，带来一个可验证证书路径到某个信任根证书；  
- 它们没有包含在任何CRL中；  
- 它们的X.509证书结构的`OU`字段被定义在MSP配置文件的组织单元中.  

更多对于当前MSP实现的身份验证信息，请参阅[MSP身份有效性规则](http://hyperledger-fabric.readthedocs.io/en/latest/msp-identity-validity-rules.html)。  

除了与验证相关的参数之外，为了使MSP能够将其实例化的节点签名或认证，需要指定：
- 用于节点签名的签名key(当前仅支持ECDSA key)  
- 节点的X.509证书，这在MSP的验证参数中是一个有效身份  

需要注意的重要一点是，MSP身份永不过期；你只能通过将身份加入CRL来取消它。另外，当前不支持TLS证书的强制撤回。  

### 怎样生成MSP证书和密钥?
生成X.509证书以提供给MSP配置，应用可以使用[Openssl](https://www.openssl.org/)。我们需要强调的是Hyperledger Fabric不支持RSA密钥证书。  
另一个选择是使用`cryptogen`工具，它的操作已经在[快速开始](http://hyperledger-fabric.readthedocs.io/en/latest/getting_started.html)中解释过。  
[Hyperledger Fabric CA](http://hyperledger-fabric-ca.readthedocs.io/en/latest/)也可以生成配置MSP需要的密钥和证书。  

### 在peer&oderer端建立MSP
为peer或oderer建立一个本地MSP，管理员需要创建一个文件夹(如`$MY_PATH/mspconfig`)，下面包含6个子文件夹和一个文件：  
1. 一个`admincerts`文件夹，包含几个PEM文件，每个文件对应一个管理员证书  
2. 一个`cacerts`文件夹，包含几个PEM文件，每个文件对应到一个根CA证书
3. (可选)一个`intermediatecerts`文件夹，包含几个PEM文件，每个文件对应一个中间CA证书  
4. (可选)一个`config.yaml`文件，包含所考虑的OU的信息；OU信息被定义在一个叫`OrganizationalUnitIdentifiers`的yaml数组条目下的键值对(`<Certificate, OrganizationalUnitIdentifier>`)，这里`Certificate`是CA证书的相对路径(根或中间)，用来证明这个组织单元成员(如`. ./cacerts/cacert.pem`)，而`OrganizationalUnitIdentifier`表示出现在X.509证书中的OU字段(例如"COP")  
5. (可选)一个`crls`文件夹，包含了撤销证书列表(CRL)  
6. 一个`keystore`文件夹，包含一个PEM文件，是节点的签名密钥；再次强调，不支持RSA密钥  
7. 一个`signcerts`文件夹，包含一个PEM文件，是节点的X.509证书  
8. (可选)一个`tlscacerts`文件夹，包含几个PEM文件，每个对应到一个TLS根CA证书  
9. (可选)一个`tlsintermediatecerts`文件夹，包含几个PEM文件，每个对应一个中间TLS CA证书   

在节点的配置文件中(对peer是core.yaml，对orderer是orderer.yaml)，需要指定到mspconfig文件夹的路径，和节点MSP的身份(Id)。到msconfig的路径是相对于`FABRIC_CFG_PATH`，对于peer由参数`mspConfigPath`的值定义，对于orderer由参数`LocalMSPDir`值定义。节点MSP的身份，对于peer由参数`localMspId`的值定义，对于orderer由参数`LocalMSPID`的值定义。  
这些变量可以被环境变量覆盖，对于peer节点由前缀为CORE的环境变量覆盖(如CORE_PEER_LOCALMSPID)，对于orderer节点由前缀为ORDERER的环境变量覆盖(如ORDERER_GENERAL_LOCALMSPID)。  
注意，对于建立orderer，需要生成和提供系统通道的创世区块(genesis block)到orderer节点。MSP配置对这个区块的需求在下一节讲到。  
重新配置一个“本地”MSP只能手工进行，需要peer和orderer进程重启。在后续版本中，我们的目标是提供在线/动态重新配置（即使用一个用节点管理系统链码来避免停止节点）。

### 通道MSP建立
在创建系统时，需要指定出现在网络中的所有MSP的验证参数，并且包括在系统通道的创世区块中。回想一下前文讲到的组成MSP身份的MSP验证参数，信任证书的根、中间CA和管理员证书，还有OU规范和CRL。系统创世区块被提供给orderer（在orderer的创建阶段），允许它们认证通道创建请求。如果区块包括两个相同身份的MSP，oderer会拒绝，使网络自举失败。  
对于应用通道，通道的创世区块包含了管理通道的MSP验证组件。我们强调这是**应用的责任**：确保在指示其一个或多个peer加入通道之前，通道的创世区块(或最新的配置区块)中包含了正确的MSP配置信息。  
在使用`configtxgen`工具启动一个通道时，需要配置通道MSP，办法是将MSP的验证参数包含在`mspconfig`文件夹，有在`configtx.yaml`的相应章节设置文件夹路径。  
重新配置通道的MSP，包括MSP的CA更新CRL公告，通过MSP的管理员证书之一的所有者创建`config_update`对象来实现。管理员管理的客户端应用会将这个更新广播到MSP出现的通道中。  

### 最佳实践
在本节中，我们将详细介绍在常见情况下MSP配置的最佳实践。  

#### 1)组织/公司和MSP之间的映射

我们建议组织和MSP之间有一对一的映射。如果选择不同的映射类型的映射，则需要考虑以下内容：

- **一个组织采用多个MSP**。这对应于一个组织包含多个部门的情况，每个部门对应自己的MSP，无论出于管理独立性原因或出于隐私原因。在这种情况下，一个peer只能由一个MSP拥有，并且不会将来自其他MSP的身份的peer识别为同一组织的peer。其含义是，peer可以与属于同一个分支的peer分享数据，而不是与组织的全部peer分享数据。  
- **多个组织使用单个MSP**。这对应于具有相似成员管理架构的多组织联盟的情况。这里需要知道的是，peer将组织范围的消息传播给具有同一MSP下的peer，而不管他们是否属于相同的实际组织。这是MSP定义和/或peer配置粒度的限制。

#### 2)一个组织有不同的部门（又叫组织单元）， 希望授予他们对不同通道的访问权限。
有两种方式处理这个情况：

- **定义一个MSP来容纳所有组织成员的成员资格**。该MSP的配置将包括根CA，中间CA和管理员证书的列表; 成员身份id将包括成员所属的组织单位(`OU`)。然后可以定义策略来捕获具体的成员OU，这些策略可以构成通道的读/写策略或链码的背书策略。这种方法的一个局限性是，gossip peer会把本地MSP下的成员身份同行视为同一组织的成员，并因此会与它们频繁gossip组织范围的数据（例如他们的状态）。  
- **定义一个MSP来代表每个部门**。这将导致为每个部门指定根CA，中间CA和管理员证书的一组证书，从而不存在跨MSP的证书路径。这意味着，例如，每个部门都需要建立独立的中间CA。这里的缺点是管理多个MSP而不是一个，但是这样规避了以前方法中存在的问题。也可以通过利用MSP配置的OU扩展来为每个部门定义一个MSP。
#### 3)将客户端与同一组织的peer分开。

在许多情况下，要求身份的“类型”可以从身份本身中获取（例如，可能需要背书保证由peer获得，而不是客户端或者仅作为orderer的节点）。

对这些需求的支持有限。

一种分开的方法是为每种节点类型创建单独的中间CA--一个用于客户端，另一个用于peer/orderer; 并配置两个不同的MSP - 一个用于客户端，另一个用于peer/orderer。这个组织应该访问的通道将需要包括两个MSP，而背书策略将只使用指向peer的MSP。这最终会导致组织被映射到两个MSP实例，并且会对peer和客户端的交互方式产生一定的影响。

由于同一组织的所有peer仍然属于一个MSP，所以gossip不会受到严重的影响。peer可以将某些系统链码的执行限制在本地基于MSP的策略中。例如，如果请求由本地MSP（只能是客户端）的管理员签名（最终用户应该位于请求的起源处），那么peer只会执行“joinChannel”请求。如果我们接受作为peer/ordererMSP成员的唯一客户将是该MSP的管理员，我们可以绕过这种不一致。

这种方法要考虑的另一点是，peer根据其本地MSP中请求发起者的成员资格来授权事件注册请求。显然，由于请求的发起者是一个客户端，所以请求始发者总是肯定与所请求的peer属于不同的MSP，并且peer将拒绝该请求。

#### 4)管理员和CA证书。

将MSP管理员证书设置为与MSP根信任或中间CA的任何证书不同是非常重要的。将成员身份组件的管理职责与颁发新证书和(或)验证现有证书的职责分开，是常见（安全）做法。

#### 5)将中间CA列入黑名单。

如前所述，通过重新配置机制来实现MSP的重新配置（对本地MSP实例的手动重新配置，及通过对通道的MSP实例构建适当的`config_update`消息）。显然，有两种方法可以确保在MSP中考虑的中间CA不再用于MSP的身份验证：

1. 将MSP重新配置，在可信中间CA证书列表中的删除某些中间证书。对于本地配置的MSP，这意味着该CA的证书将从该`intermediatecerts`文件夹中删除。  
2. 重新配置MSP，包含一个由信任根产生的CRL，该CRL包含提到的中间CA的证书。  

在当前的MSP实现中，我们只支持方法(1)，因为它更简单，不需要将不再考虑的中间CA列入黑名单。

#### 6)CA和TLS CA

MSP身份的根CA和MSP TLS证书的根CA（和相关的中间CA）需要在不同的文件夹中声明。这是为了避免不同类别证书之间的混淆。不禁止为MSP身份和TLS证书重复使用相同的CA，但是最佳实践建议在生产中避免这种情况。 

## 通道配置(configtx)
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/configtx.html)  
Hyperledger Fabric区块链网络的共享配置被保存在一个配置事务集合中，每个通道一个。每个配置事务通常叫做*configtx*。  
通道配置有如下重要属性：
1. 版本(**Versioned**)：配置的所有元素都有一个关联的版本，每次修改都会更新。此外，每次更新配置都会收到一个顺序号。  
2. 权限(**Permissioned**)：配置的每个元素有一个关联的策略，用于管理是否允许对该元素进行修改。任何拥有以前configtx副本（并且没有其他信息）的人都可以根据这些策略验证新配置的有效性。
3. 层次(**Hierarchical**)：根配置组包含子组，每个层次组都有关联的值和策略。这些政策可以利用层次结构从下层策略中推导出一个层次的策略。
### 解剖一个配置
配置被保存在区块的一个类型为`HeaderType_CONFIG`的事务中，该区块中没有其他事务。这些区块被称为**配置区块**，第一个区块被称为**创世区块**(Genesis Block)。  
配置的proto结构存储在`fabric/protos/common/configtx.proto`。封装类型`HeaderType_CONFIG`编码了一个`ConfigEnvelope`消息，作为`Payload``data`字段。`ConfigEnvelope`的proto定义如下：
```golang
message ConfigEnvelope {
    Config config = 1;
    Envelope last_update = 2;
}
```
(上面的`1``2`是go语言结构的属性tag)  
`last_update`字段定义在下面的**Updates to configuration**部分，但仅在验证配置时需要，不会读它。相反，当前提交的配置被存储在`config`字段，其中包含了一个`Config`消息。
```golang
message Config {
    uint64 sequence = 1;
    ConfigGroup channel_group = 2;
}
```
`sequence`数字在每次配置提交后会增长。`channel_group`字段是包含配置的根组(root group)。。`ConfigGroup`结构是递归定义的，构建了一个组数，它可以包含值和策略。它是如下定义的：
```golang
message ConfigGroup {
    uint64 version = 1;
    map<string,ConfigGroup> groups = 2;
    map<string,ConfigValue> values = 3;
    map<string,ConfigPolicy> policies = 4;
    string mod_policy = 5;
}
```
因为`ConfigGroup`是个递归结构，它是按层次整理的。下面的例子是用go语言符号表示的。
```golang
// Assume the following groups are defined
var root, child1, child2, grandChild1, grandChild2, grandChild3 *ConfigGroup

// Set the following values
root.Groups["child1"] = child1
root.Groups["child2"] = child2
child1.Groups["grandChild1"] = grandChild1
child2.Groups["grandChild2"] = grandChild2
child2.Groups["grandChild3"] = grandChild3

// The resulting config structure of groups looks like:
// root:
//     child1:
//         grandChild1
//     child2:
//         grandChild2
//         grandChild3
```
每个组在配置层次上定义了一个级别，每个组都有一组关联的值（由字符串类型的key索引）和策略（也由字符串类型的key索引）。  

Value被下列结构定义：
```golang
message ConfigValue {
    uint64 version = 1;
    bytes value = 2;
    string mod_policy = 3;
}
```
Policy由下列结构定义：
```golang
message ConfigPolicy {
    uint64 version = 1;
    Policy policy = 2;
    string mod_policy = 3;
}
```
请注意，Value、Policy和Group都有一个`version`和一个`mod_policy`。元素的`version`的值会在元素被修改时递增。`mod_policy`用来管理修改元素所需的签名。

对于Group，修改就是向Value、Policy或Group映射中添加或删除元素（或更改`mod_policy`）。对于Value和Policy，修改就是分别改变Value和Policy字段（或改变`mod_policy`）。

每个元素的`mod_policy`都在当前配置级别的上下文中进行评估。考虑在下面例子的`mod_policy`定义在`Channel.Groups["Application"]`（在这里，我们使用golang映射引用语法，所以`Channel.Groups["Application"].Policies["policy1"]`引用了基础`Channel`组的`Application`组的`Policies`映射的`policy1`策略。）

- `policy1`映射到`Channel.Groups["Application"].Policies["policy1"]`  
- `Org1/policy2`映射到`Channel.Groups["Application"].Groups["Org1"].Policies["policy2"]`  
- `/Channel/policy3`映射到`Channel.Policies["policy3"]`  

请注意，如果`mod_policy`引用了一个不存在的策略，则该项目不会被修改。  

### 配置更新
配置更新会递交一个`HeaderType_CONFIG_UPDAT`类型的`Envelope`消息。事务的`Payload``data`是一个marshaled`ConfigUpdateEnvelope`。` ConfigUpdateEnvelope`的定义如下：
```golan
message ConfigUpdateEnvelope {
    bytes config_update = 1;
    repeated ConfigSignature signatures = 2;
}
```
签名字段包含了一组授权配置更新的签名。它的消息定义是：
```golang
message ConfigSignature {
    bytes signature_header = 1;
    bytes signature = 2;
}
```
`signature_header`是标准事务的定义，而`signature`是`signature_header`字节和`ConfigUpdateEnvelope`消息的`config_update`字节串连后的签名。

`ConfigUpdateEnvelope`的`config_update`字节是一个marshaled`ConfigUpdate`消息，该消息定义如下：
```golang
message ConfigUpdate {
    string channel_id = 1;
    ConfigGroup read_set = 2;
    ConfigGroup write_set = 3;
}
```
这`channel_id`是更新要绑定的通道ID，这对于限定此重新配置的签名范围是必要的。

在`read_set`指定了现有的配置的一个子集，它暗示了只有`version`字段必须提供的，而且字段是可选则。字段`ConfigValue``value`或 `ConfigPolicy``policy`不允许出现在`read_set`。`ConfigGroup`可以填充它映射字段的子集，以便引用在配置树更深的元素。例如，要`read_set`中包括`Application`组，则它的母元素（该`Channel`组）也必须包含在`read_set`中，但是，该`Channel`组并不需要填充所有的key，如`Orderer``group`key，或任何`values`或`policies`key。

`write_set`定义了配置的被修改部分。由于配置的层次性，对层次结构中深层元素的写入操作也必须在`write_set`从包含其更高层次的元素。但是，对于`write_set`中任何元素，如果也出现在`read_set`中且版本相同，则会被更新忽略（？）。

例如，对于给定配置：
```
Channel: (version 0)
    Orderer (version 0)
    Appplication (version 3)
       Org1 (version 2)
```
去递交一个改变`Org1`的配置更新，`read_set`会是：
```
Channel: (version 0)
    Application: (version 3)
```
而它的`write_set`会是：
```
Channel: (version 0)
    Application: (version 3)
        Org1 (version 3)
```
当收到`CONFIG_UPDATE`，orderer会按下面的步骤计算结果集`CONFIG`：
1. 检验`channel_id`和`read_set`。所有`read_set`中元素必须存在且版本号相同。  
2. 通过比较存在于`write_set`中而不存在于`read_set`中且版本号相同的元素，得到更新元素集。  
3. 检验更新元素集中的元素版本号，确保只增长了1。  
4. 对更新元素集中的每个元素，检验针对`ConfigUpdateEnvelope`的签名集，确保符合`mod_policy`。  
5. 通过将更新集应用到当前配置，计算出配置的一个新完全版本。  
6. 将新配置写入`ConfigEnvelope`，将`CONFIG_UPDATE`作为`the last_update`字段，并与增长的`sequence`值一起将新配置编码入`config`字段。  
7. 将一个新的`ConfigEnvelope`以类型`CONFIG`写入`Envelope`，最后将这个作为唯一事务写入一个新配置区块。  

当新的peer(或任何其他的`Deliver`接收者)接收到这个配置区块，它会校验这个配置，方法是将`last_update`消息应用到当前配置，确保orderer计算出来的`config`字段包含了正确的新配置。  

### 允许的配置组(group)和值(value)
任何有效的配置都是下面配置的子集。这里我们使用符号`peer.<MSG>`来定义一个`ConfigValue`，它的`value`字段是一个名为`<MSG>`的marshaled proto消息（定义在`fabric/protos/peer/configuration.proto`）。符号`common.<MSG>`、`msp.<MSG>`和`orderer.<MSG>`含义类似，只是它们的消息分别定义在`fabric/protos/common/configuration.proto`、`fabric/protos/msp/mspconfig.proto`和`fabric/protos/orderer/configuration.proto`。  

需要注意的是key`{{org_name}}`和`{{consortium_name}}`可以是任意名字，这表明一个元素可以以任何名字重复出现。  
```golang
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
        "Orderer":&ConfigGroup{
            Groups:map<String, *ConfigGroup> {
                {{org_name}}:&ConfigGroup{
                    Values:map<string, *ConfigValue>{
                        "MSP":msp.MSPConfig,
                    },
                },
            },

            Values:map<string, *ConfigValue> {
                "ConsensusType":orderer.ConsensusType,
                "BatchSize":orderer.BatchSize,
                "BatchTimeout":orderer.BatchTimeout,
                "KafkaBrokers":orderer.KafkaBrokers,
            },
        },
        "Consortiums":&ConfigGroup{
            Groups:map<String, *ConfigGroup> {
                {{consortium_name}}:&ConfigGroup{
                    Groups:map<string, *ConfigGroup> {
                        {{org_name}}:&ConfigGroup{
                            Values:map<string, *ConfigValue>{
                                "MSP":msp.MSPConfig,
                            },
                        },
                    },
                    Values:map<string, *ConfigValue> {
                        "ChannelCreationPolicy":common.Policy,
                    }
                },
            },
        },
    },

    Values: map<string, *ConfigValue> {
        "HashingAlgorithm":common.HashingAlgorithm,
        "BlockHashingDataStructure":common.BlockDataHashingStructure,
        "Consortium":common.Consortium,
        "OrdererAddresses":common.OrdererAddresses,
    },
}
```

### 排序系统通道配置
排序系统通道需要定义排序参数，和创建通道的联盟。对一个排序服务必须存在一个排序系统通道，它是创建的第一个通道（更准确地说是引导时创建）。建议不要在排序系统通道的创世配置中定义应用部分（的通道），但可以在测试时这样做。注意，对排序系统通道具有读权限的成员可以看到所有通道（系统的和应用的所有通道）的创建，所以这个通道的访问权限需要严格控制。

排序参数用下面配置的子集来定义：
```golang
&ConfigGroup{
    Groups: map<string, *ConfigGroup> {
        "Orderer":&ConfigGroup{
            Groups:map<String, *ConfigGroup> {
                {{org_name}}:&ConfigGroup{
                    Values:map<string, *ConfigValue>{
                        "MSP":msp.MSPConfig,
                    },
                },
            },

            Values:map<string, *ConfigValue> {
                "ConsensusType":orderer.ConsensusType,
                "BatchSize":orderer.BatchSize,
                "BatchTimeout":orderer.BatchTimeout,
                "KafkaBrokers":orderer.KafkaBrokers,
            },
        },
    },
```
参与排序的每个组织在`Orderer`组下都有一个组元素。这个组定义一个`MSP`参数，它包含了组织的密钥id信息。`Orderer`组的`Values`确定了排序节点的功能。它们存在于每个通道，所以实例的`orderer.BatchTimeout`可以在不同的通道定义不同的值。

启动时，orderer会面对一个包含了多个通道信息的文件系统。orderer根据联盟组通道定义出系统通道。联盟组具有下面的结构：
```golang
&ConfigGroup{
    Groups: map<string, *ConfigGroup> {
        "Consortiums":&ConfigGroup{
            Groups:map<String, *ConfigGroup> {
                {{consortium_name}}:&ConfigGroup{
                    Groups:map<string, *ConfigGroup> {
                        {{org_name}}:&ConfigGroup{
                            Values:map<string, *ConfigValue>{
                                "MSP":msp.MSPConfig,
                            },
                        },
                    },
                    Values:map<string, *ConfigValue> {
                        "ChannelCreationPolicy":common.Policy,
                    }
                },
            },
        },
    },
},
```
注意，每个联盟定义了一组成员，就像排序组织成员一样。每个联盟还定义了一个ChannelCreationPolicy。这个策略用于对通道创建请求进行授权。通常，这个值被设为`ImplicitMetaPolicy`，意思是要求通道的新成员签名以授权频道创建。关于通道创建的更多内容在文章的后面还有。

### 应用通道配置
通道的应用配置被设计用于应用类型的事务。它的定义如下：
```golang
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
很像`Orderer`部分，每个组织被编码为一个组(group)。然而，不仅仅编码了`MSP`id信息，每个组织附加编码了一个`AnchorPeers`列表。这是一个允许不同组织之间通过peer gossip网络互相联系的peer列表（这个列表中的peer才可以被其他组织的peer“看到”）。

应用通道编码了一个orderer组织的副本，和用于确定变更这些参数的共识选项，所以包含了系统通道配置的`Orderer`部分的相同内容。然而，从应用的观点这可以被大大忽略。  

### 通道创建
当排序节点收到一个不存在的通道的`CONFIG_UPDATE`，排序节点就假定这是个通道创建请求，然后执行下列操作：
1. orderer确定发出通道创建请求的联盟身份。它通过查看顶层group的`Consortium`值来做到这一点。  
2. orderer验证包含在`Application`组的组织，确保这些组织是联盟下属(子集)，而且`ApplicationGroup`被设置为`version``1`。  
3. orderer验证联盟是否有成员，从而新通道会有应用成员(创建没有成员的联盟和通道仅用于测试)。  
4. orderer通过从排序系统通道中取得`Orderer`组来创建一个模板配置，用新的成员创建一个`Application`组并指定`mod_policy`为联盟配置里的`ChannelCreationPolicy`。需要注意的是策略是在新配置的上下文中被评估的，所以一个需要`ALL`成员的策略仅需要全部新通道成员的签名，而不是联盟的所有成员签名。  
5. orderer将`CONFIG_UPDATE`作为一个更新应用到这个模板配置。因为这个`CONFIG_UPDATE`应用变更到`Application`组(它的`version`是`1`)，配置代码用`ChannelCreationPolicy`验证这些更新。如果通道创建包含其他变更，如个别组织的锚peer，元素的相应mod policy会被调用。  
6. 为了排序，含有新通道配置的新`CONFIG`事务会被包装和发送到排序系统通道。排序后，通道就创建了。  

## 通道配置(configtxgen)
本文档介绍了`configtxgen`的用法，它是操作Hyperledger Fabric通道配置的实用程序。

目前，该工具主要侧重于生成用于引导orderer的创世区块，但是将来要增强生成新的通道配置以及重新配置现有通道。  

### 配置profile
为`configtxgen`工具提供配置参数的主要供给是文件`configtx.yaml`。这个位于fabric.git库的`fabric/sampleconfig/configtx.yaml`位置。

这个配置文件主要分为三部分：

1. `Profiles`部分。默认情况下，这个部分包含一些可以用于开发和测试场景示范配置，使用了fabric.git树中的密钥材料。这些profile对于组织一个实际的部署profile是个很好的起点。`configtxgen`工具允许你用`-profile`标志来指定profile。profile可以明确地表明所有配置，但通常从第3部分(默认部分)继承配置。  
2. `Organizations`部分。默认情况下，这部分包含指向sampleconfig MSP定义的一个简单引用。对于生产部署，示范组织会被移除，网络成员的MSP定义会被引用和定义。`Organizations`部分的每个元素都可以打上一个锚点标签如`&orgName`，这允许这个定义被`Profiles`部分引用。  
3. 默认部分。这里有`Orderer`和`Application`部分的默认配置，包括象`BatchTimeout`这样的属性和通常用作profile的基本继承值的属性。  

这个配置文件可以被编辑，或者单个属性可能被设置环境变量覆盖，比如`CONFIGTX_ORDERER_ORDERERTYPE=kafka`。请注意，`Profiles` 元素和profile名称不需要指定。

### 引导orderer
创建一个期望的配置profile，简单调用：
```
$ configtxgen -profile <profile_name> -outputBlock orderer_genesisblock.pb
```
将在当面目录下创建一个`orderer_genesisblock.pb`文件。这个创世区块被用于引导排序系统通道，该通道被orderer用于授权和编排其他通道的创建。默认情况下，由`configtxgen`生成，编码进创世区块的通道ID是`testchainid`。建议你修改这个值为某个全局唯一的值。  

为了利用生成的创世区块，在启动orderer之前，需要设定环境变量：
```
ORDERER_GENERAL_GENESISMETHOD=file
ORDERER_GENERAL_GENESISFILE=$PWD/orderer_genesisblock.pb
```
或者修改配置文件orderer.yaml，将上述值包含进文件中。  

### 创建一个通道
这个工具可以通过执行下列命令输出一个通道创建tx：
```
$ configtxgen -profile <profile_name> -channelID <channel_name> -outputCreateChannelTx <tx_filename>
```
这会输出一个可以广播出去创建通道的marshaled `Envelope`消息。

### 显示一个配置
除了生成配置以外，`configtxgen`工具还有查看配置的能力。  
它支持查看配置区块和配置事务。可以分别使用查看标志`-inspectBlock`和`-inspectChannelCreateTx`并在后面附加文件路径来输出一个JSON串来显示配置信息。  
还可以对查看标志进行组合，例如：
```json
$ configtxgen -channelID foo -outputBlock foo_genesisblock.pb -inspectBlock foo_genesisblock.pb
2017-11-02 17:56:04.489 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
2017-11-02 17:56:04.564 EDT [common/tools/configtxgen] doOutputBlock -> INFO 002 Generating genesis block
2017-11-02 17:56:04.564 EDT [common/tools/configtxgen] doOutputBlock -> INFO 003 Writing genesis block
2017-11-02 17:56:04.564 EDT [common/tools/configtxgen] doInspectBlock -> INFO 004 Inspecting block
2017-11-02 17:56:04.564 EDT [common/tools/configtxgen] doInspectBlock -> INFO 005 Parsing genesis block
{
  "data": {
    "data": [
      {
        "payload": {
          "data": {
            "config": {
              "channel_group": {
                "groups": {
                  "Consortiums": {
                    "groups": {
                      "SampleConsortium": {
                        "mod_policy": "/Channel/Orderer/Admins",
                        "values": {
                          "ChannelCreationPolicy": {
                            "mod_policy": "/Channel/Orderer/Admins",
                            "value": {
                              "type": 3,
                              "value": {
                                "rule": "ANY",
                                "sub_policy": "Admins"
                              }
                            },
                            "version": "0"
                          }
                        },
                        "version": "0"
                      }
                    },
                    "mod_policy": "/Channel/Orderer/Admins",
                    "policies": {
                      "Admins": {
                        "mod_policy": "/Channel/Orderer/Admins",
                        "policy": {
                          "type": 1,
                          "value": {
                            "rule": {
                              "n_out_of": {
                                "n": 0
                              }
                            },
                            "version": 0
                          }
                        },
                        "version": "0"
                      }
                    },
                    "version": "0"
                  },
                  "Orderer": {
                    "mod_policy": "Admins",
                    "policies": {
                      "Admins": {
                        "mod_policy": "Admins",
                        "policy": {
                          "type": 3,
                          "value": {
                            "rule": "MAJORITY",
                            "sub_policy": "Admins"
                          }
                        },
                        "version": "0"
                      },
                      "BlockValidation": {
                        "mod_policy": "Admins",
                        "policy": {
                          "type": 3,
                          "value": {
                            "rule": "ANY",
                            "sub_policy": "Writers"
                          }
                        },
                        "version": "0"
                      },
                      "Readers": {
                        "mod_policy": "Admins",
                        "policy": {
                          "type": 3,
                          "value": {
                            "rule": "ANY",
                            "sub_policy": "Readers"
                          }
                        },
                        "version": "0"
                      },
                      "Writers": {
                        "mod_policy": "Admins",
                        "policy": {
                          "type": 3,
                          "value": {
                            "rule": "ANY",
                            "sub_policy": "Writers"
                          }
                        },
                        "version": "0"
                      }
                    },
                    "values": {
                      "BatchSize": {
                        "mod_policy": "Admins",
                        "value": {
                          "absolute_max_bytes": 10485760,
                          "max_message_count": 10,
                          "preferred_max_bytes": 524288
                        },
                        "version": "0"
                      },
                      "BatchTimeout": {
                        "mod_policy": "Admins",
                        "value": {
                          "timeout": "2s"
                        },
                        "version": "0"
                      },
                      "ChannelRestrictions": {
                        "mod_policy": "Admins",
                        "version": "0"
                      },
                      "ConsensusType": {
                        "mod_policy": "Admins",
                        "value": {
                          "type": "solo"
                        },
                        "version": "0"
                      }
                    },
                    "version": "0"
                  }
                },
                "mod_policy": "Admins",
                "policies": {
                  "Admins": {
                    "mod_policy": "Admins",
                    "policy": {
                      "type": 3,
                      "value": {
                        "rule": "MAJORITY",
                        "sub_policy": "Admins"
                      }
                    },
                    "version": "0"
                  },
                  "Readers": {
                    "mod_policy": "Admins",
                    "policy": {
                      "type": 3,
                      "value": {
                        "rule": "ANY",
                        "sub_policy": "Readers"
                      }
                    },
                    "version": "0"
                  },
                  "Writers": {
                    "mod_policy": "Admins",
                    "policy": {
                      "type": 3,
                      "value": {
                        "rule": "ANY",
                        "sub_policy": "Writers"
                      }
                    },
                    "version": "0"
                  }
                },
                "values": {
                  "BlockDataHashingStructure": {
                    "mod_policy": "Admins",
                    "value": {
                      "width": 4294967295
                    },
                    "version": "0"
                  },
                  "HashingAlgorithm": {
                    "mod_policy": "Admins",
                    "value": {
                      "name": "SHA256"
                    },
                    "version": "0"
                  },
                  "OrdererAddresses": {
                    "mod_policy": "/Channel/Orderer/Admins",
                    "value": {
                      "addresses": [
                        "127.0.0.1:7050"
                      ]
                    },
                    "version": "0"
                  }
                },
                "version": "0"
              },
              "sequence": "0",
              "type": 0
            }
          },
          "header": {
            "channel_header": {
              "channel_id": "foo",
              "epoch": "0",
              "timestamp": "2017-11-02T21:56:04.000Z",
              "tx_id": "6acfe1257c23a4f844cc299cbf53acc7bf8fa8bcf8aae8d049193098fe982eab",
              "type": 1,
              "version": 1
            },
            "signature_header": {
              "nonce": "eZOKru6jmeiWykBtSDwnkGjyQt69GwuS"
            }
          }
        }
      }
    ]
  },
  "header": {
    "data_hash": "/86I/7NScbH/bHcDcYG0/9qTmVPWVoVVfSN8NKMARKI=",
    "number": "0"
  },
  "metadata": {
    "metadata": [
      "",
      "",
      "",
      ""
    ]
  }
}
```
上述命令先生成区块，再显示它。  

## 用configtxlator重新配置
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/configtxlator.html)  
### 概述
创建`configtxlator`工具的目的是支持不依赖SDK的重新配置。通道配置被当作一个事务存储在通道的配置区块中，可以被直接操作，如在bdd行为测试中。不过，在撰写本文时，SDK本身不支持直接操作配置，因此该configtxlator工具旨在提供API，供任何SDK的使用者与之交互以协助配置更新。

工具名称是configtx和translator的混合，旨在表达该工具只是在两者之间进行转换。它不会生成配置。它不递交或检索配置。它不会自己修改配置，它只是在configtx格式的不同视图之间提供一些双向操作。

标准用法为：

1. SDK检索最新的配置  
2. `configtxlator` 产生人类可读的配置版本  
3. 用户或应用程序编辑配置  
4. `configtxlator`用于计算配置的变更  
5. SDK递交签名和递交配置  

`configtxlator`工具暴露了一个真正的无状态的REST API来与配置元素进行交互。这些REST组件支持将原生配置格式转换为人类可读的JSON格式，或者根据两种配置之间的差异来计算配置变更。

由于`configtxlator`服务故意不包含任何密钥材料或其他秘密信息，因此不包含任何授权或访问控制。预期的典型部署将在本地与应用程序一起作为沙盒容器来运行，因而`configtxlator`是个为消费者提供的专用进程。

### 运行configtxlator
`configtxlator`工具可以与其他Hyperledger Fabric平台相关的二进制文件一起下载。 详细信息查看download-platform-specific-binaries。

该工具可能被配置为侦听不同的端口，您也可以使用`--port`和`--hostname`标志来指定主机名。要查看完整的命令和标志，请运行`configtxlator --help`。

该二进制包将启动一个监听指定端口的http服务器，然后就可以处理请求了。

要启动`configtxlator`服务器：
```
$ configtxlator start
2017-06-21 18:16:58.248 HKT [configtxlator] startServer -> INFO 001 Serving HTTP requests on 0.0.0.0:7059
```
### Proto转换
为了扩展性，并且由于某些字段必须被签名，许多proto字段被存储为字节。这使得无法使用`jsonpb`包来完成原生proto到JSON的翻译。作为代替，`configtxlator`暴露了一个REST组件来做更复杂的翻译。

为了将proto转换为人类可读的JSON等价物，只需将二进制proto发布到REST地址`http://$SERVER:$PORT/protolator/decode/<message.Name>`，其中`<message.Name>`是消息的完全限定的proto名称。

例如，要解码一个配置区块并另存为`configuration_block.pb`，请运行以下命令：:
```
$ curl -X POST --data-binary @configuration_block.pb http://127.0.0.1:7059/protolator/decode/common.Block
```
为了转换人类可读JSON版本的prot消息，只需要把JSON消息发送到`http://$SERVER:$PORT/protolator/encode/<message.Name`，这里`<message.Name>`仍然是消息的完全限定的proto名称。  
例如，重编码区块另存为`configuration_block.json`，运行命令：
```
$ curl -X POST --data-binary @configuration_block.json http://127.0.0.1:7059/protolator/encode/common.Block
```
任何配置相关proto，包括`common.Block`、`common.Envelope`、`common.ConfigEnvelope`、`common.ConfigUpdateEnvelope`
、`common.Config`和`common.ConfigUpdate`都是这些URL有效目标。未来，可能会增加其它proto编码类型，如背书者事务。  

### 配置更新计算
给定两种不同的配置，可以计算在它们之间转换的配置更新。只需将两个`common.Config`proto编码的配置作为`multipart/formdata`、original作为字段`original`和updated作为字段`updated`，发送到`http://$SERVER:$PORT/configtxlator/compute/update-from-configs`。

例如，给定原始配置文件`original_config.pb`和更新的配置文件`updated_config.pb`，对于通道`desiredchannel`：
```
$ curl -X POST -F channel=desiredchannel -F original=@original_config.pb -F updated=@updated_config.pb http://127.0.0.1:7059/configtxlator/compute/update-from-configs
```
### 自举例子
启动`configtxlator`:
```
$ configtxlator start
2017-05-31 12:57:22.499 EDT [configtxlator] main -> INFO 001 Serving HTTP requests on port: 7059
```
首先，为排序系统通道生成一个创世区块：
```
$ configtxgen -outputBlock genesis_block.pb
2017-05-31 14:15:16.634 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
2017-05-31 14:15:16.646 EDT [common/configtx/tool] doOutputBlock -> INFO 002 Generating genesis block
2017-05-31 14:15:16.646 EDT [common/configtx/tool] doOutputBlock -> INFO 003 Writing genesis block
```
将创世区块解码为人类可编辑格式：
```
$ curl -X POST --data-binary @genesis_block.pb http://127.0.0.1:7059/protolator/decode/common.Block > genesis_block.json
```
编辑生成的文件`genesis_block.json`，或者用程序操作它。这里我们使用JOSN CLI工具`jq`。为了简单，我们编辑通道的批大小，因为它是个简单的数字。然而，所有都可以编辑，包括策略和MSP。

首先，让我们建立一个环境变量来保存指向json中属性的路径：
```
$ export MAXBATCHSIZEPATH=".data.data[0].payload.data.config.channel_group.groups.Orderer.values.BatchSize.value.max_message_count"
```
然后，让我们显示这个属性的值：
```
$ jq "$MAXBATCHSIZEPATH" genesis_block.json
10
```
现在，我们设置新的批大小，并显示这个新值：
```
$ jq “$MAXBATCHSIZEPATH = 20” genesis_block.json > updated_genesis_block.json
$ jq “$MAXBATCHSIZEPATH” updated_genesis_block.json
20
```
创世区块现在准备好被重编码为用于自举的原生proto格式：
```
$ curl -X POST --data-binary @updated_genesis_block.json http://127.0.0.1:7059/protolator/encode/common.Block > updated_genesis_block.pb
```
`updated_genesis_block.pb`文件现在可以作为创世区块用于一个排序系统通道的自举。

### 重新配置的例子
*（先用命令删除所有docker容器`docker rm -f $(docker ps -aq)`，否则byfn.sh启动的排序服务也会绑定端口7050）*  
利用另外的终端窗口，使用默认配置启动排序服务，临时自举器会创建一个`testchainid`排序系统通道。
```
ORDERER_GENERAL_LOGLEVEL=debug orderer
```
重新配置一个通道的操作非常类似于改变一个创世配置。

*（现在configtxlator、orderer各占了一个终端窗口，下面需要启动第三个终端窗口）*  
首先，取得配置区块proto：
```
$ peer channel fetch config config_block.pb -o 127.0.0.1:7050 -c testchainid
-o 127.0.0.1:7050 -c testchainid
2017-12-11 09:00:15.658 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2017-12-11 09:00:15.660 UTC [main] main -> INFO 002 Exiting.....
```
*(当前目录下创建了文件`config_block.pb`)*  
然后，发送配置区块到`configtxlator`服务进行解码：
```
$ curl -X POST --data-binary @config_block.pb http://127.0.0.1:7059/protolator/decode/common.Block > config_block.json
```
从区块中提取配置节：
```
$ jq .data.data[0].payload.data.config config_block.json > config.json
```
编辑配置，把它另存为一个新的`updated_config.json`。这里，我们设置批大小为30。
```
$ jq ".channel_group.groups.Orderer.values.BatchSize.value.max_message_count = 30" config.json  > updated_config.json
```
对原始配置和变更的配置进行重编码到proto格式：
```
$ curl -X POST --data-binary @config.json http://127.0.0.1:7059/protolator/encode/common.Config > config.pb
$ curl -X POST --data-binary @updated_config.json http://127.0.0.1:7059/protolator/encode/common.Config > updated_config.pb
```
现在，两个配置都进行适当编码，发送它们到*configtxlator*服务，去计算它们之间的配置变更。
```
$ curl -X POST -F original=@config.pb -F updated=@updated_config.pb http://127.0.0.1:7059/configtxlator/compute/update-from-configs -F channel=testchainid > config_update.pb
```
这时，计算出来的配置变更已经准备好了。传统上，使用一个SDK签名和包装这个消息。然而，为了仅使用peer cli，*configtxlator*也可以用于完成这个任务。

首先，我们对配置变更(ConfigUPdate)进行解码，以便可以用文本方式操作它：
```
$ curl -X POST --data-binary @config_update.pb http://127.0.0.1:7059/protolator/decode/common.ConfigUpdate > config_update.json
```
然后，我们把它包裹进一个信封(envelope)消息：
```
$ echo '{"payload":{"header":{"channel_header":{"channel_id":"testchainid", "type":2}},"data":{"config_update":'$(cat config_update.json)'}}}' > config_update_as_envelope.json
```
接下来，将其转换回完整的配置事务的proto格式：
```
$ curl -X POST --data-binary @config_update_as_envelope.json http://127.0.0.1:7059/protolator/encode/common.Envelope > config_update_as_envelope.pb
```
最后，发送配置变更事务到排序服务去执行一个配置变更。
```
$ peer channel update -f config_update_as_envelope.pb -c testchainid -o 127.0.0.1:7050
```
### 增加一个组织
先启动`configtxlator`：
```
$ configtxlator start
2017-05-31 12:57:22.499 EDT [configtxlator] main -> INFO 001 Serving HTTP requests on port: 7059
```
使用`SampleDevModeSolo`profile选项启动排序服务。
```
$ ORDERER_GENERAL_LOGLEVEL=debug ORDERER_GENERAL_GENESISPROFILE=SampleDevModeSolo orderer
```
增加一个组织的过程与批大小的例子非常类似。然而，不同与设置批大小，一个新组织定义在应用级别。添加一个组织稍微牵扯一点，因为我们必须先创建一个通道，然后修改它的成员组成。  

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

# Fabric实战

## MSP配置(cryptogen)
要运行cryptogen，需要一个指定一个配置文件`crypto-config.yaml`，如：
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

## 生成创世区块(configtxgen)
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

### 用configtxgen查看区块信息
```
$ export FABRIC_CFG_PATH=/opt/bcnet
$ configtxgen -profile TwoOrgsOrdererGenesis -inspectBlock ./channel-artifacts/genesis.block
```
`configtxgen`会到FABRIC_CFG_PATH环境变量定义的目录下找`configtx.yaml`这个配置文件，并按配置文件中的定义先找`profile`，根据`profile`的值所对应的`MSPDir`下寻找密钥文件，然后利用密钥文件对创世区块进行解密并转换为json格式输出。

有的创世区块可以不加`-profile`参数就能正常显示，原因不明。  

## 生成通道(configtxgen)
```
$ export FABRIC_CFG_PATH=/opt/fabric-samples/first-network
$ configtxgen -profile TwoOrgChannel -outputCreateChannelTx ./channel-artifacts/channel1.tx -channelID channel1
```

## peer加入通道(peer)
peer直接在宿主机上直接运行会报错，一般进入`cli`容器内执行:
```
$ docker exec -it cli bash
$$ peer --help
```

