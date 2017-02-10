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
以docker安装kubernetes的库是https://github.com/kubernetes/kube-deploy，这个库中使用了gcr.io和quay.io中docker镜像，国内用需要翻墙。将库/kubernetes/kube-deploy fork到/wbwang/kube-deploy，并对docker-multinode目录下的common.sh进行了修改。  
### 创建3个docker镜像：etcd,flannel和hyperkube
在docker-multinode目录下创建了3个目录，分别是etcd-amd64,flannel,hyperkube-amd64。在目录下分别创建Dockerfile。  
在hub.docker.com下创建3个autobuild库，分别对应到上述3个Dockerfile：
```
wbwang/etcd-amd64:3.0.4
wbwang/flannel:v0.6.1-amd64
wbwang/hyperkube-amd64:v1.5.2
```
为了提高脚本运行速度，可以把上述3个镜像手工拉下来，如：
```
docker pull wbwang/etcd-amd64:3.0.4
```
### 修改common.sh
```
gcr.io/google_containers/etcd-${ARCH}:${ETCD_VERSION} 修改为：
wbwang/etcd-${ARCH}:${ETCD_VERSION} 
```
```
quay.io/coreos/flannel:${FLANNEL_VERSION}-${ARCH} 修改为：
wbwang/flannel:${FLANNEL_VERSION}-${ARCH}  
```
```
gcr.io/google_containers/hyperkube-${ARCH}:${K8S_VERSION} 修改为：
wbwang/hyperkube-${ARCH}:${K8S_VERSION}
```
```
LATEST_STABLE_K8S_VERSION=$(curl -sSL "https://storage.googleapis.com/kubernetes-release/release/stable.txt") 修改为：
LATEST_STABLE_K8S_VERSION=v1.5.2
```
设置环境变量，编辑一个env.sh：
```
export K8S_VERSION=v1.5.2
export ETCD_VERSION=3.0.4
export FLANNEL_VERSION=0.6.1
#export FLANNEL_IPMASQ=true
#export FLANNEL_NETWORK='10.1.0.0/16'
#export FLANNEL_BACKEND=udp
#export RESTART_POLICY=unless-stopped
#export MASTER_IP=192.168.1.140
#export ARCH=amd64
#export NET_INTERFACE=enp0s8
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
## vagrant部署单节点kubernetes
之前部署vagrant(如[这个文档](https://github.com/wbwangk/wbwangk.github.io/wiki/virtualbox-vagrant-gitbash%E5%85%A5%E9%97%A8))都是在windows下，而利用windows下按[这个文档](https://coreos.com/kubernetes/docs/latest/kubernetes-on-vagrant-single.ubuntu)kubernetes，碰到各种问题，无法解决。  
后来，尝试在纯linux(ubuntu)下部署。找了一台上网本，升级到ubuntu16。这台机器下称宿主机。首先在宿主机下安装vagrant和virtualbox，比windows下简单：
```
$ apt install vagrant virtualbox git docker.io -y
```
然后下载单节点kubernetes部署程序：
```
$ cd /opt
$ git clone https://github.com/coreos/coreos-kubernetes.git
$ cd coreos-kubernetes/single-node/
```
修改配置文件Vagrantfile，将NODE_ID修改为：10.10.250.190。而宿主机的IP是10.10.250.199，这样宿主机与虚拟机可以互相通信。
```
$ vagrant up
$ vagrant ssh
Last login: Thu Feb  9 06:24:30 UTC 2017 from 10.0.2.2 on ssh
Container Linux by CoreOS alpha (1313.0.0)
Failed Units: 3
  coreos-cloudinit-173822746.service
  flannel-docker-opts.service
  update-engine.service
core@localhost ~ $
```
vagrant up命令启动一个基于box'coreos-alpha'的虚拟机。vagrant ssh登录新启动的虚拟机。可以通过命令查看服务失败的原因：
```
$ systemctl status flannel-docker-opts.service
```
根据提示，是从quay.io下载东西失败，翻墙后重新启动服务成功：
```
$ systemctl restart flannel-docker-opts.service
```
用命令```ip addr```查看各网卡ip：
```
eth0: 10.0.2.15
eth1:10.10.250.190
flannel:10.2.67.0/32
```
返回宿主机操作系统(ubuntu16)，安装kubectl：
```
$ cd /opt
$ curl -O https://storage.googleapis.com/kubernetes-release/release/v1.5.2/bin/linux/amd64/kubectl
$ chmod +x kubectl
$ mv kubectl /usr/local/bin/kubectl
```
配置kubectl：
```
$ export KUBECONFIG="${KUBECONFIG}:$(pwd)/kubeconfig"
$ kubectl config use-context vagrant-single
$ kubectl config set-cluster vagrant-single-cluster --server=https://10.10.250.190:443 --certificate-authority=${PWD}/ssl/ca.pem
$ kubectl config set-credentials vagrant-single-admin --certificate-authority=${PWD}/ssl/ca.pem --client-key=${PWD}/ssl/admin-key.pem --client-certificate=${PWD}/ssl/admin.pem
$ kubectl config set-context vagrant-single --cluster=vagrant-single-cluster --user=vagrant-single-admin
$ kubectl config use-context vagrant-single
```