
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
$ openssl dgst -ecdsa-with-SHA1 -sign ./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/keystore/d63585d99627643f933630796832e55954da88911208ae54e0d69d51a1ff9981_sk README.md > signature.bin
```
#### 3. 用公钥验证签名
```
$ openssl dgst -ecdsa-with-SHA1 -verify public.pem -signature signature.bin README.md
Verified OK
```
说明假设是成立的，这个文件`./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/keystore/1deeab5433fa6e5f045eb763109d6165268fba153211af1281f00d45f54b1022_sk`是管理员私钥。

#### 4.形成可执行脚本
```
cat << EOF > certv.sh
echo "this is a file" > t.t
openssl x509 -pubkey -noout -in \$1  > public.pem
openssl dgst -ecdsa-with-SHA1 -sign \$2 t.t > signature.bin
openssl dgst -ecdsa-with-SHA1 -verify public.pem -signature signature.bin t.t
EOF
chmod +x certv.sh
```
使用时第一个参数是证书，第二个参数是私钥。如：
```
./certv.sh org1.pem org1.key
```


## 创建多节点fabirc集群
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

### Org3及其peer加入网络
在节计划启用一个新的虚拟机u1604。在这个虚拟机中将部署org3的一个peer：`peer0.org3.example.com`。  
在开始本节前，请先阅读[重新配置首个网络(First-Network)](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E9%87%8D%E6%96%B0%E9%85%8D%E7%BD%AE%E9%A6%96%E4%B8%AA%E7%BD%91%E7%BB%9Cfirst-network)。本节将遵循这篇文章的步骤，将Org3添加到系统通道，并将Org3的peer0节点加入应用通道`mychannel`中。
在准备阶段，需要生成Org3的加密材料和事务文件。准备阶段将在u1601上进行，等完成后，再把加密材料和事务文件复制到u1604。  

#### 为Org3准备加密材料和事务文件
首先，生成加密材料。
```
$ ssh root@u1601
$ cd /opt/fabric-samples/first-network/org3-artifacts
$ ../../bin/cryptogen generate --config=./org3-crypto.yaml
$ export FABRIC_CFG_PATH=$PWD && ../../bin/configtxgen -printOrg Org3MSP > ../channel-artifacts/org3.json
$ cd ../ && cp -r crypto-config/ordererOrganizations org3-artifacts/crypto-config/
```
然后进入u1601的cli容器。
```
$ docker exec -it cli bash
$$ apt update && apt install jq
$$ configtxlator start &
$$ CONFIGTXLATOR_URL=http://127.0.0.1:7059
$$ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  && export CHANNEL_NAME=mychannel
$$ peer channel fetch config config_block.pb -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
$$ curl -X POST --data-binary @config_block.pb "$CONFIGTXLATOR_URL/protolator/decode/common.Block" | jq . > config_block.json
$$ jq .data.data[0].payload.data.config config_block.json > config.json
$$ jq -s '.[0] * {"channel_group":{"groups":{"Application":{"groups": {"Org3MSP":.[1]}}}}}' config.json ./channel-artifacts/org3.json >& updated_config.json
$$ curl -X POST --data-binary @config.json "$CONFIGTXLATOR_URL/protolator/encode/common.Config" > config.pb
$$ curl -X POST --data-binary @updated_config.json "$CONFIGTXLATOR_URL/protolator/encode/common.Config" > updated_config.pb
$$ curl -X POST -F channel=$CHANNEL_NAME -F "original=@config.pb" -F "updated=@updated_config.pb" "${CONFIGTXLATOR_URL}/configtxlator/compute/update-from-configs" > org3_update.pb
$$ curl -X POST --data-binary @org3_update.pb "$CONFIGTXLATOR_URL/protolator/decode/common.ConfigUpdate" | jq . > org3_update.json
$$ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":'$(cat org3_update.json)'}}}' | jq . > org3_update_in_envelope.json
$$ curl -X POST --data-binary @org3_update_in_envelope.json "$CONFIGTXLATOR_URL/protolator/encode/common.Envelope" > org3_update_in_envelope.pb
$$ peer channel signconfigtx -f org3_update_in_envelope.pb
$$ peer channel update -f org3_update_in_envelope.pb -c $CHANNEL_NAME -o orderer.example.com:7050 --tls --cafile $ORDERER_CA
```
关于上述操作的解释，请参阅[重新配置首个网络(First-Network)](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#%E9%87%8D%E6%96%B0%E9%85%8D%E7%BD%AE%E9%A6%96%E4%B8%AA%E7%BD%91%E7%BB%9Cfirst-network)。  
这时，Org3的配置信息已经进入了系统通道的配置中。  
下面该到u1604虚拟机去启动Org3的peer了。  

#### 为Org3准备docker-compose配置

从u1601将Org3的加密材料复制到u1604：
```
$ ssh root@u1604
$ cd /opt/fabric-samples/first-network
$ scp -r root@u1601:/opt/fabric-samples/first-network/org3-artifacts .
```
编辑BYFN自带的Org3的docker-compose配置文件`docker-compose-org3.yaml`为以下内容：
```yaml
version: '2'
networks:
  byfn:
