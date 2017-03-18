[原始文档地址](https://cwiki.apache.org/confluence/display/AMBARI/Quick+Start+Guide)  

### 安装Ambari Server
在ubuntu12下安装：
```
# to test the trunk build - updated on every commit to trunk
wget -O /etc/apt/sources.list.d/ambari.list http://s3.amazonaws.com/dev.hortonworks.com/ambari/ubuntu12/2.x/latest/trunk/ambaribn.list
apt-key adv --recv-keys --keyserver keyserver.ubuntu.com B9733A7A07513CAD
apt-get update
apt-get install ambari-server -y
```
如果在ubuntu14下安装把URL中替换成```ubuntu14```。我直接在ubuntu16下使用上述ubuntu12的URL也安装成功了。

初始化Ambari Server：
```
$ ambari-server setup
```
初始化过程让选择JDK并设定JAVA_HOME。之前已经装好了openjdk-8-jdk。输入了JAVA_HOME:
```
Path to JAVA_HOME: /usr/lib/jvm/java-8-openjdk-amd64
```
启动Ambari Server:
```
$ ambari-server start
$ curl localhost:8080        (至少要几分钟后8080端口才有响应)
```
安装Ambari Server的vagrant VM的hostname是big1，IP是10.10.11.95。当```curl localhost:8080```有了响应后，直接在windows下用浏览器访问地址：```http://10.10.11.95:8080```出现登录页面，用户名口令是admin/admin。

### 安装Ambari Agent
需要在集群的各个节点上安装Ambari Agent。做测试时使用了vagrant的两个VM：
```
big1 10.10.11.96
big2 10.10.11.97
```
先添加Apache Ambari源，再安装Agent。：
```
# to test the trunk build - updated on every commit to trunk
wget -O /etc/apt/sources.list.d/ambari.list http://s3.amazonaws.com/dev.hortonworks.com/ambari/ubuntu12/2.x/latest/trunk/ambaribn.list
apt-key adv --recv-keys --keyserver keyserver.ubuntu.com B9733A7A07513CAD
apt-get update
apt-get install ambari-agent -y
```
编辑各个节点的agent配置文件/etc/ambari-agent/conf/ambari-agent.ini:
```
...
[server]
hostname=big1
...
```
将hostname配置为正确的Ambari Server地址，在本测试中Ambari Server位于big1虚机上。启动Agent：
```
$ ambari-agent start
```
### 通过Ambari部署hadoop
计划使用4台vagrant VM进行部署测试，big1/big2/big3/big4，其中Ambari Server在big1上启动。4台VM上需要安装的软件有：  
```
$ ssh-copy-id root@big2 && ssh-copy-id root@big3 && ssh-copy-id root@big4    (让big1可以免密码SSH到其他VM)
$ apt install ntp && service ntp start    (时间同步)
$ apt install openjdk-8-jdk
$ apt install ambari-agent && ambari-agent start
```
免密码SSH的方法参考[SSH入门](SSH入门)。  
各VM上Ambari Agent的配置文件要修改，设置hostname=big1。  

## Ambari安装的清理
[参考](http://blog.csdn.net/wk022/article/details/49278419)  
ambari安装hadoop，有时会安装失败，需要卸载已经安装包。  
1，通过ambari将集群中的所用组件都关闭，如果关闭不了，直接kill -9 XXX  
2，关闭ambari-server，ambari-agent：
```
$ ambari-server stop  
$ ambari-agent stop  
```
3，卸载安装的软件  
```
$ apt remove hadoop_2* hdp-select* ranger_2* zookeeper* bigtop* atlas-metadata* ambari* postgresql spark*  
```
以上命令可能不全，执行完一下命令后，再执行:
```
$ apt list | grep @HDP
```
查看是否还有没有卸载的，如果有，继续通过```$apt remove XXX```卸载。  

4，删除postgresql的数据  
      postgresql软件卸载后，其数据还保留在硬盘中，需要把这部分数据删除掉，如果不删除掉，重新安装ambari-server后，有可能还应用以前的安装数据，而这些数据时错误数据，所以需要删除掉。  
```
$ rm -rf /var/lib/pgsql  
```
5，删除用户
     ambari安装hadoop集群会创建一些用户，清除集群时有必要清除这些用户，并删除对应的文件夹。这样做可以避免集群运行时出现的文件访问权限错误的问题。     
```
userdel oozie  
userdel hive  
userdel ambari-qa  
userdel flume    
userdel hdfs    
userdel knox    
userdel storm    
userdel mapred  
userdel hbase    
userdel tez    
userdel zookeeper  
userdel kafka    
userdel falcon  
userdel sqoop    
userdel yarn    
userdel hcat  
userdel atlas  
userdel spark  
rm -rf /home/atlas  
rm -rf /home/accumulo  
rm -rf /home/hbase  
rm -rf /home/hive  
rm -rf /home/oozie  
rm -rf /home/storm  
rm -rf /home/yarn  
rm -rf /home/ambari-qa  
rm -rf /home/falcon  
rm -rf /home/hcat  
rm -rf /home/kafka  
rm -rf /home/mahout  
rm -rf /home/spark  
rm -rf /home/tez  
rm -rf /home/zookeeper  
rm -rf /home/flume  
rm -rf /home/hdfs  
rm -rf /home/knox  
rm -rf /home/mapred  
rm -rf /home/sqoop  
```
6，删除ambari遗留数据
```
rm -rf /var/lib/ambari*  
rm -rf /usr/lib/python2.6/site-packages/ambari_*  
rm -rf /usr/lib/python2.6/site-packages/resource_management  
rm -rf /usr/lib/ambri-*  
```
7，删除其他hadoop组件遗留数据
```
rm -rf /etc/hadoop  
rm -rf /etc/hbase  
rm -rf /etc/hive   
rm -rf /etc/oozie  
rm -rf /etc/sqoop   
rm -rf /etc/zookeeper  
rm -rf /etc/flume   
rm -rf /etc/storm   
rm -rf /etc/hive-hcatalog  
rm -rf /etc/tez   
rm -rf /etc/falcon   
rm -rf /etc/knox   
rm -rf /etc/hive-webhcat  
rm -rf /etc/kafka   
rm -rf /etc/slider   
rm -rf /etc/storm-slider-client  
rm -rf /etc/spark   
rm -rf /var/run/spark  
rm -rf /var/run/hadoop  
rm -rf /var/run/hbase  
rm -rf /var/run/zookeeper  
rm -rf /var/run/flume  
rm -rf /var/run/storm  
rm -rf /var/run/webhcat  
rm -rf /var/run/hadoop-yarn  
rm -rf /var/run/hadoop-mapreduce  
rm -rf /var/run/kafka  
rm -rf /var/log/hadoop  
rm -rf /var/log/hbase  
rm -rf /var/log/flume  
rm -rf /var/log/storm  
rm -rf /var/log/hadoop-yarn  
rm -rf /var/log/hadoop-mapreduce  
rm -rf /var/log/knox   
rm -rf /usr/lib/flume  
rm -rf /usr/lib/storm  
rm -rf /var/lib/hive   
rm -rf /var/lib/oozie  
rm -rf /var/lib/flume  
rm -rf /var/lib/hadoop-hdfs  
rm -rf /var/lib/knox   
rm -rf /var/log/hive   
rm -rf /var/log/oozie  
rm -rf /var/log/zookeeper  
rm -rf /var/log/falcon  
rm -rf /var/log/webhcat  
rm -rf /var/log/spark  
rm -rf /var/tmp/oozie  
rm -rf /tmp/ambari-qa  
rm -rf /var/hadoop  
rm -rf /hadoop/falcon  
rm -rf /tmp/hadoop   
rm -rf /tmp/hadoop-hdfs  
rm -rf /usr/hdp  
rm -rf /usr/hadoop  
rm -rf /opt/hadoop  
rm -rf /tmp/hadoop  
rm -rf /var/hadoop  
rm -rf /hadoop  
```
8，清理apt数据源
```
$ apt clean all  
```
通过以上清理后，重新安装ambari和hadoop集群（包括HDFS，YARN+MapReduce2，Zookeeper，Ambari Metrics，Spark）成功。如果安装其他组件碰到由于未清理彻底而导致的问题，请留言指出需要清理的数据，本人会补全该文档。

## Ambari的汉化
Apache Ambari在github的镜像库：https://github.com/apache/ambari。  
一个Ambari的汉化库：https://github.com/yantaiv/Ambari-Web-Modify。 
一个汉化的说明：[链接](http://blog.csdn.net/wang1472jian1110/article/details/50803887)。 

## Ambari本地源
[参考原文](https://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/setting_up_a_local_repository.html)  

我在vagrant VM ubuntu16下建立“无互联网模式HDP本地源”，首先安装nginx：
```
$ apt install nginx
$ cd /var/www/html/
```
下载[Ambari barball](https://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/ambari_repositories.html)  
上述页面中，Base URL是库的基础地址，如ubuntu的/etc/apt/sources.list文件中：
```
deb http://security.ubuntu.com/ubuntu yakkety-security universe
```
其中的```http://security.ubuntu.com/ubuntu```就是Base URL。在base url之下是```dists```目录。而```yakkety-security```是dists的下级目录，依次类推（空格隔开的多级目录）。   
下载ubuntu14的Armbri barball：
```
$ cd /var/www/html
$ wget http://public-repo-1.hortonworks.com/ambari/ubuntu14/2.x/updates/2.4.2.0/ambari-2.4.2.0-ubuntu14.tar.gz
$ openssl md5 ambari-2.4.2.0-ubuntu14.tar.gz   (计算Tarball的MD5码，应与上述网页上公布的一样)
$ tar -xzf ambari-2.4.2.0-ubuntu14.tar.gz
$ cd AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136 && ls -l          (可以看到下级的dists目录以及ambaribn.list)
$ cat ambaribn.list        (一会儿通过curl+nginx访问这个文件)
$ curl localhost/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136/ambaribn.list
```
如果curl返回ambaribn.list的文件内容，说明用nginx搭建的apt本地源运行正常。其中```localhost/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136/```就是Base URL。  
下面创建本地源的库配置文件。首先下载一个原始的库配置文件（上面的Ambari barball下载页面上有链接）：
```
$ cd /etc/apt/sources.list.d
$ wget http://public-repo-1.hortonworks.com/ambari/ubuntu14/2.x/updates/2.4.2.0/ambari.list
$ cat ambari.list
#VERSION_NUMBER=2.4.2.0-136
deb http://public-repo-1.hortonworks.com/ambari/ubuntu14/2.x/updates/2.4.2.0 Ambari main
```
编辑这个库配置文件，替换其中的Base URL为本地的，最终的样子：
```
$ cat ambari.list
#VERSION_NUMBER=2.4.2.0-136
deb http://localhost/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136 Ambari main
```
由于使用了localhost当hostname，导致这个源只能在本机用。如果在其他机器上使用这个apt源，则需要把localhost替换为IP地址或域名。  
测试一下这个源：
```
$ apt-get update
Get:1 http://localhost/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136 Ambari InRelease [3,190 B]
Get:2 http://localhost/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136 Ambari/main amd64 Packages [1,383 B]
···
```

下载地址：[HDP 2.5 Stack Repositories](https://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/hdp_25_repositories.html)
下载这个tarball（4.9G）：
http://public-repo-1.hortonworks.com/HDP/ubuntu14/2.x/updates/2.5.3.0/HDP-2.5.3.0-ubuntu14-deb.tar.gz
