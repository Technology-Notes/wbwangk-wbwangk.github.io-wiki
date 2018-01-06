https://github.com/hyperledger/blockchain-explorer

先启动Fabric，可以参考Fabric的[BYFN](https://hyperledgercn.github.io/hyperledgerDocs/build_network_zh/)。

安装mysql：
```bash
apt install mysql-server
```
安装时会要求输入密码，默认用户名是root。

克隆explorer源码：
```bash
git clone https://github.com/hyperledger/blockchain-explorer.git
cd blockchain-explorer
```

用exploer的sql对mysql进行初始化：
```bash
mysql -u root -p < db/fabricexplorer.sql
```
配置config.json:
```
{
    "network-config": {
                "org1": {
                        "name": "peerOrg1",
                        "mspid": "Org1MSP",
                        "peer1": {
                                "requests": "grpcs://127.0.0.1:7051",
                                "events": "grpcs://127.0.0.1:7053",
                                "server-hostname": "peer0.org1.example.com",
                                "tls_cacerts": "../fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt"
                        },
                        "admin": {
                                "key": "../fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore",
                                "cert": "../fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts"
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
explorer安装目录是`/opt/blockchain-explorer`，BYFN安装目录是`/opt/fabirc-samples/first-network`。

构建、启动：
```bash
npm install
./start.sh
```
环境是ubuntu虚拟机，宿主机是windows，虚拟机管理是vagrant。要在windows下访问blockchain-explorer，需要用NAT将虚拟机的8081端口映射到windows。所以在Vagrantfile中增加了配置：
```
config.vm.network :forwarded_port, guest: 8081, host: 8081, id: "blockchain-explorer", host_ip: "localhost", auto_correct: true # blockchain-explorer
```
修改了vagrant配置后需要重新启动虚拟机新端口映射才能生效。然后，重新启动BYFN和blockchain-explorer。然后在浏览器中访问地址`127.0.0.1:8081`就是看到blockchain-explorer界面了。