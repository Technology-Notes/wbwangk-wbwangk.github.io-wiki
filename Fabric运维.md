[Hyperledger](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger)

## 成员服务提供者(MSP)
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/msp.html)   
[中文文档](https://wbwangk.github.io/hyperledgerDocs/msp_zh/)  

## 通道配置(configtx)
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/configtx.html)  
[中文文档](https://wbwangk.github.io/hyperledgerDocs/configtx_zh/)  

## 通道配置(configtxgen)
[中文文档](https://wbwangk.github.io/hyperledgerDocs/configtxgen_zh/)  

## 用configtxlator重新配置
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/configtxlator.html)  
[中文原文](https://wbwangk.github.io/hyperledgerDocs/configtxlator_zh/)  

## 背书策略
[原文](http://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html)  
[中文原文](https://wbwangk.github.io/hyperledgerDocs/endorsement-policies_zh/)  



# 架构
## 架构解释
[中文文档](https://wbwangk.github.io/hyperledgerDocs/arch-deep-dive_zh/)  


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
$ CORE_PEER_LOCALMSPID="Org2MSP" $ CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
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