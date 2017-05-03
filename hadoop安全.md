# 《Hadoop集群与安全》
#### HDFS安全
采用了类似linux文件系统的权限模式。每个文件都拥有用户、组所有者以及权限设置。这些权限控制了用户对于特定目录或文件的访问。  
操作系统用户和用户在分布式文件系统中分配的权限之间并没有直接的联系。在更改目录或文件的所有者时，Hadoop并不会检查用户是否真正存在。

#### MapReduce安全
MapReduce安全的重心在于作业提交以及管理。  
Hadoop提供了对于集群管理员和队伍管理员这两个概念的支持。集群和队列管理员属于Linux用户和群组，他们拥有查看和修改运行作业的权限。
查看了一下本地HDP环境，发现文件/etc/hadoop/2.5.3.0-37/0/mapred-site.xml中与安全相关内容：
```
  <property>
      <name>mapreduce.cluster.administrators</name>
      <value> hadoop</value>
    </property>：
```
上述定义是mapreduce的集群管理员。  
mapred-queue-acls.xml中可以定义哪些用户或用户组可以向哪个队列中提交作业。但这个配置文件可能是个老版本(非yarn)才有的。  

####Hadoop服务级别验证
配置文件/etc/hadoop/conf/hadoop-policy.xml中定义了一些acl策略，如下列配置
```
    <property>
      <name>security.client.protocol.acl</name>
      <value>*</value>
    </property>
```
可以控制那些群组的用户可以分布式文件系统的后台服务进行通讯。  
在hadoop-policy.xml文件中有更多的选项能够对Hadoop服务(包括内部通信鞋业)访问进行控制。  

