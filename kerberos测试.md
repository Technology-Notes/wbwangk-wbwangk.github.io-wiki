## Kerberos安装
[参考](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.0/bk_security/content/_kerberos_overview.html)  
[一个kerberos入门](https://steveloughran.gitbooks.io/kerberos_and_hadoop/content/)  

#### 1.安装KDC Server
在u1404（非HDP集群节点）安装Install the KDC Server：
```
$ apt-get install krb5-kdc krb5-admin-server
```
第一次尝试时，提示krb5-user依赖冲突。用手机当热点执行apt-get update后。  
安装过程中出现提示窗口让输入Default Kerberos version 5 realm，保留默认值AMBARI.APACHE.ORG。然后出现两次让输入hostname，都输入的"u1404.ambari.apache.org"。最后提示说这个向导没有自动建立一个kerberos realm，如果想建立就执行命令"krb5_newrealm"。相关说明在/usr/share/doc/krb5-kdc/README.KDC中。  
```
$ krb5_newrealm
master key name 'K/M@AMBARI.APACHE.ORG'
Enter KDC database master key:    (输入两次密码，密码是vagrant)
```
必须使用krb5_newrealm创建realm，否则无法启动下列服务。  
启动KDC server和KDC admin server：
```
$ service krb5-kdc restart                     （如果不执行krb5_newrealm就无法启动这个服务）
$ service krb5-admin-server restart            （如果不执行krb5_newrealm就无法启动这个服务）
```
#### 2.创建Kerberos Admin
通过创建admin主体来建立KDC admin：
```
$ kadmin.local -q "addprinc root/admin"         (输入两次密码，密码是vagrant)
```
将刚创建的admin主体添加到KDC ACL中：
```
$ echo "*/admin@AMBARI.APACHE.ORG *" >> /etc/krb5kdc/kadm5.acl
$ service krb5-admin-server restart
```
#### 3. 简单测试
查看主体清单的方法：在u1404节点（安装KDC的节点）上执行：
```
$ kadmin.local
kadmin.local: list_principals  
HTTP/u1402.ambari.apache.org@AMBARI.APACHE.ORG
HTTP/u1403.ambari.apache.org@AMBARI.APACHE.ORG
K/M@AMBARI.APACHE.ORG
admin/admin@AMBARI.APACHE.ORG
(略)
```
#### 4.kerberos客户端安装
ubuntu下安装kerberos客户端：
```
$ apt install krb5-user  (如果提示包依赖错误，就用手机上网执行apt-get udpate)
```
centos下安装kerberos客户端：
```
$ yum install krb5-workstation
```
安装过程中如果让输入KDC，则输入```u1404.ambari.apache.org```；如果让输入realm则输入```AMBARI.APACHE.ORG```；如果让输入管理服务器也输入地址```u1404.ambari.apache.org```。  

#### 5.kerberos客户端的使用
kerberos客户端的常用命令是kinit、klist、kdestroy、kpasswd。
kinit用户登录。登录过程就是客户端从KDC获取票据TGT的过程。登录成功后，票据被缓存在本地。实际上就是建立了安全会话。  
klist用户查看当前票据缓存中内容。  
kdestroy用于退出登录，即销毁缓存中票据。  
kpasswd用于修改用户(主体)口令。  
```
$ klist
klist: No credentials cache found (ticket cache FILE:/tmp/krb5cc_0)
```
上面的结果表明缓存文件中没有票据。  
```
$ kinit webb
Password for webb@AMBARI.APACHE.ORG:(输入密码)
$ klist
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: webb@AMBARI.APACHE.ORG

Valid starting       Expires              Service principal
06/22/2017 08:14:21  06/22/2017 18:14:21  krbtgt/AMBARI.APACHE.ORG@AMBARI.APACHE.ORG
        renew until 06/29/2017 08:14:18
$ kdestroy    (退出，如果再执行klist就会又显示No credentials cache found)
```

### 未启用kerberos的HDFS
未启用kerberos时HDFS是没有认证和权限控制的：
```
$ curl http://u1401.ambari.apache.org:50070/webhdfs/v1/user?op=LISTSTATUS
{"FileStatuses":{"FileStatus":[
{"accessTime":0,"blockSize":0,"childrenNum":6,"fileId":16388,"group":"hdfs","length":0,"modificationTime":1493881416749,"owner":"ambari-qa","pathSuffix":"ambari-qa","permission":"770","replication":0,"storagePolicy":0,"type":"DIRECTORY"},
{"accessTime":0,"blockSize":0,"childrenNum":0,"fileId":16444,"group":"hdfs","length":0,"modificationTime":1493278869398,"owner":"hcat","pathSuffix":"hcat","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},
{"accessTime":0,"blockSize":0,"childrenNum":0,"fileId":16455,"group":"hdfs","length":0,"modificationTime":1493278917798,"owner":"hive","pathSuffix":"hive","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"}
]}}
```
而通过浏览器访问：```http://u1401.ambari.apache.org:50070/explorer.html#/user```  
发现浏览器也是调用```/webhdfs/v1/user?op=LISTSTATUS```这个API。  
使用命令行访问HDFS:
```
$ hdfs dfs -ls /user
Found 3 items
drwxrwx---   - ambari-qa hdfs          0 2017-05-04 07:03 /user/ambari-qa
drwxr-xr-x   - hcat      hdfs          0 2017-04-27 07:41 /user/hcat
drwxr-xr-x   - hive      hdfs          0 2017-04-27 07:41 /user/hive
```
### 启用了kerberos的HDFS
```
$ curl http://u1401.ambari.apache.org:50070/webhdfs/v1/user?op=LISTSTATUS
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1"/>
<title>Error 401 Authentication required</title>
</head>
<body><h2>HTTP ERROR 401</h2>
<p>Problem accessing /webhdfs/v1/user. Reason:
<pre>    Authentication required</pre></p><hr /><i><small>Powered by Jetty://</small></i><br/>   
```
提示“需要认证”了。   
使用命令行访问启用了kerberos的HDFS:
```
$ hdfs dfs -ls /user
17/05/04 08:47:03 WARN ipc.Client: Exception encountered while connecting to the server :
...  (省略)
ls: Failed on local exception: java.io.IOException: javax.security.sasl.SaslException: GSS initiate failed [Caused by GSSException: No valid credentials provided (Mechanism level: Failed to find any Kerberos tgt)]; Host Details : local host is: "u1401.ambari.apache.org/192.168.14.101"; destination host is: "u1401.ambari.apache.org":8020;
```
提示没有提供有效的凭据。  

### kerberos的HDFS授权

在KDC上创建两个测试用的用户webb和webb2：
```
$ kadmin.local -q "addprinc webb"                          (添加主体webb)
Enter password for principal "webb@AMBARI.APACHE.ORG":     (输入两次密码)
$ kadmin.local -q "addprinc webb2"                          (添加主体webb2)
Enter password for principal "webb@AMBARI.APACHE.ORG":     (输入两次密码)
```
下面分别用webb和webb2登录，并创建目录：
```
$ kinit webb                                   (登录使用kerberos客户端的kinit命令)
Password for webb3@AMBARI.APACHE.ORG:          (输入主体webb的密码)
$ klist                                        (klist显示当前登录的用户是webb)
Default principal: webb@AMBARI.APACHE.ORG
Valid starting       Expires              Service principal
05/05/2017 00:43:28  05/05/2017 10:43:28  krbtgt/AMBARI.APACHE.ORG@AMBARI.APACHE.ORG
        renew until 05/12/2017 00:43:25
$ hdfs dfs -mkdir /tmp/webb                    (用webb用户在HDFS上创建/tmp/webb目录)
$ kinit webb2                                  (切换为webb2登录，会提示输入密码)
$ hdfs dfs -mkdir /tmp/webb2                   (用webb2用户在HDFS上创建/tmp/webb2目录)
$ hdfs dfs -ls /tmp                            (显示HDFS中/tmp目录下的内容)
Found 17 items
...(省略)
drwxr-xr-x   - webb      hdfs          0 2017-05-04 11:33 /tmp/webb
drwxr-xr-x   - webb2     hdfs          0 2017-05-04 11:36 /tmp/webb2
```
通过上面的文件列表可以看到新创建的两个目录的拥有者(owner)分别是webb和webb2。  
HDFS采用与POSIX兼容的文件系统通用的授权方案。权限由三个不同类别的用户管理：拥有者，组和其他人。读取，写入和执行权限可以独立授予每个类。  
（当前用户是webb2）修改一下/tmp/webb2的文件权限：
```
$ hdfs dfs -chmod 700 /tmp/webb2          (只有文件拥有者可以改权限)
$ hdfs dfs -ls /tmp
Found 17 items
...(省略)
drwxr-xr-x   - webb      hdfs          0 2017-05-04 11:33 /tmp/webb
drwx------   - webb2     hdfs          0 2017-05-04 11:36 /tmp/webb2
```
当修改成700权限后，只有文件拥有者可以修改和查看，其他用户都没有了权限。可以切换成webb用户测试一下：
```
$ kinit webb                        (输入密码)
$ hdfs dfs -ls /tmp/webb2
...(略)
ls: Permission denied: user=webb, access=READ_EXECUTE, inode="/tmp/webb2":webb2:hdfs:drwx------
```
如果每个租户的HDFS目录权限都默认设定为700，则只有租户自己可以查看和存取目录下的文件。  

## kerberos命令备忘
#### 主体
在KDC中新增主体：
```
$ kadmin.local
kadmin.local:  addprinc webb
WARNING: no policy specified for webb@AMBARI.APACHE.ORG; defaulting to no policy
Enter password for principal "webb@AMBARI.APACHE.ORG":
Re-enter password for principal "webb@AMBARI.APACHE.ORG":
Principal "webb@AMBARI.APACHE.ORG" created.
```
或者：
```
$ kadmin.local -q "addprinc hue/u1401.ambari.apache.org@AMBARI.APACHE.ORG"
```
#### keytab
为主体webb@AMBARI.APACHE.ORG生成keytab文件，文件名是webb.keytab（当前目录下生成）
```
$ kadmin.local
kadmin.local:  ktadd -k webb.keytab webb@AMBARI.APACHE.ORG
Entry for principal webb@AMBARI.APACHE.ORG with kvno 3, encryption type aes256-cts-hmac-sha1-96 added to keytab WRFILE:webb.keytab.
Entry for principal webb@AMBARI.APACHE.ORG with kvno 3, encryption type arcfour-hmac added to keytab WRFILE:webb.keytab.
Entry for principal webb@AMBARI.APACHE.ORG with kvno 3, encryption type des3-cbc-sha1 added to keytab WRFILE:webb.keytab.
Entry for principal webb@AMBARI.APACHE.ORG with kvno 3, encryption type des-cbc-crc added to keytab WRFILE:webb.keytab.
kadmin.local:  exit
```
使用keytab文件登录（而不是密码）：
```
$ kinit -k -t webb.keytab webb
```
显示keytab文件中主体和加密类型：
```
$ klist -e -k webb.keytab 
Keytab name: FILE:webb.keytab
KVNO Principal
---- --------------------------------------------------------------------------
   2 webb@AMBARI.APACHE.ORG (aes256-cts-hmac-sha1-96)
   2 webb@AMBARI.APACHE.ORG (arcfour-hmac)
   2 webb@AMBARI.APACHE.ORG (des3-cbc-sha1)
   2 webb@AMBARI.APACHE.ORG (des-cbc-crc)
   3 webb@AMBARI.APACHE.ORG (aes256-cts-hmac-sha1-96)
   3 webb@AMBARI.APACHE.ORG (arcfour-hmac)
   3 webb@AMBARI.APACHE.ORG (des3-cbc-sha1)
   3 webb@AMBARI.APACHE.ORG (des-cbc-crc)
```
以上是安装了启用了kerberos的AMBARI下的keytab。而仅安装kerberos后测试的结果是：
```
$  klist -e -k webb.keytab
Keytab name: FILE:webb.keytab
KVNO Principal
---- --------------------------------------------------------------------------
   2 webb@AMBARI.APACHE.ORG (aes256-cts-hmac-sha1-96)
   2 webb@AMBARI.APACHE.ORG (arcfour-hmac)
   2 webb@AMBARI.APACHE.ORG (des3-cbc-sha1)
   2 webb@AMBARI.APACHE.ORG (des-cbc-crc)
```

### Kerberos认证测试
用kinit登录kerberos，然后使用CURL访问REST API，需要在CURL命令行上增加```--negotiate -u : ```参数。例如：
```
$ klist -e -k /etc/security/keytabs/hdfs.headless.keytab
Keytab name: FILE:/etc/security/keytabs/hdfs.headless.keytab
KVNO Principal
---- --------------------------------------------------------------------------
   1 hdfs-hdp1@AMBARI.APACHE.ORG (aes256-cts-hmac-sha1-96)
   1 hdfs-hdp1@AMBARI.APACHE.ORG (des-cbc-md5)
   1 hdfs-hdp1@AMBARI.APACHE.ORG (aes128-cts-hmac-sha1-96)
   1 hdfs-hdp1@AMBARI.APACHE.ORG (des3-cbc-sha1)
   1 hdfs-hdp1@AMBARI.APACHE.ORG (arcfour-hmac)
$ kinit -kt /etc/security/keytabs/hdfs.headless.keytab hdfs-hdp1@AMBARI.APACHE.ORG
$ curl -k -i --negotiate -u : http://u1401.ambari.apache.org:50070/webhdfs/v1/tmp?op=LISTSTATUS
```

### kerberos代理测试
[参考](https://pypi.python.org/pypi/kdcproxy/0.3.1)  
freeipa带有kerberos KDC代理功能。执行ipa-server-install的时候有提示：
```
Configuring the web interface (httpd). Estimated time: 1 minute
  [16/21]: create KDC proxy user
  [17/21]: create KDC proxy config
  [18/21]: enable KDC proxy
```
通过KdcProxy可以让kinit客户端以https协议通过代理连接到KDC。  
首先需要把freeipa的证书复制到客户端所在的机器上：
```
$ cd /etc/ipa
$ openssl X509 -in ca.crt -out ca.pem
$ scp ca.pem root@c7002:/etc/ssl/.    (复制到c7002节点)
```
配置c7002节点的kerberos客户端配置文件(/etc/krb5.conf)：
```
 [libdefaults]
   default_realm = AMBARI.APACHE.ORG

[realms]
   AMBARI.APACHE.ORG = {
     http_anchors = FILE:/etc/ssl/ca.pem
     kdc = https://c7004.ambari.apache.org/KdcProxy
     kpasswd_server = https://c7004.mbari.apache.org/KdcProxy
```
在c7002节点上测试KDC代理。环境变量KRBT_TRACE可以让kerberos调试信息输出到文件(/dev/stdout表示输出到屏幕)：
```
$ env KRB5_TRACE=/dev/stdout kinit admin
[29875] 1497331322.495260: Getting initial credentials for admin@AMBARI.APACHE.ORG
[29875] 1497331322.495434: Sending request (196 bytes) to AMBARI.APACHE.ORG
[29875] 1497331322.495463: Resolving hostname c7004.ambari.apache.org
[29875] 1497331322.619668: TLS certificate name matched "c7004.ambari.apache.org"
[29875] 1497331322.623893: Sending HTTPS request to https 192.168.70.104:443
[29875] 1497331322.629792: Received answer (272 bytes) from https 192.168.70.104:443
[29875] 1497331322.629809: Terminating TCP connection to https 192.168.70.104:443
[29875] 1497331322.630005: Response was not from master KDC
[29875] 1497331322.630206: Received error from KDC: -1765328359/Additional pre-authentication required
[29875] 1497331322.630269: Processing preauth types: 136, 19, 2, 133
[29875] 1497331322.630288: Selected etype info: etype aes256-cts, salt "Ar>LhD/*\>smo3/3", params ""
[29875] 1497331322.630296: Received cookie: MIT
Password for admin@AMBARI.APACHE.ORG:
（略）
```
从上面的调试信息可以看出kinit连接是通过https的443端口连接到KDC的。  
## windows下kerberos认证(非AD)
[原文](https://community.hortonworks.com/articles/28537/user-authentication-from-windows-workstation-to-hd.html)  
在windows10专业版下进行的测试，win10已经加入了AD域。  
- windows域是HOME.LANGCHAO.COM
- HDP集群域是AMBARI.APACHE.ORG  

测试步骤：  

1. 安装MIT Kerberos客户端(windows版)  
2. 安装windows版火狐浏览器  
3. 在火狐中启用Kerberos支持  
4. 通过Kerberos客户端获得kerberos票据  
5. 通过火狐打开HDP界面  

#### 1.安装MIT Kerberos客户端(windows版)
64位版下载地址：[http://web.mit.edu/kerberos/dist/kfw/4.0/kfw-4.0.1-amd64.msi](http://web.mit.edu/kerberos/dist/kfw/4.0/kfw-4.0.1-amd64.msi)  
安装的默认目录是```C:\Program Files\MIT\Kerberos```。  
将HDP的/etc/krb5.conf复制到上述目录下，改名为krb5.ini。  
在windows下添加两个环境变量：  
```
KRB5_CONFIG=C:\Program Files\MIT\Kerberos\krb5.ini
KRB5CCNAME=c:\temp\krb5cache
```
上面的c:\temp目录必须有写权限。  
![](https://community.hortonworks.com/storage/attachments/3561-1.png)  
增加完环境变量后重启windows。  

#### 2.安装windows版火狐浏览器  
火狐浏览器下载地址：  
[https://www.mozilla.org/en-US/firefox/new/](https://www.mozilla.org/en-US/firefox/new)  

#### 3.在火狐中启用Kerberos支持
在火狐中地址栏输入```about:config```并回车，然后搜索和设置下面的参数：
```
network.negotiate-auth.trusted-uris = .ambari.apache.org
network.negotiate-auth.using-native-gsslib = false
network.negotiate-auth.gsslib = C:\Program Files\MIT\Kerberos\bin\gssapi32.dll
network.auth.use-sspi = false
network.negotiate-auth.allow-non-fqdn = true
```
#### 4.通过Kerberos客户端获得kerberos票据  
安装完kerberos客户端后windows桌面上会创建叫“MIT Kerberos Ticket Manager”的快捷方式。双击快捷方式进入kerberos客户端。由于windows已经加入了域，会显示：
```
> wbwang@HOME.LANGCHAO.COM                   6月 16  20.14（9 h, 13 m remaining)
```  
点击“Get Ticket”按钮，在弹出窗口中输入kerberos主体和密码，主体如```webb@AMBARI.APACHE.ORG```。  
（实测中曾经报错kerberos 5:clock skew too great，在KDC主机上安装NTP服务后解决，时间调整后ambari-server也要重启）  
当成功获取票据后，kerberos客户端会显示两个票据：
```
> webb@AMBARI.APACHE.ORG                     6月 16 20:37 (9 h, 31 m remaining)
> wbwang@HOME.LANGCHAO.COM                   6月 16 20:14（9 h, 13 m remaining)
```
#### 5.通过火狐打开HDP界面
打开火狐，输入Ambari地址，如：```http://u1401.ambari.apache.org:8080```，逐次点击HDFS ->  Quick Links -> NameNode UI -> Utilities -> Browse the file system 或者直接打开```http://u1401.ambari.apache.org:50070/explorer.html```，就可以看到HDFS中文件清单了。  

## 跨域信任
AMBARI.APACHE.ORG领域管辖c7301、c7302、c703三个节点，KDC部署在c7301。在c7304上部署了一个新的KDC，领域(REALM)是APACHE.ORG。两个KDC的krb5.conf配置文件都包含下列内容：
```
[realms]
  APACHE.ORG = {
    admin_server = c7304.ambari.apache.org
    kdc = c7304.ambari.apache.org
  }
  AMBARI.APACHE.ORG = {
    admin_server = c7301.ambari.apache.org
    kdc = c7301.ambari.apache.org
  }

[domain_realm]
        c7301.ambari.apache.org = AMBARI.APACHE.ORG
        c7302.ambari.apache.org = AMBARI.APACHE.ORG
        c7303.ambari.apache.org = AMBARI.APACHE.ORG
        c7304.ambari.apache.org = APACHE.ORG
```
c7301-c703部署的hadoop集群启用了kerberos。以http协议访问HDFS时，提示需要认证(negotiate)：
```
$ curl -i http://c7301.ambari.apache.org:50070/webhdfs/v1/user?op=LISTSTATUS
HTTP/1.1 401 Authentication required
(下略)
```
如果用kinit登录领域`AMBARI.APACHE.ORG`，然后再用同一个URL，就可以列出HDFS中`/user`目录下的内容，如：
```
$ kinit root/admin@AMBARI.APACHE.ORG
$ curl --negotiate -u : http://c7301.ambari.apache.org:50070/webhdfs/v1/user?op=LISTSTATUS
{"FileStatuses":{"FileStatus":[
(下略)
```
### 双向互信
下面建立两个领域`AMBARI.APACHE.ORG`域`APACHE.ORG`之间的跨域信任。之后，用kinit登录领域`APACHE.ORG`，再用curl访问上面的属于`AMBARI.APACHE.ORG`领域的链接。  

为了建立两个领域的互信，需要在两个领域的KDC中分别创建两个特殊的主体(principal)：  
```
krbtgt/APACHE.ORG@AMBAIR.APACHE.ORG
krbtgt/AMBARI.APACHE.ORG@APACHE.ORG
```
具体的操作如下：
```
$ vagrant ssh c7301
$ sudo su - root
$ kadmin.local -q "addprinc -pw 1 krbtgt/APACHE.ORG@AMBAIR.APACHE.ORG"
$ kadmin.local -q "addprinc -pw 1 krbtgt/AMBARI.APACHE.ORG@APACHE.ORG"
$ vagrant ssh c7304                 （执行这个命令前先退出VM c7301）
$ sudo su - root
$ kadmin.local -q "addprinc -pw 1 krbtgt/APACHE.ORG@AMBAIR.APACHE.ORG"
$ kadmin.local -q "addprinc -pw 1 krbtgt/AMBARI.APACHE.ORG@APACHE.ORG"
```
测试一下跨域互信。在c7304上执行：
```
$ kinit root/admin@APACHE.ORG
$ curl --negotiate -u : http://c7301.ambari.apache.org:50070/webhdfs/v1/user?op=LISTSTATUS
{"FileStatuses":{"FileStatus":[
(下略)
```
从上面可以看出，虽然kinit登录的是APACHE.ORG，却可以访问AMBARI.APACHE.ORG的资源。  
### 单向信任
下面试图建立一个单向信任关系：AMBARI.APACHE.ORG信任APACHE.ORG。  
先用`delprinc`将上文创建了4个`krbtgt`主体删除。然后在c7301、c7304上分别执行：
```
$ kadmin.local -q "addprinc -pw 1 krbtgt/AMBARI.APACHE.ORG@APACHE.ORG"
```
两个主体的密码要一样。在c7304上执行：
```
$ kinit root/admin@APACHE.ORG
$ curl --negotiate -u : http://c7301.ambari.apache.org:50070/webhdfs/v1/user?op=LISTSTATUS
{"FileStatuses":{"FileStatus":[
(下略)
```
可以看到，建立单向互信只需要建立两个主体。  