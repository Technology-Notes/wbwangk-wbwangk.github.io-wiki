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
点击Test connection按钮测试数据库连接。如果测试不通过，可以尝试下一节的配置ambari JDBC驱动。     
之后出现输入ambari admin主体和密码。在主体输入：root/admin@AMBARI.APACHE.ORG。  

#### 3.（可选）配置ambari server JDBC驱动
安装hive的向导页面可能出现提示配置ambari serverjdbc驱动，则需要在u1401(ambari server安装节点)上执行：
```
$ wget -P /usr/share/java https://jdbc.postgresql.org/download/postgresql-42.1.1.jar
$ ambari-server setup --jdbc-db=postgres --jdbc-driver=/usr/share/java/postgresql-42.1.1.jar
$ ambari-server restart
$ service postgresql restart
```
### hive视图
Ambari提供了HDFS、Hive等视图功能，提供了用户与HDFS、Hive的交互界面。  
刚进入Hive视图，检查报错，提示userhome不存在。如，用admin登录Ambari，userhome是`/user/admin`。需要手工创建这个HDFS中的目录。估计以后的Ambari版本会解决这个缺陷。  
```
$ kinit -kt /etc/security/keytabs/hdfs.headless.keytab hdfs-hdp1
$ hdfs dfs -mkdir /user/admin
$ hdfs dfs -chown admin /user/admin
```
hdp1是集群id。  
创建了上述userhome后，ambari hive view就可以运行正常了。  

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

## 用beeline测试hive
环境：centos7.3 kerberized HDP 2.6.1.0  
beeline是hive2新加的命令行工具，原命令行工具hive已经不建议使用。实际上我的环境中hive不能正常执行，只能用beeline。(下文中的`beeline>`只是提高可读性的标示)  
```
$ kinit -kt  /etc/security/keytabs/hive.service.keytab hive/c7302.ambari.apache.org@AMBARI.APACHE.ORG
$ beeline -u "jdbc:hive2://c7302.ambari.apache.org:10000/default;principal=hive/c7302.ambari.apache.org@AMBARI.APACHE.ORG"
beeline> show tables;
+-----------+--+
| tab_name  |
+-----------+--+
+-----------+--+
No rows selected (0.11 seconds)
beeline> CREATE TABLE pokes (foo INT, bar STRING);
beeline> SHOW TABLES;
+-----------+--+
| tab_name  |
+-----------+--+
| pokes     |
+-----------+--+
```
hive的home目录由`hive-default.xml`配置文件中`hive.metastore.warehouse.dir`参数控制，默认是`/apps/hive/warehouse`。  
创建完表后可以去看一下HDFS中的`/apps/hive/warehouse`目录：
```
$ hdfs dfs -ls /apps/hive/warehouse
Found 1 items
drwxrwxrwx   - hive hdfs          0 2017-07-04 02:58 /apps/hive/warehouse/pokes
```
多的两个文件就是新创建的hive表。  
加载文件到hive表：
```
beeline> LOAD DATA LOCAL INPATH '/opt/data-files-ini1.txt' OVERWRITE INTO TABLE pokes;
```
hive导入文件的分隔符比较奇怪，在vi中显示为`^A`(ASCCI码的1):
```
1^A35
48^Afour
100^A100
```
在beeline命令行中查看表的内容：
```
beeline> SELECT a.foo,a.bar FROM pokes a;
+--------+--------+--+
| a.foo  | a.bar  |
+--------+--------+--+
| 1      | 35     |
| 48     | four   |
| 100    | 100    |
+--------+--------+--+
```
通过beeline命令行中创建一个能接收csv格式导入的表：
```
beeline> create table student_csv
beeline> (sid int, sname string, gender string, language int, math int, english int)
beeline> row format delimited fields terminated by ',' stored as textfile;
```
编辑一个t.csv文件，内容是：
```
4,Rose,M,78,77,76
5,Mike,F,99,98,98
```
通过beeline命令行导入该文件：
```
beeline> LOAD DATA LOCAL INPATH '/opt/t.csv' INTO TABLE student_csv;
beeline> select * from student_csv;
+----------------+------------------+-------------------+---------------------+-----------------+--------------------+--+
|student_csv.sid |student_csv.sname |student_csv.gender |student_csv.language |student_csv.math |student_csv.english |
+----------------+------------------+-------------------+---------------------+-----------------+--------------------+--+
| 4              | Rose             | M                 | 78                  | 77              | 76                 |
| 5              | Mike             | F                 | 99                  | 98              | 98                 |
+----------------+------------------+-------------------+---------------------+-----------------+--------------------+--+
```
从hive中导出数据到HDFS的`/tmp/hdfs_out`:
```
beeline> INSERT OVERWRITE DIRECTORY '/tmp/hdfs_out' SELECT a.* FROM pokes a;
```
从hive中导出数据到操作系统的`/tmp/hdfs_out`:
```
beeline> INSERT OVERWRITE LOCAL DIRECTORY '/tmp/hdfs_out' SELECT a.* FROM pokes a;
```
【碰到的问题】：如果用hdfs用户登录kerberos，执行上述beeline导出命令，报错：
```
main : run as user is hdfs
main : requested yarn user is hdfs
Requested user hdfs is banned
```
定义被ban用户清单的配置问题是`/etc/hadoop/config/container-executor.cfg`:  
```
banned.users=hdfs,yarn,mapred,bin
```
选择用hive用户登录后问题解决。  

导出的文件以`^A`为分隔符。导出的文件名是000000_0。两次导出输出到了同一个文件。  