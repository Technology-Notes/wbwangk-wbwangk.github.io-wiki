ranger是一个hadoop安全服务，开源自hortonworks，官网地址：ranger.apache.org。另一个类似的hadoop安全服务[sentry](sentry.apache.org)开源自cloudera。  
## 安装Ranger
当试图使用ambari部署ranger时，ambari的提示如下：
 1. You must have an MySQL/Oracle/Postgres/MSSQL/SQL Anywhere Server database instance running to be used by Ranger.
 2. In Assign Masters step of this wizard, you will be prompted to specify which host for the Ranger Admin. On that host, you must have DB Client installed for Ranger to access to the database. (Note: This is applicable for only Ranger 0.4.0)
 3. Ensure that the access for the DB Admin user is enabled in DB server from any host.
 4. Execute the following command on the Ambari Server host. Replace database-type with mysql|oracle|postgres|mssql|sqlanywhere and /jdbc/driver/path based on the location of corresponding JDBC driver:
```
ambari-server setup --jdbc-db={database-type} --jdbc-driver={/jdbc/driver/path}
```
第一条说的是，部署ranger需要有一个运行的数据库环境，如mysql、postgres等。第二条说，要部署ranger的服务器上要先安装对应数据库的客户端。第三条说的是，确保知道数据库的管理员账号和密码，并且可以从其他机器上访问。第四条说，需要设置ambari-server的jdbc驱动。  

部署ambari-server时，它自动安装了一个postgres数据库。下面试图直接使用这个ambari-server自带的postgres来安装ranger。这样省得再额外安装一个数据库。  

### 测试和调整postgres server
参考postgres[入门文档](http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html)。  
查看Ambari安装文档，文档说server默认安装了一个PostgreSQL数据库。启动postgresql进程的linux用户名是postgres，数据库名是ambari。数据库的默认用户名和密码是ambari/bigdata。   

在Ambari server所在的机器(u1401)上运行：
```
$ sudo -u postgres psql
postgres=# alter user postgres with password 'vagrant';              (调整数据库用户postgres的密码)
```
postgres就是管理员账号，在利用ambari安装ranger的向导页面上需要输入管理员账号和密码。  
ambari会利用postgres的权限创建ranger数据库，创建叫rangeradmin的数据库用户。当然，数据库ranger和用户rangeradmin都是可以配置个性的名称。  

postgresql数据库默认是不允许从远程客户端访问它的。为了让ranger可以远程访问postgres，还要修改postgresql的配置文件/etc/postgresql/9.3/main/pg_hba.conf，在文件的最后添加：
```
host all all 0.0.0.0 0.0.0.0 md5      #表示运行任何IP连接
``` 
重启postgresql：  
```
/etc/init.d/postgresql restart
```
### 安装postgres客户端并测试远程连接
计划在u1402上安装ranger，所以需要在u1402上安装postgres客户端：  
```
$ apt install postgresql-client
$ psql -h u1401 -U postgres -d ambari  (提示输入密码就输入vagrant)
ambari=>                  (这种提示表示进入了postgres的交互式环境)
```
-U表示数据库用户，-d表示数据库名。  

### ambari设置jdbc驱动
在ambari-server所在机器（u1401）上下载postgres的jdbc驱动，并执行ambari配置：
```
$ cd /usr/share/java
$ wget https://jdbc.postgresql.org/download/postgresql-42.0.0.jar
$ ambari-server setup --jdbc-db=postgres --jdbc-driver=/usr/share/java/postgresql-42.0.0.jar
```
### 通过ambari安装ranger
在ambari中通过菜单Services/Actions/Add Service打开安装向导，选择ranger服务。到配置页面后按下面
DB FLAVOR下拉框选择POSTGRES； 
Ranger DB host输入u1401.ambari.apache.org;  
Database Administrator (DBA) username中输入postgres，密码输入vagrant。点击测试连接按钮，应该可以成功。  
solr审计URL随便输入：http://solr_host:6083/solr/ranger_audits。  
当提示输入主体时输入：root/admin@AMBARI.APACHE.ORG  

