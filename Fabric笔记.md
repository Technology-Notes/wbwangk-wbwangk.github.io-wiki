
## 寻找管理员的证书和私钥

从`/opt/fabric-samples/basic-network`中查找，想找到系统通道的管理员私钥和证书。先说结论：  
```
$ cd /opt/fabric-samples/basic-network
$ find . -name 'Admin@*' -print
```
这样查找是因为知道管理员的证书的文件的命名习惯。结果：
```
./crypto-config/ordererOrganizations/example.com/msp/admincerts/Admin@example.com-cert.pem
 Serial Number:  54:c5:f3:d5:c7:a3:af:7b:06:80:3b:17:b8:80:4a:df
./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp/admincerts/Admin@example.com-cert.pem
./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/admincerts/Admin@example.com-cert.pem
./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/signcerts/Admin@example.com-cert.pem

./crypto-config/peerOrganizations/org1.example.com/msp/admincerts/Admin@org1.example.com-cert.pem
 Serial Number:  15:2c:67:2c:01:ac:bb:4e:33:ac:59:00:13:0c:e7:eb
./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/admincerts/Admin@org1.example.com-cert.pem
./crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/admincerts/Admin@org1.example.com-cert.pem
./crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem
```
利用openssl命令分别查看以上这些证书文件的内容：
```
$ openssl x509 -noout -text -in <证书文件>
```
可以看到前4个证书的序列号相同，是：
```
Serial Number:  
    54:c5:f3:d5:c7:a3:af:7b:06:80:3b:17:b8:80:4a:df
```
后4个证书的序列号相同，是：
```
Serial Number:  
     15:2c:67:2c:01:ac:bb:4e:33:ac:59:00:13:0c:e7:eb
```
### 寻找私钥，并与证书配对
知道Fabric私钥的文件是以`-sk`结尾的，这样查找：
```
$ cd /opt/fabric-samples/basic-network
$ find . -name '*_sk' -print
```
结果如下：
```
./crypto-config/ordererOrganizations/example.com/ca/a0606a4a860a1e31c90a23788da6f3b6b74925ed0d23061af4899409ba46ae6a_sk
./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp/keystore/4d2f776c0fef8eac3f460a7c3558dc7859c4fe458e262e674a6c23f242ea33d1_sk
./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/keystore/1deeab5433fa6e5f045eb763109d6165268fba153211af1281f00d45f54b1022_sk
./crypto-config/peerOrganizations/org1.example.com/ca/4239aa0dcd76daeeb8ba0cda701851d14504d31aad1b2ddddbac6a57365e497c_sk
./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/keystore/46be1d569fe68f33e517c9e0072a0ccfbfb42727480fb8c8d0223af321a7893d_sk
./crypto-config/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/keystore/c75bd6911aca808941c3557ee7c97e90f3952e379497dc55eb903f31b50abc83_sk
./crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/cd96d5260ad4757551ed4a5a991e62130f8008a0bf996e4e4b84cd097a747fec_sk
```
上面的这些私钥文件中，只有一个包含了字符串`Admin@example.com`，怀疑它是orderer管理员的私钥。为了证明这一点，可以找个文件用私钥签名一下，然后用公钥去验证，如果验证通过，就说明它们一对匹配的私钥和证书。
工作的目录：
```
$ cd /opt/fabric-samples/basic-network
```
#### 1. 在证书中提取提取公钥
```
$ openssl x509 -pubkey -noout -in /opt/fabric-samples/basic-network/crypto-config/ordererOrganizations/example.com/msp/admincerts/Admin@example.com-cert.pem  > public.pem
```
#### 2. 用私钥对某文件签名
```
$ openssl dgst -ecdsa-with-SHA1 -sign ./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/keystore/1deeab5433fa6e5f045eb763109d6165268fba153211af1281f00d45f54b1022_sk README.md > signature.bin
```
#### 3. 用公钥验证签名
```
$ openssl dgst -ecdsa-with-SHA1 -verify public.pem -signature signature.bin README.md
Verified OK
```
说明假设是成立的，这个文件`./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/keystore/1deeab5433fa6e5f045eb763109d6165268fba153211af1281f00d45f54b1022_sk`是管理员私钥。

