## 部署Hyperledger Composer网络到多组织Fabric
[原文：Deploying a Hyperledger Composer blockchain business network to Hyperledger Fabric (multiple organizations)](https://hyperledger.github.io/composer/unstable/tutorials/deploy-to-fabric-multi-org.html)   

这个教程示范了在多组织场景下的管理员部署一个区块链商业网络到某Hyperledger Fabric实例必须执行的几个步骤，包括怎样生成Hyperledger Composer配置。

建议首先完成前面的教程，也就是示范怎样为单个组织将一个区块链商业网络部署到某Hyperledger Fabric实例，因为它更详细地解释了一些概念。

这个教程将涵盖怎样部署一个区块链商业网络到某个跨越两个组织(`Org1`和`Org2`)的Hyperledger Fabric网络。这个教程会根据不同组织执行的步骤展现为不同的方式。

首先是两个组织都要执行的步骤显示的样子：  
**================示例步骤：一个Org1和Org2都执行的步骤================**  

表示组织`Org1`执行的步骤的样子:  
**----------------示例步骤：一个Org1执行的步骤----------------**  

表示组织`Org2`执行的步骤的样子：  
**....................示例步骤：一个Org2执行的步骤....................**  

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

### ....................步骤四：创建Org2的连接profile....................
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

### ----------------步骤五：定位Org1的Hyperledger Fabric管理员的证书和私钥----------------
我们Hyperledger Fabric网络的管理员是一个叫`Admin@org1.example.com`的用户。这个用户的证书和私钥存放在这个目录：
```
crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
```
你必须先为这个用户定位这个证书文件。证书是身份的公开部分。证书文件可以在子目录`signcerts`中找到，文件名是`Admin@org1.example.com-cert.pem`。

然后，你为这个用户定位私钥文件。私钥用于以这个身份签署交易。私钥文件可以在子目录`keystore`中找到。私钥文件名是一个长十六进制字符串，后缀是`_sk`，例如`78f2139bfcfc0edc7ada0801650ed785a11cfcdef3f9c36f3c8ca2ebfa00a59c_sk`。每次配置生成时这个名字会改变。

记住这些文件的路径，或者将它们复制到步骤三中创建的连接pfile文件`connection-org1.json`所在的目录。

### ....................步骤六：定位Org2的Hyperledger Fabric管理员的证书和私钥....................

我们Hyperledger Fabric网络的管理员是一个叫`Admin@org2.example.com`的用户。这个用户的证书和私钥存放在这个目录：
```
crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
```
你必须先为这个用户定位这个证书文件。证书是身份的公开部分。证书文件可以在子目录`signcerts`中找到，文件名是`Admin@org2.example.com-cert.pem`。

然后，你为这个用户定位私钥文件。私钥用于以这个身份签署交易。私钥文件可以在子目录`keystore`中找到。私钥文件名是一个长十六进制字符串，后缀是`_sk`，例如`d4889cb2a32e167bf7aeced872a214673ee5976b63a94a6a4e61c135ca2f2dbb_sk`。每次配置生成时这个名字会改变。

记住这些文件的路径，或者将它们复制到步骤四中创建的连接pfile文件`connection-org2.json`所在的目录。

### ----------------步骤七：为Org1的Hyperledger Fabric管理员生成商业网络卡片----------------

在这个步骤中你会为管理员创建商业网络卡片，用于将区块链商业网络部署到 Hyperledger Fabric网络。

运行`composer card create`命令创建一个商业网络卡片，使用包含`Org1`peer的连接profile。你必须指定三个文件的路径，无论新建还是前面步骤定位的(注意：*sk*文件不同)。
```bash
composer card create -p connection-org1-only.json -u PeerAdmin -c Admin@org1.example.com-cert.pem -k 78f2139bfcfc0edc7ada0801650ed785a11cfcdef3f9c36f3c8ca2ebfa00a59c_sk -r PeerAdmin -r ChannelAdmin
```
如果命令执行成功，一个叫`PeerAdmin@byfn-network-org1-only.card`的商业网络卡片会被写入到当前目录。

运行`composer card create`命令创建一个商业网络卡片，使用包含`Org1`和`Org2`的连接profile。你必须指定三个文件的路径，无论新建还是前面步骤定位的：
```bash
composer card create -p connection-org1.json -u PeerAdmin -c Admin@org1.example.com-cert.pem -k 78f2139bfcfc0edc7ada0801650ed785a11cfcdef3f9c36f3c8ca2ebfa00a59c_sk -r PeerAdmin -r ChannelAdmin
```
如果命令执行成功，一个叫`PeerAdmin@byfn-network-org1.card`的商业网络卡片会被写入到当前目录。

### ....................步骤八：为Org2的Hyperledger Fabric管理员创建商业网络卡片....................

在这个步骤中你会为管理员创建商业网络卡片，用于将区块链商业网络部署到 Hyperledger Fabric网络。

运行`composer card create`命令创建一个商业网络卡片，使用包含`Org2`peer的连接profile。你必须指定三个文件的路径，无论新建还是前面步骤定位的(注意：*sk*文件不同)。
```bash
composer card create -p connection-org2-only.json -u PeerAdmin -c Admin@org2.example.com-cert.pem -k d4889cb2a32e167bf7aeced872a214673ee5976b63a94a6a4e61c135ca2f2dbb_sk -r PeerAdmin -r ChannelAdmin
```
如果命令执行成功，一个叫`PeerAdmin@byfn-network-org2-only.card`的商业网络卡片会被写入到当前目录。

运行`composer card create`命令创建一个商业网络卡片，使用包含`Org2`和`Org1`的连接profile。你必须指定三个文件的路径，无论新建还是前面步骤定位的：
```bash
composer card create -p connection-org2.json -u PeerAdmin -c Admin@org2.example.com-cert.pem -k d4889cb2a32e167bf7aeced872a214673ee5976b63a94a6a4e61c135ca2f2dbb_sk -r PeerAdmin -r ChannelAdmin
```
如果命令执行成功，一个叫`PeerAdmin@byfn-network-org2.card`的商业网络卡片会被写入到当前目录。

### ----------------步骤九：为Org1的Hyperledger Fabric管理员导入商业网络卡片----------------

运行`composer card import`命令将包含`Org1`peer的商业网络卡片导入进钱包：
```bash
composer card import -f PeerAdmin@byfn-network-org1-only.card
```
如果这个命令执行成功，一个叫`PeerAdmin@byfn-network-org1-only`的商业网络卡片会被导入进钱包。

运行`composer card import`命令将包含`Org1`和`Org2`peer的商业网络卡片导入进钱包：
```bash
composer card import -f PeerAdmin@byfn-network-org1.card
```
如果命令执行成功，一个叫`PeerAdmin@byfn-network-org1`的商业网络卡片会被导入进钱包。

### ....................步骤十：为Org2的Hyperledger Fabric管理员导入商业网络卡片....................

运行`composer card import`命令将包含`Org2`peer的商业网络卡片导入进钱包：
```bash
composer card import -f PeerAdmin@byfn-network-org2-only.card
```
如果这个命令执行成功，一个叫`PeerAdmin@byfn-network-org2-only`的商业网络卡片会被导入进钱包。

运行`composer card import`命令将包含`Org2`和`Org1`peer的商业网络卡片导入进钱包：
```bash
composer card import -f PeerAdmin@byfn-network-org2.card
```
如果命令执行成功，一个叫`PeerAdmin@byfn-network-org2`的商业网络卡片会被导入进钱包。

### ----------------步骤十一：为Org1的Hyperledger Fabric peer节点安装Hyperledger Composer运行时----------------

使用你在步骤三中创建的连接profle，运行`composer runtime install`命令为所有的`Org1`peer节点安装Hyperledger Composer运行时：
```bash
composer runtime install -c PeerAdmin@byfn-network-org1-only -n tutorial-network
```

### ....................步骤十二：为Org2的Hyperledger Fabric peer节点安装Hyperledger Composer运行时....................

使用你在步骤四中创建的连接profle，运行`composer runtime install`命令为所有的`Org2`peer节点安装Hyperledger Composer运行时：
```bash
composer runtime install -c PeerAdmin@byfn-network-org2-only -n tutorial-network
```

### ================步骤十三：为商业网络定义背书策略================

一个运行中的商业网络具有一个背书策略，它定义交易在提交到区块链之前那些组织必须对交易进行背书的规则。默认情况下，商业网络部署的背书策略是，一个交易被提交到区块链之前只有一个组织背书即可。

在现实世界的区块链商业网络中，多个组织会在交易提交到区块链之前对交易背书，所以默认背书策略不适合。相反，当启动一个商业网络时，你可以指定一个定制的背书策略。

你可以在Hyperledger Fabric文档，[背书策略](https://hyperledgercn.github.io/hyperledgerDocs/endorsement-policies_zh/)中，找到更多关于背书策略的信息。

*请注意，用于商业网络的背书策略必须是Hyperledger Fabric Node.js SDK使用的JSON格式。这是与Hyperledger Fabric CLI使用的简单背书策略不同的一种格式，你可以在Hyperledger Fabric文档中看到。*

创建一个叫`endorsement-policy.json`的背书策略，包含下列内容并保存到磁盘。你会在后续步骤中使用这个文件，所以记住存放位置！
```json
{
    "identities": [
        {
            "role": {
                "name": "member",
                "mspId": "Org1MSP"
            }
        },
        {
            "role": {
                "name": "member",
                "mspId": "Org2MSP"
            }
        }
    ],
    "policy": {
        "2-of": [
            {
                "signed-by": 0
            },
            {
                "signed-by": 1
            }
        ]
    }
}
```
你刚创建的背书策略指出，在商业网络中，交易在提交到区块链之前必须同时得到`Org1`和`Org2`的背书。如果`Org1`或`Org2`没有对交易背书，或不同意交易结果，那么交易会被商业网络拒绝。

### ================步骤十四：理解和选择商业网络管理员================

当一个商业网络启动时，商业网络必须配置一组初始参与者。这些参与者负责商业网络的引导和加载其他参与者进入商业网络。在Hyperledger Composer中，我们叫这些初始参与者为商业网络管理员。

在我们的商业网络中，组织`Org1`和`Org2`具有相同权限。每个组织都为商业网络提供了一个商业网络管理员，这些商业网络管理员会加载他们组织中的其他参与者进入商业网络。`Org1`的商业网络管理员是Alice，而`Org2`的商业网络管理员是Bob。

当商业网络启动时，所有商业网络管理员的证书(身份的公开部分)都必须传送到组织，来执行启动商业网络的命令。当商业网络启动后，所有的商业网络管理员可以使用他们的身份与商业网络进行交互。

你可以在[部署商业网络](https://hyperledger.github.io/composer/unstable/business-network/bnd-deploy.html)中找到更多关于商业网络管理员的信息。

### ----------------步骤十五：为Org1获取商业网络管理员证书----------------

运行`composer identity request`命令获取Alice的证书，以便`Org1`作为商业网络管理员使用：
```bash
composer identity request -c PeerAdmin@byfn-network-org1-only -u admin -s adminpw -d alice
```
这个命令的`-u admin`和`-s adminpw`选项需要与Hyperledger Fabric CA (Certificate Authority)注册的默认用户一致。

这个证书会被保存到当前目录的`alice`子目录中。创建了三个证书文件，但只有两个重要。它们是`admin-pub.pem`，证书(包括公钥)，和`admin-priv.pem`，私钥。只有文件`admin-pub.pem`适合与其他组织共享。文件`admin-priv.pem`必须秘密保存，它可以代表组织签署交易。

### ....................步骤十六：为Org2获取商业网络管理员证书....................

运行`composer identity request`命令获取Bob的证书，以便`Org2`作为商业网络管理员使用：
```bash
composer identity request -c PeerAdmin@byfn-network-org2-only -u admin -s adminpw -d bob
```
这个命令的`-u admin`和`-s adminpw`选项需要与Hyperledger Fabric CA (Certificate Authority)注册的默认用户一致。

这个证书会被保存到当前目录的`bob`子目录中。创建了三个证书文件，但只有两个重要。它们是`admin-pub.pem`，证书(包括公钥)，和`admin-priv.pem`，私钥。只有文件`admin-pub.pem`适合与其他组织共享。文件`admin-priv.pem`必须秘密保存，它可以代表组织签署交易。

### ----------------步骤十七：启动商业网络----------------

运行`composer network start`命令启动商业网络。只有`Org`需要执行这个操作。这个命令使用步骤十三创建的`endorsement-policy.json`文件，以及步骤十五(Alice)和步骤十六(Bob)创建的`admin-pub.pem`文件，你必须确保这个命令能访问这些文件：
```bash
composer network start -c PeerAdmin@byfn-network-org1 -a tutorial-network@0.0.1.bna -o endorsementPolicyFile=endorsement-policy.json -A alice -C alice/admin-pub.pem -A bob -C bob/admin-pub.pem
```
当这个命令一结束，商业网络就启动了。Alice和Bob就可以访问商业网络，开始建立商业网络，并从各自的组织中加载其他参与者。然而，无论Alice还是Bob都必须先使用前面步骤创建的证书来创建商业网络卡片，之后才能访问商业网络。

### ----------------步骤十八：为了Org1访问商业网络而创建商业网络卡片----------------

为了`Org1`的商业网络管理员Alice访问商业网络，运行`composer card create`命令创建一个商业网络卡片：
```bash
composer card create -p connection-org1.json -u alice -n tutorial-network -c alice/admin-pub.pem -k alice/admin-priv.pem
```
运行`composer card import`命令导入你刚创建的商业网络卡片：
```bash
composer card import -f alice@tutorial-network.card
```
运行`composer network ping`命令测试到区块链商业网络的连接：
```bash
composer network ping -c alice@tutorial-network
```
如果名称执行成功，你可以在命令输出中看到完全限定参与者身份`org.hyperledger.composer.system.NetworkAdmin#alice`。你现在可以使用这个商业网络卡片去与区块链商业网络交互，并加载你组织中的其他参与者。

### ....................步骤十九：为了Org1访问商业网络而创建商业网络卡片....................

为了`Org2`的商业网络管理员Bob访问商业网络，运行`composer card create`命令创建一个商业网络卡片：
```bash
composer card create -p connection-org2.json -u bob -n tutorial-network -c bob/admin-pub.pem -k bob/admin-priv.pem
```
运行`composer card import`命令导入你刚创建的商业网络卡片：
```bash
composer card import -f bob@tutorial-network.card
```
运行`composer network ping`命令测试到区块链商业网络的连接：
```bash
composer network ping -c bob@tutorial-network
```
如果名称执行成功，你可以在命令输出中看到完全限定参与者身份`org.hyperledger.composer.system.NetworkAdmin#bob`。你现在可以使用这个商业网络卡片去与区块链商业网络交互，并加载你组织中的其他参与者。

### ================结论================
在这个教程中你已经看到了怎样使用所有必要信息配置Hyperledger Composer去连接到跨组织的Hyperledger Fabric网络，以及在一个跨组织的Hyperledger Fabric网络中怎样部署一个区块链商业网络。

## 实践：部署Hyperledger Composer网络到多组织Fabric
```bash
cd /opt/fabric-samples/first-network
cp COMPOSE_FILE=docker-compose-e2e.yaml COMPOSE_FILE=docker-compose-e2e2.yaml
cp byfn.sh byfn2.sh
```
编辑byfn2.sh:
```bash
export ORG1_ADMIN_KEY=crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/*_sk
export ORG2_ADMIN_KEY=crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/keystore/*_sk
export ORG1_ADMIN_CRT=crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem
export ORG2_ADMIN_CRT=crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/Admin@org2.example.com-cert.pem
(略)
#COMPOSE_FILE=docker-compose-cli.yaml
COMPOSE_FILE=docker-compose-e2e2.yaml
(略)
```
编辑docker-compose-e2e2.yaml：
```yaml
ca0:
    container_name: ca.org1.example.com
ca1:
    container_name: ca.org2.example.com
  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME} ${DELAY} ${LANG}; sleep $TIMEOUT'
    volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/chaincode
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
      - orderer.example.com
      - peer0.org1.example.com
      - peer1.org1.example.com
      - peer0.org2.example.com
      - peer1.org2.example.com
    networks:
      - byfn
```
清理docker容器：
```bash
docker rm -f $(docker ps -aq)
./byfn2.sh
```
connection-org1.json:
```json
{
    "name": "byfn-network-org1",
    "type": "hlfv1",
    "mspID": "Org1MSP",
    "peers": [
        {
            "requestURL": "grpcs://localhost:7051",
            "eventURL": "grpcs://localhost:7053",
            "cert": "crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt",
            "hostnameOverride": "peer0.org1.example.com"
        },
        {
            "requestURL": "grpcs://localhost:8051",
            "eventURL": "grpcs://localhost:8053",
            "cert": "crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt",
            "hostnameOverride": "peer1.org1.example.com"
        },
        {
            "requestURL": "grpcs://localhost:9051",
            "cert": "crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt",
            "hostnameOverride": "peer0.org2.example.com"
        },
        {
            "requestURL": "grpcs://localhost:10051",
            "cert": "crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt",
            "hostnameOverride": "peer1.org2.example.com"
        }
    ],
    "ca": {
        "url": "https://localhost:7054",
        "name": "ca-org1",
        "cert": "crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt",
        "hostnameOverride": "ca.org1.example.com"
    },
    "orderers": [
        {
            "url" : "grpcs://localhost:7050",
            "cert": "crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt",
            "hostnameOverride": "orderer.example.com"
        }
    ],
    "channel": "mychannel",
    "timeout": 300
}
```
connection-org1-only.json:
```json
{
    "name": "byfn-network-org1-only",
    "type": "hlfv1",
    "mspID": "Org1MSP",
    "peers": [
        {
            "requestURL": "grpcs://localhost:7051",
            "eventURL": "grpcs://localhost:7053",
            "cert": "crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt",
            "hostnameOverride": "peer0.org1.example.com"
        },
        {
            "requestURL": "grpcs://localhost:8051",
            "eventURL": "grpcs://localhost:8053",
            "cert": "crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt",
            "hostnameOverride": "peer1.org1.example.com"
        }
    ],
    "ca": {
        "url": "https://localhost:7054",
        "name": "ca-org1",
        "cert": "crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt",
        "hostnameOverride": "ca.org1.example.com"
    },
    "orderers": [
        {
            "url" : "grpcs://localhost:7050",
            "cert": "crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt",
            "hostnameOverride": "orderer.example.com"
        }
    ],
    "channel": "mychannel",
    "timeout": 300
}
```
connection-org2.json:
```json
{
    "name": "byfn-network-org2",
    "type": "hlfv1",
    "mspID": "Org2MSP",
    "peers": [
        {
            "requestURL": "grpcs://localhost:9051",
            "eventURL": "grpcs://localhost:9053",
            "cert": "crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt",
            "hostnameOverride": "peer0.org2.example.com"
        },
        {
            "requestURL": "grpcs://localhost:10051",
            "eventURL": "grpcs://localhost:10053",
            "cert": "crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt",
            "hostnameOverride": "peer1.org2.example.com"
        },
        {
            "requestURL": "grpcs://localhost:7051",
            "cert": "crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt",
            "hostnameOverride": "peer0.org1.example.com"
        },
        {
            "requestURL": "grpcs://localhost:8051",
            "cert": "crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt",
            "hostnameOverride": "peer1.org1.example.com"
        }
    ],
    "ca": {
        "url": "https://localhost:8054",
        "name": "ca-org2",
        "cert": "crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt",
        "hostnameOverride": "ca.org2.example.com"
    },
    "orderers": [
        {
            "url" : "grpcs://localhost:7050",
            "cert": "crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt",
            "hostnameOverride": "orderer.example.com"
        }
    ],
    "channel": "mychannel",
    "timeout": 300
}
```
connection-org2-only.json:
```json
{
    "name": "byfn-network-org2-only",
    "type": "hlfv1",
    "mspID": "Org2MSP",
    "peers": [
        {
            "requestURL": "grpcs://localhost:9051",
            "eventURL": "grpcs://localhost:9053",
            "cert": "crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt",
            "hostnameOverride": "peer0.org2.example.com"
        },
        {
            "requestURL": "grpcs://localhost:10051",
            "eventURL": "grpcs://localhost:10053",
            "cert": "crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt",
            "hostnameOverride": "peer1.org2.example.com"
        }
    ],
    "ca": {
        "url": "https://localhost:8054",
        "name": "ca-org2",
        "cert": "crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt",
        "hostnameOverride": "ca.org2.example.com"
    },
    "orderers": [
        {
            "url" : "grpcs://localhost:7050",
            "cert": "crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt",
            "hostnameOverride": "orderer.example.com"
        }
    ],
    "channel": "mychannel",
    "timeout": 300
}
```
第七步和第八步：
```bash
sudo composer card create -p connection-org1-only.json -u PeerAdmin -c $ORG1_ADMIN_CRT -k $ORG1_ADMIN_KEY -r PeerAdmin -r ChannelAdmin
sudo composer card create -p connection-org1.json -u PeerAdmin -c $ORG1_ADMIN_CRT -k $ORG1_ADMIN_KEY -r PeerAdmin -r ChannelAdmin
sudo composer card create -p connection-org2-only.json -u PeerAdmin -c $ORG2_ADMIN_CRT -k $ORG2_ADMIN_KEY -r PeerAdmin -r ChannelAdmin
sudo composer card create -p connection-org2.json -u PeerAdmin -c $ORG2_ADMIN_CRT -k $ORG2_ADMIN_KEY -r PeerAdmin -r ChannelAdmin
```
剩下的9到16步按完全按原教程操作就可以。执行到17步的时候，提示`tutorial-network@0.0.1.bna`不存在，执行不下去了。

### 问题
Error: Failed to load connector module "composer-connector-hlfv1" for connection type "hlfv1". Cannot find module '/usr/local/lib/node_modules/composer-cli/node_modules/grpc/src/node/extension_binary/node-v57-linux-x64/grpc_node.node'  
解决办法：  
ubuntu@ubuntu-xenial:/usr/local/lib/node_modules/composer-cli$ npm rebuild --unsafe-perm  


## 为单个组织部署Hyperledger Composer区块链网络到Hyperledger Fabric
[原文](https://hyperledger.github.io/composer/unstable/tutorials/deploy-to-fabric-single-org)

在[开发环境](https://hyperledger.github.io/composer/unstable/installing/development-tools.html)中，为你创建了一个简单Hyperledger Fabric网络(`fabric-dev-servers`)，以及为部署区块链商业网络所需的所有Hyperledger Composer配置。

本教程将演示为单个组织将区块链商业网络部署到的Hyperledger Fabric实例，管理员需要执行的步骤，包括如何生成必需的Hyperledger Composer配置。随后的教程将演示如何将区块链商业网络部署到多个组织的Hyperledger Fabric实例。

### 先决条件
 1. 在你继续前，确保你已经完成了[安装一个开发环境](https://hyperledger.github.io/composer/unstable/installing/development-tools.html)中步骤。  

### 步骤一：启动Hyperledger Fabric网络
为了遵循本教程，必须必须启动Hyperledger Fabric网络。你可以使用开发环境中提供的简单Hyperledger Fabric网络，也可以使用你根据Hyperledger Fabric文档构建的自己的Hyperledger Fabric网络。

本教程将假设你使用开发环境中提供的简单Hyperledger Fabric网络。如果你使用自己的Hyperledger Fabric网络，则必须在下面详细介绍的配置和你自己的配置之间进行映射。

 1. 通过运行以下命令启动一个干净的Hyperledger Fabric：
```bash
cd ~/fabric-tools
./stopFabric.sh
./teardownFabric.sh
./downloadFabric.sh
export FABRIC_VERSION=hlfv11        
./startFabric.sh
```
 2. 删除你的钱包中的所有商业网络卡片。忽略无法找到商业网络卡片的错误，这是安全的：
```
composer card delete -n PeerAdmin@fabric-network
composer card delete -n admin@tutorial-network
```

### 步骤二：探索Hyperledger Fabric网络
这一步将探索刚刚启动的Hyperledger Fabric网络，以便了解它是如何配置的，以及它由哪些组件组成。在后续步骤中配置Hyperledger Composer时，你将使用本节中的所有信息。

#### 配置文件
在开发环境中提供的简单Hyperledger Fabric网络，已经使用Hyperledger Fabric配置工具`cryptogen`和`configtxgen`来配置。

`cryptogen`的配置存储在下面的文件中：
```
~/fabric-tools/fabric-scripts/hlfv11/composer/crypto-config.yaml
```
`configtxgen`的配置存储在下面的文件中：
```
~/fabric-tools/fabric-scripts/hlfv11/composer/configtx.yaml
```
你可以通过阅读Hyperledger Fabric文档来找到有关这些配置工具的更多信息，它们做什么和如何使用它们。

#### 组织
简单的Hyperledger Fabric网络由一个叫做`Org1`的组织构成。该组织使用域名`org1.example.com`。此外，该组织的成员服务提供商(MSP)被称为`Org1MSP`。在本教程中，你将部署只有组织`Org1`才能与之交互的区块链商业网络。

#### 网络组件
Hyperledger Fabric网络由多个组件组成：  
- 一个Org1的peer节点，名叫peer0.org1.example.com  
  - 请求端口是7051  
  - 事件hub端口是7053  
- 一个Org1的CA(Certificate Authority)，名叫ca.org1.example.com  
  - CA端口是7054  
- 一个排序节点，名叫orderer.example.com  
  - 排序端口是7050  

Hyperledger Fabric网络组件在Docker容器中运行。在Docker容器中运行Hyperledger Composer时，可以使用上面的名称（例如，`peer0.org1.example.com`）与Hyperledger Fabric网络进行交互。

本教程将在Docker主机上运行Hyperledger Composer命令，而不是在Docker网络内运行。这意味着Hyperledger Composer命令必须使用`localhost`主机名和暴露的容器端口与Hyperledger Fabric网络进行交互。

#### 用户
组织`Org1`使用名为`Admin@org1.example.com`的用户进行配置。此用户是管理员。组织的管理员有权将区块链商业网络的代码安装到其组织的peer，并且还可以具有启动区块链商业网络的权限，具体取决于配置。在本教程中，你将以`Admin@org1.example.com`用户身份部署区块链商业网络。

用户`Admin@org1.example.com`有一组存储在下列目录中的证书和私钥文件：
```
~/fabric-tools/fabric-scripts/hlfv11/composer/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
```
你稍后将使用其中一些文件与Hyperledger Fabric网络进行交互。

除了管理员之外，`Org1`的CA（证书颁发机构）使用一个默认用户进行配置。此默认用户登记ID是`admin`，登记密码是`adminpw`。但是，这个用户无权部署区块链商业网络。

#### 信道
最后，创建了一个名为`channel`的信道。peer节点`peer0.org1.example.com`已加入此通道。你只能将Hyperledger Composer区块链商业网络部署到已存在的信道，但您可以按照Hyperledger Fabric文档创建其他信道。

### 步骤三：建立连接profile
连接profile指定了定位和连接Hyperledger Fabric网络所需的所有信息，例如所有Hyperledger Fabric网络组件的主机名和端口。在此步骤中，你将为Hyperledger Composer创建一个连接profile，用于连接到Hyperledger Fabric网络。

 1. 创建一个名为的`connection.json`的连接profile。

 2. 通过将以下三行添加到`connection.json`顶部来提供连接profile的`name`和`type`属性：
```json
{
  "name": "fabric-network",
  "type": "hlfv1",
```
连接profile中的`name`属性给出Hyperledger Fabric网络的名称，所以稍后可以引用它。在刚创建的连接profile，名称是`fabric-network`。你可以使用喜欢的名字为Hyperledger Fabric网络命名。

Hyperledger Composer被设计为兼容不同类型的区块链网络。目前，仅支持Hyperledger Fabric v1.0，但你必须指定要使用的区块链网络的类型。Hyperledger Fabric v1.0的类型是`hlfv1`。

 3. 必须指定用于连接到Hyperledger Fabric网络的MSP的名称：
```json
"mspID": "Org1MSP",
```
我们正在以`Org1`连接，`Org1`的MSP被称为`Org1MSP`。

我们必须指定要连接的Hyperledger Fabric网络中所有peer节点的主机名和端口。
```json
"peers": [
    {
        "requestURL": "grpc://localhost:7051",
        "eventURL": "grpc://localhost:7053"
    }
],
```
在这里，我们已经指定了单个peer节点`peer0.org1.example.com`（使用主机名`localhost`），请求端口7051和事件hub端口7053。

`peers`数组可以包含多个peer节点。如果您有多个peer节点，则应将其全部添加到peers数组中，以便Hyperledger Composer可以与它们进行交互。

区块链商业网络将部署到所有指定的peer节点。一旦区块链商业网络部署完毕，指定的peer节点将用于查询区块链商业网络、为交易背书和订阅事件。

 5. 我们必须在Hyperledger Fabric网络中指定证书颁发机构（CA）的主机名和端口，用于登记现有用户和注册新用户。
```json
"ca": {
    "url": "http://localhost:7054",
    "name": "ca.org1.example.com"
},
```
这里我们指定了一个CA`ca.or1.example.com`（使用主机名`localhost`）和CA端口7054。

 6. 我们必须指定要连接的Hyperledger Fabric中所有排序节点的主机名和端口。
```json
"orderers": [
    {
        "url" : "grpc://localhost:7050"
    }
],
```
在这里，我们已经指定了一个排序节点`orderer.example.com`（使用主机名`localhost`）和排序端口7050。

该`orderers`数组可以包含多个排序节点。如果你有多个排序节点，则应将其全部添加到`orderers`数组中，以便Hyperledger Composer可以与其进行交互。

 7. 我们必须指定一个现有信道的名称。我们会把区块链商业网络部署到信道`composerchannel`中。
```json
"channel": "composerchannel",
```
 8. 最后，我们可以选择指定与区块链商业网络进行交互时，对交易背书的超时时间。
```json
  "timeout": 300
}
```
在这里，我们已经指定了300秒的超时时间。如果一个交易需要超过300秒的时间来背书，那么将会抛出一个超时错误。

 9. 保存你的更改到`connection.json`。完成的连接profile应该如下所示：
```json
{
  "name": "fabric-network",
  "type": "hlfv1",
  "mspID": "Org1MSP",
  "peers": [
      {
          "requestURL": "grpc://localhost:7051",
          "eventURL": "grpc://localhost:7053"
      }
  ],
  "ca": {
      "url": "http://localhost:7054",
      "name": "ca.org1.example.com"
  },
  "orderers": [
      {
          "url" : "grpc://localhost:7050"
      }
  ],
  "channel": "composerchannel",
  "timeout": 300
}
```

### 步骤四：定位Hyperledger Fabric管理员的证书和私钥

要将区块链商业网络部署到Hyperledger Fabric网络，我们必须将自己标识为具有执行此操作权限的管理员。在这一步骤中，你可以定位证明自己为管理员需要的文件。

我们的Hyperledger Fabric网络的管理员是一名叫`Admin@org1.example.com`的用户。该用户的证书和私钥文件存储在以下目录中：


~/fabric-tools/fabric-scripts/hlfv11/composer/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
您必须首先找到该用户的证书文件。证书是身份的公共部分。证书文件可以在signcerts子目录中找到并命名Admin@org1.example.com-cert.pem。如果你看这个文件的内容，那么你会发现一个类似于下面的PEM编码证书：


-----BEGIN CERTIFICATE-----
MIICGjCCAcCgAwIBAgIRANuOnVN+yd/BGyoX7ioEklQwCgYIKoZIzj0EAwIwczEL
MAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG
cmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHDAaBgNVBAMTE2Nh
Lm9yZzEuZXhhbXBsZS5jb20wHhcNMTcwNjI2MTI0OTI2WhcNMjcwNjI0MTI0OTI2
WjBbMQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMN
U2FuIEZyYW5jaXNjbzEfMB0GA1UEAwwWQWRtaW5Ab3JnMS5leGFtcGxlLmNvbTBZ
MBMGByqGSM49AgEGCCqGSM49AwEHA0IABGu8KxBQ1GkxSTMVoLv7NXiYKWj5t6Dh
WRTJBHnLkWV7lRUfYaKAKFadSii5M7Z7ZpwD8NS7IsMdPR6Z4EyGgwKjTTBLMA4G
A1UdDwEB/wQEAwIHgDAMBgNVHRMBAf8EAjAAMCsGA1UdIwQkMCKAIBmrZau7BIB9
rRLkwKmqpmSecIaOOr0CF6Mi2J5H4aauMAoGCCqGSM49BAMCA0gAMEUCIQC4sKQ6
CEgqbTYe48az95W9/hnZ+7DI5eSnWUwV9vCd/gIgS5K6omNJydoFoEpaEIwM97uS
XVMHPa0iyC497vdNURA=
-----END CERTIFICATE-----
接下来，您必须找到该用户的私钥文件。私钥用于以这种身份对交易进行签名。私钥文件可以在keystore子目录中找到。私钥文件的名称是一个很长的十六进制字符串，_sk例如后缀为114aab0e76bf0c78308f89efc4b8c9423e31568da0c340ca187a9b17aa9a4457_sk。每次生成配置名称都会更改。如果你看看这个文件的内容，那么你会发现一个PEM编码的私钥类似于以下内容：


-----BEGIN PRIVATE KEY-----
MIGHAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBG0wawIBAQQg00IwLLBKoi/9ikb6
ZOAV0S1XeNGWllvlFDeczRKQn2uhRANCAARrvCsQUNRpMUkzFaC7+zV4mClo+beg
4VkUyQR5y5Fle5UVH2GigChWnUoouTO2e2acA/DUuyLDHT0emeBMhoMC
-----END PRIVATE KEY-----
记住这两个文件的路径，或将它们到与connection.json上一步创建的连接配置文件相同的目录中。您将在下一步中需要这些文件。

第五步：为Hyperledger Fabric管理员创建一张商业网卡
商业网卡包含连接区块链商业网络和底层Hyperledger Fabric网络所需的全部信息。此信息包括步骤三中创建的连接配置文件，以及步骤四中的管理员的证书和专用密钥。

在这一步中，您将创建一个商业网卡供管理员用来将区块链商业网络部署到Hyperledger Fabric网络。

运行该composer card create命令创建一个商业网卡。您必须指定您在以前的步骤中创建或找到的所有三个文件的路径：


composer card create -p connection.json -u PeerAdmin -c Admin@org1.example.com-cert.pem -k 114aab0e76bf0c78308f89efc4b8c9423e31568da0c340ca187a9b17aa9a4457_sk -r PeerAdmin -r ChannelAdmin
被调用的商业网卡文件PeerAdmin@fabric-network.card将被写入当前目录。我们来探讨一下我们传递给composer card create命令的选项。


-p connection.json
这是我们在第三步创建的连接配置文件的路径。


-u PeerAdmin
这是我们用来引用管理员用户的名字。而不是Admin@org1.example.com到处使用，这是相当长的类型，我们给了一个名字，PeerAdmin所以我们可以很容易地引用这个用户。


-c Admin@org1.example.com-cert.pem
这是Admin@org1.example.com我们在第四步找到的用户证书文件的路径。


-k 114aab0e76bf0c78308f89efc4b8c9423e31568da0c340ca187a9b17aa9a4457_sk
这是Admin@org1.example.com我们在第四步找到的用户的私钥文件的路径。


-r PeerAdmin -r ChannelAdmin
在这里，我们指定用户拥有哪些角色。此信息是必需的，以便Hyperledger Composer知道哪些用户能够执行哪些操作。用户Admin@org1.example.com是Hyperledger Fabric网络的管理员，具有角色PeerAdmin（能够安装链式代码）和ChannelAdmin（实例化链式代码的能力）。

第六步：为Hyperledger Fabric管理员导入商业网卡
Hyperledger Composer只能使用放置在钱包中的商业网卡。钱包是包含商业网卡的文件系统上的目录。在这一步中，您将将步骤五中创建的商业网卡导入到钱包中，以便您可以在后续步骤中使用商业网卡。

运行composer card import命令将商业网卡导入钱包：


composer card import -f PeerAdmin@fabric-network.card
我们来探讨一下我们传递给composer card import命令的选项。


-f PeerAdmin@fabric-network.card
这是我们在第五步创建的商业网卡文件的路径。

您现在可以通过指定名称来使用此商业网卡PeerAdmin@fabric-network。您现在已经准备好将区块链商业网络部署到Hyperledger Fabric网络。

我们将部署tutorial-network按照开发者教程创建的区块链商业网络。

第七步：将Hyperledger Composer运行时安装到Hyperledger Fabric对等节点上
Hyperledger Composer包含一个名为Hyperledger Composer运行时的组件，该组件提供用于托管和支持商业网络归档的所有功能，例如数据验证，错误处理，事务处理函数执行和访问控制。在Hyperledger Fabric术语中，Hyperledger Composer运行时是一个标准的链式代码。

在这一步中，您将在所有Hyperledger Fabric对等节点上安装Hyperledger Composer运行时。在Hyperledger Fabric中，这是一个chaincode安装操作。

运行以下composer runtime install命令将Hyperledger Composer运行时安装到您在步骤三中创建的连接配置文件中指定的所有Hyperledger Fabric对等节点上：


composer runtime install -c PeerAdmin@fabric-network -n tutorial-network
我们来探讨一下我们传递给composer runtime install命令的选项。


-c PeerAdmin@fabric-network
这是我们在步骤6中导入到钱包中的商业网卡的名称。


-n tutorial-network
您必须为每个区块链商业网络安装Hyperledger Composer运行时的副本，并指定区块链商业网络的名称。在这里，我们指定我们正在部署的区块链商业网络的名称tutorial-network。

第八步：启动区块链商业网络
在这一步中，您将启动区块链商业网络。在Hyperledger Fabric术语中，这是链式代码实例化操作。

运行composer network start命令启动区块链商业网络：


composer network start -c PeerAdmin@fabric-network -a tutorial-network.bna -A admin -S adminpw
我们来探讨一下我们传递给composer network start命令的选项。


-c PeerAdmin@fabric-network
这是我们在步骤6中导入到钱包中的商业网卡的名称。


-a tutorial-network.bna
这是包含我们区块链商业网络的商业网络定义的商业网络存档的路径tutorial-network。


-A admin
在部署区块链商业网络时，您必须创建至少一个将成为区块链商业网络管理员的参与者。此参与者负责将其他参与者加入区块链商业网络。在这里，我们正在指定我们要创建一个叫做的区块链商业网络管理员admin。


-S adminpw
这指定我们的区块链商业网络管理员admin将使用注册adminpw密钥从CA（证书颁发机构）请求证书和私钥。指定此选项时，为商业网络管理员指定的名称必须是已经向CA注册的用户的现有注册ID。

现在我们的区块链商业网络已经启动，我们可以使用admin@tutorial-network.card创建的商业网卡文件与它进行交互。

第九步：为商业网络管理员导入商业网卡
运行composer card import命令将商业网卡导入钱包：


composer card import -f admin@tutorial-network.card
您现在可以通过指定名称来使用此商业网卡admin@tutorial-network。现在您已经准备好与正在运行的区块链商业网络进行交互了！

第十步：测试与区块链商业网络的连接
运行composer network ping命令测试与区块链商业网络的连接：


composer network ping -c admin@tutorial-network
结论
在本教程中，您已经了解了如何配置Hyperledger Composer以及连接到Hyperledger Fabric网络所需的所有信息，以及如何将区块链商业网络部署到该Hyperledger Fabric网络。

如果您使用了开发环境中提供的简单的Hyperledger Fabric网络，那么为什么不尝试按照Hyperledger Fabric文档构建自己的Hyperledger Fabric网络，并查看是否可以成功部署区块链商业网络？