#### Hadoop与Kerberos
在配置文件/etc/hadoop/conf/core-site.xml中默认值是：
```
    <property>
      <name>hadoop.security.authentication</name>
      <value>simple</value>
    </property>
```
如果开启kerberos，需要把simple替换为kerberos。  
我选择让Ambari自动安装kerberos，参考[Ambari测试](https://github.com/wbwangk/wbwangk.github.io/wiki/Ambari%E6%B5%8B%E8%AF%95#ambari-security)。  
当使用ambari安装完kerberos后，会发现上面的core-site.xml中simple被修改成了kerberos，文件中还有其他的一些相关配置变化。  


## 《Kerberos and SPNEGO》
#### Kerberos and SPNEGO
http://www.thekspace.com/home/component/content/article/54-kerberos-and-spnego.html
Kerberos一般部署在C/S环境，很少用于web应用和瘦客户端环境。SPNEGO提供了一个机制，通过HTTP协议将kerberos扩展到了web应用。

[认证模式之Spnego模式](http://blog.csdn.net/wangyangzhizhou/article/details/51163782)


# 《O'reilly Hadoop Security》

kerberos主体(principal)分成两类：用户主体UPN和服务主体SPN。  
KDC由三个组件组成：Kerberos数据库，认证服务（AS）和票证授予服务（TGS）。  
A Kerberos realm is a set of managed nodes that share the same Kerberos database.[来源](http://publib.boulder.ibm.com/tividd/td/framework/GC32-0803-00/en_US/HTML/plan20.htm)    

#### 主体principal命名规范
UPN(用户主体名称)的命名规范：  
 - alice@EXAMPLE.COM  用户alice在领域EXAMPLE.COM  
 - bob/admin@EXAMPLE.COM 管理员用户bob在领域EXAMPLE.COM  

SPN(服务主体名称)的命令规范：  
 - hdfs/node1.example.com@EXAMPLE.COM  该主体代表了hdfs 服务的SPN ，位于Kerberos领域EXAMPLE.COM 的主机node1.example.com上。（a service name, a hostname, and a realm） 

#### Kerberos术语
术语 | 名称 | 描述
----|-------|---------------
TGT | Ticket-granting ticket | 一个特殊票据类型，发放给被AS认证成功的用户
TGS | Ticket-granting service | 一个验证TGT和授予服务票据的KDC服务
服务票据 | service ticket | 用TGT向TGS换取，相当于令牌
keytab file | keytab文件保存了实际的加密密码，用于完成给定主体的密码挑战

#### Kerberos流程：一个简单例子
例子使用的数据如下：
 - EXAMPLE.COM   Kerberos领域(realm)
 - Alice  一个系统的用户，UPN是alice@EXAMPLE.COM
 - myservice  主机server1.example.com上的一个服务，SPN是myservice/server1.example.com@EXAMPLE.COM
 - kdc.example.com   Kerberos领域EXAMPLE.COM的KDC

Alice为了使用myservice，她需要提交一个有效的服务票据(service ticket)到myservice。下面的步骤显示了她是怎么做的：  
1. Alice需要获得一个TGT。为了得到它，她发出一个请求到位于kdc.example.com的KDC，以便证明她是主体alice@EXAMPLE.COM。
2. AS的响应提供了一个TGT，TGT被主体alice@EXAMPLE.COM的密码加密。
3. 收到加密的消息后，Alice被提示输入主体alice@EXAMPLE.COM的密码，以便解密消息。
4. 当成功解密包含TGT的消息后，Alice向位于kdc.example.com的TGS请求myservice/server1.exapmle.com@EXAMPLE.COM的服务票据，TGT随请求一起发出。
5. TGS验证TGT，提供给Alice一个服务票据(service ticket)，服务票据使用主体myservice/server.example.com@EXAMPLE.COM的密码(key)进行加密。
6. Alice现在提交服务票据到myservice，myservice可以随后使用myservice/server1.example.com@EXAMPLE.COM的密码来解密票据。
7. 服务myservice准许Alice使用服务，因为她已经被成功认证。  
![](https://github.com/wbwangk/wbwangk.github.io/blob/master/images/Kerbberos_workflow_example.png?raw=true)  

#### MIT Kerberos
[MIT Kerberos的官方网站](http://web.mit.edu/~kerberos/)  
当前的版本是MIT Kerberos V5，或称krb5。  
在之前的例子中，我们提到Alice发送认证请求。在实践中，Alice使用工具kinit做这个。  
Example 4-1. kinit using the default user
```
[alice@server1 ~]$ kinit
Enter password for alice@EXAMPLE.COM:
[alice@server1 ~]$
```
这个例子使用当前Linux用户名alice和默认领域组合成了建议主体alice@EXAMPLE.COM。kinit工具还允许用户显式指定认证主体。  
Example 4-2. kinit using a specified user
```
[alice@server1 ~]$ kinit alice/admin@EXAMPLE.COM
Enter password for alice/admin@EXAMPLE.COM:
[alice@server1 ~]$
```
另一个认证选线是使用keytab文件。keytab文件保存了实际的加密密码，用于完成给定主体的密码挑战。创建keytab文件对于非交互式主体很有用，如SPN，它经常与长时间运行进程(如Hadoop守护进程)关联。多个不同主体密码(key)可以存放在同一个keytab文件中。用户可以使用kinit附加一个keytab文件和主体名称（因为keytab文件可能含有多个主体密码）来进行认证。  
Example 4-3. kinit using a keytab file  
```
[alice@server1 ~]$ kinit -kt alice.keytab alice/admin@EXAMPLE.COM
[alice@server1 ~]$
```
另一个有用的MIT Kerberos工具是klist。这个工具允许用户查看在凭据缓存中存在的Kerberos凭据。凭据缓存位于本地文件系统中，当成功被AS认证后，保存TGT。默认地址一般是文件 /tmp/krb5cc_<uid>，<uid>是本地系统的用户ID。当成功执行kinit，alice可以使用klist显示凭据缓存。  
Example 4-4. Viewing the credentials cache with klist  
```
[alice@server1 ~]$ kinit
Enter password for alice@EXAMPLE.COM:
[alice@server1 ~]$ klist
Ticket cache: FILE:/tmp/krb5cc_5000
Default principal: alice@EXAMPLE.COM
Valid starting Expires Service principal
02/13/14 12:00:27 02/14/14 12:00:27 krbtgt/EXAMPLE.COM@EXAMPLE.COM
 renew until 02/20/14 12:00:27
[alice@server1 ~]$
```
如果在认证之前查看凭据缓存，会显示没有发现凭据。  
Example 4-5. No credentials cache found  
```
[alice@server1 ~]$ klist
No credentials cache found (ticket cache FILE:/tmp/krb5cc_5000
[alice@server1 ~]$
```
另一个有用的MIT Kerberos工具是kdestory。就像名称显示的那样，它允许用户删除凭据缓存中的凭据。这个可用于切换用户，或调试新的配置。    
Example 4-6. Destroying the credentials cache with kdestroy  
```
[alice@server1 ~]$ kinit
Enter password for alice@EXAMPLE.COM:
[alice@server1 ~]$ klist
Ticket cache: FILE:/tmp/krb5cc_5000
Default principal: alice@EXAMPLE.COM
Valid starting Expires Service principal
02/13/14 12:00:27 02/14/14 12:00:27 krbtgt/EXAMPLE.COM@EXAMPLE.COM
 renew until 02/20/14 12:00:27
[alice@server1 ~]$ kdestroy
[alice@server1 ~]$ klist
No credentials cache found (ticket cache FILE:/tmp/krb5cc_5000
[alice@server1 ~]$
```
## 第5章 id和认证

Kerberos没有提供先进id功能，如分组和角色。特别是，Kerberos将id表示为一个简单的两部分字符串(或表示服务时的三部分字符串) ，一个短名称和一个领域。

#### kerberos主体映射到用户名
Kerberos使用了一个两部分字符串（如alice@EXAMPLE.COM）或三部分字符串（如hdfs/namenode.example.com@EXAMPLE.COM）,包含名称、领域和一个可选的实例名或主机名(hostname)。Hadoop将kerberos主体名称映射为本地用户名。 映射策略定义在krb5.conf文件的```auth_to_local```参数中，或core-site.xml中的hadoop特定规则hadoop.security.auth_to_local参数中。  

#### hadoop用户到用户组映射
hadoop用参数hadoop.security.group.mapping去控制用户到用户组的映射。默认实现是通过原生调用或本地shell命令，并利用标准UNIX接口去查找用户到组的映射。如果集群中各个主机定义的用户组不同，会导致用户到组的映射不一致。对于hadoop，映射发生在NodeNode、JobTracker(对于MR1)和ResourceManager(对于YARN/MR2)。

#### Hadoop用户供给
hadoop集群需要一个统一的用户数据库，并且将用户同步到集群中所有服务器的本地操作系统中。并且要控制这些用户在本地操作系统中权限。

### 认证

Table 5-4. Hadoop ecosystem authentication methods  

Service | Protocol | Methods
--------|----------|--------
HDFS | RPC | Kerberos, delegation token
HDFS | Web UI | SPNEGO (Kerberos), pluggable
HDFS| REST (WebHDFS)| SPNEGO (Kerberos), delegation token
HDFS| REST (HttpFS)| SPNEGO (Kerberos), delegation token
MapReduce| RPC| Kerberos, delegation token
MapReduce| Web UI| SPNEGO (Kerberos), pluggable
YARN| RPC| Kerberos, delegation token
YARN| Web UI| SPNEGO (Kerberos), pluggable
Hive Server 2| Thrift| Kerberos, LDAP (username/password)
Hive Metastore| Thrift| Kerberos, LDAP (username/password)
Impala| Thrift| Kerberos, LDAP (username/password)
HBase| RPC| Kerberos, delegation token
HBase| Thrift Proxy| None
HBase| REST Proxy| SPNEGO (Kerberos)
Accumulo| RPC| Username/password, pluggable
Accumulo| Thrift Proxy| Username/password, pluggable
Solr| HTTP| Based on HTTP container
Oozie| REST| SPNEGO (Kerberos, delegation token)
Hue| Web UI| Username/password (database, PAM, LDAP), SAML, OAuth, SPNEGO (Kerberos), remote user (HTTP proxy)
ZooKeeper| RPC| Digest (username/password), IP, SASL (Kerberos), pluggable

#### Kerberos

Hadoop支持两种认证机制：simple和kerberos。simple模式下，hadoop服务器信任所有的客户端。simple模式仅适合POC或实验室环境。  
HDFS、MapReduce、YARN、HBase、Oozie和ZooKeeper都支持Kerberos作为客户端的一个认证机制，但实现因服务和接口有所不同。  
对于以RPC为基础的协议，Simple Authentication and Security Layer (SASL)框架被用于在协议底层添加认证。理论上，所有SASL机制都应支持，实际上，仅支持GSSAPI(需要Kerberos V5)和DISGEST-MD5。  
Oozie没有RPC协议，只提供REST接口。Oozie使用 Simpleand Protected GSSAPI Negotiation Mechanism (SPNEGO),一个通过HTTP进行Kerberos认证的协议。HDFS、MapReduce、YARN、Oozie和Hue的web接口，以及HDFS (both WebHDFS and HttpFS)和HBase的REST接口也支持SPNEGO。对于SASL和SPNEGO，

Alice使用kerberos通过HDFS NameNode认证的步骤：  
1. Alice向位于kdc.example.com的TGS请求一个服务票据，请求的HDFS服务id是hdfs/namenode.example.com@EXAMPLE.COM，随请求附上她的TGT。  
2. TGS验证TGT，然后为Alice提供一个服务票据(service ticket)，服务票据用主体hdfs/namenode.example.com@EXAMPLE.COM的key(密码)加密。  
3. Alice发送服务票据到NameNode(通过SASL)，NameNode可以用hdfs/namenode.example.com@EXAMPLE.COM的key(也就是它自己的)解密和验证这个票据。

#### 用户名和密码认证

Zookeeper支持通过用户名和密码的认证。用户名和密码认证由摘要认证供应商实现，供应商会生成用户名和密码的SHA-1摘要。

#### 令牌(Tokens)

### 模拟(Impersonation)
Hadoop生态系统中有许多服务代表最终用户执行操作。为了确保安全，这些服务必须对客户端进行认证，以确保客户端可以模拟那些用户。 Oozie、Hive (in HiveServer2)和Hue都支持模拟最终用户访问HDFS、MapReduce、YARN或HBase。  
模拟有时被称为代理(proxying)。可以执行模拟的用户被称为代理人(proxy)。启用模拟的配置参数是hadoop.proxyuser.<proxy>.hosts和hadoop.proxyuser.<proxy>.groups，<proxy>是执行模拟的用户的用户名。

#### 配置
当位置好kerberos认证后，所有用户和守护进程必须提供有效的凭据才能访问RPC接口。 这意味着你必须为集群中的每一个服务器/守护进程对创建一个kerberos服务凭据。回顾一下服务主体名称(SPN)的概念，它有三部分组成：一个服务名，一个主机名，和一个领域。在hadoop中，每个作为特定服务组成部分的守护进程使用这个服务名称(对于HDFS是hdfs，对于MapReduce是mapred，对于YARN是yarn)。另外，如果你想为各种web接口启用kerberos认证，那么你还需要为HTTP服务名称提供主体。  

假设有一个hadoop集群的主机和服务如下表：  
Table 5-5. Service layout

Hostname| Daemon
--------|-------
nn1.example.com| NameNode
 |JournalNode
nn2.example.com| NameNode
  |JournalNode
snn.example.com| SecondaryNameNode
 | JournalNode
rm.example.com| ResourceManager
jt.example.com| JobTracker
 | JobHistoryServer
dn1.example.com| DataNode
 |TaskTracker
 |NodeManager
dn2.example.com| DataNode
 |TaskTracker
 |NodeManager
dn3.example.com| DataNode
 |TaskTracker
 |NodeManager

第一步是在kerberos KDC上创建所有需要的SPN，然后为每个服务器上的每个守护进程到处一个keytab文件。需要的SPN列表如下：

Table 5-6. Required Kerberos principals

Hostname| Daemon| Keytab file| SPN
--------|-------|------------|-----
nn1.example.com| NameNode/JournalNode |hdfs.keytab |hdfs/nn1.example.com@EXAMPLE.COM
 | | |HTTP/nn1.example.com@EXAMPLE.COM
nn2.example.com |NameNode/JournalNode |hdfs.keytab |hdfs/nn2.example.com@EXAMPLE.COM
 | | |HTTP/nn2.example.com@EXAMPLE.COM
snn.example.com |SecondaryNameNode/JournalNode |hdfs.keytab |hdfs/snn.example.com@EXAMPLE.COM
 | | |HTTP/snn.example.com@EXAMPLE.COM
rm.example.com |ResourceManager |yarn.keytab |yarn/rm.example.com@EXAMPLE.COM
jt.example.com |JobTracker |mapred.keytab |mapred/jt.example.com@EXAMPLE.COM
 | | |HTTP/jt.example.com@EXAMPLE.COM
 |JobHistoryServer |mapred.keytab |mapred/jt.example.com@EXAMPLE.COM
dn1.example.com |DataNode |hdfs.keytab |hdfs/dn1.example.com@EXAMPLE.COM
 | | |HTTP/dn1.example.com@EXAMPLE.COM
 |TaskTracker |mapred.keytab |mapred/dn1.example.com@EXAMPLE.COM
 | | |HTTP/dn1.example.com@EXAMPLE.COM
 |NodeManager |yarn.keytab |yarn/dn1.example.com@EXAMPLE.COM
 | | |HTTP/dn1.example.com@EXAMPLE.COM
dn2.example.com |DataNode |hdfs.keytab |hdfs/dn2.example.com@EXAMPLE.COM
 | | |HTTP/dn2.example.com@EXAMPLE.COM
 |TaskTracker |mapred.keytab |mapred/dn2.example.com@EXAMPLE.COM
 | | |HTTP/dn2.example.com@EXAMPLE.COM
 |NodeManager |yarn.keytab |yarn/dn2.example.com@EXAMPLE.COM
 | | |HTTP/dn2.example.com@EXAMPLE.COM
dn3.example.com |DataNode |hdfs.keytab |hdfs/dn3.example.com@EXAMPLE.COM
 | | |HTTP/dn3.example.com@EXAMPLE.COM
 |TaskTracker |mapred.keytab |mapred/dn3.example.com@EXAMPLE.COM
 | | |HTTP/dn3.example.com@EXAMPLE.COM
 |NodeManager |yarn.keytab |yarn/dn3.example.com@EXAMPLE.COM
 | | |HTTP/dn3.example.com@EXAMPLE.COM

keytab文件的建议存放目录是$HADOOP_CONF_DIR目录(一般是/etc/hadoop/conf)。  
当创建好需要的SPN，并将keytab文件分发好，需要配置Hadoop使用Kerberos认证。首先，设置core-site.xml文件的hadoop.security.authentication参数：
```xml
<property>
 <name>hadoop.security.authentication</name>
 <value>kerberos</value>
 </property>
```
以HDFS的配置文件是hdfs-site.xml。NameNode服务的配置如下：
```xml
<property>
 <name>dfs.block.access.token.enable</name>
 <value>true</value>
 </property>
 <property>
 <name>dfs.namenode.keytab.file</name>
 <value>hdfs.keytab</value>
 </property>
 <property>
 <name>dfs.namenode.kerberos.principal</name>
 <value>hdfs/_HOST@EXAMPLE.COM</value>
 </property>
 <property>
 <name>dfs.namenode.kerberos.internal.spnego.principal</name>
 <value>HTTP/_HOST@EXAMPLE.COM</value>
 </property>
```
上例中的```_HOST```是通配符，实际执行时会被替换为具体的主机名。  
（其他Hadoop服务的配置文件略。）  

## 第6章 授权

身份验证仅仅是整体安全性故事的一部分，还需要一种方法来对经过身份验证的用户可以访问的操作或数据进行建模。以这种方式保护资源称为授权。

### HDFS授权
每次尝试访问HDFS中的文件或目录必须首先通过授权检查。HDFS采用与POSIX兼容的文件系统通用的授权方案。权限由三个不同类别的用户管理：所有者，组和其他人。读取，写入和执行权限可以独立授予每个类。  
无论文件或目录的权限如何，NameNode运行的用户（通常为hdfs）和dfs.permis sions.superusergroup（默认为超组）中定义的组中的任何成员都可以读取，写入或删除任何文件和目录。就HDFS而言，它们在Linux系统上相当于root。
#### 服务级别授权
Hadoop支持服务级别授权。授权策略保存在hadoop-policy.xml中。在该xml中可以定义“权力清单（如提交作业到集群的权力）”与用户（用户组）的对应。用户之间用逗号隔开，用户与用户组之间用空格隔开。以空格开始表示只定义了用户组，以空格结尾表示只定义了用户。  

### MapReduce和YARN授权
无论MapReduce还是YARN都不需要控制数据访问，而是要控制集群资源，如CPU、内存、磁盘I/O和网络I/O。服务级别授权仅可以控制到特定协议(protocol)，如谁可以提交一个作业到集群，但无法细致到控制集群资源的访问。  
Hadoop支持将ACL定义到作业队列。这些ACL控制哪些用户可以提交作业到特定队列，还可以控制哪些用户可以管理(administer)队列。

## 第11章 数据提取和客户端访问安全
#### 边缘节点
边缘节点托管web接口、代理和客户端配置。边缘节点部署的服务有：  
 - HDFS HttpFS and NFS gateway
 - Hive HiveServer2 and WebHCatServer
 - Network proxy/load balancer for Impala
 - Hue server and Kerberos ticket renewer
 - Oozie server
 - HBase Thrift server and REST server
 - Flume agent
 - Client configuration files

下面是一些典型的边缘节点类型：
 - Data Gateway  
HDFS HttpFS and NFS gateway, HBase Thrift server and REST server, Flume agent  
  - SQL Gateway  
Hive HiveServer2 and WebHCatServer, Impala load-balancing proxy (e.g., HAP‐roxy)
 - User Portal  
Hue server and Kerberos ticket renewer, Oozie server, client configuration files  

### Hadoop命令行接口
核心Hadoop命令行工具（hdfs、yarn和mapred）仅支持Kerberos或授权令牌进行身份验证。认证这些命令的最简单的方法是在执行命令之前使用kinit 获取Kerberos票证授予票据(ticket-granting ticket)。
```
$ kinit webb3@AMBARI.APACHE.ORG                (登录使用kerberos的kinit工具)
$ hdfs dfs -mkdir /tmp/webb          (创建/tmp/webb目录)
$ hdfs dfs -ls /tmp                  (查看/tmp目录)
...
drwxr-xr-x   - webb3     hdfs          0 2017-04-25 02:31 /tmp/webb     (这是新创建的目录，注意这个目录的拥有者是webb3)
```

#### hive命令行测试
HDFS的superuser是hdfs。当haddop集群启用kerberos后，hdfs的主体并没有在KDC中建立。测试中碰到针对HDFS的权限不足时，可能会用到HDFS的超级用户，可以用kadmin命令来在KDC创建hdfs的主体。在u1404上执行：
```
$ kadmin
kadmin:  add_principal hdfs      
Enter password for principal "hdfs@AMBARI.APACHE.ORG":            (需要输入hdfs的密码)
Re-enter password for principal "hdfs@AMBARI.APACHE.ORG":         (再次输入hdfs的密码)
Principal "hdfs@AMBARI.APACHE.ORG" created.
```
使用hive命令行工具beeline连接HiveServer2。其中```jdbc:hive2://u1403.ambari.apache.org:10000/default```是JDBC URL，```principal=hive/u1403.ambari.apache.org@AMBARI.APACHE.ORG```是连接使用的Hive Kerberos主体。beeline命令以感叹号!开始。
```
$ kinit webb3@AMBARI.APACHE.ORG          (使用主体webb3访问Hive)
$ beeline
Beeline version 1.2.1000.2.5.3.0-37 by Apache Hive
beeline> !connect jdbc:hive2://u1403.ambari.apache.org:10000/default;principal=hive/u1403.ambari.apache.org@AMBARI.APACHE.ORG
Connecting to jdbc:hive2://u1403.ambari.apache.org:10000/default;principal=hive/u1403.ambari.apache.org@AMBARI.APACHE.ORG
Enter username for jdbc:hive2://u1403.ambari.apache.org:10000/default;principal=hive/u1403.ambari.apache.org@AMBARI.APACHE.ORG:
Enter password for jdbc:hive2://u1403.ambari.apache.org:10000/default;principal=hive/u1403.ambari.apache.org@AMBARI.APACHE.ORG:
Connected to: Apache Hive (version 1.2.1000.2.5.3.0-37)
Driver: Hive JDBC (version 1.2.1000.2.5.3.0-37)
Transaction isolation: TRANSACTION_REPEATABLE_READ
```
下面在Hive创建一个外部表，表文件存放在前面创建的HDFS```/tmp/webb/student```目录下：
```
0: jdbc:hive2://u1403.ambari.apache.org:10000> CREATE EXTERNAL TABLE student(name string, age int, gpa double) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'STORED AS TEXTFILE LOCATION '/tmp/webb/student';
0: jdbc:hive2://u1403.ambari.apache.org:10000> show tables;    （显示hive中数据库表清单）
+---------------+--+
|   tab_name    |
+---------------+--+
| student       |
+---------------+--+
3 rows selected (0.284 seconds)
0: jdbc:hive2://u1403.ambari.apache.org:10000> SELECT * FROM student;       （显示hive表student中数据，可看到列名，数据空）
+---------------+--------------+--------------+--+
| student.name  | student.age  | student.gpa  |
+---------------+--------------+--------------+--+
+---------------+--------------+--------------+--+
No rows selected (0.425 seconds)
```
下面利用直接利用hdfs命令行看看HDFS中hive建立的/tmp/webb/student文件：
```
$ hdfs dfs -ls /tmp/webb
Found 1 items
drwxr-xr-x   - webb3 hdfs          0 2017-04-25 07:48 /tmp/webb/student
```
看到hive表对应到HDFS中是个同名的目录。  

