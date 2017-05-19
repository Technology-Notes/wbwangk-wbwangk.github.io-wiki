
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
$ wget http://public-repo-1.hortonworks.com/HDP/centos6/2.x/updates/2.6.0.3/HDP-2.6.0.3-centos6-rpm.tar.gz
$ wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.21/repos/centos6/HDP-UTILS-1.1.0.21-centos6.tar.gz
```
然后下载ubuntu14的([参考3](https://docs.hortonworks.com/HDPDocuments/Ambari-2.4.2.0/bk_ambari-installation/content/hdp_25_repositories.html))：
```
$ wget http://public-repo-1.hortonworks.com/ambari/ubuntu14/2.x/updates/2.4.2.0/ambari-2.4.2.0-ubuntu14.tar.gz
$ wget http://public-repo-1.hortonworks.com/HDP/centos6/2.x/updates/2.5.3.0/HDP-2.5.3.0-centos6-rpm.tar.gz
$ wget http://public-repo-1.hortonworks.com/HDP-UTILS-1.1.0.21/repos/centos6/HDP-UTILS-1.1.0.21-centos6.tar.gz
```
上面的参考2和参考3网页中每个tarball都显示有md5，可以用下列命令计算出md5码，以避免下载不完整：
```
$ openssl md5 ambari-2.4.2.0-ubuntu14.tar.gz   (计算Tarball的MD5码，应与上述网页上公布的一样。这一步式可选的)
```
解压tarball的命令类似：
```
$ tar -xzf ambari-2.4.2.0-ubuntu14.tar.gz
```
解压后需要确认一下Amarbi、HDP、HDP-UTILS的目录结构。
```
Ambari Base URL
http://<web.server>/Ambari-2.5.0.3/<OS>

HDP Base URL
http://<web.server>/hdp/HDP/<OS>/2.x/updates/<latest.version>

HDP-UTILS Base URL
http://<web.server>/hdp/HDP-UTILS-<version>/repos/<OS>
```
### 
建立Amabiri本地源描述文件：
```
$ echo "deb http://$(hostname)/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136 Ambari main" > ambari.list
$ curl http://$(hostname)/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136/ambari.list   （测试一下）
```
如果curl返回ambari.list的文件内容，说明用nginx搭建的apt本地源运行正常。其中```http://$(hostname)/AMBARI-2.4.2.0/ubuntu14/2.4.2.0-136```就是Base URL。  