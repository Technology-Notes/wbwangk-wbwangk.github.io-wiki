## 手工部署FreeIPA
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
Continue to configure the system with these values? [no]: y

The following operations may take some minutes to complete.
Please wait until the prompt is returned.

Configuring NTP daemon (ntpd)
  [1/4]: stopping ntpd
  [2/4]: writing configuration
  [3/4]: configuring ntpd to start on boot
  [4/4]: starting ntpd
Done configuring NTP daemon (ntpd).
Configuring directory server (dirsrv). Estimated time: 1 minute
  [1/47]: creating directory server user
  [2/47]: creating directory server instance
  [3/47]: updating configuration in dse.ldif
  [4/47]: restarting directory server
  [5/47]: adding default schema
  [6/47]: enabling memberof plugin
  [7/47]: enabling winsync plugin
  [8/47]: configuring replication version plugin
  [9/47]: enabling IPA enrollment plugin
  [10/47]: enabling ldapi
  [11/47]: configuring uniqueness plugin
  [12/47]: configuring uuid plugin
  [13/47]: configuring modrdn plugin
  [14/47]: configuring DNS plugin
  [15/47]: enabling entryUSN plugin
  [16/47]: configuring lockout plugin
  [17/47]: configuring topology plugin
  [18/47]: creating indices
  [19/47]: enabling referential integrity plugin
  [20/47]: configuring certmap.conf
  [21/47]: configure autobind for root
  [22/47]: configure new location for managed entries
  [23/47]: configure dirsrv ccache
  [24/47]: enabling SASL mapping fallback
  [25/47]: restarting directory server
  [26/47]: adding sasl mappings to the directory
  [27/47]: adding default layout
  [28/47]: adding delegation layout
  [29/47]: creating container for managed entries
  [30/47]: configuring user private groups
  [31/47]: configuring netgroups from hostgroups
  [32/47]: creating default Sudo bind user
  [33/47]: creating default Auto Member layout
  [34/47]: adding range check plugin
  [35/47]: creating default HBAC rule allow_all
  [36/47]: adding sasl mappings to the directory
  [37/47]: adding entries for topology management
  [38/47]: initializing group membership
  [39/47]: adding master entry
  [40/47]: initializing domain level
  [41/47]: configuring Posix uid/gid generation
  [42/47]: adding replication acis
  [43/47]: enabling compatibility plugin
  [44/47]: activating sidgen plugin
  [45/47]: activating extdom plugin
  [46/47]: tuning directory server
  [47/47]: configuring directory to start on boot
Done configuring directory server (dirsrv).
Configuring certificate server (pki-tomcatd). Estimated time: 3 minutes 30 seconds
  [1/31]: creating certificate server user
  [2/31]: configuring certificate server instance
  [3/31]: stopping certificate server instance to update CS.cfg
  [4/31]: backing up CS.cfg
  [5/31]: disabling nonces
  [6/31]: set up CRL publishing
  [7/31]: enable PKIX certificate path discovery and validation
  [8/31]: starting certificate server instance
  [9/31]: creating RA agent certificate database
  [10/31]: importing CA chain to RA certificate database
  [11/31]: fixing RA database permissions
  [12/31]: setting up signing cert profile
  [13/31]: setting audit signing renewal to 2 years
  [14/31]: restarting certificate server
  [15/31]: requesting RA certificate from CA
  [16/31]: issuing RA agent certificate
  [17/31]: adding RA agent as a trusted user
  [18/31]: authorizing RA to modify profiles
  [19/31]: authorizing RA to manage lightweight CAs
  [20/31]: Ensure lightweight CAs container exists
  [21/31]: configure certmonger for renewals
  [22/31]: configure certificate renewals
  [23/31]: configure RA certificate renewal
  [24/31]: configure Server-Cert certificate renewal
  [25/31]: Configure HTTP to proxy connections
  [26/31]: restarting certificate server
  [27/31]: migrating certificate profiles to LDAP
  [28/31]: importing IPA certificate profiles
  [29/31]: adding default CA ACL
  [30/31]: adding 'ipa' CA entry
  [31/31]: updating IPA configuration
