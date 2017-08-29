[1](https://arthurdejong.org/nss-pam-ldapd/setup)   

本文档描述了LDAP服务器中定义的用户和组如何登录到系统。系统通过NSS模块知道用户的存在，并通过PAM模块进行认证。

如果您正在使用Debian，您应该可以跳过这些步骤，安装libnss-ldapd和libpam-ldapd软件包([2](https://wiki.debian.org/LDAP/NSS))，回答配置问题并使其正常工作。有关详细信息，请参阅】[Debian wiki](http://wiki.debian.org/LDAP/NSS)。其他分销商也可能提供配置nss-pam-ldapd的帮助工具。

本指南涵盖最常见的配置，但[nss-pam-ldapd](https://arthurdejong.org/nss-pam-ldapd/)还支持TLS加密，使用Kerberos对LDAP服务器进行身份验证，使用Active Directory等等。有关详细信息，请参阅示例配置，[手册页](https://arthurdejong.org/nss-pam-ldapd/nslcd.conf.5)和[README](https://arthurdejong.org/nss-pam-ldapd/README)。

#### 开始前
假定你已经不是了一个LDAP服务器：
- LDAP服务器URI(如ldap://c7301.ambari.apache.org)
- LDAP服务器搜索基础(如dc=ambari,dc=apache,dc=org)
本测试在centos7.3下进行，而且是刚刚部署的干净的centos7.3。  

#### 安装
```
$ yum -y install nss-pam-ldapd
```
nss-pam-ladpd依赖[nscd](https://linux.die.net/man/8/nscd)。Nscd是为最常见的名称服务请求提供缓存的守护程序。默认配置文件`/etc/nscd.conf`确定缓存守护程序的行为。Nscd为passwd、group和hosts数据库的访问提供缓存。 

NSS-PAM-ldapd软件包允许LDAP目录服务器被用作linux系统名称服务(Name Service)信息的主要来源。名称服务信息通常包括历史上存储在平面文件或NIS中的用户，主机，组和其他此类数据。  
NSS-PAM-ldapd软件包可以启用一个叫[nslcd](https://linux.die.net/man/8/nslcd)是一个守护进程，它将根据简单的配置文件对本地进程执行LDAP查询。nslcd的配置文件是`/etc/nslcd.conf`。  

#### 启动进程和配置
利用centos的auth配置工具配置ldap：
```
$ service nslcd status                (服务状态Active: inactive (dead))
$ authconfig --enableldap --ldapserver="ldap://c7301.ambari.apache.org/" --ldapbasedn="dc=ambari,dc=apache,dc=org" --disableldaptls --disableldapstarttls --updateall
```
执行上述配置后，会改写`/etc/nslcd.conf`，会启动服务`nslcd`。服务状态查看（）：
```
$ service nslcd status                (服务状态Active: active (running) )
```
查看一下`/etc/nslcd.conf`中的重要配置：
```
uid nslcd
gid ldap
uri ldap://c7301.ambari.apache.org/
base dc=ambari,dc=apache,dc=org
ssl no
tls_cacertdir /etc/openldap/cacerts
```
估计不使用`authconfig`，手工修改上述配置文件，手工启用`nslcd`服务也是可以的。  

#### 测试LDAP
```
$ cat /etc/passwd | grep webb                  (本地不存在用户webb)
$ getent passwd webb
webb:*:10002:5002:webb:/home/webb:/bin/sh
```
上面的星号表示这个用户可以找到，但不能用来登录。  

#### Name Service Switch
linux的用户查找通过NSS机制，NSS的配置文件`/etc/nsswitch.conf`中与LDAP相关的配置：
```
passwd:     files sss ldap
shadow:     files sss ldap
group:      files sss ldap
```
根据上述配置，系统查找用户、组时，先找本地文件(/etc/passwd、/etc/shadow、/etc/group)，然后到SSSD，最后找LDAP。本机没装SSSD，所以webb用户在本地文件(/etc/passwd)没找到后，通过`nslcd`找到了。如果停止`nslcd`服务，webb用户就找不到了。  
```
$ service nslcd stop
$ getent passwd webb                            (显示不出webb用户的信息)
```
#### 离线缓存NSCD
手工将c7301停机，然后再执行:
```
$ getent passwd webb                       (LDAP服务器当机后，不再显示webb用户信息)
```
NSCD提供NSS的缓存功能。NSCD的配置文件是`/etc/nscd.conf`，配置文件中与离线缓存有关的是：
```
reload-count            unlimited
positive-time-to-live   <service>          #number of second
```
将passwd和group服务缓存设置为30天的配置：
```
positive-time-to-live   passwd          2592000
positive-time-to-live   group           2592000
```
手工刷新缓存：
```
# nscd -i passwd
# nscd -i group
```
