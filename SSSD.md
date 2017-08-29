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

PAM配置文件`vi /etc/pam.d/password-auth`