Done configuring certificate server (pki-tomcatd).
Configuring directory server (dirsrv). Estimated time: 10 seconds
  [1/3]: configuring ssl for ds instance
  [2/3]: restarting directory server
  [3/3]: adding CA certificate entry
Done configuring directory server (dirsrv).
Configuring Kerberos KDC (krb5kdc). Estimated time: 30 seconds
  [1/9]: adding kerberos container to the directory
  [2/9]: configuring KDC
  [3/9]: initialize kerberos container
  [4/9]: adding default ACIs
  [5/9]: creating a keytab for the directory
  [6/9]: creating a keytab for the machine
  [7/9]: adding the password extension to the directory
  [8/9]: starting the KDC
  [9/9]: configuring KDC to start on boot
Done configuring Kerberos KDC (krb5kdc).
Configuring kadmin
  [1/2]: starting kadmin
  [2/2]: configuring kadmin to start on boot
Done configuring kadmin.
Configuring ipa_memcached
  [1/2]: starting ipa_memcached
  [2/2]: configuring ipa_memcached to start on boot
Done configuring ipa_memcached.
Configuring ipa-otpd
  [1/2]: starting ipa-otpd
  [2/2]: configuring ipa-otpd to start on boot
Done configuring ipa-otpd.
Configuring ipa-custodia
  [1/5]: Generating ipa-custodia config file
  [2/5]: Making sure custodia container exists
  [3/5]: Generating ipa-custodia keys
  [4/5]: starting ipa-custodia
  [5/5]: configuring ipa-custodia to start on boot
Done configuring ipa-custodia.
Configuring the web interface (httpd). Estimated time: 1 minute
  [1/21]: setting mod_nss port to 443
  [2/21]: setting mod_nss cipher suite
  [3/21]: setting mod_nss protocol list to TLSv1.0 - TLSv1.2
  [4/21]: setting mod_nss password file
  [5/21]: enabling mod_nss renegotiate
  [6/21]: adding URL rewriting rules
  [7/21]: configuring httpd
  [8/21]: configure certmonger for renewals
  [9/21]: setting up httpd keytab
  [10/21]: setting up ssl
  [11/21]: importing CA certificates from LDAP
  [12/21]: setting up browser autoconfig
  [13/21]: publish CA cert
  [14/21]: clean up any existing httpd ccache
  [15/21]: configuring SELinux for httpd
  [16/21]: create KDC proxy user
  [17/21]: create KDC proxy config
  [18/21]: enable KDC proxy
  [19/21]: restarting httpd
  [20/21]: configuring httpd to start on boot
  [21/21]: enabling oddjobd
Done configuring the web interface (httpd).
Applying LDAP updates
Upgrading IPA:
  [1/9]: stopping directory server
  [2/9]: saving configuration
  [3/9]: disabling listeners
  [4/9]: enabling DS global lock
  [5/9]: starting directory server
  [6/9]: upgrading server
  [7/9]: stopping directory server
  [8/9]: restoring configuration
  [9/9]: starting directory server
Done.
Restarting the directory server
Restarting the KDC
Please add records in this file to your DNS system: /tmp/ipa.system.records.i7x6cH.db
Restarting the web server
Configuring client side components
Using existing certificate '/etc/ipa/ca.crt'.
Client hostname: c7001.ambari.apache.org
Realm: AMBARI.APACHE.ORG
DNS Domain: ambari.apache.org
IPA Server: c7001.ambari.apache.org
BaseDN: dc=ambari,dc=apache,dc=org

Skipping synchronizing time with NTP server.
New SSSD config will be created
Configured sudoers in /etc/nsswitch.conf
Configured /etc/sssd/sssd.conf
trying https://c7001.ambari.apache.org/ipa/json
Forwarding 'ping' to json server 'https://c7001.ambari.apache.org/ipa/json'
Forwarding 'ca_is_enabled' to json server 'https://c7001.ambari.apache.org/ipa/json'
Systemwide CA database updated.
Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ecdsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ed25519_key.pub
Forwarding 'host_mod' to json server 'https://c7001.ambari.apache.org/ipa/json'
Could not update DNS SSHFP records.
SSSD enabled
Configured /etc/openldap/ldap.conf
Configured /etc/ssh/ssh_config
Configured /etc/ssh/sshd_config
Configuring ambari.apache.org as NIS domain.
Client configuration complete.

