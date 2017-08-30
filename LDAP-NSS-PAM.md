## 第一章：LDAP NSS
[原文](https://wiki.debian.org/LDAP/NSS )[1](https://arthurdejong.org/nss-pam-ldapd/setup)   

本文档描述了LDAP服务器中定义的用户和组如何登录到系统。系统通过NSS模块知道用户的存在，并通过PAM模块进行认证。

如果您正在使用Debian，您应该可以跳过这些步骤，安装libnss-ldapd和libpam-ldapd软件包([2](https://wiki.debian.org/LDAP/NSS))，回答配置问题并使其正常工作。有关详细信息，请参阅】[Debian wiki](http://wiki.debian.org/LDAP/NSS)。其他分销商也可能提供配置nss-pam-ldapd的帮助工具。

本指南涵盖最常见的配置，但[nss-pam-ldapd](https://arthurdejong.org/nss-pam-ldapd/)还支持TLS加密，使用Kerberos对LDAP服务器进行身份验证，使用Active Directory等等。有关详细信息，请参阅示例配置，[手册页](https://arthurdejong.org/nss-pam-ldapd/nslcd.conf.5)和[README](https://arthurdejong.org/nss-pam-ldapd/README)。

#### 开始前
假定你已经不是了一个LDAP服务器：
- LDAP服务器URI(如ldap://c7301.ambari.apache.org)
- LDAP服务器搜索基础(如dc=ambari,dc=apache,dc=org)
本测试在centos7.3下进行，而且是刚刚部署的干净的centos7.3。  
本测试中使用webb用户可以通过[ldapscripts](https://github.com/wbwangk/wbwangk.github.io/wiki/LDAP#ldap%E5%B7%A5%E5%85%B7ldapscritps)添加到LDAP中。  

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
webb:*:40091:5002:webb:/home/webb:/bin/bash
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

## 第二章：LDAP PAM
[原文](https://wiki.debian.org/LDAP/PAM)  
基本上有两种方法配置PAM以使linux用LDAP服务器认证。第一个方法利用[libpam-ldap](https://packages.debian.org/libpam-ldap)包中的pam_ldap模块检查LDAP服务器的凭据。第二种方法是使用NSS从LDAP服务器发送到客户端的密码哈希值。然后传统的pam_unix模块进行认证。在后面方法中，pam_ldap仍然可以用于更改密码。这两种解决方案都各有利弊。

在这两种情况下，用户都通过NSS暴露。值得注意的是，如果在以root用户身份运行时使用getent shadow返回密码散列，则需要pam_unix方法。

纯pam_ldap解决方案允许通过将用户存储在不同的目录来限制登录（例如，仅允许登录目录中的某个目录中的用户，需要某些属性等）。它还需要较少的LDAP目录访问权限，并且不会公开密码散列。

pam_unix的解决方案可能是系统管理员比较熟悉。

### 方法一：使用libpam-ldap
通过第一章安装`nss-pam-ladpd`软件包后，会安装一个叫`pam_ldap.so`PAM模块：
```
$ yum -y install nss-pam-ldapd
$ ls /lib64/security/pam_ldap.so
```
现在利用这个pam模块来配置linux使用LDAP登录。编辑配置文件`/etc/pam.d/system-auth`，增加4个pam_ldap.so的行：
```
#%PAM-1.0
# This file is auto-generated.
# User changes will be destroyed the next time authconfig is run.
auth        required      pam_env.so
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
auth        sufficient    pam_ldap.so use_first_pass
auth        required      pam_deny.so

account     required      pam_unix.so broken_shadow
account     sufficient    pam_localuser.so
account     sufficient    pam_succeed_if.so uid < 1000 quiet
account     [default=bad success=ok user_unknown=ignore] pam_ldap.so
account     required      pam_permit.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
password    sufficient    pam_ldap.so
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     optional      pam_mkhomedir.so umask=0077
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
session     optional      pam_ldap.so
```
测试一下使用webb用户登录：
```
$ getent passwd webb
webb:*:10002:5002:webb:/home/webb:/bin/sh
$ su - webb
Password:
Last login: Fri Aug 25 02:47:32 UTC 2017 on pts/0
$ whoami                                  (显示当前用户)
webb
$ id -Gn                                  (显示当前用户所属的组)
wang
```
### 方法二：使用pam_unix
也可以使用普通旧的pam_unix，确保/etc/nsswitch.conf包含像shadow：files ldap和getent shadow显示密码哈希。  
实测发现getent shadow无法显示密码哈希值：
```
$ getent shadow webb                    (没有返回密码哈希值)
```
webb用户按说已经用下列命令[ldapscritps](https://github.com/wbwangk/wbwangk.github.io/wiki/LDAP#ldap%E5%B7%A5%E5%85%B7ldapscritps)在LDAP中生成了密码：
```
$ sudo ldapsetpasswd webb
```
导致这种方法不能用，原因不明！

## 第三章：浅析PAM
[一篇深入浅出的PAM文章](https://wenku.baidu.com/view/40c039fe04a1b0717fd5dd28.html)  [1](http://wpollock.com/AUnix2/PAM-Help.htm)  
老版本PAM的配置文件是`/etc/pam.conf`的格式：
```
service module_type control_flag module_path
------- ----------- ------------ -----------
login   auth        required     pam_unix_auth.so
```
在PAM新版本中，配置文件放在`/etc/padm.d/`目录下，文件名是service名，文件内容只留下了module_type、control_flash、module_path三部分：
```
#%PAM-1.0
auth            sufficient      pam_rootok.so
```
#### LDAP
要使用LDAP，您可以：  
- 使用pam _ldap模块。  
- 使用pam_unix模块并配置NSS(name service switch)以使用LDAP。  
- 使用pam _sssd模块并配置SSSD使用LDAP。   
- 使用pam_unix模块，并将NSS配置为使用SSSD，并将SSSD其配置为使用LDAP。  
- 直接使用一些LDAP库，并绕过PAM。  
- 使用标准系统调用（绕过PAM），并将NSS配置为使用LDAP（或使用SSSD，SSSD又使用LDAP）。  

#### PAM的例子
[2](https://www.linux.com/news/understanding-pam)  
```
auth       required     /lib/security/pam_securetty.so
auth       required     /lib/security/pam_env.so
auth       sufficient   /lib/security/pam_ldap.so
auth       required     /lib/security/pam_unix.so try_first_pass
```
由于模块被按顺序调用，所以会发生什么：

1. 'pam_securetty'模块将检查其配置文件`/etc/securetty`，并查看该文件中是否列出了用于此登录的终端。如果不是，root登录将不被允许。如果您尝试以root身份登录到“坏”终端，则此模块将失败。由于它是“必需”，它仍将调用堆栈中的所有模块。但是，即使其中每一个都成功，登录也将失败。有趣的是，如果模块被列为“必需”，则操作将立即终止，并且不调用任何其他模块，而不管其状态如何。
2. “pam_env”模块将根据管理员在/etc/security/pam_env.conf中设置的内容来设置环境变量。在Redhat 9，Fedora Core 1和Mandrake 9.2 的默认设置下，此模块的配置文件实际上并没有设置任何变量。一个很好的用处可能是自动为通过SSH登录的用户设置一个DISPLAY环境变量，如果他们想要将“xterm”回到远程桌面，那么它们不必自己设置（尽管可以采取这种方式关心OpenSSH自动化）。
3. 'pam_ldap'模块将提示用户输入密码，然后检查/etc/ldap.conf中指定的ldap目录来验证用户。如果失败，如果'pam_unix'成功验证用户，操作仍然可以成功。如果pam_ldap成功，将不会调用“pam_unix”。
4. 在这种情况下，'pam_unix'模块不会提示用户输入密码。'try_first_pass'参数将告诉模块使用前面的模块给出的密码（在这种情况下是pam_ldap）。它将尝试使用标准的`getpw*`系统调用来验证用户。如果pam_unix失败，并且pam_ldap失败，操作将失败。如果pam_ldap失败，但是pam_unix成功，则操作将成功（这在root不在ldap目录中但仍在本地/etc/ passwd文件中的情况下非常有用）。