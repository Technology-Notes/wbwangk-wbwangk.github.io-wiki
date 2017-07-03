[hortonworks hive教程](https://hortonworks.com/tutorial/how-to-process-data-with-apache-hive/)  
## 用ambari安装hive
#### 1.为hive准备存储元数据的数据库
参考postgres[入门文档](http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html)。  
通过Ambari部署hive。在向导页面上，选择使用现有的postgreSQL数据库存放Hive的元数据。Ambari安装时自动安装了一个postgreSQL数据库。需要为hive创建额外的数据库hive和用户hive。  

在Ambari-server所在的节点(u1401)上执行：
```
$ sudo -u postgres psql
postgres=# CREATE USER hive WITH PASSWORD 'vagrant';              (新建一个数据库用户hive，密码是vagrant)
CREATE ROLE
postgres=# CREATE DATABASE hive OWNER hive;                       (创建用户数据库hive，并指定所有者为hive)
CREATE DATABASE
postgre=# \q                                                      (退出)
```
postgresql数据库默认是不允许从远程客户端访问它的。为了让hive可以远程访问postgres，还要修改postgresql的配置文件:，在文件的最后添加配置项。如果是在ubuntu下：
```
$ echo "host all all 0.0.0.0 0.0.0.0 md5" >> /etc/postgresql/9.3/main/pg_hba.conf
$ /etc/init.d/postgresql restart            (重启postgresql)
```
centos7下的操作：
```
$ echo "host all all 0.0.0.0 0.0.0.0 md5" >> /var/lib/pgsql/data/pg_hba.conf
$ service postgresql restart
```
#### 2.ambari安装hive的向导
在Hive Matestore参数页中选择“Existing PostgreSQL Database”。  
然后输入其他参数：  
```
Database Name : hive
Database Username : hive
Database Password : vagrant   vagrant
JDBC Driver Class : org.postgresql.Driver
Database URL : jdbc:postgresql://u1401.ambari.apache.org:5432/hive
```
点击Test connection按钮测试数据库连接。   
之后出现输入ambari admin主体和密码。在主体输入：root/admin@AMBARI.APACHE.ORG。  

#### 3.（可选）配置ambari server JDBC驱动
安装hive的向导页面可能出现提示配置ambari serverjdbc驱动，则需要在u1401(ambari server安装节点)上执行：
```
$ wget -P /usr/share/java https://jdbc.postgresql.org/download/postgresql-42.0.0.jar
$ ambari-server setup --jdbc-db=postgres --jdbc-driver=/usr/share/java/postgresql-42.0.0.jar
```

## 碰到的问题
测试环境是启用kerberos的HDP 2.5.3。  
执行hive试图进入交互环境：
```
$ hive
Exception in thread "main" java.lang.RuntimeException: java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
(略)
Caused by: MetaException(message:Could not connect to meta store using any of the URIs provided. Most recent failure: org.apache.thrift.transport.TTransportException: GSS initiate failed
```
推测是因为没有登录kerberos。于是用root/admin(kerberos管理员)账号登录后执行hive：
```
$ kinit root/admin
$ hive
Exception in thread "main" java.lang.RuntimeException: org.apache.hadoop.security.AccessControlException: Permission denied: user=root, access=WRITE, inode="/user/root":hdfs:hdfs:drwxr-xr-x
```
提示hdfs中没有"/user/root"目录。  
下面用hdfs的特权用户登录kerberos，创建"/user/root"目录，并修改拥有者为root：
```
$ kinit -kt /etc/security/keytabs/hdfs.headless.keytab hdfs-hdp1
$ hdfs dfs -mkdir /user/root
$ hdfs dfs -chown root /user/root
$ hive 
Exception in thread "main" java.lang.RuntimeException: org.apache.tez.dag.api.SessionNotRunning: TezSession has already shutdown. Application application_1498799080862_0006 failed 2 times due to AM Container for appattempt_1498799080862_0006_000002 exited with  exitCode: -1000
main : run as user is hdfs
main : requested yarn user is hdfs
Requested user hdfs is banned
```
把上述的hdfs用户换成yarn、root、hive、webb等都试过，要么被ban，要么就是用户不存在。  