## 配置Ranger
### 在ambari中启用ranger插件
通过Services/Ranger/configs进入Ranger配置页面，选择Ranger Plugin选项卡。由于我已经安装了HDFS/YARN/Hive，在选项卡中显示了HDFS Ranger Plugin、YARN Ranger Plugin、Hive Ranger Plugin三个选项，三个选项都选On，然后点Save按钮。  
重启受影响的多个服务。  
对于启用了Kerberos的集群，还需要额外操作。  

#### 对于HDFS的额外操作
对于Kerberos集群，启用Ranger HDFS插件，还需要额外的操作：
 1. 在u1402（Ranger安装节点）上创建一个操作系统用户rangerhdfslookup：
```
$ adduser rangerhdfslookup
```
通过菜单Services=>Ranger=>Quick Links=>Ranger Admin UI进入Ranger管理员界面(```http://u1402:6080```)。用户名口令是admin/admin。通过菜单Settings=>Users/Groups可以看到Ranger中用户，确保看到新建的rangerhdfslookup用户已经同步过来了。  
 2. 在Kerberos KDC(u1404)上新建一个rangerhdfslookup的主体：  
```
$ kadmin.local -q 'addprinc -pw rangerhdfslookup rangerhdfslookup@AMBARI.APACHE.ORG'   (密码是rangerhdfslookup)
```
 3. 导航到Ambari的HDFS服务，点Config选项卡，进入advanced ranger-hdfs-plugin-properties，更新参数：

 参数名称 | 值
---------|--------
Ranger repository config user | rangerhdfslookup@AMBARI.APACHE.ORG
Ranger repository config password | rangerhdfslookup
common.name.for.certificate | (空)
 4. 点Save按钮，重启HDFS服务。

#### 对Hive的额外操作
对于Kerberos集群，启用Ranger Hive插件，还需要额外的操作：
 1. 在Ranger安装节点创建操作系统用户rangerhivelookup，象HDFS一样，确保这个用户被同步到了Ranger Admin。  
 2. 为rangerhivelookup创建Kerberos主体：
```
$ kadmin.local -q 'addprinc -pw rangerhivelookup rangerhivelookup@AMBARI.APACHE.ORG'
```
 3. 导航到Ambari的Hive服务，点Config选项卡，进入advanced ranger-hive-plugin-properties，更新参数：

 参数名称 | 值
---------|--------
Ranger service config user | rangerhivelookup@AMBARI.APACHE.ORG
Ranger service config password | rangerhivelookup
common.name.for.certificate | (空)
 4. 点Save按钮，重启Hive服务。

## 使用Ranger为Hadoop提供授权
当用户被认证后，需要确定他的访问权限。授权定义用户对资源的访问权限。你可以用Ranger建立和管理针对Hadoop各种服务的访问权限。你可以建立基于标签的服务，挺对这些服务添加访问策略。使用基于标签(tag-based)的策略，允许你跨越多个Hadoop组件控制资源访问权限，而不用在每个组件中建立单独的服务和策略。你可以使用Ranger TagSync去同步Ranger标签库到外部元数据服务，如Apache Atlas。

### 关于Ranger策略
#### Ranger基于资源的策略
Ranger允许你为特定Hadoop资源(HDFS、HBase、Hive等)创建服务，和添加访问策略到这些服务。
#### Ranger基于标签的策略
 - Ranger标签授权的一个重要特征是资源分类与访问授权的分离。不同的资源包含了不同的数据，则分别表打上不同的标签。如HDFS文件包含了社会安全号码、信用卡号、敏感健康数据可以分别被标签为PII/PCI/PHI。
 - 使用基于标签的策略，允许你跨越多个Hadoop组件来控制资源访问，而不同为每个组件单独建立服务或策略。
 - 标签明细被保存在标签库中。Ranger TagSync可以用于在标签库和外部元数据服务(如Apache Altas)之间同步标签。
#### 标签和策略评估
![](http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.3/bk_security/content/figures/3/figures/Ranger-Policy-Evaluation-Flow-with-Tags.png)

## Ranger配置
Ranger的认证方式有3种：  
- LDAP
- 活动目录(AD)
- UNIX
第3种指使用Ranger主机的操作系统用户/口令作为认证凭据，这也是默认的和配置最容易的认证方式。  


