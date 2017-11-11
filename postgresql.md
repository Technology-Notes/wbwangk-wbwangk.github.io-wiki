环境centos7.3  

#### 安装客户端
```
$ yum install postgresql
```
#### 安装服务器
```
$ yum install postgresql-server
```

### ambari数据库
postgresql客户端远程连接HDP ambari postgresql数据库。ambari的IP是73.101，postgresql客户端IP是73.103：
```
$ psql -h 192.168.73.101 -U ambari -d ambari 
Password for user ambari: bigdata
```
#### 创建数据库
在73.101节点上执行下列命令创建数据库pdi_logging：
```
$ sudo -u postgres psql
postgres=# CREATE USER kettle WITH PASSWORD 'vagrant';              (新建一个数据库用户hive，密码是vagrant)
CREATE ROLE
postgres=# CREATE DATABASE pdi_logging OWNER kettle;                       (创建用户数据库hive，并指定所有者为hive)
CREATE DATABASE
postgre=# \q    
```
远程连接数据库，并查询：
```
$ psql -h 192.168.73.101 -U kettle -d pdi_logging
Password for user kettle: vagrant
pdi_logging=>\dt          (查看所有表)
         List of relations
 Schema |  Name   | Type  | Owner
--------+---------+-------+--------
 public | pdi_log | table | kettle

pdi_logging=>select * from pdi_log;
```