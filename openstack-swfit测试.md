openstack swfit在github上的项目地址是：https://github.com/openstack/swift
安装过程参考了文档：[SAIO - Swift All In One](http://docs.openstack.org/developer/swift/development_saio.html)
主要测试了Using a loopback device for storage方式安装。安装过程中碰到mkfs.xfs找不到的情况，需要手工安装mkfs.xfs：
```yum -y install xfsprogs```或```apt-get -y install xfsprogs```


