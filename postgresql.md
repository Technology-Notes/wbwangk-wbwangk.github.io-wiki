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