[参考1](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/SSSD-Introduction.html)  
本文与另一篇文章《[LDAP NSS PAM](https://github.com/wbwangk/wbwangk.github.io/wiki/LDAP-NSS-PAM)》相关。这两篇文章都是关于使用LDAP实现linux的远程命名服务和远程认证功能。一个使用nslcd进程，一个使用sssd进程。  

SSSD服务是一种远程身份提供程序的本地缓存。即使本机或远程身份提供程序脱机，仍可以利用SSSD缓存进行用户身份的验证。  
### 安装
```
$ sudo yum -y install sssd
```
### 配置文件
SSSD服务的默认配置文件是`/etc/sssd/sssd.conf`。  
SSSD服务的配置文件分成三部分：  
- [sssd]，用于一般的SSSD过程和操作配置; 这基本上列出了每个配置的服务，域和配置参数  
- [service_name]，用于每个支持的系统服务的配置选项，如`[nss]`、`[pam]`、`[sudo]`、`[autofs]`、`[ssh]`、`[pac]`、`[ifp]`  
- [domain_type/DOMAIN_NAME]，用于每个配置的身份提供者的配置选项。如`[domain/abc.com]`、`[domain/a.abc.com]`  

[sssd]小节主要包括三个重要参数：  
- domains 列出了将SSSD充当id提供者的所有域名。    
- services 列出了哪些系统服务使用SSSD。  
- config_file_version 配置文件格式版本号，最新的是2。  

将`/etc/sssd/sssd.conf`设置为下面的样子：
```
[sssd]
domains = ambari.apache.org
services = nss,pam

[domain/ambari.apache.org]
id_provider = ldap
auth_provider = ldap

ldap_uri = ldaps://c7301.ambari.apache.org
ldap_search_base = dc=ambari,dc=apache,dc=org

#ldap_id_use_start_tls = true
ldap_tls_reqcert = demand
ldap_tls_cacert = /etc/ssl/ca.crt

[nss]
filter_groups = root
filter_users = root
entry_cache_timeout = 300
entry_cache_nowait_percentage = 75
```
创建完sssd.conf文件后一定要确保它的访问权限是600，否则sssd服务会启动失败：  
```
# chmod 600 /etc/sssd/sssd.conf
```

c7301节点的OpenLDAP服务器已经被配置为启用TLS/SSL。OpenLDAP启用TLS/SSL的方法参考博文《[LDAP进阶](https://imaidata.github.io/blog/ldap2/)》。  
如果ca.crt还没有从LDAP服务器上复制过来就执行：
```
$ scp root@c7301:/opt/ca/ca.crt /etc/ssl
```
### 服务启动
```
$ chmod 600 /etc/sssd/sssd.conf
$ service sssd start
```
### 测试一下NSS功能
```
$ id sam
uid=10004(sam) gid=5004(scientist) groups=5004(scientist),5003(analyst)
```
#### 解释一下NSS+SSSD的原理
看一下NSS(Name Service Switch)的配置文件`/etc/nsswitch.conf`，可以看到以下的内容：
```
passwd:     files sss
shadow:     files sss
group:      files sss
```   
当执行`id sam`命令时，linux会根据`nsswitch.conf`配置文件中的`passwd`参数查看用户。根据`files`设置，系统会去`/etc/passwd`文件中去找。发现sam里面没有sam用户。然后NSS根据配置文件中`sss`设置，向SSSD服务请求，SSSD服务根据sssd.conf中的配置到LDAP服务器中去找，终于找到了sam用户。  

### 配置服务:PAM
pam的配置文件位于`/etc/pam.d/`目录下。这个目录下有多个文件，貌似带auth的文件是与认证有关的。可以打开`/etc/pam.d/password-auth`看看。可以看到利用的用户认证模块是`pam_unix.so`。

使用`authconfig`启用SSSD作为系统认证：
```
# authconfig --enablesssdauth --update
```
上述命令导致PAM配置文件增加了`pam_sss.so`配置项，以`/etc/pam.d/password-auth`为例：
```
#%PAM-1.0
# This file is auto-generated.
# User changes will be destroyed the next time authconfig is run.
auth        required      pam_env.so
auth        [default=1 success=ok] pam_localuser.so
auth        [success=done ignore=ignore default=die] pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
auth        sufficient    pam_sss.so forward_pass
auth        required      pam_deny.so

account     required      pam_unix.so
account     sufficient    pam_localuser.so
account     sufficient    pam_succeed_if.so uid < 1000 quiet
account     [default=bad success=ok user_unknown=ignore] pam_sss.so
account     required      pam_permit.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
password    sufficient    pam_sss.so use_authtok
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
session     optional      pam_sss.so
```
测试SSSD的LDAP中的数据参考文章《[LDAP进阶](https://imaidata.github.io/blog/ldap2/)》。  

测试PAM的认证功能：
```
$ id                         (显示当前用户为vagrant)
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant)
$ su - sam                   (切换为定义在LDAP中的sam用户，提示输入密码，密码也在LDAP中定义)
$ id                         (显示当前用户为sam，表示通过PAM的SSSD认证成功)
uid=10004(sam) gid=5004(scientist) groups=5004(scientist),5003(analyst) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```