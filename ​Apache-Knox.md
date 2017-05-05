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

安装后Knox被部署在```/usr/hdp/current/knox-server```。  
