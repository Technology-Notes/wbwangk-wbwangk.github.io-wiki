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
```bash
composer network start -c PeerAdmin@fabric-network -a tutorial-network.bna -A admin -S adminpw
```
`-A`和`-S`参数引起了文件`admin@tutorial-network.card`的创建，这是业务网络管理员。  
导入业务网络管理员卡片：
```bash
composer card import -f admin@tutorial-network.card
```

### 部署Hyperledger Composer网络到多组织Fabric
[部署Hyperledger Composer网络到多组织Fabric](https://wbwangk.github.io/ComposerDocs/tutorials_deploy-to-fabric-multi-org/)  

安装Composer运行时后，导出的业务网络管理员（而不是单组织的在启动业务网络时生成）：
```
composer identity request -c PeerAdmin@byfn-network-org1-only -u admin -s adminpw -d alice
```
然后，又导出了org2的业务网络管理员bob。  
启动业务网络：
```bash
composer network start -c PeerAdmin@byfn-network-org1 -a tutorial-network@0.0.1.bna -o endorsementPolicyFile=endorsement-policy.json -A alice -C alice/admin-pub.pem -A bob -C bob/admin-pub.pem
```
两个组织的业务网络，启动一次就可以了。也是由于只能启动一次，不能在启动时同时为两个组织的业务网络管理员创建卡片了。    
然后是，创建业务网络管理员卡片：
```
composer card create -p connection-org1.json -u alice -n tutorial-network -c alice/admin-pub.pem -k alice/admin-priv.pem
```
之后是导入卡片和ping业务网络了。  

## Hyperledger Composer Playground
[本地安装文档](https://wbwangk.github.io/ComposerDocs/installing_using-playground-locally/)、[本地环境](E:\vagrant10\ambari-vagrant\fabric\devenv)  
```
curl -sSL https://hyperledger.github.io/composer/install-hlfv1.sh | bash
```
以上脚本会下载1.0.4本版的Fabric Docker镜像、启动Fabric、拉取Playground镜像并启动。看上去该脚本与fabric-tools的功能类似，启动了一个单组织单节点的Fabric，只是多了一个`hyperledger/composer-playground`的容器。  
由于VM已经做了8080端口的NAT映射，可以直接在windows下用浏览器通过地址`http://localhost:8080`打开本地Playground的Web界面。  
安装Composer运行时：
```bash
cd :~/composer/tutorial-network
composer runtime install --card PeerAdmin@hlfv1 --businessNetworkName tutorial-network
```
部署业务网络：
```
composer network start --card PeerAdmin@hlfv1 --networkAdmin admin --networkAdminEnrollSecret adminpw --archiveFile tutorial-network@0.0.1.bna --file networkadmin.card
```
导入业务网络管理员时报告该身份已经存在：
```
composer card import --file networkadmin.card
Error: Card already exists: admin@tutorial-network
```
ping一下业务网络，确保网络已经正确运行：
```
composer network ping --card admin@tutorial-network
```
当前目录下`networkadmin.card`是一个业务网络管理员卡片，需要导入到Playground，然后Playground才能联上Fabric实例。  
由于本VM是用vagrant搭建的，`/vagrant`目录是个windows共享目录，直接把`networkadmin.card`复制到`/vagrant`目录，就可以在windows访问这个文件了，然后再导入到playground。  

在本地Playground环境的`http://localhost:8080/login`网页上点击`Import Business Network Card`按钮把上面的`.card`文件导入。
点击新建的业务网络`tutorial-network`的"Connect now"按钮，提示：
```
[Error: connect ECONNREFUSED 127.0.0.1:7054]
```
这是因为playground运行在容器里，其访问fabric使用的域名，如ca.org1.example.com。这一点可以进入composer playground容器(容器名是composer)，`telnet 127.0.0.1 7054`，会发现不通，但`telnet ca.org1.example.com 7054`是通的。  

之前运行composer命令行是在容器的宿主机中，所以连接配置文件的地址都是127.0.0.1。导入到playground中的card地址不对。
制作一个`connection2.json`，把域名改成下面的样子:
```json
{
  "name": "fabric-network",
  "type": "hlfv1",
  "mspID": "Org1MSP",
  "peers": [
      {
          "requestURL": "grpc://peer0.org1.example.com:7051",
          "eventURL": "grpc://peer0.org1.example.com:7053"
      }
  ],
  "ca": {
      "url": "http://ca.org1.example.com:7054",
      "name": "ca.org1.example.com"
  },
  "orderers": [
      {
          "url" : "grpc://orderer.example.com:7050"
      }
  ],
  "channel": "composerchannel",
  "timeout": 300
}
```
运行命令创建新的业务网络卡片：
```
composer card create -u admin -s adminpw -n tutorial-network -p connection2.json -f admin@tutorial-network.card
```
导入到playground后提示身份admin已经存在了。

到这里才终于明白按[官方文档](https://wbwangk.github.io/ComposerDocs/installing_using-playground-locally/)部署的playground是和composer一起跑在容器中的。而容器中缺乏必要的调试工具，而且其连接Fabric使用的是example.com域名(而不是localhost)，这使得playground连接之前的fabric环境（如BYFN）变得很困难。  
所以决定采用本地部署的playground重新调试，而不是使用容器中的。

## Hyperledger Composer Playground(新)
准备Fabric环境：
```bash
cd ~/fabric-tools
docker rm -f $(docker ps -aq)
./startFabric.sh
sudo ./createPeerAdminCard.sh
```
首先删除了所有容器，然后启动Fabric网络，最后创建了一些Composer业务网络卡片。由于使用的不是root，所以不加sudo会报告没有权限访问/tmp。
执行创建卡片后的屏幕提示：
```
Development only script for Hyperledger Fabric control
Running 'createPeerAdminCard.sh'
FABRIC_VERSION is unset, assuming hlfv1
FABRIC_START_TIMEOUT is unset, assuming 15 (seconds)

Using composer-cli at v0.16.2
Deleted Business Network Card: PeerAdmin@hlfv1
Command succeeded

Successfully created business network card file to
        Output file: /tmp/PeerAdmin@hlfv1.card
Command succeeded

Successfully imported business network card
        Card file: /tmp/PeerAdmin@hlfv1.card
        Card name: PeerAdmin@hlfv1
Command succeeded

Hyperledger Composer PeerAdmin card has been imported
The following Business Network Cards are available:

Connection Profile: byfn-network-org1
┌─────────────────────────────┬───────────┬──────────────────┐
│ Card Name                   │ UserId    │ Business Network │
├─────────────────────────────┼───────────┼──────────────────┤
│ PeerAdmin@byfn-network-org1 │ PeerAdmin │                  │
└─────────────────────────────┴───────────┴──────────────────┘

Connection Profile: byfn-network-org1-only
┌──────────────────────────────────┬───────────┬──────────────────┐
│ Card Name                        │ UserId    │ Business Network │
├──────────────────────────────────┼───────────┼──────────────────┤
│ PeerAdmin@byfn-network-org1-only │ PeerAdmin │                  │
└──────────────────────────────────┴───────────┴──────────────────┘

Connection Profile: byfn-network-org2
┌─────────────────────────────┬───────────┬──────────────────┐
│ Card Name                   │ UserId    │ Business Network │
├─────────────────────────────┼───────────┼──────────────────┤
│ PeerAdmin@byfn-network-org2 │ PeerAdmin │                  │
└─────────────────────────────┴───────────┴──────────────────┘

Connection Profile: byfn-network-org2-only
┌──────────────────────────────────┬───────────┬──────────────────┐
│ Card Name                        │ UserId    │ Business Network │
├──────────────────────────────────┼───────────┼──────────────────┤
│ PeerAdmin@byfn-network-org2-only │ PeerAdmin │                  │
└──────────────────────────────────┴───────────┴──────────────────┘

Connection Profile: first-network
┌─────────────────────┬────────┬──────────────────┐
│ Card Name           │ UserId │ Business Network │
├─────────────────────┼────────┼──────────────────┤
│ admin@first-network │ admin  │                  │
└─────────────────────┴────────┴──────────────────┘

Connection Profile: hlfv1
┌────────────────────────┬───────────┬──────────────────┐
│ Card Name              │ UserId    │ Business Network │
├────────────────────────┼───────────┼──────────────────┤
│ admin@tutorial-network │ admin     │ tutorial-network │
├────────────────────────┼───────────┼──────────────────┤
│ PeerAdmin@hlfv1        │ PeerAdmin │                  │
└────────────────────────┴───────────┴──────────────────┘

Issue composer card list --name <Card Name> to get details a specific card
Command succeeded
```
然后再执行` composer card list`可以重新列出上面的卡片清单。  
创建的卡片实际上在下面的目录中：
```bash
ls ~/.composer/cards
admin@first-network     PeerAdmin@byfn-network-org1       PeerAdmin@byfn-network-org2-only
admin@tutorial-network  PeerAdmin@byfn-network-org1-only  PeerAdmin@hlfv1
composer-logs           PeerAdmin@byfn-network-org2
```
启动playground：
```
composer-playground
```
屏幕会锁定为日志输出。当前VM是fabric官方提供的vagrant VM，已经提前8080映射到了宿主机，所以用windows浏览器打开`localhost:8080`就可以进入playground。而且playground会在屏幕上显示所有刚刚创建的那8个卡片，但只有`admin@tutorial-network`卡片对应的`Connect now`按钮是可用的，因为我们刚刚启动了用`./startFabric.sh`脚本启动了这个卡片对应的Fabric实例。  

点击`Connect now`进入业务网络`tutorial-network`，但提示链码`tutorial-network`不存在。（在Composer中，业务网络貌似就是链码）
执行下面的步骤安装Composer运行时并启动业务网络（分别对应安装链码和链码实例化）：
```
sudo composer runtime install --card PeerAdmin@hlfv1 --businessNetworkName tutorial-network
sudo composer network start --card PeerAdmin@hlfv1 --networkAdmin admin --networkAdminEnrollSecret adminpw --archiveFile tutorial-network@0.0.1.bna --file networkadmin.card
```
至于为啥要加上`sudo`，可能是之前的Fabric环境是用root安装的，导致一些文件权限不正常。  
现在playground终于连接上本地的Fabric环境了。

#### playground连接BYFN
启动BYFN:
```
cd /opt/fabric-samples/first-network
docker rm -f $(docker ps -aq)
sudo ./byfn -m up
```
用另外的终端shell继续操作：
```
cd /opt/fabric-samples/first-network
sudo composer runtime install --card PeerAdmin@byfn-network-org1-only --businessNetworkName tutorial-network
cp ~/composer/tutorial-network/tutorial-network@0.0.1.bna .
sudo composer network start --card PeerAdmin@byfn-network-org1-only --networkAdmin admin --networkAdminEnrollSecret adminpw --archiveFile tutorial-network@0.0.1.bna --file networkadmin.card
```
上面的`composer runtime install`就是安装composer运行时。安装运行时的时候会用到网络卡片中指定的证书，而BYFN的证书是相对于`fabric-samples/first-network`目录的。如果在其他目录安装运行时，会提示：
```
Error: ENOENT: no such file or directory, open 'crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt'
```
`cp`是把业务网络档案复制到当前目录。`composer network start`命令本质上是将业务网络档案中定义的链码程序用docker容器实例化。






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

## Hyperledger环境部署的整理
Hyperledger涉及多个组件，有时本地安装有时又跑在容器中，尤其当其部署在同一个VM上时，如果不整理清楚容易引起混乱。

### playground
[部署文档](https://wbwangk.github.io/ComposerDocs/installing_using-playground-locally/)  
部署脚本：
```bash
curl -sSL https://hyperledger.github.io/composer/install-hlfv1.sh | bash
```
导致的结果是启动了多个容器，相对Composer的部署，多了一个composer容器，该容器暴露localhost:8080端口，是Playground的web界面。  
由于composer跑在容器中，所以连接配置文件中定义的fabric地址不是localhost，而是类似`ca.org1.example.com`。

### Hyperledger Composer开发环境
[部署文档](https://wbwangk.github.io/ComposerDocs/installing_development-tools/)  
利用npm安装composer相关包，如安装和运行playground：
```
npm install -g composer-playground
composer-playground
```
所以这里的playground是本地安装，而不是跑在容器中，它连接Fabric的连接配置文件中使用的Fabric地址是localhost。

composer自带的Fabric安装脚本是：
```
mkdir ~/fabric-tools && cd ~/fabric-tools
curl -O https://raw.githubusercontent.com/hyperledger/composer-tools/master/packages/fabric-dev-servers/fabric-dev-servers.zip
unzip fabric-dev-servers.zip
```
一般启动Fabric脚本：
```
cd ~/fabric-tools
./downloadFabric.sh
./startFabric.sh
./createPeerAdminCard.sh
```
### Composer开发教程
[教程文档](https://wbwangk.github.io/ComposerDocs/tutorials_developer-tutorial/)  
由于要演示Yeoman生成代码，它有自己的目录。当执行：
```
yo hyperledger-composer:businessnetwork
```
输入业务网络名称`tutorial-network`，它会自动创建目录tutorial-network，之后的操作都是在这个目录中进行了。  
之后还会启动`composer-rest-server`，也是在这个目录中。  
再后的“[查询教程](https://wbwangk.github.io/ComposerDocs/tutorials_queries/)”也是在这个tutorial-network目录中。  

### 为单组织部署Fabric
[文档地址](https://wbwangk.github.io/ComposerDocs/tutorials_deploy-to-fabric-single-org/)  
使用的`~/fabric-tools`目录。

### 为多组织部署Fabric
[文档地址](https://wbwangk.github.io/ComposerDocs/tutorials_deploy-to-fabric-multi-org/)  
从安装方式可以看出它使用的目录：
```
git clone -b issue-6978 https://github.com/sstone1/fabric-samples.git
```
它是在FYBN(first-network)脚本的基础上进行了个性化修改。所以默认目录就是`fabric-samples`

### Fabric
安装过程在[快速入门](https://hyperledgercn.github.io/hyperledgerDocs/getting_started/)文档中，但多数学习时间是花在[第一个Fabric网络](https://hyperledgercn.github.io/hyperledgerDocs/build_network_zh/)文档中。  
安装快速开始：
```
git clone https://github.com/hyperledger/fabric.git
```
安装fabric-samples：
```
git clone https://github.com/hyperledger/fabric-samples
```
进入BYFN目录则是：
```
cd fabric-samples/first-network
```
first-network会启动多个容器，主要特点是包括了两个组织共四个peer。

### 问题
Error: Failed to load connector module "composer-connector-hlfv1" for connection type "hlfv1". Cannot find module '/usr/local/lib/node_modules/composer-cli/node_modules/grpc/src/node/extension_binary/node-v57-linux-x64/grpc_node.node'  
解决办法：  
ubuntu@ubuntu-xenial:/usr/local/lib/node_modules/composer-cli$ npm rebuild --unsafe-perm  

