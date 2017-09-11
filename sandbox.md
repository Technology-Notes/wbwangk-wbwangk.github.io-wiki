登录vm：
```
$ ssh root@127.0.0.1 -p 2122            (密码hadoop)
```
登录docker：
```
$ ssh root@127.0.0.1 -p 2222            (默认密码hadoop，后改成dia****)
```
退出hdfs安全模式：
```
# su hdfs -c "hdfs dfsadmin -safemode leave"
```
从本地向sandbox复制文件：
```
$ scp -P 2222 ~/Downloads/HDF-1.2.0.1-1.tar.gz root@localhost:/root
```
ambari口令重置：
```
# Updates password
ambari-admin-password-reset
# If Ambari doesn't restart automatically, restart ambari service
ambari-agent restart
```