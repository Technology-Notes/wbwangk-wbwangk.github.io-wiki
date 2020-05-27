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
$ git clone --depth 1 https://github.com/wbwangk/kube-deploy
$ cd kube-deploy/docker-multinode
$ export IP_ADDRESS=192.168.1.140
$ ./master.sh
```
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
等主节点的所有容器起来后（可能需要翻墙），执行docker ps：
```
CONTAINER ID        IMAGE                                             COMMAND                  CREATED             STATUS              PORTS               NAMES
8295aab03d15        gcr.io/google_containers/hyperkube-amd64:v1.5.2   "/copy-addons.sh mult"   3 minutes ago       Up 3 minutes                            k8s_kube-addon-manager-data.21c62009_kube-addon-manager-10.0.2.15_kube-system_92f3be110d19c44f081ae988b5d212e5_9efd60c0
0781c87d0be8        gcr.io/google_containers/hyperkube-amd64:v1.5.2   "/hyperkube apiserver"   4 minutes ago       Up 4 minutes                            k8s_apiserver.f8f4c740_k8s-master-10.0.2.15_kube-system_641e44c2313a0e2de04ba5de4e9414be_298cce5e
b20415580373        gcr.io/google_containers/hyperkube-amd64:v1.5.2   "/setup-files.sh IP:1"   4 minutes ago       Up 4 minutes                            k8s_setup.26d03c3a_k8s-master-10.0.2.15_kube-system_641e44c2313a0e2de04ba5de4e9414be_0fa806a7
892314bda257        gcr.io/google_containers/hyperkube-amd64:v1.5.2   "/hyperkube scheduler"   4 minutes ago       Up 4 minutes                            k8s_scheduler.c4a803ee_k8s-master-10.0.2.15_kube-system_641e44c2313a0e2de04ba5de4e9414be_330ccdae
db7d5552163a        gcr.io/google_containers/hyperkube-amd64:v1.5.2   "/hyperkube controlle"   4 minutes ago       Up 4 minutes                            k8s_controller-manager.1db66019_k8s-master-10.0.2.15_kube-system_641e44c2313a0e2de04ba5de4e9414be_9b0ea747
8bebc2ee70a9        gcr.io/google_containers/pause-amd64:3.0          "/pause"                 31 minutes ago      Up 31 minutes                           k8s_POD.d8dbe16c_kube-addon-manager-10.0.2.15_kube-system_92f3be110d19c44f081ae988b5d212e5_6c1ec6cd
a8ee8be1147a        gcr.io/google_containers/pause-amd64:3.0          "/pause"                 31 minutes ago      Up 31 minutes                           k8s_POD.d8dbe16c_k8s-master-10.0.2.15_kube-system_641e44c2313a0e2de04ba5de4e9414be_495c287c
1d970f2b00f4        wbwang/hyperkube-amd64:v1.5.2                     "/hyperkube kubelet -"   31 minutes ago      Up 31 minutes                           kube_kubelet_a6663
```
### 部署worker节点
利用vagrant启动worker节点(ip 192.168.1.141)：
```
$ vagrant up k8s1
$ vagrant ssh k8s1
```
（可选）测试一下主节点的etcd是否正常工作：
```
$ export MASTER_IP=192.168.1.140
$ curl $MASTER_IP:2379/version
{"etcdserver":"3.0.4","etcdcluster":"3.0.0"}
```
下载部署脚本：
```
$ cd /opt
$ git clone --depth 1 https://github.com/wbwangk/kube-deploy
$ cd kube-deploy/docker-multinode
$ docker pull wbwang/flannel:v0.6.1-amd64
$ docker pull wbwang/hyperkube-amd64:v1.5.2
$ export MASTER_IP=192.168.1.140
$ export IP_ADDRESS=192.168.1.141
$ ./worker.sh
```
如果节点上未安装docker引擎,可以用下列命令安装:
```
$ apt install docker.io
```
手工pull用到的两个docker镜像可以提高worker.sh运行成功的概率。实测手工pull镜像后，可以不用翻墙就成功运行worker.sh。  
注意运行worker.sh前，需要设置好MASTER_IP，并用curl测试主节点的etcd能否链接上。  
worker.sh执行成功后会提示：
```
+++ [0213 00:07:15] Done. After about a minute the node should be ready.
```
利用同样的办法，再启动k8s2节点(192.168.1.142)和k8s3节点。
### 环境变量的总结
k8s0(master节点)：
```
$ export IP_ADDRESS=192.168.1.140
```
k8s1(woker节点1)：
```
$ export MASTER_IP=192.168.1.140
$ export IP_ADDRESS=192.168.1.141
```
k8s2(woker节点2)：
```
$ export MASTER_IP=192.168.1.140
$ export IP_ADDRESS=192.168.1.142
```
k8s3(woker节点3)：
```
$ export MASTER_IP=192.168.1.140
$ export IP_ADDRESS=192.168.1.143
```
### 安装kubectl
```
$ wget https://storage.googleapis.com/kubernetes-release/release/v1.5.2/bin/linux/amd64/kubectl
$ chmod +x kubectl
$ mv kubectl /usr/local/bin/
```
等主节点的所有容器都启动后：
```
$ kubectl get nodes
NAME            STATUS     AGE
192.168.1.140   Ready      4m
192.168.1.141   Ready      9m
192.168.1.142   Ready      10m
192.168.1.143   Ready      11m
```
kubectl要访问localhost:8080端口（即apiserver的监听端口）获取数据，这种提示说明apiserver已经成功启动了。