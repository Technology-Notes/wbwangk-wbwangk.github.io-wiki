openstack最新版是newton。目前（2016-11-28）在ubuntu上仅支持ubuntu16的xenial版（实测trusty版不行）。
vagrant box 是"ubuntu/xenial64"，provider是virtualbiox。在公司下载这个box花费了3个小时。

安装keystone的机器域名是controller，在/etc/hosts文件中增加了一条记录：```192.168.1.116 controller```
也可以在Vagrantfile中直接定义hostname为"controller"。

官方安装说明中授权的是'keystone'@'localhost',实测需要增加'keystone'@'controller'的授权(该观点未确认是否正确).
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'controller' \
  IDENTIFIED BY 'vagrant';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
  IDENTIFIED BY 'vagrant';

## 按keystone开发文档安装
文档地址：http://docs.openstack.org/developer/keystone/index.html
首先使用apt安装keystone：
```
apt-get install keystone
```