services:
  peer0.org3.example.com:
    container_name: peer0.org3.example.com
    extends:
      file: base/peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer0.org3.example.com
      - CORE_PEER_ADDRESS=peer0.org3.example.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org3.example.com:7051
      - CORE_PEER_LOCALMSPID=Org3MSP
    volumes:
        - /var/run/:/host/var/run/
        - ./org3-artifacts/crypto-config/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/msp:/etc/hyperledger/fabric/msp
        - ./org3-artifacts/crypto-config/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls:/etc/hyperledger/fabric/tls
    ports:
      - 7051:7051
      - 7053:7053
    extra_hosts:
     - "peer0.org1.example.com:192.168.16.101"
     - "peer1.org1.example.com:192.168.16.102"
     - "orderer.example.com:192.168.16.103"
    networks:
      - byfn

  Org3cli:
    container_name: Org3cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_LOGGING_LEVEL=DEBUG
      - CORE_PEER_ID=Org3cli
      - CORE_PEER_ADDRESS=peer0.org3.example.com:7051
      - CORE_PEER_LOCALMSPID=Org3MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash -c 'sleep 10000'
    volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/chaincode
        - ./org3-artifacts/crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
    depends_on:
      - peer0.org3.example.com
#      - peer1.org3.example.com
    extra_hosts:
     - "orderer.example.com:192.168.16.103"
    networks:
      - byfn
```
启动Org3的容器：
```
$ docker-compose -f docker-compose-org3.yaml up -d
```
进入Org3cli容器：
```
$ docker exec -it Org3cli bash
```
下面的操作分别是设置环境变量、获取`mychannel`的创世区块、将当前peer(`peer0.org3.example.com`)加入到`mychannel`：
```
$$ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem && export CHANNEL_NAME=mychannel
$$ peer channel fetch 0 mychannel.block -o orderer.example.com:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA
$$ peer channel join -b mychannel.block
```

#### 安装链码和升级背书策略
在`peer0.org3.example.com`(u1604)上安装链码`mycc`，指定版本号为2.0是为了升级背书策略。
```
$$ peer chaincode install -n mycc -v 2.0 -p github.com/chaincode/chaincode_example02/go/
```
回到u1601。由于通道`mychannel`是u1601上的peer0创建的，peer0的管理员就是通道的管理员，所以升级链码的操作应在u1601上进行。
```
$ ssh root@u1601
$ docker exec -it cli bash
$$ peer chaincode install -n mycc -v 2.0 -p github.com/chaincode/chaincode_example02/go/
$$ peer chaincode upgrade -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -v 2.0 -c '{"Args":["init","a","90","b","210"]}' -P "OR ('Org1MSP.member','Org3MSP.member')"
```
在u1601的cli容器中升级链码时，注意export上面的环境变量(如`$ORDERER_CA`)。

## Fabric环境重装

Fabric自带的VM空间太小，改用box`bento/ubuntu-16.4`重建。  
```
$ cd /e/vagrant9/ambari-vagrant
$ git clone --depth 1 https://github.com/hyperledger/fabric.git
$ cd fabric/devenv
```
值修改Vagrantfile使用空间为40G的新box：
```
#  config.vm.box = "ubuntu/xenial64"
  config.vm.box = "bento/ubuntu-16.04"
