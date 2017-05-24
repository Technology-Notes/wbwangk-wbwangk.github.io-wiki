# 手工部署HUE到HDP
本次部署参考了HDP官方文档[Command Line Installation](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.5.3/bk_command-line-installation/content/installing_hue.html)。  
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
在配置文件的[hadoop][[hdfs_clusters]] [[[default]]]小节中： 
```
fs_defaultfs=hdfs://c6801.ambari.apache.org:8020
webhdfs_url=http://c6801.ambari.apache.org:50070/webhdfs/v1/
```
2. 配置YARN(MR2)集群
在配置文件的[hadoop][[yarn_clusters]] [[[default]]]小节中：
```
resourcemanager_api_url=http://c6802.ambari.apache.org:8088
resourcemanager_rpc_url=http://c6802.ambari.apache.org:8050
proxy_api_url=http://c6802.ambari.apache.org:8088
history_server_api_url=http://c6802.ambari.apache.org:19888
app_timeline_server_api_url=http://c6802.ambari.apache.org:8188
node_manager_api_url=http://c6802.ambari.apache.org:8042
```
通过Ambari界面查看各个组件安装的主机FQDN。  
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
在浏览器中输入这个地址：```http://c6801.ambari.apache.org:8000/dump_config```。如果Hue安装正确，会出现Hue登录界面。登录界面提示：*由于这是您第一次登录，请选择任何用户名和密码。一定要记住这些，因为 它们将成为您的Hue超级用户凭据。*  可以输入类似admin/admin当Hue的管理员账号。  
登录后可以点File Browser菜单看看HDFS上的文件清单。  
可以点击About > Configuration 看看配置，点击About > Check 检查配置问题。  

