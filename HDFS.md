#### 未启用kerberos的情况下访问HDFS
```
$ curl http://u1401.ambari.apache.org:50070/webhdfs/v1/user?op=LISTSTATUS
{"FileStatuses":{"FileStatus":[
{"accessTime":0,"blockSize":0,"childrenNum":6,"fileId":16388,"group":"hdfs","length":0,"modificationTime":1493881416749,"owner":"ambari-qa","pathSuffix":"ambari-qa","permission":"770","replication":0,"storagePolicy":0,"type":"DIRECTORY"},
{"accessTime":0,"blockSize":0,"childrenNum":0,"fileId":16444,"group":"hdfs","length":0,"modificationTime":1493278869398,"owner":"hcat","pathSuffix":"hcat","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"},
{"accessTime":0,"blockSize":0,"childrenNum":0,"fileId":16455,"group":"hdfs","length":0,"modificationTime":1493278917798,"owner":"hive","pathSuffix":"hive","permission":"755","replication":0,"storagePolicy":0,"type":"DIRECTORY"}
]}}
```
而通过浏览器访问：```http://u1401.ambari.apache.org:50070/explorer.html#/user```  
发现浏览器也是调用```/webhdfs/v1/user?op=LISTSTATUS```这个API。  
使用命令行访问HDFS:
```
$ hdfs dfs -ls /user
Found 3 items
drwxrwx---   - ambari-qa hdfs          0 2017-05-04 07:03 /user/ambari-qa
drwxr-xr-x   - hcat      hdfs          0 2017-04-27 07:41 /user/hcat
drwxr-xr-x   - hive      hdfs          0 2017-04-27 07:41 /user/hive
```
#### 启用了kerberos后访问HDFS
```
$ curl http://u1401.ambari.apache.org:50070/webhdfs/v1/user?op=LISTSTATUS
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1"/>
<title>Error 401 Authentication required</title>
</head>
<body><h2>HTTP ERROR 401</h2>
<p>Problem accessing /webhdfs/v1/user. Reason:
<pre>    Authentication required</pre></p><hr /><i><small>Powered by Jetty://</small></i><br/>   
```
提示“需要认证”了。   
使用命令行访问启用了kerberos的HDFS:
```
$ hdfs dfs -ls /user
17/05/04 08:47:03 WARN ipc.Client: Exception encountered while connecting to the server :
...  (省略)
ls: Failed on local exception: java.io.IOException: javax.security.sasl.SaslException: GSS initiate failed [Caused by GSSException: No valid credentials provided (Mechanism level: Failed to find any Kerberos tgt)]; Host Details : local host is: "u1401.ambari.apache.org/192.168.14.101"; destination host is: "u1401.ambari.apache.org":8020;
```
提示没有提供有效的凭据。  
下面先使用kerberos客户端登录，然后再使用命令行：
```
$ kinit root/admin
Password for root/admin@AMBARI.APACHE.ORG:    (输入密码vagrant)
$ hdfs dfs -ls /user
Found 3 items
drwxrwx---   - ambari-qa hdfs          0 2017-05-04 08:22 /user/ambari-qa
drwxr-xr-x   - hcat      hdfs          0 2017-04-27 07:41 /user/hcat
drwxr-xr-x   - hive      hdfs          0 2017-04-27 07:41 /user/hive
```