==============================================================================
Setup complete

Next steps:
        1. You must make sure these network ports are open:
                TCP Ports:
                  * 80, 443: HTTP/HTTPS
                  * 389, 636: LDAP/LDAPS
                  * 88, 464: kerberos
                UDP Ports:
                  * 88, 464: kerberos
                  * 123: ntp

        2. You can now obtain a kerberos ticket using the command: 'kinit admin'
           This ticket will allow you to use the IPA tools (e.g., ipa user-add)
           and the web user interface.

Be sure to back up the CA certificates stored in /root/cacert.p12
These files are required to create replicas. The password for these
files is the Directory Manager password
```
无提示安装的命令：
```
$ ipa-server-install --hostname=c7004.ambari.apache.org --domain=ambari.apache.org --realm=AMBARI.APACHE.ORG --ds-password=vagrant2 --master-password=vagrant2 --admin-password=vagrant2 --unattended
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
测试FreeIPA的界面(Web UI)。在windows的浏览器中（如chrome）输入地址：```https://c7004.ambari.apache.org```(需要提前把这个域名的ip加入到windows的etc/hosts配置文件中)。会提出登录页面，由于windows并没有加入AMBARI.APACHE.ORG域，无法使用SPNEGO认证（参考"[linux桌面下SPNEGO测试(firefox)](https://github.com/wbwangk/wbwangk.github.io/wiki/knox%E6%B5%8B%E8%AF%95#linux%E6%A1%8C%E9%9D%A2%E4%B8%8Bspnego%E6%B5%8B%E8%AF%95firefox)"）。取消弹出窗口，进入登录表单，输入admin/vagrant2，点登录，就进入了FreeIPA的界面。  

