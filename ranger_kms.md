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