## 手工建fabirc集群
参考了[这篇文章](http://www.cnblogs.com/studyzy/p/7237287.html)，但差异很大。  
Hyperledger Fabric的`fabric-samples/first-network`是在单机上运行的。本章描述如何将其部署到多台VM上。  
假设有三台VM：  
- hostname：u1601 ip:192.168.16.101  
- hostname：u1602 ip:192.168.16.102  
- hostname：u1603 ip:192.168.16.103  
计划将u1603当orderer节点，另两台当peer节点。  
首先，按[Hyperledger Fabric Samples](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#hyperledger-fabric-samples)的描述，在3台VM上都创建Fabirc运行环境，包括安装必要的二进制包，部署docker引擎和docker-compose，部署golang环境，下载Fabric相关docker镜像等。  

提示一个常用的docker命令，用于将所有容器删除，可用于peer或orderer的重启：
```
$ docker rm -f $(docker ps -aq)
```

### 生成密钥文件并复制(u1601)
登录u1601。首先进行环境清理，停止和删除现有docker容器，删除原有密钥文件。方法是：
```
$ cd /opt/fabric-samples/first-network
$ ./byfn.sh -m down
```
下面的命令生成密钥文件。密钥文件输出到了`crypto-config`目录下。
```
$ ../bin/cryptogen generate --config=./crypto-config.yaml
```
生成的密钥文件分别属于排序服务管理员(实际上是全局管理员)、两个组织管理员、4个peer管理员以及每个peer各一个的一般用户([参考](https://github.com/wbwangk/wbwangk.github.io/wiki/Fabric%E7%AC%94%E8%AE%B0#%E5%AF%BB%E6%89%BE%E7%AE%A1%E7%90%86%E5%91%98%E7%9A%84%E8%AF%81%E4%B9%A6%E5%92%8C%E7%A7%81%E9%92%A5))。  

按正规要求，排序服务管理员的私钥文件应复制到u1603；org1组织的peer1节点的私钥文件复制到u1602。由于org1组织的peer0节点就是u1601，所以不用复制。因为本章只是验证Fabric-samples分布式部署，所以加密材料只是简单复制到了u1602和u1603，并没有删除应当保密的管理员私钥。
```
$ scp -r ./crypto-config root@u1602:/opt/fabric-samples/first-network/
$ scp -r ./crypto-config root@u1603:/opt/fabric-samples/first-network/
```

### 启动排序服务

要启动排序服务，首先要生成系统创世区块，然后编辑排序服务的docker-compose文件并启动。
```
$ ssh root@u1603
$ cd /opt/fabric-samples/first-network
```
#### 生成系统通道创世区块
下面的命令生成系统通道的创世区块。文件输出到`channel-artifacts`目录下。
```
$ export FABRIC_CFG_PATH=$PWD
$ ../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
```

#### 编辑docker-compose配置文件

排序服务的docker-compose配置文件可以在原生`docker-compose-cli.yaml`上修改。
```
$ cp docker-compose-cli.yaml orderer.yaml
```
编辑orderer.yaml，删除大部分内容，只留下orderer相关的：
```yaml
version: '2'
services:
  orderer.example.com:
    extends:
      file:   base/docker-compose-base.yaml
      service: orderer.example.com
    container_name: orderer.example.com
```
同原生定义的排序服务docker-compose配置文件相比，仅仅删除了`byfn`网络。这意味着，排序服务容器与其他容器的通信不再走docker的虚拟网络，而是通过宿主机网络（至于是bridge还是host模式，不清楚）。

#### 启动orderer节点
```
$ docker-compose -f orderer.yaml up -d
```
（如果启动orderer时不加`-d`参数，屏幕会一直输出orderer容器的日志。）

### 准备peer0节点
使用u1601充当org1组织的peer0节点。
```
$ ssh root@u1601
$ cd /opt/fabric-samples/basic-network
```  

#### 生成事务文件
首先，生成创建应用通道的事务文件：
```
$ export CHANNEL_NAME=mychannel  
$ ../bin/configtxgen -profile TwoOrgsChannel \
 -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
```
然后，生成“将peer0设置Org1的锚点peer”的事务文件：
```
$ ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate \
./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
```

#### 编辑docker-compose配置文件

计划在u1601上运行两个容器，peer0.org1.example.com和cli。前者是peer容器，后者是管理员用的客户端工具。在原生`docker-compose-cli.yaml`配置文件的基础上修改。  
```
$ cp docker-compose-cli.yaml cli.yaml
```
编辑cli.yaml，修改成下面的样子：
```yaml
version: '2'
networks:
  byfn:

services:
  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.org1.example.com
    extra_hosts:
     - "orderer.example.com:192.168.16.103"
#     - "peer1.org1.example.com:192.168.16.102"
    networks:
      - byfn

  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_default
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
#    command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME} ${DELAY} ${LANG}; sleep $TIMEOUT'
    volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/chaincode
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
      - peer0.org1.example.com
    extra_hosts:
     - "orderer.example.com:192.168.16.103"
    networks:
      - byfn
```
同原来的`docker-compose-cli.yaml`相比，首先多了一个`extra_hosts`定义。`extra_hosts`的值会自动加入到容器的`/etc/hosts`文件中，以便根据hostname找到位于其他VM的服务，如找到排序服务(orderer)。  
（`byfn`这个docker虚拟网络也是需要的。如果不定义，会导致链码实例化报错。用`docker network ls`命令可以看到`byfn`被自动命名为`net_byfn`。而peer服务是从`base/docker-compose-base.yaml`继承来的，其中有个环境变量的定义`CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_byfn`，在运行时这个环境变量的值应该是`net_byfn`。）  
（如果`peer0.org1.example.com`服务中不定义orderer的`extra_hosts`，会导致链码无法部署。）  

### 启动peer容器并初始化
使用`cli.yaml`启动peer0：
```
$ TIMEOUT=10000 CHANNEL_NAME=$CHANNEL_NAME docker-compose -f cli.yaml up -d
```
用`docker ps`命令可以看到启动了两个容器：`peer0.org1.example.com`和`cli`。  

#### 创建通道和将peer0加入通道
还记得之前生成的两个事务文件`channel.tx`和`Org1MSPanchors.tx`吗？下面利用这两个事务文件对Fabric进行初始化。

进入cli容器，并查看4个环境变量，这四个变量反应了当前cli正在以peer0的身份运行：
```
$ docker exec -it cli bash
$$ echo $CORE_PEER_MSPCONFIGPATH && echo $CORE_PEER_ADDRESS && echo $CORE_PEER_LOCALMSPID && echo $CORE_PEER_TLS_ROOTCERT_FILE
```
(根据fabric目前版本的通道权限策略，谁创建谁就是通道管理员。所以`mychannel`的管理员应为peer0的管理员)
为创建通道，之前需要创建通道需要的其他设置环境变量：
```
$$ export CHANNEL_NAME=mychannel && export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
$$ peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f \
./channel-artifacts/channel.tx --tls --cafile $ORDERER_CA
```
上面命令将包含在`channel-artifacts/channel.tx`中的配置信息发送给orderer，并在当前目录下生成新通道(`mychannel`)的创世区块文件`mychannel.block`。

通道创建成功后，需要将当前peer(即peer0)加入到通道`mychannel`：
```
$$ peer channel join -b mychannel.block
```
如果你弄丢了`mychannel.block`（例如你重启了peer0容器），还可以用下列命令重新获取：
```
$$ peer channel fetch 0 mychannel.block -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
```

#### 改变锚点peer定义
回忆一下，工件`Org1MSPanchors.tx`是之前生成的。下面利用这个`Org1MSPanchors.tx`对通道定义进行变更，将peer0.org1.example.com定义为Org1的锚点peer。
```
$$ peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile $ORDERER_CA
```
（当初犯过一个错误，现在明白了：工件属于peer，与orderer无关。工件可以视为peer对配置进行修改的一个参数。）  

### 链码安装、实例化和使用
#### 链码安装
将`fabric-samples`自带的例子链码安装到peer0(u1601)上：
```
$$ peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
```
如果不配置peer0对orderer的`extra_hosts`依赖，上面的链码安装会失败。  

#### 链码实例化
下面是对将链码在peer0(u1601)上实例化：
```
$$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
```
在实测中一开始没有定义`byfn`这个网络，导致了链码实例化失败。  

#### 链码使用
查询：
```
$$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
Query Result: 100
```
调用(即a减少10，b增加10)：
```
$$ peer chaincode invoke -o orderer.example.com:7050  --tls --cafile  $ORDERER_CA  -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'
Chaincode invoke successful. result: status:200
```
重新查询一下a的值：
```
$$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
Query Result: 90
```
### 建立peer1(u1602)
在u1602上部署org1的第二个peer节点，即peer1。  

首先，从u1603（排序服务所在节点）上将加密材料复制过来：
```
$ ssh root@u1602
$ cd /opt/fabric-samples/first-network
$ scp -r root@u1603:/opt/fabric-samples/first-network/crypto-config .
$ scp root@u1601:/opt/fabric-samples/first-network/cli.yaml .
```
上面将peer0节点的docker-compose配置文件`cli.yaml`也复制过来了。  
对`cli.yaml`进行适当的编辑，成下面的样子：
```yaml
version: '2'
networks:
  byfn:

services:

  peer1.org1.example.com:
    container_name: peer1.org1.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer1.org1.example.com
    extra_hosts:
     - "orderer.example.com:192.168.16.103"
     - "peer0.org1.example.com:192.168.16.101"
    networks:
      - byfn

  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer1.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
#    command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME} ${DELAY} ${LANG}; sleep $TIMEOUT'
    volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/chaincode
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on:
      - peer1.org1.example.com
    extra_hosts:
     - "orderer.example.com:192.168.16.103"
    networks:
      - byfn
```
另外还需要修改u1602节点的`first-network/base/docker-compose-base.yaml`文件，原来peer1的暴露端口是8051，现在改成7051：
```
  peer1.org1.example.com:
(略)
    ports:
      - 7051:7051
      - 7053:7053
```
(`extra_hosts: - "peer0.org1.example.com:192.168.16.101"`这一条必须要，否则会报错)
启动容器：：
```
$ TIMEOUT=10000 CHANNEL_NAME=mychannel docker-compose -f cli.yaml up -d
```
#### 将peer1加入通道
peer1的操作同peer0相比就比较简单了。它不是组织`org1`在通道`mychannel`的锚peer，仅需要加入通道即可。
```
$ docker exec -it cli bash
$$ export CHANNEL_NAME=mychannel && export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
$$ peer channel fetch 0 mychannel.block -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
$$ peer channel join -b mychannel.block
```

### peer1链码测试
#### 链码安装
将`fabric-samples`自带的例子链码安装到peer1(u1602)上：
```
$$ peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/
```

#### 链码查询
对于通道中的同一组织的新peer，链码不用实例化直接调用（原理不明）：
```
$$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
Query Result: 90
```