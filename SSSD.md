[参考1](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/SSSD-Introduction.html)  
SSSD服务是一种远程身份提供程序的本地缓存。即使本机或远程身份提供程序脱机，仍可以利用SSSD缓存进行用户身份的验证。  
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
c7301节点的OpenLDAP服务器已经被配置为启用TLS/SSL。OpenLDAP启用TLS/SSL的方法参考博文《[LDAP进阶](https://imaidata.github.io/blog/ldap2/)》。

### 服务启动
```
$ service sssd start
```
默认SSSD不是配置为自动运行的。有两种方式可以将SSSD设置为自动运行:
1.用`authconfig`命令启用SSSD：
```
~]# authconfig --enablesssd --enablesssdauth --update
```
2.用chkconfig命令将SSSD进入加入到启动列表中：
```
~]# chkconfig sssd on
```
#### 配置服务:NSS
Name Service Switch (NSS)  
SSSD作为NSS的一个提供者服务，提供了几种NSS映射：
- Passwords (passwd)
- User groups (shadow)
- Groups (groups)
- Netgroups (netgroups)
- Services (services)

1. 使用身份验证配置工具启用SSSD。这将自动配置`/etc/nsswitch.conf`文件以使用SSSD作为提供者。  
```
~]# authconfig --enablesssd --update
```
然后nsswithch.conf变成下面这样：  
```
passwd:     files sss
shadow:     files sss
group:      files sss
netgroup:   files sss
``` 
2. 也可以手工修改`/etc/nsswitch.conf`配置文件，增加sss模块到服务映射：  
```
services: file sss
```

#### 配置服务:PAM
使用`authconfig`启用SSSD作为系统认证：
```
# authconfig --update --enablesssd --enablesssdauth
```
导致PAM配置文件增加了`pam_sss.so`配置项，以`/etc/pam.d/password-auth`为例：
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
首先测试NSS功能：
```
$ id sam
uid=10004(sam) gid=5004(scientist) groups=5004(scientist),5003(analyst)
```
再测试PAM功能：
```
$ id                         (显示当前用户为vagrant)
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant)
$ su - sam                   (切换为定义在LDAP中的sam用户，提示输入密码，密码也在LDAP中定义)
$ id                         (显示当前用户为sam，表示通过PAM的SSSD认证成功)
uid=10004(sam) gid=5004(scientist) groups=5004(scientist),5003(analyst) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```