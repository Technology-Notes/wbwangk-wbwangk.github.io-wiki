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
flannel官方库位于：```https://github.com/coreos/flannel```，点击release标签可以下载二进制程序。 
```
$ mkdir /opt/flannel
$ cd /opt/flannel
$ wget https://github.com/coreos/flannel/releases/download/v0.7.0/
$ tar -xzf flannel-v0.7.0-linux-amd64.tar.gz && rm *.tar.gz
```
### 安装kubernetes
```
$ wget https://github.com/kubernetes/kubernetes/releases/download/v1.5.2/kubernetes.tar.gz
$ mkdir /opt/k8s && cd /opt/k8s
$ tar -xzf kubernetes.tar.gz
```