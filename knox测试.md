#### core-site.xml配置
knox是一种用户代理程序，集群授权给knox后，它才能代理用户请求。通过修改hadoop的核心配置文件core-site.xml中相关参数来授权给knox代理用户请求的权限。  
core-site.xml在文件系统中的位置是`/etc/hadoop/conf`，通过Ambari修改core-site.xml更方便。在Ambari中前往Service -> HDFS -> Configs -> Advanced -> Custom core-site，修改用`hadoop.proxyuser.knox.hosts`和`hadoop.proxyuser.knox.groups`参数：
```
<property>
    <name>hadoop.proxyuser.knox.groups</name>
    <value>users</value>
</property>
<property>
    <name>hadoop.proxyuser.knox.hosts</name>
    <value>FQDN_OF_KNOX_HOST</value>
</property>
```
在测试中FQDN_OF_KNOX_HOST被替换为```u1401.ambari.apache.org```，也就是安装knox网关的虚拟机。把```users```修改成了```*```，即允许knox代理（仿冒）所有用户组的用户。  

## 两种认证方式的测试
 - ShiroProvider  
对于LDAP/AD身份验证，使用用户名和密码。没有SPNEGO/Kerberos支持。([这篇](https://cwiki.apache.org/confluence/display/KNOX/Using+Apache+Knox+with+ActiveDirectory)apache官方文档对于knox与LDAP的集成讲得很到位)  
 - HadoopAuth  
对于SPNEGO/Kerberos身份验证，使用委派令牌。没有LDAP/AD支持。  

#### ShiroProvider(LDAP认证)
用ambari安装的knox，默认安装目录是```/usr/hdp/current/knox-server```。默认cluster-name是default，对应的拓扑配置文件是：
```
/usr/hdp/current/knox-server/conf/topologies/default.xml
```
有两种方式创建LDAP服务器，一是手工安装OpenLDAP；二是使用Knox自带的Demo LDAP：
- 如果要手工安装OpenLDAP，参考[这个](https://github.com/wbwangk/wbwangk.github.io/wiki/LDAP)LDAP测试的文档。  
- 如果要使用Knox自带的DemoLDAP服务器，则在Ambari中前往 Services -> Knox -> Service Actions -> Start Demo LDAP。  

下面的测试使用的u1401上手工部署OpenLDAP，并在LDAP上创建了一个测试用户john（dn: uid=john,ou=People,dc=ambari,dc=apache,dc=org），该用户的密码是johnldap。初学者可以安装一个[JXplorer](http://www.jxplorer.org)来链接LDAP服务器查看其中的数据。    

为了修改Knox的默认集群拓扑文件(default.xml)，前往 Services -> Knox -> Configs -> Advanced topology，将第一个xml的第一个provider元素替换为下列内容：
```
<topology>
            <gateway>
                <provider>
                    <role>authentication</role>
                    <name>ShiroProvider</name>
                    <enabled>true</enabled>
                    <param>
                        <name>sessionTimeout</name>
                        <value>30</value>
                    </param>
                    <param>
                        <name>main.ldapRealm</name>
                        <value>org.apache.hadoop.gateway.shirorealm.KnoxLdapRealm</value>
                    </param>
                    <param>
                        <name>main.ldapRealm.userDnTemplate</name>
                        <value>uid={0},ou=people,dc=ambari,dc=apache,dc=org</value>
                    </param>
                    <param>
                        <name>main.ldapRealm.contextFactory.url</name>
                        <value>ldap://{{knox_host_name}}</value>
                    </param>
                    <param>
                        <name>main.ldapRealm.contextFactory.authenticationMechanism</name>
                        <value>simple</value>
                    </param>
                    <param>
                        <name>urls./**</name>
                        <value>authcBasic</value>
                    </param>
                </provider>
(省略一些内容)
        </topology>
```
相对于默认配置，只修改了两个参数，一个是将```main.ldapRealm.userDnTemplate```的值定义为```uid={0},ou=people,dc=ambari,dc=apache,dc=org```；另一个是将```main.ldapRealm.contextFactory.url```定义为```ldap://{{knox_host_name}}```。这因为没有启用TLS，所以没有定义为```ldapi://{{knox_host_name}}:33389```。

在ambari界面中点击Save按钮保存，并通过橙黄色按钮重启相关服务。之后会发现```/usr/hdp/current/knox-server/conf/topologies/```目录下的default.xml修改更新了。  

测试使用OpenLDAP中的用户john（dn: uid=john,ou=People,dc=ambari,dc=apache,dc=org），该用户的密码是johnldap。使用该用户测试Knox：
```
$ curl -i -k -u john:johnldap -X GET \
    'https://u1401.ambari.apache.org:8443/gateway/default/webhdfs/v1/tmp/webb?op=LISTSTATUS'
{"FileStatuses":{"FileStatus":[{"accessTime":1493947036270,"blockSize":134217728,"childrenNum":0,"fileId":22980,"group":"hdfs","length":3,"modificationTime":1493947036592,"owner":"webb","pathSuffix":"t1.txt","permission":"644","replication":3,"storagePolicy":0,"type":"FILE"}]}}
```
可以故意输入错误的密码，如john:1到-u参数，则：
```
$ curl -i -k -u john:johnldap -X GET \
    'https://localhost:8443/gateway/default/webhdfs/v1/tmp/webb?op=LISTSTATUS'
HTTP/1.1 401 Unauthorized
```
```/tmp/webb```是HDFS中的目录，也可以获取很目录的内容：
```
$ curl -i -k -u john:johnldap -X GET \
    'https://localhost:8443/gateway/default/webhdfs/v1/?op=LISTSTATUS'
(返回值略)
```
curl -k -i --negotiate -u : https://localhost:8443/gateway/default/webhdfs/v1/tmp?op=LISTSTATUS

#### Hadoop Auth Provider(SPNEGO/Kerberos认证)
本节参考了[这个文章](https://community.hortonworks.com/articles/85550/hadoop-auth-spnego-and-delegation-token-based-auth.html)。  
本节介绍使用Apache Knox配置Hadoop Auth（SPNEGO和基于代理令牌的身份验证）。
[Hadoop Auth](https://hadoop.apache.org/docs/stable/hadoop-auth/index.html)是一个Java库，可以为HTTP请求启用Kerberos SPNEGO身份验证。它在对受保护的资源进行身份验证后，成功认证后，Hadoop Auth创建一个带有认证令牌，用户名，用户主体，认证类型和到期时间的签名HTTP Cookie。该cookie用于所有后续的HTTP客户端请求，以访问受保护的资源，直到cookie过期。

鉴于Apache Knox的可插拔身份验证提供程序，只需少量配置更改即可轻松地使用Apache Knox设置Hadoop Auth。  
假设已经有一个正常工作的Hadoop集群与Apache Knox，而且集群是Kerberized。  

#### 配置
要在Apache Knox中使用Hadoop Auth，我们需要更新Knox拓扑。Hadoop Auth配置为提供程序，因此我们需要通过提供程序参数进行配置。Apache Knox使用与Apache Hadoop相同的配置参数，可以预期其类似的行为。要使用Ambari更新Knox拓扑，请转到Knox - > Configs - > Advanced拓扑结构。  
以下是Apache Knox拓扑文件中HadoopAuth提供程序代码段的示例（如果通过ambari修改knox的配置文件，在配置文件的Advanced atopology小节中）：
```
<provider>
  <role>authentication</role>
  <name>HadoopAuth</name>
  <enabled>true</enabled>
  <param>
    <name>config.prefix</name>
    <value>hadoop.auth.config</value>
  </param>
  <param>
    <name>hadoop.auth.config.signature.secret</name>
    <value>knox-signature-secret</value>
  </param>
  <param>
    <name>hadoop.auth.config.type</name>
    <value>kerberos</value>
  </param>
  <param>
    <name>hadoop.auth.config.simple.anonymous.allowed</name>
    <value>false</value>
  </param>
  <param>
    <name>hadoop.auth.config.token.validity</name>
    <value>1800</value>
  </param>
  <param>
    <name>hadoop.auth.config.cookie.domain</name>
    <value>ambari.apache.org</value>
  </param>
  <param>
    <name>hadoop.auth.config.cookie.path</name>
    <value>gateway/default</value>
  </param>
  <param>
    <name>hadoop.auth.config.kerberos.principal</name>
    <value>HTTP/u1401.ambari.apache.org@AMBARI.APACHE.ORG</value>
  </param>
  <param>
    <name>hadoop.auth.config.kerberos.keytab</name>
    <value>/etc/security/keytabs/spnego.service.keytab</value>
  </param>
  <param>
    <name>hadoop.auth.config.kerberos.name.rules</name>
    <value>DEFAULT</value>
  </param>
</provider>
```
（需要额外说明的是config.prefix参数(默认是hadoop.auth.config)，它指定了多个参数的前缀）  
以下是需要最少更新的参数：
 1. hadoop.auth.config.signature.secret - 这是用于在hadoop.auth cookie中签署委托令牌的秘密。需要在给定群集中的Knox网关的所有实例中使用相同的秘密。否则，委托令牌将失败验证，每个请求将重复身份验证。
 2. cookie.domain - 用于存储身份验证令牌的HTTP cookie的域（例如mycompany.com）
 3. hadoop.auth.config.kerberos.principal - Web应用程序Kerberos主体名称。Kerberos主体名称必须以HTTP / ...开头。
 4. hadoop.auth.config.kerberos.keytab - 包含上面指定的kerberos主体的凭据的keytab文件的路径。

如果您使用Ambari，您将不得不重新启动Knox，这是一个Ambari要求，如果拓扑在Ambari之外更新，则不需要重新启动（Apache Knox每次更新拓扑时间戳时重新加载拓扑）。
如果通过ambari更新参数，需要在ambari界面中点击Save按钮保存，并通过橙黄色按钮重启相关服务。之后会发现```/usr/hdp/current/knox-server/conf/topologies/```目录下的default.xml修改更新了。 

#### 测试

 1. 让我们创建一个用户'guest'与组'用户'。请注意，由于属性“hadoop.proxyuser.knox.groups = users”选择了组用户。
```
$ useradd guest -u 1590 -g users
```
 2. 用'kadmin.local'命令增加主体：
```
$ kadmin.local -q "addprinc guest/u1401.ambari.apache.org”
```
 3. 用kinit登录：
```
$ kinit guest/u1401.ambari.apache.org@AMBARI.APACHE.ORG
```
用kinit登录后，kerberos会话就可以跨客户端请求使用，如curl。以下curl命令可用于从HDFS请求目录列表，同时通过-negotiate标志与SPNEGO进行身份验证。首先是不通过knox网关，直接访问启用了kerberos的HDFS(取HDFS中/tmp目录下对象清单)：
```
$ curl -k -i --negotiate -u : http://u1401.ambari.apache.org:50070/webhdfs/v1/tmp?op=LISTSTATUS
```
通过knox网关访问HDFS:
```
$ curl -k -i --negotiate -u : https://u1401.ambari.apache.org:8443/gateway/default/webhdfs/v1/tmp?op=LISTSTATUS
HTTP/1.1 401 Authentication required
Date: Fri, 24 Feb 2017 14:19:25 GMT
WWW-Authenticate: Negotiate
Set-Cookie: hadoop.auth=; Path=gateway/default; Domain=ambari.apache.org; Secure; HttpOnly
Content-Type: text/html; charset=ISO-8859-1
Cache-Control: must-revalidate,no-cache,no-store
Content-Length: 320
Server: Jetty(9.2.15.v20160210)
 
HTTP/1.1 200 OK
Date: Fri, 24 Feb 2017 14:19:25 GMT
WWW-Authenticate: Negotiate YGwGCSqGSIb3EgECAgIAb10wW6ADAgEFoQMCAQ+iTzBNoAMCARCiRgRE26GeVRA0WkP7eb3csszuxUnSBDFK0NWH2+ai5pFY1onksiVOqjLkY8YS1xF5CshT4IwfrOHz6ivG6218X6oOSb0oCaU=
Set-Cookie: hadoop.auth="u=guest&p=guest/u1401.ambari.apache.org@AMBARI.APACHE.ORG&t=kerberos&e=1487947765114&s=fNpq9FYy2DA19Rah7586rgsAieI="; Path=gateway/default; Domain=ambari.apache.org; Secure; HttpOnly
Cache-Control: no-cache
Expires: Fri, 24 Feb 2017 14:19:25 GMT
Date: Fri, 24 Feb 2017 14:19:25 GMT
Pragma: no-cache
Expires: Fri, 24 Feb 2017 14:19:25 GMT
Date: Fri, 24 Feb 2017 14:19:25 GMT
Pragma: no-cache
Content-Type: application/json; charset=UTF-8
X-FRAME-OPTIONS: SAMEORIGIN
Server: Jetty(6.1.26.hwx)
Content-Length: 276
 
{"FileStatuses":{"FileStatus":[{"accessTime":0,"blockSize":0,"childrenNum":1,"fileId":16398,"group":"hdfs","length":0,"modificationTime":1487855904191,"owner":"hdfs","pathSuffix":"entity-file-history","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"}]}}
```
服务器首先返回401(Authentication required)，并在响应头上放置了```WWW-Authenticate: Negotiate```标志。这个Negotiate表示要挑战浏览器（收到这个挑战后，在windows的chrome表现为弹出让输入用户名口令的窗口）。而在linux下的curl的则调用了自己的GSS-Negotiate功能来计算出一个请求来再次调用同一个URL，如果在curl中加入--verbose参数可以看到请求增加的内容：
```
> GET /webhdfs/v1/tmp?op=LISTSTATUS HTTP/1.1
> Authorization: Negotiate YIIC4QYJKoZIhvcSAQICAQBuggLQMIICzKADAgEFoQMCAQ6iBwMFACAAAACjggGcYYIBmDCCAZSgAwIBBaETGxFBTUJBUkkuQVBBQ0hFLk9SR6IqMCigAwIBA6EhMB8bBEhUVFAbF3UxNDAxLmFtYmFyaS5hcGFjaGUub3Jno4IBSjCCAUagAwIBEKEDAgEBooIBOASCATQsvtJkb821p1278/N+uJkQLdSHxwFjikXItktYEUizskJu5l4BkQhs/SPF0Zq0QrSxzMMB9x7WO90w1edyM8lv5oaMNPs7nzTOIYDW52K47NdIY/TwDScNlFubWAg/Aq5b9wxrLrJ7r+G7J9DheYLUTglgztIV0jqFlWFgxr6ZGtkGHx4QdMDeGQDmcdeGEQWlNYbrkO1D5iMCLWySZe3ijBZ77DU22F5ukS6BiCAVbAVouRPCTe22ey19kknygqvsc8TcVS7/5kKcNL1CbZsiJUxwaEeDRNjEFCmRp4and0hO3l2iAc0P+hsCuTBz2oEtQaUV4tCgJ78V7sFRHcT+un/nfLd7P246odeOFOR/e4KOBMH56+WpyabbbH4cZJ3wFcB4dldkj0BKZftIG5pRV2v2g6SCARUwggERoAMCARCiggEIBIIBBFbDBLfvzjDA97j0RqFkxWG7GP97uDOSO3sJ5sgdajePDG/nvP+TBMelraUiJFPe/MZX9aInDyrod4RTTZ4oqoYyyrYte2wuczSuuzcJd1tpzDkSq64EtC9ZU/Ir9Ix0OrFtgmL+Cq4YbrB3FRtzq59xRaiPzDAnwAwG57f6lrBOKpwCLflOiNj7cKiDuDhczSM3p+G0ZE1o9VkDhP5RkXBD8+vSrAZtqM6HrmJ+BukmpNvPMDXGN9eijgfMDC2g1IY2afViC8a2ZiuLVvKnBc8yn+bkwXxejJXMEzz9t0z/qWCF8RPrsRnRHsT87wRuqXrdzT8Awsb6R0qHJhmbfduRyd3C
> User-Agent: curl/7.35.0
```

上述请求还可以这样发送：
```
$ curl -c c.txt -k -i --negotiate -u : "https://u1401.ambari.apache.org:8443/gateway/default/webhdfs/v1/tmp?op=LISTSTATUS" 
$  curl -b c.txt -k -i -H 'WWW-Authenticate: Negotiate YGwGCSqGSIb3EgECAgIAb10wW6ADAgEFoQMCAQ+iTzBNoAMCARCiRgREyTPm8S3VhWLJgckAfCtBtaF4ppYYN+LXFDVA4bu9q/zQo1MXAo2A2OaIoaOKVNil3NGXh/oIYcalWb6baxsSeF8giT8=' "https://u1401.ambari.apache.org:8443/gateway/default/webhdfs/v1/tmp?op=LISTSTATUS"
```
```curl -c c.txt```参数表示把http响应中的Set-Cookie字段写入了本地的c.txt文件。```curl -b c.txt```表示带着c.txt文件中的cookie内容发送请求，并利用```-H```参数设置了令牌。  

#### windows下chrome测试
现在用浏览器(我用的windows下chrome)访问地址```https://u1401.ambari.apache.org:8443/gateway/default/webhdfs/v1/tmp?op=LISTSTATUS```,会弹出类似基础认证的窗口让输入用户名口令，但无论输入啥都报告401或403错误:
```
HTTP ERROR 403
Problem accessing /index.html. Reason: 
    GSSException: Defective token detected (Mechanism level: GSSHeader did not find the right tag)
```
利用chrome插件EditThisCookie，在u1401.ambari.apache.org域中写入之前用curl返回的cookie：
```
hadoop.auth="u=guest&p=guest/u1401.ambari.apache.org@AMBARI.APACHE.ORG&t=kerberos&e=1487947765114&s=fNpq9FYy2DA19Rah7586rgsAieI="
```
然后浏览器就能显示正确的HDFS下文件清单了。这说明服务器端的配置正确，但浏览器端无法像kinit一样建立kerberos上下文。  
windows下可以安装kerberos客户端，但目前还没有找到在windows下利用kinit登录到u1401.ambari.apache.org的办法。可能与windows自带的kerberos功能冲突了。  

### linux桌面下SPNEGO测试(firefox)
[参考](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/5/html/Deployment_Guide/sso-config-firefox.html)  
利用vagrant安装了ubuntu16桌面系统，在Vagrantfile中配置的box：
```
u1408.vm.box = "box-cutter/ubuntu1404-desktop"
```
利用virtaulbox的菜单"设备->安装增强功能"调大了ubuntu桌面的分辨率（要先为VM添加光驱）。  
在ubuntu桌面上组合按键"ctrl+alt+f2"进入命令行状态，按"ctrl+alt+f7"返回图形界面。在命令行状态下安装kerberos客户端并登录：
```
$ apt install krb5-user
```
需要注意的是ubuntu桌面系统的当前用户是vagrant而不是root。执行kinit也必须在vagrant用户下，否则图形界面中的firefox将无法与kinit共享会话。按"ctrl+alt+f2"，以vagrant用户登录(密码也是vagrant)。
```
$ kinit guest/u1401.ambari.apache.org@AMBARI.APACHE.ORG
```
#### 启用firefox的negotiate认证
火狐默认不启用negotiate认证。需要在火狐中手工配置启用negotiate的URI。  
按"ctrl+alt+f7"返回桌面，点击桌面左侧的火狐图标，在火狐地址栏输入：```about:config```并回车。然后在search输入框中输入"negotiate"，双击配置项```network.negotiate-auth.trusted-uris```，输入```.ambari.apache.org```(别忽略最前面的点)。  

#### 测试firefox的negotiate认证
在火狐浏览器地址栏中输入：
```
https://u1401.ambari.apache.org:8443/gateway/default/webhdfs/v1/tmp?op=LISTSTATUS
```
火狐可能会提示"你的连接是不安全的"，点"高级"按钮将这个地址添加到例外。然后就可以看到浏览器以json格式显示了HDFS中```/tmp```目录下的文件清单。  
这说明firefox与kerberos客户端共享了会话。在火狐中按"shift+ctrl+i"进入调试模式，刷新URL请求，可以看到cookie的内容：
```
hadoop.auth="u=guest&p=guest/u1401.ambari.apache.org@AMBARI.APACHE.ORG&t=kerberos&e=1487947765114&s=fNpq9FYy2DA19Rah7586rgsAieI="
```
由于cookie的存在，浏览器不会每次请求都与kerberos客户端交互，只有cookie失效后，浏览器才会与kerberos客户交互，重新认证。

## SPNEGO原理浅析
SPNEGO是基于GSSAPI的，SPENGO认证的发起方是浏览器(或类似客户端)，通信协议是HTTP。SPNEGO的由服务器端发起，会在http响应头上放置：
```WWW-Authenticate:Negotiate```，浏览器收到这个响应后，会试图通过GSSAPI调用本地的kerberos客户端。如果本地存在kerberos票据缓存，kerberos客户端会将kerberos票据返回给GSSAPI调用者（猜的，反正是类似的东西）。浏览器收到票据，就构造出下一个http请求发往服务器，http请求头的格式是:``` Authorization: Negotiate YIIC4QYJKoZIhvcSAQ(后略)```。Authorization中应包含了kerberos票据。  
如果本地没有kerberos票据缓存或服务器不认可票据，windows的IE/Chrome会弹出内置窗口让用户输入用户名口令（与基础认证类似）。

## Ambari单点登录到Knox
[原文](https://cwiki.apache.org/confluence/display/KNOX/Ambari+via+KnoxSSO+and+Default+IDP)，也可参考[HDP的有关文档](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.0/bk_security/content/setting_up_knox_sso_for_ambari.html)   
Knox提供了基于表单的认证(默认IDP)。利用它，可以实现Ambari与Knox的单点登录。  
knox提供的登录页面URL：
```
https://u1401.ambari.apache.org:8443/gateway/knoxsso/knoxauth/login.html
```
登录页的样子：
![](https://cwiki.apache.org/confluence/download/attachments/62690515/Screen%20Shot%202016-04-03%20at%2011.56.21%20PM.png?version=1&modificationDate=1459742722000&api=v2)  
对应用来说，单点登录的端点是：
```
https://u1401.ambari.apache.org:8443/gateway/knoxsso/api/v1/websso
```
可以把上面的URL输入到浏览器中测试，发现当没有登录时，会被重定向到登录页面URL。由于登录页面URL是可以配置的，这带来了灵活性，比如可以自己开发个性化登录页。  
之前无论是测试Knox与LDAP的集成(ShiroProvider)还是SPNEGO/Kerberos认证(HadoopAuth)，都操作的Ambari中Knox配置的```Advanced topology```，而单点登录需要操作的是Ambari中Knox配置的```Advanced knoxsso-topology```。  
在配置中修改了如下内容：
```
<param>
  <name>knoxsso.redirect.whitelist.regex</name>
  <value>^https?:\/\/(u14\d\d\.ambari\.apache\.org|localhost|127\.0\.0\.1|0:0:0:0:0:0:0:1|::1):[0-9].*$</value>
</param>
```
这样是为了接受来自u1401-u1410等节点的重定向请求?  
还有就是确保参数``` knoxsso.cookie.secure.only```为false，这是因为ambari并没有启用https。  

#### 提取Knox的gateway-identity公钥
```
$ cd /usr/hdp/current/knox-server
$ keytool -exportcert -keystore data/security/keystores/gateway.jks -alias gateway-identity -rfc -file gateway.pem
Enter keystore password:{master secret}
Certificate stored in file <gateway.pem>
$ cat gateway.pem
–—BEGIN CERTIFICATE–— 
(公钥略)
–—END CERTIFICATE–—
```
生成了gateway.pem的文件，里面是公钥。过程中用到的[master secret](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.0/bk_Security_Guide/content/manage_master_secret.html)保存在文件```data/security/master```中（密文）。master secret好像是安装knox时定义的，也叫主密码。  
重启knox服务。然后设置ambari单点登录：
```
$ ambari-server setup-sso
Using python  /usr/bin/python
Setting up SSO authentication properties...
Do you want to configure SSO authentication [y/n] (y)?y
Provider URL [URL]: https://u1401.ambari.apache.org:8443/gateway/knoxsso/api/v1/websso
Public Certificate pem (empty) (empty line to finish input):
（输入或粘贴文件gateway.pem中的公钥，并回车两次）
Do you want to configure advanced properties [y/n] (n) ?n
Ambari Server 'setup-sso' completed successfully.
$ ambari-server restart
(要取消SSO，就再次运行ambar-server setup-sso并重启服务)
```
在浏览器中输入：```http://u1401.ambari.apache.org:8080```，用户会被重定向到knox的登录界面。在chrome中使用DevTools监测到的重定向请求（UrlEncode后）：
```
https://u1401.ambari.apache.org:8443/gateway/knoxsso/api/v1/websso?originalUrl=http://u1401.ambari.apache.org:8080/#/login?redirected=true
```
重定向的响应还是个302的重定向，以下是响应中Location值：
```
Location: https://u1401.ambari.apache.org:8443/gateway/knoxsso/knoxauth/login.html?originalUrl=http://u1401.ambari.apache.org:8080/#/login?redirected=true
```
上面的websso和login.html两个URL均在前文中提到。  
输入用户名```john```和密码```johnldap```后点登录，发出的请求：
```
https://u1401.ambari.apache.org:8443/gateway/knoxsso/api/v1/websso?originalUrl=http://u1401.ambari.apache.org:8080/
(请求头中的信息如下，Basic后面的乱码是john:johnldap的base64编码)
Authorization:Basic am9objpqb2hubGRhcA==
```
从上面的请求可以看出，KnoxSSO的表单认证(默认IDP)遵循了http基础认证规范。  
再看响应，Knox验证john密码通过，返回了307重定向，并在设置cookie：
```
Set-Cookie: JSESSIONID=1tgu9opzwl60m10c5ogw2yf5da; Path=/gateway/knoxsso
Set-Cookie: rememberMe=deleteMe; Path=/gateway/knoxsso
```
然后在根路径上写入了hadoop-jwt的cookie：
```
Set-Cookie: hadoop-jwt=eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJqb2huIiwiaXNzIjoiS05PWFNTTyIsImV4cCI6MTQ5Njg4MjI0NX0.(略);Path=/;Domain=.ambari.apache.org
```
这个JWT令牌base64解码后：
```
{"alg":"RS256"}.{"sub":"john","iss":"KNOXSSO","exp":1496882245}.(乱码)
```
这个JWT令牌有在线工具可以解析和验证，地址：```https://jwt.io/#debugger```。将前面的```eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJqb2huIiwiaXNzIjoiS05PWFNTTyIsImV4cCI6MTQ5Njg4MjI0NX0.(略)```复制到工具的Encoded输入框中，然后在VERIFY SIGNATURE框中粘贴前文提到的knox公钥（公钥包括–—BEGIN CERTIFICATE–—和END这两行，但在执行ambari-server setup-sso粘贴时却不能包含这两行），如果令牌验证通过，下面大按钮就显示“Signature Verified”。  

http响应的最后是指定了重定向的地址，这个地址是最初通过url参数originalUrl传递给Knox的：
```
Location: http://u1401.ambari.apache.org:8080/
```
小结：通过这个请求/响应，KnoxSSO验证了john的密码，为域ambari.apache.org的根路径写入了JWT令牌，并重定向回Ambari的入口页面。

可以用CURL模拟JWT的发放过程：
```
$  curl -c t.txt -ivk -u john:johnldap https://u1401.ambari.apache.org:8443/gateway/knoxsso/api/v1/websso?originalUrl=http://u1401.ambari.apache.org:8080/
(省略一些，-c t.txt参数表示将cookie内容写入t.txt文件中)
Set-Cookie: hadoop-jwt=eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJqb2huIiwiaXNzIjoiS05PWFNTTyIsImV4cCI6MTQ5NjkwMTE5M30.F9H2EMP-8uDx74AbNJTYlWoLRfm05uFRpvfheoP8fhmKSN_j4UZR1R9Updn0FZsG3OGBM8QsLO2u828yK6f-dPhg_0HntXxGjYE64lnX6_GoLOvBp1CugUGVGH5iOkgWUiKOW3xzxGI2oP3n3pSV_gdt5vKfq2RO8-fAT2r0Xg4;Path=/;Domain=.ambari.apache.org;HttpOnly
Location: http://u1401.ambari.apache.org:8080/
$ curl -b t.txt -iv http://u1401.ambari.apache.org:8080/
```
下面的curl请求模拟了ambari收到附带JWT的请求，```-b t.txt```参数表示把t.txt文件中的cookie信息一起发出。如果ambari配置正确则应直跳过登录页面，直接把用户重定向到ambari内部首页。  

## 测试Knox的用户模拟
需要先说明的是，hadoop测试集群的三个节点的操作系统均没有一个john的用户。Knox启用的是ShiroProvider认证方式。现在通过Knox的网关API在HDFS上创建一个```/tmp/john```的目录，注意这个目录的拥有者：
```
$ curl -k -i -u john:johnldap -X PUT "https://u1401.ambari.apache.org:8443/gateway/default/webhdfs/v1/tmp/john?op=MKDIRS"
$ curl  -k -u john:johnldap -X GET 'https://u1401.ambari.apache.org:8443/gateway/default/webhdfs/v1/tmp?op=LISTSTATUS'
（省略一些返回值）
      {
        "accessTime": 0,
        "blockSize": 0,
        "childrenNum": 0,
        "fileId": 44214,
        "group": "hdfs",
        "length": 0,
        "modificationTime": 1496819976113,
        "owner": "john",
        "pathSuffix": "john",
        "permission": "755",
        "replication": 0,
        "storagePolicy": 0,
        "type": "DIRECTORY"
      },
```
这个新建目录的拥有者是john。看上去简单。实际上这里面包含了“用户模拟”。knox服务使用自己的kerberos凭据访问HDFS，但成功模拟了最终用户john在HDFS上创建目录。  
关于用户模拟的原理可看考[hadoop安全](https://github.com/wbwangk/wbwangk.github.io/wiki/hadoop%E5%AE%89%E5%85%A8)。  

## 用户组映射
[关于用户组查找的apache官方文档](http://knox.apache.org/books/knox-0-13-0/user-guide.html#LDAP+Group+Lookup)  
为了更好地研究**用户组映射**功能，在新环境(centos7.3)中用ambari重新安装knox和openldap([1](https://imaidata.github.io/blog/ldap/))，然后用ShiroProvider将openldap配置为knox的用户存储(参照章节"[ShiroProvider(LDAP认证)](https://github.com/wbwangk/wbwangk.github.io/wiki/knox%E6%B5%8B%E8%AF%95#shiroproviderldap%E8%AE%A4%E8%AF%81)")。  
#### LDAP中增加用户、用户组
部署了LDAP工具ldapscripts，利用ldapscripts添加用户`webb`和用户组`wang`：
```
$ sudo ldapaddgroup wang                 (新增用户组wang)
$ sudo ldapadduser webb wang             (添加用户webb，隶属于用户组wang。webb条目多了gidNumber=5002属性)
$ sudo ldapsetpasswd webb                (为webb用户设置密码为"1")
$ sudo ldapaddusertogroup webb wang      (在用户组wang中添加用户webb。wang条目多了memberUid=webb属性)
```
#### 配置knox的用户模拟
在Ambair面板中通过HDFS-config的`Custom core-site`小节配置：
```
hadoop.proxyuser.knox.groups = wang
```
重启相关服务后测试knox对webhdfs的反向代理功能：
```
$ curl -k -u webb:1 'https://c7301.ambari.apache.org:8443/gateway/default/webhdfs/v1/tmp?op=LISTSTATUS'
message":"Failed to obtain user group information: org.apache.hadoop.security.authorize.AuthorizationException: User: knox is not allowed to impersonate webb"
```
提示不能获取用户webb的**用户组**信息。说明knox的默认LDAP设置不能正确找到用户组。

#### 不控制的用户模拟
将knox的用户模拟用户组设置为"*":
```
hadoop.proxyuser.knox.groups = *
```
这表示所有用户组的用户都可以被**模拟**。  
再执行上面curl，发现可以正常执行。说明当不控权限时，不需要根据用户查找用户组，所以knox对webhdfs的反向代理可以正常执行。

#### 在本地创建用户和用户组
在knox安装的节点手工创建用户和用户组：
```
$ sudo groupadd wang                     (增加用户组)
$ useradd webb -g wang                   (增加用户并指定用户组)
```
将knox的用户模拟用户组设置恢复为:
```
hadoop.proxyuser.knox.groups = wang
```
重新测试knox对webhdfs的反向代理功能，执行成功：
```
$ curl -k -u webb:1 'https://c7301.ambari.apache.org:8443/gateway/default/webhdfs/v1/tmp?op=LISTSTATUS'
```
说明knox利用了linux本地的“根据用户id查找用户组”功能。

## LDAP与knox用户组权限
首先，将[users2.ldif](https://github.com/wbwangk/wbwangk.github.io/wiki/LDAP#knox%E8%87%AA%E5%B8%A6sample)导入到LDAP中。  
找到knox的`GATEWAY_HOME`(如`/usr/hdp/2.6.1.0-129/knox`)，新建一个文件`<GATEWAY_HOME>/conf/topologies/sample5.xml`：
```
<topology>
    <gateway>
        <provider>
            <role>authentication</role>
            <name>ShiroProvider</name>
            <enabled>true</enabled>
            <param name="main.ldapRealm" value="org.apache.hadoop.gateway.shirorealm.KnoxLdapRealm"/>
            <param name="main.ldapContextFactory" value="org.apache.hadoop.gateway.shirorealm.KnoxLdapContextFactory"/>
            <param name="main.ldapRealm.contextFactory" value="$ldapContextFactory"/>

            <param name="main.ldapRealm.contextFactory.url" value="ldap://c7301.ambari.apache.org:389"/>
            <param name="main.ldapRealm.contextFactory.systemUsername" value="cn=admin,dc=ambari,dc=apache,dc=org"/>
            <param name="main.ldapRealm.contextFactory.systemPassword" value="1"/>

            <param name="main.ldapRealm.userSearchBase" value="ou=People,dc=ambari,dc=apache,dc=org"/>
            <param name="main.ldapRealm.userSearchAttributeName" value="uid"/>
            <param name="main.ldapRealm.userObjectClass" value="person"/>

            <param name="main.ldapRealm.authorizationEnabled" value="true"/>
            <param name="main.ldapRealm.groupSearchBase" value="ou=Group,dc=ambari,dc=apache,dc=org"/>
            <param name="main.ldapRealm.groupObjectClass" value="groupofnames"/>
            <param name="main.ldapRealm.groupIdAttribute" value="cn"/>
            <param name="main.ldapRealm.memberAttribute" value="member"/>

            <param name="urls./**" value="authcBasic"/>
        </provider>
               <provider>
                    <role>identity-assertion</role>
                    <name>Default</name>
                    <enabled>true</enabled>
                </provider>

                <provider>
                    <role>authorization</role>
                    <name>AclsAuthz</name>
                    <enabled>true</enabled>
                    <param name="knox.acl" value="*;scientist;*"/>
                </provider>

    </gateway>

    <service>
        <role>KNOX</role>
    </service>
</topology>
```
用knox自带的测试工具进行测试：
```
$ /usr/hdp/2.6.1.0-129/knox/bin/knoxcli.sh user-auth-test --cluster default --u sam --p '1' --g
LDAP authentication successful!
sam is a member of: scientist
sam is a member of: analyst
$ /usr/hdp/2.6.1.0-129/knox/bin/knoxcli.sh user-auth-test --cluster default --u tom --p '1' --g
LDAP authentication successful!
tom is a member of: analyst
```
上面的测试说明sam隶属于scientist组，而tom不是。  
`<param name="knox.acl" value="*;scientist;*"/>`的含义是只有`scientist`用户组的用户可以通过sample5拓扑文件访问Knox管理API。  
用curl测试：
```
$ curl -ik -u sam:1 'https://c7301.ambari.apache.org:8443/gateway/sample5/api/v1/version'
HTTP/1.1 200 OK
$ curl -ik -u tom:1 'https://c7301.ambari.apache.org:8443/gateway/sample5/api/v1/version'
HTTP/1.1 403 Forbidden
```
说明knox的用户组权限起作用了。

## 备忘

knox安装目录: ```/usr/hdp/current/knox-server```。  
ambari的knox目录：```/var/lib/ambari-server/resources/stacks/HDP/2.5/services/KNOX```。  
[Support HUE interacting with a Hadoop cluster via the Gateway](https://issues.apache.org/jira/browse/KNOX-44)  