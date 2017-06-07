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

