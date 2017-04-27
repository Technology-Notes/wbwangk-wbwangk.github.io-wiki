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
```
postgresql数据库默认是不允许从远程客户端访问它的。为了让hive可以远程访问postgres，还要修改postgresql的配置文件:，在文件的最后添加：
```
$ echo "host all all 0.0.0.0 0.0.0.0 md5" >> /etc/postgresql/9.3/main/pg_hba.conf
$ /etc/init.d/postgresql restart            (重启postgresql)
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