# ambari-hue-service
### 准备
gethue.com背书的Ambari定制HUE服务的项目位于[ambari-hue-service](https://github.com/EsharEditor/ambari-hue-service)。 
测试环境的三台VM是(操作系统centos6.8)c6801/c6802/c6803，是Ambari安装的HDP。环境搭建参考[这个](https://github.com/imaidev/imaidev.github.io/wiki/%E5%A4%A7%E6%95%B0%E6%8D%AE%E6%9C%AC%E5%9C%B0%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83)文档的centos6部分。  
在c6801上执行：
```
$ VERSION=`hdp-select status hadoop-client | sed 's/hadoop-client - \([0-9]\.[0-9]\).*/\1/'`
$ rm -rf /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/HUE  
$ sudo git clone https://github.com/EsharEditor/ambari-hue-service.git /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/HUE
```
可以把上面的VERSION定义一行添加到文件```~/.bashrc```的最后，以便这个环境变量随时可用。  
重启ambari：
```
$ ambari-server restart
```
gethue.com上有hue 3.10/3.11/3.12的下载链接，但需要翻墙。翻墙后下载到了repo.imaicloud.com下，地址是：
```
http://repo.imaicloud.com/hue/hue-3.10.0.tgz
http://repo.imaicloud.com/hue/hue-3.11.0.tgz
http://repo.imaicloud.com/hue/hue-3.12.0.tgz
```
```EsharEditor/ambari-hue-service```项目的```/var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/HUE/package/scripts/params.py```脚本中有个download_url的变量：
```
#download_url = 'cat /etc/yum.repos.d/HDP.repo | grep "baseurl" | awk -F \'=\' \'{print $2"hue/hue-3.11.0.tgz"}\''
download_url = 'cat /etc/yum.repos.d/HDP.repo | grep "baseurl" | awk -F \'=\' \'{print $2"/hue/hue-3.11.0.tgz"}\''  (或)
download_url = 'echo "http://repo.imaicloud.com/hue/hue.tgz"'
```
由于从yum.repos.d目录取下载地址的写法并不适用与ubuntu，所以只能使用上面的第2种写法。    
HDP的本地源中是没有上述包的，为了解决该文件，在HDP的本地源目录下建立符号链接：
```
$ cd /opt/nginx/repo/HDP/centos6/2.x/updates/2.5.3.0/hue
$ ln -s /opt/nginx/repo/hue/hue-3.10.0.tgz hue-3.10.0.tgz
$ ln -s /opt/nginx/repo/hue/hue-3.11.0.tgz hue-3.11.0.tgz
$ ln -s /opt/nginx/repo/hue/hue-3.12.0.tgz hue-3.12.0.tgz
```
然后重启ambari-server:
```
$ ambari-server restart
```
### 安装解决的问题
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
#### 服务启动

# ubuntu14下通过ambari安装HUE 

ubuntu下的/var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/HUE/metainfo.xml需要增加以下内容。默认的xml文件中定义的依赖包并不适合ubuntu。
```
<osSpecifics>
  <osSpecific>
    <osFamily>ubuntu14</osFamily>
    <packages>
      <package><name>ant</name></package>
      <package><name>gcc</name></package>
      <package><name>g++</name></package>
      <package><name>libffi-dev</name></package>
      <package><name>libkrb5-dev</name></package>
      <package><name>libmysqlclient-dev</name></package>
      <package><name>libsasl2-dev</name></package>
      <package><name>libsasl2-modules-gssapi-mit</name></package>
      <package><name>libsqlite3-dev</name></package>
      <package><name>libssl-dev</name></package>
      <package><name>libtidy-0.99-0</name></package>
      <package><name>libxml2-dev</name></package>
      <package><name>libxslt-dev</name></package>
      <!--package><name>make</name></package-->
      <package><name>maven</name></package>
      <package><name>libldap2-dev</name></package>
      <package><name>python-dev</name></package>
      <package><name>python-setuptools</name></package>
      <!--package><name>libgmp3-dev</name></package-->
      <!--package><name>libz-dev</name></package-->
    </packages>
  </osSpecific>
</osSpecifics>
```

## Ambari安装hue
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
# 编译Hue
按HUE官方github库的[提示](https://github.com/cloudera/hue)，编译环境需要先装Oracle JDK。  
更详细的手册在[这里](https://github.com/cloudera/hue/blob/bedc719efbaa1a09fbb27a699d3fe9f1ad31fabf/docs/manual.txt#L56)。  
首先，卸载已有JDK的方法：
```
$ rpm -qa | grep java      或  rpm -qa | grep jdk
java-1.8.0-openjdk-headless-1.8.0.131-0.b11.el6_9.x86_64
$ rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.131-0.b11.el6_9.x86_64
```
到[Oracle Java下载页面](http://www.oracle.com/technetwork/java/javase/downloads/index.html)下载RPM包。直接用wget不行，先下载到windows下，然后进git bash，然后用scp复制到centos中(命令如scp jdk-8u131-linux-x64.rpm root@c6801:/opt)。  
```
$ rpm -ivh jdk-8u131-linux-x64.rpm  或  yum install jdk-8u131-linux-x64.rpm
```
安装编译需要的其他包(centos)：
```
yum install ant gcc gcc-c++ wget tar asciidoc krb5-devel libxml2-devel libxslt-devel openldap-devel python-devel python-simplejson python-setuptools sqlite-devel rsync saslwrapper-devel pycrypto gmp-devel libyaml-devel cyrus-sasl-plain cyrus-sasl-devel cyrus-sasl-gssapi libffi-devel mysql-devel openssl-devel make apache-maven libtidy 
```
实测中发现如果未安装gcc-c++，则执行make apps报错：
```
Entering directory `/opt/hue/desktop'
make -C core env-install
```
下载hue源码，然后编译：
```
$ git clone https://github.com/cloudera/hue.git
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
$ scp hue.tgz root@repo.imaicloud.com:/opt/nginx/repo/hue/           (将hue.tgz上传到本地源)
$ cd /opt/nginx/repo/HDP/centos6/2.x/updates/2.5.3.0/hue
$ ln -s /opt/nginx/repo/hue/hue.tgz hue.tgz      (配合ambari-hue-service的下载路径)
```
新生成的barball的下载路径是```http://repo.imaicloud.com/hue/hue.tgz```。  