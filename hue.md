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
修改```/var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/HUE/package/scripts/params.py```：
```
download_url = 'http://repo.imaicloud.com/hue/hue-3.12.0.tgz'
```
然后重启ambari-server:
```
$ ambari-server restart
```
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
      <package><name>make</name></package>
      <package><name>maven</name></package>
      <package><name>libldap2-dev</name></package>
      <package><name>python-dev</name></package>
      <package><name>python-setuptools</name></package>
      <package><name>libgmp3-dev</name></package>
      <package><name>libz-dev</name></package>
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
