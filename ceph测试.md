安装apt-get密钥，apt-get利用该密钥校验安装包的hash sum值。
```
wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
```
将ceph加入APT源，hammer是最新版的ceph版本编号：
```
echo deb http://download.ceph.com/debian-hammer/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
```
安装ceph：
```
sudo apt-get update && sudo apt-get install ceph ceph-mds
```
以上操作在每个ceph节点上都要进行。本来ceph提供了安装工具ceph-deploy，希望借助SSH实现网络安装。实测ceph-deploy问题多，所以选择手工安装。
尝试unbuntu官方APT源总是提示hash sum错误，换成163源后不再报错。编辑/etc/apt/sources.list，把内容换成以下：
```
deb http://mirrors.163.com/ubuntu/ precise main universe restricted multiverse
deb-src http://mirrors.163.com/ubuntu/ precise main universe restricted multiverse
deb http://mirrors.163.com/ubuntu/ precise-security universe main multiverse restricted
deb-src http://mirrors.163.com/ubuntu/ precise-security universe main multiverse restricted
deb http://mirrors.163.com/ubuntu/ precise-updates universe main multiverse restricted
deb http://mirrors.163.com/ubuntu/ precise-proposed universe main multiverse restricted
deb-src http://mirrors.163.com/ubuntu/ precise-proposed universe main multiverse restricted
deb http://mirrors.163.com/ubuntu/ precise-backports universe main multiverse restricted
deb-src http://mirrors.163.com/ubuntu/ precise-backports universe main multiverse restricted
deb-src http://mirrors.163.com/ubuntu/ precise-updates universe main multiverse restricted

```