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