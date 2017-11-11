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

pdi_logging=>select * from pdi_log
id_batch |channel_id| transname | status | lines_read | lines_written| lines_updated | lines_input | lines_output | lines_rejected | errors |startdate|enddate | logdate | depdate | replaydate| log_field
...
```

id_batch |              channel_id              | transname | status | lines_read | lines_written
| lines_updated | lines_input | lines_output | lines_rejected | errors |      startdate      |
    enddate         |         logdate         |         depdate         |       replaydate        |
                                                log_field

----------+--------------------------------------+-----------+--------+------------+---------------
+---------------+-------------+--------------+----------------+--------+---------------------+-----
--------------------+-------------------------+-------------------------+-------------------------+
---------------------------------------------------------------------------------------------------
-------
        0 | 0cfd2552-58ba-4154-b573-e1e34600fbae | hello     | end    |          0 |             0
|             0 |           0 |            0 |              0 |      0 | 1900-01-01 07:00:00 | 2017
-11-11 11:46:38.777 | 2017-11-11 11:46:39.227 | 2017-11-11 11:46:38.777 | 2017-11-11 11:46:38.777 |
 2017/11/11 11:46:38 - Spoon - Using legacy execution engine\r
      +
          |                                      |           |        |            |
|               |             |              |                |        |                     |
                    |                         |                         |                         |
 2017/11/11 11:46:38 - Spoon - Transformation opened.\r
      +
          |                                      |           |        |            |
|               |             |              |                |        |                     |
                    |                         |                         |                         |
 2017/11/11 11:46:38 - Spoon - Launching transformation [hello]...\r
      +
          |                                      |           |        |            |
|               |             |              |                |        |                     |
                    |                         |                         |                         |
 2017/11/11 11:46:38 - Spoon - Started the transformation execution.\r
      +
          |                                      |           |        |            |
|               |             |              |                |        |                     |
                    |                         |                         |                         |
 2017/11/11 11:46:38 - hello - Dispatching started for transformation [hello]\r
      +
          |                                      |           |        |            |
|               |             |              |                |        |                     |
                    |                         |                         |                         |
 2017/11/11 11:46:38 - name list.0 - Header row skipped in file 'E:\data-integration\Tutorial\list.
csv'\r+
          |                                      |           |        |            |
|               |             |              |                |        |                     |
                    |                         |                         |                         |
 2017/11/11 11:46:38 - name list.0 - Finished processing (I=7, O=0, R=0, W=6, U=0, E=0)\r
      +
          |                                      |           |        |            |
