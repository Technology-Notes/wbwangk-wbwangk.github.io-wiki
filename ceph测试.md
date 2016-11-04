准备3台虚拟机（node1/node2/node3)，在各台虚拟机的/etc/hosts文件中分别加入3台虚拟机的主机名和ip：
```
10.10.56.1      node1
10.10.56.2      node2
10.10.56.3      node3
```
每台虚机上安装ssh：
```
apt-get install openssh-server
```
每台虚机上创建ceph专用的用户ceph2：
```
# useradd -d /home/ceph2 -m ceph2
# passwd ceph2
```
确保各个虚机的ceph用户都有sudo权限：
```
echo "ceph ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ceph2
sudo chmod 0440 /etc/sudoers.d/ceph2
```
允许无密码SSH登录：
```
$ ssh-keygen

Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
```
复制公钥到其他各个节点：
```
ssh-copy-id ceph2@node2
ssh-copy-id ceph2@node3
```

安装apt-get密钥，apt-get利用该密钥校验安装包的hash sum值。
```
wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
```
将ceph加入APT源，hammer是最新版的ceph版本编号：
```
echo deb http://download.ceph.com/debian-hammer/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
```
lsb_release -sc命令会得到linux的版本类型号。增加了apt源后需要执行apt-get update，否则一些依赖包会缺失。
安装ceph：
```
sudo apt-get update && sudo apt-get install ceph ceph-mds
```
以上操作在每个ceph节点上都要进行。本来ceph提供了安装工具ceph-deploy，希望借助SSH实现网络安装。实测ceph-deploy问题多，所以选择手工安装。

apt-get install ceph碰到大量依赖包缺失，后从网上查到aptitude install ceph命令安装。

为虚机增加一块硬盘，以便ceph使用：http://www.linuxidc.com/Linux/2011-02/31868.htm