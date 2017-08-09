- 一、[安全知识](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E4%B8%80%E5%AE%89%E5%85%A8%E7%9F%A5%E8%AF%86)  
- 二、[java访问https服务器](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E4%BA%8Cjava%E8%AE%BF%E9%97%AEhttps%E6%9C%8D%E5%8A%A1%E5%99%A8)  
- 三、[建立https服务器](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E4%B8%89%E5%BB%BA%E7%AB%8Bhttps%E6%9C%8D%E5%8A%A1%E5%99%A8)  
- 四、[双向SSL](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E5%9B%9B%E5%8F%8C%E5%90%91ssl)  
- 五、[建立内部CA](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E4%BA%94%E5%88%9B%E5%BB%BA%E5%86%85%E9%83%A8ca)  
- 六、[HDP的SSL证书](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E5%85%ADhdp%E7%9A%84ssl%E8%AF%81%E4%B9%A6)  
- 七、[申请Let's Encrypt证书](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E4%B8%83%E7%94%B3%E8%AF%B7lets-encrypt%E8%AF%81%E4%B9%A6)
- 附: [命令备忘](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E5%91%BD%E4%BB%A4%E5%A4%87%E5%BF%98)
## 一、安全知识
### (一)术语
#### 私钥/公钥对
同时生成的两个字符串。私钥用于签名，公钥验证签名。公钥加密消息，私钥可以解密。  

#### 证书(certificate)
(公钥)证书封装了公钥及配套信息，如公钥拥有者(subject)、签署者的签名、签署者(issuer)。自签名证书的subject与issuer相同。根证书是自签名证书。  
公钥证书可以被认为是护照的数字等价物。它由可信任的组织颁发，可为持有人提供身份证明。发布公钥证书的受信任组织称为证书颁发机构（CA）。  
公钥证书包含以下字段：  

 - 发行方(Issuer)  
  发行证书的CA。如果用户信任颁发证书的CA，并且证书有效，则用户可以信任证书。  
 - 有效期  
  证书有有效期限。在验证证书的有效性时，应检查此日期。  
 - 主体(Subject)  
  包括有关证书所代表的实体的信息。实体可以是个人或组织。  
 - 主体的公钥  
  证书提供的主要信息是主体的公钥。提供所有其他字段以确保此密钥的有效性。  
 - 签名  
  证书由颁发证书的CA进行数字签名。使用CA的私钥创建签名（签署），并确保证书的有效性。因为只有证书被签名，而不是SSL事务中发送的数据，SSL不提供不可否认性。  

自己的私钥给自己的公钥签名，叫自签名证书。更普遍的是其他的私钥（可以视为CA的私钥）为自己的证书签名。  
([X.509证书细节](http://www.cnblogs.com/chnking/archive/2007/08/28/872104.html))  

#### 密钥库(keystore)
将密钥和对应的证书保存在一个受密码保护的数据库中，就叫密钥库。密钥库格式有JKS、PCKS12等。java默认支持jks。  

#### 可信证书库(truststore)
SSL单向认证依赖可信证书库。如IE浏览器中内置了主要证书颁发机构(CA)的根证书和中间人证书，形成了IE的可信证书库。Oracle JDK、CURL等都自带可信证书库，OpenJDK使用Linux操作系统的可信证书库(windows下貌似使用IE的可信证书库)。  

#### 证书签名请求(CSR)
CSR是Certificate Signing Request的简写。某实体要获得CA认可，需要利用自己的私钥/公钥对构造一个证书签名请求文件发给CA。  

#### 证书链
多个证书可以链接成**证书链**。当使用证书链时，第一个证书始终是主体的证书。接下来是颁发发件人证书的实体的证书。如果链中有更多证书，那么每个证书都是颁发先前证书的上级CA。链中的最终证书是根CA的证书。  

在被签署的证书与根证书之间可能存在多级签名，多个证书组成一个链条。最底下的是根证书。根CA是广泛信任的公共证书颁发机构。著名的根CA包括VeriSign，Entrust和GTE Cyber​​Trust。下面是letsencrypt.org的公钥证书的证书链：
```
Certificate chain
 0 s:/CN=letsencrypt.org/O=INTERNET SECURITY RESEARCH GROUP/L=Mountain View/ST=California/C=US
   i:/C=US/O=IdenTrust/OU=TrustID Server/CN=TrustID Server CA A52
 1 s:/C=US/O=IdenTrust/OU=TrustID Server/CN=TrustID Server CA A52
   i:/C=US/O=IdenTrust/CN=IdenTrust Commercial Root CA 1
 2 s:/C=US/O=IdenTrust/CN=IdenTrust Commercial Root CA 1
   i:/O=Digital Signature Trust Co./CN=DST Root CA X3
```
### (二)Java安全体系
[参考](https://docs.oracle.com/javaee/7/tutorial/security-intro002.htm)  

#### Java认证和授权服务（JAAS）
是一组API，可使服务对用户进行身份验证和强制访问控制。JAAS为程序化用户认证和授权提供了可插拔和可扩展的框架。JAAS是一个核心Java SE API，是Java EE安全机制的基础技术。(*博文:[JAAS认证](https://imaidata.github.io/blog/jaas/)*)  

#### Java通用安全服务（Java GSS-API）
是一种基于令牌的API，用于在通信应用程序之间安全地交换消息。GSS-API为应用程序员提供了对各种基础安全机制（包括Kerberos）上的安全性服务的统一访问。(*博文:[java使用SPNEGO认证](https://imaidata.github.io/blog/java_spnego/)*)  

#### Java加密扩展（JCE）
提供了加密，密钥生成和密钥协商以及消息认证码（MAC）算法的框架和实现。对加密的支持包括对称，非对称，块和流密码。块密码对字节组进行操作; 流密码一次操作一个字节。该软件还支持安全流和密封对象。(*博文:[java结合keytool实现非对称加密和解密](https://imaidata.github.io/blog/2017/08/08/java%E7%BB%93%E5%90%88keytool%E5%AE%9E%E7%8E%B0%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E5%92%8C%E8%A7%A3%E5%AF%86/)*)  

#### Java安全套接字扩展（[JSSE](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html)）  
为Java版本的安全套接字层（SSL）和传输层安全（TLS）协议提供了一个框架和实现，并且包括用于数据加密，服务器认证，消息完整性和可选客户端认证的功能以实现安全的互联网通信。(本文主要讲这个模块！)

#### 简单认证和安全层（SASL）
是一种互联网标准（RFC 2222），它规定了客户端和服务器应用程序之间用于认证和可选建立安全层的协议。SASL定义了如何交换认证数据，但本身并不指定该数据的内容。SASL是指定认证数据的内容和语义的特定认证机制适合的框架。

Java SE还提供了一套用于管理密钥库，证书和策略文件的工具; 生成和验证JAR签名; 并获得，列出和管理Kerberos门票。

### (三)SSL如何工作
对网络传输的数据进行加密和解密的算法通常分为两类：**对称加密**和**非对称加密**。  
**对称加密**的加密密码和解密密码相同。对称加密的算法有：DES(Data Encryption Standard)、3DES(Triple DES)、RC2(Rivest Cipher 2)、和RC4(Rivest Cipher 4)。  

**非对称加密**的密钥分为私钥和公钥。公钥可以公开传播，私钥保存在安全的地方。公钥加密，只有对应的私钥能解密。私钥加密，只有对应的公钥能解密（这种特性还可以用来确认发送者身份）。  
经常与SSL一起使用的公钥加密算法是RSA(Rivest Shamir Adleman)算法。使用专门用于密钥交换的SSL的另一种公钥算法是DH(Diffie-Hellman)算法。公钥密码学需要大量的计算，使其非常慢。因此，它通常仅用于加密小块数据，例如秘密密钥，而不是大量的加密数据通信。  

互联网上启用https的网站越来越多。https除了能够加密传输数据外，另一个重要目的是解决网站的“可信”问题。这是一种单向信任的需求，不需要认证客户端。https实际上是HTTPS协议下SSL/TLS。

在商用环境下，可能还需要增加对客户机的认证。这是一种双向认证，又称Mutual Authentication或two-way SSL。

#### 单向SSL
为了实现对服务器的“可信”检验，需要在客户机上搭建“可信证书库”（或称可信密钥库），而在服务器搭建私钥库。

验证过程：客户机向服务器发送随机数(挑战)，要求服务器用公钥加密。客户机收到响应后，用服务器公钥解密。如果解密结果与随机数相同，则说明服务器是公钥证书的主体(Subject)。然后看证书的签名是否来自“可信证书库”，是则说明服务器是可信的。
#### 双向SSL
为了实现双向SSL（互信），需要客户机和服务器上同时建立“可信任证书库”，而且客户机和服务器都有自己的私钥库。

服务器上的“可信证书库”与客户机上的私钥库不同于单向SSL。它们往往是自建CA来实现。
验证过程：除了客户机服务器发“挑战”外，服务器也会向客户机“挑战”。为了确保客户机证书的签署者位于服务器的可信证书库中。

## 二、java访问https服务器

现代浏览器都内嵌了一系列可信CA的公钥证书。如果你访问一个不可信的https网站（如证书是自签名的），浏览器会弹出警告，只有把要访问的网站加入“例外”目录，浏览器才运行继续访问。Java也实现了类似机制。但OpenJDK和OracleJDK的机制有差异。OracleJDK使用自带的可信证书库(文件名为cacerts)，而OpenJDK则使用linux系统的证书体系。  

以下测试的java程序放在c7304的`/opt/https`目录下。而`/opt/ca`目录存放了openssl生成的证书。  

### Oracle JDK访问https链接
OracleJDK的[下载地址](http://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html#X509TrustManager)，可以先在windows下下载，再scp到linux下。  
OracleJDK自带可信证书库文件位置是`$JAVA_HOME/jre/lib/security/cacerts`，这是个JKS格式的keystore文件。可以这样查找可信证书库位置：
```
$  find / -name cacerts
/etc/pki/ca-trust/extracted/java/cacerts
/etc/pki/java/cacerts
/usr/java/jdk1.8.0_131/jre/lib/security/cacerts
```
前两个是同一个文件，是linux系统的可信证书库。第三个OracleJDK带的。  
可以用`keytool`命令查看该文件内容：  
```
$ keytool -list -keystore <cacerts-file> -storepass changeit
```
`changeit`是密钥库的默认密码。 把`<cacerts-file>`替换成`/usr/java/jdk1.8.0_131/jre/lib/security/cacerts`。  
下面编写一个java类来测试。  
#### java访问https网站的源码
这个源码参考了[这个网页](https://zhidao.baidu.com/question/460681916465240325.html))：
```java
import java.io.BufferedReader;
import java.net.URL;
import java.io.OutputStream;
import java.io.InputStreamReader;
import java.io.InputStream;

import javax.net.ssl.HttpsURLConnection;
import javax.net.ssl.SSLContext;
import javax.net.ssl.TrustManager;
import javax.net.ssl.SSLSocketFactory;

public class HttpsTest {
/*
* 处理https GET/POST请求
* 请求地址、请求方法、参数
* */
  public static String httpsRequest(String requestUrl,String requestMethod,String outputStr) {
    StringBuffer buffer=null;
    try{
       //创建SSLContext
       SSLContext sslContext=SSLContext.getInstance("SSL");
       //初始化。第一个null是KeyManager，第二个null是TrustManager。
       sslContext.init(null, null, new java.security.SecureRandom());;
       //获取SSLSocketFactory对象
       SSLSocketFactory ssf=sslContext.getSocketFactory();
       URL url=new URL(requestUrl);
       HttpsURLConnection conn=(HttpsURLConnection)url.openConnection();
       conn.setDoOutput(true);
       conn.setDoInput(true);
       conn.setUseCaches(false);
       conn.setRequestMethod(requestMethod);
       //设置当前实例使用的SSLSoctetFactory
       conn.setSSLSocketFactory(ssf);
       conn.connect();
       //往服务器端写内容
       if(null!=outputStr) {
          OutputStream os=conn.getOutputStream();
          os.write(outputStr.getBytes("utf-8"));
          os.close();
        }
        //读取服务器端返回的内容
        InputStream is=conn.getInputStream();
        InputStreamReader isr=new InputStreamReader(is,"utf-8");
        BufferedReader br=new BufferedReader(isr);
        buffer=new StringBuffer();
        String line=null;
        while((line=br.readLine())!=null) {
          buffer.append(line);
        }
      } catch(Exception e){
        e.printStackTrace();
      }
    return buffer.toString();
  }

  public static void main(String[] args) {
    if(args.length==0) {
      System.out.println("Please enter URL.");
      return;
    }
    String s = httpsRequest(args[0],"GET",null);
    System.out.println(s);
  }
}
```
编译运行：
```
$ javac HttpsTest.java
$ java HttpsTest https://baidu.com
(返回一堆html代码)
$ java HttpsTest https://kyfw.12306.cn
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
(下略)
```
可以尝试用`openssl s_client -connect <host-domain-name>:<port>`命名分别查询一下上面两网站的证书信息。会发现baidu.com的证书签署者`Symantec Class 3 Secure Server CA - G4`出现在了IE的“中间证书发放机构中”，而kyfw.12306.cn的签署者`SRCA`则没有。
#### 导入信任库
首先，查询kyfw.12306.cn的公钥：
```
$ openssl s_client -connect kyfw.12306.cn:443
```
将显示在屏幕上的`-----BEGIN CERTIFICATE-----`和`-----END CERTIFICATE-----`之间的内容（包括这两行）复制到一个文件(12306.crt)中。然后用keytool导入到OracleJDK的信任库：
```
$ keytool -import -trustcacerts -keystore /usr/java/jdk1.8.0_131/jre/lib/security/cacerts -alias 12306 -file 12306.crt  (cacerts的默认密码是changeid)
```
重新发送请求：
```
$ java HttpsTest https://kyfw.12306.cn
```
不再报错。说明`kyfw.12306.cn`已经在JDK的信任库中，JDK允许客户端发送请求到这个主机。  

#### cacerts的修改测试
为了测试cacerts的作用，现在把它改名，然后用一个空文件代替。实测中，下面的<cacerts file>被替换为`/usr/java/jdk1.8.0_131/jre/lib/security/cacerts`：
```
$ mv <cacerts file> <cacerts file>.old
$ echo "" > <cacerts file>
$ java HttpsTest https://baidu.com
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed:
(下略)
```
baidu.com也不行了，是因为cacerts文件中的证书被清空了。别忘记把文件cacerts还原。  

### OpenJDK访问https链接
卸载OracleJDK，安装OpenJDK。再查找`cacerts`文件：
```
$ find / -name cacerts
/etc/pki/ca-trust/extracted/java/cacerts
/etc/pki/java/cacerts    （符号链接，指向第一个cacerts）
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64/jre/lib/security/cacerts
```
再运行`java HttpsTest <URL>`，发现`$JAVA_HOME/jre/lib/security/cacerts`这个文件不再管用，管用的是`/etc/pki/ca-trust/extracted/java/cacerts`。  
说明OpenJDK对于可信证书库的使用与OracleJDK不同，OpenJDK优先使用linux下的可信证书库，而OracleJDK优先使用JDK自带的。  

### HttpClient访问https
利用apache [HttpClient](https://hc.apache.org/)访问https的写法略有差异，但可信证书库的位置、作用等于java完全相同。下面测试在OracleJDK下进行。  
```
$ wget http://mirror.bit.edu.cn/apache//httpcomponents/httpclient/binary/httpcomponents-client-4.5.3-bin.tar.gz
$ tar xzvf httpcomponents-client-4.5.3-bin.tar.gz
$ vi HttpClientSSL.java
```
HttpClientSSL.java类的写法参考了[这个官方例子](https://hc.apache.org/httpcomponents-client-4.5.x/httpclient/examples/org/apache/http/examples/client/ClientCustomSSL.java)。  
```java
import java.io.File;

import org.apache.http.util.EntityUtils;
import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.HttpClients;

public class HttpClientSSL {

    public final static void main(String[] args) throws Exception {
        if(args.length==0) {
                System.out.println("Please enter URL.");
        return;
        }
        CloseableHttpClient httpclient = HttpClients.createDefault();
        try {
            HttpGet httpget = new HttpGet(args[0]);

            System.out.println("Executing request " + httpget.getRequestLine());

            CloseableHttpResponse response = httpclient.execute(httpget);
            try {
                HttpEntity entity = response.getEntity();

                System.out.println("----------------------------------------");
                System.out.println(response.getStatusLine());
                EntityUtils.consume(entity);
            } finally {
                response.close();
            }
        } finally {
            httpclient.close();
        }
    }
}
```
编译执行：
```
$ javac -cp ".:/opt/https/httpcomponents-client-4.5.3/lib/*" HttpClientSSL.java
$ java -cp ".:/opt/https/httpcomponents-client-4.5.3/lib/*" HttpClientSSL https://cn.bing.com
Executing request GET https://cn.bing.com HTTP/1.1
----------------------------------------
HTTP/1.1 200 OK
$ java -cp ".:/opt/https/httpcomponents-client-4.5.3/lib/*" HttpClientSSL https://kyfw.12306.cn
Executing request GET https://kyfw.12306.cn HTTP/1.1
Exception in thread "main" javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
(下略)
```
## 三、建立https服务器
[参考](https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04)  

编辑文件`/etc/yum.repos.d/nginx.repo`：
```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/OS/OSRELEASE/$basearch/
gpgcheck=0
enabled=1
```
由于本地环境是centos7.3，将`OS`替换为`centos`，将`OSRELEASE`替换为`7`。然后：
```
$ yum update -y
$ yum install nginx
```
### (一)自签名证书https服务器
生成私钥和自签名证书：
```
$ openssl req -new -newkey rsa:2048 -nodes -x509 -keyout nginx.key -out nginx.crt -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=c7304.ambari.apache.org"
```
`-nodes`表示密钥不加密。不加这个参数会提示输入密码，而nginx不认加密后的key。如果不加`-x509`参数，产生的将是一个CSR(证书签名请求)。在正规的流程中，应产生CSR，CA签名后变成.crt。现在是个简单测试，所以直接生成了自签名证书。  
编辑nginx配置文件`/etc/nginx/conf.d/default.conf`，添加以下内容：
```
server {
    listen       443;
    server_name  c7304.ambari.apache.org;
                ssl on;
                ssl_certificate      /opt/ca/nginx.crt;
                ssl_certificate_key  /opt/ca/nginx.key;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
```
重启nginx，加载新的配置文件，测试https服务器：
```
$ service nginx start           (刚装上nginx时要执行，之后就不用了)
$ ps -ef | grep nginx           (看看nginx进程是否已经启动)
$ nginx -t                      (测试nginx配置文件)
$ nginx -s reload               (重新加载配置文件，修改nginx配置文件有执行这个命令)
```
可以用前面的java程序测试一下https：
```
$ java -cp ".:/opt/https/httpcomponents-client-4.5.3/lib/*" HttpClientSSL https://c7304.ambari.apache.org
```
必然报错，因为服务器证书没加入到可信证书库中。

#### 将公钥导入JDK可信证书库
```
$ keytool -import -trustcacerts -keystore /usr/java/jdk1.8.0_131/jre/lib/security/cacerts -alias nginx -file /opt/ca/nginx.crt -storepass changeit
```
有时要多次导入同名条目。如果提示重名，则删除条目的命令：
```
$ keytool -delete -keystore /usr/java/jdk1.8.0_131/jre/lib/security/cacerts -alias nginx -storepass changeit
```
重新用java程序测试：
```
$ cd /opt/https
$ java -cp ".:/opt/https/httpcomponents-client-4.5.3/lib/*" HttpClientSSL https://c7304.ambari.apache.org
Executing request GET https://c7304.ambari.apache.org HTTP/1.1
----------------------------------------
HTTP/1.1 200 OK
```
可以看到，不报错了。说明nginx的公钥证书导入到JDK可信证书库的操作起作用了。  

#### 用curl进行可信https测试
```
$ curl https://c7304.ambari.apache.org  --cacert /opt/ca/nginx.crt
(不报错，返回了网页)
```  
curl增加`--cacert`参数后，相当于把参数后的证书临时加入了curl的可信库。  
注意，curl的--cacert参数接受的是PEM格式的证书。把JKS格式的密钥库当参数传给curl是不行的，如`/etc/pki/java/cacerts`当curl参数不行。  
curl还有一个`-k`参数可以临时关闭可信检测：
```
$ curl -k https://c7304.ambari.apache.org
```
加上`-k`参数后不需要将目标网站的证书加入可信库。  

### (二)CA签名证书https服务器
要获得一个CA签名的nginx公钥证书，除了通过免费`let's encrypt`网站或商用CA证书公司外，还可以自己搭建一个“内部CA”。搭建办法参见第五章。以下测试假定你已经搭建好了自己的CA。内部CA的另一个用途是颁发个人证书，这在双向SSL(第四章)时会用到。  
首先，生成一个私钥和证书签名请求：
```
$ openssl req -new -newkey rsa:2048 -nodes -keyout nginx2.key -out nginx2.csr -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=c7304.ambari.apache.org"
```
命令格式同前文的自签名相比，去掉了一个`-x509`。输出的文件命名也改了.csr。生成好后可以cat命令看一下.csr文件的内容，发现它的开始一行是`-----BEGIN CERTIFICATE REQUEST-----`，而证书的开始行是`-----BEGIN CERTIFICATE-----`。  

然后，是自建CA处理证书签名请求(CSR)，会提示输入CA私钥密码：
```
$ openssl ca -in nginx2.csr -out nginx2.crt
```
可以用下列命令查看一下签名后的证书，能看到Issuer和Subject不一样了：
```
$ openssl x509 -noout -text -in nginx2.crt
```
#### 配置nginx
nginx的配置文件需要修改，改成新生成的私钥和证书：
```
                ssl_certificate      /opt/ca/nginx2.crt;
                ssl_certificate_key  /opt/ca/nginx2.key;
```
重新加载nginx配置文件：
```
$ nginx -s reload
```
#### 测试https服务器
首先，用curl测试。需要说明的是，对于CA签名的证书，客户端需要信任是CA的公钥证书，而不是https服务器自己的。所以，curl的命令是：
```
$ curl https://c7304.ambari.apache.org  --cacert /root/CA/certs/ca-cert
```
CA公钥的位置参见第五章。还可以把CA公钥分发到curl所在的机器。  

为了测试java访问https服务器，将CA公钥导入到JDK的可信密钥库中(先删除之前条目)：
```
$ keytool -delete -keystore /usr/java/jdk1.8.0_131/jre/lib/security/cacerts -alias nginx -storepass changeit
$ keytool -import -trustcacerts -keystore /usr/java/jdk1.8.0_131/jre/lib/security/cacerts -alias nginx -file /root/CA/certs/ca-cert -storepass changeit
```
重申，导入到可信密钥库的不是https服务器自己的公钥证书，而是CA公钥证书。这是与自签名证书搭建https服务器的显著区别。  
java测试：
```
$ cd /opt/https
$ java -cp ".:/opt/https/httpcomponents-client-4.5.3/lib/*" HttpClientSSL https://c7304.ambari.apache.org
Executing request GET https://c7304.ambari.apache.org HTTP/1.1
----------------------------------------
HTTP/1.1 200 OK
```
### (三)OpenSSL搭建https服务器
openssl自带了https服务器的模拟功能。在c7403上执行：
```
$ cd /opt/ca
$ openssl s_server -accept 444 -cert nginx2.crt -key nginx2.key -www
```
为了不与nginx的https服务器冲突，用了444端口。  
这个openssl的模拟服务器用curl测试：
```
$ curl https://c7304.ambari.apache.org:444  --cacert /root/CA/certs/ca-cert
```
java测试不再冗述。  


## 四、双向SSL
参考：[Nginx实现双向SSL](http://nategood.com/client-side-certificate-authentication-in-ngi)、[双向SSL15分钟教程](http://monduke.com/2006/06/04/the-fifteen-minute-guide-to-mutual-authentication/)  
双向SSL(two-way SSL)又叫Mutual Authentication，或基于证书的相互认证，是指双方通过验证提供的数字证书相互认证，以便双方确保对方的身份。在技​​术术语中，它是指客户端（Web浏览器或客户端应用程序）向服务器（网站或服务器应用程序）进行身份验证，并且服务器也向客户端进行身份验证，验证过程用到了CA颁发的公钥证书/数字证书。
双向SSL下客户端与服务器的交互过程：
1. 客户端请求访问受保护的资源。  
2. 服务器将其证书提交给客户端。  
3. 客户端验证服务器的证书。  
4. 如果成功，客户端将其证书发送到服务器。  
5. 服务器验证客户端的凭据。  
6. 如果成功，则服务器授予对客户端请求的受保护资源的访问权限。  
对以上过程的详细描述可参考[这个](https://community.developer.visa.com/t5/Developer-Tools/What-is-Mutual-Authentication/ba-p/5757)。  

为了实现双向SSL，需要为客户端生成私钥和证书。假定客户端设定为c7302节点。在c7302的`/opt/twowayssl`为客户端用户webb创建签名请求：
```
$ openssl req -new -newkey rsa:2048 -nodes -keyout client.key -out client.csr -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=webb"
$ scp client.csr root@c7304:/opt/ca                (将客户端的证书签名请求发送到CA所在机器)
```
CA建立在c7304上（参见第五章），需要在c7304上对该CSR进行签名：
```
$ cd /opt/ca
$ openssl ca -in client.csr -out client.crt
$ scp client.crt root@c7302:/opt/twowayssl            (将签署后的证书复制回c7302)
```
nginx的配置需要变更为：
```
server {
    listen       443 ssl;
    server_name  c7304.ambari.apache.org;
    ssl_certificate        /opt/ca/nginx.crt;
    ssl_certificate_key    /opt/ca/nginx.key;

    ssl_client_certificate /root/CA/certs/ca-cert;
    ssl_verify_depth 1;
    ssl_verify_client on;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
```
指令`ssl_verify_depth 1`必须有。应该是证书链的验证深度，0表示客户端证书，1表示签署客户端证书的CA。虽然没有做实验，但估计如果客户端证书是自签名的，则深度0（默认值）就可以。  
用`nginx -s reload`重启nginx。
#### curl测试双向SSL
到c7402上用curl测试一下：  
```
$ cd /opt/twowayssl
$ curl -k https://c7304.ambari.apache.org
<html>
<head><title>400 No required SSL certificate was sent</title></head>
(后略)
```
注意到返回的状态码不再是`200`。这说明nginx双向SSL的配置是起作用了。`-k`参数禁用了服务器可信检测，但由于没有提供客户端凭据，仍报错。  
为了用curl测试双向SSL，先将客户端的私钥和证书合并：
```
$ cat client.crt client.key > client.pem
$ curl -k --cert ./client.pem https://c7304.ambari.apache.org
<h1>Welcome to nginx!</h1>
(其它略)
$ curl https://c7304.ambari.apache.org --cacert ca-cert --cert ./client.pem
<h1>Welcome to nginx!</h1>
(其它略)
```
注意[--cert](https://curl.haxx.se/docs/manpage.html#-E)参数如果后面跟文件，必须加上相对或绝对路径，否则后面的参数会当成NSS数据库的nickname。  

#### windows下的双向SSL测试
将自建CA的公钥`ca-cert`文件和`client.pem`两个文件复制到宿主windows下。然后分别导入到IE。其中client.pem导入到了“其他人”标签页(显示颁发给webb)，`ca-cert`导入到了“受信任的发布者”标签页中(显示颁发给iMaiCA)。可能需要在“高级”按钮中选中“用于客户端认证”。  
然后用IE访问地址`https://c7304.ambari.apache.org`，发现可以访问了。  

### java下的双向SSL
[使用keytool将私钥导入到Java密钥库中](http://cunning.sharp.fm/2008/06/importing_private_keys_into_a.html)  
前文用openssl工具生成了客户端私钥和公钥证书。要利用实现java程序的双向SSL访问服务器，首先需要把客户端私钥导入到keystore。keytool没有直接导入私钥的功能，但提供了密钥库合并功能，可以利用这个功能实现密钥导入keystore。  
在c7302上执行：
```
$ /opt/twowayssl
$ openssl pkcs12 -export -in client.crt -inkey client.key -out client.p12 -name webb
Enter Export Password: vagrant
$  keytool -importkeystore -deststorepass vagrant -destkeystore client.jks -srckeystore client.p12 -srcstoretype PKCS12 -srcstorepass vagrant -alias webb
$ keytool -list -keystore client.jks -alias webb -storepass vagrant
webb, Aug 4, 2017, PrivateKeyEntry,
Certificate fingerprint (SHA1): 99:6D:E2:E4:ED:46:91:8C:FC:D6:73:EC:42:74:3C:BF:6E:E8:9F:87
```
由于之前client.jsk并不存在，所以合并库变成了新建一个密钥库。把这个密钥库文件(client.jsk)复制到c7304的`/opt/https`目录下。（复制到c7304只是因为c7304上的JDK可信库已经导入了自建CA的公钥证书，测试方便而已）。  
测试类为[MutualAuthenticationHTTP.java](https://raw.githubusercontent.com/imaidata/blog/master/_posts/kerberos/MutualAuthenticationHTTP.java)。这个类比较长，不再全部贴在文章里了。关键的几行：
```java
        String url = "https://c7304.ambari.apache.org";
        String keyStoreFileName = "client.jks";
        String keyStorePassword = "vagrant";
        String trustStoreFileName = "/usr/java/jdk1.8.0_131/jre/lib/security/cacerts";
        String trustStorePassword = "changeit";
        String alias = "webb";
```
上面6行代码定义了客户端的密钥库与可信库。编译和运行：
```
$ javac MutualAuthenticationHTTP.java -Xlint:deprecation
$ java MutualAuthenticationHTTP
```
程序先打印出了密钥库的条目，又打印出了可信库的条目，最后显示了URL的响应。这说明java程序通过了服务器的双向认证。
尝试将main方法中的源码这样修改：
```
//            TrustManager[] trustManagers = createTrustManagers(trustStoreFileName, trustStorePassword);
            //init context with managers data
            SSLSocketFactory factory = initItAll(keyManagers, null);
```
编译后执行，发现仍能成功。这是因为JDK使用了默认的可信库，而我们手工加载的本来就是OracleJDK自带的可信证书库。  

## 五、创建内部CA
[参考](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_security/content/create-internal-ca.html)，如果对keytool不熟悉建议先读[这个](https://github.com/wbwangk/wbwangk.github.io/wiki/java%E7%BB%93%E5%90%88keytool%E5%AE%9E%E7%8E%B0%E5%85%AC%E7%A7%81%E9%92%A5%E7%AD%BE%E5%90%8D%E4%B8%8E%E9%AA%8C%E8%AF%81)。  

一般使用开源软件OpenSSL来创建CA。首先，生成CA根证书公钥和私钥。然后，将公私钥证书配置到OpenSSL的配置文件。之后，就可以使用内部CA来处理证书签名请求(CSR)，生成签名证书了。  

#### 1.生成密钥对和证书
将创建CA的根证书。
```
$ openssl req -new -x509 -keyout ca-key -out ca-cert -days 365 -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=iMaiCA"
..............+++
.......................................+++
writing new private key to 'ca-key'
Enter PEM pass phrase: vagrant
Verifying - Enter PEM pass phrase: vagrant
-----
```
生成的CA一个公钥-私钥对和证书，旨在签署其他证书。当前目录下多了两个文件ca-key和ca-cert。  
ca-key文件的第一行：`-----BEGIN ENCRYPTED PRIVATE KEY-----`  
ca-cert文件的第一行：`-----BEGIN CERTIFICATE-----`  

#### 2.创建CA目录和复制文件
设置CA目录结构：
```
$ mkdir -m 0700 /root/CA /root/CA/certs /root/CA/crl /root/CA/newcerts /root/CA/private
```
将CA密钥移动到`/root/CA/private`，将CA证书移动到`/root/CA/certs`。  
```
$ mv ca-key /root/CA/private; mv ca-cert /root/CA/certs
```
添加所需文件：
```
$ touch /root/CA/index.txt; echo 1000 >> /root/CA/serial
```
设置权限ca-key：
```
chmod 0400 /root/CA/private/ca-key
```
#### 3.修改OpenSSL配置文件
打开OpenSSL配置文件(`/etc/pki/tls/openssl.cnf`)，修改为以下内容：
```
[ CA_default ]

dir             = /root/CA                  # Where everything is kept
certs           = /root/CA/certs            # Where the issued certs are kept
crl_dir         = /root/CA/crl              # Where the issued crl are kept
database        = /root/CA/index.txt        # database index file.
#unique_subject = no                        # Set to 'no' to allow creation of
                                            # several certificates with same subject.
new_certs_dir   = /root/CA/newcerts         # default place for new certs.

certificate     = /root/CA/certs/ca-cert    # The CA certificate
serial          = /root/CA/serial           # The current serial number
crlnumber       = /root/CA/crlnumber        # the current crl number
                                            # must be commented out to leave a V1 CRL
crl             = /root/CA/crl.pem          # The current CRL
private_key     = /root/CA/private/ca-key   # The private key
RANDFILE        = /root/CA/private/.rand    # private random number file

x509_extensions = usr_cert              # The extensions to add to the cert
```
保存配置文件并重启OpenSSL。  

#### 4.生成CSR并签署
这一步是在测试、验证刚刚建立的内部CA。  
CA的一个重要用途是处理“证书签名请求”，生成签名后的证书。  
假设一个场景：利用nginx搭建https网站。  
首先，生成CSR:
```
$ openssl req -new -newkey rsa:2048 -nodes -keyout nginx.key -out nginx.csr -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=c7304.ambari.apache.org"
```
生成了私钥nginx.key和证书签名请求nginx.csr。nginx.csr的开始一行是`-----BEGIN CERTIFICATE REQUEST-----`。  

下面利用刚创建的CA处理这个证书签名请求：
```
$ openssl ca -in nginx.csr -out nginx.crt
```
会提示输入CA的私钥密码。可以查看一下签名后的证书：
```
$ openssl x509 -noout -text -in nginx.crt
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=CN, ST=Shan Dong, L=Ji Nan, O=Inspur, OU=SBG, CN=iMaiCA
        Validity
            Not Before: Aug  2 00:50:53 2017 GMT
            Not After : Aug  2 00:50:53 2018 GMT
        Subject: C=CN, ST=Shan Dong, O=Inspur, OU=SBG, CN=c7304.ambari.apache.org
```
已上信息省略了一部分。可以看出Issuer就是刚建立的CA，Subjcet是nginx的信息。  

## 六、HDP的SSL证书
#### 1.各节点创建密钥库
为hadoop集群中的个节点，用JDK的keytool创建一个密钥库，用于保存本服务器的证书私钥。以下测试默认在c7302节点上进行。  
```
($ keytool -keystore <keystore-file> -alias localhost -validity <validity> -genkey)
 keytool -keystore c7302.jks -alias localhost -validity 1800 -genkey
Enter keystore password:
Re-enter new password:
What is your first and last name?
  [Unknown]:  c7302.ambari.apache.org
What is the name of your organizational unit?
  [Unknown]:  ec
What is the name of your organization?
  [Unknown]:  sbg
What is the name of your City or Locality?
  [Unknown]:  jinan
What is the name of your State or Province?
  [Unknown]:  shandong
What is the two-letter country code for this unit?
  [Unknown]:  CN
Is CN=c7302.ambari.apache.org, OU=ec, O=sbg, L=jinan, ST=shandong, C=CN correct?
  [no]:  yes

Enter key password for <localhost>
        (RETURN if same as keystore password): （回车）
```
确保公用名称（CN）与服务器的完全限定域名（FQDN）匹配。客户端将CN与DNS域名进行比较，以确保它确实连接到所需的服务器，而不是恶意服务器。  

#### 2.创建CA
参考第一章。

#### 3.将CA添加到各服务器的信任库
```
keytool -keystore c7302.jks -alias CARoot -import -file ca-cert
Enter keystore password: vagrant
Owner: EMAILADDRESS=wbwang@inspur.com, CN=webbca, OU=ec, O=sbg, L=jinan, ST=shandong, C=cn
Issuer: EMAILADDRESS=wbwang@inspur.com, CN=webbca, OU=ec, O=sbg, L=jinan, ST=shandong, C=cn
Serial number: c27c41b2be3b8701
Valid from: Mon Jul 31 01:10:31 UTC 2017 until: Tue Jul 31 01:10:31 UTC 2018
Certificate fingerprints:
         MD5:  A9:72:FB:FD:AE:D7:8C:C6:DA:99:8E:20:5D:6A:6B:CF
         SHA1: C2:FA:D3:C4:FE:98:CF:78:1B:FE:3C:01:68:99:A5:1D:42:D2:D6:E7
         SHA256: 90:FD:EA:5E:70:A2:69:3E:AB:48:DF:4B:A3:5E:C1:19:4C:1F:3E:E7:C9:94:FA:18:4F:A5:EC:E7:48:0E:46:5C
         Signature algorithm name: SHA256withRSA
         Version: 3

Extensions:

#1: ObjectId: 2.5.29.35 Criticality=false
AuthorityKeyIdentifier [
KeyIdentifier [
0000: B5 3E 7B 21 21 A5 8A 18   0C 76 DC D7 D4 65 7C 03  .>.!!....v...e..
0010: 08 88 8D BD                                        ....
]
]

#2: ObjectId: 2.5.29.19 Criticality=false
BasicConstraints:[
  CA:true
  PathLen:2147483647
]

#3: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: B5 3E 7B 21 21 A5 8A 18   0C 76 DC D7 D4 65 7C 03  .>.!!....v...e..
0010: 08 88 8D BD                                        ....
]
]

Trust this certificate? [no]:  yes
Certificate was added to keystore
```
#### 5.签署证书
用步骤2产生的CA签署步骤1生成的所有器证书。  
首先，生成签名请求：
```
$ keytool -keystore c7302.jks -alias localhost -certreq -file cert-file
```
生成了一个cert-fiel文件，其内容：
```
-----BEGIN NEW CERTIFICATE REQUEST-----
(下略)
```
CA进行对签名请求进行签名：
```
$ openssl x509 -req -CA ca-cert -CAkey ca-key -in cert-file -out cert-signed -days 1800 -CAcreateserial -passin pass:vagrant
Signature ok
subject=/C=CN/ST=shandong/L=jinan/O=sbg/OU=ec/CN=c7302.ambari.apache.org
Getting CA Private Key
```
生成了一个文件cert-signed，其内容（第一行与签署前一样）：
```
-----BEGIN CERTIFICATE-----
(下略)
```
#### 6.将签署后的证书导入密钥库
```
$ keytool -keystore c7302.jks -alias localhost -import -file cert-signed
Enter keystore password: vagrant
Certificate reply was installed in keystore
```
## 七、申请Let's Encrypt证书
[参考](https://imququ.com/post/letsencrypt-certificate.html)

letsencrypt.org提供免费https证书。每张证书可以放100个域名，有效期3个月，支持自动续订。2018年开始支持通配符证书。一般的证书方法颁发机构通过邮件接受证书签名请求(CSR)，而Let's Encrypt则是通过互联网实时接受申请、实时发放。将来有可能互联网证书被Let's Encrypt一统江湖。  
服务器证书通过Let's Encrypt申请的好处是，它的证书可以被常见可信证书库信任，不用额外把服务器证书导入可信证书库或添加浏览器例外。如果启用双向SSL，个人证书仍需要自建CA颁发。

Let's Encrypt推荐使用[Certbot](https://certbot.eff.org/)软件申请证书，而本文采用的是[acme-tiny](https://github.com/diafygi/acme-tiny)(也在Let's Encrypt认可的申请方式清单中)。acme-tiny使用python+openssl+bash脚本完成证书申请和续订，对于懂得CA的人更容易控制，也更容易自动化。  

ACME全称是 Automated Certificate Management Environment(自动化证书管理环境)，Let's Encrypt 的证书签发过程使用的就是ACME协议。有关ACME协议的更多资料可以在[这个仓库](https://github.com/ietf-wg-acme/acme/)找到。

Let's Encrypt在证书申请过程中，有一个“挑战”过程，主要挑战你对域名的控制权。当你申请一个域名的证书，它的申请工具(如Cetbot或acme-tiny)会向CA(Let's Encrypt)发送一个用account_key签名的请求，返回值(象是个token)被写入约定的nginx的`<域名>/.well-known/acme-challenge`目录，然后申请工具会请求对应的URL(http://<域名>/.well-known/acme-challenge/<token>)。请求的返回值正确就说明向Let's Encrypt申请证书的域名处于你的控制之下。  

#### 创建账号、证书签名请求CSR
执行Let's Encrypt证书申请的机器必须有互联网IP，而且需要通过域名解析服务商(如阿里云)把要申请域名解析到这个互联网IP。例如与下面的测试相配合，在阿里域名解析中存在一个imaicloud.com域名的条目：`记录类型:A，主机记录:*，解析线路:默认，记录值：<IP地址>`。这是一个通配符解析，意味着`imaicloud.com`的所有下级域名都解析到这个IP地址。

在前面讲的有互联网IP的机器上安装nginx。然后创建一个目录，用来存放密钥对和其它临时文件，如`/root/ssl`：
```
$ cd /root/ssl
$ openssl genrsa 4096 > account.key
$ openssl genrsa 4096 > domain.key
$ openssl req -new -sha256 -key domain.key -subj "/" -reqexts SAN -config <(cat /etc/pki/tls/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:dp.imaicloud.com")) > domain.csr
```
Let's Encrypt可能是处于管理的要求让你生成两个密钥对：acccout和domain。证书签名申请用domain.key签名，而挑战请求用account签名。openssl的配置文件`/etc/pki/tls/openssl.cnf`在第五章提到过。上面的脚本是临时在openssl.cnf文件最后增加了下面的内容：
```
[SAN]
subjectAltName=DNS:dp.imaicloud.com
```
如果一次申请多个域名，就用逗号隔开，如`subjectAltName=DNS:yoursite.com,DNS:www.yoursite.com`。生成证书签名请求(CSR)保存在文件domain.csr中。

如果找不到openssl.cnf的位置，可以查找(不同的linux位置可能不同)：
```
$ find /etc -name openssl*
```
#### 配置nginx
配置nginx的目的是为了迎接来自Let's Encrypt申请工具的挑战。  
NGINX的HOME目录是`/opt/nginx`。
```
$ mkdir /opt/nginx/dp
$ mkdir /opt/nginx/dp/challenge
```
`/opt/nginx/dp/challenge`存放证书申请工具产生的临时文件，用来响应挑战。这个目录通过nginx的别名指令映射为URL`http://dp.imaicloud.com//.well-known/acme-challenge/`，也就是Let's Encrypt约定的挑战目录。

Nginx的配置文件：
```
    server {
        listen 80;
        server_name dp.imaicloud.com;
        root dp;
        index index.html index.htm;

        location ^~ /.well-known/acme-challenge/ {
                alias /opt/nginx/dp/challenge/;
                try_files $uri =404;
        }
```
#### 申请证书

```
$ cd /root/ssl
$ wget https://raw.githubusercontent.com/diafygi/acme-tiny/master/acme_tiny.py
$ python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir /opt/nginx/dp/challenge/ > ./signed.crt
Parsing account key...
Parsing CSR...
Registering account...
Already registered!
Verifying dp.imaicloud.com...
dp.imaicloud.com verified!
Signing certificate...
Certificate signed!
```
签名后的证书是signd.crt。
从Let's Encrypt网站下载中间证书、根证书，组合成证书链。nginx的配置需要两种证书链，一个是中间证书链，一个是全证书链。
```
$ wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem
$ cat signed.crt intermediate.pem > chained.pem
$ wget -O - https://letsencrypt.org/certs/isrgrootx1.pem > root.pem
$ cat intermediate.pem root.pem > full_chained.pem
```
配置nginx，以便处理https请求：
```
    server {
        listen               443 ssl;
        server_name dp.imaicloud.com;
        root dp;
        index index.html index.htm;

        # 中间证书 + 站点证书
        ssl_certificate      /root/ssl/chained.pem;
        # 创建 CSR 文件时用的密钥
        ssl_certificate_key  /root/ssl/domain.key;
        # 根证书 + 中间证书
        # https://imququ.com/post/why-can-not-turn-on-ocsp-stapling.html
        ssl_trusted_certificate    /root/ssl/full_chained.pem;
        location / {
            proxy_pass               http://10.10.250.221:8080;
        }
    }
    server {
        listen 80;
        server_name dp.imaicloud.com;
        root dp;
        index index.html index.htm;

        location ^~ /.well-known/acme-challenge/ {
                alias /opt/nginx/dp/challenge/;
                try_files $uri =404;
        }
        location / {
            rewrite ^/(.*)$ https://dp.imaicloud.com/$1 permanent;
        }
    }
```
#### Let's Encrypt证书链
下图说明了Let's Encrypt的证书链：  
![](https://letsencrypt.org/certs/isrg-keys.png)  
上图有三行，分别是根证书、中间证书、按申请颁发的证书。对应上文中的`root.pem`、`intermediate.pem`、`signd.crt`。可以下openssl命令查看一下这三张证书的主体(Subject)和颁发者(Issuer)。
```
$ openssl x509 -noout -text -in <cert-file>
```
下面是上述命令的输出内容摘录：
```
intermediate.pem
        Issuer: O=Digital Signature Trust Co., CN=DST Root CA X3
        Subject: C=US, O=Let's Encrypt, CN=Let's Encrypt Authority X3
root.pem
        Issuer: C=US, O=Internet Security Research Group, CN=ISRG Root X1
        Subject: C=US, O=Internet Security Research Group, CN=ISRG Root X1
signed.crt
        Issuer: C=US, O=Let's Encrypt, CN=Let's Encrypt Authority X3
        Subject: CN=dp.imaicloud.com
```
有趣的是letsencrypt.org网站本身的证书是另外的证书链：
```
letsencrypt.org
 0 s:/CN=letsencrypt.org/O=INTERNET SECURITY RESEARCH GROUP/L=Mountain View/ST=California/C=US
 1 s:/C=US/O=IdenTrust/OU=TrustID Server/CN=TrustID Server CA A52
 2 s:/C=US/O=IdenTrust/CN=IdenTrust Commercial Root CA 1
   i:/O=Digital Signature Trust Co./CN=DST Root CA X3
```

## 命令备忘
### openssl
#### 生成私钥和证书
```
$ openssl req -new -x509 -keyout <key-file> -out <cert-file> -days 365 -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=iMaiCA"
```
生成的证书是自签名的(?)。CA的根证书就是这样生成的。
#### 生成证书签名请求CSR
```
$ openssl req -new -newkey rsa:2048 -nodes -keyout <key-file> -out <CSR-file> -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=c7304.ambari.apache.org"
```
`-nodes`表示私钥无密码保护
#### CA对证书签名请求CSR进行签署
```
$ openssl x509 -req -CA <ca-cert> -CAkey <ca-key-file> -in <csr-file> -out <cert-signed-file> -days 1800 -CAcreateserial -passin pass:<passwd>
```
建立CA后，签署CSR变得更简单：
```
$ openssl ca -in <csr-file> -out <cert-signed-file>
```
建立CA，本质上是在配置文件中增加了很多默认配置，从而导致命令行变短。
#### 查看证书内容
```
$ openssl x509 -noout -text -in <cert-file>
```
#### 查看服务器的公钥证书
```
$ openssl s_client -connect <host-domain-name>:<port> | tee logfile
```
#### 模拟https服务器
```
$ openssl s_server -accept <port> -cert <server-cert-file> -key <server-key-file> -www
```
#### 创建pkcs12格式密钥库
```
$ openssl pkcs12 –export –out <keystore-file> –inkey <private-key-file> –in <cert-file> –certfile <ca-cert-file>
```
### keytool
[官方文档](http://docs.oracle.com/javase/7/docs/technotes/tools/solaris/keytool.html)  
#### 显示密钥库中的条目
```
$ keytool -list -keystore <keystore-file> -alias <alias> -storepass <password> -v
```
#### 显示证书内容
```
$  keytool -printcert -file <cert-file>
```
#### 将证书导入可信密钥库
```
$ keytool -import -trustcacerts -keystore <storefile> -alias <alias> -file <certReplyFile>
```
#### 删除条目
```
$ keytool -delete -keystore <keystore-file> -alias <alias> -storepass <password>
```
#### 密钥库合并
这个功能常用于导入私钥到密钥库。  
```
$  keytool -importkeystore -deststorepass <password> -destkeystore <destkeystore-file> -srckeystore <source-keystore-file> -srcstoretype PKCS12 -srcstorepass <password> -alias <alias>
```
源密钥库一般是pkcs12格式，而且一般由openssl命令生成：
```
$ openssl pkcs12 -export -in <cert-file> -inkey <key-file> -out <pkcs12-file> -name <alias>
```
#### 生成私钥和证书
本文中没用到这个命令，而是用openssl命令生成的公私钥对（公钥和证书两个术语容易混淆，证书是公钥的一种常见封装格式）。如果不明确指定RSA算法，默认生成DSA私钥公钥对。  
```
$ keytool -genkey -alias signLegal -keystore examplestanstore2 -validity 1800 -keyalg RSA
```
#### 导出证书
本文也没用到这个命令。不加`-rfc`生成二进制CER证书，加上`-rfc`生成文本PEM格式证书。PEM格式更常用。  
```
$ keytool -export -keystore <keystore-file> -alias <alias> -file <cert-file> -rfc
```
### curl命令
使用自定义可信证书库访问https主机：
```
$ curl <url>  --cacert <truststore-file>
```
双向SSL：
```
$ curl <url> --cacert <truststore-file> --cert <cert-file>
```
`--cert`参数接受pem格式文件，jks格式不行。生成pem格式文件的方式（就是把两个文本文件拼接在一起）：
```
$ cat <cert-file> <key-file> > <key-cert-file>
```

## 相关文档
[java结合keytool实现非对称签名与验证](https://imaidata.github.io/blog/2017/07/29/java%E7%BB%93%E5%90%88keytool%E5%AE%9E%E7%8E%B0%E9%9D%9E%E5%AF%B9%E7%A7%B0%E7%AD%BE%E5%90%8D%E4%B8%8E%E9%AA%8C%E8%AF%81/)   
[java结合keytool实现非对称加密和解密](https://imaidata.github.io/blog/2017/08/08/java%E7%BB%93%E5%90%88keytool%E5%AE%9E%E7%8E%B0%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E5%92%8C%E8%A7%A3%E5%AF%86/)  
[java使用SPNEGO认证](https://imaidata.github.io/blog/java_spnego/)   
[JAAS认证](https://imaidata.github.io/blog/jaas/)   