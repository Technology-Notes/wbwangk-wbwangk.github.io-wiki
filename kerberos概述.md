## 概念
- principal(主体)，某实体在系统中的身份。实体可能是用户或服务。   
    用户主体类似john@EXAMPLE.COM；服务主体类似hdfs/node2.example.com@EXAMPLE.COM；特殊主体类似root/admin@EXAMPLE.COM。  
- KDC(Key Distribution Center密钥分发中心)，存储和分发密钥。常见的KDC有MIT KDC和域控制器(AD)。  
- REALM(领域)是一组主机的集合。一般是大写字母的域名。  
- Ticket(门票)由KDC发放，表明实体身份的一组字符串。门票的有效期一般10-24小时。可以理解为表示会话的令牌。  
     门票缓存：以文件形式把门票存放在客户端存储中。linux中一般路径是`/tmp/krb5cache`。  
- 凭据(credential)由主体和密码构成，表示实体身份。  
- Keytab(密钥表)由凭据生成的一个文件，等同于凭据，用于无法输入密码的自动执行场景，如服务主体。  
    linux中一般存放在`/etc/security/keytabs/`目录中。  

## 命令
- kinit 登录
```
$ kinit root/admin@AMBARI.APACHE.ORG
$ Kinit -kt /etc/security/keytabs/hdfs.headless.keytab hdfs-<集群id>@AMBARI.APACHE.ORG
```
- klist 列出门票缓存
```
$ klist
$ klist -kt /etc/security/keytabs/hdfs.headless.keytab     (特殊用途：列出keytab中的主体)
```
- kdestroy 登出
- ktutil 工具，可用于生成密钥表(keytab)
```
$ ktutil
    ktutil:  addent -password -p root/admin -k 1 -e RC4-HMAC
    ktutil:  wkt root.keytab  
```
- kpasswd 改密码

## 配置文件
linux下一般是`/etc/krb5.conf`。内容：
```
[libdefaults]
  default_realm = AMBARI.APACHE.ORG
[realms]
  AMBARI.APACHE.ORG = {
    kdc = c7301.ambari.apache.org
  }
 HOME.LANGCHAO.COM = {
    kdc =  jtjndc008.home.langchao.com
  }
```
上面基本上是最小配置了。定义了两个领域(REALM)，每个领域一个KDC。还指定了一个为默认领域。

## kerberos相关协议
- GSSAPI
    供其他应用调用，用于交换令牌。目的是共享安全会话，实现单点登录。windows的kerberos实现不知道GSSAPI，而是一个自定义协议SSPI。  
- SPNEGO
    简单和受保护的GSSAPI谈判机制（SPNEGO）。使kerberos支持HTTP协议（negotiate认证）。

## MIT kerberos安装
服务器端参考[Kerberos管理(admin)](https://imaidata.github.io/blog/kerberos_admin/)    
客户端参考[Kerberos使用(client)](https://imaidata.github.io/blog/kerberos_client/)  

## kerberos使用
- hadoop各服务的命令行  
- curl  
- REST  
   [kerberized集群下HBase REST服务器的启动和测试](https://imaidata.github.io/blog/2017/06/27/kerberized%E9%9B%86%E7%BE%A4%E4%B8%8BHBase-REST%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9A%84%E5%90%AF%E5%8A%A8%E5%92%8C%E6%B5%8B%E8%AF%95/)  
   [Knox的HadoopAuth(SPNEGO和基于委派令牌的认证)](https://imaidata.github.io/blog/2017/06/27/Knox%E7%9A%84HadoopAuth(SPNEGO%E5%92%8C%E5%9F%BA%E4%BA%8E%E5%A7%94%E6%B4%BE%E4%BB%A4%E7%89%8C%E7%9A%84%E8%AE%A4%E8%AF%81)/)  
- 浏览器  
   [windows下用kerberos客户端和火狐登录HDP域](https://imaidata.github.io/blog/2017/06/27/windows%E4%B8%8B%E7%94%A8kerberos%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%92%8C%E7%81%AB%E7%8B%90%E7%99%BB%E5%BD%95HDP%E5%9F%9F/)  
- Java应用  
   [JAAS认证](https://imaidata.github.io/blog/jaas/)  
   [java使用SPNEGO认证](https://imaidata.github.io/blog/java_spnego/)  
   [一个安全HDFS客户端的例子](https://imaidata.github.io/blog/2017/07/20/%E4%B8%80%E4%B8%AA%E5%AE%89%E5%85%A8HDFS%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%9A%84%E4%BE%8B%E5%AD%90/)  
   [java连接HBASE(kerberized集群)](https://imaidata.github.io/blog/2017/07/11/java%E8%BF%9E%E6%8E%A5HBASE(kerberized%E9%9B%86%E7%BE%A4)/)    