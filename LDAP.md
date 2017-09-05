## 概述
LDAP条目数据结构  
![](http://figures.oreilly.com/tagoreillycom20090511oreillybooks296268I_book_d1e1/figs/I_mediaobject_d1e40679-web.png)  
 - 条目(entry)  
条目由多个属性组成，属性包含了条目的数据。条目都有一个名称，叫DN。如果使用数据库属于，属性就像一条数据库记录的字段。
 - DN(distinguished name)  
DN是条目的唯一标识，是由逗号分隔的多个RDN组成的字符串。通过DN的层次型语法结构，可以方便地表示出条目在LDAP树中的位置，通常用于检索。
 - RDN(relative distinguished name)  
一个RDN是由一个或多个属性名称/值对组合。例如，```cn=Jay Sekora```可以是一个RDN(cn代表"通用名称common name")。属性名是```cn```，值是```Jay Sekora```。
 - 目录信息树(DIT)  
一个LDAP目录是一个由数据条目组成的树形结构，称为DIT。  
 - Base DN
LDAP目录树的最顶部就是根，也就是所谓的“Base DN"。
 - Schema
对象类（ObjectClass）、属性类型（AttributeType）、语法（Syntax）分别约定了条目、属性、值。所以这些构成了模式(Schema)——对象类的集合。条目数据在导入时通常需要接受模式检查，它确保了目录中所有的条目数据结构都是一致的。
 - LDIF
LDIF（LDAP Data Interchange Format，数据交换格式）是LDAP数据库信息的一种文本格式，用于数据的导入导出。

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
#### 常见属性(Attribute)的解释  
 - c：国家。
 - cn：common name，指一个对象的名字。如果指人，需要使用其全名。
 - dc：domain Component，常用来指一个域名的一部分。如imaicloud.com这个域名可以分解为dc=imaicloud,dc=com。
 - givenName：指一个人的名字，不能用来指姓。
 - l：location? 指一个地名，如一个城市或者其他地理区域的名字。
 - mail：电子信箱地址。
 - o：organizationName，指一个组织的名字。
 - ou：organizationalUnitName，指一个组织下级单元的名字，即组织下的部门。
 - sn：surname，指一个人的姓。
 - telephoneNumber：电话号码，应该带有所在的国家的代码。
 - uid：userid，通常指某个用户的登录名，与Linux系统中用户的uid不同。

既然LDAP用于表示用户，则用户DN可以由域名、组织、用户名三大部分组成，即一到几个DC，o或ou，cn。  

## 安装OpenLDAP
[ubuntu14官方文档的LDAP部分](https://help.ubuntu.com/14.04/serverguide/openldap-server.html)  
[OpenLDAP官方用户手册](http://www.openldap.org/software/man.cgi)(点击Section Indexes后面的数字)  
[OpenLDAP管理员文档](http://www.openldap.org/doc/)  
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
$ service slapd status
 * slapd is running
$ netstat -anp | grep 389
tcp        0      0 0.0.0.0:389             0.0.0.0:*               LISTEN      5860/slapd
```
ldapsearch是ldap搜索命令(搜索cn=config目录下的条目)：
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
(-Y指定了SASL认证机制；-Q启用SASL静默模式；-LLL控制输出到屏幕上内容的明细程度；-H指定了LDAP服务器的URI；-b指定搜索的基础目录；dn是被搜索的内容，如果去掉dn，则显示出了包括属性在内的全部内容)  
OpenLDAP可以通过多种后端存储数据。默认后端使用linux文件系统，存储为LDIF格式的文本文件。  

测试一下本机的DIT(dc=ambari,dc=apache,dc=org):
```
$ ldapsearch -x -LLL -H ldap:/// -b dc=ambari,dc=apache,dc=org dn
dn: dc=ambari,dc=apache,dc=org
dn: cn=admin,dc=ambari,dc=apache,dc=org
```
```dc=ambari,dc=apache,dc=org```是DIT的基础目录。OpenLDAP貌似会为本机根据/etc/hosts的内容自动创建条目。  
```cn=admin,dc=ambari,dc=apache,dc=org```是这个DIT的管理员(rootDN)。这个管理员是安装过程中创建的。  

带过滤器的ldapsearch命令：
```
$ ldapsearch -x -LLL -b dc=ambari,dc=apache,dc=org -s sub '(&(uid=webb2)(objectclass=posixAccount)(uid=*)(&(uidNumber=*)(!(uidNumber=0))))'
dn: uid=webb2,ou=People,dc=ambari,dc=apache,dc=org
objectClass: account
objectClass: posixAccount
cn: webb2
uid: webb2
uidNumber: 10003
gidNumber: 5002
homeDirectory: /home/webb2
loginShell: /bin/sh
gecos: webb2
description: User account
```
### 向LDAP数据库中添加记录
OpenLDAP的默认后端会把数据存放在LDIF格式的文本文件中。下面测试通过创建LDIF格式的文件来向数据库中添加条目。  
将添加的内容有：
 1. 一个叫*People*的节点，用于存放用户
 2. 一个叫*Groups*的节点，用于存放用户组
 3. 一个叫*miners*的用户组
 4. 一个叫*john*的用户
首先创建一个LDIF文件叫```add_content.ldif```（我把这个文件放在了/etc/ldap/slap.d/目录下）:
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
objectClass定义了条目类型，如posixAccount表示这是个操作系统用户。  
执行下列命令将上述文件中的定义添加到LDAP数据库：
```
$ ldapadd -x -D cn=admin,dc=ambari,dc=apache,dc=org -W -f add_content.ldif
Enter LDAP Password:                           (输入安装OpenLDAP过程中创建的密码)
adding new entry "ou=People,dc=ambari,dc=apache,dc=org"
adding new entry "ou=Groups,dc=ambari,dc=apache,dc=org"
adding new entry "cn=miners,ou=Groups,dc=ambari,dc=apache,dc=org"
adding new entry "uid=john,ou=People,dc=ambari,dc=apache,dc=org"
```
（-x表示使用simple认证来代替SASL；-D指定LDAP目录，如果SASL认证则忽略这个值；-W表示提示输入simple认证的密码；-f表示从文件读入命令，否则会从交互式命令行中读取命令）  
利用*ldapsearch*命令检查一下新增用户的信息：
```
$ ldapsearch -x -LLL -b dc=ambari,dc=apache,dc=org 'uid=john' cn gidNumber
dn: uid=john,ou=People,dc=ambari,dc=apache,dc=org
cn: John Doe
gidNumber: 5000
$ ldapsearch -x -LLL -H ldap:/// -b dc=ambari,dc=apache,dc=org 'uid=john' cn gidNumber   (多了个-H参数而已，搜索结果一样)
$ ldapsearch -x -LLL -H ldap://u1401.ambari.apache.org -b dc=ambari,dc=apache,dc=org 'uid=john' cn gidNumber  (也一样)
```
uid=john：一个查找用户john的“过滤器”；  
cn gidNumber：请求和显示特定属性（默认是显示全部属性）。   

### 修改slapd配置数据库
RDN为```cn=config```的目录成为slapd-config DIT，是slapd的配置数据库。  
(一). 使用*ldapmodify*增加一个"索引"(DbIndex属性)到*{1}hdb,cn=config*数据库。创建一个叫uid_index.ldif的文件，包括以下内容：
```
dn: olcDatabase={1}hdb,cn=config
add: olcDbIndex
olcDbIndex: uid eq,pres,sub
```
执行以下命令：
```
$ sudo ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f uid_index.ldif
modifying entry "olcDatabase={1}hdb,cn=config"
```
可以执行下列命令来确认修改是否成功：
```
$ sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b \
     cn=config '(olcDatabase={1}hdb)' olcDbIndex
dn: olcDatabase={1}hdb,cn=config
olcDbIndex: objectClass eq
olcDbIndex: uid eq,pres,sub
```
(二). 添加一个schema。为此，首先要转换到LDIF格式。你可以在/etc/ldap/schema目录下发现没有转换的schema。  
在增加schema前，可以查看一下已经安装的schema：
```
$ sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=schema,cn=config dn
dn: cn=schema,cn=config
dn: cn={0}core,cn=schema,cn=config
dn: cn={1}cosine,cn=schema,cn=config
dn: cn={2}nis,cn=schema,cn=config
dn: cn={3}inetorgperson,cn=schema,cn=config
```
在下面的例子中我们增加一个CORBA schema。  
（略，因为与后面的“kerberos和LDAP”操作近似）  
（注：SASL有几大工业实现标准：Kerveros V5、DIGEST-MD5、EXTERNAL、PLAIN、LOGIN。EXTERNAL一般用于初始化添加schema时使用。）  

### LDAP认证
启动一个专门的虚拟机u1410来安装LDAP客户端，*libnss-ldap*：
```
$ sudo apt install libnss-ldap
```
当提示让输入LDAP服务器URI时输入：```ldap://u1401.ambari.apache.org```；  
当提示让输入Search Base时输入：```dc=ambari,dc=apache,dc=org```；  
LDAP版本选择3； Make local root Database admin时选择yes；  
Does the LDAP database require login? 选择no；  
LDAP account for root时输入：``` cn=admin,dc=ambari,dc=apache,dc=org```；  
最后输入root口令，输入了1。  
如果想重新设置可以这样：
```
$ sudo dpkg-reconfigure ldap-auth-config
```
界面配置的结果存放在/etc/ldap.conf文件中。下面为NSS配置LDAP profile：
```
$ sudo auth-client-config -t nss -p lac_ldap
```
配置系统使用LDAP做认证：
```
$ sudo pam-auth-update
```
在弹出的窗口中需要保证LDAP认证的选项被选中。现在可以使用LDAP做认证了。  
查看一下/etc/ldap.conf，看是否存在以下配置：
```
uri ldap://u1401.ambari.apache.org
```
实测发现，如果配置成```uri ldapi://u1401.ambari.apache.org```不行。估计是LDAP Server没有正确启用TLS。  
现在测试一下使用LDAP的用户登录：
```
$ sudo su - john
No directory, logging in with HOME=/
john@u1410:/$
```
john这个用户是之前的测试添加到LDAP数据库中的。由于在当前操作系统下没有/home/john目录，所以提示john用户的HOME目录变成了根目录。

### ldap工具ldapscritps
先在u1401上安装一个需要用到的软件包*ldapscritps*([1](https://github.com/martymac/ldapscripts))：
```
$ sudo apt install ldapscripts
```
centos7.3下安装*ldapscritps*([1](http://rpmfind.net/linux/rpm2html/search.php?query=ldapscripts))：
```
$ wget ftp://rpmfind.net/linux/Mandriva/devel/cooker/x86_64/media/contrib/release/ldapscripts-1.10.0-1-mdv2011.0.noarch.rpm
$ yum install ldapscripts-1.10.0-1-mdv2011.0.noarch.rpm 
```
编辑配置文件/etc/ldapscripts/ldapscripts.conf，修改配置为以下的样子：
```
SERVER="ldap://localhost"
BINDDN='cn=admin,dc=ambari,dc=apache,dc=org'
BINDPWDFILE="/etc/ldapscripts/ldapscripts.passwd"
SUFFIX='dc=ambari,dc=apache,dc=org'
GSUFFIX='ou=Groups'
USUFFIX='ou=People'
MSUFFIX='ou=Computer'
GIDSTART=5000
UIDSTART=10000
MIDSTART=20000
```
(测试中发现配置项`ICONVCHAR`的注释需要去掉！)  
创建密码文件，并确保这个文件只能被rootDN读取：
```
sudo sh -c "echo -n '1' > /etc/ldapscripts/ldapscripts.passwd"
sudo chmod 400 /etc/ldapscripts/ldapscripts.passwd
```
```1```是我的rootDN密码，是安装slapd过程中弹出提示窗口中输入的那个密码。如果你创建的rootDN密码不是1，就改成你自己设置的密码。  
创建一个新用户并创建密码，miners是用户组（这个用户组是前文创建的）：
```
$ sudo ldapadduser george miners
$ sudo ldapsetpasswd george
$ ldapsearch -x -LLL -b dc=ambari,dc=apache,dc=org george  (在ldap数据库中搜索刚刚创建的用户)
```
创建用户组：
```
$ sudo ldapaddgroup qa
```
用ldapsearch也可以搜索到ldap数据库中的qa用户组条目。   
将用户添加到用户组：
```
$ ldapaddusertogroup george miners
```
添加用户到用户组后，用户组miners条目会增加一个memberUid属性(组中每增加一个用户就多一个属性)：
```
memberUid: george
```
## Kerberos与LDAP
要将Kerberos与LDAP进行集成，首先需要在LDAP服务器上安装```krb5-kdc-ldap```包：
```
$ sudo apt install krb5-kdc-ldap
```
需要将kerberos的相关schema加载到LDAP数据库中。为此，解压```kerberos.schema.gz```:
```
$ sudo gzip -d /usr/share/doc/krb5-kdc-ldap/kerberos.schema.gz
$ sudo cp /usr/share/doc/krb5-kdc-ldap/kerberos.schema /etc/ldap/schema/
```
再后，kerberos schema需要添加到```cn-config```树中。  
 1. 创建一个配置文件schema_convert.conf，内容如下：
```
include /etc/ldap/schema/core.schema
include /etc/ldap/schema/collective.schema
include /etc/ldap/schema/corba.schema
include /etc/ldap/schema/cosine.schema
include /etc/ldap/schema/duaconf.schema
include /etc/ldap/schema/dyngroup.schema
include /etc/ldap/schema/inetorgperson.schema
include /etc/ldap/schema/java.schema
include /etc/ldap/schema/misc.schema
include /etc/ldap/schema/nis.schema
include /etc/ldap/schema/openldap.schema
include /etc/ldap/schema/ppolicy.schema
include /etc/ldap/schema/ldapns.schema
include /etc/ldap/schema/pmi.schema
include /etc/ldap/schema/kerberos.schema
```
注意，上述文件内容与[ubuntu官方的](https://help.ubuntu.com/lts/serverguide/kerberos-ldap.html)略有不同，比官方的多了2条记录，从而导致后续的索引编号由12变成了14。上述配置文件有15行，从0开始kerberos.schema位于序号是14的行。
 2. 创建一个临时目录存放LDIF格式文件：
```
$ mkdir /tmp/ldif_output
``` 
 3. 使用*slapcat*去转换schema文件：
```
slapcat -f schema_convert.conf -F /tmp/ldif_output -n0 -s \
"cn={14}kerberos,cn=schema,cn=config" > /tmp/cn=kerberos.ldif
```
官方文档中是```{12}```。
 4. 编辑``` /tmp/cn\=kerberos.ldif```文件，修改下列属性(即删除```{14}```)：
```
dn: cn=kerberos,cn=schema,cn=config
...
cn: kerberos
```
删除文件最后类似的下列7行：
```
structuralObjectClass: olcSchemaConfig
entryUUID: 18ccd010-746b-102d-9fbe-3760cca765dc
creatorsName: cn=config
createTimestamp: 20090111203515Z
entryCSN: 20090111203515.326445Z#000000#000#000000
modifiersName: cn=config
modifyTimestamp: 20090111203515Z
```
 5. 使用*ldapadd*命令加载新的schema：
```
$ sudo ldapadd -Q -Y EXTERNAL -H ldapi:/// -f /tmp/cn\=kerberos.ldif 
```
这个命令也与官方文档不同。官方文档使用simple认证，但提示```ldap_bind: Invalid credentials (49)```，猜测是没有创建```cn=admin,cn=config```这个rootDN用户。官方的命令行：
```
ldapadd -x -D cn=admin,cn=config -W -f /tmp/cn\=kerberos.ldif
```  
 6. 为*krb5principalname*属性添加一个索引：
```
$ sudo ldapmodify -Q -Y EXTERNAL -H ldapi:///           (进入交互命令行模式)
dn: olcDatabase={1}hdb,cn=config
add: olcDbIndex
olcDbIndex: krbPrincipalName eq,pres,sub
                                                        (回车两次，看来回车两次可以提交上述命令)
modifying entry "olcDatabase={1}hdb,cn=config"
```
上述命令也与官方文档不同。  
可以用下列命令来检查添加的索引：
```
$ sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config '(olcDatabase={1}hdb)' olcDbIndex
dn: olcDatabase={1}hdb,cn=config
olcDbIndex: objectClass eq
olcDbIndex: uid eq,pres,sub
olcDbIndex: krbPrincipalName eq,pres,sub
```
 7. 最后更新访问控制列表(ACL)
先查看一下现有ACL:
```
$ sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b \
    cn=config '(olcDatabase={1}hdb)' olcAccess
dn: olcDatabase={1}hdb,cn=config
olcAccess: {0}to attrs=userPassword,shadowLastChange by self write by anonymou
 s auth by dn="cn=admin,dc=ambari,dc=apache,dc=org" write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=admin,dc=ambari,dc=apache,dc=org" write by * read
```
利用交互式命令行输入更新后的ACL:
```
$ sudo ldapmodify -Q -Y EXTERNAL -H ldapi:///           (进入交互命令行模式)
dn: olcDatabase={1}hdb,cn=config
replace: olcAccess
olcAccess: to attrs=userPassword,shadowLastChange,krbPrincipalKey by dn="cn=admin,dc=ambari,dc=apache,dc=org" write by anonymous auth by self write by * none
-
add: olcAccess
olcAccess: to dn.base="" by * read
-
add: olcAccess
olcAccess: to * by dn="cn=admin,dc=ambari,dc=apache,dc=org" write by * read
                                              （回车两次）
modifying entry "olcDatabase={1}hdb,cn=config"              （ctrl+D返回到操作系统）
```
注意，第一个OlcAccess后面的内容很长，中间不要有换行，否则两次回车后会报错。进入命令行状态后（没有任何提示，只是光标进入下一行），直接把“两次回车”之前的内容粘贴即可。可以用ldapsearch命令检查修改的结果。  
完成上述7个步骤后，LDAP将可以作为kerberos的主体数据库了。  

## kerberos与LDAP(centos)
本章的主要目的是将kerberos的数据库由默认的本地文件系统更换为LDAP服务器(LDAP可能部署在其他节点)。主要工作分成n个部分：
1. 将kerberos Schema导入到LDAP服务器，以便按规定格式存放KDC数据  
2. 更改LDAP访问控制列表(ACL)，以便kerberos可以读写LDAP中的数据  
3. 用 kdb5_ldap_util在LDAP创建kerberos领域  

测试环境：centos7.3(使用了节点c7306)，已安装kerberos KDC和OpenLDAP。kerberos KDC安装好后，默认使用文件系统作为数据存储。前面的章节“kerberos与LDAP”讲的是ubuntu下将KDC存储更换为LDAP。本章讲的是如何在centos下将KDC后台存储更换为LDAP。centos下的操作与ubuntu基本一样，只在细节上略有差异。  
  
### 1.导入kerberos Schema到LDAP
随`krb5-server-ldap`安装包带了kerberos的LDAP schema，将它复制到openldap相应目录下：
```
# cp /usr/share/doc/krb5-server-ldap-1.14.1/kerberos.schema /etc/openldap/schema
```
openldap的schema目录(/etc/openldap/schema)下有很多.schema文件，但不见得都已经导入了数据库。可以参考博文《[LDAP入门](https://imaidata.github.io/blog/ldap/)》，在博文中导入了3个schema。也可以这样看一下哪些schema已经导入到了数据库中：
```
# ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=schema,cn=config dn: cn=
dn: cn=schema,cn=config
dn: cn={0}core,cn=schema,cn=config
dn: cn={1}cosine,cn=schema,cn=config
dn: cn={2}nis,cn=schema,cn=config
dn: cn={3}inetorgperson,cn=schema,cn=config
```
core是LDAP默认带的，另3个schema是《LDAP入门》中导入到数据库中的，现在要导入的是第5个shema。  

创建一个 schema_convert.conf，包括下列内容：
```
include /etc/openldap/schema/core.schema
include /etc/openldap/schema/cosine.schema
include /etc/openldap/schema/nis.schema
include /etc/openldap/schema/inetorgperson.schema
include /etc/openldap/schema/kerberos.schema
```
然后执行：
```
# mkdir /tmp/ldif_output
# slapcat -f schema_convert.conf -F /tmp/ldif_output -n0 -s "cn={4}kerberos,cn=schema,cn=config" > /tmp/cn=kerberos.ldif
```
编辑`/tmp/cn\=kerberos.ldif`文件，将内容修改成类似下列的样子(就是把`{4}`删除):
```
dn: cn=kerberos,cn=schema,cn=config
...
cn: kerberos
```
并且删除文件最后几行，删除的几行大约的样子：
```
structuralObjectClass: olcSchemaConfig
entryUUID: 18ccd010-746b-102d-9fbe-3760cca765dc
creatorsName: cn=config
createTimestamp: 20090111203515Z
entryCSN: 20090111203515.326445Z#000000#000#000000
modifiersName: cn=config
modifyTimestamp: 20090111203515Z
```
将这个ldif文件导入到LDAP数据库中：
```
# ldapadd -Q -Y EXTERNAL -H ldapi:/// -f /tmp/cn\=kerberos.ldif
```
为`krb5principalname`属性增加索引(交互式命令行，也可以把输入的内容放入文件，然后用`-f`参数从文件读入)：
```
# ldapmodify -Q -Y EXTERNAL -H ldapi:///

dn: olcDatabase={2}hdb,cn=config
add: olcDbIndex
olcDbIndex: krbPrincipalName eq,pres,sub

modifying entry "olcDatabase={2}hdb,cn=config"
```
### 2.修改LDAP ACL
创建一个临时文件krb5.acl：
```
dn: olcDatabase={2}hdb,cn=config
replace: olcAccess
olcAccess: to attrs=userPassword,shadowLastChange,krbPrincipalKey by dn="cn=admin,dc=ambari,dc=apache,dc=org" write by anonymous auth by self write by * none

dn: olcDatabase={2}hdb,cn=config
add: olcAccess
olcAccess: to dn.base="" by * read

dn: olcDatabase={2}hdb,cn=config
add: olcAccess
olcAccess: to * by dn="cn=admin,dc=ambari,dc=apache,dc=org" write by * read
```
将上述acl导入到数据库中：
```
# ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f krb5.acl
modifying entry "olcDatabase={2}hdb,cn=config"
```
### 3.为kerberos生成领域

安装kerberos与LDAP集成的软件包：
```
# yum -y install krb5-server-ldap
```
kerberos有两个配置文件：`/etc/krb5.conf`和`/var/kerberos/krb5kdc/kdc.conf`。这两个配置文件最终会合并在一起。MIT kerberos官方文档《[Configuring Kerberos with OpenLDAP back-end](https://web.mit.edu/kerberos/krb5-latest/doc/admin/conf_ldap.html)》中使用的`kdc.conf`，而ubuntu的[官方文档](https://help.ubuntu.com/lts/serverguide/kerberos-ldap.html)中使用的`/etc/krb5.conf`。本章采用的方案是`krb5.conf`。  

编辑KDC配置文件`/etc/krb5.conf`为下面的样子：
```
# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 default_realm = AMBARI.APACHE.ORG
 default_ccache_name = KEYRING:persistent:%{uid}

[realms]
 AMBARI.APACHE.ORG = {
  kdc = c7306.ambari.apache.org
  admin_server = c7306.ambari.apache.org
  default_domain = ambari.apache.org
  database_module = openldap_ldapconf
 }

[domain_realm]
 .ambari.apache.org = AMBARI.APACHE.ORG
 ambari.apache.org = AMBARI.APACHE.ORG

[dbdefaults]
        ldap_kerberos_container_dn = cn=krbContainer,dc=ambari,dc=apache,dc=org

[dbmodules]
        openldap_ldapconf = {
                db_library = kldap
                ldap_kdc_dn = "cn=admin,dc=ambari,dc=apache,dc=org"

                # this object needs to have read rights on
                # the realm container, principal container and realm sub-trees
                ldap_kadmind_dn = "cn=admin,dc=ambari,dc=apache,dc=org"

                # this object needs to have read and write rights on
                # the realm container, principal container and realm sub-trees
                ldap_service_password_file = /var/kerberos/krb5kdc/service.keyfile
                ldap_servers = ldap://c7306.ambari.apache.org/
                ldap_conns_per_server = 5
        }
```

用`kdb5_ldap_util`工具创建领域(realm):
```
# kdb5_ldap_util -D cn=admin,dc=ambari,dc=apache,dc=org create -subtrees dc=ambari,dc=apache,dc=org -r AMBARI.APACHE.ORG -s -H ldap://c7306.ambari.apache.org/
```
先输入的是DN(cn=admin,dc=ambari,dc=apache,dc=org)的密码，然后输入领域的主密码。  

生成管理员DN(cn=admin,dc=ambari,dc=apache,dc=org)的密码文件。因为kerberos需要用管理员权限来访问LDAP。
```
# kdb5_ldap_util -D cn=admin,dc=ambari,dc=apache,dc=org -w 1 stashsrvpw -f /var/kerberos/krb5kdc/service.keyfile cn=admin,dc=ambari,dc=apache,dc=org
```
在`krb5.conf`配置文件引用了这个密码文件。在[另一篇](http://secfree.github.io/blog/2015/06/29/kerberos-ldap-deploy.html)参考的文章中创建了独立的DN和独立的密码文件，不和管理员用同一个DN，这样无疑更安全。  

别忘记检查`/var/kerberos/krb5kdc/kdc.conf`中的配置，需要把`EXAMPLE.COM`替换成`AMBARI.APACHE.ORG`。  
启用kerberos服务：
```
# service krb5kdc start
# service kadmin start
```
### 测试一下
创建一个主体(principal)：
```
# kadmin.local
Authenticating as principal root/admin@AMBARI.APACHE.ORG with password.
kadmin.local:  addprinc steve
WARNING: no policy specified for steve@AMBARI.APACHE.ORG; defaulting to no policy
Enter password for principal "steve@AMBARI.APACHE.ORG":
Re-enter password for principal "steve@AMBARI.APACHE.ORG":
Principal "steve@AMBARI.APACHE.ORG" created.
```
用LDAP客户端查询一下刚刚创建的主体：
```
# ldapsearch -x -b cn=krbContainer,dc=ambari,dc=apache,dc=org
# steve@AMBARI.APACHE.ORG, AMBARI.APACHE.ORG, krbContainer, ambari.apache.org
dn: krbPrincipalName=steve@AMBARI.APACHE.ORG,cn=AMBARI.APACHE.ORG,cn=krbContai
 ner,dc=ambari,dc=apache,dc=org
krbLoginFailedCount: 0
krbPrincipalName: steve@AMBARI.APACHE.ORG
krbLastPwdChange: 20170904074817Z
krbExtraData:: AAJBBa1Za2FkbWluL2FkbWluQEFNQkFSSS5BUEFDSEUuT1JHAA==
krbExtraData:: AAgBAA==
objectClass: krbPrincipal
objectClass: krbPrincipalAux
objectClass: krbTicketPolicyAux
```
`cn=krbContainer,dc=ambari,dc=apache,dc=org`目录下的条目很多，上面仅显示出来刚刚创建的`steve`条目。  
用steve登录：
```
$ kinit steve
Password for steve@AMBARI.APACHE.ORG:
$ klist
Ticket cache: KEYRING:persistent:0:0
Default principal: steve@AMBARI.APACHE.ORG

Valid starting       Expires              Service principal
09/04/2017 09:05:28  09/05/2017 09:05:28  krbtgt/AMBARI.APACHE.ORG@AMBARI.APACHE.ORG
```
在[这篇文章](http://blog.clanzx.net/2013/09/27/ldap-kerberos.html)中讲解了如何将LDAP的用户条目密码设置为：`{SASL}test@DEMO.LOCAL`，然后通过SASL让kerberos去验证用户密码。  

## OpenLDAP日志
OpenLDAP logs via syslogd (using LOCAL4)。参考[loglevel](http://www.zytrax.com/books/ldap/ch6/#loglevel)  
centos7内置的日志系统是rsyslog，它的主配置文件是` /etc/rsyslog.conf`。在该配置文件上增加：
```
local4.* /var/log/ldap.log
```
手工创建该日志：  
```
$ touch /var/log/ldap.log
$ service rsyslog restart
```
## knox自带sample
knox在目录`/usr/hdp/2.6.1.0-129/knox/conf`下有个users.ldif文件，里面的数据带有用户和用户组，可以用来测试knox与LDAP的集成。现将这个文件进行修改，只留下用户、用户组的定义(users2.ldif)：
```ldif
version: 1
# entry for sample user sam
dn: uid=sam,ou=People,dc=ambari,dc=apache,dc=org
objectclass:top
objectclass:person
objectclass:organizationalPerson
objectclass:inetOrgPerson
cn: sam
sn: sam
uid: sam
userPassword:1

# entry for sample user tom
dn: uid=tom,ou=People,dc=ambari,dc=apache,dc=org
objectclass:top
objectclass:person
objectclass:organizationalPerson
objectclass:inetOrgPerson
cn: tom
sn: tom
uid: tom
userPassword:1

# create the analyst group under groups
dn: cn=analyst,ou=Groups,dc=ambari,dc=apache,dc=org
objectclass:top
objectclass: groupofnames
cn: analyst
description:analyst  group
member: uid=sam,ou=People,dc=ambari,dc=apache,dc=org
member: uid=tom,ou=People,dc=ambari,dc=apache,dc=org

# create the scientist group under groups
dn: cn=scientist,ou=Groups,dc=ambari,dc=apache,dc=org
objectclass:top
objectclass: groupofnames
cn: scientist
description: scientist group
member: uid=sam,ou=People,dc=ambari,dc=apache,dc=org
```
将上述文件导入到LDAP中：
```
$ sudo ldapadd -x -D cn=admin,dc=ambari,dc=apache,dc=org -W -f users2.ldif 
```

## LDAP工具
JXplorer是一个开源的LDAP浏览器，支持多种操作系统。官网：[jxplorer.org](http://www.jxplorer.org/)  
[Ldap Admin](http://www.ldapadmin.org/download/ldapadmin.html)是一个免费的windows客户端和管理员工具。  
