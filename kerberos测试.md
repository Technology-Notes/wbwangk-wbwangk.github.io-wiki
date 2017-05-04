Kerberos KDC的安装参考[这个](https://github.com/wbwangk/wbwangk.github.io/wiki/Ambari%E6%B5%8B%E8%AF%95#ambari-security)  

#### 环境介绍
u1401：ambari-server
u1402、u1403：HDFS  
u1404：kerberos KDC

#### 创建用户
在u1404上执行：
```
$ kadmin.local -q "addprinc webb3"                          (添加主体webb3)
Enter password for principal "webb3@AMBARI.APACHE.ORG":     (输入两次密码)
```
#### 登录
在u1402上登录webb3：
```
$ kinit webb3@AMBARI.APACHE.ORG                (登录使用kerberos的kinit工具)
Password for webb3@AMBARI.APACHE.ORG:          (输入主体webb3的密码)
$ klist                                        (klist显示凭据缓存)
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: webb3@AMBARI.APACHE.ORG     (webb3是默认主体)

Valid starting       Expires              Service principal
04/25/2017 02:02:29  04/25/2017 12:02:29  krbtgt/AMBARI.APACHE.ORG@AMBARI.APACHE.ORG
        renew until 05/02/2017 02:02:27
```
#### 访问HDFS
仍在u1402上操作HDFS，当前的默认主体是webb3。显示HDFS的目录：
```
$ hdfs dfs -ls /
Found 9 items
drwxrwxrwx   - yarn   hadoop          0 2017-03-28 18:06 /app-logs
drwxr-xr-x   - hdfs   hdfs            0 2017-03-28 16:04 /apps
drwxr-xr-x   - yarn   hadoop          0 2017-03-28 15:52 /ats
drwxr-xr-x   - hdfs   hdfs            0 2017-03-28 15:52 /hdp
drwxr-xr-x   - mapred hdfs            0 2017-03-28 15:52 /mapred
drwxrwxrwx   - mapred hadoop          0 2017-03-28 15:53 /mr-history
drwxrwxrwx   - spark  hadoop          0 2017-04-25 02:29 /spark-history
drwxrwxrwx   - hdfs   hdfs            0 2017-04-20 02:43 /tmp
drwxr-xr-x   - hdfs   hdfs            0 2017-03-28 18:05 /user
```
在HDFS上创建/tmp/webb目录：
```
$ hdfs dfs -mkdir /tmp/webb          (创建/tmp/webb目录)
$ hdfs dfs -ls /tmp                  (查看/tmp目录)
Found 5 items
drwxr-xr-x   - hdfs      hdfs          0 2017-03-28 15:52 /tmp/entity-file-history
drwx-wx-wx   - ambari-qa hdfs          0 2017-03-28 16:09 /tmp/hive
-rwxr-xr-x   3 hdfs      hdfs       1724 2017-04-20 02:44 /tmp/ida8c0670e_date432017
-rwxr-xr-x   3 hdfs      hdfs       1425 2017-03-28 10:54 /tmp/ida8c0670e_date532817
drwxr-xr-x   - webb3     hdfs          0 2017-04-25 02:31 /tmp/webb     (这是新创建的目录)
```
可以看到新创建目录/tmp/webb3的拥有者是webb3。这说明kerberos用户与HDFS集成的很好。