卸载的命令：
```
$ iap-server-install --uninstall
$ yum erase freeipa-server
```
## 配置freeipa
刚安装的freeipa的kerberos访问控制列表如下:
```
$ cat /var/kerberos/krb5kdc/kadm5.acl
*/admin@EXAMPLE.COM     *
```
域明显没有设置正确,修改成:
```
#/admin@AMBARI.APACHE.ORG     *
```
如果不修改这个ACL配置文件，freeipa内置的KDC将无法作为Ambari启用kerberos的服务器。会报告的错误是：
```
017-06-13 12:06:29,528 - Failed to create principal, hdp33-061317@AMBARI.APACHE.ORG - Failed to create service principal for hdp33-061317@AMBARI.APACHE.ORG
STDOUT: Authenticating as principal admin@AMBARI.APACHE.ORG with password.
Password for admin@AMBARI.APACHE.ORG: 
Enter password for principal "hdp33-061317@AMBARI.APACHE.ORG": 
Re-enter password for principal "hdp33-061317@AMBARI.APACHE.ORG": 

STDERR: WARNING: no policy specified for hdp33-061317@AMBARI.APACHE.ORG; defaulting to no policy
add_principal: Operation requires ``add'' privilege while creating "hdp33-061317@AMBARI.APACHE.ORG".
```
## ambari-freeipa-service
[原文](https://github.com/hortonworks-gallery/ambari-freeipa-service)   
测试环境CENTOS7.0，三个VM分别是c7001/c7002/c7003。已经安装了Ambari和HDP集群，集群未启用kerberos，未安装OpenLDAP。  
在要安装freeiap的节点上（我选择c7003）添加163国内源，如果只是用国外源实测安装freeipa不成功：
```
$ wget -O /etc/yum.repos.d/CentOS7-Base-163.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo
$ yum update -y
```
实测中发现CentOS7-Base-163.repo中有个变量$releasever的返回值有问题。预期返回"7"，实际返回的是"7server"，只能手工修改repo文件。可能因为使用的box是"centos7.0"，安装的是oracle linux，不是“真正”的centos7有关:
```
$ sed -i 's/$releasever/7/' /etc/yum.repos.d/CentOS7-Base-163.repo          (由于$是特殊符号，只能用单引号)
```
在ambari所在的节点（我的是c7001）上，下载定制的Ambari服务freeipa：
```
$ VERSION=`hdp-select status hadoop-client | sed 's/hadoop-client - \([0-9]\.[0-9]\).*/\1/'`
$ sudo git clone https://github.com/imaidev/ambari-freeipa-service.git   /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/FREEIPA-DEMO 
$ ambari-server restart
```
浏览器输入地址Ambari(c7001.ambari.apache.org:8080)，登录后添加服务，选择FreeIPA Server，自动选择安装在节点c7001。但实测发现FreeIPA与Ambari有冲突（如freeipa的CA要占用8080端口），所以手工修改安装节点到c7003。参数配置：
```
【Advanced freeipa-config】
freeipa.server.admin.password: vagrant2                       (至少8位密码)
freeipa.server.dns.forwarder: 202.102.128.68
freeipa.server.dns.setup: false
freeipa.server.domain: ambari.apache.org
freeipa.server.ds.password: vagrant2                          (至少8位密码)
freeipa.server.hostname: c7003.ambari.apache.org
freeipa.server.master.password: vagrant2                      (至少8位密码)
freeipa.server.realm: AMBARI.APACHE.ORG
```

### Ambari 2.4 Kerberos with FreeIPA
[原文](https://community.hortonworks.com/articles/59645/ambari-24-kerberos-with-freeipa.html)  
HDP不支持kerberos凭据缓存的内存keyring存储。编辑ipa-server节点的/etc/krb5.conf文件，将:
```
default_ccache_name = KEYRING:persistent:%{uid}
```
修改为：
```
default_ccache_name = FILE:/tmp/krb5cc_%{uid}
```
#### 为ambari创建管理员用户
```
$ kinit admin@AMBARI.APACHE.ORG
$ ipa user-add hadoopadmin --first=Hadoop --last=Admin
$ ipa group-add-member admins --users=hadoopadmin
$ ipa passwd hadoopadmin         (密码是vagrant2，备忘)
```
新用户必须加入admins用户组，否则会权限不足，ambari在启用kerberos时需要新建kerberos主体的权限。  
Ambari还需要创建一个叫ambari-managed-principals的用户组。这个用户组目前还不能被Ambari的向导创建。创建用户组：
```
$ ipa group-add ambari-managed-principals
```
Because of the way FreeIPA automatically expires the new password, it is necessary to kinit as hadoopadmin and change the initial password. The password can be set to the same password unless the password policy prohibits password reuse:
FreeIPA由于安全的考虑，会让新密码自动过期，因此有必要用hadoopadmin用户去kinit，修改初始密码：
```
$ kinit hadoopadmin@AMBARI.APACHE.ORG
```
如果要使用FreeIPA的DNS就执行：
```
$ echo "nameserver $ipaserver_ip_address" > /etc/resolv.conf
```
在HDP集群的所有节点上安装ipa-client并将节点加入FreeIPA服务器：
```
$ yum -y install ipa-client
$ ipa-client-install --domain=ambari.apache.org \
    --server=c7004.ambari.apache.org \
    --realm=AMBARI.APACHE.ORG \
    --principal=hadoopadmin@AMBARI.APACHE.ORG \
    --enable-dns-updates
```
如果未使用DNS，最后这个参数```--enable-dns-updates```可以不加。  
如果ambari完全支持freeIPA时，以上的客户端估计会自动安装，就像自动安装kerberos客户端一样。现在只能手工装。  
还要在ambari服务器节点上安装：
```
$ yum -y install ipa-admintools
```
用浏览器打开URL(c7001是ambari服务器所在节点):
```
http://c7001.ambari.apache.org:8080/#/experimental
```
选中```enableipa```这个单选框。  
运行启用kerberos的向导，发现多一个选项：```Existing IPA```。选择这个选项，就可以用IPA充当KDC了。输入的管理员主体是hadoopadmin@AMBARI.APACHE.ORG，其他的与一般的启用kerberos一样。  

### KDC代理测试
参考[另一篇wiki文章](https://github.com/wbwangk/wbwangk.github.io/wiki/kerberos%E6%B5%8B%E8%AF%95#kerberos%E4%BB%A3%E7%90%86%E6%B5%8B%E8%AF%95)。  
### 使用LDAP工具
在windows下安装LDAP浏览器[JXplorer](http://www.jxplorer.org)。打开界面后输入c7004.ambari.apache.org（端口默认），就可以连接上FreeIPA的LDAP，查看其中内容。  