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


## Kerberos and SPNEGO
http://www.thekspace.com/home/component/content/article/54-kerberos-and-spnego.html
Kerberos一般部署在C/S环境，很少用于web应用和瘦客户端环境。SPNEGO提供了一个机制，通过HTTP协议将kerberos扩展到了web应用。


