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

可以用下面的方式查看导入的卡片：
```
ls -l ~/.composer/cards
drwxr-x--- 3 ubuntu ubuntu 4096 Jan 10 02:18 PeerAdmin@hlfv1
```
#### 两种选择：composer-cli或composer-playground
如果用composer-cli方式创建业务网络就按照[开发者教程](https://wbwangk.github.io/ComposerDocs/tutorials_developer-tutorial/)的步骤逐步完成，直到用ping命令测试业务网络是否正常。  
这时如果检查一下composer卡片可以看到的内容类似下面：
```
ls -l ~/.composer/cards
drwxr-x--- 3 ubuntu ubuntu 4096 Jan 10 02:30 admin@tutorial-network
drwxr-x--- 3 ubuntu ubuntu 4096 Jan 10 02:18 PeerAdmin@hlfv1
```

下面讲第二种选择，就是用composer-playground来创建、部署业务网络。Composer官方文档中，虽然在[Playground教程](https://wbwangk.github.io/ComposerDocs/tutorials_playground-tutorial/)讲到了，但那讲解的是在线Playground。下面讲解的是利用本地Playground创建和部署业务网络，这样可以与comopser-cli方式进行对比，以便更好地理解Composer的功能。 

启动composer-playground：
```
composer-playground
```
屏幕playground监听8080端口。利用vagrant已经把8080端口映射到了windows下，利用windows下的浏览器访问地址`localhost:8080`就可以进入playground界面。

首先进入的界面是My Business Networks(url：`localhost:8080/login`)。在这个网页上，可以看到两个连接：hlfv1和Web Browser。前者是本地的Fabric运行时，后者是浏览器内执行的Composer web运行时。

在hlfv1连接下有两个卡片：PeerAdmin@hlfv1和admin@tutorial-network。前者是`createPeerAdminCard.sh`脚本导入到Composer中的，后者是前面的composer-cli（即开发者教程）创建的。而且后者的Connect now按钮是“亮”的，点进去可以看到该业务网络的内容。

#### 用playground创建业务网络
点击连接hlfv1的“Deploy a new business network”（空白卡片的位置）就进入了新建基于Fabric运行时的业务网络新建业务（别点击Web Browser连接的）。

在“Give your new Business Network a name:”输入框中输入业务网络名称，如`tutorial-network2`;

在“Give the network admin card that will be created a name”输入框中输入业务网络管理员的id，如`admin@tutorial-network2`;

在“Choose a Business Network Definition to start with:”选项中可以选择默认的basic-sample-network；

在“CREDENTIALS FOR NETWORK ADMINISTRATOR”中的单选框选择ID and Secret，然后输入An Enrollment ID and Secret。对于Fabric来说，一般默认ID是`admin`，密码是`adminpw`。这应是CA的管理员用户和密码，Composer需要这个权限来来新建身份。  

最后，点击“Deploy”按钮。

之后进入的内容就跟[Playground教程](https://wbwangk.github.io/ComposerDocs/tutorials_playground-tutorial/)中一样了，也很用composer-cli创建业务网络类似。

### Fabric环境重启后建Composer业务网络
当Fabric环境(如fabric-tools)重启后，发现业务网络卡片还在，但业务网络已经提示不存在了：
```
$ composer network ping --card admin@tutorial-network
 Error trying to query business network. Error: could not find chaincode with name 'tutorial-network' 
```
重新安装业务网络，并用ping测试：
```
composer runtime install --card PeerAdmin@hlfv1 --businessNetworkName tutorial-network
composer network start --card PeerAdmin@hlfv1 --networkAdmin admin --networkAdminEnrollSecret adminpw --archiveFile tutorial-network@0.0.1.bna
composer network ping --card admin@tutorial-network
```
需要说明的是，Fabric环境重启后，业务网络卡片已经导入到了本地Composer（~/.composer/cards目录下）。所以，同[官方教程](https://wbwangk.github.io/ComposerDocs/tutorials_developer-tutorial/#_9)中的提示相比，不用生成 `networkadmin.card`，也不用再倒入业务网络卡片了。


### 为单个组织部署Hyperledger Composer区块链网络到Hyperledger Fabric
[为单个组织部署Hyperledger Composer区块链网络到Hyperledger Fabric](https://wbwangk.github.io/ComposerDocs/tutorials_deploy-to-fabric-single-org/)  

使用`Admin@org1.example.com`(组织管理员)来创建业务网络卡片，创建了文件`PeerAdmin@fabric-network.card`，并导入Composer钱包。  
安装Composer运行时：
```
composer runtime install -c PeerAdmin@fabric-network -n tutorial-network
```
业务网络名称`tutorial-network`在安装运行时指定，后面会引用。  
可以尝试创建`tutorial-network2`。如果以`-n tutorial-network2`的方式安装Composer运行时，可以成功。但之后如何启动网络时，如果`.bna`档案中定义的业务网络名称不是`tutorial-network2`会报错。   
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

## Hyperledger Composer Playground(容器方式)
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

## Hyperledger Composer Playground(容器外)
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
至于为啥要加上`sudo`？可能是之前的Fabric环境是用root安装的，导致一些文件权限不正常。  
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

下面将业务网卡卡片导入到业务网络：
```
composer network ping --card admin@tutorial-network
```
这时报错了，说127.0.0.1:7054的服务不可用。这是因为BYFN并没有启动CA容器。
幸好[为多个组织部署Hyperledger Composer](https://wbwangk.github.io/ComposerDocs/tutorials_deploy-to-fabric-multi-org/)中启动了一个改良的带CA的一组容器。删除原来的BYFN的容器，重新启动新的带CA的Fabric实例：
```
docker rm -f $(docker ps -aq)
sodu ./byfn2.sh -m up
```
这个脚本是我自己改的，功能与文章“为多个组织部署Hyperledger Composer”中的脚本一样，可以用那个代替（有一点不用：我定义的byfn2.sh启动的CA容器名称是ca.org1.example.com，而那篇文章启动的CA容器名称是ca_peerOrg1，这个容器名称实际上与文章中讲的不符）。
这时再运行ping就可以了:
```
composer network ping --card admin@tutorial-network
```
如果你纳闷为啥ping要用到CA服务。可以打开下列目录看看：
```
ls -l  ~/.composer/cards/admin@tutorial-network/credentials
```
你可以看到目录是空的，所以Composer需要CA服务来获取证书和私钥。

容易犯的一个错误：  
业务网络档案中有个package.json文件，里面确定了业务网络的名称。当运行`composer runtime install`时有个参数指定业务网络名称，这个名称必须与档案中的一致，否则当业务网络启动（即链码实例化）时会报错。  



# 实践：部署Hyperledger Composer网络到多组织Fabric
[原文](https://wbwangk.github.io/ComposerDocs/tutorials_deploy-to-fabric-multi-org)  

### Fabric环境准备
原文是重新克隆了一个改良的BYFN。其实可以直接在官方BYFN上自己改造出一个适合Composer的BYFN。  
Composer的运行需要CA容器的支持，原BYFN没有启动CA。所以要修改`docker-compose-cli.yaml`，在配置文件的最后增加以下内容：
```yaml
  ca0:
    image: hyperledger/fabric-ca
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca-org1
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_SERVER_TLS_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem
      - FABRIC_CA_SERVER_TLS_KEYFILE=/etc/hyperledger/fabric-ca-server-config/8f4cd492a4eb564930b9f0f30831adeaae5fa0504251a62aa572729ff2b3b8ec_sk
    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/8f4cd492a4eb564930b9f0f30831adeaae5fa0504251a62aa572729ff2b3b8ec_sk -b admin:adminpw -d'
    volumes:
      - ./crypto-config/peerOrganizations/org1.example.com/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.org1.example.com
    networks:
      - byfn

  ca1:
    image: hyperledger/fabric-ca
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca-org2
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_SERVER_TLS_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org2.example.com-cert.pem
      - FABRIC_CA_SERVER_TLS_KEYFILE=/etc/hyperledger/fabric-ca-server-config/2b1061a9f600848e25ac3ca2e2fd0f0ebdc0cec03dffd17f3dc54b3f78c2bc67_sk
    ports:
      - "8054:7054"
    command: sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.org2.example.com-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/2b1061a9f600848e25ac3ca2e2fd0f0ebdc0cec03dffd17f3dc54b3f78c2bc67_sk -b admin:adminpw -d'
    volumes:
      - ./crypto-config/peerOrganizations/org2.example.com/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.org2.example.com
    networks:
      - byfn
```
注意：上面的TLS私钥（如`1b90792dab005fbc00417d52f075d5ebe725b2acbd3b83a594e30c58ea998155_sk`）需要修改成你自己环境的文件名。不要轻易执行`byfn.sh -m down`，因为会导致私钥的文件名改变。  
注意：在[这个](https://github.com/wbwangk/wbwangk.github.io/wiki/Fabric%E7%AC%94%E8%AE%B0#4%E5%BD%A2%E6%88%90%E5%8F%AF%E6%89%A7%E8%A1%8C%E8%84%9A%E6%9C%AC)文档中定义了一个脚本certv.sh，可以用来检测证书和私钥是否匹配。在实测中发现当CA容器起来时会修改cryptogen生成的证书，具体的是`./crypto-config/peerOrganizations/org1.example.com/ca`和`./crypto-config/peerOrganizations/org2.example.com/ca`目录中的pem文件。建议用`chmod -w`命令将证书文件改成不允许修改，以避免麻烦。（CA容器修改证书，也可能是因为私钥文件设错了）

`docker-compose-cli.yaml`被增加以上内容后，再用byfn.sh启动BYFN会多启动两个CA容器：`ca.org1.example.com`、`ca.org2.example.com`。

最好用非root用户（与启动Composer用同一个用户）来克隆`fabric-samples`和启动`first-network`，否则会面临一系列的文件系统权限问题：
```
cd /opt/fabric-samples/first-network
./byfn.sh -m up -s couchdb
```
注意：加上`-s`参数为了启动couchdb作为全局状态数据库。Composer的查询需要依赖couchdb的能力，如果不测试Composer的查询，也可以不启动couchdb。

注意：不要随便随便执行`byfn.sh -m down`，因为会删除所有密码文件而导致导入的Composer卡片失效。如果想重启BYFN，可以使用命令`docker rm -f $(docker ps -aq)`删除所有容器，然后再执行`byfn.sh -m up`。

### 步骤三和四

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
### 步骤七到十(创建和导入卡片、安装运行时)
#### 创建业务网络卡片
下面的命令一定不要在root下执行，而是执行Composer的某个非root用户。
```bash
export ORG1_ADMIN_KEY=crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/b9571fc3d7bcfbc86015afa70a16f10c711b6c495bae8d49d5648b219b384b92_sk
export ORG2_ADMIN_KEY=crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/keystore/c0f793317984083079f7ff65b593c7e11e87f07f2da303a0c3e69734d601e19a_sk
export ORG1_ADMIN_CRT=crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem
export ORG2_ADMIN_CRT=crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/Admin@org2.example.com-cert.pem
sudo composer card create -p connection-org1-only.json -u PeerAdmin -c $ORG1_ADMIN_CRT -k $ORG1_ADMIN_KEY -r PeerAdmin -r ChannelAdmin
sudo composer card create -p connection-org1.json -u PeerAdmin -c $ORG1_ADMIN_CRT -k $ORG1_ADMIN_KEY -r PeerAdmin -r ChannelAdmin
sudo composer card create -p connection-org2-only.json -u PeerAdmin -c $ORG2_ADMIN_CRT -k $ORG2_ADMIN_KEY -r PeerAdmin -r ChannelAdmin
sudo composer card create -p connection-org2.json -u PeerAdmin -c $ORG2_ADMIN_CRT -k $ORG2_ADMIN_KEY -r PeerAdmin -r ChannelAdmin
```

#### 将卡片导入Composer
```bash
composer card import -f PeerAdmin@byfn-network-org1-only.card
composer card import -f PeerAdmin@byfn-network-org1.card
composer card import -f PeerAdmin@byfn-network-org2-only.card
composer card import -f PeerAdmin@byfn-network-org2.card
```
注意：如果提示卡片重复，可以用`rm -f ~/.composer/cards`命令删除已经导入的重名卡片。  


### 安装Composer运行时
```bash
composer runtime install -c PeerAdmin@byfn-network-org1-only -n tutorial-network
composer runtime install -c PeerAdmin@byfn-network-org2-only -n tutorial-network
```
我在这里多次碰到错误提示：
```
because of "x509: ECDSA verification failure" while trying to verify candidate authority certificate "ca.org1.example.com"
```
这是因为执行`byfn.sh -m down`删除了所有证书和私钥，执行`byfn.sh -m up`重新生成了证书和私钥。而老的私钥和证书已经作为卡片导入到了Composer中，导致Fabric环境中的证书与Composer中的不符合。

### 步骤十五（创建业务网络管理员）
下面命令的目的是根据已知的登记id和登记密码获取私钥和证书，复用了原卡片中的连接配置文件定义。
```
composer identity request -c PeerAdmin@byfn-network-org1-only -u admin -s adminpw -d alice
```
这个命令的`-u admin`和`-s adminpw`选项需要与Hyperledger Fabric CA (Certificate Authority)注册的默认用户一致。

这个证书会被保存到当前目录的alice子目录中。创建了三个证书文件，其中`admin-pub.pem`是证书(包括公钥)，`admin-priv.pem`是私钥。

### 步骤十五到十七：获取业务管理员证书、启动业务网络
```
sudo composer identity request -c PeerAdmin@byfn-network-org1-only -u admin -s adminpw -d alice
sudo composer identity request -c PeerAdmin@byfn-network-org2-only -u admin -s adminpw -d bob
```
命令会新建目录alice和bob，里面存放了获取到的证书和私钥。  

启动业务网络：
```
composer network start -c PeerAdmin@byfn-network-org1 -a tutorial-network@0.0.2.bna -o endorsementPolicyFile=endorsement-policy.json -A alice -C alice/admin-pub.pem -A bob -C bob/admin-pub.pem
```
这里能够启动网络成功，但事件通知经常超时。

### 剩余步骤
创建业务网络卡片、导入Composer、用ping连接业务网络进行测试：
```
sudo composer card create -p connection-org1.json -u alice -n tutorial-network -c alice/admin-pub.pem -k alice/admin-priv.pem
composer card import -f alice@tutorial-network.card
composer network ping -c alice@tutorial-network
sudo composer card create -p connection-org2.json -u bob -n tutorial-network -c bob/admin-pub.pem -k bob/admin-priv.pem
composer card import -f bob@tutorial-network.card
composer network ping -c bob@tutorial-network
```
到这里就成功了。

### 用playground连接BYFN
启动composer-playground：
```
cd /opt/fabric-samples/first-network
composer-playground
```
注意：一定要在上述目录下启动playground，否则会提示找不到`crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt`。  
playgrouand监听8080端口。virtualbox已经将8080端口用NAT映射到了windows，所以在windows下用浏览器打开地址[127.0.0.1:8080](127.0.0.1:8080)就可以进入playground界面。

页面上会显示多个业务网络卡片，其中“连接可用”的卡片是`alice@tutorial-network`和`bob@tutorial-network`。

### 用compser-rest-server连接到BYFN
```
cd /opt/fabric-samples/first-network
composer-rest-server
? Enter the name of the business network card to use: alice@tutorial-network
? Specify if you want namespaces in the generated REST API: never use namespaces
? Specify if you want to enable authentication for the REST API using Passport: No
? Specify if you want to enable event publication over WebSockets: Yes
? Specify if you want to enable TLS security for the REST API: No
```
composer-rest-server会监听3000端口，virtualbox已经用NAT将3000端口映射到了windows（如果没有就自己配置）。在windows用浏览器打开[http://localhost:3000](http://localhost:3000)端口查看所有REST API。

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

## BYFN改进(增加CA)
```
./byfn.sh -m up -s couchdb
```
增加了一个CA的docker-compose配置文件：
```yaml
version: '2'

networks:
  net_byfn:
    external: true

services:
  ca0:
    image: hyperledger/fabric-ca
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca-org1
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_SERVER_TLS_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem
      - FABRIC_CA_SERVER_TLS_KEYFILE=/etc/hyperledger/fabric-ca-server-config/1b90792dab005fbc00417d52f075d5ebe725b2acbd3b83a594e30c58ea998155_sk
    ports:
      - "7054:7054"
    command: sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/1b90792dab005fbc00417d52f075d5ebe725b2acbd3b83a594e30c58ea998155_sk -b admin:adminpw -d'
    volumes:
      - ./crypto-config/peerOrganizations/org1.example.com/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.org1.example.com
    networks:
      - net_byfn

  ca1:
    image: hyperledger/fabric-ca
    environment:
      - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
      - FABRIC_CA_SERVER_CA_NAME=ca-org2
      - FABRIC_CA_SERVER_TLS_ENABLED=true
      - FABRIC_CA_SERVER_TLS_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org2.example.com-cert.pem
      - FABRIC_CA_SERVER_TLS_KEYFILE=/etc/hyperledger/fabric-ca-server-config/a64893a739c00b45aeda0422d3fe7dd0de39e9738cbcc7fce2b220ab84c875f3_sk
    ports:
      - "8054:7054"
    command: sh -c 'fabric-ca-server start --ca.certfile /etc/hyperledger/fabric-ca-server-config/ca.org2.example.com-cert.pem --ca.keyfile /etc/hyperledger/fabric-ca-server-config/a64893a739c00b45aeda0422d3fe7dd0de39e9738cbcc7fce2b220ab84c875f3_sk -b admin:adminpw -d'
    volumes:
      - ./crypto-config/peerOrganizations/org2.example.com/ca/:/etc/hyperledger/fabric-ca-server-config
    container_name: ca.org2.example.com
    networks:
      - net_byfn
```
当使用byfn.sh启动后，会自动创建一个叫`net_byfn`的docker网络（不知道为啥会自动加上`net_`）。为了让CA的容器可以与其他的Fabric容器通信，需要定义一个外部网络。

启动CA容器：
```
docker-compose -f docker-compose-ca.yaml up -d
```

## 讲解Composer
[Composer架构图](https://wbwangk.github.io/ComposerDocs/introduction_solution-architecture/)  
[安装Composer和启动Fabric](https://wbwangk.github.io/ComposerDocs/installing_development-tools/)  
[开发者教程](https://wbwangk.github.io/ComposerDocs/tutorials_developer-tutorial/)  
[查询教程](https://wbwangk.github.io/ComposerDocs/tutorials_queries/)  
[Playground教程](https://wbwangk.github.io/ComposerDocs/tutorials_playground-tutorial/)  

[建模语言](https://wbwangk.github.io/ComposerDocs/reference_cto_language/)  
### 问题
Error: Failed to load connector module "composer-connector-hlfv1" for connection type "hlfv1". Cannot find module '/usr/local/lib/node_modules/composer-cli/node_modules/grpc/src/node/extension_binary/node-v57-linux-x64/grpc_node.node'  
解决办法：  
ubuntu@ubuntu-xenial:/usr/local/lib/node_modules/composer-cli$ npm rebuild --unsafe-perm  

