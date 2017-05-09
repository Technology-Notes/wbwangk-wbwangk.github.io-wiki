## 概述
[来源](http://archive.oreilly.com/pub/a/perl/excerpts/system-admin-with-perl/ten-minute-ldap-utorial.html)  
LDAP条目数据结构  
![](http://figures.oreilly.com/tagoreillycom20090511oreillybooks296268I_book_d1e1/figs/I_mediaobject_d1e40679-web.png)  
 - 条目(entry)  
条目由多个属性组成，属性包含了条目的数据。条目都有一个名称，叫DN。如果使用数据库属于，属性就像一条数据库记录的字段。
 - DN(distinguished name)  
DN是由逗号分隔的多个RDN组成的字符串。DN是唯一标识符，有时称为目录条目的主键。
 - RDN(relative distinguished name)  
一个RDN是由一个或多个属性名称/值对组合。例如，```cn=Jay Sekora```可以是一个RDN(cn代表"通用名称common name")。属性名是```cn```，值是```Jay Sekora```。
 - 目录信息树(DIT)  
一个LDAP目录是一个由数据条目组成的树形结构，称为DIT。  

遍历树形去生成一个DN：  
![](http://figures.oreilly.com/tagoreillycom20090511oreillybooks296268I_book_d1e1/figs/I_mediaobject_d1e40961-web.png)  
上图中左边树形的DN是：  
```
cn=Robert Smith, l=main campus, ou=CCS, o=Hogwarts School, c=US
```
右边树形的DN是：
```
uid=rsmith, ou=system, ou=people, dc=ccs, dc=hogwarts, dc=edu
```
几个缩写的解释：  
```
ou => organizational unit, o => organization, dc => “domain component”, c =>country 
cn => Common Name (user name / server name)
```
domain domponent指互联网域名组件，如imaicloud.com这个域名可以分解为dc=imaicloud,dc=com。  
既然LDAP用于表示用户，则用户DN可以由域名、组织、用户名三大部分组成，即一到几个DC，o或ou，cn。  

## 安装OpenLDAP
[来源](https://help.ubuntu.com/lts/serverguide/openldap-server.html)  
本测试使用的[大数据本地开发环境](https://github.com/imaidev/imaidev.github.io/wiki/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%9C%AC%E5%9C%B0%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)的u1401主机。查看一下主机的域名定义：
```
$ cat /etc/hosts
192.168.14.101 u1401.ambari.apache.org u1401                   (其他内容的略)
```
以上设置表明，将安装的LDAP数据的后缀是：```dc=ambari,dc=apache,dc=org```。  
安装OpenLDAP:
```
$ sudo apt install slapd ldap-utils
```
安装过程中会要求输入管理员用户的密码。该管理员用户的的DN是:```cn=admin,dc=ambari,dc=apache,dc=org```。安装后的LDAP数据库位于```/etc/ldap/slapd.d/```。  
slapd是openldap的后台进程名，该进程默认监听389端口：
```
$ netstat -anp | grep 389
tcp        0      0 0.0.0.0:389             0.0.0.0:*               LISTEN      5860/slapd
```
ldapsearch是ldap搜索命令：
```
$ sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config dn
dn: cn=config
dn: cn=module{0},cn=config
dn: cn=schema,cn=config
dn: cn={0}core,cn=schema,cn=config
dn: cn={1}cosine,cn=schema,cn=config
dn: cn={2}nis,cn=schema,cn=config
dn: cn={3}inetorgperson,cn=schema,cn=config
dn: olcBackend={0}hdb,cn=config
dn: olcDatabase={-1}frontend,cn=config
dn: olcDatabase={0}config,cn=config
dn: olcDatabase={1}hdb,cn=config
```
OpenLDAP使用linux文件系统充当了数据库结构。直接通过文件系统可以看到上述目录或文件。  

测试一下本机的DIT(dc=ambari,dc=apache,dc=org):
```
$ ldapsearch -x -LLL -H ldap:/// -b dc=ambari,dc=apache,dc=org bn
dn: dc=ambari,dc=apache,dc=org
dn: cn=admin,dc=ambari,dc=apache,dc=org
```
```dc=ambari,dc=apache,dc=org```是DIT的基础（根目录？）
```cn=admin,dc=ambari,dc=apache,dc=org```是这个DIT的管理员(rootDN)。这个管理员是安装过程中创建的。  

## 向LDAP数据库中添加记录
将添加的内容有：
 1. 一个叫*People*的节点，用于存放用户
 2. 一个叫*Groups*的节点，用于存放用户组
 3. 一个叫*miners*的用户组
 4. 一个叫*john*的用户
首先创建一个LDIF文件叫```add_content.ldif```:
```
dn: ou=People,dc=ambari,dc=apache,dc=org
objectClass: organizationalUnit
ou: People

dn: ou=Groups,dc=ambari,dc=apache,dc=org
objectClass: organizationalUnit
ou: Groups

dn: cn=miners,ou=Groups,dc=ambari,dc=apache,dc=org
objectClass: posixGroup
cn: miners
gidNumber: 5000

dn: uid=john,ou=People,dc=ambari,dc=apache,dc=org
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: john
sn: Doe
givenName: John
cn: John Doe
displayName: John Doe
uidNumber: 10000
gidNumber: 5000
userPassword: johnldap
gecos: John Doe
loginShell: /bin/bash
homeDirectory: /home/john
```
执行下列命令将上述文件中的定义添加到LDAP数据库：
```
$ ldapadd -x -D cn=admin,dc=ambari,dc=apache,dc=org -W -f add_content.ldif
Enter LDAP Password:                           (输入安装slapd过程中输入的密码)
adding new entry "ou=People,dc=ambari,dc=apache,dc=org"
adding new entry "ou=Groups,dc=ambari,dc=apache,dc=org"
adding new entry "cn=miners,ou=Groups,dc=ambari,dc=apache,dc=org"
adding new entry "uid=john,ou=People,dc=ambari,dc=apache,dc=org"
```
利用*ldapsearch*命令检查一下新增用户的信息：
```
$ ldapsearch -x -LLL -b dc=ambari,dc=apache,dc=org 'uid=john' cn gidNumber
dn: uid=john,ou=People,dc=ambari,dc=apache,dc=org
cn: John Doe
gidNumber: 5000
```

## Kerberos与LDAP
要将Kerberos与LDAP进行集成，首先需要在LDAP服务器上安装```krb5-kdc-ldap```包：
```
$ sudo apt install krb5-kdc-ldap
```
然后，解压```kerberos.schema.gz```:
```
$ sudo gzip -d /usr/share/doc/krb5-kdc-ldap/kerberos.schema.gz
$ sudo cp /usr/share/doc/krb5-kdc-ldap/kerberos.schema /etc/ldap/schema/
```
再后，kerberos schema需要添加到```cn-config```树中。

 
