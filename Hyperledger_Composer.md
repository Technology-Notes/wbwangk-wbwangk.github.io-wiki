Composer中文文档移到了[这里](https://wbwangk.github.io/ComposerDocs)  

# 笔记
### 安装开发环境
[安装开发环境](https://wbwangk.github.io/ComposerDocs/installing_development-tools/)  

在`~/fabric-tools`目录下下载了一些脚本和其他文件。其中，`createPeerAdminCard.sh`用于为peer管理员创建一个业务网络卡片。下面是这个脚本中创建PeerAdmin业务网络卡片的逻辑：
```bash
cat << EOF > /tmp/.connection.json
{
    "name": "hlfv1",
    "type": "hlfv1",
    "orderers": [
       { "url" : "grpc://localhost:7050" }
    ],
    "ca": { "url": "http://localhost:7054", "name": "ca.org1.example.com"},
    "peers": [
        {
            "requestURL": "grpc://localhost:7051",
            "eventURL": "grpc://localhost:7053"
        }
    ],
    "channel": "composerchannel",
    "mspID": "Org1MSP",
    "timeout": 300
}
EOF

PRIVATE_KEY="${DIR}"/composer/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/114aab0e76bf0c78308f89efc4b8c9423e31568da0c340ca187a9b17aa9a4457_sk
CERT="${DIR}"/composer/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem

composer card create -p /tmp/.connection.json -u PeerAdmin -c "${CERT}" -k "${PRIVATE_KEY}" -r PeerAdmin -r ChannelAdmin --file /tmp/PeerAdmin@hlfv1.card
composer card import --file /tmp/PeerAdmin@hlfv1.card
```
这个脚本，创建了连接配置文件`connection.json`，用定位了Fabric组织管理员`Admin@org1.example.com`证书和私钥，利用公私钥创建了一个网页网络卡片`PeerAdmin@hlfv1.card`，并导入到网络中。

### 为单个组织部署Hyperledger Composer区块链网络到Hyperledger Fabric
[为单个组织部署Hyperledger Composer区块链网络到Hyperledger Fabric](https://wbwangk.github.io/ComposerDocs/tutorials_deploy-to-fabric-single-org/)  

使用`Admin@org1.example.com`(组织管理员)来创建业务网络卡片，创建了文件`PeerAdmin@fabric-network.card`，并导入Composer钱包。  
安装Composer运行时：
```
composer runtime install -c PeerAdmin@fabric-network -n tutorial-network
```
业务网络名称`tutorial-network`在安装运行时指定，后面会引用。  
可以尝试创建`tutorial-network2`。可以把**业务网络视为通道的细分？**  
启动业务网络：
```
composer network start -c PeerAdmin@fabric-network -a tutorial-network.bna -A admin -S adminpw
```
`-A`和`-S`参数引起了`admin@tutorial-network.card`的创建，这是业务网络管理员。  
导入业务网络管理员卡片：
```
composer card import -f admin@tutorial-network.card
```

### 部署Hyperledger Composer网络到多组织Fabric
[部署Hyperledger Composer网络到多组织Fabric](https://wbwangk.github.io/ComposerDocs/tutorials_deploy-to-fabric-multi-org/)  

安装Composer







# 实践：部署Hyperledger Composer网络到多组织Fabric
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

