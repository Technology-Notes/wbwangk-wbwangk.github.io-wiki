## 部署Hyperledger Composer网络到多组织Fabric
[原文：Deploying a Hyperledger Composer blockchain business network to Hyperledger Fabric (multiple organizations)](https://hyperledger.github.io/composer/unstable/tutorials/deploy-to-fabric-multi-org.html)   

这个教程示范了在多组织场景下的管理员部署一个区块链商业网络到某Hyperledger Fabric实例必须执行的几个步骤，包括怎样生成Hyperledger Composer配置。

建议首先完成前面的教程，也就是示范怎样为单个组织将一个区块链商业网络部署到某Hyperledger Fabric实例，因为它更详细地解释了一些概念。

这个教程将涵盖怎样部署一个区块链商业网络到某个跨越两个组织(`Org1`和`Org2`)的Hyperledger Fabric网络。这个教程会根据不同组织执行的步骤展现为不同的方式。

首先是两个组织都要执行的步骤显示的样子：  
*================示例步骤：一个Org1和Org2都执行的步骤================*  

表示组织`Org1`执行的步骤的样子:  
*----------------示例步骤：一个Org1执行的步骤----------------  

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