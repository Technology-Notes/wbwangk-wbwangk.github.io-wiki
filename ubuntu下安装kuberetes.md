要安装kubernetes要首先安装etcd(做服务发现)和flannel(虚拟网络)。

### 安装etcd
```
$ apt-get update
$ apt-get install etcd
$ etcd  (执行etcd，它会监听2379和2380)
$ netstat -an | grep 2380(或2379)  (可以看到etcd已经监听2379和2380)
```
按etcd的官方[README](https://github.com/coreos/etcd)来测试一下etcd：
```
$ ETCDCTL_API=3 etcdctl set mykey "this is awesome"
$ ETCDCTL_API=3 etcdctl get mykey
this is awesome
```
etcdctl是etcd自带的命令行工具。这里etcd只装了一个节点，在生产环境下etcd至少要装3个节点，以便保证高可用。  

### 安装flannel
