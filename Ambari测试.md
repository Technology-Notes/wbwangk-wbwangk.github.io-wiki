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

###安装Ambari Agent
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