```
新box的默认用户不再是ubuntu，而是vagrant。导致devenv/setup.sh需要改，将脚本中ubuntu用户改成vagrant。这个文件在VM中映射为了`/hyperledger/devenv/setup.sh`。

由于GWF的存在，golang官方被封，需要翻墙。而且翻墙后一些CURL管道不管用。解决办法[在这里](https://github.com/wbwangk/wbwangk.github.io/wiki/Hyperledger#vm%E7%BF%BB%E5%A2%99%E9%97%AE%E9%A2%98)。

包括下面的这个命令也需要设置http_proxy:
```
cd $GOPATH/src/github.com/hyperledger/fabric
make clean gotools
```
#### 安装fabric-samples库
```
$ cd /opt
$ git clone --depth 1 -b master https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples
$ curl -sSL https://goo.gl/6wtTN5 | bash 
（$ curl -sSL https://goo.gl/6wtTN5 | bash -s 1.1.0-preview）
```
为了加快安装速度，可以用docker save/load命令从其他环境中导出fabric的相关docker镜像：
```
docker save hyperledger/fabric-javaenv:x86_64-1.0.4 > /vagrant/fabric-javaenv-1.0.4
docker load < /vagrant/fabric-javaenv-1.0.4
```
#### 安装composer
[[1](https://wbwangk.github.io/ComposerDocs/installing_development-tools/)]  
使用用户vagrant，需要翻墙和设置http_proxy。
```
curl -O https://hyperledger.github.io/composer/prereqs-ubuntu.sh
chmod u+x prereqs-ubuntu.sh
./prereqs-ubuntu.sh
```
安装composer-cli等二进制包。由于需要在`/usr/local/bin`下建composer的符号链接，导致报告没有权限，解决办法：
```
$ npm install --unsafe-perm --verbose -g composer-cli
```  
安装composer-rest-server需要sudo权限，加上`--unsafe-perm`也不行，yo也是这样。

安装fabric-tools，即composer自带的fabric环境，特点是带CA和couchdb镜像。  
执行`./downloadFabric.sh`和`./startFabric.sh`时发现需要sudo权限。解决办法是把当前用户vagrant加入到docker组：  
```
sudo gpasswd -a ${USER} docker
sudo service docker restart
```
#### 第二天composer出错
```
$ composer network ping --card admin@food-supply
Error: Failed to load connector module "composer-connector-hlfv1" for connection type "hlfv1". Cannot find module '/usr/local/lib/node_modules/composer-cli/node_modules/grpc/src/node/extension_binary/node-v57-linux-x64/grpc_node.node'
```
解决办法[[1](https://github.com/hyperledger/composer/issues/1531)]：
```
cd /usr/local/lib/node_modules/composer-cli
npm rebuild --unsafe-perm
```
问题解决。

### 完整启动过程
#### 1.启动Fabric(fabric-tools)
```
cd ~/fabric-tools
./startFabric.sh
```

#### 2.启动业务网络
```
cd ~/BlockchainPublicRegulationFabric-Food
composer runtime install --card PeerAdmin@hlfv1 --businessNetworkName food-supply
composer network start --card PeerAdmin@hlfv1 --networkAdmin admin --networkAdminEnrollSecret adminpw --archiveFile dist/food-supply.bna
composer network ping --card admin@food-supply
```

#### 3.启动composer-playground
```
composer-playground
```
然后用浏览器访问`localhost:8080`。然后连接到food-supply业务网络。

#### 4.启动composer-rest-server
```
composer-rest-server
```
业务网络卡片输入admin@food-supply，不用命名空间，不启用认证，启用事件广播，不用TLS。  
用浏览器访问`localhost:3000`可以进入它自带的Swagger-UI。  

#### 5.启动blockchain-explorer
```
cd ~/blockchain-explorer
./start.sh
```
用浏览器进入`http://localhost:8081/`就可以看到blockchain-explorer的界面。

