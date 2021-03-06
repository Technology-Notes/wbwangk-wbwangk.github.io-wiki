目录：
- [手工部署HUE到centos6下的HDP](#手工部署HUE到centos6下的HDP)  
- [编译Hue](#编译Hue)
- [ambari-hue-service](#ambari-hue-service)
- [笔记:部署hue碰到的问题](#部署hue碰到的问题)


不是所有版本的HDP源都包含了HUE服务。HDP的ubuntu14版本就不含HUE服务，而HDP的centos6版本包含HUE服务。如果要使用HDP官方源部署HUE，只能在centos6环境下进行。  

从测试历程上，首先是手工把HUE在centos6下部署到了HDP，挺容易的；然后想利用HDP包含在centos6下hue安装包，通过ambari部署hue，没有成功；然后转向自己编译hue、自己打包到repo.imaicloud.com本地源，在centos6下的HDP集群上部署成功；然后用同样hue tar包，在centos7下的HDP集群下部署，能部署成功但不能启动，才意识到centos6编译的tar包无法在centos7下运行；重新在centos7下编译hue，部署成功；又在ubuntu14下编译hue，通过ambari部署到ubuntu14下的HDP集群，也成功。  

# 手工部署HUE到centos6下的HDP
本次部署参考了HDP官方文档[Command Line Installation](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.3/bk_command-line-installation/content/installing_hue.html)。
为了进行本次测试，需要先搭建基于centos6的大数据环境。参考[大数据本地开发环境](https://github.com/imaidev/imaidev.github.io/wiki/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%9C%AC%E5%9C%B0%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)，只是要使用```ambari-vagrant/centos6.8```目录下Vagrantfile来启动虚拟机。然后安装Ambari，再利用Ambari部署HDP。    
- 部署环境：vagrant管理下的3台VM(c6801/c6802/c6803)；  
- 操作系统：centos6.8；
- Ambari管理下的HDP组件：HDFS/YARN/MR2/ZooKeeper/AmbariMetrics。  
- HUE安装源：HDP本地源([参考](https://github.com/wbwangk/wbwangk.github.io/wiki/%E6%90%AD%E5%BB%BAHDP%E6%9C%AC%E5%9C%B0%E6%BA%90))  
检测一下安装源是否可用：
```
$ cat /etc/yum.repos.d/HDP.repo
[HDP-2.5]
name=HDP-2.5
baseurl=http://repo.imaicloud.com/HDP/centos6/2.x/updates/2.5.3.0

$ yum list hue hue-*
Installed Packages
hue.x86_64                                    2.6.1.2.5.3.0-37.el6                           @HDP-2.5
hue-beeswax.x86_64                            2.6.1.2.5.3.0-37.el6                           @HDP-2.5
hue-common.x86_64                             2.6.1.2.5.3.0-37.el6                           @HDP-2.5
hue-hcatalog.x86_64                           2.6.1.2.5.3.0-37.el6                           @HDP-2.5
hue-oozie.x86_64                              2.6.1.2.5.3.0-37.el6                           @HDP-2.5
hue-pig.x86_64                                2.6.1.2.5.3.0-37.el6                           @HDP-2.5
hue-server.x86_64                             2.6.1.2.5.3.0-37.el6                           @HDP-2.5
```
#### 配置HDP支持HUE
通过Ambmari界面停止：NameNode服务。
通过Ambari在HDFS服务的Custom core-site配置中增加一下参数(点击Add Property)：
```
hadoop.proxyuser.hue.groups = *
hadoop.proxyuser.hue.hosts = *
```
#### 安装hue包
通过Ambari停止所有的HDP服务。  
执行以下命令安装hue：
```
$ yum install hue
```
#### 修改hue配置文件
hue配置文件位于```/etc/hue/conf/hue.ini```。  
1. 配置HDFS集群
在配置文件的```[hadoop][[hdfs_clusters]] [[[default]]]```小节中： 
```
fs_defaultfs=hdfs://c6801.ambari.apache.org:8020
webhdfs_url=http://c6801.ambari.apache.org:50070/webhdfs/v1/
```
2. 配置YARN(MR2)集群
在配置文件的```[hadoop][[yarn_clusters]] [[[default]]]```小节中：
```
resourcemanager_api_url=http://c6802.ambari.apache.org:8088
resourcemanager_rpc_url=http://c6802.ambari.apache.org:8050
proxy_api_url=http://c6802.ambari.apache.org:8088
history_server_api_url=http://c6802.ambari.apache.org:19888
app_timeline_server_api_url=http://c6802.ambari.apache.org:8188
node_manager_api_url=http://c6802.ambari.apache.org:8042
```
需要先通过Ambari界面查看各个组件安装的主机FQDN，不同环境下的主机HQDN可能不同。  
本测试环境没有安装其他的HDP服务，只配置了上述服务。  

#### 启停hue服务
启动、停止、重启分别执行下列命令：
```
$ /etc/init.d/hue start
$ /etc/init.d/hue stop
$ /etc/init.d/hue restart
```
这里当然执行启动命令。  

#### 验证Hue安装
确保你的windows主机配置文件hosts(如c:/windows/system32/drivers/etc/hosts)中定义了:
```
192.168.68.101 c6801.ambari.apache.org
```
在浏览器中输入这个地址：
```http://c6801.ambari.apache.org:8000/dump_config```。
如果Hue安装正确，会出现Hue登录界面。登录界面提示：  
*由于这是您第一次登录，请选择任何用户名和密码。一定要记住这些，因为 它们将成为您的Hue超级用户凭据。*    
可以输入类似admin/admin当Hue的管理员账号。  
登录后可以点File Browser菜单看看HDFS上的文件清单。  
可以点击About > Configuration 看看配置，点击About > Check 检查配置问题。  
需要说明的是，HDP的HUE版本的界面与githue.com版本(最新的是3.12.0版本)有所不同，可能是HDP内置的HUE版本较古老。  

# 编译Hue
为什么要自己编译HUE？因为无论githue.com还是HDP都没有为HUE提供各种版本linux的下载源。  
测试发现，在centos6.8下编译的hue在centos7.0下运行不了（貌似因为python版本不同）。因此需要分别在centos6.8/centos7.3/ubuntu14.4下进行了编译。本章主要描述centos7.3和ubuntu14.4下的编译过程。  
## centos7.3下编译HUE
使用下面的vagrant box搭建的centos7编译环境：
```
config.vm.box = "geerlingguy/centos7"
```
实测发现下面的box执行yum update有很多错误，不如上面的那个box：
```
config.vm.box = "bento/centos-7.3"
```
### 准备
按HUE官方github库的[提示](https://github.com/cloudera/hue)，编译环境需要先装Oracle JDK。  
更详细的手册在[这里](https://github.com/cloudera/hue/blob/bedc719efbaa1a09fbb27a699d3fe9f1ad31fabf/docs/manual.txt#L56)。  
首先，卸载已有JDK的方法：
```
$ rpm -qa | grep java      或  rpm -qa | grep jdk
java-1.8.0-openjdk-headless-1.8.0.131-0.b11.el6_9.x86_64
$ rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.131-0.b11.el6_9.x86_64
```
到[Oracle Java下载页面](http://www.oracle.com/technetwork/java/javase/downloads/index.html)下载RPM包。直接用wget不行，需要先下载到windows下。现已经把安装包上传到repo.imaicloud.com。
```
$ wget http://repo.imaicloud.com/hue/jdk-8u131-linux-x64.rpm
$ rpm -ivh jdk-8u131-linux-x64.rpm  或  yum install jdk-8u131-linux-x64.rpm
```
安装编译需要的其他包(centos6)：
```
yum install git ant gcc gcc-c++ wget tar asciidoc krb5-devel libxml2-devel libxslt-devel openldap-devel python-devel python-simplejson python-setuptools sqlite-devel rsync saslwrapper-devel pycrypto gmp-devel libyaml-devel cyrus-sasl-plain cyrus-sasl-devel cyrus-sasl-gssapi libffi-devel mysql-devel openssl-devel make apache-maven libtidy 
```
centos7下maven需要这样安装：```yum install maven```。 
sqlite-devel的下载需要翻墙。可以手工下载和安装：
```
$ wget http://repo.imaicloud.com/hue/sqlite-devel-3.7.17-8.el7.x86_64.rpm
$ yum install sqlite-devel-3.7.17-8.el7.x86_64.rpm
```
### 下载源码和编译
下载hue源码，然后编译：
```
$ git clone --depth 1 https://github.com/cloudera/hue.git
$ cd hue
$ make apps > hue_make.log 2>&1
```
make apps后面的代码是为了把编译过程输出到文件，以便以查错。  
#### 测试hue
```
$ build/env/bin/hue runserver
$ curl -L http://127.0.0.1:8000
```
#### 打包和本地源
把编译结果打包成tarball：
```
$ cd ..    
$ tar -zcvf hue.tgz hue     (把hue目录打包成hue.tgz)
$ mv hue.tgz hue-3.12.0-centos7.tgz               (文件改名)
$ scp hue-3.12.0-centos7.tgz root@repo.imaicloud.com:/opt/nginx/repo/hue/           (将tar包上传到本地源)
```
新生成的barball的下载路径是```http://repo.imaicloud.com/hue/hue-3.12.0-centos7.tgz```。  
(Vagrant：/e/t)

## ubuntu14下编译hue

准备编译需要的包：
```
apt-get install git ant gcc g++ wget tar libkrb5-dev libxml2-dev libxslt1-dev libldap2-dev python-dev python-setuptools libsqlite3-dev libsasl2-dev libsasl2-modules-gssapi-mit libmysqlclient-dev maven libtidy-0.99-0 libffi-dev libssl-dev
```
安装oracle JDK：
```
$ apt list --installed | grep openjdk    (根据安装openjdk决定下面的remove)
$ apt remove openjdk-7-jre
$ apt remove openjdk-7-jre-headless
$ apt-get install software-properties-common
$ add-apt-repository ppa:webupd8team/java
$ apt-get update
$ apt-get install oracle-java8-installer
```
编译、打包同centos下。
上传tar包到imaicloud.com本地源：
```
$ scp hue-3.12.0-ubuntu14.tgz root@repo.imaicloud.com:/opt/nginx/repo/hue/           (将tar包上传到本地源)
```
新生成的barball的下载路径是```http://repo.imaicloud.com/hue/hue-3.12.0-ubuntu14.tgz```，大小大约400M。 
(Vagrant:/e/vagrant9/ambari-vagrant/ubuntu14.4/u1409)

# ambari-hue-service
github上的[EsharEditor/ambari-hue-service]((https://github.com/EsharEditor/ambari-hue-service))库是将hue制作成了Ambari的服务，通过Ambari将hue部署到HDP集群。

## 在centos6下通过ambari部署hue
### 准备
测试环境的三台VM是(操作系统centos6.8)c6801/c6802/c6803，是Ambari安装的HDP。环境搭建参考[这个](https://github.com/imaidev/imaidev.github.io/wiki/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%9C%AC%E5%9C%B0%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)文档的centos6部分。  
在c6801上执行：
```
$ VERSION=`hdp-select status hadoop-client | sed 's/hadoop-client - \([0-9]\.[0-9]\).*/\1/'`
$ rm -rf /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/HUE  
$ sudo git clone --depth 1 https://github.com/imaidev/ambari-hue-service.git /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/HUE
```
可以把上面的VERSION定义一行添加到文件```~/.bashrc```的最后，以便这个环境变量随时可用。  
不同的操作系统需要不同的hue安装包，repo.imaicloud.com上现有三个操作系统的HUE-3.12.0安装包：
 - **centos6**：https://repo.imaicloud.com/hue/hue-3.12.0-centos6.tgz  
 - **centos7**: https://repo.imaicloud.com/hue/hue-3.12.0-centos7.tgz
 - **ubuntu14**: https://repo.imaicloud.com/hue/hue-3.12.0-ubuntu14.tgz  

编辑文件```/var/lib/ambari-server/resources/stacks/HDP/2.5/services/HUE/package/scripts/params.py```中的代码，根据操作系统选择正确的tar包，如ubuntu14下需要修改成：
```
download_url = 'echo "http://repo.imaicloud.com/hue/hue-3.12.0-ubuntu14.tgz"'
```
重启ambari：
```
$ ambari-server restart
```
通过ambari界面添加服务，在服务清单中可以看到多了一个HUE服务。选中HUE服务，一路按默认值点next按钮。hue一般默认装在u1401节点上。如果启用了kerberos需要输入管理员的主体和密码（如root/amdin@AMBARI.APACHE.ORG）。  
一般会在“Install, Start and Test”一步出问题，逐个解决。  
可能需要重启一些受影响的服务。然后用浏览器访问地址：```http://u1401.ambari.apache.org:8000```。会出现hue登录界面，这就表示安装成功了。也可直接输入ip，即```192.168.14.101:8000```。  

#### 卸载HUE
通过Ambari界面卸载HUE服务，然后手工删除hue相关目录：
```
rm -rf /opt/hue*         (原来是 rm -rf /usr/local/hue*)
rm -rf /var/log/hue
rm -rf /var/run/hue
rm /usr/hdp/current/hadoop-client/lib/hue-plugins-3.12.0-SNAPSHOT.jar
```
#### centos7下部署hue
与centos6基本一样，下载的hue包是hue-3.12.0-centos7.tgz。

## ubuntu14下通过ambari部署HUE 
参照前面centos6下的说明，下载ambari-hue-service到Ambari的服务目录下。  
imaidev/ambari-hue-server默认是部署在centos7环境下，需要修改修在链接。编辑文件```/var/lib/ambari-server/resources/stacks/HDP/2.5/services/HUE/package/scripts/params.py```中的内容：
```
download_url = 'echo "http://repo.imaicloud.com/hue/hue-3.12.0-ubuntu14.tgz"'
```
默认安装目录是```/opt/hue```，确保这个目录不存在。  
重启ambari服务：
```
$ ambari-server restart
```
通过ambari界面添加服务，在服务清单中可以看到多了一个HUE服务。选中HUE服务，一路按默认值点next按钮。hue一般默认装在u1401节点上。如果启用了kerberos需要输入管理员的主体和密码（如root/amdin@AMBARI.APACHE.ORG）。  
一般会在“Install, Start and Test”一步出问题，逐个解决。貌似ubuntu14比centos6/7出的问题少。  
可能需要重启一些受影响的服务。然后用浏览器访问地址：```http://u1401.ambari.apache.org:8000```。会出现hue登录界面，这就表示安装成功了。也可直接输入ip，即```192.168.14.101:8000```。  

## Ambari安装hue（未完成）
#### 准备hue元数据库
直接使用Ambari自带的PostgreSQL数据库，为HUE创建数据库hue和数据库用户hue：
```
$ sudo -u postgres psql
postgres=# CREATE USER hue WITH PASSWORD '1';              (新建一个数据库用户hue，密码是1)
CREATE ROLE     (这是个成功的提示)
postgres=# CREATE DATABASE hue OWNER hue;                (创建用户数据库hue，并指定所有者为hue)
CREATE DATABASE
```
#### Ambari向导
Hue Metastore配置:
```
DB FLAVOR = PostgreSQL
Database Name = hue
Database Username = hue
Database Password = 1
Hue Metastore Host = u1401.ambari.apache.org
Hue Metastore Port = 5432
```
将PostgreSQL Configs的开关打开：
```
PostgreSQL Nice Name = "PostgreSQL DB"   （默认）
PostgreSQL User = hue
PostgreSQL Password = 1
PostgreSQL Database Name = hue
PostgreSQL Host = u1401.ambari.apache.org
PostgreSQL Port = 5432
PostgreSQL Options = {}
```
之后提示“Configure principal name and keytab location for service users and hadoop service components.” 说明需要手工创建HUE的主体和相应的keytab。
```
$ kadmin.local -q "addprinc hue/u1401.ambari.apache.org@AMBARI.APACHE.ORG"
$ kadmin.local
kadmin.local:  ktadd -k hue.keytab hue/u1401.ambari.apache.org@AMBARI.APACHE.ORG
$ mv hue.keytab /etc/security/keytabs/        (将hue.keytab移动到HDP默认的keytabs目录)
```
# 部署hue碰到的问题
对EsharEditor/ambari-hue-service的修改主要集中在目录```/var/lib/ambari-server/resources/stacks/HDP/2.5/services/HUE/package/scripts```下几个py源码中。
1. download_url
将```{print $2"hue/hue-3.11.0.tgz"}```修改成了```{print $2"/hue/hue-3.11.0.tgz"}```(加了个斜杠,params.py)。  
2. tar解压路径
按common.py脚本的预期，hue将解压到```/usr/local/hue```目录下。实际上tar命令解压后的路径是```/usr/local/hue-3.11.0```。为了解决这个问题，在params.py中定义了一个hue_version变量(params.py)：
```
hue_version = "hue-3.11.0"
#hue_dir = format('{hue_install_dir}/hue')
hue_dir = format('{hue_install_dir}/{hue_version}')
```
注释掉的是原来的写法。手工打包的hue.tgz，如果包中的路径是hue，而不带版本号，则不需要做这个修改。  
3. 符号链接
如果符号链接存在就报错，现在增加-f参数，可以覆盖原有符号链接(common.py)：
```
#Execute('ln -s {0} /usr/hdp/current/hue-server'.format(params.hue_dir))
Execute('ln -f -s {0} /usr/hdp/current/hue-server'.format(params.hue_dir))
```
做以上修改后，可以通过Ambari成功安装Hue。  
4. pseudo-distributed.ini
{ambari-hue-service}/package/scripts/setup_hue.py文件中注释掉了以下几行：
```
# Logger.info(format("Creating {hue_conf_dir}/pseudo-distributed.ini config file"))
#  File(format("{hue_conf_dir}/pseudo-distributed.ini"), 
#    content = InlineTemplate(params.hue_pseudodistributed_content), 
#    owner = params.hue_user
#  )
```
如果以上代码在，通过ambari启动hue服务的时候报ecodeing错误。  
5. supervisor.log写权限
在hue-install.log报告没有写入/opt/hue/logs/supervisor.log的权限。解决办法如下：
```
$ chmod +w /opt/hue/logs
$ chmod +w /opt/hue/logs/*
```
赋予了用户hue针对上述目录的权限。感觉这个问题与在当前机器上进行的编译、本地调试有关。按说正常不会出现。由于在本机上进行了编译和调试，导致/opt/hue/logs目录的拥有者是root，按说这个目录的拥有者应是hue用户。  

6. 符号链接hue-plugins-3.11.SNAPSHOT.jar
```
  Logger.info("Creating symlinks /usr/hdp/current/hadoop-client/lib/hue-plugins-3.11.0-SNAPSHOT.jar")
#  Link("{0}/desktop/libs/hadoop/java-lib/*".format(params.hue_dir),to = "/usr/hdp/current/hadoop-client/lib")
  Link("/usr/hdp/current/hadoop-client/lib/hue-plugins-3.12.0-SNAPSHOT.jar",to = "{0}/desktop/libs/hadoop/java-lib/hue-plugins-3.12.0-SNAPSHOT.jar".format(params.hue_dir))
```
不明白原来的写法的目的。根据直觉改写了。原写法在ubuntu14下执行报错，但在centos6下没问题。  

7. (启动)使用github.com/cloudera/hue下载包
该下载包中没有build目录，自然就没有```build/env/bin/supervisor```文件，导致日志文件中报错：
```
$ cat /var/log/hue/hue-install.log
-su: /usr/local/hue-release-3.12.0/build/env/bin/supervisor: No such file or directory
```
说明github.com/cloudera/hue下载包不能用，download_url只能用手工编译的tar包。  
8. /opt/hue目录不存在
手工编译使用的目录是/opt/hue, 启动hue服务提示找不到/opt/hue/build/env/bin目录。临时解决方案是建立一个软符号链接：
```
ln -s /usr/local/hue /opt/hue
```
9. 提示"找不到yarn-site配置文件"  
通过ambari先装YARN，再装hue，不再报错。  

10. 在Centos7.3安装的时候会发现libtidy-0.99-0这个包并不存在，需要更改为libtidy，在metainfo.xml 这个文件里修改。

11. 如果ambari 的密码更改过，在安装hue之后启动的过程中会有错误，需要更改/var/lib/ambari-agent/cache/stacks/HDP/2.6/services/HUE/package/files/configs.sh这个文件里面PASSWD这给参数。
