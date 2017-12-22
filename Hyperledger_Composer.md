## 部署Hyperledger Composer网络到多组织Fabric
[原文：Deploying a Hyperledger Composer blockchain business network to Hyperledger Fabric (multiple organizations)](https://hyperledger.github.io/composer/unstable/tutorials/deploy-to-fabric-multi-org.html)   

这个教程示范了在多组织场景下的管理员部署一个区块链商业网络到某Hyperledger Fabric实例必须执行的几个步骤，包括怎样生成Hyperledger Composer配置。

建议首先完成前面的教程，也就是示范怎样为单个组织将一个区块链商业网络部署到某Hyperledger Fabric实例，因为它更详细地解释了一些概念。

这个教程将涵盖怎样部署一个区块链商业网络到某个跨越两个组织(`Org1`和`Org2`)的Hyperledger Fabric网络。这个教程会根据不同组织执行的步骤展现为不同的方式。

首先是两个组织都要执行的步骤显示的样子：  
*================示例步骤：一个Org1和Org2都执行的步骤================*  

表示组织`Org1`执行的步骤的样子:  
*----------------示例步骤：一个Org1执行的步骤----------------*  

表示组织`Org2`执行的步骤的样子：  
*________________示例步骤：一个Org2执行的步骤________________*  

你可以自己执行这些步骤，或者与朋友或同事一起执行这些步骤。  
让我们开始！  

### ================准备================
如果你已经安装了开发环境，你需要首先停止开发环境提供的Hyperledger Fabric：
```bash
cd ~/fabric-tools
./stopFabric.sh
./teardownFabric.sh
```

克隆下面的Github库：
```bash
git clone -b issue-6978 https://github.com/sstone1/fabric-samples.git
```
按照[建立你的首个网络](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#toc7)教程，确保你使用了上面步骤克隆的Github库。你不能克隆和使用Hyperledger Fabric版本的Github库，因为它当前没有更新到这个教程需要的内容。

### ================步骤一：启动一个Hyperledger Fabric网络================
为了遵照这个教程，你必须启动一个Hyperledger Fabric网络。

这个教程假定你使用了Hyperledger Fabric的[建立你的首个网络](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#toc7)教程中提供的 Hyperledger Fabric网络。我会称这个 Hyperledger Fabric网络为BYFN(Building Your First Network)网络。

你现在可以启动BYFN网络。你必须使用额外标志，这个标志在BYFN教程中并没有使用。这是因为我们想使用CouchDB作为世界状态数据库，并且我们想为每个组织启动一个CA(Certificate Authority)。
```bash
./byfn.sh -m generate
./byfn.sh -m up -s couchdb -a
```
(估计作者用的BYFN版本旧，现在的1.1.0版BYFN默认启动couchdb，但不启动CA容器)  

如果你的命令执行成功，BYFN网络被启动，你会看到下面的输出：
```
========= All GOOD, BYFN execution completed ===========

_____   _   _   ____   
| ____| | \ | | |  _ \  
|  _|   |  \| | | | | |
| |___  | |\  | | |_| |
|_____| |_| \_| |____/
```
然后，删除存在于你钱包中的所有商业网络卡片。这是为了避免商业网络卡片没有找到的错误状态：
```bash
composer card delete -n PeerAdmin@byfn-network-org1-only
composer card delete -n PeerAdmin@byfn-network-org1
composer card delete -n PeerAdmin@byfn-network-org2-only
composer card delete -n PeerAdmin@byfn-network-org2
composer card delete -n alice@tutorial-network
composer card delete -n bob@tutorial-network
composer card delete -n admin@tutorial-network
composer card delete -n PeerAdmin@fabric-network
```
### ================步骤二：探索Hyperledger Fabric网络================
这一步骤会探索BYFN网络配置和组件。为了完成后续步骤需要一些配置明细。

#### 组织
BYFN网络由两个组织组成：`Org1`和`Org2`。组织`Org1`使用域名`org1.example.com`。`Org1`的成员服务提供者(MSP)叫`Org1MSP`。组织`Org2`使用域名`org2.example.com`。`Org2`的MSP叫`Org2MSP`。在这个教程中，你会部署一个区块链商业网络，在里面组织`Org1`和`Org2`互相交互。

#### 网络组件
Hyperledger Fabric网络由下面几个组件组成：
 - Org1的两个peer节点，叫peer0.org1.example.com 和 peer1.org1.example.com  
   - peer0的请求端口是7051  
   - peer0的事件hub端口是7053  
   - peer1的请求端口是8051  
   - peer1的事件hub端口是8053  
 - 一个Org1的CA(Certificate Authority)，叫 ca.org1.example.com  
   - CA端口是7054  
 - Org2的两个peer节点，叫peer0.org2.example.com 和 peer1.org2.example.com  
   - peer0的请求端口是9051  
   - peer0的事件hub端口是9053  
   - peer1的请求端口是10051  
   - peer1的事件hub端口是10053  
 - 一个Org1的CA(Certificate Authority)，叫 ca.org1.example.com  
   - CA端口是7054  
 - 一个排序节点，叫orderer.example.com  
   - 排序端口是7050  

这些组件运行在Docker容器中。当在一个Docker容器中运行Hyperledger Composer，与Hyperledger Fabric网络交互可以使用上面的名字(如`peer0.org1.example.com`)。

这个教程会在Docker宿主机中运行Hyperledger Composer命令，而不是在Docker网络中。这意味着Hyperledger Composer命令与Hyperledger Fabric网络交互使用`localhost`作为主机名和容器暴露的端口。

所有这些网络组件使用TLS加密通信来确保安全。为了连接到这些网络组件，你会使用它们的CA(Certificate Authority)证书。CA证书所在目录可以在byfn.sh脚本中找到。

排序(orderer)节点的CA证书：
```
crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
```
`Org1`的CA证书：
```
crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```
`Org2`的CA证书：
```
crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
```
之后你会使用这些文件与Hyperledger Fabric网络交互。

#### 用户
组织`Org1`配置了一个叫`Admin@org1.example.com`的用户。这个用户是一个管理员。

用户`Admin@org1.example.com`有一组证书和私钥文件保存在这个目录：
```
crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
```
组织`Org2`配置了一个叫`Admin@org2.example.com`的用户。这个用户是一个管理员。

用户`Admin@org2.example.com`有一组证书和私钥文件保存在这个目录：
```
crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
```
之后你会使用这些文件与Hyperledger Fabric网络交互。

除了管理员，`Org1`和`Org2`的CA(Certificate Authority)还配置了一个默认用户。这个默认用户有一个叫`admin`的登记ID和为`adminpw`的登记密码。然而，这个用户没有部署区块链商业网络的权限。

#### 信道
创建了一个叫`mychannel`的信道。所有四个节点`peer0.org1.example.com`、`peer1.org1.example.com`、`peer0.org2.example.com`和`peer1.org2.example.com`已经加入了这个信道。

### ----------------步骤三：创建Org1的连接profile----------------
`Org1`需要两个连接profile。一个连接profile仅包含属于`Org1`的peer节点，另一个连接profile包含属于`Org1`和`Org2`的peer节点。

创建一个叫`connection-org1-only.json`的连接profile，包含下列内容并保存到磁盘。这个连接profile仅包含属于`Org1`的peer节点。你会在后续步骤中使用这个文件，所以记住存放位置！
```json
{
    "name": "byfn-network-org1-only",
    "type": "hlfv1",
    "mspID": "Org1MSP",
    "peers": [
        {
            "requestURL": "grpcs://localhost:7051",
            "eventURL": "grpcs://localhost:7053",
            "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer0.org1.example.com"
        },
        {
            "requestURL": "grpcs://localhost:8051",
            "eventURL": "grpcs://localhost:8053",
            "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer1.org1.example.com"
        }
    ],
    "ca": {
        "url": "https://localhost:7054",
        "name": "ca-org1",
        "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
        "hostnameOverride": "ca.org1.example.com"
    },
    "orderers": [
        {
            "url" : "grpcs://localhost:7050",
            "cert": "INSERT_ORDERER_CA_CERT_FILE_PATH",
            "hostnameOverride": "orderer.example.com"
        }
    ],
    "channel": "mychannel",
    "timeout": 300
}
```
用指向`Org1`peer节点的CA证书文件的全路径名覆盖所有的`INSERT_ORG1_CA_CERT_FILE_PATH`文本：
```
crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```
用指向排序节点的CA证书文件的全路径名覆盖所有的`INSERT_ORDERER_CA_CERT_FILE_PATH`文本：
```
crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
```

创建一个叫`connection-org1.json`的连接profile，包含下列内容并保存到磁盘。这个连接profiled包含属于`Org1`和`Org2`的peer节点。你会在后续步骤中使用这个文件，所以记住存放位置！
```json
{
    "name": "byfn-network-org1",
    "type": "hlfv1",
    "mspID": "Org1MSP",
    "peers": [
        {
            "requestURL": "grpcs://localhost:7051",
            "eventURL": "grpcs://localhost:7053",
            "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer0.org1.example.com"
        },
        {
            "requestURL": "grpcs://localhost:8051",
            "eventURL": "grpcs://localhost:8053",
            "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer1.org1.example.com"
        },
        {
            "requestURL": "grpcs://localhost:9051",
            "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer0.org2.example.com"
        },
        {
            "requestURL": "grpcs://localhost:10051",
            "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer1.org2.example.com"
        }
    ],
    "ca": {
        "url": "https://localhost:7054",
        "name": "ca-org1",
        "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
        "hostnameOverride": "ca.org1.example.com"
    },
    "orderers": [
        {
            "url" : "grpcs://localhost:7050",
            "cert": "INSERT_ORDERER_CA_CERT_FILE_PATH",
            "hostnameOverride": "orderer.example.com"
        }
    ],
    "channel": "mychannel",
    "timeout": 300
}
```
用指向`Org1`peer节点的CA证书文件的全路径名覆盖所有的`INSERT_ORG1_CA_CERT_FILE_PATH`文本：
```
crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```
用指向`Org2`peer节点的CA证书文件的全路径名覆盖所有的`INSERT_ORG2_CA_CERT_FILE_PATH`文本：
```
crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
```
用指向排序节点的CA证书文件的全路径名覆盖所有的`INSERT_ORDERER_CA_CERT_FILE_PATH`文本：
```
crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
```
注意，这个连接profile包含了`Org2`peer节点的明细，它仅包含了请求端口，而没有包含事件hub端口。这是因为一个组织不能访问另一个组织的事件hub端口。

### ________________步骤四：创建Org2的连接profile________________
`Org2`需要两个连接profile。一个连接profile仅包含属于`Org2`的peer节点，另一个连接profile包含属于`Org2`和`Org1`的peer节点。

创建一个叫`connection-org2-only.json`的连接profile，包含下列内容并保存到磁盘。这个连接profile仅包含属于`Org2`的peer节点。你会在后续步骤中使用这个文件，所以记住存放位置！
```json
{
    "name": "byfn-network-org2-only",
    "type": "hlfv1",
    "mspID": "Org2MSP",
    "peers": [
        {
            "requestURL": "grpcs://localhost:9051",
            "eventURL": "grpcs://localhost:9053",
            "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer0.org2.example.com"
        },
        {
            "requestURL": "grpcs://localhost:10051",
            "eventURL": "grpcs://localhost:10053",
            "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer1.org2.example.com"
        }
    ],
    "ca": {
        "url": "https://localhost:8054",
        "name": "ca-org2",
        "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
        "hostnameOverride": "ca.org2.example.com"
    },
    "orderers": [
        {
            "url" : "grpcs://localhost:7050",
            "cert": "INSERT_ORDERER_CA_CERT_FILE_PATH",
            "hostnameOverride": "orderer.example.com"
        }
    ],
    "channel": "mychannel",
    "timeout": 300
}
```
用指向`Org2`peer节点的CA证书文件的全路径名覆盖所有的`INSERT_ORG2_CA_CERT_FILE_PATH`文本：
```
crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
```
用指向排序节点的CA证书文件的全路径名覆盖所有的`INSERT_ORDERER_CA_CERT_FILE_PATH`文本：
```
crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
```
创建一个叫`connection-org2.json`的连接profile，包含下列内容并保存到磁盘。这个连接profile包含属于`Org2`和`Org1`的peer节点。你会在后续步骤中使用这个文件，所以记住存放位置！
```json
{
    "name": "byfn-network-org2",
    "type": "hlfv1",
    "mspID": "Org2MSP",
    "peers": [
        {
            "requestURL": "grpcs://localhost:9051",
            "eventURL": "grpcs://localhost:9053",
            "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer0.org2.example.com"
        },
        {
            "requestURL": "grpcs://localhost:10051",
            "eventURL": "grpcs://localhost:10053",
            "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer1.org2.example.com"
        },
        {
            "requestURL": "grpcs://localhost:7051",
            "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer0.org1.example.com"
        },
        {
            "requestURL": "grpcs://localhost:8051",
            "cert": "INSERT_ORG1_CA_CERT_FILE_PATH",
            "hostnameOverride": "peer1.org1.example.com"
        }
    ],
    "ca": {
        "url": "https://localhost:8054",
        "name": "ca-org2",
        "cert": "INSERT_ORG2_CA_CERT_FILE_PATH",
        "hostnameOverride": "ca.org2.example.com"
    },
    "orderers": [
        {
            "url" : "grpcs://localhost:7050",
            "cert": "INSERT_ORDERER_CA_CERT_FILE_PATH",
            "hostnameOverride": "orderer.example.com"
        }
    ],
    "channel": "mychannel",
    "timeout": 300
}
```
用指向`Org2`peer节点的CA证书文件的全路径名覆盖所有的`INSERT_ORG2_CA_CERT_FILE_PATH`文本：
```
crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
```
用指向`Org1`peer节点的CA证书文件的全路径名覆盖所有的`INSERT_ORG1_CA_CERT_FILE_PATH`文本：
```
crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```
用指向排序节点的CA证书文件的全路径名覆盖所有的`INSERT_ORDERER_CA_CERT_FILE_PATH`文本：
```
crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt
```
注意，这个连接profile包含了`Org1`peer节点的明细，它仅包含了请求端口，而没有包含事件hub端口。这是因为一个组织不能访问另一个组织的事件hub端口。

### ----------------步骤五：一个Org1执行的步骤----------------