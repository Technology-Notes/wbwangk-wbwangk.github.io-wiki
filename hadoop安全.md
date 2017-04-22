##《Hadoop集群与安全》
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


## 研究Kerberos and SPNEGO
#### Kerberos and SPNEGO
http://www.thekspace.com/home/component/content/article/54-kerberos-and-spnego.html
Kerberos一般部署在C/S环境，很少用于web应用和瘦客户端环境。SPNEGO提供了一个机制，通过HTTP协议将kerberos扩展到了web应用。

[认证模式之Spnego模式](http://blog.csdn.net/wangyangzhizhou/article/details/51163782)

## O'reilly Hadoop Security

kerberos主体(principal)分成两类：用户主体UPN和服务主体SPN。  
KDC由三个组件组成：Kerberos数据库，认证服务（AS）和票证授予服务（TGS）。  

#### 主体principal命名规范
UPN的命名规范：  
 - alice@EXAMPLE.COM  用户alice在领域EXAMPLE.COM  
 - bob/admin@EXAMPLE.COM 管理员用户bob在领域EXAMPLE.COM  
SPN的命令规范：  
 - hdfs/node1.example.com@EXAMPLE.COM  该主体代表了hdfs 服务的SPN ，位于Kerberos领域EXAMPLE.COM 的主机node1.example.com上  

#### Kerberos术语缩写
术语 | 名称 | 描述
----|-------|---------------
TGT | Ticket-granting ticket | 一个特殊票据类型，发放给被AS认证成功的用户
TGS | Ticket-granting service | 一个验证TGT和授予服务票据的KDC服务
服务票据| service ticket | 用TGT向TGS换取，相当于令牌

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

#### [MIT Kerberos](http://web.mit.edu/~kerberos/)
