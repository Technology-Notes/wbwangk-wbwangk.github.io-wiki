## 安装
[参考](http://docs.ceph.org.cn/start/quick-start-preflight/#ceph)

node1（10.10.56.1)充当管理节点，node2/3/4当存储节点。

管理节点node1安装部署工具ceph-deploy。用jewel替换{ceph-stable-release}。

node2/3/4安装ssh：
```
sudo apt-get install openssh-server 或 sudo yum install ntp ntpdate ntp-doc
```
node2/3/4安装NTP（时间同步）：
```
apt-get install ntp 或 yum install ntp ntpdate ntp-doc
```
在node2/3/4上创建用户ceph2，并赋予sudo权限。
```
ssh wbwang@10.10.56.2
sudo useradd -d /home/ceph2 -m ceph2
sudo passwd ceph2
echo "ceph2 ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph2
sudo chmod 0440 /etc/sudoers.d/ceph2
```
在node1上使用wbwang用户生成ssh密钥。
将ssh公钥复制到各存储节点：
```
ssh-copy-id ceph2@10.10.56.2
ssh-copy-id ceph2@10.10.56.3
ssh-copy-id ceph2@10.10.56.4
```
