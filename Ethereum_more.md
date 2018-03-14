## 以太坊网络测试

### 主网
运行`geth`默认会连接到以太坊主网。然后会自动同步区块数据。下面的命令可以查看geth连接到peer数量：
```
$ geth console
(一些提示)
> net.peerCount
8
```
admin.peers() 会返回当前已连接的所有节点信息：
```
> admin.peers
```
admin.nodeInfo返回的是本节点信息：
```
> admin.nodeInfo
```

### 私链
> eth.accounts
["0xbf624b9337264580fe8d69891117d0939eef4087", "0x49f620de8dd28957c4e16a7ab626ff84f94d7904"]
>


    "config": {
        "chainId": 15,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },

~geth/CustomGenesis.json:
```
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
    "alloc": {  }
}
```
geth init ~/geth/CustomGenesis.json

#### 创世块中设定部分账户余额
```
$ geth --datadir ~/geth2/chain1 account new
Your new account is locked with a password. Please give a password. Do not forget this password.
Passphrase: 1
Repeat passphrase: 1
Address: {67352ce02631da33a3f4112685b521217283d482}
```

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

geth --datadir ~/geth2/chain1 init ~/geth2/CustomGenesis.json