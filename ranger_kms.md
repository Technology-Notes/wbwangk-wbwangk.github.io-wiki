数据库使用ambari带的postgres。关于ambari自带postgres的内容可参考[ranger测试](ranger测试)。  
下面是用ambari安装Ranger KMS时填入的参数：
```
Ranger KMS DB host: u1401.ambari.apache.org
Ranger KMS DB name: rangerkms
Ranger KMS DB password: vagrant
Database Administrator (DBA) username: postgres
Database Administrator (DBA) password: vagrant
KMS Master Secret Password: 1
````
其它参数使用默认值。由于启用了kerberos，会提示输入管理员主体（root/admin@AMBARI.APACHE.ORG，密码是vagrant）。  

### HDFS加密类型
- 卷加密。加密整个卷。
- 应用加密。应用程序完成加密。
- Rest加密。加密文件或目录。这是一种端到端加密，传输的是密文数据。HDFS系统不能访问加密后的明文数据。
详细内容参考[​HDFS Encryption Overview](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_security/content/hdfs-encryption-overview.html)

### 操作过程
#### 1.创建加密区密钥
在ranger界面中以用户keyadmin:keyadmin登录。密钥管理的管理员与一般的管理员分离。  
通过菜单路径Encrypting -> Add New Key 来添加*加密区密钥*。密钥名称是`zonekey1`。  
#### 2.设置密钥策略
通过菜单路径Access Manager -> Resource Based Policies进入策略定义界面，点击`HDP2610_kms`，这是安装kms后默认创建的策略。显示了一条初始的策略`all - keyname`。编辑这个策略，将用户`jj`添加到这个策略中。如下图：  
![(No Title)](https://github.com/wbwangk/wbwangk.github.io/raw/master/images/kms1.png)
如果不添加这个策略，将来向加密目录上传文件时会报错：
```
put: User:jj not allowed to do 'DECRYPT_EEK' on 'zonekey1'
```
#### 3.创建加密区
```
$ kinit -kt /etc/security/keytabs/hdfs.headless.keytab hdfs-hdp2610
$ hdfs dfs -mkdir /tmp/webb                               (创建目录)
$ hdfs dfs -chmod 777 /tmp/webb                           (设置权限，其它用户有写权限)
$ hdfs crypto -createZone -keyName zonekey1 -path /tmp/webb     (创建加密区)
```
#### 4.上传加密文件
不能直接用hdfs用户(HDFS的管理用户)访问加密目录（hadoop基于安全的考虑），所以要使用jj用户来测试。  
```
$ kadmin.local -q "addprinc jj”
$ kinit jj@AMBARI.APACHE.ORG                              (换用户jj登录)
$ hdfs dfs -put ca.key /tmp/webb
```
#### 权限控制相关
在KMS的配置`Advanced dbks-site`(文件路径是`/etc/ranger-kms/2.6.1.0-129/0/dbks-site.xml`)中有个属性`hadoop.kms.blacklist.DECRYPT_EEK`，是个用户黑名单，名单上的用户不能访问Ranger的解密功能。黑名单的默认值是`hdfs`。所以在默认情况下：
```
$ kinit -kt /etc/security/keytabs/hdfs.headless.keytab hdfs-hdp2610
$  hdfs dfs -copyFromLocal ca.crt /tmp/webb
copyFromLocal: User:hdfs not allowed to do 'DECRYPT_EEK' on 'zonekey1'
```
如果把这个默认值改掉，如随便输入个不存在的用户，保存并服务重启后就可以执行上面的向加密区上传文件的命令了。
#### 其它
查看加密区列表：
```
$ kinit -kt /etc/security/keytabs/hdfs.headless.keytab hdfs-hdp2610
$ hdfs crypto -listZones
/tmp/webb                           zonekey1
```
删除加密区：
```
$ hdfs dfs -rm -R /tmp/webb
```
## hbase存储启用加密
#### 1.集群已经安装hbase
通过ambari界面停止hbase服务。  
将hbase的home目录改名：
```
$ kinit -kt /etc/security/keytabs/hdfs.headless.keytab hdfs-hdp2610
$ hdfs dfs -mv /apps/hbase /apps/hbase-tmp
```
将/apps/hbase转化为加密区：
```
$ hdfs crypto -createZone -keyName zonekey1 -path /apps/hbase
```
将/apps/hbase-tmp目录下内容复制回hbase的根目录中：
```
$ hadoop distcp -skipcrccheck -update /apps/hbase-tmp /apps/hbase
```
实测报错，提示`user:hbase not allowed to do 'DECRYPT_EEK' on 'zonekey1'`。  
以`keyadmin:keyadmin`登录ranger，编辑策略`all-keyname`，在`Decrypt EEK`条目中增加用户`hbase`，问题解决。  

#### 2.集群尚未安装hbase
创建`/apps/hbase`目录，将该目录设置为加密区：
```
$ kinit -kt /etc/security/keytabs/hdfs.headless.keytab hdfs-hdp2610
$ hdfs dfs -mkdir /apps/hbase
$ hdfs crypto -createZone -keyName zonekey1 -path /apps/hbase
```
通过ambari界面设置hbase参数：
```
$ hbase.rootdir=/apps/hbase/data
$ hbase.bulkload.staging.dir=/apps/hbase/staging
```
## WebHDFS启用加密
## 为数据加密创建独立的HDFS管理员
[HDP相关文档](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.0/bk_security/content/hdfs-encr-appendix.html)   
[社区文章](https://community.hortonworks.com/content/supportkb/49506/how-to-add-an-hdfs-admin-user-for-ranger-kms.html)

## 其它
报错：  
```
$ hdfs dfs -put /opt/ca/ca.key /tmp/webb
put: User:hbase not allowed to do 'DECRYPT_EEK' on 'zonekey1'
```
在社区的[相关讨论](https://community.hortonworks.com/questions/118544/user-not-allowed-to-do-decrypt-eek-despite-the-gro.html)。  