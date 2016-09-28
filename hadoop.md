[原始apache文档](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html)

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