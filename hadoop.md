
# hadoop体系
官网:http://hadoop.apache.org/  
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
## 集群安装
选择了stable版本2.7.3，下载了一个200多兆的tar.gz包。按要求，先装openjdk-8-jdk。  
NameNode和ResourceManager各占一台机器，这是**主节点**。而其他服务根据负载情况，可能运行在专用硬件上，也可能运行在共享基础设施上。  
集群中的剩余机器可充当 DataNode和NodeManager，这些是**从节点**。  
各个节点都要设置JAVA_HOME:
```
$ export JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
```
环境变量HADOOP_PREFIX定义了hadoop的安装目录:
```
$ export HADOOP_PREFIX="/opt/hadoop-2.7.3"
$ export HADOOP_CONF_DIR="$HADOOP_PREFIX/etc/hadoop"
```
## 安装单机版hadoop
[原始apache文档](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html)  
HDFS: namenode 存放文件系统元数据；datanode存放文件。  

安装oracle java7：
``` 
apt-get install oracle-java7-installer
```
java的home目录：/usr/lib/jvm/java-7-oracle/jre

[下载](http://apache.fayea.com/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz)和解压hadoop2.7.3到/opt/hadoop-2.7.3目录下，这是hadoop的根目录。然后编辑 etc/hadoop/hadoop-env.sh
```
  # set to the root of your Java installation
  export JAVA_HOME="/usr/lib/jvm/java-7-oracle/jre"
```
设置hadoop的home：
```
$ export HADOOP_COMMON_HOME="/opt/hadoop-2.7.3"
```
测试hadoop：
```
  $ mkdir input
  $ cp etc/hadoop/*.xml input
  $ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar grep input output 'dfs[a-z.]+'
  $ cat output/*
```