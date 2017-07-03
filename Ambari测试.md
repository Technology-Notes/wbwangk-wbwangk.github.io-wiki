[原始文档地址](http://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/ch_Installing_Ambari.htmle)  
相关文档：[大数据本地开发环境](https://github.com/imaidev/imaidev.github.io/wiki/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%9C%AC%E5%9C%B0%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)  
曾经尝试从[源代码](https://github.com/apache/ambari)构建Ambari，使用maven，耗时几个小时，但没有成功。  
使用官方apt源在ubuntu14下安装Ambari server的方法：
```
wget -O /etc/apt/sources.list.d/ambari.list http://s3.amazonaws.com/dev.hortonworks.com/ambari/ubuntu14/2.x/latest/trunk/ambaribn.list
apt-key adv --recv-keys --keyserver keyserver.ubuntu.com B9733A7A07513CAD
apt-get update
apt-get install ambari-server -y
```
当在内网部署hadoop时，有些主机没有互联网连接。针对这种情况，ambari的官方文档提供了搭建本地apt源的方法，通过本地源安装ambari（及其他hadoop组件）。  

### 搭建Ambari本地源
[参考原文](https://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/setting_up_a_local_repository.html)  
关于搭建HDP本地源，更详细文档在[这里](https://github.com/wbwangk/wbwangk.github.io/wiki/%E6%90%AD%E5%BB%BAHDP%E6%9C%AC%E5%9C%B0%E6%BA%90)。  
下列测试的目的是建立“无互联网模式HDP本地源”，环境是vagrant VM + ubuntu14。[官方文档](https://cwiki.apache.org/confluence/display/AMBARI/Quick+Start+Guide)提供了搭建测试环境的vagrant配置文件及相关脚本，下载该vagrant配置文件：
```
$ git clone https://github.com/u39kun/ambari-vagrant.git
```
上述命令我在windows10的git bash窗口中执行的。然后：
```
$ cd ambari-vagrant/ubuntu14.4
$ ./up.sh 4    (或者按下面的)
$ vagrant up u1401 u1402 u1403 u1404       (up.sh提供了更简单的启动方式)
```
看一下各个VM的/etc/hosts文件，发现各个节点的hostname已经定义了。  
【解释一下什么是Base URL】  
在网页[Ambari barball](https://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/ambari_repositories.html)中提到了Base URL。Base URL是库的基础地址，如ubuntu的/etc/apt/sources.list文件中：
```
deb http://security.ubuntu.com/ubuntu yakkety-security universe
```
其中的```http://security.ubuntu.com/ubuntu```就是Base URL。如果进入linux操作系统查看，发现在base url之下是```dists```目录，这应是apt打包系统的约定。而```yakkety-security```是dists的下级目录，依次类推（空格隔开的多级目录）。   

#### 开始建立Ambari的本地源
计划在u1401上建立apt本地源，首先安装nginx：
```
$ vagrant ssh u1401
$ apt install nginx
$ cd /var/www/html
```
```/var/www/html```是nginx的index.html所在的目录。可以编辑一下这个目录下的index.html(也可能是其他文件名)，然后用```curl localhost```测试一下。  
然后下载ubuntu14的Ambari barball（1.3G)，里面是Ambari apt打包文件：
```
$ wget http://public-repo-1.hortonworks.com/ambari/ubuntu14/2.x/updates/2.4.2.0/ambari-2.4.2.0-ubuntu14.tar.gz
$ openssl md5 ambari-2.4.2.0-ubuntu14.tar.gz   (计算Tarball的MD5码，应与上述网页上公布的一样。这一步式可选的)
$ tar -xzf ambari-2.4.2.0-ubuntu14.tar.gz
$ cd AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136          (ls可以看到下级的dists目录以，Base URL是dists之前的路径)
```
建立Amabiri本地源描述文件：
```
$ echo "deb http://$(hostname)/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136 Ambari main" > ambari.list
$ curl http://$(hostname)/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136/ambari.list   （测试一下）
```
如果curl返回ambari.list的文件内容，说明用nginx搭建的apt本地源运行正常。其中```http://$(hostname)/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136```就是Base URL。  

#### 用本地源安装ambari server
在同一台机器上：
```
$ echo "$(hostname)"
u1401     
$ apt-key adv --recv-keys --keyserver keyserver.ubuntu.com B9733A7A07513CAD
$ apt-get update
Get:1 http://u1401/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136 Ambari InRelease [3,190 B]
$ apt-get install ambari-server -y
$ ambari-server setup -s
$ ambari-server start
$ curl http://u1401:8080    (启动需要几分钟，用户名口令是admin/admin)
```
使用本地源安装ambari-server速度快多了。直接用互联网安装时300k/s，而本地源可以达10M/s。   

直接在宿主机windows下用浏览器访问地址：```http://u1401的IP:8080```出现登录页面，用户名口令是admin/admin。

## 创建HDP 2.5本地源
官方下载地址：[HDP 2.5 Stack Repositories](https://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/hdp_25_repositories.html)
下载这个tarball（4.9G）：
```
$ cd /var/www/html
$ mkdir hdp && cd hdp
$ wget http://public-repo-1.hortonworks.com/HDP/ubuntu14/2.x/updates/2.5.3.0/HDP-2.5.3.0-ubuntu14-deb.tar.gz
$ tar -xzf HDP-2.5.3.0-ubuntu14-deb.tar.gz
$ cd hdp/HDP/ubuntu14
$ echo "deb http://$(hostname)/hdp/HDP/ubuntu14 HDP main" > hdp.list
```
#### 同样的办法安装HDP-UTILS的本地源
这个tarball小多了，只有24M：
```
$ cd /var/www/html/hdp
$ wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.21/repos/ubuntu14/HDP-UTILS-1.1.0.21-ubuntu14.tar.gz
$ tar -xzf HDP-UTILS-1.1.0.21-ubuntu14.tar.gz
$ cd hdp/HDP-UTILS-1.1.0.21/repos/ubuntu14
$ echo "deb http://$(hostname)/hdp/HDP-UTILS-1.1.0.21/repos/ubuntu14 HDP-UTILS main" > hdp-utils.list
```
## 使用Ambari本地源搭建hadoop
计划使用u1402,u1403,u1404三个VM来尝试性部署hadoop。而u1401即是本地源的安装点，也是ambari server的安装点。  

### 准备环境
首先，实现免密码ssh登录，免密码SSH的方法参考[SSH入门](SSH入门)。在u1401上：  
```
$ ssh-keygen  (回车几次)
$ ssh-copy-id u1402   （输入yes并输入各VM的root密码，下同）
$ ssh-copy-id u1403
$ ssh-copy-id u1404
```
之前要参考[SSH入门](SSH入门)的说明在三个VM为root用户创建密码。然后在三个VM上分别安装：
```
$ apt install ntp -y 
$ service ntp start    (时间同步)
$ apt install openjdk-8-jdk  (也可安装oracle jdk 8)
$ cd /etc/apt/sources.list.d
$ wget http://u1401/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136/ambari.list
$ wget http://u1401/hdp/HDP/ubuntu14/hdp.list
$ wget http://u1401/hdp/HDP-UTILS-1.1.0.21/repos/ubuntu14/hdp-utils.list
$ apt-key adv --recv-keys --keyserver keyserver.ubuntu.com B9733A7A07513CAD
$ apt-get update
```
编辑三个节点的ambari agent配置文件/etc/ambari-agent/conf/ambari-agent.ini:
```
[server]
hostname=u1401
```
安装和启动Agent：
```
$ apt install ambari-agent
$ ambari-agent start
```
### 用ambari安装HDP
通过宿主机的浏览器进入http://u1401:8080，admin/admin登录。  
通过界面只能创建一个集群。通过REST API可以创建多个集群。IDAP提供了界面来部署多个集群。  

参考：Ambari日志的位置：
```
/var/log/ambari-server/ambari-server.log
```
## Ambari安装的清理
[参考](http://blog.csdn.net/wk022/article/details/49278419)  

## Ambari的汉化
Apache Ambari在github的镜像库：https://github.com/apache/ambari。  
一个Ambari的汉化库：https://github.com/yantaiv/Ambari-Web-Modify。 
一个汉化的说明：[链接](http://blog.csdn.net/wang1472jian1110/article/details/50803887)。 

## Ambari server的数据库
Ambari server默认安装了一个PostgreSQL数据库。启动postgresql进程的linux用户名是postgres，数据库名是ambari。数据库的默认用户名和密码是ambari/bigdata。   
如果要远程连接这个数据库，（比如在u1402上装ranger）远程连接这个数据库需要在u1402上这样执行：  
```
$ apt install postgresql-client
$ psql -h 192.168.14.101 -U ambari -d ambari  (提示输入密码就输入bigdata)
```
-U表示数据库用户，-d表示数据库名。  
还要修改postgresql的配置文件，以便从远程访问postgresql。
如果是在ubuntu下：
```
$ echo "host all all 0.0.0.0 0.0.0.0 md5" >> /etc/postgresql/9.3/main/pg_hba.conf
$ /etc/init.d/postgresql restart            （重启postgresql）
```
centos7下的操作：
```
$ echo "host all all 0.0.0.0 0.0.0.0 md5" >> /var/lib/pgsql/data/pg_hba.conf
$ service postgresql restart
```
而在postgresql所在机器的本地执行：
```
$ sudo -u postgres psql
postgres=# alter user postgres with password 'bigdata';
```
可以进入postgres的交互式命令行。postgres是启动数据库进程的linux用户。上述操作将postgres用户的密码改成了```bigdata```。在安装ranger的时候需要postgresql的管理员账户及密码。

安装ranger时要求ambari server配置jdbc驱动：
```
$ wget -P /usr/share/java https://jdbc.postgresql.org/download/postgresql-42.0.0.jar
$ ambari-server setup --jdbc-db=postgres --jdbc-driver=/usr/share/java/postgresql-42.0.0.jar
```

## Ambari Security
[Apache Ambari Security](http://docs.hortonworks.com/HDPDocuments/Ambari-2.5.0.3/bk_ambari-security/content/ch_amb_sec_guide.html)

#### 1.安装KDC Server
在u1404（非HDP集群节点）安装Install the KDC Server：
```
$ apt-get install krb5-kdc krb5-admin-server
```
第一次尝试时，提示krb5-user依赖冲突。用手机当热点执行apt-get update后，问题解决。更彻底的解决办法是利用163源替换ubuntu官方源，可参考[这个](https://github.com/imaidev/imaidev.github.io/wiki/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%9C%AC%E5%9C%B0%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83#ubuntu%E4%B8%8B%E7%9A%84%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)。  
安装过程中出现提示窗口让输入Default Kerberos version 5 realm，保留默认值AMBARI.APACHE.ORG。然后出现两次让输入hostname，都输入的"u1404.ambari.apache.org"。最后提示说这个向导没有自动建立一个kerberos realm，如果想建立就执行命令"krb5_newrealm"。相关说明在/usr/share/doc/krb5-kdc/README.KDC中。  
```
$ krb5_newrealm
master key name 'K/M@AMBARI.APACHE.ORG'
Enter KDC database master key:    (输入两次密码，密码是vagrant)
```
启动KDC server和KDC admin server：
```
$ service krb5-kdc restart                     （如果不执行krb5_newrealm就无法启动这个服务）
$ service krb5-admin-server restart            （如果不执行krb5_newrealm就无法启动这个服务）
```
#### 2.创建Kerberos Admin
通过创建admin主体来建立KDC admin：
```
$ kadmin.local -q "addprinc root/admin"
Enter password for principal "root/admin@AMBARI.APACHE.ORG":    (输入两次密码，密码是vagrant)
Principal "admin/admin@AMBARI.APACHE.ORG" created.
```
将刚创建的admin主体添加到KDC ACL中：
```
$ echo "*/admin@AMBARI.APACHE.ORG *" >> /etc/krb5kdc/kadm5.acl
$ service krb5-admin-server restart
```

#### 3.安装jce
(实测发现,不需要安装JCE也可以启用kerberos)。  
在u1401、u1402、u1403三个节点上安装JCE：
```
$ apt-get install software-properties-common -y
$ add-apt-repository ppa:webupd8team/java    （如果出现提示，回车继续）
$ apt-get update
$ apt-get install oracle-java8-installer   （这个貌似不用装，因为ambari自己装了oracle jdk）
$ apt install oracle-java8-unlimited-jce-policy  （有弹窗让接受协议）
$ export JAVA_HOME=/usr/lib/jvm/java-8-oracle
```
在u1401上（ambari server所在节点）按[这个说明](http://docs.hortonworks.com/HDPDocuments/Ambari-2.5.0.3/bk_ambari-security/content/distribute_and_install_the_jce.html)下载JCE策略文件，并解压到```$JAVA_HOME/jre/lib/security/```目录下。  
重启ambarserver。  

#### 4.在ambari中启动kerberos安装向导
由于已经在u1404上部署了KDC，所有在向导中选择“Existing MIT KDC”（已经存在的MIT KDC）。  
KDC hosts中输入"u1404.ambari.apache.org"，在realm中输入"AMBARI.APACHE.ORG"。点击test KDC connection按钮，应显示连接成功。  
Kadmin host中输入"u1404.ambari.apache.org"，在Admin principal中输入"root/admin@AMBARI.APACHE.ORG"，输入密码，点击next按钮。  
页面切换到了“Install and Test Kerberos Client”，并开始安装kerberos client。  
过了一段时间后报错，发现是版本依赖（见下文）的问题。切换为用手机热点上网（公司网更新ubuntu14会有部分包报错），然后在u1402和u1403上执行apt-get update。点击“Retry”按钮继续向导。  
安装完成kerberos client后执行测试，但测试出错，说kadmin找不到。猜测是u1401节点未安装kerberos客户端导致，所以在u1401节点上执行：
```
$ apt install krb5-user  (如果提示包依赖错误，就用手机上网执行apt-get udpate)
```
安装后问题解决，继续执行向导。实测中碰到服务重启失败的情况，强行结束向导，然后手工重启有问题的hadoop服务。  

查看主体清单的方法：在u1404节点（安装KDC的节点）上执行：
```
$ kadmin.local
kadmin.local: list_principals  
HTTP/u1402.ambari.apache.org@AMBARI.APACHE.ORG
HTTP/u1403.ambari.apache.org@AMBARI.APACHE.ORG
K/M@AMBARI.APACHE.ORG
admin/admin@AMBARI.APACHE.ORG
(略)
```

#### 安装kerberos碰到的问题
ambari启用kerberos后所有服务要重启，其中datanode服务报告内存溢出。将datanode所在VM的内存由2048改成3048后，datanode服务重启成功。  

某次安装kerberos客户端时报错：
```
$  apt-get install krb5-user
The following packages have unmet dependencies:
 krb5-user : Depends: libkrb5-3 (= 1.12+dfsg-2ubuntu5.2) but 1.12+dfsg-2ubuntu5.3 is to be installed
E: Unable to correct problems, you have held broken packages.
```
krb5-user依赖libkrb5-3(= 1.12+dfsg-2ubuntu5.2)，但当前系统却安装上了libkrb5-3的(1.12+dfsg-2ubuntu5.3)版。  
安装指定版本的librkb5-3：
```
$ apt-get install libkrb5-3=1.12+dfsg-2ubuntu5.2
```
librkb5-3的版本号比较长：1.12+dfsg-2ubuntu5.2。  

检查安装成功的u1403，发现其安装krb5-user是5.3版。这说明apt的索引更新有问题。用手机充当热点，执行apt-get udpate。然后查看krb5-user版本：
```
$ apt show krb-user
Version: 1.12+dfsg-2ubuntu5.3
```
再使用apt install krb-user安装就正常了。这说明公司的局域网访问国外有问题。

## 定制ambari服务
Ambari待安装的各服务的配置文件位于/var/lib/ambari-server/resources/stacks/HDP/2.5/services下，每个服务占一个目录。定制的Ambari服务也要放在这个目录下。