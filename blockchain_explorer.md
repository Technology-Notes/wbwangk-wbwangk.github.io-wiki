https://github.com/hyperledger/blockchain-explorer

先启动Fabric，可以参考Fabric的[BYFN](https://hyperledgercn.github.io/hyperledgerDocs/build_network_zh/)。

安装mysql：
```bash
apt install mysql-server
```
安装时会要求输入密码，默认用户名是root。

克隆explorer源码：
```bash
git clone --depth 1 https://github.com/hyperledger/blockchain-explorer.git
cd blockchain-explorer
```

用exploer的sql对mysql进行初始化：
```bash
mysql -u root -p < db/fabricexplorer.sql
```
#### 连接BYFN的配置文件
blockchain-explorer安装目录是`~/blockchain-explorer`，blockchain-explorer自带了一个BYFN(first-network目录下)。
配置config.json:
```json
{
    "network-config": {
                "org1": {
                        "name": "peerOrg1",
                        "mspid": "Org1MSP",
                        "peer1": {
                                "requests": "grpcs://127.0.0.1:7051",
                                "events": "grpcs://127.0.0.1:7053",
                                "server-hostname": "peer0.org1.example.com",
                                "tls_cacerts": "./first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"
                        },
                        "admin": {
                                "key": "./first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore",
                                "cert": "./first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts"
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

#### 连接Composer开发环境(fabric-tools)的配置文件
fabric-tools的安装目录是`~/fabric-tools`，blockchain-explorer的安装目录是`~/blockchain-explorer`。  
config.json:
```json
{
    "network-config": {
                "org1": {
                        "name": "peerOrg1",
                        "mspid": "Org1MSP",
                        "peer1": {
                                "requests": "grpc://127.0.0.1:7051",
                                "events": "grpc://127.0.0.1:7053",
                                "server-hostname": "peer0.org1.example.com",
                                "tls_cacerts": "../fabric-tools/fabric-scripts/hlfv1/composer/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"
                        },
                        "admin": {
                                "key": "../fabric-tools/fabric-scripts/hlfv1/composer/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore",
                                "cert": "../fabric-tools/fabric-scripts/hlfv1/composer/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts"
                        }
                }
        },
   "host":"localhost",
   "port":"8081",
   "channel": "composerchannel",
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
曾经发现报告错误（原因是`grpc://`写成了`grpcs://`）：
```
E0113 02:41:26.907444268   29526 ssl_transport_security.cc:976] Handshake failed with fatal error SSL_ERROR_SSL: error:1408F10B:SSL routines:SSL3_GET_RECORD:wrong version number.
```

注意`fabric-tools`与`first-network`的差异。前者使用`grpc`，后者使用`grpcs`；前者通道名是`composerchannel`，后者通道名是`mychannel`。

#### 构建、启动blockchain-explorer
```bash
npm install
./start.sh
```
环境是ubuntu虚拟机，宿主机是windows，虚拟机管理是vagrant。要在windows下访问blockchain-explorer，需要用NAT将虚拟机的8081端口映射到windows。所以在Vagrantfile中增加了配置：
```
config.vm.network :forwarded_port, guest: 8081, host: 8081, id: "blockchain-explorer", host_ip: "localhost", auto_correct: true # blockchain-explorer
```
修改了vagrant配置后需要重新启动虚拟机新端口映射才能生效。然后，重新启动BYFN和blockchain-explorer。然后在浏览器中访问地址`127.0.0.1:8081`就是看到blockchain-explorer界面了。

#### 停止blockchain-explorer
从start.sh中看到，blockchain-explorer是通过执行`node main.js`的方式启动的。那么查看一下node相关进程：
```
$ ps -ef | grep node
root      7718     1  3 Jan08 pts/2    01:13:26 node main.js
root     10620 10602  3 00:32 ?        00:20:58 peer node start --peer-defaultchain=false
ubuntu   12652  2099  0 07:32 pts/0    00:00:36 node /usr/local/bin/composer-playground
root     15036 14825  0 11:39 pts/3    00:00:00 grep --color=auto node
```
第一个进程就是blockchain-explorer的进程，则：
```
kill 7718
```

#### 重启blockchain-explorer
当fabric重启后，原来的区块可能删除了。这时需要重新初始化blockchain-explorer：
```
mysql -u root -p < db/fabricexplorer.sql
```
然后发现blockchain-explorer就可以显示最新的区块信息了。