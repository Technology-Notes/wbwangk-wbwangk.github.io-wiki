## 部署多节点以太坊私有链
[参考](https://www.cnblogs.com/zl03jsj/p/6876064.html)

### 安装过程
安装环境是windows10下的三个ubuntu虚拟机：
```
u1601  192.168.16.101
u1602  192.168.16.102
u1603  192.168.16.103
```
u1601充当近似于种子节点的作用，先启动。另两个节点启动时主动连接到u1602。

三个VM使用的用户是vagrant，都创建了`~/geth`目录，并将数据目录放在`~/geth/chain1`。

三个节点都需要安装geth，都要执行下列命令 ([参考](https://github.com/ethereum/go-ethereum/wiki/Installation-Instructions-for-Ubuntu))：
```
sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install geth
```
（用公司的网络apt-get update报错，用手机上网才行）  

### 启动u1601节点（节点1）

#### 创建一个以太坊账号
首先，创建一个以太坊账号，充当节点1的默认账号。这个默认账号即是节点1挖矿收益的存入账号，也是它运行交易的付费账号。
```
$ cd ~/geth
$ geth --datadir ~/geth/chain1 account new
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase: 1
Repeat passphrase: 1
Address: {67352ce02631da33a3f4112685b521217283d482}
```
创建账号是不需要以太坊网络。在创世区块中给这个账号预分配一些余额。

#### 区块链初始化
先创建创世区块配置文件，然后执行初始化。 

在创建一个创世区块配置文件（`~/geth/CustomGenesis.json`）:
```json
{
    "config": {
        "chainId": 322,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "nonce": "0x0000000000000043",     "timestamp": "0x0",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "extraData": "0x01",     "gasLimit": "0x8000001",     "difficulty": "0x401",
    "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "coinbase": "0x3333333333333333333333333333333333333332",
    "alloc": {
       "67352ce02631da33a3f4112685b521217283d482": {
         "balance": "100000000000" }
    }
}
```
DAO事件后以太坊硬分叉为ETH和ETC。两者的网络id都是1，为了区分只好引入了chainId。ETH的主网chainId是1，ETC是62。私有链要与主链区分开来，networkid和chainid都设置得个性一些，这里是322。

用`alloc`属性为新建了账号分配了一些ETH。

另外两个节点初始化也需要这个创世区块配置文件，复制到另外两个节点：
```
$ scp CustomGenesis.json vagrant@u1602:~/geth
$ scp CustomGenesis.json vagrant@u1603:~/geth
```

#### 初始化
```
$ geth --datadir ~/geth/chain1 init ~/geth/CustomGenesis.json
```
#### 启动节点1
```
$ geth --identity "u1601" --rpc --rpccorsdomain "*" --datadir "~/geth/chain1" --port "30303" --rpcapi "db,eth,net,web3" --networkid 85105780 console
> admin.nodeInfo.enode
"enode://f6f8a89474e43dc082e738786f73bb8f41b5a194465d6795dd5452389b81e09f1883a308ba72224cf463a4dd639262cf9d02c0aec426e10c7f28cdce52595eb4@[::]:30303"
```
通过`admin.nodeInfo.enode`获取节点id（enode），这个enode在启动另外两个节点时会用到。  

rpc默认监听`localhost:8545`。有时rpc需要从其他节点发出。可以增加`--rpcaddr "192.168.16.101"`参数来修改监听的IP地址。  

可以在控制台中看一下当前的peer数量：
```
> net.peerCount
0
```
没有任何peer连接到当前节点，所以显示数量是0

### 启动另外两个节点

#### 启动u1602（节点2）
在另外的终端窗口中ssh到u1602虚拟机：
```
$ geth --datadir ~/geth/chain1 init ~/geth/CustomGenesis.json
$ geth --identity "u1602" --rpc --rpccorsdomain "*" --datadir "~/geth/chain1" --port "30303" --rpcapi "db,eth,net,web3" --networkid 85105780 --bootnodes "enode://f6f8a89474e43dc082e738786f73bb8f41b5a194465d6795dd5452389b81e09f1883a308ba72224cf463a4dd639262cf9d02c0aec426e10c7f28cdce52595eb4@192.168.16.101:30303"
```
回到节点1（u1601）可以查看peer数量和peer信息：
```
> net.peerCount
1
```
可以看到peer数量不再是0，说明两个以太坊peer已经互相找到了。

#### 启动u1603（节点3）
在另外的终端窗口中ssh到u1603虚拟机：
```
$ geth --datadir ~/geth/chain1 init ~/geth/CustomGenesis.json
$ geth --identity "u1603" --rpc --rpccorsdomain "*" --datadir "~/geth/chain1" --port "30303" --rpcapi "db,eth,net,web3" --networkid 85105780 --bootnodes "enode://f6f8a89474e43dc082e738786f73bb8f41b5a194465d6795dd5452389b81e09f1883a308ba72224cf463a4dd639262cf9d02c0aec426e10c7f28cdce52595eb4@192.168.16.101:30303"
```
回到节点1（u1601）可以查看peer数量和peer清单：
```
> net.peerCount
2
> admin.peers
[{
    caps: ["eth/63"],
    id: "0e2ffa09f3ba3b5b72d2756ffe0acb96d1b4e19f4ef77bcce92eba825440bef304b88a3d3b207d0ddab1c7d138c02cb7a7dd0c5cbb98a2722f1b23327a991ccf",
    name: "Geth/u1602/v1.8.2-stable-b8b9f7f4/linux-amd64/go1.9.4",
    network: {
      inbound: true,
      localAddress: "192.168.16.101:30303",
      remoteAddress: "192.168.16.102:58530",
      static: false,
      trusted: false
    },
    protocols: {
      eth: {
        difficulty: 1025,
        head: "0x838cda0e9ed6a103ec86a1de64f7bebffcfadf645f4b63114ce37461bc5fc2d0",
        version: 63
      }
    }
}, {
    caps: ["eth/63"],
    id: "9377c9cd5501934d875d64b0da59b3e74b44b86f401e771732709f5f92e4ddd3fe41d0a12ba096bb211d3cbe9a480ba146c073f63de5f5e34d4f3fd28b6483d0",
    name: "Geth/u1603/v1.8.2-stable-b8b9f7f4/linux-amd64/go1.9.4",
    network: {
      inbound: true,
      localAddress: "192.168.16.101:30303",
      remoteAddress: "192.168.16.103:43064",
      static: false,
      trusted: false
    },
    protocols: {
      eth: {
        difficulty: 1025,
        head: "0x838cda0e9ed6a103ec86a1de64f7bebffcfadf645f4b63114ce37461bc5fc2d0",
        version: 63
      }
    }
}]
```

### 测试
打开一个新终端窗口，ssh到u1602。在u1602上新建一个账号：
```
$ geth --datadir ~/geth/chain1 account new
Passphrase: 1
Repeat passphrase: 1
Address: {862ddc87ba62b4644fd02aba13e390237b4e028d}
```
进入节点2的控制台：
```
$ geth --datadir ~/geth/chain1 attach
Welcome to the Geth JavaScript console!

instance: Geth/u1602/v1.8.2-stable-b8b9f7f4/linux-amd64/go1.9.4
coinbase: 0x862ddc87ba62b4644fd02aba13e390237b4e028d
at block: 0 (Thu, 01 Jan 1970 00:00:00 UTC)
 datadir: /home/vagrant/geth/chain1
 modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

> eth.accounts
["0x862ddc87ba62b4644fd02aba13e390237b4e028d"]
> eth.getBalance("0x862ddc87ba62b4644fd02aba13e390237b4e028d")
0
> eth.getBalance("67352ce02631da33a3f4112685b521217283d482")
100000000000
```
`geth attach`命令会用IPC与geth进程通信，然后打开一个Geth JavaScript控制台。`geth attach`时还显示了coinbase地址，这就是挖矿收益的保存账户地址。  

在节点2上新建的这个账户（地址是`0x862ddc87ba62b4644fd02aba13e390237b4e028d`）的ETH余额是0。而在创世区块中分配了余额的那个账号（地址`67352ce02631da33a3f4112685b521217283d482`）的余额是`100000000000`。

（你还可以在getBalance时故意输入错误的地址，会显示Error: invalid address）

#### 挖矿
在u1601上启动挖矿：
```
> eth.blockNumber
0
> eth.accounts
["0xb7c3a15402706b7bbecbf9c4895d434f9836e13b"]
> eth.getBalance("0xb7c3a15402706b7bbecbf9c4895d434f9836e13b")
0
> miner.start()
true
...
INFO [03-15|00:59:52] Successfully sealed new block            number=8 hash=1dc639…ddbc0d
INFO [03-15|00:59:52] 🔗 block reached canonical chain          number=3 hash=e48dbd…351f31
INFO [03-15|00:59:52] 🔨 mined potential block                  number=8 hash=1dc639…ddbc0d
INFO [03-15|00:59:52] Commit new mining work                   number=9 txs=0 uncles=0 elapsed=244.944µs
...
> miner.stop()
true
> eth.blockNumber
79
> eth.getBalance("0xb7c3a15402706b7bbecbf9c4895d434f9836e13b")
70000000000000000000
```

#### 转账交易
```
> personal.unlockAccount("0xb7c3a15402706b7bbecbf9c4895d434f9836e13b", "1")
true
> eth.sendTransaction({from: "0xb7c3a15402706b7bbecbf9c4895d434f9836e13b", to: "0x862ddc87ba62b4644fd02aba13e390237b4e028d", value: web3.toWei(1, "ether")})
INFO [03-15|01:26:52] Submitted transaction                    fullhash=0x20f2705d5701cb08a714689f3aec02fad90afa7f258aacf211b6498318a83e96 recipient=0x862Ddc87bA62b4644Fd02AbA13E390237b4e028d
"0x20f2705d5701cb08a714689f3aec02fad90afa7f258aacf211b6498318a83e96"
> eth.getBalance("0x862ddc87ba62b4644fd02aba13e390237b4e028d")
```
转账前要解锁账户，可以理解为密码换私钥。unlockAccount的参数`1`是密码。
交易通过挖矿才能打包到区块，而从确认交易。所以如果只是向上面这样发出交易，账户余额是不会变的。需要启动挖矿（不小心在u1602上执行的下列挖矿操作）：
```
> miner.start()
true
...
>miner.stop()
true
> eth.getBalance("0x862ddc87ba62b4644fd02aba13e390237b4e028d")
457000756000000000000
```
上面的数字之所以这么大，是因为不小心启动了u1602的挖矿，这个账户是u1602的coinbase账户，里面有挖矿得到的收益。

## 以太坊浏览器
以太坊主网和测试网可以通过etherscan.io来浏览。但私有以太坊网络只能使用开源的浏览器了。[这个回答](https://ethereum.stackexchange.com/questions/13667/ethereum-private-chain-explorers)中提到的两个开源浏览器项目是：

- https://github.com/gobitfly/etherchain-light

- https://github.com/etherparty/explorer

前者仅支持parity，测试了一下后者。

### etherparty/explorer
安装：
```
$ git clone https://github.com/etherparty/explorer
$ cd explorer 
$ npm start
```
(问题：npm start报错，说是找不到node。通过创建node软连接解决问题：`sudo ln -s /usr/bin/nodejs /usr/bin/node`)  

做了两个修改。一是修改package.json：
```
"start": "http-server ./app -a 0.0.0.0 -p 8000 -c-1"
```
二是修改了app/app.js：
```
var eth_node_url = 'http://192.168.16.101:8545'; // TODO: remote URL
```
geth实例的启动命令也做了修改。否则geth仅支持localhost:8545的rpc调用。将u1601的geth启动命令修改为：
```
geth --identity "u1601" --rpc --rpcaddr "192.168.16.101" --rpccorsdomain "*" --datadir "~/geth/chain1" --port "30303" --rpcapi "db,eth,net,web3" --networkid 85105780 console
```
即增加了`--rpcaddr "192.168.16.101"`。

当上面的修改都做好后执行npm start提示：
```
$ npm start
...
> EthereumExplorer@0.1.0 start /home/vagrant/geth/explorer
> http-server ./app -a 0.0.0.0 -p 8000 -c-1
...
```
然后在宿主机windows下的浏览器中访问地址：192.168.16.101:8000，就显示了当前区块链的数据。

## 部署多节点Parity PoA区块链
参考1: https://wiki.parity.io/Demo-PoA-tutorial  
参考2：[上面教程的翻译](https://github.com/wbwangk/wbwangk.github.io/blob/a6e547fb648ec5c16bbcd6ed9446056f7d999b4d/Parity/Demo-PoA-tutorial.md)  
还有[一篇文章](http://blog.csdn.net/fidelhl/article/details/55805638)但使用了废弃的命令行选项  

部署环境是win10宿主机下的3个ubuntu16.4虚拟机[1](https://raw.githubusercontent.com/wbwangk/wbwangk.github.io/master/tmp/Vagrantfile_ubuntu16.4)：
```
u1601  192.168.16.101
u1602  192.168.16.102
u1603  192.168.16.103
```
在三台VM上都创建目录`~/parity`，都利用下列命令安装parity[[1](https://wiki.parity.io/Setup#one-line-binary-installer)]：
```
$ bash <(curl https://get.parity.io -kL)
```
(用`sudo snap install parity`方式安装的parity版本不是最新的)  
本文章的一些配置与官方原文不同。是因为官方的环境是带有桌面的单机环境，而我的环境是多个节点的非桌面环境，需要通过windows宿主机带的浏览器访问以太坊区块链的UI，所以无法使用类似`localhost:8180`的方式，而是使用类似`192.168.16.101:8180`的方式。这导致在很多地方需要设定IP，否则parity会默认使用127.0.0.1。    

### 1. 选择你的链
在u1601的`~/parity`目录下创建`demo-spec.json`文件，作为创世区块配置文件：
```
{
    "name": "DemoPoA",
    "engine": {
        "authorityRound": {
            "params": {
                "stepDuration": "5",
                "validators" : {
                    "list": []
                }
            }
        }
    },
    "params": {
        "gasLimitBoundDivisor": "0x400",
        "maximumExtraDataSize": "0x20",
        "minGasLimit": "0x1388",
        "networkID" : "0x2323"
    },
    "genesis": {
        "seal": {
            "authorityRound": {
                "step": "0x0",
                "signature": "0x0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
            }
        },
        "difficulty": "0x20000",
        "gasLimit": "0x5B8D80"
    },
    "accounts": {
        "0x0000000000000000000000000000000000000001": { "balance": "1", "builtin": { "name": "ecrecover", "pricing": { "linear": { "base": 3000, "word": 0 } } } },
        "0x0000000000000000000000000000000000000002": { "balance": "1", "builtin": { "name": "sha256", "pricing": { "linear": { "base": 60, "word": 12 } } } },
        "0x0000000000000000000000000000000000000003": { "balance": "1", "builtin": { "name": "ripemd160", "pricing": { "linear": { "base": 600, "word": 120 } } } },
        "0x0000000000000000000000000000000000000004": { "balance": "1", "builtin": { "name": "identity", "pricing": { "linear": { "base": 15, "word": 3 } } } }
    }
}
```
这个文件内容与原文完全一样。`engine`是指定共识引擎，这里使用的共识引擎是[Aura](https://wiki.parity.io/Pluggable-Consensus.html#aura)。  

### 2. 安装两个节点
可以用两种方式设置parity的启动参数，命令行和配置文件。推荐使用配置文件方式。[这里](https://wiki.parity.io/Configuring-Parity#config-file.md)是配置文件的完整说明。  
在u1601节点上创建配置文件`node0.toml`:
```
[parity]
chain = "demo-spec.json"
base_path = "/home/vagrant/parity"
[network]
port = 30300
[rpc]
port = 8540
interface = "192.168.16.101"
cors = ["all"]
apis = ["web3", "eth", "net", "personal", "parity", "parity_set", "traces", "rpc", "parity_accounts"]
[ui]
port = 8180
interface = "192.168.16.101"
[websockets]
port = 8450
interface = "192.168.16.101"
```
在u1602节点的`~/parity`目录下创建配置文件`node1.toml`:
```
[parity]
chain = "demo-spec.json"
base_path = "/home/vagrant/parity"
[network]
port = 30300
[rpc]
port = 8540
interface = "192.168.16.102"
cors = ["all"]
apis = ["web3", "eth", "net", "personal", "parity", "parity_set", "traces", "rpc", "parity_accounts"]
[ui]
port = 8180
interface = "192.168.16.102"
[websockets]
port = 8450
interface = "192.168.16.102"
[ipc]
disable = true
```

而且需要把`demo-speck.json`从u1601上复制过来（）：
```
scp vagrant@u1601:/home/vagrant/parity/demo-spec.json .
```

#### RPC
用命令`parity --config node0.toml`启动节点。然后在另外的终端窗口用curl创建第一个权威地址(账号和密码都是node0)：
```
curl --data '{"jsonrpc":"2.0","method":"parity_newAccountFromPhrase","params":["node0", "node0"],"id":0}' -H "Content-Type: application/json" -X POST 192.168.16.101:8540
```
上述请求的响应会返回地址`0x00bd138abd70e2f00903268f3db08f2d25677c9e`。

创建user地址(账号和密码都是user)：
```
curl --data '{"jsonrpc":"2.0","method":"parity_newAccountFromPhrase","params":["user", "user"],"id":0}' -H "Content-Type: application/json" -X POST 192.168.16.101:8540
```
上述请求的响应会返回地址`0x004ec07d2329997267ec62b4166639513386f32e`。

用另外的终端窗口登录u1602，并使用命令`parity --config node1.toml`启动parity。然后在另外的终端窗口用curl创建第二个权威地址(账号和密码都是node1)：
```
curl --data '{"jsonrpc":"2.0","method":"parity_newAccountFromPhrase","params":["node1", "node1"],"id":0}' -H "Content-Type: application/json" -X POST 192.168.16.102:8541
```
上述请求的响应会返回地址`0x00aa39d30f0d20ff03a22ccfc30b7efbfca597c2`。

#### UI
1. 用命令`parity --config node0.toml`启动u1601节点上的parity。
(注意: 1.10版本及之后版本的parity默认没有启动UI(8180端口)，需要用`parity ui --config node0.toml`来启动。如果是非本机访问UI，还要加上`--jsonrpc-cors all`来允许跨域ajax调用。所以完整的命令是`parity ui --config node0.toml --jsonrpc-cors all`)

2. 在宿主机windows下用浏览器访问地址`192.168.16.101:8180`。第一次会让创建一个账户（原文中没提），挺麻烦还跳不过去，只能耐心按步骤做，除了让手工输入英文的“我知道了xxx”还让用手工输入助记码。

3. 点击顶部的accounts按钮后（http://192.168.16.101:8180/#/accounts）再点"RESTORE"按钮。

4. account recovery phrase中输入`node0`，account name也可以输入node0，然后在密码中输入两次`node0`，再点Import按钮就成功导入了之前创建的node0账号。

5. node0账号的地址是`0x00Bd138aBD70e2F00903268F3Db08f2D25677C9e`。

6. 用上面类似的办法导入账号`user`，密码也是`user`。user账号的地址是`0x004ec07d2329997267Ec62b4166639513386F32E`。

7. 在u1602节点上执行`parity --config node1.toml`

8. 在宿主windows下的浏览器访问地址`192.168.16.102:8181`

9. 用账户`node1`和密码`node1`恢复账户。node1账户的地址是`0x00Aa39d30F0D20FF03a22cCfc30B7EfbFca597C2`

还可以使用命令`parity account new --config node0.toml`，在不启动parity的情况下新建账号，只能这样不能控制产生的地址。

### 3.完成链配置
盘点一下手头的地址：
- 节点0（u1601）有地址是`0x00Bd138aBD70e2F00903268F3Db08f2D25677C9e`的权威账号和地址为`0x004ec07d2329997267Ec62b4166639513386F32E`的user账号。

- 节点1（u1602）有地址是`0x00Aa39d30F0D20FF03a22cCfc30B7EfbFca597C2`的权威账号。

重新打开两个节点的`demo-spec.json`文件，将两个权威账户加入到`validators`(验证者)数组中：
```json
"validators" : {
    "list": [
        "0x00Bd138aBD70e2F00903268F3Db08f2D25677C9e",
        "0x00Aa39d30F0D20FF03a22cCfc30B7EfbFca597C2"
    ]
}
```
将user账号也加入到创世区块，并分配一些余额。
```
"0x004ec07d2329997267Ec62b4166639513386F32E": { "balance": "10000000000000000000000" }
```
完整的`demo-spec.json`是下面的样子：
```json
{
    "name": "DemoPoA",
    "engine": {
        "authorityRound": {
            "params": {
                "gasLimitBoundDivisor": "0x400",
                "stepDuration": "5",
                "validators" : {
                    "list": [
                        "0x00Bd138aBD70e2F00903268F3Db08f2D25677C9e",
                        "0x00Aa39d30F0D20FF03a22cCfc30B7EfbFca597C2"
                    ]
                }
            }
        }
    },
    "params": {
        "gasLimitBoundDivisor": "0x400",
        "maximumExtraDataSize": "0x20",
        "minGasLimit": "0x1388",
        "networkID" : "0x2323"
    },
    "genesis": {
        "seal": {
            "authorityRound": {
                "step": "0x0",
                "signature": "0x0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
            }
        },
        "difficulty": "0x20000",
        "gasLimit": "0x5B8D80"
    },
    "accounts": {
        "0x0000000000000000000000000000000000000001": { "balance": "1", "builtin": { "name": "ecrecover", "pricing": { "linear": { "base": 3000, "word": 0 } } } },
        "0x0000000000000000000000000000000000000002": { "balance": "1", "builtin": { "name": "sha256", "pricing": { "linear": { "base": 60, "word": 12 } } } },
        "0x0000000000000000000000000000000000000003": { "balance": "1", "builtin": { "name": "ripemd160", "pricing": { "linear": { "base": 600, "word": 120 } } } },
        "0x0000000000000000000000000000000000000004": { "balance": "1", "builtin": { "name": "identity", "pricing": { "linear": { "base": 15, "word": 3 } } } },
        "0x004ec07d2329997267Ec62b4166639513386F32E": { "balance": "10000000000000000000000" }
    }
}
```
两个节点的创始区块配置都要改成上面的内容。

### 4.运行权威节点

在节点0（u1601）上创建密码文件`/home/vagrant/parity/node0.pwds`(node0.pwds与node0.toml在同一目录下)，内容是账号node0的密码：
```
node0
```
在节点1（u1602）上创建密码文件`/home/vagrant/parity/node1.pwds`(node1.pwds与node1.toml在同一目录下)，内容是账号node1的密码：
```
node1
```
在节点0的`node0.toml`的最后添加：
```
[account]
password = ["node0.pwds"]
[mining]
engine_signer = "0x00Bd138aBD70e2F00903268F3Db08f2D25677C9e"
reseal_on_txs = "none"
```
在节点1的`node1.toml`的最后添加：
```
[account]
password = ["node1.pwds"]
[mining]
engine_signer = "0x00Aa39d30F0D20FF03a22cCfc30B7EfbFca597C2"
reseal_on_txs = "none"
```
在节点0上启动parity：
```
parity --config node0.toml
```
在节点1上启动parity：
```
parity --config node1.toml
```

### 5.连接到节点
要让节点连接在一起需要知道它们的[enode地址](https://github.com/ethereum/wiki/wiki/enode-url-format)。

有三种方式获取enode：

- RPC

- UI (in the footer) (这个我没找到)

- 启动时的控制台输出

获取了节点的enode后，可以在`demo-spec.json`[配置文件](https://wiki.parity.io/Configuring-Parity#config-file.md)的network部分的`bootnodes = []`属性中添加。这相当于在配置种子节点。

这里使用curl来获取enode（由于指定了IP，即使在u1602上执行也行）：
```
curl --data '{"jsonrpc":"2.0","method":"parity_enode","params":[],"id":0}' -H "Content-Type: application/json" -X POST 192.168.16.101:8540
```
上述请求会返回节点0的enode。

下面要将node0的enode告诉node1。需要用上面请求返回的enode替换掉`enode://RESULT`。在node1（u1602）上执行：
```
curl --data '{"jsonrpc":"2.0","method":"parity_addReservedPeer","params":["enode://acb45956b6095c16110e522bf60023906b2f27509a0fd0b8a96a7c7929ad013a005ed42d8fc77f40b851323830129af89856330db9e33c02e641e9f90ef67d25@192.168.16.101:30300"],"id":0}' -H "Content-Type: application/json" -X POST 192.168.16.102:8541
```
注意：节点0有多个IP，为了一致可能需要手工改成`enode://RESUILT@192.168.16.101:30300`。

之后，看一下node0上的parity启动控制台上的输出，显示的是`1/25 peers`（之前是`0/25 peers`），表示有一个任何对等节点连接过来了。

### 6.发送交易
#### RPC
通过浏览器界面可以看到：user账户的余额是10000ETH，权威账户node0的余额是0。通过下面的命令从user发送1个ETH到node0：
```
curl --data '{"jsonrpc":"2.0","method":"personal_sendTransaction","params":[{"from":"0x004ec07d2329997267Ec62b4166639513386F32E","to":"0x00Bd138aBD70e2F00903268F3Db08f2D25677C9e","value":"0xde0b6b3a7640000"}, "user"],"id":0}' -H "Content-Type: application/json" -X POST 192.168.16.101:8540
```
浏览器界面会通过websocket自动刷新，现在user账户的余额是9999.00ETH，node0余额成了1.00ETH。

也可以用curl
```
curl --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x00Bd138aBD70e2F00903268F3Db08f2D25677C9e", "latest"],"id":1}' -H "Content-Type: application/json" -X POST 192.168.16.101:8540
{"jsonrpc":"2.0","result":"0xde0bb2ca241f9b0","id":1}
```
如果把上面的16进制结果转换成10进制会是`1000004917651438000`。这里的单位是wei，小数点左移18位后是1.00，即1.00ETH。

用下面的命令从user账户转账1.00ETH到node1权威账号：
```
curl --data '{"jsonrpc":"2.0","method":"personal_sendTransaction","params":[{"from":"0x004ec07d2329997267Ec62b4166639513386F32E","to":"0x00Aa39d30F0D20FF03a22cCfc30B7EfbFca597C2","value":"0xde0b6b3a7640000"}, "user"],"id":0}' -H "Content-Type: application/json" -X POST 192.168.16.101:8540
```
仍然可以用下面的命令检查余额：
```
curl --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x00Aa39d30F0D20FF03a22cCfc30B7EfbFca597C2", "latest"],"id":1}' -H "Content-Type: application/json" -X POST 192.168.16.102:8541
{"jsonrpc":"2.0","result":"0xde0b6b3a7640000","id":1}
```
上面的命令发送到`192.168.16.101:8540`也是可以的。这说明整个以太坊网络是连通的。

在浏览器界面上可以看到user用户的余额变成了9998.00ETH。

### 7.进一步开发
非权威节点不需要对共识消息进行签名，所以在配置文件中不需要加入类似下列的内容：
```
[account]
password = ["node1.pwds"]
[mining]
engine_signer = "0x00Aa39d30F0D20FF03a22cCfc30B7EfbFca597C2"
reseal_on_txs = "none"
```
当某VM上只会运行一个parity实例时，很多参数都可以直接用默认值代替。

在u1603节点上运行一个非权威parity实例，其配置文件`node2.toml`为：
```
[parity]
chain = "demo-spec.json"
[network]
bootnodes = ["enode://0b3f61826d32939de15b08efad0b6892bf91ffc70e3c5c719e78fa56d58ab3fe8a94c1a688cd259394f9fe420aac65e2927e942c14c0cde3042abba4804a16e1@192.168.16.101:30300"]
[rpc]
apis = ["web3", "eth", "net", "personal", "parity", "parity_set", "traces", "rpc", "parity_accounts"]
```
将`demo-spec.json`也要复制过来：
```
scp vagrant@192.168.16.101:~/parity/demo-spec.json .
```
启动节点2：
```
parity --config node2.toml
```
在节点0的控制台上可以看到`2/25 peers`的输出，说明节点2已经连接到了网络上。而节点1和节点2上仍显示`1/25 peers`。

### 问题

#### 安全token
用浏览器访问parity界面时，有时会提示安全令牌错误。还提示让`parity signer new-token`来生成令牌，并在页面上输入。

例如，访问地址`192.168.16.102:8181`（节点1）时提示令牌错误。则在u1602的`~/parity`目录下执行：
```
$ parity signer new-token --config node1.toml
Loading config file from node1.toml

Open: http://192.168.16.107:8181/#/auth?token=jdsS-OVYO-rF0z-eJhY
to authorize your browser.
Or use the generated token:
jdsS-OVYO-rF0z-eJhY
```
可以在页面上输入上述token`jdsS-OVYO-rF0z-eJhY`，或者直接在地址栏中输入带token的URL`http://192.168.16.107:8181/#/auth?token=jdsS-OVYO-rF0z-eJhY`。

（其实在parity启动控制台上会显示类似`Open: http://0.0.0.0:8180/#/auth?token=RtVa-6AVt-kxAq-vku6 to authorize your browser`）

#### cors
执行后续的[dapp教程](https://wiki.parity.io/Tutorial-Part-1.html)时，mydapp应该显示的`hello world`出不来。发现是cors的问题。需要用下列命令启动：
```
parity --config node0.toml --jsonrpc-cors all ui
```
或者在配置文件中定义：
```
[rpc]
cors = ["all"]
```