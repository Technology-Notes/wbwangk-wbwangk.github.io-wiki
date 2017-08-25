测试环境：centos7、[3节点hadoop](https://imaidata.github.io/blog/ambari_centos/)、[openldap](https://imaidata.github.io/blog/ldap/)。  
OpenLdap部署在c7301节点(FQDN是c7301.ambari.apache.org)。  
#### 基于LDAP的网络认证
hadoop授权功能很多都依赖**用户到用户组的映射**（指找到用户所属的用户组，可能存在多个用户组）。用户和用户组一般定义在LDAP中。hadoop支持多种方式将用户映射到用户组，默认是通过操作系统查找，类似通过命名`id -Gn`获取当前用户的用户组。虽然可以将hadoop配置为直接通过LDAP进行用户到用户组的映射，但这将失去定义在操作系统本地的hadoop服务用户，如`hdfs`。  
所以，建议在操作系统级别配置LDAP，让操作系统同时从本地文件(/etc/passwd)和远程LDAP中获取用户和用户组，这又被称为**网络认证**。实现网络认证可选`nslcd`或`sssd`。  
([1](https://www.certdepot.net/ldap-client-configuration-authconfig/))这篇文章和([2](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System-Level_Authentication_Guide/authconfig-ldap.html))红帽的官方文档都讲解了如何配置网络认证。`authconfig`是centos7自带的一个配置认证方式的命令。  
到c7303节点(FQDN是c7303.ambari.apache.org)上执行`authconfig`来配置远程认证：
```
$ sudo authconfig --enableldap --enableldapauth --enablemkhomedir --ldapserver=ldap://c7301.ambari.apache.org:389 --ldapbasedn="dc=ambari,dc=apache,dc=org" --update
authconfig: Authentication module /lib64/security/pam_ldap.so is missing. Authentication process might not work correctly.
```
上面的提示说明c7303节点缺少pam_ldap的认证模块。安装sssd:
```
$ sudo yum install -y sssd
$ authconfig --enableldap --enableldapauth --enablemkhomedir --ldapserver=ldap://c7301.ambari.apache.org:389/ --ldapbasedn="dc=ambari,dc=apache,dc=org" --update
$ getent passwd webb                               
webb:*:10002:5002:webb:/home/webb:/bin/sh
```
安装sssd后，默认是不自动运行的。要让sssd自动运行，需要使用`authconfig`来配置，例如：
```
$ authconfig --enablesssd --enablesssdauth --update
```
`webb`是LDAP中定义一个用户，本地没有这个用户。  
sssd的默认配置文件是`vi /etc/sssd/sssd.conf`。打开这个配置文件可以看到之前`authconfig`命令行中的`ldapserver`、`ldapbasedn`等配置项。  