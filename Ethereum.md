## 发行自己的以太坊ERC20 Token
[原文](http://blog.csdn.net/sinat_34070003/article/details/79107181)  

### 安装以太坊轻钱包MetaMask
[MetaMask](https://chrome.google.com/webstore/detail/metamask/nkbihfbeogaeaoehlefnkodbefgpgknn?authuser=2) 是一款在谷歌浏览器Chrome上使用的插件类型的以太坊钱包，该钱包不需要下载，只需要在谷歌浏览器添加对应的扩展程序即可，非常轻量级，使用起来也非常方便。（可能需要翻墙才能安装）[1](http://8btc.com/thread-76137-1-5.html)   

安装好MetaMask后Chrome右上角会多了一个橙色的狐狸头图标。点图标会打开MetaMask浮动框。新建钱包，输入密码。

MetaMask会为用户创建12个英文助记词，一定要保存好这些助记词，一定要保存好这些助记词，一定要保存好这些助记词。在其他钱包导入这个新创建的账户的时候需要输入这些助记词。

在以太坊的主网络上发布智能合约需要以太币(ETH)，而使用测试网络可以直接申请到测试用的以太币。在MetaMask浮动框的左上角的下拉框中选择`Ropsten Test Network`选项，来切换到测试网。这个测试网貌似由`infura.io`提供。

点击“BUY”按钮，然后点“ROPSTEN TEST FAUCET”。点击绿色的按钮“request 1 ether from faucet”，就可以领取1ETH。测试的话一个以太币够用了，如果再想多领点，就点[这个网址](http://faucet.ropsten.be:3001) 再领3个以太币(ETH)。倒是可以熟悉一下以太币地址的概念。

MetaMask浮动框上账户的地址显示不全，可以点账户傍边的三个黑点调出一个下拉框选择将账户地址复制到剪切板。如果我刚新建的账户地址是：
```
0x313117B29B6980603525e2de87145315879c17Eb
```
所有的加密货币都会有类似的地址，这代表了用户的账户，账户往往并余额。比特币的地址是用户公钥的哈希，以太币应该也差不多。

如果两个途径都领了，则你有了4个ETH，这可以在MetaMask中看到，最新版的MetaMask甚至显示这4个ETH价值多少美元。我这里显示的是价值2951.74美元，当然是这时的真正的以太币的价值。

做到这里你已经在Ropsten以太坊测试网络中拥有了4个以太币，可以部署自己的智能合约了。如果你拥有真正的以太币，当然也可以在以太坊的主网络上做测试，那么就是可以跳过这一节。

### 开发智能合约

以太坊的智能合约使用Solidity语言开发，这个语言（相对于比特币的脚本）是图灵完备的。为了让本测试简单，使用在线的开发工具。原文使用的是[Remix](https://ethereum.github.io/browser-solidity) ，然我这里打开后只是一个白网页。从github的`https://github.com/ethereum/browser-solidity`项目上查到了Remix的另一个部署[https://remix.ethereum.org](https://remix.ethereum.org/) ，用这个来调试智能合约源码。

一个代币智能合约的例子中[这个网址](https://ropsten.etherscan.io/address/0x655275d5ea52c62531aa43a85949c63dd858e0e0#code) 可以看到，复制到Remix的左侧源码区即可。

**Run**选项卡中的**Environment**设置为`Injected Web3`，Create按钮左边的输入框中输入：
```
100000000,"zhongxh's test token",8,"ZTT"
```
含义是Token的数量、Token的全称、Token的精度(即Token最小为小数点后几位)、Token的符号。这个内容后面会用到，如果改变了需要记下来。

需要注意**Settings**选项卡中的Solidity版本，是在下拉框中选择的。但Remix中的Solidity版本往往与合约校验时选择的Solidity版本不同。两者要完全相同，需要回来查看这个版本，确保一致。Solidity版本不一致会导致校验会失败。

点击Create按钮后，MetaMask会弹出的“CONFIRM TRANSACTION”的页面，这个页面是用来确认发送的，点击**SUBMIT**提交，再点**SENT**。

这时，创建智能合约的提案已经提交到了以太坊测试网Ropsten(infura.io)。在MetaMask的SENT选项卡中可以看到刚刚提交的智能合约提案，点击这个提案就跳转到了以太坊浏览器[etherscan](https://ropsten.etherscan.io/tx/0x5901912c0f23302a813c4462c3c1fee6724c767654dbb51cf9e2677ca1d1e8d3) 我刚刚创建的交易中。智能合约的提案也是一种特殊的交易，也会被打包到区块中。

该交易的关键信息有：
```
From: 0x313117b29b6980603525e2de87145315879c17eb
To:[Contract 0x4292f38c4e1f13cf6f7a37a194e1616b365d5247 Created]  
Nonce: 1
```
From是我的钱包地址，To是智能合约的地址(Contract Address)。下面会用到智能合约地址。

这里的Nonce是当前账户发出的交易序号。我这里显示1是因为这是当前账户的第2个交易（貌似以太坊的合约地址是账户地址加上这个Nonce后哈希得到的）。

合约地址需要记录下来，后面会用到。这个页面上的下半部分会显示Input Data，如：
```
(略)
7b8dad8d8542c6b7ac007dba67bc3a243497503e95a99848723dde87f496700290000000000000000000000000000000000000000000000000000000005f5e1000000000000000000000000000000000000000000000000000000000000000080000000000000000000000000000000000000000000000000000000000000000800000000000000000000000000000000000000000000000000000000000000c000000000000000000000000000000000000000000000000000000000000000147a686f6e6778682773207465737420746f6b656e00000000000000000000000000000000000000000000000000000000000000000000000000000000000000035a54540000000000000000000000000000000000000000000000000000000000
```
需要留意`...87f49670029`后面的部分，这是合约实例化时的初始化参数。后面也会用到。

### 发布智能合约
在etherscan浏览器的**MISC**下拉菜单中选择**Verify Contract**，进入了合约校验页面。对于智能合约来说，需要校验的内容一是源码，二是初始化参数（构造函数参数）。因为源码决定了代币(Token)的运行逻辑，而初始化参数代表了初始状态（如代币总数）。

在etherscan中校验的合约不支持导入(import)，即仅支持把合约代码写在同一个程序文件中。

在输入框中输入的内容：
```
Contract address: 0x4292f38c4e1f13cf6f7a37a194e1616b365d5247
Contract name: HumanStandardToken
Compiler: 0.4.20-nightly.2018.1.29+commit.a668b9de
Optimization: No
```
上述信息可以在etherscan浏览器的[这个页面](https://ropsten.etherscan.io/address/0x4292f38c4e1f13cf6f7a37a194e1616b365d5247#code) 查询到。
上述页面中还有合约源码和初始化参数。其中，初始化参数（Constructor Arguments (ABI-encoded and is the last bytes of the Contract Creation Code above)）如下：
```
0000000000000000000000000000000000000000000000000000000005f5e1000000000000000000000000000000000000000000000000000000000000000080000000000000000000000000000000000000000000000000000000000000000800000000000000000000000000000000000000000000000000000000000000c000000000000000000000000000000000000000000000000000000000000000147a686f6e6778682773207465737420746f6b656e00000000000000000000000000000000000000000000000000000000000000000000000000000000000000035a54540000000000000000000000000000000000000000000000000000000000
```
上面的字符串是`100000000,"zhongxh's test token",8,"ZTT"`的ABI编码。

在合约校验页面上需要分别将合约源码和初始化参数分别粘贴好，然后点**Verify And Publish**按钮。

如果一切正常会显示校验通过。

### 测试智能合约
因为Metamask不支持toekn的发送，需要切换到[MyEtherWallet](https://www.myetherwallet.com/) 钱包。

在**MyEtherWallet**页面上，点击右上角的下拉框来切换网络到`Ropsten(infura.io)`。把语言选择为简体中文。然后点超链接**发送以太币/发送代币**。用私钥的方式导入钱包。

点击Metamask右上角的“...”，然后点击"Export Private Key "，输入密码，即可获得私钥。将私钥拷到MyEtherWallet，就可以解锁你的钱包了。

钱包解锁后在屏幕右侧显示：
```
账户地址：
0x313117B29B6980603525e2de87145315879c17Eb
账户余额：
3.986141362 ROPSTEN ETH
```
这只是你账户上的以太币余额，而不是你要发行的ZTT代币的余额。

点击右下角的**Add Custom Token**按钮，调出三个输入框，输入：
```
地址 Token Contract: 0x4292f38c4e1f13cf6f7a37a194e1616b365d5247
代币符号: ZTT
小数点位数: 8
```
点击**保存**按钮，然后就出现了ZTT的余额的显示：
```
1 ZTT
```
#### MetaMask显示代币
在MetaMask浮动框中选择**Tokens**选项卡，点击**ADD TOKEN**按钮，输入合约地址：
```
0x4292f38c4e1f13cf6f7a37a194e1616b365d5247
```
MetaMask自动填充剩余输入框，然后显示余额：
```
You own 1 token
1.000 ZTT
```

### 测试代币
下面测试一下代币在多个账户之间的转移。  

在MetaMask中新建一个账号。方法是点击MetaMask浮动框右上角的头像图标，点选**Create Account**，就创建了`Acccount 2`。新账户的地址：
```
0xaD32794E0681ab84972F74D799bF683aA87a45c6
```
Account2的余额是：0ETH、0ZTT。  

象上文一样向faucet讨要1个ETH。会启动一个交易，然后在MetaMask中看到Account2拥有了1ETH。

#### Account2向Account1转账ETH
MetaMask的当前账户是Account2，点**SEND**按钮，在交易的目标地址中粘贴入Account1的地址：
```
0x313117B29B6980603525e2de87145315879c17Eb
```
数量上输入0.5，然后点**NEXT**按钮，再点**SUBMIT**就提交了转账交易。在**SENT**选项卡中可以看到新建的交易。点击该交易会跳到Etherscan浏览器，浏览器会显示这个交易的详细信息。

交易被确认后（以太坊的确认周期是15秒左右），MetaMask中可以看到账户中ETH余额的变化。

#### 代币的转账
仍通过 [https://www.myetherwallet.com](https://www.myetherwallet.com) 来完成代币ZTT的转账。

1. 通过MetaMask切换账户到Account1，导出Account1的私钥。

2. 进入myetherwallet.com，通过“发送以太币/发送代币”菜单导入私钥(解锁)。

3. 在“发送至地址”中输入Account1的地址`0x313117B29B6980603525e2de87145315879c17Eb`，转账金额是0.1，下拉框中选择ZTT，点“生成交易”按钮

4. 会有提示，意思是得等以太坊的矿工生成区块才算转账成功。可以按提示去看交易状态。

5. 成功后在MetaMask中切换账户到Account2，可以看到Account2的ZTT代币的余额现在是0.1ZTT了。



## 搭建基于以太坊go-ethereum的私有链环境
[原博文](http://blog.csdn.net/wo541075754/article/details/53064877)
安装在ubuntu16上。

### 安装go-ethereum客户端
```
sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo add-apt-repository -y ppa:ethereum/ethereum-dev
sudo apt-get update
sudo apt-get install ethereum
```
#### 创世区块
工作目录是`~/geth`。新建创世块json文件piccgenesis.json。内容如下：
```
{
 "nonce":"0x0000000000000042",
 "mixhash":"0x0000000000000000000000000000000000000000000000000000000000000000",
 "difficulty": "0x4000",
 "alloc": {},
 "coinbase":"0x0000000000000000000000000000000000000000",
 "timestamp": "0x00",
 "parentHash":"0x0000000000000000000000000000000000000000000000000000000000000000",
 "extraData": "SecBroBlock",
 "gasLimit":"0x0000ffff"
}
```
初始化：
```
geth init ~/geth/genesis.json
```
#### 启动私有链
```
$ geth --identity "secbro etherum" --rpc --rpccorsdomain "*" --datadir "~/geth/chain2" --port "30303" --rpcapi "db,eth,net,web3" -- networkid 95518 console --dev 
...
IPC endpoint opened: /home/vagrant/geth/chain2/geth.ipc
HTTP endpoint opened: http://127.0.0.1:8545
```
最后的--dev表示以开发模式启动

#### 命令行测试
看到启动页面之后，新开启一个终端，并执行一下命令，并把日志输出到文本文件当中：
```
$ geth --dev console 2>> file_to_log_output
> eth.accounts
["0xedfc191463e44db52fd7dc1562bd8379d28c32c2"]
> personal.newAccount("1")
"0xe03a714a803e54525b8a6352990ead0b4e747bd6"
> eth.accounts
["0xedfc191463e44db52fd7dc1562bd8379d28c32c2", "0xe03a714a803e54525b8a6352990ead0b4e747bd6"]
> eth.blockNumber
0
```
eth.accounts是查看账户。personal.newAccount是新建用户，括号里是密码。最后是查看区块高度。

启动和停止挖矿：`miner.start()`、`miner.stop()`。

查看账户余额和转账：
```
> eth.getBalance("0xe03a714a803e54525b8a6352990ead0b4e747bd6")
0
> eth.sendTransaction({from:"0xedfc191463e44db52fd7dc1562bd8379d28c32c2",to:"0xe03a714a803e54525b8a6352990ead0b4e747bd6",value:web3.toWei(3,"ether")})
0x9c6c89f73133ee8d28ac07db9a7f670b79f8c4f24cabdb13e92d185f30b57646
> eth.getBalance("0xe03a714a803e54525b8a6352990ead0b4e747bd6")
3000000000000000000
```
转了3ETH却显示3后面18个0，因为显示的计量单位是Wei。

按ctrl+D可以退出交互环境。

### truffle和testrpc
truffle是以太坊最受欢迎的一个开发框架。安装：
```
$ npm install -g truffle
```
truffle是本地的用来编译、部署智能合约的工具。testrpc不同于geth，geth是真正的以太坊环境，testrpc是在本地使用内存模拟的一个以太坊环境，对于开发调试来说，更为方便快捷，当合约在testrpc中测试通过后，再部署到geth中去。所以可以说truffle和testrpc就是两大杀器。

项目初始化：
```
$ cd ~/geth
$ mkdir truffleProject && cd truffleProject
$ truffle init
```
执行以上命令之后，truffle会默认生成一个MetaCoin的demo。

安装testrpc：
```
npm install -g ethereumjs-testrpc
```
屏幕提示该项目已经改名为ganache-cli。重新安装：
```
npm install -g ganache-cli
```
运行命令也由testrpc更名为ganache-cli。

ganache(即testrpc)也会使用8545端口，所以需要先停止之前另一个终端窗口的geth。然后执行：
```
$ ganache-cli
...
Available Accounts
==================
(0) 0x1cc3599dce8bfbe2d96b2a0f7507cfbd71b1ef0b
(1) 0x2fc3a30008ce738195a2704a37cec3509c6edbbe
(2) 0x9c7d4ac0e0d5d13a267c28a8da9c050e3ce81f13
(3) 0xb369d8bcd384fd55b4cf718661eba862ef7feb32
(4) 0x37d18b941366708ce02bfc1df660eb7a68b72ea1
(5) 0xbded52ecc06e1e07bd4f0a6b3aa2067e74cec832
(6) 0xbf585d6aeaa82445dd2b4b326b021f4bf070411c
(7) 0xbed1d6ca867e6ab1cd17cb4a5bf91101641e45dc
(8) 0xa9cc18829e9bd5ad858d83f7bf5848f379afe814
(9) 0xa9157fbce92203255f0be7a906d01841e4320ed9

Private Keys
==================
(0) 484c3525ddd00e22b65fbed1609d220007eaa45682e4bee7db001708f20cfcbf
(1) 4907f855fe3479d8a3113d7c818324044fb33afb78921ab78b3cf1c70f525c54
(2) 96ad5a8b9d2c7376d76e4d4f24a278f0b201f40583ede5b47c2f8648cadb1787
(3) 5d87b37179b7f241ca6b5e628769effac939ee59d121f544cd0fd4271901ba38
(4) 82e6003b37668866d5ba4a57946f65e7868e89c3530f41444995a4b062cd59e9
(5) 92b01af2eb726f76e1bf7c14311804de8536737001564070ec5efe454a179ef9
(6) 9e18ba2e0d6b281e8d40a7de27fc25d33b7e3dab82c8a1ba32fe55036479d09c
(7) 84f5ed50f8c0e5b8fcd1a3badc7e3215165769a6c2eb234d745eeeac24e1ba69
(8) 47b954d9d163ad47d811e5693fe15b7ed4100d06e8b45e0a142679029e7ee4b3
(9) 6546f97235297ecdcd533fc7fdb23c940cd075055d0bdffce94e2a37e7147055

HD Wallet
==================
Mnemonic:      clever trust network couple give become cereal skate idle planet term spin
Base HD Path:  m/44'/60'/0'/0/{account_index}

Listening on localhost:8545
```
看上去ganache代替geth提供了一个以太坊的运行环境。屏幕上显示的是一些测试用的账号以及对应的私钥。

#### 部署和运行
将当前项目部署到测试环境：
```
$ cd truffleProject
$ truffle deploy
Using network 'development'.
Network up to data.
```
运行web服务，原文是`truffle serve`。但实测报错：
```
TypeError: fsevents is not a constructor
```
网上找到的[答案](https://github.com/trufflesuite/truffle/issues/463) 是用`npm run dev`代替。
```
$ cd truffleProject2
 npm run dev

> truffle-init-webpack@0.0.2 dev /home/vagrant/geth/truffleProject2
> webpack-dev-server

Project is running at http://localhost:8080/
webpack output is served from /
Hash: 06cbdfeabaa7f12839e6
Version: webpack 2.7.0
...
webpack: Compiled successfully.
```
这是用`webpack-dev-server`代替了`truffle serve`。truffle官方文档也说应该这么做。

#### 测试
按博文原文用浏览器访问http://localhost:8080 应显示如下页面：
![](http://img.blog.csdn.net/20161114210305453)
我用curl访问localhost:8080与上面的图片内容一样。虽然用NAT将8080映射到了windows，但在windows下浏览器访问localhost:8080却出不来页面，可能是webpack-dev-server监听的方式与NAT不兼容。

通过上面的页面可以完成转账.

## 笔记
### 详解以太坊的工作原理
[1](https://baijiahao.baidu.com/s?id=1581231980661527205&wfr=spider&for=pc)
以太坊的哈希算法:KECCAK-256  

以太坊有两种类型的账户：

1. 外部拥有的账户，被私钥控制且没有任何代码与之关联

2. 合约账户，被它们的合约代码控制且有代码与之关联

![](https://t12.baidu.com/it/u=3959818670,2742092085&fm=173&s=B1B14D308322650B5B4D7C5A030050F0&w=640&h=298&img.JPEG) 

### 以太坊黄皮书
[地址](https://github.com/yuange1024/ethereum_yellowpaper/blob/master/Paper_Chinese.pdf)  

**账户状态**包含以下四个字段：

- nonce，随机数：这个值等于账户发出的交易数与这个账户创建的合约数量之和。（创建合约本来就是交易的一种，所以这个值就是该账户的交易数）

- balance，余额：表示这个账户拥有多少Wei。（1ETH=Wei*10^18）

- storageRoot，存储根节点：保存账户内容的Merkle Paricia树根节点的256位哈希编码到字典树中，作为从256位整数键值哈希的Keccak 256位哈希到256位整数的RLP-编码映射。

- codeHash，代码哈希：这个账户的ENM(Ethereum Virtual Machine，以太坊虚拟机)代码哈希值——代码执行时，这个地址会接受一个消息调用；它和其他字段不同，创建后不可更改。状态数据库中包含所有像这样的代码片段哈希，以便后续使用。

#### 交易的字段
- nonce，随机数：账户状态中nonce的当前值

- gasPrice，燃料价格：为执行交易所需要的计算资源付出的gas价格，以Wei为单位

- gasLimit，燃料上限：用于执行交易的最大gas数量。这个值须在交易前设置，且设置后不能再修改。（可以理解为油箱大小，如果路程远请换大油箱）

- to，接收者地址：消息调用接收者的160位地址。对于合约创建交易，无需接收者地址

- value，转账额度：转到接收者账户的额度，以Wei为单位。对于合约创建，表示捐助到合约地址的额度

- init，初始化（合约创建专用字段）：一个不限制大小的字节数组，表示账户初始化程序的EVM代码。init是EVM代码片段；执行init后会返回另外一个代码片段，每次合约接受小说调用（通过交易或内部调用）后都会执行这个代码片段。仅当合约账户创建时会执行一次init。

- data，数据（消息调用专用字段）：一个不限制大小的字节数据，表示消息调用的输入数据

#### 区块的字段
区块头字段：

- parentHash，父块哈希：父区块头的Keccak 256位哈希

- ommersHash，叔链哈希：当前区块的叔链列表Keccak 256位哈希

- beneficiary，受益者地址：成功挖到这个区块的160位地址，这个区块中的所有交易费都会转到这个地址

- stateRoot，状态字典树根节点哈希：状态字典树根节点的Keccak 256位哈希，交易打包到当前区块且区块定稿后可以生成这个值

- transactionsRoot，交易字典树根节点哈希：交易字典树根节点Keccak 256位哈希，在交易字典树含有区块中的所有交易列表

- receiptsRoot，接受者字典树根节点哈希：接受者字典树根节点的Keccak 256位哈希，在接受者字典树含有区块中的所有交易信息中的接收者。

（后略）

#### 交易收据
为了让交易信息编码能有利于零知识证明、索引、搜索，我们将每个包含一定信息的交易收据进行编码。保存在一个索引字典树中。交易收据是一个包含四个条目的元组：交易后的状态；当前区块中交易累计燃料使用量，交易发生后会立即更新这个值；交易执行过程中创建的日志集合；日志Bloom过滤器。

#### 整体有效性

如果一个区块同时满足一下几个条件，我们才能认为这个区块是有效的：当从起始状态（父块的最终状态）按顺序执行完生成新的状态后，在内部要保持一致，包括叔链、交易区块哈希、给定的交易。

### truffle
初始化，生成项目骨架：
```
cd ~/geth/truffleProject
truffle init
testrpc
```
testrpc运行了一个模拟的以太坊环境，服务端口是8545。

另外起一个终端窗口，编辑当前目录下的truffle.js：
```javascript
module.exports = {
  networks: {
    development: {
      host: "localhost",
      port: 8545,
      network_id: "*" // Match any network id
    }
  }
};
```
然后执行：
```
$ truffle deploy
Using network 'development'.
Network up to date.
```
truffle serve已经不推荐了，而用类似webpack：http://truffleframework.com/boxes/webpack

###  以太坊客户端Ethereum Wallet与Geth区别简介
[原文](http://blog.csdn.net/wo541075754/article/details/77619533)   
Geth是以太坊客户端，含有分布式账本数据，可以达几十G。   
Ethereum Wallet是个图形UI，含有账户私钥，可以代表个人身份与以太坊交互，如转账。   

## Truffle官方教程
[官方地址](http://truffleframework.com/tutorials/)   

### 以太坊概述
[原文地址](http://truffleframework.com/tutorials/ethereum-overview)  
进行Truffle教程前，要求先读一下这个“以太坊概述”。这里将概述中关键内容摘录在这里。不包括区块链的介绍，只有关于以太坊的部分。  

#### 以太坊是什么
以太坊是一个区块链，允许你在其可信赖的环境中运行程序。这与比特币区块链形成鲜明对比，区块链只允许你管理加密货币。

为此，以太坊有一个名为以太坊虚拟机（EVM）的虚拟机。EVM允许代码在区块链中进行验证和执行，从而保证其在每个人的机器上以相同的方式运行。此代码包含在“智能合约”中。

除了追踪账户余额之外，以太坊还会在区块链上维护EVM的状态。所有节点运行智能合约，来验证合约及其输出的完整性。

#### 什么是智能合约
**智能合约是在EVM上运行的代码**。智能合约可以接受并存储以太币、数据，或两者的组合。然后，使用编入合同的逻辑，它可以将以太币分配给其他账户甚至其他智能合约。

这是Bob和Alice的智能合约示例。爱丽丝想聘请鲍勃建立一个露台，他们正在使用一个托管合同（一个地方来存钱，直到满足条件）在最终交易之前存储他们的以太币。
![](http://truffleframework.com/tutorials/images/ethereum-overview/smart-contract-step1.png)
1. 爱丽丝同意在托管合同中存入露台的付款，而鲍勃同意存入相等的金额  
![](http://truffleframework.com/tutorials/images/ethereum-overview/smart-contract-step2.png)  
2. 鲍勃完成露台项目，爱丽丝给予智能合同许可释放资金
![](http://truffleframework.com/tutorials/images/ethereum-overview/smart-contract-step3.png)
3. 鲍勃收到爱丽丝的付款和押金
（可以把规定写入合同代码中：如果鲍勃不能建立露台或者如果他表现不佳的话，需要把鲍勃的押金赔偿给爱丽丝。）
**智能合约是用一种名为Solidity的语言编写的**。[Solidity](https://solidity.readthedocs.io/)是静态类型的，支持继承、库和复杂的用户定义类型等等。Solidity语法与JavaScript类似。

#### 以太坊网络
到目前为止，我们已经描述了以太坊公开链（或称“MainNet”）。在MainNet上，包括账户余额和交易在内的链上数据是公开的，任何人都可以创建节点并开始验证交易。以太币在这个网络上有市场价值，可以兑换成其他加密货币或者美元等法定货币（fiat currency，简称法币）。

1. 本地测试网络
以太坊区块链可以在本地模拟开发。本地测试网络可以即时处理事务，并可根据需要分配以太币。存在一系列以太坊模拟器; 我们推荐[Ganache](http://truffleframework.com/ganache)。

2. 公共测试网络
在最终部署到主网络之前，开发人员使用公共测试网络（或测试网）来测试以太网应用程序。这些网络上的以太币仅用于测试目的，没有任何价值。

有三个公共测试网络被广泛使用：

- **Ropsten**：由以太坊基金会创建的官方测试网络。其功能类似于MainNet。

- **Kovan**：一种使用称为“权威证明”( "proof-of-authority")的共识方法的网络。这意味着它的交易通过选定的成员进行验证，从而带来一贯性的四秒的锁定时间。此测试网上的以太币的供应也受到控制，以减轻垃圾迁移（智能合约的部署或升级被称为迁移mitigate）攻击。

- **Rinkeby**：一个也使用权威证明的测试网，由以太坊基金会创建的。

3. 私有/企业网络
以太网专用网络允许各方共享数据，而不公开访问。私人区块链是以下选择的不错选择：

- 共享敏感数据，如医疗保健记录
- 由于较小的网络规模，扩展可以处理更高的读写吞吐量

私有企业区块链的一个例子是最初由摩根大通实现的[Quorum](https://www.jpmorgan.com/country/US/EN/Quorum) 。（请阅读我们的[博客文章](http://truffleframework.com/tutorials/building-dapps-for-quorum-private-enterprise-blockchains) ，了解使用Truffle调试Quorum。）

#### 分布式应用程序(dapp)
使用智能合约进行处理的应用程序称为“分布式应用程序”或“dapps”。这些dapps的用户界面包括熟悉的语言，如HTML、CSS和JavaScript。应用程序本身可以托管在传统的Web服务器上，也可以驻留在分散的文件服务（如[Swarm](http://swarm-gateways.net/bzz:/theswarm.eth/) 或[IPFS](http://ipfs.io/) ）上。

鉴于以太坊区块链的好处，dapp可能成为许多行业的解决方案，包括但不限于：

- 记账  
- 金融  
- 供应链  
- 房地产  
- 交易市场  

创建自己的dapp，测试它并将其部署到你选择的以太坊网络的最佳方法是什么？当然是使用[Truffle](http://truffleframework.com/docs/getting_started/project) 。

### 调试一个智能合约
[1](http://truffleframework.com/tutorials/debugging-a-smart-contract)  
本教程使用的是[Solidity](https://solidity.readthedocs.io/en/develop/introduction-to-smart-contracts.html) 文档中的例子。大约相当于Solidity的hello world。  

例子的程序源码是：
```solidity
pragma solidity ^0.4.17;

contract SimpleStorage {
  uint myVariable;

  function set(uint x) public {
    myVariable = x;
  }

  function get() constant public returns (uint) {
    return myVariable;
  }
}
```
1.创建工作目录; 2.创建一个空的Truffle项目(在本机的/home/vagrant/geth目录下)：
```
mkdir simple-storage && cd simple-storage
truffle init
```
3.在`contracts/`目录下创建一个文件`Store.sol`，内容如下：
```
pragma solidity ^0.4.17;

contract SimpleStorage {
  uint myVariable;

  function set(uint x) public {
    myVariable = x;
  }

  function get() constant public returns (uint) {
    return myVariable;
  }
}
```
4. 创建了一个叫`SimpleStorage`的合约。  
在`migrations/`目录下创建一个叫`2_deploy_contracts.js`的文件，包含下列内容：
```
var SimpleStorage = artifacts.require("SimpleStorage");

module.exports = function(deployer) {
  deployer.deploy(SimpleStorage);
};
```
这个文件用于将合约部署到区块链。

5. 编译合约。注意在项目的根目录下运行：
```
truffle compile
```
6. 打开一个新的终端窗口，运行`truffle develop`来启动Truffle内置的开发环境（这是Truffle内置的区块链模拟环境，可用于合约的调试。在下一个教程中将会使用一个叫Ganache的区块链模拟环境）：
```
truffle develop
```
屏幕上会显示一些预置的账户和私钥信息，然后进入`truffle(develop)>`提示符。在这个truffle命令行中，输入下列命令将合约部署到区块链：
```
truffle(develop)> migrate
```
#### 与这个基础智能合约进行交互
通过`truffle develop`智能合约被部署到了一个测试网络中，这将进入一个类似[Ganache](http://truffleframework.com/ganache) 的[控制台](http://truffleframework.com/docs/getting_started/console) 。Ganache是专门为Truffle实现的开发用区块链。  

*注意：可能你疑惑为啥我们不需要通过挖矿来确保交易安全，Truffle开发控制台已经为我们做好了这些。如果使用一个不同的网络，你可能需要通过挖矿确保交易提交到区块链。* 

1. 在`trffle develop`工作台中运行下列命令：
```
SimpleStorage.deployed().then(function(instance){return instance.get.call();}).then(function(value){return value.toNumber()});
```
这个命令访问SimpleStorage合约，调用`get()`函数，返回值是字符串，转换为数字，显示为`0`。*Solidity的整数变量的初始值是0*，这不同于其他语言，可能会`NULL`和`undefined`。  

2. 现在在合约上运行一个交易。
```
SimpleStorage.deployed().then(function(instance){return instance.set(4);});
```
这会将变量设置为`4`。提示中会显示交易ID(`tx`和`transactionHash`)。下面的调试中会用到这个交易ID。

3. 使用`get()`函数来检查修改后的值：
```
SimpleStorage.deployed().then(function(instance){return instance.get.call();}).then(function(value){return value.toNumber()});
4
```
#### 单步执行
Truffle Develop比较厉害的地方是支持单步调试。  
故意在原来的合约中增加死循环。将`set()`函数修改为：
```
function set(uint x) public {
  while(true) {
    myVariable = x;
  }
}
```

1. 在Truffle开发控制台中，更新合约：
```
migrate --reset
```
这将触发编译和迁移。
2. 为了显示调试信息，可以另外启动一个控制台专门显示日志。在另外的终端中执行：
```
truffle develop --log
```
3. 执行`set()`函数：
```
SimpleStorage.deployed().then(function(instance){return instance.set(4);});
Error: VM Exception while processing transaction: out of gas
```
这是因为死循环耗尽了燃料(gas)，导致报告`out of gas`错误。  


4. 通过调试定位bug位置。调试的命令格式是`debug <Transaction ID>`。
```
debug 0xe493340792ab92b95ac40e43dca6bc88fba7fd67191989d59ca30f79320e883f
```
注意把交易ID替换成你自己的。
看到的提示如下：
```
   Gathering transaction data...

   Addresses affected:
     0x377bbcae5327695b32a1784e0e13bedc8e078c9c - SimpleStorage

   Commands:
   (enter) last command entered (step next)
   (o) step over, (i) step into, (u) step out, (n) step next
   (;) step instruction, (p) print instruction, (h) print this help, (q) quit

   Store.sol | 0x377bbcae5327695b32a1784e0e13bedc8e078c9c:

   1: pragma solidity ^0.4.17;
   2:
   3: contract SimpleStorage {
      ^^^^^^^^^^^^^^^^^^^^^^^

   debug(develop:0xe4933407...)>
```
不停按回车键可以进行单步执行合约。直到找到bug位置。

### 以太坊宠物商店
[英文原文](http://truffleframework.com/tutorials/pet-shop)  
本教程将带您完成第一个dapp——一个宠物商店的收养跟踪系统！

本教程面向那些具有以太坊和智能合约基础知识的人员，他们具有HTML和JavaScript的一些知识，但对于dapps是新手。

*注意：对于以太坊的基础知识，请在继续之前阅读Truffle的[以太坊概述](https://github.com/wbwangk/wbwangk.github.io/wiki/Ethereum#%E4%BB%A5%E5%A4%AA%E5%9D%8A%E6%A6%82%E8%BF%B0) 。*

在本教程中，我们将涵盖：

- 建立开发环境  
- 使用Truffle Box创建Truffle项目  
- 写智能合约  
- 编译和迁移智能合约  
- 测试智能合约  
- 创建用户界面以与智能合约进行交互  
- 在浏览器中与dapp交互  

#### 背景
皮特宠物店的皮特·斯坎德隆（Pete Scandlon）有意使用以太坊作为一种有效的方式来处理宠物的收养。商店在给定的时间有16个宠物的空间，他们已经有一个宠物数据库。作为一个概念的初步证明，**Pete希望看到一个将以太坊地址与宠物关联起来的dapp**。

将提供网站结构和样式。**我们的工作是为其使用编写智能合同和前端逻辑**。

#### 建立开发环境
假定git、nodejs、npm已经装好。安装Truffle：
```
npm install -g truffle
```
执行`truffle version`来验证Truffle是否安装好。
我们也将使用[Ganache](http://truffleframework.com/ganache) ，这是一个个人区块链，可用于部署合约、开发应用程序和运行测试。您可以通过导航到[这里](http://truffleframework.com/ganache) 下载Ganache 。

*注意：如果您在没有图形界面的环境中开发，还可以使用Truffle Develop，即Truffle的内置个人区块链代替Ganache。您将需要更改一些设置（例如区块链运行的端口），以便适应Truffle Develop教程。*

#### 使用truffle创建项目
本教程使用了一个特殊的Truffle Box `pet-shop`，包括基本的项目结构以及用户界面的代码。

1. 创建工作目录；2. 下载`pet-shop`(在本机的~/geth目录下)
```
mkdir pet-shop-tutorial && cd pet-shop-tutorial
truffle unbox pet-shop
```
*注：Truffle可以通过几种不同的方式进行初始化。另一个有用的初始化命令是truffle init创建一个空的Truffle项目，不包含示例合约。有关更多信息，请参阅[创建项目](http://truffleframework.com/docs/getting_started/project) 的文档。*

【目录结构】
默认的Truffle目录结构包含以下内容：

- `contracts/`：包含我们的智能合约的Solidity源文件。这里有一个重要的合约Migrations.sol，我们稍后再谈。

- `migrations/`：Truffle使用迁移系统来处理智能合同部署。迁移是追踪变化的额外特殊智能合约。

- `test/`：包含我们智能合约的JavaScript和Solidity测试

- `truffle.js`：Truffle配置文件

`pet-shop`Truffle Box有额外的文件和文件夹，但我们现在还不担心这些。

#### 编写智能合约
我们通过编写处理后台逻辑和存储的智能合约来开始我们的dapp。

1. 在`contracts/`目录下创建一个叫`Adoption.sol`的文件。2. 添加下列内容：
```solidity
pragma solidity ^0.4.17;

contract Adoption {

}
```
- `pragma`命令意味着这是“只有编译器关心的附加信息”，插入符号(^)表示不能低于这个版本号。  
- 就像JavaScript或PHP，语句用分号结束。  

【创建变量】
Solidity是一种静态类型的语言，意味着像字符串，整数和数组等数据类型必须被定义。**Solidity有一个称为地址的独特类型**。地址是以太坊地址，存储为20个字节的值。以太坊区块链上的每个账户和智能合约都有一个地址，并可以通过此地址发送和接收以太币。
1. 之后在下一行添加以下变量`contract Adoption {`。
```
address[16] public adopters;
```
注意事项：

- 我们已经定义了一个变量：`adopters`。这是一个以太坊地址**数组**。数组包含一个类型，可以有一个固定的或可变的长度。在这种情况下，类型是地址`address`，长度是`16`。

- 你可能已注意到`adopters`是公共的。**公共**变量具有自动getter方法，但是在数组的情况下，键是必需的并且只会返回一个值。稍后，我们将编写一个函数来返回整个数组，供我们的用户界面使用。

#### 你的第一个函数：收养一个宠物
让我们允许用户做出收养请求。

在上面设置的变量声明之后，将下面的函数添加到智能合约中。
```
// Adopting a pet
function adopt(uint petId) public returns (uint) {
  require(petId >= 0 && petId <= 15);

  adopters[petId] = msg.sender;

  return petId;
}
```
注意事项：

- 在Solidity中，必须指定函数参数和输出的类型。在这里，我们将输入`petId`（整数）并返回一个整数。

- 我们正在检查确保`petId`在我们`adopters`数组的范围内。Solidity数组索引为0，因此ID值将需要在0到15之间。我们使用`require()`语句来确保ID在范围内。

- 如果ID在范围内，就添加调用者地址到adopters数组。*调用这个函数的人或智能合约的地址用`msg.sender`表示*。

- 最后，我们返回`petId`以便提供一个确认。

#### 你的第二个函数：获取收养者
如上所述，数组getter只从给定的键返回一个单一的值。我们的UI需要更新所有的宠物收养状态，但是做16个API调用并不理想。所以我们下一步是编写一个函数来返回整个数组。

在我们上面添加的`adopt()`函数之后，添加以下函数`getAdopters()`到智能合约：
```
// Retrieving the adopters
function getAdopters() public view returns (address[16]) {
  return adopters;
}
```
因为`adopters`已经声明，我们可以简单地返回它。确保指定返回类型（在这里，类型为`adopters`）为`address[16]`。

#### 编译和迁移智能合约
现在我们已经编写了智能合约，接下来的步骤是编译和迁移它。

Truffle有一个内置的开发者控制台，我们称之为Truffle Develop，它生成一个开发区块链，我们可以用来测试部署合约。它还能够直接从控制台运行Truffle命令。在本教程中我们将使用Truffle Develop执行我们合约中的大部分操作。

【编译】
Solidity是一种编译语言，这意味着我们需要将Solidity编译为用于以太坊虚拟机（EVM）执行的字节码。把它看作是将我们人类可读的Solidity翻译成EVM所理解的东西。

1. 在一个终端窗口中，在dapp的根目录中输入：
```
truffle compile
```

【迁移】
现在已经成功编译了合约，现在是时候将它们迁移到区块链了！

迁移是一个部署脚本，旨在改变应用程序合约的状态，将其从一个状态转移到另一个状态。对于第一次迁移，您可能只是部署新的代码，但随着时间的推移，其他迁移可能会移动数据或替换成一个新的合约。

*注意：在[Truffle文档](http://truffleframework.com/docs/getting_started/migrations) 中了解更多关于迁移的信息*。

你会在`migrations/`目录看到一个JavaScript文件：`1_initial_migration.js`。这将处理部署`Migrations.sol`合约以观察后续的智能合同迁移，并确保我们未来不会双重迁移不变的合同。

现在我们准备创建我们自己的迁移脚本。

1. 在`migrations/`目录中创建一个名为`2_deploy_contracts.js`的新文件。

2. 将以下内容添加到`2_deploy_contracts.js`文件中：
```
var Adoption = artifacts.require("Adoption");

module.exports = function(deployer) {
  deployer.deploy(Adoption);
};
```
3. 在我们可以将合约迁移到区块链之前，我们需要先运行区块链。在原文中使用的是图形界面的[Ganache](http://truffleframework.com/ganache) 。对于没有图形界面的服务器环境，使用[ganache-cli](https://github.com/trufflesuite/ganache-cli/blob/master/README.md)（以前叫testrpc）。  

安装ganache-cli: `npm install -g ganache-cli`。

在项目的根目录下启动ganache-cli：
```
$ ganache-cli
...
Listening  on localhost:8545
```
这样就在8545端口上启动了一个以太坊的客户端模拟程序。  
屏幕上会显示一些ganache-cli用于测试的内置账户地址和私钥。最后显示了钱包信息，其中的Mnemonic是12个字符串。根据这12个字符串可以重建钱包，很重要相当于私钥。

4. 将合约迁移到区块链。
```
$ truffle migrate
Using network 'development'.
...
```
5. 在Ganache-cli中，显示了整个迁移过程，区块号从0开始增加到了4。不同于Ganache的图形界面可以看到以太币的余额，从而观察到迁移的交易成本。之后想办法用其他方式观察账户余额的变化。

#### 测试智能合约
在智能合约测试方面，Truffle非常灵活，因为测试可以用JavaScript或Solidity编写。在本教程中，将在Solidity中编写我们的测试。

（整个测试代码的编写过程省略，请参考原文。）

【运行测试】
1. 在终端窗口中执行：
```
truffle test
```
2. 如果所有的测试都通过了，你会看到类似这样的控制台输出：
```
...
  3 passing (670ms)
```
如果在ganache-cli的终端窗口中，可以看到整个测试过程，区块高度从4长到了14。  

#### 创建一个用户界面与智能合约交互
现在，我们已经创建了智能合约，将其部署到我们的本地测试区块链中，并确认我们可以通过控制台与它进行交互，现在是时候创建一个UI，让Pete有一些东西可以用于他的宠物店！

包括在`pet-shop`Truffle Box是应用程序的前端代码。该代码存在于该`src/`目录中。

前端不使用构建系统（webpack，grunt等）尽可能简单地开始。该应用程序的结构已经在那里; 我们将填补以太坊特有的函数。这样，您可以将这些知识应用于您自己的前端开发。
（后略）

#### 安装和配置MetaMask
[MetaMask](https://metamask.io/) 是Chrome和Firebox的浏览器扩展，是一个以太坊钱包。通过MetaMask可以连接上以太坊主网或测试网进行以太坊账户的管理，包括查看余额、转账等，支持代币。在本文的[其它章节](https://github.com/wbwangk/wbwangk.github.io/wiki/Ethereum#%E5%8F%91%E8%A1%8C%E8%87%AA%E5%B7%B1%E7%9A%84%E4%BB%A5%E5%A4%AA%E5%9D%8Aerc20-token) 中也讲到了MetaMask的使用。  

1. 在浏览器中安装MetaMask（需要墙）。2-4. 略

5.6. 在MetaMask浮动框中，点击**Import Existing DEN**。输入ganache-cli刚运行时提示的12个英文单词。
![](http://truffleframework.com/tutorials/images/pet-shop/metamask-initial.png)  
在下面输入两次密码，点击**OK**。

7. 在MetaMask浮动框的左上角下拉框中选择**Custom RPC**。

8. 输入地址`http://127.0.0.1:8545`，点**Save**按钮。这里的端口号和原文不一样。我这里ganache-cli启动时的默认端口是8545，而不是7545。  
下拉框上的网络名称切换成了"Private Network"。

9. 点一下"Settings"左侧的黑色箭头回到账户页面。ganache-cli为每个账户设定的以太币初始值是100。你会注意到它在第一个账户上稍微少一些，因为当合同本身被部署时以及测试运行时使用了一些燃料。
![](http://truffleframework.com/tutorials/images/pet-shop/metamask-account1.png) 

#### 安装和配置lite-server
我们现在可以启动一个本地web服务器并使用dapp。我们使用`lite-server`库来服务于静态文件。这个`lite-server`已经包含在`pet-shop`Truffle Box中，但让我们看一下它如何工作的。

1. 打开项目根目录下的`bs-config.json`，可以看到下列内容：
```
{
  "server": {
    "baseDir": ["./src", "./build/contracts"]
  }
}
```
这告诉`lite-server`哪些文件包含在我们的基本目录中。我们为网站文件添加`./src`目录，为合约工件添加`./build/contracts`目录。

我们还在项目根目录下的`package.json`文件中的`scripts`对象添加了一个`dev`命令。该`scripts`对象允许我们将控制台命令别名为单个npm命令。在这种情况下，我们只是做一个单一的命令，但可能有更复杂的配置。这是你的应该看起来像：
```
"scripts": {
  "dev": "lite-server",
  "test": "echo \"Error: no test specified\" && exit 1"
},
```
这告诉npm，当我们在控制台中执行`npm run dev`时运行本地安装的`lite-server`。

1. 启动本地web服务器：
```
npm run dev
```
按说dev服务器会运行并自动打开一个新浏览器标签页来展现你的dapp。但我运行的ubuntu16没有图形界面，只是在屏幕上提示了四个URL：
```
      Local: http://localhost:3000
   External: http://10.0.2.15:3000
         UI: http://localhost:3001
UI External: http://10.0.2.15:3001
```
VM的3000和3001端口已经通过NAT映射到了windows。在windows下用浏览器打开地址`http://localhost:3000`可以看到如下的宠物商店页面：
![](http://truffleframework.com/tutorials/images/pet-shop/dapp.png) 
（3001端口应是lite-server的管理控制台）  

2. 为了使用dapp，点击选择宠物图片下的**Adopt**按钮。  

3. 你会被MetaMask自动提示来批准交易。点击**Submit**按钮来批准交易。
![](http://truffleframework.com/tutorials/images/pet-shop/metamask-transactionconfirm.png) 
（通过Foxfire和Chrome的MetaMask插件都试了，无法自动提示批准交易。可能与NAT有关）

4. 被收养宠物旁边的按钮变成了“Sucess”和被禁用，说明这个宠物已经被收养。  
![](http://truffleframework.com/tutorials/images/pet-shop/dapp-success.png)

5. 在MetaMask中，你可以看到刚刚的交易：
![](http://truffleframework.com/tutorials/images/pet-shop/metamask-transactionsuccess.png) 

您还可以在Ganache中看到在列出的同一交易。

恭喜！你已经迈出了一大步，成为一个成熟的dapp开发者。为了在本地进行开发，您可以使用所有工具开始制作更高级的dapp。如果您希望让您的dapp能够让其他人使用，请继续关注我们将来部署到Ropsten测试网络的教程。

### 为QUORUM构建DAPP：企业私有链
Quorum是增加了新特性的以太坊版本。特别是，**Quorum提供了在选定参与者之间建立私有链的能力，更重要的是在以太坊交易之上增加了交易隐私性。**

交易隐私提供了许多有用的用例，特别是在企业和银行业中。例如，大型银行可能希望利用区块链技术（如以太坊）的优势，但不希望他们的交易向所有人公开。Quorum提供了一个有用的额外选择。

让我们用一个简单的例子：说鲍勃、汤姆和艾丽斯都一起创建了区块链，爱丽丝想要发送20个TruffleCoin给鲍勃。但是这里有一个要求：她不希望汤姆（或者鲍勃以外的任何人）知道，因为她关心她的隐私。使用Quorum，艾丽斯可以轻松发送一个交易，而交易数据仅她和Bob可见。

本教程代表了Truffle对Quorum的官方支持。在本教程结束时，您将学习如何将Truffle和Quorum一起使用来构建启用隐私功能的dapps。

#### 需求
本教程希望您了解Truffle、以太坊、Quorum和Solidity的一些知识。有关这些主题的更多信息，请参阅以下链接：

- [Truffle文档](http://truffleframework.com/docs/) 

- [以太坊概览](https://github.com/wbwangk/wbwangk.github.io/wiki/Ethereum#toc15) 

- [Quorum概览](https://www.jpmorgan.com/country/US/EN/Quorum) 和[文档](https://github.com/jpmorganchase/quorum/wiki) 

- [Solidity文档](https://solidity.readthedocs.io/en/develop/) 
您将主要使用本教程的命令行。请确保您已熟悉打开和使用操作系统提供的命令行。此外，在继续之前，您需要安装以下软件：

- VirtualBox

- Vagrant

- Git


#### 入门
在本教程中，我们将向您展示如何使用Truffle和Quorum的[7节点](https://github.com/jpmorganchase/quorum-examples/tree/master/examples/7nodes) 示例为Quorum开发dapps 。步骤如下：

- 设置您的Quorum客户端

- 将Truffle连接到 Quorum

- 在Quorum上部署智能合约

- 使用Quorum的隐私功能使交易保密

- 与合约互动时保持私密

您将会看到使用 Truffle进行Quorum开发就像开发以太坊区块链一样。Truffle支持Quorum开箱即用，并且为Ethereum公共区块链构建启用以太坊的以太网应用程序的策略和方法也适用于在Quorum上构建dapps。

#### 设置您的Quorum客户端
Quorum客户端是以太坊客户端的替代品。使用Quorum客户端，您可以设置私人区块链，该区块链只对您和您允许参与的人员可用。

我们将使用一个由七个节点组成的Quorum集群（所以有七个Quorum客户端）已经在虚拟机内为我们设置和配置。您可以选择[直接下载](https://github.com/jpmorganchase/quorum) 并从源代码构建Quorum ，但对于此示例，使用预配置的群集要容易得多。

1. 要建立群集，请打开终端并导航到您想要安装的目录。在这里，我们选择了目录名为`workspace`：
```
mkdir workspace && cd workspace
```

2. 下载Quorum示例库、启动虚拟机、登录虚拟机：
```
git clone https://github.com/jpmorganchase/quorum-examples
cd quorum-examples && vagrant up
vagrant ssh
```

3. 进入示例目录、启动Quorum客户端：
```
$ cd quorum-examples/7nodes/
$ ./raft-init.sh
$ ./raft-start.sh
```
第一个命令创建了7个Quorum节点，用于模拟一个实际的Quorum部署。第而个命令启动这7个节点。第一个命令只需要执行一次，你可以执行第二个命令多次来重启虚拟机。

成功！我们现在建立了7个Quorum节点，我们可以使用它们在私有网络中代表7个不同的角色。


#### 连接Truffle到Quorum
为了建立Truffle，我们创建一个没有任何合约或代码的空Truffle项目。

1. 旧的命令行窗口运行着Quorum示例，为了不影响它运行我们启动第二个终端窗口。

2. 在新的终端中，导航到工作区并为Truffle项目创建一个新目录。初始化空的Truffle项目：
```
cd workspace && mkdir myproject
cd myproject && truffle init
```

3. 在转到编码前，我们需要配置Truffle来指向我们正在运行的Quorum客户端。在本例中，我们将编辑`truffle.js`中的`development`网络配置，以指向7节点示例中可用的第一个节点：
```javascript
// File: `truffle.js` (edited for 7nodes example)
module.exports = {
  networks: {
    development: {
      host: "127.0.0.1",
      port: 22000, // was 8545
      network_id: "*", // Match any network id
      gasPrice: 0,
      gas: 4500000
    }
  }
};
```
请注意，我们更改了Truffle通常连接到的端口（我们将其更改为22000）。由于VirtualBox和Vagrant的魔力，虚拟机内运行的节点通过本地端口提供给我们，所以通过连接127.0.0.1和22000将工作得很好。

*注意：七个Quorum客户端监听的端口是从22000（节点1）通过22006（节点7）。*

现在已经建立了Truffle，我们可以开始编码了。

#### 在Quorum上部署智能合约
由于我们有[充足的文档](http://truffleframework.com/docs) ，因此我们不会花太多时间在Truffle上编写或部署合约，但是我们想要展示如何将部署合约到Quorum。

1. 首先，将以下合约复制到称为`SimpleStorage.sol`的新文件中。将其放置在您的`contracts/`目录中：
```solidity
// File: `./contracts/SimpleStorage.sol`

pragma solidity ^0.4.17;

contract SimpleStorage {
  uint public storedData;

  function SimpleStorage(uint initVal) public {
    storedData = initVal;
  }

  function set(uint x) public {
    storedData = x;
  }

  function get() constant public returns (uint retVal) {
    return storedData;
  }
}
```
2. 确保在项目目录中用`truffle compile`命令编译合约。
```
truffle compile
```
3. 接下来，在您的`migrations/`目录中创建一个新的迁移`2_deploy_simplestorage.js`。请注意，此迁移与您为Truffle创建的任何其他迁移一样，但有一个重要区别。我们来看看你能否抓住它。
```javascript
// File: `./migrations/2_deploy_simplestorage.js`

var SimpleStorage = artifacts.require("SimpleStorage");

module.exports = function(deployer) {
  // Pass 42 to the contract as the first constructor parameter
  deployer.deploy(SimpleStorage, 42, {privateFor: ["ROAZBWtSacxXQrOe3FGAqJDyJjFePR5ce4TSIzmJ0Bc="]})
};
```
如果你猜到差异是`privateFor`，那么你抓住它了！`privateFor`是添加到Quorum中的额外事务参数，它指定您正在进行的交易（在本例中为合约部署）对于由给定公钥标识的特定帐户是私有的。对于这个交易，我们选择的公钥代表节点7。由于我们以前将Truffle配置为连接到节点1，因此此交易将从节点1部署合约，使交易在节点1和节点7之间保密。

4. 现在是部署合约的时候了。运行`truffle migrate`命令并注意您的合约已成功部署：
```
truffle migrate
```
现在合约已经部署了，它已经开始工作了。

#### 使用Quorum的隐私功能使交易保密
我们最初将Truffle配置为将开发环境指向示例提供的七个节点中的第一个。您可以将第一个节点视为“我们”，就好像我们正在为多个其他方使用的专用网络开发dapp一样。由于7节点示例为我们提供了7个节点，我们可以告诉Truffle其他节点，以便我们可以“成为”其他节点，并确保我们部署的合约是私有的。

1. 要添加新的网络配置，请再次编辑`truffle.js`文件并添加其他网络配置，并选择最能描述您所添加的网络连接的网络名称。在这种情况下，我们将添加一个连接到节点4（nodefour）和节点7（nodeseven）：
```javascript
// File: `truffle.js`
module.exports = {
  networks: {
    development: {
      host: "127.0.0.1",
      port: 22000, // was 8545
      network_id: "*", // Match any network id
      gasPrice: 0,
      gas: 4500000
    },
    nodefour:  {
      host: "127.0.0.1",
      port: 22003,
      network_id: "*", // Match any network id
      gasPrice: 0,
      gas: 4500000
    },
    nodeseven:  {
      host: "127.0.0.1",
      port: 22006,
      network_id: "*", // Match any network id
      gasPrice: 0,
      gas: 4500000
    }
  }
};
```
和以前一样，VirtualBox和Vagrant通过本地端口让我们可以使用这些节点，所以`development`除了指定不同的端口之外，这些配置看起来与我们的配置相同。

2. 现在我们的配置已经建立，我们可以使用Truffle控制台与我们已部署的合约进行交互。首先通过配置`development`配置，让我们成为“我们”（节点1）。最简单的方法是启动Truffle控制台，它可以让我们直接与已部署合约进行交互。
```
truffle console
truffle(development)>
```
3. 在这里，我们将获得已部署`SimpleStorage`合约的实例，然后获取在部署时指定的整数值。输入以下命令：
```
truffle(development)> SimpleStorage.deployed().then(function(instance) { return instance.get(); })
{ [String: '42'] s: 1, e: 1, c: [ 42 ] }
```
请注意， Truffle的合约抽象使用Promise与以太坊进行互动。这在控制台上可能有点麻烦，因为它需要一些额外的按键来完成任务，但在您的应用程序中，它使控制流程更顺畅。另外，看看我们收到的输出：我们取回了`42`，但作为一个对象。这是因为以太坊可以表示比原生JavaScript更大的数字，所以我们需要一个抽象来与它们进行交互。

4. 现在让我们尝试作为节点四访问SimpleStorage合约。为此，请退出控制台（使用`Ctrl + C`/ `Command + C`），然后再次启动控制台，但是这次指定连接到节点4而不是节点1：
```
truffle console --network nodefour
truffle(nodefour)>
```
5. 运行与上面相同的命令来获取SimpleStorage合约中的整数值：
```
truffle(nodefour)> SimpleStorage.deployed().then(function(instance) { return instance.get(); })
{ [String: '0'] s: 1, e: 0, c: [ 0 ] }
```
你会注意到这次取回的是`0`。这是因为由节点4代表的账户不是这个合约的参与者。

6. 最后，我们可以尝试节点7，它是这个合约的参与者。再次退出并重新启动控制台：
```
truffle console --network nodeseven
truffle(nodeseven)>
```

7. 运行与上面相同的命令以获取SimpleStorage合约中的整数值：
```
truffle(nodeseven)> SimpleStorage.deployed().then(function(instance) { return instance.get(); })
{ [String: '42'] s: 1, e: 1, c: [ 42 ] }
```
正如你所看到的，我们得到了`42`！这表明我们部署的合约可以仅对指定的参与方可见。

#### 私下与合约互动
到目前为止，我们已经向您展示了如何在迁移中部署私有的合约。在Quorum上构建dapp时，了解如何使所有交易私有化也很有帮助。

在JavaScript中使用合约的地方，Truffle使用[truffle-contract](https://github.com/trufflesuite/truffle-contract) 抽象。例如，当您在上面的控制台中与`SimpleStorage`进行交互时，您正在使用`truffle-contract`合约抽象。这些抽象也用于您的迁移、基于JavaScript的单元测试以及使用Truffle执行外部脚本。

Truffle的合约抽象允许您针对合约中可用的任何函数创建一个交易。它通过评估合约的函数并将它们提供给JavaScript来实现。要看到这些交易正在进行，我们将使用Truffle的一项高级功能，让我们在我们的Truffle环境中执行外部脚本。

1. 创建一个叫`sampletx.js的文件并将其保存到项目的根目录（与truffle.js文件相同的目录）。然后用下面的代码填充它：
```javascript
var SimpleStorage = artifacts.require("SimpleStorage");

module.exports = function(done) {
  console.log("Getting deployed version of SimpleStorage...")
  SimpleStorage.deployed().then(function(instance) {
    console.log("Setting value to 65...");
    return instance.set(65, {privateFor: ["ROAZBWtSacxXQrOe3FGAqJDyJjFePR5ce4TSIzmJ0Bc="]});
  }).then(function(result) {
    console.log("Transaction:", result.tx);
    console.log("Finished!");
    done();
  }).catch(function(e) {
    console.log(e);
    done();
  });
};
```
这段代码做了两件事：首先，它要求Truffle获得SimpleStorage合约的合约抽象。然后，它找到已部署的合约，并使用合约的`set()`函数将SimpleStorage管理的值设置为`65`。与我们先前写的迁移一样，可以在交易结束时将参数`privateFor`添加到对象中，以告诉Quorum该交易在发起人和给定公钥代表的账户之间是私密的。

2. 使用`truffle exec`运行此代码：
```
truffle exec sampletx.js
```
你的输出应该看起来像这样（你的交易ID将会不同）：
```
Using network 'development'.

Getting deployed version of SimpleStorage...
Setting value to 65...
Transaction: 0x0a7a661e657f5a706b0c39b4f197038ef0c3e77abc9970a623327c6f48ca9aff
Finished!
```
我们现在可以像以前一样使用Truffle控制台来检查此交易的结果。让我们作为节点1看看这个值：
```
truffle console
truffle(development)> SimpleStorage.deployed().then(function(instance) { return instance.get(); })
{ [String: '65'] s: 1, e: 1, c: [ 65 ] }
```
我们取到了`65`！现在让我们作为节点4来运行（不是交易的参与者）：
```
truffle console --network nodefour
truffle(nodefour)> SimpleStorage.deployed().then(function(instance) { return instance.get(); })
{ [String: '0'] s: 1, e: 0, c: [ 0 ] }
```
如预期的那样，我们得到了`0`。现在让我们尝试节点7：
```
truffle console --network nodeseven
truffle(nodeseven)> SimpleStorage.deployed().then(function(instance) { return instance.get(); })
{ [String: '65'] s: 1, e: 1, c: [ 65 ] }
```
正如我们所期望的那样，我们又得到了`65`。这就是我们如何使用Truffle的合约抽象与Quorum进行私下交易。

#### 这是所有Truffle都可以做到的吗？
绝对不是！我们今天向您展示的是让Quorum构建dapps不同于为共有以太坊构建dapps的一切。而且你会发现它并没有什么不同：唯一的区别是为你希望保密的部署和交易添加`privateFor`参数。其余的都是一样的！

事实上，现在您已经掌握了基本知识，您可以探索我们所有其他用Truffle构建dapp的[资源](http://truffleframework.com/docs)，包括[教程](http://truffleframework.com/tutorials) 、[编写高级部署脚本](http://truffleframework.com/docs/getting_started/migrations) 、[单元测试](http://truffleframework.com/docs/getting_started/javascript-tests) （[还包括Solidity](http://truffleframework.com/docs/getting_started/solidity-tests) ）等等。

通过使用Truffle进行构建，您现在不仅可以访问最佳开发工具和技术（如Quorum），还可以访问最大的以太坊开发者社区。不要犹豫，[给我们一条线](https://gitter.im/ConsenSys/truffle) 或从[社区](https://gitter.im/ConsenSys/truffle) 获得帮助。总有人可以回答您的任何问题。

干杯，快乐的编码！

