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

#### ShiroProvider(LDAP认证)
用ambari安装的knox，默认安装目录是```/usr/hdp/current/knox-server```。默认cluster-name是default，对应的配置文件是：
```
/usr/hdp/current/knox-server/conf/topologies/default.xml
```
通过ambari修改knox的配置文件，在配置文件的Advanced admin-topology小节中修改的配置文件如下：
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
            <provider>
                <role>authorization</role>
                <name>AclsAuthz</name>
                <enabled>true</enabled>
                <param>
                    <name>knox.acl</name>
                    <value>admin;*;*</value>
                </param>
            </provider>
            <provider>
                <role>identity-assertion</role>
                <name>Default</name>
                <enabled>true</enabled>
            </provider>
        </gateway>
        <service>
            <role>KNOX</role>
        </service>
    </topology>
```
只修改了两个参数，一个是将```main.ldapRealm.userDnTemplate```的值定义为```uid={0},ou=people,dc=ambari,dc=apache,dc=org```；另一个是将```main.ldapRealm.contextFactory.url```定义为```ldap://{{knox_host_name}}```，因未启用TLS所以没有定义为```ldapi://{{knox_host_name}}:33389```。

在配置文件的Advanced topology小节中，修改位置文件如下：
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
                <provider>
                    <role>identity-assertion</role>
                    <name>Default</name>
                    <enabled>true</enabled>
                </provider>
                <provider>
                    <role>authorization</role>
                    <name>XASecurePDPKnox</name>
                    <enabled>true</enabled>
                </provider>
            </gateway>
            <service>
                <role>NAMENODE</role>
                <url>hdfs://{{namenode_host}}:{{namenode_rpc_port}}</url>
            </service>
            <service>
                <role>JOBTRACKER</role>
                <url>rpc://{{rm_host}}:{{jt_rpc_port}}</url>
            </service>
            <service>
                <role>WEBHDFS</role>
                {{webhdfs_service_urls}}
            </service>
            <service>
                <role>WEBHCAT</role>
                <url>http://{{webhcat_server_host}}:{{templeton_port}}/templeton</url>
            </service>
            <service>
                <role>OOZIE</role>
                <url>http://{{oozie_server_host}}:{{oozie_server_port}}/oozie</url>
            </service>
            <service>
                <role>WEBHBASE</role>
                <url>http://{{hbase_master_host}}:{{hbase_master_port}}</url>
            </service>
            <service>
                <role>HIVE</role>
                <url>http://{{hive_server_host}}:{{hive_http_port}}/{{hive_http_path}}</url>
            </service>
            <service>
                <role>RESOURCEMANAGER</role>
                <url>http://{{rm_host}}:{{rm_port}}/ws</url>
            </service>
        </topology>
```
修改的配置项仍是```main.ldapRealm.userDnTemplate```和```main.ldapRealm.contextFactory.url```。  
Advanced knoxsso-topology小节也同样修改userDnTemplate和contextFactory.url。  
在ambari界面中点击Save按钮保存，并重启相关服务后。会发现```/usr/hdp/current/knox-server/conf/topologies/```目录下的admin.xml、default.xml和knoxsso.xml都按ambari中的修改更新了。  

#### ShiroProvider(LDAP认证)测试
首先需要通过ambari禁用kerberos，否则会报一些错误。  
然后需要准备LDAP环境。参考[这个](https://github.com/wbwangk/wbwangk.github.io/wiki/LDAP)LDAP测试的文档。  
按上述文档进行的测试在u1401上安装了OpenLDAP，并创建了一个测试用户john（dn: uid=john,ou=People,dc=ambari,dc=apache,dc=org），该用户的密码是johnldap。使用该用户测试Knox：
```
$ curl -i -k -u john:johnldap -X GET \
    'https://localhost:8443/gateway/default/webhdfs/v1/tmp/webb?op=LISTSTATUS'
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
