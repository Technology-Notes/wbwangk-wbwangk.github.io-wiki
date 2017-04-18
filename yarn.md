基于[大数据本地开发环境](https://github.com/imaidev/imaidev.github.io/wiki/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%9C%AC%E5%9C%B0%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)的环境。  
yarn通过ambari装在u1403上。下文中域名u1403.ambari.apache.org的IP地址是192.168.14.103。可以在C:\Windows\System32\drivers\etc\hosts中添加：
```
192.168.14.103 u1403.ambari.apache.org
```
（参考了书《hadoop yarn 权威指南》Arun C. Murthy等著，下称参考书）。  

在u1403上执行mapreduce的例子：
```
$ yarn jar /usr/hdp/2.5.3.0-37/hadoop-mapreduce/hadoop-mapreduce-examples.jar pi 16 1000
Number of Maps  = 16
Samples per Map = 1000
（然后提示没有权限：）
 Permission denied: user=yarn, access=WRITE, inode="/user/yarn/QuasiMonteCarlo_1490724309339_665171043/in":hdfs:hdfs:drwxr-xr-x
```
切换用户后执行：
```
$ sudo su - hdfs  (切换用户为hdfs)
$ yarn jar /usr/hdp/2.5.3.0-37/hadoop-mapreduce/hadoop-mapreduce-examples.jar pi 16 1000
Number of Maps  = 16
Samples per Map = 1000
Wrote input for Map #0
Wrote input for Map #1
Wrote input for Map #2
Wrote input for Map #3
Wrote input for Map #4
Wrote input for Map #5
Wrote input for Map #6
Wrote input for Map #7
Wrote input for Map #8
Wrote input for Map #9
Wrote input for Map #10
Wrote input for Map #11
Wrote input for Map #12
Wrote input for Map #13
Wrote input for Map #14
Wrote input for Map #15
Starting Job
17/03/28 18:06:40 INFO impl.TimelineClientImpl: Timeline service address: http://u1403.ambari.apache.org:8188/ws/v1/timeline/
17/03/28 18:06:40 INFO client.RMProxy: Connecting to ResourceManager at u1403.ambari.apache.org/192.168.14.103:8050
17/03/28 18:06:45 INFO client.AHSProxy: Connecting to Application History server at u1403.ambari.apache.org/192.168.14.103:10200
17/03/28 18:06:51 INFO input.FileInputFormat: Total input paths to process : 16
17/03/28 18:06:52 INFO mapreduce.JobSubmitter: number of splits:16
17/03/28 18:06:53 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1490716389565_0001
17/03/28 18:06:59 INFO impl.YarnClientImpl: Submitted application application_1490716389565_0001
17/03/28 18:07:01 INFO mapreduce.Job: The url to track the job: http://u1403.ambari.apache.org:8088/proxy/application_1490716389565_0001/
17/03/28 18:07:01 INFO mapreduce.Job: Running job: job_1490716389565_0001
17/03/28 18:11:22 INFO mapreduce.Job: Job job_1490716389565_0001 running in uber mode : false
17/03/28 18:11:22 INFO mapreduce.Job:  map 0% reduce 0%
17/03/28 18:11:50 INFO mapreduce.Job:  map 13% reduce 0%
17/03/28 18:12:00 INFO mapreduce.Job:  map 25% reduce 0%
17/03/28 18:12:09 INFO mapreduce.Job:  map 38% reduce 0%
17/03/28 18:12:18 INFO mapreduce.Job:  map 50% reduce 0%
17/03/28 18:12:27 INFO mapreduce.Job:  map 63% reduce 0%
17/03/28 18:12:35 INFO mapreduce.Job:  map 75% reduce 0%
17/03/28 18:12:45 INFO mapreduce.Job:  map 88% reduce 0%
17/03/28 18:12:53 INFO mapreduce.Job:  map 94% reduce 0%
17/03/28 18:12:54 INFO mapreduce.Job:  map 100% reduce 0%
17/03/28 18:13:00 INFO mapreduce.Job:  map 100% reduce 100%
17/03/28 18:13:01 INFO mapreduce.Job: Job job_1490716389565_0001 completed successfully
17/03/28 18:13:01 INFO mapreduce.Job: Counters: 49
        File System Counters
                FILE: Number of bytes read=358
                FILE: Number of bytes written=2420877
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=4438
                HDFS: Number of bytes written=215
                HDFS: Number of read operations=67
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=3
        Job Counters
                Launched map tasks=16
                Launched reduce tasks=1
                Data-local map tasks=16
                Total time spent by all maps in occupied slots (ms)=165113
                Total time spent by all reduces in occupied slots (ms)=10586
                Total time spent by all map tasks (ms)=165113
                Total time spent by all reduce tasks (ms)=5293
                Total vcore-milliseconds taken by all map tasks=165113
                Total vcore-milliseconds taken by all reduce tasks=5293
                Total megabyte-milliseconds taken by all map tasks=28069210
                Total megabyte-milliseconds taken by all reduce tasks=1799620
        Map-Reduce Framework
                Map input records=16
                Map output records=32
                Map output bytes=288
                Map output materialized bytes=448
                Input split bytes=2550
                Combine input records=0
                Combine output records=0
                Reduce input groups=2
                Reduce shuffle bytes=448
                Reduce input records=32
                Reduce output records=0
                Spilled Records=64
                Shuffled Maps =16
                Failed Shuffles=0
                Merged Map outputs=16
                GC time elapsed (ms)=3979
                CPU time spent (ms)=6430
                Physical memory (bytes) snapshot=3256246272
                Virtual memory (bytes) snapshot=30993297408
                Total committed heap usage (bytes)=2117926912
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters
                Bytes Read=1888
        File Output Format Counters
                Bytes Written=97
Job Finished in 382.815 seconds
Estimated value of Pi is 3.14250000000000000000
```
实测中，作业不是马上就执行的。通过浏览器查看：
```
http://u1403.ambari.apache.org:8088/cluster
```
当作业执行成功后，点击屏幕上History链接可以看到作业的历史摘要。  
### 安装YARN
安装的主力是一个开源工具pdsh([parallel distributed shell](http://sourceforge.net/projects/pdsh))。pdsh能远程地在主机上执行命令行或文件中命令。pdsh发行版中包含的pdcp命令能分发复制文件。（参考书的50页）  
 1. 安装pdsh 
```
$ apt install pdsh
```
 2. 免密码ssh
参考[SSH入门](https://github.com/wbwangk/wbwangk.github.io/wiki/SSH%E5%85%A5%E9%97%A8)。  
 3. 使用pdsh
测试pdsh：
```
$ pdsh -w ssh:u1402 hostname
u1402: u1402
```
为了同时在多台机器上执行命令，首先创建一个all_hosts的文件：
```
u1402
u1403
```
然后执行：
```
$ pdsh -R ssh -w ^all_hosts date    (-R指定rcmd模块为ssh)
u1403: Wed Mar 29 01:13:35 UTC 2017
u1402: Wed Mar 29 01:05:26 UTC 2017
```
上述结果显示：date命令在u1402和u1403两个主机上都执行成功了。也就是说，pdsh可以批量在多个主机上执行命令。  