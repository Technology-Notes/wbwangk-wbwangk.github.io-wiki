# [以太坊中的梅克尔(Merkle)树](https://blog.ethereum.org/2015/11/15/merkling-in-ethereum/)

梅克尔树是区块链的基本组成部分。虽然从理论上讲，制作没有梅克尔树的区块链是可能的，方法是创建直接包含每个交易的巨大区块头，这样做会带来巨大的可扩展性挑战，只能将区块链长期置于最强大的电脑上。借助梅克尔树，可以构建在所有计算机和笔记本电脑上运行的以太坊节点，这些节点可以是大型和小型智能手机，甚至是物联网设备，例如由[Slock.it](http://slock.it/)生产的[设备](http://slock.it/)。那么这些梅克尔树究竟是如何工作的呢？他们现在和将来都会提供什么样的价值？

首先，基础知识。梅克尔树就一般意义上说是一种将大量“数据块”哈希在一起的方法，这种方式依赖于将块拆分成桶，其中每个桶只包含几个块，然后计算每个桶的哈希并重复相同的过程，继续这样做直到剩余的哈希总数变成只有一个：根哈希。

梅克尔树最常见和最简单的形式是二元梅克尔树，其中一个桶总是由两个相邻的块或哈希组成; 它可以描述如下：

![img](https://blog.ethereum.org/wp-content/uploads/2015/11/merkle.png)

那么这种奇怪的哈希算法有什么好处呢？为什么不把所有块连接在一起成为一个大块，并使用常规的哈希算法呢？答案是它允许一个称为梅克尔证明(Merkle proofs)的优雅机制：

![img](https://blog.ethereum.org/wp-content/uploads/2015/11/merkle2.png)

梅克尔证明由一个块（树的根哈希）和“分支”组成，分支由沿着从(叶)块到根的路径上的所有哈希组成。阅读证明的人可以验证哈希，至少对于那个分支来说与路径是一致的，因此给定块确实存在于树中的那个位置。该应用程序很简单：可以想象成一个大型数据库，数据库的全部内容都存储在一个梅克尔树中，梅克尔树的根是众所周知和可信的（例如，它是由足够的信任方进行数字签名的，或者有很多工作证明）。然后，想要在数据库上执行键值查找的用户（例如，“告诉我位置85273中的对象”）可以请求一个梅克尔证明，如果收到证明后验证为正确，则收到的值*确实*存在于该特定根的数据库中的位置85273处。这个机制允许从验证*小*的数据量（如哈希）扩展到验证*大*的潜在任意大小的数据库。

### 比特币中的梅克尔证明

梅克尔证据的最初应用是在比特币中，中本聪在2009年描述和创建。比特币区块链使用梅克尔证据来存储每个区块的交易：

![img](https://blog.ethereum.org/wp-content/uploads/2015/11/mining.jpg)

这提供的好处是Satoshi将其描述为“简化支付验证(SPV)”的概念：不用下载*每个*交易和每个区块，“轻型客户”只下载*区块头*链，每个区块80个字节的数据块只包含五件事情：

- 前一个区块头的哈希
- 时间戳
- 挖矿难度值
- 工作量证明nonce
- 包含该区块交易的梅克尔树的根哈希。

如果轻客户想要确定一个交易的状态，它可以简单地要求一个梅克尔证明，证明一个特定的交易存在于梅克尔树中，该梅克尔树的根保存在主链的区块头中。

这很管用，但比特币风格的轻客户端确实有其局限性。一个特别的限制是，尽管它们可以证明包含了该交易，但它们不能证明任何关于当前状态的信息（例如持有的数字资产、名称注册、金融合约状态等）。你现在有多少比特币？比特币轻客户端使用了一个协议：它查询多个节点，并相信其中至少一个节点会告诉您，您的地址向外发出的全部支付交易。这对于当前场景是够用的，但对于其他更复杂的应用这还远远不够; 交易的确定性影响可能取决于几项先前交易，而这些交易本身取决于更早的交易，所以最终你必须认证整个链中的每一笔交易。为了解决这个问题，以太坊将梅克尔树的概念更进一步。

### 以太坊中的梅克尔证明

以太坊中的每个区块头不仅包含一个梅克尔树，而是包含了*三个*树，对应三种类型的对象：

- 交易
- 收据（本质上是表明每笔交易带来的*影响*的数据）
- 状态

![img](https://blog.ethereum.org/wp-content/uploads/2015/11/ethblockchain_full.png)

这是一个很先进的轻客户端协议，允许轻客户端轻松制作和验证下面多种查询结果：

- 此交易是否已包含在特定区块中？
- 告诉我过去30天内由此地址发出的所有类型为X（例如，众筹合约达成目标）的事件实例
- 我账户的当前余额是多少？
- 这个账户是否存在？
- 模拟在这个合约上运行此交易。输出是什么？

第一个由交易树处理; 第三个和第四个由状态树处理，第二个由收据树处理。前四项计算相当简单; 服务器只需简单地查找对象、获取梅克尔分支（从对象到树根的哈希列表）和将分支回复到轻客户端。

第五个也是由状态树处理的，但它的计算方式更复杂。在这里，我们需要构建一个称为**梅克尔状态转移证明**的东西。从本质上讲，它是一个证明，该证明声称“如果你在根为`S`的状态上运行交易`T`，结果将是一个新状态，新状态的根为`S'`、带有日志`L`和输出`O`”（在“以太坊”中存在“输出”的概念，因为每个交易都是一个函数调用;这在理论上是不必要的）。

为了计算证明，服务器在本地创建一个模拟块，将状态设置为S，并在应用交易时假装成为轻客户端。也就是说，如果应用交易的过程要求客户端确定账户的余额，轻客户端会进行余额查询。如果轻客户需要检查特定合约的存储中的特定项目，则轻客户为此进行查询，等等。服务器正确回应所有自己的查询，但会跟踪它发回的所有数据。然后服务器向客户端发送来自所有这些请求的组合数据作为一个证明。然后客户进行完全相同的程序，但是*使用提供的证明作为其数据库*; 如果其结果与服务器声称的结果相同，则客户接受该证明。

![img](https://blog.ethereum.org/wp-content/uploads/2015/11/lightproof.png)

### 帕特里夏树

上面提到，最简单的梅克尔树是二元梅克尔树; 然而，以太坊使用的树更复杂 - 这是你在我们的文档中听到的“梅克尔帕特里夏(Patricia)树”。本文不会详细说明; 可以看[这篇文章](https://github.com/ethereum/wiki/wiki/Patricia-Tree)和[这一篇](https://easythereentropy.wordpress.com/2014/06/04/understanding-the-ethereum-trie/)，虽然我将讨论基本的推理。

对于验证处于“列表”格式的信息，二进制梅克尔树是非常好的数据结构; 从本质上讲，是一个块的系列，一个块紧跟另一个块。对于交易树，它们也是很好的，因为一旦树被创建，*编辑*树需要多少时间并不重要，因为树被创建一次，然后永久冻结。

但是，对于状态树，情况更为复杂。以太坊的状态基本上由一个键值映射组成，其中键是地址，值是账户声明，列出每个账户（存储本身就是树）的余额、随机数、代码和存储。例如，Morden testnet创世状态如下所示：

```
{
    "0000000000000000000000000000000000000001": {
        "balance": "1"
    },
    "0000000000000000000000000000000000000002": {
        "balance": "1"
    },
    "0000000000000000000000000000000000000003": {
        "balance": "1"
    },
    "0000000000000000000000000000000000000004": {
        "balance": "1"
    },
    "102e61f5d8f9bc71d0ad4a084df4e65e05ce0e1c": {
        "balance": "1606938044258990275541962092341162602522202993782792835301376"
    }
}
```

但是，与交易历史不同，状态需要频繁更新：账户的余额和随机数经常更改，而且新账户频繁插入，并且存储中的键经常被插入和删除。因此需要的是一种数据结构，我们可以在插入、更新或删除操作之后快速计算新的树根，而无需重新计算整个树。还有两个非常需要的次要属性：

- 树的深度有界限，即使攻击者故意制造交易使树尽可能深。否则，攻击者可以通过操纵树来执行拒绝服务攻击，使得每个更新变得非常缓慢。
- 树的根仅取决于数据，而不取决于更新的顺序。以不同的顺序进行更新，甚至从头开始重新计算树不应该改变根目录。

[帕特里夏树](https://en.wikipedia.org/wiki/Radix_tree)，简单来说，也许最接近于同时实现所有这些特性。关于它如何工作的最简单的解释是，存储值的键(key)被编码到必须取下树的“路径”中。每个节点有16个子节点，所以路径由十六进制编码决定：例如，key`dog`的十六进制编码是`6 4 6 15 6 7`，所以你将从根开始，沿着第六个子节点，然后是第四个子节点，依此类推，直到你到达结尾。在实践中，当树很稀疏时，我们可以做一些额外的优化来使过程更加高效，但这是基本原则。[上面](https://easythereentropy.wordpress.com/2014/06/04/understanding-the-ethereum-trie/)提到的两篇[文章](https://github.com/ethereum/wiki/wiki/Patricia-Tree) 更详细地描述所有功能。