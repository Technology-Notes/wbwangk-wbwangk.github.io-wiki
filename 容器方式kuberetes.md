以docker安装kubernetes的库是```https://github.com/kubernetes/kube-deploy```，这个库中使用了gcr.io和quay.io中docker镜像，国内用需要翻墙。将库/kubernetes/kube-deploy fork到/wbwang/kube-deploy，并对docker-multinode目录下的common.sh进行了修改。  
### 创建3个docker镜像：etcd,flannel和hyperkube
在docker-multinode目录下创建了3个目录，分别是etcd-amd64,flannel,hyperkube-amd64。在目录下分别创建Dockerfile。  
在hub.docker.com下创建3个autobuild库，分别对应到上述3个Dockerfile：
```
wbwang/etcd-amd64:3.0.4
wbwang/flannel:v0.6.1-amd64
wbwang/hyperkube-amd64:v1.5.2
```
为了提高脚本运行速度，可以把上述3个镜像手工拉下来:
```
docker pull wbwang/etcd-amd64:3.0.4
docker pull wbwang/flannel:v0.6.1-amd64
docker pull wbwang/hyperkube-amd64:v1.5.2
```
master节点需要拉取3个镜像，而worker节点只需要后两个。
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
### 部署master节点
利用vagrant启动master节点(ip 192.168.1.140)：
```
$ vagrant up k8s0
$ vagrant ssh k8s0
```
下载部署脚本：
```
$ cd /opt
$ git clone https://github.com/wbwangk/kube-deploy
$ cd kube-deploy/docker-multinode
$ ./master.sh
```
### 部署worker节点
利用vagrant启动worker节点(ip 192.168.1.141)：
```
$ vagrant up k8s1
$ vagrant ssh k8s1
```
测试一下主节点的etcd是否正常工作：
```
$ export MASTER_IP=192.168.1.140
$ curl $MASTER_IP:2379/version
{"etcdserver":"3.0.4","etcdcluster":"3.0.0"}
```
下载部署脚本：
```
$ cd /opt
$ git clone https://github.com/wbwangk/kube-deploy
$ cd kube-deploy/docker-multinode
$ ./worker.sh
```