如果fabric-tools被重新启动，需要执行第2步“启动业务网络”后，blockchain-explorer才能重新显示账本内容。

#### 用pyresttest进行功能或基准测试
```
cd ~/pyresttest
pyresttest http://localhost:3000 test.yaml
```

#### peer命令查看一些信息
这个fabric环境未启用TLS。设置环境变量:
```
export CORE_PEER_MSPCONFIGPATH=/home/vagrant/fabric-tools/fabric-scripts/hlfv1/composer/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
export CORE_PEER_LOCALMSPID="Org1MSP"
```
执行peer命令:
```
$ peer channel list
...
INFO 003 composerchannel
$ peer channel fetch newest 1.block -o localhost:7050 -c composerchannel
```
后一个命令获取通道composerchannel的最新区块保存到1.block文件中。

## 使用fabric-tools开发
在教程[编写第一个应用](https://wbwangk.github.io/hyperledgerDocs/write_first_app_zh/)中，自带了一个fabric环境，现在改造成使用fabric-tools，以便与composer保持一致并读取fabric-tools环境中的信息。

fabcar自带环境的CA域名是ca.example.com，而fabric-tools的CA域名是ca.org1.example.com。所以要修改enrollAdmin.js:
```javascript
 fabric_ca_client = new Fabric_CA_Client('http://localhost:7054', tlsOptions , 'ca.org1.example.com', crypto_suite);
```
然后执行`node enrollAdmin.js`，脚本会创建`hfc-key-store`目录，并在里面存放admin的共私钥。

执行`node registerUser.js`时碰到错误，说缺乏identity type，解决办法[在这里](https://stackoverflow.com/questions/47175691/unable-to-registeruser-for-hyperledger-fabric-fabcar-sample-project)  

下面执行query.js，之前也需要做简单修改：
```
var channel = fabric_client.newChannel('composerchannel');
```

### 向fabric-tools安装链码fabcar
#### 向fabric-tools中增加cli容器
fabric-tools没有带cli容器，根据fabcar自带的容器环境的配置文件为原型，改造fabric-tools的docker-compose配置文件。该配置文件是`/home/vagrant/fabric-tools/fabric-scripts/hlfv1/composer/docker-compose.yml`，增加下列内容：
```yaml
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
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
      - CORE_CHAINCODE_KEEPALIVE=10
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
        - /var/run/:/host/var/run/
        - /opt/fabric-samples/chaincode/:/opt/gopath/src/github.com/
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
```
然后重启整个fabric-tools环境：
```
cd ~/fabric-tools && ./stopFabric.sh
./startFabric.sh
```
#### 安装fabcar链码
fabcar环境是依靠`/opt/fabric-samples/fabcar/startFabric.sh`启动的。该脚本中有安装fabcar链码的命令，现在改造这个脚本：
```
cd /opt/fabric-samples/fabcar
cp startFabric.sh startFabric2.sh
vi startFabric2.sh
```
将startFabric2.sh修改成下列的内容：
```bash
LANGUAGE=${1:-"golang"}
CC_SRC_PATH=github.com/fabcar/go
if [ "$LANGUAGE" = "node" -o "$LANGUAGE" = "NODE" ]; then
        CC_SRC_PATH=/opt/gopath/src/github.com/fabcar/node
fi

docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp" cli peer chaincode install -n fabcar -v 1.0 -p "$CC_SRC_PATH" -l "$LANGUAGE"
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp" cli peer chaincode instantiate -o orderer.example.com:7050 -C composerchannel -n fabcar -l "$LANGUAGE" -v 1.0 -c '{"Args":[""]}' -P "OR ('Org1MSP.member')"
sleep 10
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp" cli peer chaincode invoke -o orderer.example.com:7050 -C composerchannel -n fabcar -c '{"function":"initLedger","Args":[""]}'
```
主要变化一是通道名原来是`mychannel`，现在是`composerchannel`；背书策略原来有`Org2MSP.member`，现在只剩下了Org1MSP.member。

执行刚刚创建的脚本来安装链码并测试：
```
./startFabric2.sh
docker ps
```
用`docker ps`命令可以看到刚刚实例化的`fabcar`链码。

### 继续执行query.js
原来的query.js中的通道名称是`mychannel`需要改成`composerchannel`：
```
var channel = fabric_client.newChannel('composerchannel');
```
执行链码查询：
```
node query.js
Store path:/opt/fabric-samples/fabcar/hfc-key-store
Successfully loaded user1 from persistence
Query has completed, checking results
Response is  [{"Key":"CAR0", "Record":{"colour":"blue","make":"Toyota","model":"Prius","owner":"Tomoko"}},{"Key":"CAR1", "Record":{"colour":"red","make":"Ford","model":"Mustang","owner":"Brad"}},{"Key":"CAR2", "Record":{"colour":"green","make":"Hyundai","model":"Tucson","owner":"Jin Soo"}},{"Key":"CAR3", "Record":{"colour":"yellow","make":"Volkswagen","model":"Passat","owner":"Max"}},{"Key":"CAR4", "Record":{"colour":"black","make":"Tesla","model":"S","owner":"Adriana"}},{"Key":"CAR5", "Record":{"colour":"purple","make":"Peugeot","model":"205","owner":"Michel"}},{"Key":"CAR6", "Record":{"colour":"white","make":"Chery","model":"S22L","owner":"Aarav"}},{"Key":"CAR7", "Record":{"colour":"violet","make":"Fiat","model":"Punto","owner":"Pari"}},{"Key":"CAR8", "Record":{"colour":"indigo","make":"Tata","model":"Nano","owner":"Valeria"}},{"Key":"CAR9", "Record":{"colour":"brown","make":"Holden","model":"Barina","owner":"Shotaro"}}]
```
### 用playground创建tutorial-network
fabcar是个fabric链码，现在创建一个composer链码(composer叫业务网络)，叫做`tutorial-network`。创建tutorial-network的过程可以参考composer官方文档[Playground教程](https://wbwangk.github.io/ComposerDocs/tutorials_playground-tutorial/)。  
启动Composer Playground：
```
composer-playground
```
用浏览器访问localhost:8080，点击新建一个业务网络。  
使用空白模板，业务网络名称是tn(tutorial-network)，业务网络管理员卡片是admin@tn，网络管理员凭据选择ID and Secret，默认管理员和密码是`admin:adminpw`。  

安装教程添加模型、交易处理函数、基础数据等。使用的命名空间是`org.acme.mynetwork`。执行教程的所有操作，直到把资产ABC的拥有者改成TRADER2。  

### composer-rest-server  
安装openldap[[1](https://github.com/wbwangk/wbwangk.github.io/wiki/LDAP)]。DIT后缀是`dc=example,dc=com`。添加一个叫john的用户，密码是johnldap。这个用户需要一个mail的属性[[2](https://github.com/wbwangk/wbwangk.github.io/wiki/passport_ldapauth)]。  

设置环境变量`COMPOSER_PROVIDERS`并启动composer-rest-server，选项是启用认证。  
获取令牌并请求商品`ABC`:
```
$ curl -i -X POST http://localhost:3000/auth/ldap -H "Content-Type:application/json" -d '{"username": "john", "password":"johnldap"}
set-cookie: access_token=(略); Max-Age=1209600; Path=/; Expires=Mon, 05 Feb 2018 06:35:24 GMT
$ curl -X GET --cookie "access_token=(略)" --header 'Accept: application/json' 'http://localhost:3000/api/Commodity/ABC'
{"$class":"org.acme.mynetwork.Commodity","tradingSymbol":"ABC","description":"Test commodity","mainExchange":"Euronext","quantity":72.297,"owner":"resource:org.acme.mynetwork.Trader#TRADER2"}v
```
上述获取商品的API是用浏览器通过localhost:3000查询到的。

  






## fabric-sdk-rest
项目地址：`https://github.com/hyperledger/fabric-sdk-rest`  
这个项目可以把Fabric SDK封装为REST API。

#### 下载和配置fabric-sdk-rest
下载项目：
```
cd /opt
git clone --depth 1 https://github.com/hyperledger/fabric-sdk-rest.git
cd fabric-sdk-rest
```
利用npm进行一些依赖包的下载、构建等：
```
cd packages/loopback-connector-fabric && npm link && cd -
cd packages/fabric-rest && npm link loopback-connector-fabric && npm install && cd -
npm install
```
下面使用`fabric-samples/fabcar`[[1](https://wbwangk.github.io/hyperledgerDocs/write_first_app_zh/)]的**basic-network**环境进行测试。   
首先，利用项目自带脚本配置`datasources.json`:
```
setup.sh -f /opt/fabric-samples/basic-network/ -ukat
cat packages/fabric-rest/server/datasources.json
```
#### 启动basic-network
```
cd /opt/fabric-samples/fabcar
./startFabric.sh
node enrollAdmin.js
node registerUser.js
node query.js
```
最后的query.js可以测试出Fabric网络是否好用。如果之前曾经执行过用户注册，可以跳过前面两个js。  

最后启动REST服务：
```
cd packages/fabric-rest
./fabric-rest-server -p 3000
```
VM的3000端口已经用NAT映射到了宿主机windows下，所以用浏览器可以打开地址localhost:3000来测试该REST服务。

#### 启动blockchain-explorer
启动blockchain-explorer服务的目的是为了与fabric-sdk-rest的查询结果进行对比，当然可以跳过这个部分。  

blockchain-explorer的配置文件config.json需要修改，以便连接到**basic-network**环境。config.json的配置：
```
{
    "network-config": {
                "org1": {
                        "name": "peerOrg1",
                        "mspid": "Org1MSP",
                        "peer1": {
                                "requests": "grpc://127.0.0.1:7051",
                                "events": "grpc://127.0.0.1:7053",
                                "server-hostname": "peer0.org1.example.com",
                                "tls_cacerts": "./basic-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"
                        },
                        "admin": {
                                "key": "./basic-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore",
                                "cert": "./basic-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts"
                        }
                }
        },
   "host":"localhost",
   "port":"8081",
   "channel": "mychannel",
   "GOPATH":"../artifacts",
   "keyValueStore":"/tmp/fabric-client-kvs",
   "eventWaitTime":"30000",
   "mysql":{
      "host":"127.0.0.1",
      "port":"3306",
      "database":"fabricexplorer",
      "username":"root",
      "passwd":"1"
   }
}
```
目前的**blockchain-explorer**还处于开发阶段，不太完善，只能访问相对它自己根路径的证书文件，所以只好把**basic-network**使用的Fabric证书文件复制过来：
```
cd /opt/fabric-samples
cp basic-network/ ~/blockchain-explorer/ -R
```
启动**blockchain-explorer**:
```
cd ~/blockchain-explorer && ./start.sh
```
宿主windows下浏览器访问localhost:8081，可以看到**basic-network**的基本区块链信息，包括某个块的事务id，如：`	d6e4713ba3e5c7dc54d5d1d8f93f0b78d97291fd4f1f7120259630140e0e4eff`。

#### 使用fabric-sdk-rest
用浏览器打开地址localhost:3000，进入fabric-sdk-rest的Swagger-UI页面。  
点击链接`GET /fabric/1_0/channels/{channelName}/transactions/{transactionID}`，在channelName域中输入`mychannel`，在transactionId域中输入(粘贴)`d6e4713ba3e5c7dc54d5d1d8f93f0b78d97291fd4f1f7120259630140e0e4eff`，点`Try it out!`按钮可以看到响应中的json串。可以到[这里](https://github.com/wbwangk/wbwangk.github.io/tree/master/tmp)查看输出结果。  

还可以点击链接`GET /fabric/1_0/channels/{channelName}/blocks`，然后在BlockId域中输入`4`(或其他正确的值)，blockHash留空，然后点击Try it out!`按钮，查询4号区块的信息。响应的json串包含了这个区块的信息和交易信息。可以参考这个[区块结构图](https://blockchain-fabric.blogspot.jp/2017/04/hyperledger-fabric-v10-block-structure.html)来查看区块数据和交易数据。


