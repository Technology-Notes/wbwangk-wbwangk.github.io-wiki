要安装kubernetes要首先安装etcd(做服务发现)和flannel(虚拟网络)。

### 安装etcd
```
$ apt-get update
$ apt-get install etcd
$ etcd  (执行etcd，它会监听2379和2380)
$ etcdctl --version
etcdctl version 2.25
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
# 以docker方式安装kubernetes
设置环境变量，编辑一个env.sh：
```
export K8S_VERSION=v1.5.2
export ETCD_VERSION=2.2.5
export FLANNEL_VERSION=0.7.0
export FLANNEL_IPMASQ=true
export FLANNEL_NETWORK='10.1.0.0/16'
export FLANNEL_BACKEND=udp
export RESTART_POLICY=unless-stopped
export MASTER_IP=192.168.1.140
export ARCH=amd64
export NET_INTERFACE=enp0s8
```
执行env.sh:
```
$ . env.sh
$ env
```
下载部署脚本：
```
$ git clone https://github.com/kubernetes/kube-deploy
$ cd kube-deploy/docker-multinode
$ ./master.sh
```
注意：执行master.sh前需要先执行前面的env.sh，否则出错。  
根据master.sh执行时的屏幕提示发现需要几个docker镜像：
```
gcr.io/google_containers/etcd-amd64:2.2.5
quay.io/coreos/flannel:0.7.0-amd64
```
由于GFW的原因，上述镜像不能正常pull下来。好在可以在[阿里镜像库](https://cs.console.aliyun.com/#/repo)中可以找到类似的：
```
docker pull registry.cn-hangzhou.aliyuncs.com/kubernetes/etcd-amd64:2.2.5
docker pull registry.cn-hangzhou.aliyuncs.com/mykubernetes/flannel:v0.7.0-amd64
docker tag registry.cn-hangzhou.aliyuncs.com/kubernetes/etcd-amd64:2.2.5 \
gcr.io/google_containers/etcd-amd64:2.2.5
docker tag registry.cn-hangzhou.aliyuncs.com/mykubernetes/flannel:v0.7.0-amd64 \
quay.io/coreos/flannel:0.7.0-amd64
docker tag registry.cn-hangzhou.aliyuncs.com/kubernetes/etcd-amd64:2.2.5 \
kubernetes/etcd-amd64:latest
docker tag  registry.cn-hangzhou.aliyuncs.com/mykubernetes/flannel:v0.7.0-amd64 \
mykubernetes/flannel:latest
docker tag registry.cn-hangzhou.aliyuncs.com/kubernetes/etcd-amd64:2.2.5 \
google_containers/etcd-amd64:2.2.5
```