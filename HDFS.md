未启用kerberos的情况下访问WebHDFS：
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