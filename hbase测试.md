## Import CSV data into HBase using importtsv
[原文](https://community.hortonworks.com/articles/4942/import-csv-data-into-hbase-using-importtsv.html)  
首先，利用Ambari部署Hbase，并在过程中启用了Phoenix。hbase安装在了u1403节点。  
然后，由于集群启用了kerberos，需要用hbase的主体登录。首先要修改hbase主体的密码，否则没法用kinit登录KDC。在u1403上执行：
```
$ kinit root/admin            (登录管理员账号)
$ kadmin 
kadmin: cpw hbase-hdp1        (修改hbase-hdp1主体密码)
Enter password for principal "hbase-hdp1@AMBARI.APACHE.ORG": 1    (密码是1)
```
hdp1是ambari集群的名字，实际主体的名称需要替换成你自己的集群名字。  
先登录kerberos，然后利用hbase shell建表。如果不登录kerberos，建表会报错。  
```
$ kinit hbase-hdp1            
$ hbase shell
hbase(main):001:0> create 'sensor','temp','vibration','pressure'    (创建表)
hbase(main):002:0> exit                                             (退出hbase shell)
```  
