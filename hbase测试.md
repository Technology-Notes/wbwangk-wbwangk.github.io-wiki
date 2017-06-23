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
创建一个文本文件hbase.csv，包含以下内容：
```
id, temp:in,temp:out,vibration,pressure:in,pressure:out
5842,  50,     30,       4,      240,         340
```
把该文件上传到HDFS中：
```
$ hdfs dfs -copyFromLocal hbase.csv /tmp
```
执行Loadtsv语句
```
$ hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=,  -Dimporttsv.columns="HBASE_ROW_KEY,id,temp:in,temp:out,vibration,pressure:in,pressure:out" sensor hdfs://u1401.ambari.apache.org:/tmp/hbase.csv
```

## Connecting to HBase in a Kerberos Enabled Cluster
[原文](https://community.hortonworks.com/articles/48831/connecting-to-hbase-in-a-kerberos-enabled-cluster.html)  
讲解如何通过Java或Scala在启用Kerberos的群集中连接到HBase。  
本测试需要一个启用了kerberos的HDP集群。首先，下载java样例代码：
```
$ cd /opt
$ git clone https://github.com/jjmeyer0/hdp-test-examples
```
【创建keytab】  
用管理员账号登录KDC，然后创建叫webb的主体，并导出keytab：
```
$ kinit root/admin@AMBARI.APACHE.ORG
$ kadmin -q "addprinc webb"         (创建webb主体，需要输入两次密码，密码是1)
$ ktutil
ktutil:  addent -password -p webb -k 1 -e RC4-HMAC
Password for webb@AMBARI.APACHE.ORG: 1
ktutil:  wkt webb.keytab
ktutil:  q
$ scp webb.keytab root@u1403:/etc/security/keytabs/             (把生成的keytab复制到hbase节点)
```
【准备hbase用户】  
回到hbase所在的u1403节点。webb用户必须在HBase中获得正确的权限。要做到这一点，请执行以下操作：
```
$ kinit hbase-hdp1               (实测只能用这个主体登录，即使root/admin主体都不行)
$ hbase shell
hbase(main):001:0> grant 'webb','RW'
```
【准备需要的文件】  
运行例子需要的文件有三个：
- hbase-site.xml            
- <username>.keytab
- krb5.conf
由于使用安装hbase的节点u1403充当运行例子的节点，上述三个文件可以可以直接在本节点上复制：
```
$ cp /etc/hbase/conf/hbase-site.xml /opt/htp-test-examples/src/main/resources/
$ cp /etc/security/keytabs/webb.keytab /opt/htp-test-examples/src/main/resources/
$ cp /etc/krb5.conf /opt/htp-test-examples/src/main/resources/
```
对于测试，建议在hbase-site.xml中更改“hbase.client.retries.number”属性。默认情况下为35.这在运行某些测试时相当高。  
【代码运行】
例子的java代码位于```src/main/java/com/jj/hbase/HBaseClient.java```。在代码中，首先需要做的是创建和加载HBase配置：
```
// Setting up the HBase configuration
Configuration configuration = new Configuration();
configuration.addResource("src/main/resources/hbase-site.xml");
```
接下来指向krb5.conf文件并设置kerberos主体和keytab。
```
// Point to the krb5.conf file.
System.setProperty("java.security.krb5.conf", "src/main/resources/krb5.conf");
System.setProperty("sun.security.krb5.debug", "true");
 
// Override these values by setting -DkerberosPrincipal and/or -DkerberosKeytab
String principal = System.getProperty("kerberosPrincipal", "webb@AMBARI.APACHE.ORG");
String keytabLocation = System.getProperty("kerberosKeytab", "src/main/resources/webb.keytab");
```
现在使用上面定义的主键和keytab登录。
```
UserGroupInformation.setConfiguration(configuration);
UserGroupInformation.loginUserFromKeytab(principal, keytabLocation)
```
用maven构建：
```
$ cd /opt/hdp-test-examples
$ mvn clean test -P hdp-2.4.2    (耗时较长)
Tests in error:
  calculatingBuildingAverageShouldProperlyStoreAverage(com.jj.pig.BuildingAvgPigTest): Unable to open iterator for alias building_avg
  makeSureTestDataFromFileProperlyProducesAverage(com.jj.pig.BuildingAvgPigTest): Unable to open iterator for alias building_avg
....
[ERROR] Please refer to /opt/hdp-test-examples/target/surefire-reports for the individual test results.
[ERROR] -> [Help 1]
org.apache.maven.lifecycle.LifecycleExecutionException: Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.10:test (default-test) on project hdp-test-examples: There are test failures.

Please refer to /opt/hdp-test-examples/target/surefire-reports for the individual test results.
....
```
（等以后再查找问题所在！！！）
这里是[完整代码](https://github.com/jjmeyer0/hdp-test-examples)  

## HOWTO: Start and test HBase REST server in a kerberized environment
[原文](https://community.hortonworks.com/articles/91425/howto-start-and-test-hbase-rest-server-in-a-kerber.html)  
先确认你的HDP集群启用了kerberos并安装了hbase。  
检查一下你的hbase服务的keytab，我的hbase服务安装在u1403节点上，所以在u1403节点上执行：
```
$ klist -kt /etc/security/keytabs/hbase.service.keytab
Keytab name: FILE:/etc/security/keytabs/hbase.service.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   1 06/20/2017 14:45:57 hbase/u1403.ambari.apache.org@AMBARI.APACHE.ORG    (下略)
```
在Ambari中前往 Hbase -> Configs -> Advanced -> Custom Hbase-Site，增加或修改下列参数：
```
hbase.master.keytab.file=/etc/security/keytabs/hbase.service.keytab
hbase.master.kerberos.principal=hbase/_HOST@AMBARI.APACHE.ORG
hbase.rest.authentication.type=kerberos
hadoop.proxyuser.HTTP.groups=*
hadoop.proxyuser.HTTP.hosts=*
hbase.security.authorization=true
hbase.rest.authentication.kerberos.keytab=/etc/security/keytabs/spnego.service.keytab
hbase.rest.authentication.kerberos.principal=HTTP/_HOST@AMBARI.APACHE.ORG
hbase.security.authentication=kerberos
hbase.rest.kerberos.principal=hbase/_HOST@AMBARI.APACHE.ORG
hbase.rest.keytab.file=/etc/security/keytabs/hbase.service.keytab
```
还可以增加两个可选参数来定义Rest服务的端口：
```
hbase.rest.port = 17000
hbase.rest.info.port = 17050
```
在Ambari中前往HDFS -> Configs -> Advanced -> Custom core-site，增加或修改下列参数：
```
hadoop.proxyuser.HTTP.groups=*
hadoop.proxyuser.HTTP.hosts=*
```
重启受影响的HBase和HDFS服务。  
在安装HBase Master的节点(u1403)上执行：
```
$ su - hbase                        (切换为hbase用户)
$ kinit -kt hbase.service.keytab hbase/u1403.ambari.apache.org@AMBARI.APACHE.ORG
$ /usr/hdp/current/hbase-master/bin/hbase-daemon.sh start rest -p 17000 --infoport 17050       (启动REST服务器)
```
分别在无票据和有票据的情况下，测试REST服务器：
```
$ kdestroy
$ klist
klist: No credentials cache found (ticket cache FILE:/tmp/krb5cc_0)
 
$ curl --negotiate -u : 'http://u1403.ambari.apache.org:17000/status/cluster'
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1"/>
<title>Error 401 Authentication required</title>
 
# kinit -kt /etc/security/keytabs/hbase.service.keytab hbase/u1403.ambari.apache.org@AMBARI.APACHE.ORG
# curl --negotiate -u : 'http://u1403.ambari.apache.org:17000/status/cluster'
3 live servers, 0 dead servers, 10.6667 average load
 
3 live servers
    hdp253k1.hdp:16020 1490688381983
        requests=0, regions=11
        heapSizeMB=120        maxHeapSizeMB=502
```
测试过程中层出现hbase RegionServer启动失败的情况，导致curl调用超时。从日志`hbase-hbase-master-u1403.log`上看，报告`Clock skew too great`，推测是三个节点的时间不一致，导致kerberos票据失效。调整了三个节点的时间([参考](https://github.com/wbwangk/wbwangk.github.io/wiki/0%E7%AC%94%E8%AE%B0#%E6%97%B6%E9%97%B4%E5%90%8C%E6%AD%A5ntpd))。重启所有HDP服务，curl终于正确返回结果了。  