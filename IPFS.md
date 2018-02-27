IPFS（InterPlanetary File System）是一个点对点的分布式超媒体分发协议，它整合了过去几年最好的分布式系统思路，为所有人提供全球统一的可寻址空间，包括Git、自证明文件系统SFS、BitTorrent和DHT，同时也被认为是最有可能取代HTTP的新一代互联网协议。

IPFS用基于内容的寻址替代传统的基于域名的寻址，用户不需要关心服务器的位置，不用考虑文件存储的名字和路径。我们将一个文件放到IPFS节点中，将会得到基于其内容计算出的唯一加密哈希值。哈希值直接反映文件的内容，哪怕只修改1比特，哈希值也会完全不同。当IPFS被请求一个文件哈希时，它会使用一个分布式哈希表找到文件所在的节点，取回文件并验证文件数据。

本文参考了官方原文[Getting Started](https://ipfs.io/docs/getting-started/)。  
#### 下载go-ipfs
在节点1(一个ubuntu16的VM)上执行：
```
mkdir ~/ipfs && cd ~/ipfs
wget https://dist.ipfs.io/go-ipfs/v0.4.13/go-ipfs_v0.4.13_linux-amd64.tar.gz
tar xzvf go-ipfs_v0.4.13_linux-amd64.tar.gz && rm go-ipfs_v0.4.13_linux-amd64.tar.gz && cd go-ipfs
```

### 快速开始
#### 初始化
```
sudo ./install.sh
ipfs init
initializing IPFS node at /home/vagrant/.ipfs
generating 2048-bit RSA keypair...done
peer identity: QmfMFY37oG4HLZrATjdfD1mrvv32p9DSr4zT23yMnfiF6Z
to get started, enter:
       ipfs cat /ipfs/QmS4ustL54uo8FzR9455qaxZwuMiUhyvMcX9Ba8nUH4uVv/readme
```
#### 添加文件到ipfs
```
echo "hello world" >hello.txt
ipfs add hello.txt
added QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o hello.txt
```

#### 启动ipfs服务
```
ipfs config Addresses.Gateway /ip4/0.0.0.0/tcp/8082
ipfs daemon
API server listening on /ip4/127.0.0.1/tcp/5001
Gateway (readonly) server listening on /ip4/0.0.0.0/tcp/8082
Daemon is ready
```
之所以执行ipfs config，是因为本地默认的8080端口被占用了。

另一个常见的可设置端口是Addresses.API:
```
ipfs config Addresses.API /ip4/0.0.0.0/tcp/5002
```
#### 远程访问本地文件
`QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o`是之前hello.txt的哈希值。下面在浏览器中访问这个文件：
```
curl https://ipfs.io/ipfs/QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o
hello world
```
我还特意找了一个VM安装ipfs，并添加同样内容（"hello world"）的文件到ipfs，发现hash是一样的。相信全球大量的人员在测试ipfs时都可能会使用"hello world"作为文件内容。  
所以当访问地址`https://ipfs.io/ipfs/QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o`时，访问的不见得是你自己的电脑上的这个hello.txt文件，除非这个文件的内容独特到全球只有你的电脑上有。  
文档`https://ipfs.io/ipfs/QmXZXP8QRMG7xB4LDdKeRL5ZyZGZdhxkkLUSqoJDV1WRAp`的内容比较独特，恐怕只有我自己的电脑上有，如果我不开机并启动ipfs进程，估计别人访问不了。

### API
注意：要使用IPFS HTTP API，必须先启动ipfs daemon。
IPFS的CLI与HTTP API是一一对应的。

例如，命令行`ipfs swarm peers`对应的HTTP API是：
```
curl http://127.0.0.1:5001/api/v0/swarm/peers
```
#### 参数
下面的命令行与HTTP API是一样的效果
```
ipfs swarm disconnect /ip4/54.93.113.247/tcp/48131/ipfs/QmUDS3nsBD1X4XK5Jo836fed7SErTyTuQzRqWaiQAyBYMP
curl "http://127.0.0.1:5001/api/v0/swarm/disconnect?arg=/ip4/54.93.113.247/tcp/48131/ipfs/QmUDS3nsBD1X4XK5Jo836fed7SErTyTuQzRqWaiQAyBYMP"
{
  "Strings": [
    "disconnect QmUDS3nsBD1X4XK5Jo836fed7SErTyTuQzRqWaiQAyBYMP success",
  ]
}
```
#### 标志(Flag)
命令行的标志（选项）通过查询参数的形式添加。如标志`--encoding=json`用查询参数`&encoding=json`：
```
curl "http://127.0.0.1:5001/api/v0/object/get?arg=QmaaqrHyAQm7gALkRW8DcfGX3u8q9rWKnxEMmf7m9z515w&encoding=json"
```

## 命令
```
ipfs config show
```
查看配置

```
ipfs id
```
查看本地节点的id、公钥、地址等

```
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Methods '["PUT", "GET", "POST", "OPTIONS"]'
ipfs config --json API.HTTPHeaders.Access-Control-Allow-Origin '["*"]'
```
CORS配置

#### cat ls
```
$ ipfs cat  /ipfs/QmS4ustL54uo8FzR9455qaxZwuMiUhyvMcX9Ba8nUH4uVv
Error: this dag node is a directory
$ ipfs ls  /ipfs/QmS4ustL54uo8FzR9455qaxZwuMiUhyvMcX9Ba8nUH4uVv
QmZTR5bcpQD7cFgTorqxZDYaew1Wqgfbd2ud9QqGPAkK2V 1688 about
QmYCvbfNbCwFR45HiNP45rwJgvatpiW38D961L5qAhUM5Y 200  contact
QmY5heUM5qgRubMDD1og9fhCPA6QdkMp3QCwd4s7gJsyE7 322  help
QmejvEPop4D7YUadeGqYWmZxHhLc4JBUCzJJHWMzdcMe2y 12   ping
QmXgqKTbzdh83pQtKFb19SpMCpDDcKR2ujqk3pKph9aCNF 1692 quick-start
QmPZ9gcCEpqKTo6aq61g2nXGUhM4iCL3ewB6LDXZCtioEB 1102 readme
QmQ5vhrL7uv6tuoN9KeVBwd4PwfQkXdVVmDLUZuTNxqgvm 1173 security-notes
$ ipfs cat /ipfs/QmPZ9gcCEpqKTo6aq61g2nXGUhM4iCL3ewB6LDXZCtioEB
(显示了readme文件的内容)
```
`cat`显示单个文件，`ls`显示文件夹。