## 部署多节点以太坊私有链
geth是以太坊的go语言实现的客户端。另一个常用的是parity。

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



