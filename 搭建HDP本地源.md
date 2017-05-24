
[参考1](https://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/setting_up_a_local_repository.html)  
计划搭建HDP的ubuntu14和centos6的本地源。  

#### 什么是Base URL
在网页[Ambari barball](https://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/ambari_repositories.html)中提到了Base URL。Base URL是库的基础地址。  
对于ubuntu操作系统，目录/etc/apt/sources.list文件中：
```
deb http://security.ubuntu.com/ubuntu yakkety-security universe
```
中的```http://security.ubuntu.com/ubuntu```就是Base URL。  
而对于CentOS操作系统，库文件位于```/etc/yum.repos.d/```目录下，可以看一下```/etc/yum.repos.d/hdp.repo```文件中的baseurl，如：
```
baseurl=http://public-repo-1.hortonworks.com/HDP/centos6/2.x/updates/2.6.0.3
```
如果进入unbuntu操作系统(DPKG标准)查看，发现在base url对应的目录之下是```dists```目录，这应是apt打包系统的约定。而```yakkety-security```是dists的下级目录，依次类推（空格隔开的多级目录）。   
centos下(YUM标准)则没有```dists```目录，直接按baseurl创建目录即可。  

### 建立本地源服务器
计划在10.0.9.105上建立apt本地源，定义一个域名repo.imaicloud.com指向10.0.9.105这个IP地址，这是个内网地址，只能在浪潮内网访问，在公司外需要先启动VPN。9.105的操作系统是centos7，已经安装有nginx。增加nginx的配置如下：
```
server {
        listen 80;
        root repo;
        index index.html index.htm index.nginx-debian.html;
        server_name repo.imaicloud.com;
        location / {
                autoindex on;
                try_files $uri $uri/ =404;
        }
}
```
进入repo目录，并从HDP官方下载Amarbi、HDP、HDP-UTILS的ubuntu14和centos6下的tarball。首先下载centos6的([参考2](http://docs.hortonworks.com/HDPDocuments/Ambari-2.5.0.3/bk_ambari-installation/content/hdp_26_repositories.html))：
```
$ cd /opt/nginx/repo
$ wget http://public-repo-1.hortonworks.com/ambari/centos6/2.x/updates/2.5.0.3/ambari-2.5.0.3-centos6.tar.gz
$ wget http://public-repo-1.hortonworks.com/HDP/centos6/2.x/updates/2.5.3.0/HDP-2.5.3.0-centos6-rpm.tar.gz
$ wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.21/repos/centos6/HDP-UTILS-1.1.0.21-centos6.tar.gz
```
然后下载ubuntu14的([参考3](https://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/hdp_25_repositories.html))：
```
$ wget http://public-repo-1.hortonworks.com/ambari/ubuntu14/2.x/updates/2.4.2.0/ambari-2.4.2.0-ubuntu14.tar.gz
$ wget http://public-repo-1.hortonworks.com/HDP/ubuntu14/2.x/updates/2.5.3.0/HDP-2.5.3.0-ubuntu14-deb.tar.gz
$ wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.21/repos/ubuntu14/HDP-UTILS-1.1.0.21-ubuntu14.tar.gz
```
上面的参考2和参考3网页中每个tarball都显示有md5，可以用下列命令计算出md5码，以避免下载不完整：
```
$ openssl md5 ambari-2.4.2.0-ubuntu14.tar.gz
```
解压tarball的命令类似：
```
$ tar -xzf ambari-2.4.2.0-ubuntu14.tar.gz
```
解压后需要确认一下Amarbi、HDP、HDP-UTILS的目录结构:  
 - Ambari Base URL : ```http://<web.server>/Ambari-2.5.0.3/<OS>```
 - HDP Base URL : ```http://<web.server>/HDP/<OS>/2.x/updates/<latest.version>```
 - HDP-UTILS Base URL : ```http://<web.server>/HDP-UTILS-<version>/repos/<OS>```  

注意，[参考1]的官方文档中，HDP的Base URL是```http://<web.server>/hdp/HDP```，本文中去掉了hdp这个目录。  
9.105上实际部署的HDP版本可能与之前描述的不同（目前部署的ambari2.4.2.0），但原理相同。目前9.105上的本地源目录：
```
repo/AMBARI-2.4.2.0/centos6
repo/AMBARI-2.4.2.0/ubuntu14
repo/HDP/centos6/2.x/updates/2.5.3.0
repo/HDP/centos7/2.x/updates/2.5.3.0
repo/HDP/ubuntu14
repo/HDP-UTILS-1.1.0.21/repos/ubuntu14
repo/HDP-UTILS-1.1.0.21/repos/centos6
repo/HDP-UTILS-1.1.0.21/repos/centos7
```
根据上述路径，将repo替换为域名就是BaseURL，如```http://repo.imaicloud.com/HDP/centos7/2.x/updates/2.5.3.0```就HDP2.5.3在centos7的BaseURL。  
### 库描述文件 
对于部署HDP的集群中的所有机器都要创建HDP本地源的库描述文件。  
【ubuntu14】:
```
$ wget -O /etc/apt/sources.list.d/ambari.list http://repo.imaicloud.com/AMBARI-2.4.2.0/ubuntu/142.4.2.0-136/ambari.list
$ wget -O /etc/apt/sources.list.d/HDP.list http://repo.imaicloud.com/HDP/ubuntu14/HDP.list
```
【centos6】：
```
$ wget -O /etc/yum.repos.d/ambari.repo http://repo.imaicloud.com/AMBARI-2.4.2.0/centos6/2.4.2.0-136/ambari.repo
$ wget -O /etc/yum.repos.d/hdp.repo http://repo.imaicloud.com/HDP/centos6/2.x/updates/2.5.3.0/hdp.repo
```
【centos7】：
```
$ wget -O /etc/yum.repos.d/ambari.repo http://repo.imaicloud.com/AMBARI-2.4.2.0/centos6/2.4.2.0-136/ambari.repo
$ wget -O /etc/yum.repos.d/hdp.repo http://repo.imaicloud.com/HDP/centos7/2.x/updates/2.5.3.0/hdp.repo
```

### 使用163源
163源centos的[官方帮助](http://mirrors.163.com/.help/centos.html)。  
```
$ wget -O /etc/yum.repo.d/CentOS7-Base-163.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo
```
在CentOS7-Base-163.repo中有个变量```$releasever```解析不正确（解析成了7server），直接手工替换成了```7```。  
然后执行：
```
$ yum clean all
$ yum makecache
```