在centos7上部署。首先要检查```/etc/hosts```(相当于DNS)、```/etc/hostname```和本机IP，三者要匹配，否则安装会出错。主机名要用全限定名（如expample.com，而不是类似c7007）。  
```
$ cat /etc/hosts
192.168.14.207 c7007.ambari.apache.org c7007
$ cat /etc/hostname
c7007.ambari.apache.org
$ ip addr
    inet 192.168.14.207/24 brd 192.168.14.255 scope global enp0s8
```
安装：
```
$ yum install freeipa-server
$ ipa-server-install              (install只是安装了二进制代码，这步才是真正的部署)
Do you want to configure integrated DNS (BIND)? [no]:

Enter the fully qualified domain name of the computer
on which you're setting up server software. 
Server host name [c7007.ambari.apache.org]:

The domain name has been determined based on the host name.
Please confirm the domain name [ambari.apache.org]:
The kerberos protocol requires a Realm name to be defined.
This is typically the domain name converted to uppercase.
Please provide a realm name [AMBARI.APACHE.ORG]: 
Certain directory server operations require an administrative user.
The password must be at least 8 characters long.
Directory Manager password: 123456a?

The IPA server requires an administrative user, named 'admin'.
IPA admin password: 123456a?

The IPA Master Server will be configured with:
Hostname:       c7007.ambari.apache.org
IP address(es): 10.0.2.15, 192.168.14.207
Domain name:    ambari.apache.org
Realm name:     AMBARI.APACHE.ORG

Continue to configure the system with these values? [no]: y
（略）
```
帮助、登录admin、创建用户，注意创建用户和设置密码是分开的：
```
$ ipa help topics       (列出帮助主题)
$ ipa help <topic>      (列出某个主题帮助，如user、passwd)
$  kinit admin
Password for admin@AMBARI.APACHE.ORG: 123456a?
$ ipa user-add guest
First name: webb
Last name: wang
$ ipa passwd guest          (为guest用户设置密码)
New Password: 1
$ kinit guest               (使用新建的用户登录，需要重设密码)
$ klist
```
下面需要测试FreeIPA的界面(Web UI)，需要用到火狐浏览器。但目前安装的机器是个服务器，没有图形界面。启动另一个ubuntu桌面的虚拟机u1408（IP是192.168.14.108），并进入u1408。在/etc/hosts配置文件中添加新KDC的DNS解析：
```
echo "192.168.14.207 c7007.ambari.apache.org c7007" >> /etc/hosts
```  
u1408之前的KDC不是c7007，需要修改u1408的/etc/krb5.conf配置文件：
```
[realms]
  AMBARI.APACHE.ORG = {
     kdc = c7007.ambari.apache.org
     admin_server = c7007.ambari.apache.org
 }
```
用火狐浏览器测试SPNEGO认证的背景知识参考"[linux桌面下SPNEGO测试(firefox)](https://github.com/wbwangk/wbwangk.github.io/wiki/knox%E6%B5%8B%E8%AF%95#linux%E6%A1%8C%E9%9D%A2%E4%B8%8Bspnego%E6%B5%8B%E8%AF%95firefox)"。需要把".ambari.apache.org"加入negotiate信任域。  
首先是获得KDC票据：
```
$ su - vagrant
$ kinit admin
```
之所以切换为vagrant用户，是因为我安装的ubuntu14桌面的默认用户是vagrant。执行kinit和运行火狐浏览器必须同一个用户，否则无法共享kerberos票据。然后在火狐地址栏输入地址：
```
https://c7007.ambari.apache.org
```
如果按F12（或shift+ctrl+i）打开火狐的调试工具，可以看到这个SPNEGO认证过程。