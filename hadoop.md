
# hadoop体系
官网:http://hadoop.apache.org/，还有个hadoop的[中文文档](http://hadoop-learning-notes.readthedocs.io/en/latest/)。  

主要模块:  
 - Hadoop Common: 支撑其他模块的通用工具  
 - Hadoop Distributed File System (HDFS™): 分布式文件系统  
 - Hadoop YARN: 一个作业调度框架和集群管理  
 - Hadoop MapReduce: 一个基于YARN的大数据集并行处理系统  
其他apche相关模块：  
 - Ambari™: 一个供应、管理和监控apache hadoop集群的web界面工具，支持Hadoop HDFS, Hadoop MapReduce, Hive, HCatalog, HBase, ZooKeeper, Oozie, Pig and Sqoop。Ambari还提供了面板显示集群健康状况，如heatmaps and ability to view MapReduce, Pig and Hive applications visually alongwith features to diagnose their performance characteristics in a user-friendly manner.  
 - Avro™: 一个数据序列化系统。  
 - Cassandra™: 一个无单点失效的可伸缩多主数据库。  
 - Chukwa™: 一个为了管理大型分布式系统的数据收集系统。  
 - HBase™: 一个支持大表结构化数据存储的可伸缩分布式数据库。  
 - Hive™: 一个提供数据汇总和ad hoc查询的数据仓库基础设施。  
 - Mahout™: 一个可伸缩机器学习和数据挖掘库。  
 - Pig™: 一个为了并行计算的高级数据流语言和执行框架。  
 - Spark™: 一个针对hadoop数据的高速和通用计算引擎。Spark provides a simple and expressive programming model that supports a wide range of applications, including ETL, machine learning, stream processing, and graph computation.  
 - Tez™: 一个通用的数据流编程框架。构建在YARN之上，which provides a powerful and flexible engine to execute an arbitrary DAG of tasks to process data for both batch and interactive use-cases. Tez is being adopted by Hive™, Pig™ and other frameworks in the Hadoop ecosystem, and also by other commercial software (e.g. ETL tools), to replace Hadoop™ MapReduce as the underlying execution engine.  
 - ZooKeeper™: 一个针对分布式应用的高性能协调服务。  
## 安装单机版hadoop
[原始apache文档](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html)  
HDFS: namenode 存放文件系统元数据；datanode存放文件。  

安装jdk:
``` 
apt-get install openjdk-8-jdk
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
```
[下载](http://apache.fayea.com/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz)和解压hadoop2.7.3到/opt/hadoop-2.7.3目录下。测试一下：
```
cd /opt/hadoop-2.7.3
bin/hadoop
```
单节点运行hadoop有两种模式：
 - Local (Standalone) Mode  
 - Pseudo-Distributed Mode  

###Standalone运行
默认情况下，hadoop运行在非分布式模式下，用于调试：
```
  $ mkdir input
  $ cp etc/hadoop/*.xml input
  $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep input output 'dfs[a-z.]+'
  $ cat output/*
```
上述代码扫码input目录，创建并输出到了output目录下。

###伪分布式运行
编辑配置文件etc/hadoop/core-site.xml:
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```
etc/hadoop/hdfs-site.xml:
```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```
hadoop的伪分布模式运行时需要SSH到localhost。下面的操作使ssh可以访问localhost，有两种方式：
```
$ ssh-copy-id localhost   （方式1：ssh原生的方式）
$ cat ~/.ssh/id_dsa.pub >> ~/.ssh/authorized_keys    （方式2：直接操作文本文件）
```
可以通过查看 ~/.ssh/authorized_keys文件中的内容来了解ssh授权的原理，就把信任节点的公钥放入了authorized_keys而已。
测试一下：
```
$ ssh localhost  或者ssh root@localhost
```
发现不再提示输入密码。执行exit返回到原来的上下文。关于SSH的其他知识可参考[SSH入门](SSH入门)。

格式化HDFS:
```
bin/hdfs namenode -format
```
编辑$HADOOP_PREFIX/etc/hadoop目录下的hadoop-env.sh，设置JAVA_HOME:
```
# export JAVA_HOME=${JAVA_HOME}
export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
```
启动NameNode和DataNode两个守护进程：
```
$ sbin/start-dfs.sh
$ curl http://localhost:50070/   (测试一下NameNode的web接口，也可以用浏览器访问这个web接口)
```
在HDFS上创建执行MapReduce作业需要的目录：
```
$ bin/hdfs dfs -mkdir /user
$ bin/hdfs dfs -mkdir /user/root     (root是当前用户)
```
将输入从本地文件系统复制到HDFS中：
```
$ bin/hdfs dfs -put etc/hadoop input      (在HDFS中的绝对地址是/user/root/input)
```
执行例子的MapReduce：
```
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep input output 'dfs[a-z.]+'
```
输出是hfds中的/user/root/output目录。  

有两种方式查看输出结果。首先，是将文件从HDFS复制到本地文件系统：
```
$ bin/hdfs dfs -get output output
$ cat output/*
```
直接查看HDFS中结果：
```
$ bin/hdfs dfs -cat output/*
```
最后，停止NameNode和DataNode：  
```
$ sbin/stop-dfs.sh
```

## 集群安装

需要在集群的所有节点上下载和解压hadoop的二进制包（200多M）。这里使用2.7.3版本（稳定版），下载路径参考上文的单节点安装hadoop。

NameNode和ResourceManager各占一台机器，这是**主节点**。而其他服务根据负载情况，可能运行在专用硬件上，也可能运行在共享基础设施上。  
集群中的剩余机器可充当 DataNode和NodeManager，这些是**从节点**。  

HDFS daemons由NameNode, SecondaryNameNode,和DataNode组成。 YARN damones由ResourceManager, NodeManager,和WebAppProxy组成. 如果使用MapReduce，那么MapReduce Job History Server也需要运行。大规模集群下，通常都运行在专门的服务器上。

使用```bento/ubuntu-16.10```这个vagrant box创建了3个VM，分别是big1、big2、big3。计划使用big1充当NameNode。  
使用[SSH入门](SSH入门)中的方法，使上述三个节点之间可以互相免密码SSH（3个节点共执行了9次ssh-copy-id，包括localhost）。  

#### 配置Hadoop Daemons运行环境
在/etc/profile.d/目录下建立一个my-hadoop.sh的文件（该目录下的脚本会在linux启动后自动执行）：
```
JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export JAVA_HOME
HADOOP_PREFIX=/opt/hadoop-2.7.3
export HADOOP_PREFIX
HADOOP_CONF_DIR=$HADOOP_PREFIX/etc/hadoop
export HADOOP_CONF_DIR
```
