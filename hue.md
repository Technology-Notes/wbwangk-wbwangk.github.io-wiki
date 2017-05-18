### 准备
gethue.com背书的Ambari定制HUE服务的项目位于[ambari-hue-service](https://github.com/EsharEditor/ambari-hue-service)。  

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
      <package><name>make</name></package>
      <package><name>maven</name></package>
      <package><name>libdap2-dev</name></package>
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
