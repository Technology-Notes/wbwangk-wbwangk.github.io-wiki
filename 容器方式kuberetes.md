kubernetes官方提供了以容器方式部署kubernetes的代码库，地址是```https://github.com/kubernetes/kube-deploy```。只是在这个库中使用了gcr.io和quay.io中的3个docker镜像：
```
gcr.io/google_containers/etcd-amd64:3.0.4
quay.io/coreos/flannel:v0.6.1-amd64
gcr.io/google_containers/hyperkube-amd64:v1.5.2
```
这三个镜像国内使用需要翻墙。
我fork了一个地址是```https://github.com/wbwangk/kube-deploy```的库，做了些修改，以便国内使用。首先在库中制作3个Dockerfile，然后利用docker hub的autobuild自动构建镜像到docker hub。
### 3个Dockerfile
[etcd](https://github.com/wbwangk/kube-deploy/blob/master/docker-multinode/hyperkube-amd64/Dockerfile):
```
FROM gcr.io/google_containers/etcd-amd64:3.0.4
```
[flannel](https://github.com/wbwangk/kube-deploy/blob/master/docker-multinode/flannel/Dockerfile):
```
FROM quay.io/coreos/flannel:v0.6.1-amd64
```
[hyperkube](https://github.com/wbwangk/kube-deploy/blob/master/docker-multinode/hyperkube-amd64/Dockerfile):
```
FROM gcr.io/google_containers/hyperkube-amd64:v1.5.2
```
### 利用docker hub自动构建3个docker镜像：etcd,flannel和hyperkube
我在docker hub的账号是```wbwang```
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
手工pull上面提到的3个镜像可以提高master.sh执行成功的概率。
master.sh执行过程中可能会提示：
```
Do you want to clean /var/lib/kubelet? [Y/n]
```
如果选择Y，则会重新拉取docker镜像，一般选择n。如果执行过程中报错，如：
```
!!! [0213 00:02:55] flannel failed to start. Exiting...
```
重新执行master.sh，错误往往就没有了。
如果master.sh执行成功，会提示：
```
+++ [0213 00:04:26] Done. It may take about a minute before apiserver is up.
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
$ docker pull wbwang/flannel:v0.6.1-amd64
$ docker pull wbwang/hyperkube-amd64:v1.5.2
$ export MASTER_IP=192.168.1.140
$ ./worker.sh
```
如果节点上未安装docker引擎,可以用下列命令安装:
```
$ apt install docker.io
```
手工pull用到的两个docker镜像可以提高worker.sh运行成功的概率。实测手工pull镜像后，可以不用翻墙就成功运行worker.sh。  
注意运行worker.sh前，需要设置好MASTER_IP，并用curl测试主节点的etcd能否链接上。  

利用同样的办法，再启动k8
### 安装kubectl
```
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.5.2/bin/linux/amd64/kubectl
$ chmod +x kubectl
$ mv kubectl /usr/local/bin/
```