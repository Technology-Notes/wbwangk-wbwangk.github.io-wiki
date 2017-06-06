为了更好读懂本文，最好之前阅读[Hadoop安全](https://github.com/wbwangk/wbwangk.github.io/wiki/hadoop%E5%AE%89%E5%85%A8)和[LDAP测试](https://github.com/wbwangk/wbwangk.github.io/wiki/LDAP)。  
## Apache Knox网关概述
Apache Knox网关（“Knox”）是一种系统，可将Apache™Hadoop®服务的覆盖范围扩展到Hadoop集群外的用户，而不会减少Hadoop安全性。Knox还为访问群集数据和执行作业的用户简化了Hadoop安全性。

Knox与企业中使用的身份管理和SSO系统集成，并允许将这些系统的身份用于访问Hadoop集群。

Knox网关为多个Hadoop集群提供安全性，具有以下优点：

 - **简化访问**：通过将Kerberos封装到群集中来扩展Hadoop的REST/HTTP服务。
 - **提高安全性**：暴露Hadoop的REST/HTTP服务，而不会透露网络细节，提供SSL开箱即用。
 - **集中控制**：集中执行REST API安全性，将请求路由到多个Hadoop集群。
 - **企业集成**：支持LDAP，AD，SSO，SAML等认证系统。  

![](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.3/bk_security/content/figures/2/figures/Typical_security_flow_via_Knox.png)

典型安全流程：防火墙，通过Knox网关路由  

Knox可以与两个不安全的Hadoop集群和Kerberos安全集群一起使用。在采用Kerberos安全集群的企业解决方案中，Apache Knox Gateway提供了企业安全解决方案：  
 - 与企业身份管理解决方案相结合
 - 保护Hadoop集群部署的细节（主机和端口从最终用户隐藏）
 - 简化客户需要交互的服务数量

#### Knox网关部署体系结构

用户通过Knox访问Hadoop集群，可以使用Apache REST API或Hadoop CLI工具进行。
下图显示了Apache Knox如何适应Hadoop部署。  
![](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.3/bk_security/content/figures/2/figures/REST_API_Security_Drill-down.png)  
NN = NameNode，RM =资源管理器，DN = DataNode，NM = NodeManager

#### Knox支持的Hadoop服务

Apache Knox Gateway支持Kerberized和Non-Kerberized集群中的以下Hadoop服务版本：  

服务 | 版本  
---- | ------  
YARN | 2.6.0  
WebHDFS | 2.6.0  
WebHCat/Templeton | 0.13.0  
Oozie | 4.1.0  
​HBase/Stargate | 0.98.4  
Hive（通过WebHCat） | 0.14.0  
Hive（通过JDBC） | 0.14.0  
Ambari | 2.4.0  
Ranger | 0.6.0  

### 认证配置
Knox支持两种类型的提供者：  
 - 认证提供者  
 - 联邦提供者  
认证提供者直接接受用户的凭据，并通过特定的用户数据库进行验证。而联邦提供者只验证由可信认证中心（IdP）为用户颁发的令牌（不直接验证凭据）。

提供者有基于名称的配置。存在多种认证提供者类型：  
 - Anonymous  
诺克斯用于让代理的服务或用户界面进行自己的身份验证。
 - ShiroProvider  
对于LDAP/AD身份验证，使用用户名和密码。没有SPNEGO/Kerberos支持。
 - HadoopAuth  
对于SPNEGO/Kerberos身份验证，使用委派令牌。没有LDAP/AD支持。

### Knox网关的网址映射
网关配置文件（即{GATEWAY_HOME}/conf/gateway-site.xml）中定义gateway-port（默认8443）、gateway-path(默认是gateway)。而GATEWAY_HOME的默认路径是/usr/hdp/current/knox-server。  
集群拓扑描述符（{GATEWAY_HOME}/conf/topologies/{cluster-name}.xml）定义了knox管理的hadoop集群。  
*可以到```{GATEWAY_HOME}/conf/topologies/```目录下看看有哪几个xml文件，一般会有amdin.xml、default.xml和knoxsso.xml等。*  
下面实现了各个hadoop服务的原始URL和通过knox网关后的URL：  
- WebHDFS
  - 网关： ```https://{gateway-host}:{gateway-port}/{gateway-path}/{cluster-name}/webhdfs```
  - 集群： ```http://{webhdfs-host}:50070/webhdfs```
- WebHCat（Templeton）
  - 网关： ```https://{gateway-host}:{gateway-port}/{gateway-path}/{cluster-name}/templeton```
  - 集群： ```http://{webhcat-host}:50111/templeton}```
- Oozie的
  - 网关： ```https://{gateway-host}:{gateway-port}/{gateway-path}/{cluster-name}/oozie```
  - 集群： ```http://{oozie-host}:11000/oozie}```
- HBase的
  - 网关： ```https://{gateway-host}:{gateway-port}/{gateway-path}/{cluster-name}/hbase```
  - 集群： ```http://{hbase-host}:8080```
- Hive JDBC
  - 网关： ```jdbc:hive2://{gateway-host}:{gateway-port}/;ssl=true;sslTrustStore={gateway-trust-store-path};trustStorePassword={gateway-trust-store-password};transportMode=http;httpPath={gateway-path}/{cluster-name}/hive```
  - 集群： ```http://{hive-host}:10001/cliservice```

#### knox 配置
core-site.xml中定义了knox了安装主机(hadoop.proxyuser.knox.hosts)和可以代理的用户组(hadoop.proxyuser.knox.groups)。
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
在我的测试中FQDN_OF_KNOX_HOST被替换为```u1401.ambari.apache.org```，也就是安装knox网关的虚拟机。把```users```修改成了```*```，即允许knox代理（仿冒）所有用户组的用户。  

#### ShiroProvider(LDAP认证)
用ambari安装的knox，默认安装目录是```/usr/hdp/current/knox-server```。默认cluster-name是default，对应的配置文件是：
```
/usr/hdp/current/knox-server/conf/topologies/default.xml
```
这个配置文件可以在ambari中通过界面修改，点击Knox服务后点config然后在配置文件的Advanced topology小节中。将第一个xml的第一个provider元素替换为下列内容：
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

#### ShiroProvider(LDAP认证)测试
首先需要通过ambari禁用kerberos，否则会报一些错误。  
然后需要准备LDAP环境。参考[这个](https://github.com/wbwangk/wbwangk.github.io/wiki/LDAP)LDAP测试的文档。  
按上述文档进行的测试在u1401上安装了OpenLDAP，并创建了一个测试用户john（dn: uid=john,ou=People,dc=ambari,dc=apache,dc=org），该用户的密码是johnldap。使用该用户测试Knox：
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

## 授权
本章内容源自[knox的apache官网文档](http://knox.apache.org/books/knox-0-12-0/user-guide.html#Authorization)  
#### 服务级别授权
Knox Gateway具有开箱即用的授权提供者程序，允许管理员限制对Hadoop集群中各个服务的访问。该提供者利用简单熟悉的ACL模式来通过指定允许访问的用户，组和IP地址来保护Hadoop资源。  
使用ACL授权的配置如下：
```
<provider>
    <role>authorization</role>
    <name>AclsAuthz</name>
    <enabled>true</enabled>
</provider>
```
上述默认设置没有包含任何ACL清单，因此对于访问Hadoop服务没有任何限制。了保护资源和指定用户、组或IP的访问权限，需要提供如下参数：
```
<param>
    <name>{serviceName}.acl</name>
    <value>username[,*|username...];group[,*|group...];ipaddr[,*|ipaddr...]</value>
</param>
```
#### 身份断言的例子
身份断言提供者的主要映射方面对于理解为了充分利用该提供者的授权特征是重要的。
此功能允许我们将经过身份验证的主体映射到后端中的Hadoop服务的断定或假冒主体。
```
        <provider>
            <role>identity-assertion</role>
            <name>Default</name>
            <enabled>true</enabled>
            <param>
                <name>principal.mapping</name>
                <value>guest=hdfs;</value>
            </param>
            <param>
                <name>group.principal.mapping</name>
                <value>*=users;hdfs=admin</value>
            </param>
        </provider>
```