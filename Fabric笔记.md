
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

### orderer
工作目录是`/opt/fabric-samples/first-network`。  
orderer的docker-compose(`orderer.yaml`)文件定义：  
```yaml
version: '2'
services:
  orderer.example.com:
    container_name: orderer.example.com
    image: hyperledger/fabric-orderer
    environment:
      - ORDERER_GENERAL_LOGLEVEL=debug
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp
      # enabled TLS
      - ORDERER_GENERAL_TLS_ENABLED=true
      - ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: orderer
    volumes:
    - ./channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
    - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/var/hyperledger/orderer/msp
    - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/:/var/hyperledger/orderer/tls
    ports:
      - 7050:7050
```
启动：
```
$ CHANNEL_NAME=$CHANNEL_NAME docker-compose -f orderer.yaml up -d
```
需要说明的是，环境变量`ORDERER_GENERAL_TLS_ENABLED=true`指定了orderer只能用TLS协议访问。而sample默认的peer镜像定义中，并没有证书目录映射到容器内。也就是说该peer镜像不是为管理员用的，在里面执行`peer`命令不加`--TLS true`标志访问orderer会失败，查看orderer日志会显示：
```
first record does not look like a TLS handshake
```
而peer容器中报错是：
```
172.18.0.3:55136->172.18.0.2:7050: read: connection reset by peer.
```
如果不加
### peer
工作目录是`/opt/fabric-samples/first-network`。  
peer的docker-compose(`peer0.yaml`)文件定义：  
```
version: '2'
services:
  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    extends:
      file: ./base/peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer0.org1.example.com
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/fabric/msp
        - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls:/etc/hyperledger/fabric/tls
#        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
    ports:
      - 7051:7051
      - 7053:7053
```
需要指出的是，`/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/`这个卷是为了管理员执行`peer`命令而增加的，标准的peer镜像定义没有这个卷，这个卷一般出现在CLI镜像(fabric-tools)的定义中。  
