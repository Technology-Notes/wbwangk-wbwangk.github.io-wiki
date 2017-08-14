- 一、[基础知识](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E4%B8%80%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86)  
- 二、[java访问https服务器](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E4%BA%8Cjava%E8%AE%BF%E9%97%AEhttps%E6%9C%8D%E5%8A%A1%E5%99%A8)  
- 三、[建立https服务器](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E4%B8%89%E5%BB%BA%E7%AB%8Bhttps%E6%9C%8D%E5%8A%A1%E5%99%A8)  
- 四、[双向SSL](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E5%9B%9B%E5%8F%8C%E5%90%91ssl)  
- 五、[hadoop集群启用SSL](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E4%BA%94hadoop%E9%9B%86%E7%BE%A4%E5%90%AF%E7%94%A8ssl)  
- 六、[hadoop集群启用SSL(letsencrypt证书)](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E5%85%ADhadoop%E9%9B%86%E7%BE%A4%E5%90%AF%E7%94%A8sslletsencrypt%E8%AF%81%E4%B9%A6)

附1、[建立内部CA](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E9%99%841%E5%88%9B%E5%BB%BA%E5%86%85%E9%83%A8ca)    
附2、[申请Let's Encrypt证书](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E9%99%842%E7%94%B3%E8%AF%B7lets-encrypt%E8%AF%81%E4%B9%A6)  
附3、 [命令备忘](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E9%99%843%E5%91%BD%E4%BB%A4%E5%A4%87%E5%BF%98)  
## 一、基础知识
### (一)术语
#### 私钥/公钥对
同时生成的两个字符串。私钥用于签名，公钥验证签名。公钥加密消息，私钥可以解密。  

#### 证书(certificate)
(公钥)证书封装了公钥及配套信息，如公钥拥有者(subject)、签署者的签名、签署者(issuer)。自签名证书的subject与issuer相同。根证书是自签名证书。  
公钥证书可以被认为是护照的数字等价物。它由可信任的组织颁发，可为持有人提供身份证明。发布公钥证书的受信任组织称为证书颁发机构（CA）。  

自己的私钥给自己的公钥签名，叫自签名证书。更普遍的CA签名的证书。  
([X.509证书细节](http://www.cnblogs.com/chnking/archive/2007/08/28/872104.html))  

#### 密钥库(keystore)
将密钥和对应的证书保存在一个受密码保护的数据库中，就叫密钥库。密钥库格式有JKS、PCKS12等。java默认支持jks。jks中保存两种类型条目：PrivateKeyEntry和trustedCertEntry。前者是私钥，后者是可信证书。  

#### 可信证书库(truststore)
SSL单向认证依赖可信证书库。如IE浏览器中内置了主要证书颁发机构(CA)的根证书和中间人证书，形成了IE的可信证书库。JDK、CURL等都自带可信证书库。java自带可信证书库的格式是jks，里面的条目类型是trustedCertEntry。  

#### 证书签名请求(CSR)
CSR是Certificate Signing Request的简写。某实体要获得CA认可，需要利用自己的私钥/公钥对构造一个证书签名请求文件发给CA。  

#### 证书链
多个证书可以链接成**证书链**(java中叫证书路径)。当使用证书链时，第一个证书始终是主体的证书。接下来是颁发发件人证书的实体的证书。如果链中有更多证书，那么每个证书都是颁发先前证书的上级CA。链中的最终证书是根CA的证书。  

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

现代浏览器都内嵌了一系列可信CA的公钥证书。如果你访问一个不可信的https网站（如证书是自签名的），浏览器会弹出警告，只有把要访问的网站加入“例外”目录，浏览器才运行继续访问。Java也实现了类似机制。JDK自带了一个jks格式的可信证书库文件cacerts。Oracle JDK和OpenJDK的cacerts文件的路径不同。
- OracleJDK自带可信证书库文件是`$JAVA_HOME/jre/lib/security/cacerts`  
- OpenJDK自带可信证书库的文件是`/etc/pki/java/cacerts`  

可以这样查找可信证书库位置：
```
$  find / -name cacerts
/etc/pki/ca-trust/extracted/java/cacerts
/etc/pki/java/cacerts
/usr/java/jdk1.8.0_131/jre/lib/security/cacerts
```
前两个是同一个文件，是OpenJDK的可信证书库(不装OpenJDK也有这个文件)。第三个OracleJDK的可信证书库。  
可以用`keytool`命令查看该文件内容(初始口令是changeit)：  
```
$ keytool -list -keystore <cacerts-file> -storepass changeit
```
把`<cacerts-file>`替换成cacerts文件的真实路径。  

cacerts中已经内置了常见的公共CA证书。当java程序访问https网站时，jre会利用cacerts来检验https网站是否可信。  
下面的测试在节点c7304上进行，操作系统centos7.3，JDK是Oracle JDK 1.8。测试目录`/opt/https`。  

### java访问https网站
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
       sslContext.init(null, null, null);

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
在上述代码中，需要关注是的`sslContext.init(null, null, null);`这行代码。第2个参数代表“可信证书库”，null表示使用默认的可信证书库(即cacerts)。至于第1个参数，双向SSL会用到。  
编译运行：
```
$ javac HttpsTest.java
$ java HttpsTest https://baidu.com                         (正常执行)
$ java HttpsTest https://kyfw.12306.cn                     (报错)
```
`kyfw.12306.cn`报错是因为它的证书不是*公共CA*签发的。  

#### 自建信任库
首先，查询kyfw.12306.cn的公钥：
```
$ openssl s_client -connect kyfw.12306.cn:443
```
将显示在屏幕上的`-----BEGIN CERTIFICATE-----`和`-----END CERTIFICATE-----`之间的内容（包括这两行）复制到一个文件(12306.crt)中。  
执行下列keytool命令后会生成文件trust.jks，`-trustcacerts`参数保证了导入的条目类型是“可信证书”：
```
$ keytool -import -trustcacerts -keystore trust.jks -alias 12306 -file 12306.crt -storepass vagrant
$ java -Djavax.net.ssl.trustStore=trust.jks HttpsTest https://kyfw.12306.cn      (正常执行)
```
自制信任库中只有`kyfw.12306.cn`网站的证书，访问其他网站会报错，如：
```
$ java -Djavax.net.ssl.trustStore=trust.jks HttpsTest https://baidu.com         (报错)
```

#### 导入java信任库
用keytool将12306.crt导入到JDK的信任库：
```
$ keytool -import -trustcacerts -keystore /usr/java/jdk1.8.0_131/jre/lib/security/cacerts -alias 12306 -file 12306.crt -storepass changeit
```
重新发送请求：
```
$ java HttpsTest https://kyfw.12306.cn
```
不再报错。说明`kyfw.12306.cn`已经在JDK的信任库中，JDK允许客户端发送请求到这个主机。  


#### 小结
java用以下优先级处理可信证书库：
1. `sslContext.init()`的第2个参数传入优先级最高。这个在讲“双向SSL”的章节中有说明。
2. `-Djavax.net.ssl.trustStore`系统属性传入优先级次之
3. JDK默认可信证书库(cacerts)优先级最低

### HttpClient访问https网站
利用apache [HttpClient](https://hc.apache.org/)访问https的写法略有差异，但可信证书库的位置、作用等于java完全相同。下面测试在OracleJDK下进行。  
```
$ wget http://mirror.bit.edu.cn/apache//httpcomponents/httpclient/binary/httpcomponents-client-4.5.3-bin.tar.gz
$ tar xzvf httpcomponents-client-4.5.3-bin.tar.gz
$ keytool -delete -keystore /usr/java/jdk1.8.0_131/jre/lib/security/cacerts -alias 12306 -storepass changeit
```
删除JDK可信证书库中的`12306`条目是为了下面的测试。  

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
$ javac -cp ".:/opt/https/httpcomponents-client-4.5.3/lib/*" HttpClientSSL.java           (编译)
$ java -cp ".:/opt/https/httpcomponents-client-4.5.3/lib/*" HttpClientSSL https://cn.bing.com      (正常执行)
----------------------------------------
HTTP/1.1 200 OK
$ java -cp ".:/opt/https/httpcomponents-client-4.5.3/lib/*" HttpClientSSL https://kyfw.12306.cn     (报错)
$ java -Djavax.net.ssl.trustStore=trust.jks -cp ".:/opt/https/httpcomponents-client-4.5.3/lib/*" HttpClientSSL https://kyfw.12306.cn                      (正常执行)
```
#### 小结
HttpClient的可信证书库规则与纯java编程相同。

## 三、建立https服务器
[参考](https://www.digitalocean.com/community/tutorials/how-to-create-an-ssl-certificate-on-nginx-for-ubuntu-14-04)  

增加一个文本文件`/etc/yum.repos.d/nginx.repo`，内容是：
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
`-nodes`表示密钥不加密。  
编辑nginx配置文件`/etc/nginx/conf.d/default.conf`，添加以下内容：
```
server {
    listen       442 ssl;
    server_name  c7304.ambari.apache.org;

                ssl_certificate      /opt/https/nginx.crt;               (证书)
                ssl_certificate_key  /opt/https/nginx.key;               (私钥)

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
```
注意使用的442端口，而不是https默认的443端口。index.html文件没有就手工需要一个。  
重启nginx，加载新的配置文件，测试https服务器：
```
$ service nginx start           (刚装上nginx时要执行，之后就不用了)
$ nginx -t                      (测试nginx配置文件)
$ nginx -s reload               (重新加载配置文件，修改nginx配置文件有执行这个命令)
```
可以用前面的java程序测试一下https：
```
$ java HttpsTest https://c7304.ambari.apache.org:442               (报错)
```
必然报错，因为服务器证书没加入到可信证书库中。

#### 将公钥导入JDK可信证书库
```
$ keytool -import -trustcacerts -keystore /usr/java/jdk1.8.0_131/jre/lib/security/cacerts -alias nginx -file nginx.crt -storepass changeit
```
有时要多次导入同名条目。如果提示重名，则删除条目的命令：
```
$ keytool -delete -keystore /usr/java/jdk1.8.0_131/jre/lib/security/cacerts -alias nginx -storepass changeit
```
重新用java程序测试：
```
$ java HttpsTest https://c7304.ambari.apache.org:442                  (正常执行)
```
可以看到，不报错了。说明nginx的公钥证书导入到JDK可信证书库的操作起作用了。  

#### 用curl进行可信https测试
```
$ curl https://c7304.ambari.apache.org:442  --cacert /opt/ca/nginx.crt              (不报错，返回了网页)
```  
curl增加`--cacert`参数后，相当于把参数后的证书临时加入了curl的可信库。  
注意，curl的--cacert参数接受的是PEM格式的证书。把JKS格式的密钥库当参数传给curl是不行的，如`/etc/pki/java/cacerts`当curl参数不行。  
curl还有一个`-k`参数可以强制关闭可信检测：
```
$ curl -k https://c7304.ambari.apache.org                           (正常显示)
```
### (二)CA签名证书https服务器
要获得一个CA签名的nginx公钥证书，除了通过免费`let's encrypt`网站或商用CA证书公司外，还可以自己搭建一个“内部CA”。搭建办法参见附录。 
 
除了搭建正式的CA外，还可以自建一个临时CA。其实就是生成一个自签名的证书来充当CA根证书：
```
$ openssl req -new -newkey rsa:2048 -x509 -keyout ca.key -out ca.crt -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=TempCA"
```
为了CA签名，先生成一个私钥和证书签名请求：
```
$ openssl req -new -newkey rsa:2048 -nodes -keyout nginx2.key -out nginx2.csr -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=c7304.ambari.apache.org"
```
命令格式同前文的自签名相比，去掉了一个`-x509`。输出的文件命名也改了.csr。生成好后可以cat命令看一下.csr文件的内容，发现它的开始一行是`-----BEGIN CERTIFICATE REQUEST-----`，而证书的开始行是`-----BEGIN CERTIFICATE-----`。  

然后，是自建CA处理证书签名请求(CSR)，会提示输入CA私钥密码：
```
$ openssl x509 -req -CA ca.crt -CAkey ca.key -in nginx2.csr -out nginx2.crt -days 365 -CAcreateserial -passin pass:vagrant
```
可以用下列命令查看一下签名后的证书，能看到Issuer和Subject不一样了：
```
$ openssl x509 -noout -text -in nginx2.crt
```
注意，在下一章讲双向SSL时，也用这个临时CA对客户端证书进行签名。  

#### 配置nginx
修改nginx的配置文件(`/etc/nginx/conf.d/default.conf`)，添加以下内容：
```
server {
    listen       444 ssl;
    server_name  c7304.ambari.apache.org;
    ssl_certificate        /opt/https/nginx2.crt;
    ssl_certificate_key    /opt/https/nginx2.key;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```
使用端口444来测试自建CA签名证书搭建的https服务器。重新加载nginx配置文件：
```
$ nginx -s reload
```
#### 测试https服务器
首先，用curl测试。需要说明的是，对于CA签名的证书，客户端需要信任是CA的公钥证书，而不是https服务器自己的。所以，curl的命令是：
```
$ curl https://c7304.ambari.apache.org  --cacert ca.crt            (正常执行)
```
用之前java程序测试，先要把CA证书导入自建可信证书库：
```
$  keytool -import -trustcacerts -keystore trust.jks -alias TempCA -file ca.crt -storepass vagrant
$ java -Djavax.net.ssl.trustStore=trust.jks HttpsTest https://c7304.ambari.apache.org:444     (正常执行)
```
下面删除`TempCA`这个条目，改成导入nginx2.crt：
```
$ keytool -delete -keystore trust.jks -alias TempCA -storepass vagrant
$ keytool -import -trustcacerts -keystore trust.jks -alias nginx2 -file nginx2.crt -storepass vagrant
$ java -Djavax.net.ssl.trustStore=trust.jks HttpsTest https://c7304.ambari.apache.org:444     (正常执行)
```
无论导入https服务器自身的证书，还是为它签名的CA的证书**都可以**。这与CURL有显著不同。  

把https服务器证书导入JDK官方信任库的测试就不做了。  
#### 导入IE测试
将ca.crt、nginx.cr、nginx2.crt复制到windows下，然后导入IE的“受信任的发布者”证书。但访问`https://c7304.ambaria.apache.org:442`或`https://c7304.ambaria.apache.org:444`都报告“此网站的安全证书存在问题”。

### (三)OpenSSL搭建https服务器
openssl自带了https服务器的模拟功能：
```
$ cd /opt/ca
$ openssl s_server -accept 445 -cert nginx2.crt -key nginx2.key -www
```
这个openssl的模拟服务器用curl测试：
```
$ curl https://c7304.ambari.apache.org:445  --cacert ca.key
```
java测试不再冗述。  


## 四、双向SSL
参考：[Nginx实现双向SSL](http://nategood.com/client-side-certificate-authentication-in-ngi)、[双向SSL15分钟教程](http://monduke.com/2006/06/04/the-fifteen-minute-guide-to-mutual-authentication/)  
双向SSL(two-way SSL)又叫Mutual Authentication，或基于证书的相互认证，是指双方通过验证提供的数字证书相互认证，以便双方确保对方的身份。  
认证过程的详细描述可参考[这个](https://community.developer.visa.com/t5/Developer-Tools/What-is-Mutual-Authentication/ba-p/5757)。  

验证单向SSL时已经为服务器生成了私钥(nginx2.key)和公钥证书(nginx2.crt)。为了实现双向SSL，还需要为客户端生成私钥和证书。利用目录`/opt/twowayssl`为客户端用户webb生成私钥(client.key)和公钥证书(client.crt)。（服务器私钥和证书位于`/opt/https`）首先，创建签名请求CSR，以便供CA签名：
```
$ /opt/twowayssl
$ openssl req -new -newkey rsa:2048 -nodes -keyout client.key -out client.csr -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=webb"
```
利用上一章创建的临时CA对这个CSR进行签名(临时CA文件位于`/opt/https`目录下)：
```
$ openssl x509 -req -CA /opt/https/ca.crt -CAkey /opt/https/ca.key -in client.csr -out client.crt -days 365 -CAcreateserial -passin pass:vagrant
```
修改nginx的配置文件(`/etc/nginx/conf.d/default.conf`)，添加以下内容：
```
server {
    listen       443 ssl;
    server_name  c7304.ambari.apache.org;
    ssl_certificate        /opt/https/nginx2.crt;
    ssl_certificate_key    /opt/https/nginx2.key;

    ssl_client_certificate /opt/https/ca.crt;
    ssl_verify_depth 1;                 
    ssl_verify_client on;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```
(总结一下SSL端口：442是自签名单向SSL、443是双向SSL、444是CA签名单向SSL)

指令`ssl_verify_depth 1`必须有。应该是证书链的验证深度，0表示客户端证书，1表示签署客户端证书的CA。虽然没有做实验，但估计如果客户端证书是自签名的，则深度0（默认值）就可以。  
用`nginx -s reload`重启nginx。

#### curl测试双向SSL
```
$ curl -k https://c7304.ambari.apache.org                    (报错400 No required SSL certificate was sent)
```
注意到返回的状态码不再是`200`。这说明nginx双向SSL的配置是起作用了。`-k`参数禁用了服务器可信检测，但由于没有提供客户端凭据，仍报错。  
为了用curl测试双向SSL，先将客户端的私钥和证书合并：
```
$ cat client.crt client.key > client.pem
$ curl -k --cert ./client.pem https://c7304.ambari.apache.org         (正常返回网页)
$ curl https://c7304.ambari.apache.org --cacert /opt/https/ca.crt --cert ./client.pem        (正常返回网页)
```
注意[--cert](https://curl.haxx.se/docs/manpage.html#-E)参数如果后面跟文件，必须加上相对或绝对路径，否则后面的参数会当成NSS数据库的nickname。  

#### windows下的双向SSL测试
要想把个人私钥证书导入到IE，需要用openssl把client.crt和client.key合并转化成pk12格式：
```
$ openssl pkcs12 -export -in client.crt -inkey client.key -out client.p12 -name webb -passout pass:vagrant
```
将生成的client.p12文件复制到宿主windows下。然后分别导入到IE。client.p12导入到了“个人”标签页(显示颁发给webb)，按提示输入密码vagrant。需要在IE->internet选项->内容->证书->“高级”按钮中选中“用于客户端认证”。  
然后用IE访问地址`https://c7304.ambari.apache.org`，会有确认提示，确认后就正常显示网页了。  

### java下的双向SSL
[使用keytool将私钥导入到Java密钥库中](http://cunning.sharp.fm/2008/06/importing_private_keys_into_a.html)  
前文用openssl工具生成了客户端私钥和公钥证书。要利用实现java程序的双向SSL访问服务器，首先需要把客户端私钥导入到密钥库keystore。keytool没有直接导入私钥的功能，但提供了密钥库合并功能，可以利用这个功能实现密钥导入keystore。  
```
$ cd /opt/twowayssl
$ keytool -importkeystore -deststorepass vagrant -destkeystore client.jks -srckeystore client.p12 -srcstoretype PKCS12 -srcstorepass vagrant -alias webb              (为客户端创建私钥库client.jks，导入了client.p12)
$ keytool -list -keystore client.jks -alias webb -storepass vagrant
webb, Aug 4, 2017, PrivateKeyEntry,
Certificate fingerprint (SHA1): 99:6D:E2:E4:ED:46:91:8C:FC:D6:73:EC:42:74:3C:BF:6E:E8:9F:87
```
由于之前client.jsk并不存在，openssl会先创建一个空库，然后再合并。  
测试类为[MutualAuthenticationHTTP.java](https://raw.githubusercontent.com/imaidata/blog/master/_posts/kerberos/MutualAuthenticationHTTP.java)。这个类比较长，不再全部贴在文章里了。下面是与证书相关的定义：
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
$ keytool -import -trustcacerts -keystore /usr/java/jdk1.8.0_131/jre/lib/security/cacerts -alias nginx2 -file /opt/https/ca.crt -storepass chengeit           (临时CA的根证书导入到JDK可信证书库)
$ javac MutualAuthenticationHTTP.java
$ java MutualAuthenticationHTTP             (正常返回了nginx欢迎页)
```
程序先打印出了密钥库的条目，又打印出了可信库的条目，最后显示了URL的响应。这说明java程序通过了服务器的双向认证。
注意一下`SSLContext`的初始化写法：
```
    context.init(keyManagers, trustManagers, null);
```
在单向SSL测试使用的`HttpsTest.java`类中，这个初始化方法使用了3个null参数。其实，在`MutualAuthenticationHTTP`类的`context.init`方法中，第2个参数也可以是null，这表示使用JDK默认可信库，而在本例中传入的可信库参数恰好就是默认可信库。  

#### 更深入的测试
重申一下java用以下优先级处理可信证书库：
- sslContext.init()的第2个参数传入优先级最高。
- -Djavax.net.ssl.trustStore系统属性传入优先级次之
- JDK默认可信证书库(cacerts)优先级最低
现在修改代码，不再方法中传入可信证书库：
```
    context.init(keyManagers, null, null);
```
重新执行，仍可以正常执行：
```
$ javac MutualAuthenticationHTTP.java
$ java MutualAuthenticationHTTP             (正常返回了nginx欢迎页)
```
如果想测试从系统属性传入自定义的可信证书库trust.jks，则需要将`/opt/https/ca.crt`倒入到`/opt/https/trust.jks`:
```
$ keytool -import -trustcacerts -keystore /opt/https/trust.jks -alias TempCA -file /opt/https/ca.crt -storepass vagrant
$ java -Djavax.net.ssl.trustStore=/opt/https/trust.jks MutualAuthenticationHTTP          (执行正常)
```

## 五、hadoop集群启用SSL
[官方文档](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_security/content/create-internal-ca.html)，本文参考了[这篇社区文章](https://community.hortonworks.com/articles/22756/quickly-enable-ssl-encryption-for-hadoop-component.html)   
本章首先尝试把HDP集群的Ambari界面升级为SSL（非双向SSL）。然后把HDP集群中几个常见服务(HDFS、HBASE等)升级为SSL。为了便于调试，本章把[原文](https://github.com/vzlatkin/EnableSSLinHDP)的脚本(enable-ssl.sh)分拆为几个小脚本文件。赋予脚本文件可执行权限：
```
$ chmod +x ssl1.sh
$ ./ssl1.sh
```

#### 0.创建临时CA
在c7301节点创建临时CA。假定c7301可以免密码ssh到c7301、c7302、c7303。工作目录是`/opt/ca`:
```
$ openssl genrsa -out ca.key 2048
$ openssl req -new -x509 -days 1826 -key ca.key -out ca.crt -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=AmbariCA"
```
生成临时CA的私钥`ca.key`和公钥证书`ca.crt`。主体是`AmbariCA`。

#### 1.为各节点创建证书
生成各个节点的私钥和证书签名请求CSR。用临时CA回应CSR，生成各个节点的证书。创建`ssl1.sh`：
```
#!/usr/bin/env bash
server1="c7301.ambari.apache.org"
server2="c7302.ambari.apache.org"
server3="c7303.ambari.apache.org"

for host in ${ALL_REAL_SERVERS}; do
    if [  -e "${host}.crt" ]; then break; fi
    openssl req -new -newkey rsa:2048 -nodes -keyout ${host}.key -out ${host}.csr  -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=$host"
    openssl x509 -req -CA ca.crt -CAkey ca.key -out ${host}.crt -in ${host}.csr -days 365 -CAcreateserial
done
```
确保公用名称（CN）与服务器的完全限定域名（FQDN）匹配。本例中，c7302节点的FQDN是`c7302.ambari.apache.org`。客户端将CN与DNS域名进行比较，以确保它确实连接到所需的服务器，而不是恶意服务器。  

#### 2.创建各节点keystore
将CA公钥复制到各个节点，导入各节点可信证书库。各个节点的可信证书库往往有两个：OracleJDK的和OpenJDK的。拿不准就都导进去。创建ssl2.sh:
```
#!/usr/bin/env bash
server1="c7301.ambari.apache.org"
server2="c7302.ambari.apache.org"
server3="c7303.ambari.apache.org"
#TRUST_STORE=/etc/pki/java/cacerts
TRUST_STORE=/usr/jdk64/jdk1.8.0_112/jre/lib/security/cacerts

#copy public ssl certs to all hosts
for host in ${ALL_REAL_SERVERS}; do
    scp ca.crt ${host}:/tmp/ca.crt
    ssh $host "keytool -import -noprompt -alias myOwnCA -file /tmp/ca.crt -storepass changeit -keystore $TRUST_STORE; rm -f /tmp/ca.crt"

    for cert in ${ALL_REAL_SERVERS}; do
        scp $cert.crt ${host}:/tmp/$cert.crt
        ssh $host "keytool -import -noprompt -alias ${cert} -file /tmp/${cert}.crt -storepass changeit -keystore $TRUST_STORE; rm -f \"/tmp/${cert}.crt\""
    done
done
```
可信证书库是JSSE客户端使用的，用于访问SSL服务器，而不是创建SSL服务器。  
TRUST_STORE是OracleJDK的可信证书库。如果你HDP部署使用OpenJDK，需要更换成注释掉的TRUST_STORE。

#### 3.Ambari服务器启用SSL
Ambari启用https使用的交互界面，如果变成脚本需要安装额外工具(expect)。也可以不使用脚本，而直接运行ambari-server命令来启用https。创建脚本文件为ssl3.sh:
```bash
#!/usr/bin/env bash
server1="c7301.ambari.apache.org"
export AMBARI_SERVER=$server1

#TRUST_STORE=/etc/pki/java/cacerts
TRUST_STORE=/usr/jdk64/jdk1.8.0_112/jre/lib/security/cacerts
    rpm -q expect || yum install -y expect
    cat <<EOF > ambari-ssl-expect.exp
#!/usr/bin/expect
spawn "/usr/sbin/ambari-server" "setup-security"
expect "Enter choice"
send "1\r"
expect "Do you want to configure HTTPS"
send "y\r"
expect "SSL port"
send "\r"
expect "Enter path to Certificate"
send "/opt/ca/\$env(AMBARI_SERVER).crt\r"
expect "Enter path to Private Key"
send "/opt/ca/\$env(AMBARI_SERVER).key\r"
expect "Please enter password for Private Key"
send "\r"
send "\r"
interact
EOF

    cat <<EOF > ambari-truststore-expect.exp
#!/usr/bin/expect
spawn "/usr/sbin/ambari-server" "setup-security"
expect "Enter choice"
send "4\r"
expect "Do you want to configure a truststore"
send "y\r"
expect "TrustStore type"
send "jks\r"
expect "Path to TrustStore file"
send "/etc/pki/java/cacerts\r"
expect "Password for TrustStore"
send "changeit\r"
expect "Re-enter password"
send "changeit\r"
interact
EOF
    if ! grep -q 'api.ssl=true' /etc/ambari-server/conf/ambari.properties; then
        /usr/bin/expect ambari-ssl-expect.exp
            /usr/bin/expect ambari-truststore-expect.exp

        service ambari-server restart

        while true; do if tail -100 /var/log/ambari-server/ambari-server.log | grep -q 'Started Services'; then break; else echo -n .; sleep 3; fi; done; echo
    fi

    rm -f ambari-ssl-expect.exp  ambari-truststore-expect.exp
    #validate wget -O-  --no-check-certificate "https://${AMBARI_SERVER}:8443/#/main/dashboard/metrics"
```
配置ambari的TrustStore的目的需要解释一下，这是为了ambari作为SSL客户端访问其他启用SSL后的hadoop服务。如果仅仅是ambari自己启用SSL，可以不为ambari配置TrustStore。  

这一步执行后，已经可以用https访问ambari了。访问地址是：`https://c7301.ambari.apache.org:8443`。

#### 4.对hadoop启用SSL
创建了一个`hadoopSSL.sh`脚本：
```
#!/usr/bin/env bash

server1="c7301.ambari.apache.org"
server2="c7302.ambari.apache.org"
server3="c7303.ambari.apache.org"

NAMENODE_SERVER_ONE=$server1
RESOURCE_MANAGER_SERVER_ONE=$server2
HISTORY_SERVER=$server2
ALL_NAMENODE_SERVERS="${NAMENODE_SERVER_ONE} $server2"
ALL_HADOOP_SERVERS="$server1 $server2 $server3"

export AMBARI_SERVER=$server1
AMBARI_PASS=admin
CLUSTER_NAME=HDP2610

#TRUST_STORE=/etc/pki/java/cacerts
TRUST_STORE=/usr/jdk64/jdk1.8.0_112/jre/lib/security/cacerts
#
# Enable Hadoop UIs SSL encryption. Stop all Hadoop components first
#
function hadoopSSLEnable() {

    for host in ${ALL_HADOOP_SERVERS}; do
        if [ -e "${host}.p12" ]; then continue; fi
        openssl pkcs12 -export -in ${host}.crt -inkey ${host}.key -out ${host}.p12 -name ${host} -CAfile ca.crt -chain -passout pass:vagrant
    done

    for host in ${ALL_HADOOP_SERVERS}; do
        scp ${host}.p12 ${host}:/tmp/${host}.p12
        scp ca.crt ${host}:/tmp/ca.crt
        ssh $host "
            keytool -import -noprompt -alias myOwnCA -file /tmp/ca.crt -storepass vagrant -keypass vagrant -keystore /etc/hadoop/conf/hadoop-private-keystore.jks
            keytool --importkeystore -noprompt -deststorepass vagrant -destkeypass vagrant -destkeystore /etc/hadoop/conf/hadoop-private-keystore.jks -srckeystore /tmp/${host}.p12 -srcstoretype PKCS12 -srcstorepass vagrant -alias ${host}
            chmod 440 /etc/hadoop/conf/hadoop-private-keystore.jks
            chown yarn:hadoop /etc/hadoop/conf/hadoop-private-keystore.jks
            rm -f /tmp/ca.crt \"/tmp/${host}.p12\";
            "
    done

    cat <<EOF | while read p; do p=${p/,}; p=${p//\"}; if [ -z "$p" ]; then continue; fi; /var/lib/ambari-server/resources/scripts/configs.sh -u admin -p $AMBARI_PASS -port 8443 -s set $AMBARI_SERVER $CLUSTER_NAME $p &> /dev/null || echo "Failed to change $p in Ambari"; done
        hdfs-site "dfs.https.enable"   "true",
        hdfs-site "dfs.http.policy"   "HTTPS_ONLY",
        hdfs-site "dfs.datanode.https.address"   "0.0.0.0:50475",
        hdfs-site "dfs.namenode.https-address"   "0.0.0.0:50470",

        core-site "hadoop.ssl.require.client.cert"   "false",
        core-site "hadoop.ssl.hostname.verifier"   "DEFAULT",
        core-site "hadoop.ssl.keystores.factory.class"   "org.apache.hadoop.security.ssl.FileBasedKeyStoresFactory",
        core-site "hadoop.ssl.server.conf"   "ssl-server.xml",
        core-site "hadoop.ssl.client.conf"   "ssl-client.xml",

        mapred-site "mapreduce.jobhistory.http.policy"   "HTTPS_ONLY",
        mapred-site "mapreduce.jobhistory.webapp.https.address"   "${HISTORY_SERVER}:19443",
        mapred-site mapreduce.jobhistory.webapp.address "${HISTORY_SERVER}:19443",

        yarn-site "yarn.http.policy"   "HTTPS_ONLY"
        yarn-site "yarn.log.server.url"   "https://${HISTORY_SERVER}:19443/jobhistory/logs",
        yarn-site "yarn.resourcemanager.webapp.https.address"   "${RESOURCE_MANAGER_SERVER_ONE}:8090",
        yarn-site "yarn.nodemanager.webapp.https.address"   "0.0.0.0:45443",

        ssl-server "ssl.server.keystore.password"   "vagrant",
        ssl-server "ssl.server.keystore.keypassword"   "vagrant",
        ssl-server "ssl.server.keystore.location"   "/etc/hadoop/conf/hadoop-private-keystore.jks",
        ssl-server "ssl.server.truststore.location"   "${TRUST_STORE}",
        ssl-server "ssl.server.truststore.password"   "changeit",

        ssl-client "ssl.client.keystore.location"   "${TRUST_STORE}",
        ssl-client "ssl.client.keystore.password"   "changeit",
        ssl-client "ssl.client.truststore.password"   "changeit",
        ssl-client "ssl.client.truststore.location"   "${TRUST_STORE}"
EOF
    rm -f doSet_version*
    # In Ambari, perform Start ALL

    #validate through:
}
hadoopSSLEnable
```
定义了一个函数hadoopSSLEnable，并调用它。  
首先，导出了所有节点的私钥(p12格式)，然后复制到了各个节点，同时复制的还有CA的根证书。  
然后，为HDP集群的各个节点创建密钥库(hadoop-private-keystore.jks)。密钥库中导入CA根证书，导入p12格式的私钥。  
最后调用ambari的配置脚本将https相关的参数配置到HDP集群中。当脚本执行成功后，可以ambari界面查看。  
用openssl测试一下SSL服务器：
```
$ openssl s_client -connect c7301.ambari.apache.org:50470 -showcerts
```
还可以用curl访问启用SSL的hdfs来测试：
```
$ kinit root/admin
$ curl -k --negotiate -u :  https://c7301.ambari.apache.org:50470/webhdfs/v1/user?op=LISTSTATUS
```
我的测试集群启用了kerberos，所以需要先登录KDC。注意URL是https的。  
完整的脚本[在这](https://github.com/wbwangk/EnableSSLinHDP/blob/master/enable-ssl.sh)。


## 六、hadoop集群启用SSL(letsencrypt证书)
在上一章中，通过自建CA发放证书，将hadoop集群启用了SSL。自建CA发放的证书，会被浏览器报告为“非安全网站”。如果把hadoop集群中每个节点的证书更换为letsencrypt.org方法的证书，则浏览器就不会报错了。  

### 向letsencrypt.org申请证书
申请过程与[附录2](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL%E7%A0%94%E7%A9%B6#%E9%99%842%E7%94%B3%E8%AF%B7lets-encrypt%E8%AF%81%E4%B9%A6)描述的基本一致。只是把生成CSR的命令修改为：
```
openssl req -new -sha256 -key domain.key -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=dp.imaicloud.com" -reqexts SAN -config <(cat /etc/pki/tls/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:dp.imaicloud.com,DNS:c7301.dp.imaicloud.com,DNS:c7302.dp.imaicloud.com,DNS:c7303.dp.imaicloud.com,DNS:c7304.dp.imaicloud.com,DNS:c7305.dp.imaicloud.com,DNS:c7306.dp.imaicloud.com,DNS:c7307.dp.imaicloud.com,DNS:c7308.dp.imaicloud.com,DNS:c7309.dp.imaicloud.com,DNS:c7310.dp.imaicloud.com,DNS:c7311.dp.imaicloud.com,DNS:c7312.dp.imaicloud.com,DNS:c7313.dp.imaicloud.com,DNS:c7314.dp.imaicloud.com,DNS:c7315.dp.imaicloud.com")) > domain.csr
```
上述证书签名请求包含了15个节点的FQDN。当证书申请成功后，获得的文件有：  
- domain.key，私钥  
- signed.crt，签名后的证书  
- chained.pem，含签名后的证书和中间人证书  
- full_chained.pem，含中间人证书和根证书  

### 搭建新的hadoop集群
原集群的域名和kerberos领域与从letsencrypt申请的证书不符。担心现有集群修改域名和领域带来未知问题，决定重新搭建一个hadoop集群。  
新集群有三个节点：
192.168.73.101 c7301.dp.imaicloud.com  
192.168.73.102 c7302.dp.imaicloud.com  
192.168.73.103 c7303.dp.imaicloud.com  
保证上述内容定义在三个节点的`/etc/hosts`文件中。为了在windows下测试，上述内容也要定义到windows的`c:/windows/system32/drivers/etc/hosts`文件中。  
新集群上部署ambari([参考](https://imaidata.github.io/blog/ambari_centos/))，利用ambari部署的hadoop服务有：  
HDFS、YARN、MR2、Hive、HBase、Oozie、Zookeeper等。新集群启用kerberos。  

### hadoop集群启用SSL
将上述letsencrypt证书相关的4个文件(domain.key、signed.crt、chained.pem、full_chained.pem)复制到ambari server所在节点(c7301)的`/tmp/security`目录下。 

#### 1.创建各节点keystore
将CA公钥复制到各个节点，导入各节点可信证书库。创建`/tmp/seurity/ssl2.sh`:
```
#!/usr/bin/env bash

server1="c7301.dp.imaicloud.com"
server2="c7302.dp.imaicloud.com"
server3="c7303.dp.imaicloud.com"
ALL_REAL_SERVERS="$server1 $server2 $server3"

#TRUST_STORE=/etc/pki/java/cacerts
TRUST_STORE=/usr/jdk64/jdk1.8.0_112/jre/lib/security/cacerts

#copy public ssl certs to all hosts
for host in ${ALL_REAL_SERVERS}; do
        scp full_chained.pem ${host}:/tmp/full_chained.pem
        ssh $host "keytool -import -noprompt -alias full_chained -file /tmp/full_chained.pem -storepass changeit -keystore $TRUST_STORE; rm -f /tmp/full_chained.pem"

        scp chained.pem ${host}:/tmp/chained.pem
        ssh $host "keytool -import -noprompt -alias chained -file /tmp/chained.pem -storepass changeit -keystore $TRUST_STORE; rm -f \"/tmp/chained.pem\""

done
```
full_chained.pem中包含了中间人证书和CA根证书。chained.pem中包含了从letsencypt申请的证书和中间人证书。  

#### 2.Ambari服务器启用SSL
Ambari启用https使用的交互界面，如果变成脚本需要安装额外工具(expect)。也可以不使用脚本，而直接运行ambari-server命令来启用https。创建脚本文件ssl3.sh:
```bash
#!/usr/bin/env bash
server1="c7301.dp.imaicloud.com"
export AMBARI_SERVER=$server1

#TRUST_STORE=/etc/pki/java/cacerts
TRUST_STORE=/usr/jdk64/jdk1.8.0_112/jre/lib/security/cacerts
export TRUST_STORE=$TRUST_STORE

    rpm -q expect || yum install -y expect
    cat <<EOF > ambari-ssl-expect.exp
#!/usr/bin/expect
spawn "/usr/sbin/ambari-server" "setup-security"
expect "Enter choice"
send "1\r"
expect "Do you want to configure HTTPS"
send "y\r"
expect "SSL port"
send "\r"
expect "Enter path to Certificate"
send "/tmp/security/chained.pem\r"
expect "Enter path to Private Key"
send "/tmp/security/domain.key\r"
expect "Please enter password for Private Key"
send "\r"
send "\r"
interact
EOF

    cat <<EOF > ambari-truststore-expect.exp
#!/usr/bin/expect
spawn "/usr/sbin/ambari-server" "setup-security"
expect "Enter choice"
send "4\r"
expect "Do you want to configure a truststore"
send "y\r"
expect "TrustStore type"
send "jks\r"
expect "Path to TrustStore file"
send "\$env(TRUST_STORE)\r"
expect "Password for TrustStore"
send "changeit\r"
expect "Re-enter password"
send "changeit\r"
interact
EOF
    if ! grep -q 'api.ssl=true' /etc/ambari-server/conf/ambari.properties; then
        /usr/bin/expect ambari-ssl-expect.exp
            /usr/bin/expect ambari-truststore-expect.exp

        service ambari-server restart

        while true; do if tail -100 /var/log/ambari-server/ambari-server.log | grep -q 'Started Services'; then break; else echo -n .; sleep 3; fi; done; echo
    fi

    rm -f ambari-ssl-expect.exp  ambari-truststore-expect.exp
    #validate wget -O- --no-check-certificate "https://${AMBARI_SERVER}:8443/#/main/dashboard/metrics"
```
配置ambari的TrustStore的目的：当hadoop的各服务启用https后，ambari作为客户端连接各服务时，需要一个可信证书库（想象一下IE浏览器中可信CA证书和中间人证书）。如果仅仅是ambari自己启用SSL(其它hadoop服务不启用SSL)，ambari可以不配置TrustStore。  

这一步执行后，已经可以用https访问ambari了。可以在windows下用浏览器访问地址是：`https://c7301.ambari.apache.org:8443`。会发现浏览器的地址栏显示这是一个安全网站。  
也可用这样测试：  
```
$ wget -O- --no-check-certificate "https://c7301.ambari.apache.org:8443/#/main/dashboard/metrics"
```

#### 3.对hadoop启用SSL
创建了一个`hadoopSSL.sh`脚本：
```
#!/usr/bin/env bash

server1="c7301.dp.imaicloud.com"
server2="c7302.dp.imaicloud.com"
server3="c7303.dp.imaicloud.com"

NAMENODE_SERVER_ONE=$server1
RESOURCE_MANAGER_SERVER_ONE=$server2
HISTORY_SERVER=$server2
ALL_NAMENODE_SERVERS="${NAMENODE_SERVER_ONE} $server2"
ALL_HADOOP_SERVERS="$server1 $server2 $server3"

export AMBARI_SERVER=$server1
AMBARI_PASS=admin
CLUSTER_NAME=HDP2610

#TRUST_STORE=/etc/pki/java/cacerts
TRUST_STORE=/usr/jdk64/jdk1.8.0_112/jre/lib/security/cacerts
#
# Enable Hadoop UIs SSL encryption. Stop all Hadoop components first
#
function hadoopSSLEnable() {

    for host in ${ALL_HADOOP_SERVERS}; do
        if [ -e "domain.p12" ]; then continue; fi
        openssl pkcs12 -export -in signed.crt -inkey domain.key -out domain.p12 -name mydomain -passout pass:vagrant
    done

        ssh $host "keytool -import -noprompt -alias full_chained -file /tmp/full_chained.pem -storepass changeit -keystore $TRUST_STORE; rm -f /tmp/full_chained.pem"

    for host in ${ALL_HADOOP_SERVERS}; do
        scp domain.p12 ${host}:/tmp/domain.p12
        scp full_chained.pem ${host}:/tmp/full_chained.pem
        ssh $host "
            keytool -import -noprompt -alias letsCA -file /tmp/full_chained.pem -storepass vagrant -keypass vagrant -keystore /etc/hadoop/conf/hadoop-private-keystore.jks
            keytool --importkeystore -noprompt -deststorepass vagrant -destkeypass vagrant -destkeystore /etc/hadoop/conf/hadoop-private-keystore.jks -srckeystore /tmp/domain.p12 -srcstoretype PKCS12 -srcstorepass vagrant -alias mydomain
            chmod 440 /etc/hadoop/conf/hadoop-private-keystore.jks
            chown yarn:hadoop /etc/hadoop/conf/hadoop-private-keystore.jks
            rm -f /tmp/full_chained.pem /tmp/domain.p12;
            "
    done

    cat <<EOF | while read p; do p=${p/,}; p=${p//\"}; if [ -z "$p" ]; then continue; fi; /var/lib/ambari-server/resources/scripts/configs.sh -u admin -p $AMBARI_PASS -port 8443 -s set $AMBARI_SERVER $CLUSTER_NAME $p &> /dev/null || echo "Failed to change $p in Ambari"; done
        hdfs-site "dfs.https.enable"   "true",
        hdfs-site "dfs.http.policy"   "HTTPS_ONLY",
        hdfs-site "dfs.datanode.https.address"   "0.0.0.0:50475",
        hdfs-site "dfs.namenode.https-address"   "0.0.0.0:50470",

        core-site "hadoop.ssl.require.client.cert"   "false",
        core-site "hadoop.ssl.hostname.verifier"   "DEFAULT",
        core-site "hadoop.ssl.keystores.factory.class"   "org.apache.hadoop.security.ssl.FileBasedKeyStoresFactory",
        core-site "hadoop.ssl.server.conf"   "ssl-server.xml",
        core-site "hadoop.ssl.client.conf"   "ssl-client.xml",

        mapred-site "mapreduce.jobhistory.http.policy"   "HTTPS_ONLY",
        mapred-site "mapreduce.jobhistory.webapp.https.address"   "${HISTORY_SERVER}:19443",
        mapred-site mapreduce.jobhistory.webapp.address "${HISTORY_SERVER}:19443",

        yarn-site "yarn.http.policy"   "HTTPS_ONLY"
        yarn-site "yarn.log.server.url"   "https://${HISTORY_SERVER}:19443/jobhistory/logs",
        yarn-site "yarn.resourcemanager.webapp.https.address"   "${RESOURCE_MANAGER_SERVER_ONE}:8090",
        yarn-site "yarn.nodemanager.webapp.https.address"   "0.0.0.0:45443",

        ssl-server "ssl.server.keystore.password"   "vagrant",
        ssl-server "ssl.server.keystore.keypassword"   "vagrant",
        ssl-server "ssl.server.keystore.location"   "/etc/hadoop/conf/hadoop-private-keystore.jks",
        ssl-server "ssl.server.truststore.location"   "${TRUST_STORE}",
        ssl-server "ssl.server.truststore.password"   "changeit",

        ssl-client "ssl.client.keystore.location"   "${TRUST_STORE}",
        ssl-client "ssl.client.keystore.password"   "changeit",
        ssl-client "ssl.client.truststore.password"   "changeit",
        ssl-client "ssl.client.truststore.location"   "${TRUST_STORE}"
EOF
    rm -f doSet_version*
    # In Ambari, perform Start ALL
}
hadoopSSLEnable
```
首先，导出了所有节点的私钥(p12格式)，然后复制到了各个节点，同时复制的还有CA的根证书。  
然后，为HDP集群的各个节点创建密钥库(hadoop-private-keystore.jks)。密钥库中导入CA根证书，导入p12格式的私钥。  
最后调用ambari的配置脚本将https相关的参数配置到HDP集群中。当脚本执行成功后，可以ambari界面查看。  
用openssl测试一下SSL服务器：
```
$ openssl s_client -connect c7301.dp.imaicloud.com:50470 -showcerts
```
还可以用curl访问启用SSL的hdfs来测试：
```
$ kinit root/admin
$ curl -k --negotiate -u :  https://c7301.dp.imaicloud.com:50470/webhdfs/v1/user?op=LISTSTATUS
```
我的测试集群启用了kerberos，所以需要先登录KDC。注意URL是https的。  
完整的脚本[在这](https://github.com/wbwangk/EnableSSLinHDP/blob/master/dp_ssl.sh)。

## 附1、创建内部CA
[参考](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_security/content/create-internal-ca.html)，如果对keytool不熟悉建议先读[这个](https://github.com/wbwangk/wbwangk.github.io/wiki/java%E7%BB%93%E5%90%88keytool%E5%AE%9E%E7%8E%B0%E5%85%AC%E7%A7%81%E9%92%A5%E7%AD%BE%E5%90%8D%E4%B8%8E%E9%AA%8C%E8%AF%81)。  

一般使用开源软件OpenSSL来创建CA。首先，生成CA根证书公钥和私钥。然后，将公私钥证书配置到OpenSSL的配置文件。之后，就可以使用内部CA来处理证书签名请求(CSR)，生成签名证书了。  

#### 0.确定CA文件存储位置
对ubuntu或centos，openssl一般都预装了。没有装，就用apt或yum自己装。首先需要确定CA根证书文件的默认位置。CA根证书的保存位置在配置文件openssl.cnf中有定义。查找一下openssl.cnf：
```
$ find /etc -name openssl.cnf
/etc/pki/tls/openssl.cnf
$ cat /etc/pki/tls/openssl.cnf | grep dir
dir             = /etc/pki/CA           # Where everything is kept
database        = $dir/index.txt        # database index file.
certificate     = $dir/cacert.pem       # The CA certificate
serial          = $dir/serial           # The current serial number
private_key     = $dir/private/cakey.pem# The private key
certs           = $dir/cacert.pem       # Certificate chain to include in reply
(其它略)
```
`dir`定义了CA的根目录，`certificate`是根证书，`private_key`是CA的私钥。  
提醒注意的是，不同的linux版本上述配置文件也许有所不同。为了省事，下面的文件名尽量按上述的默认值。  

#### 1.生成密钥对和证书
将创建CA的根证书。
```
$ cd /etc/pki/CA
$ openssl req -new -x509 -keyout cakey.pem -out cacert.pem -days 365 -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=iMaiCA"
..............+++
.......................................+++
writing new private key to 'ca-key'
Enter PEM pass phrase: vagrant
Verifying - Enter PEM pass phrase: vagrant
-----
```
生成的CA一个公钥-私钥对和证书，旨在签署其他证书。当前目录下多了两个文件cakey.pem和cacert.pem。  
ca-key文件的第一行：`-----BEGIN ENCRYPTED PRIVATE KEY-----`  
ca-cert文件的第一行：`-----BEGIN CERTIFICATE-----`  

#### 2.创建和移动CA文件
将CA密钥移动到`$dir/private`：  
```
$ mv cakey.pem /etc/pki/CA/private
```
添加所需文件：
```
$ touch  /etc/pki/CA/index.txt; echo 1000 >>  /etc/pki/CA/serial
```
设置权限ca-key：
```
chmod 0400 /etc/pki/CA/private/ca-key
```
#### 3.修改OpenSSL配置文件
打开OpenSSL配置文件(`/etc/pki/tls/openssl.cnf`)，确认以下内容。由于我是按默认值生成的文件，配置文件不用改：
```
[ CA_default ]

dir             = /etc/pki/CA           # Where everything is kept
certs           = $dir/certs            # Where the issued certs are kept
crl_dir         = $dir/crl              # Where the issued crl are kept
database        = $dir/index.txt        # database index file.
#unique_subject = no                    # Set to 'no' to allow creation of
                                        # several ctificates with same subject.
new_certs_dir   = $dir/newcerts         # default place for new certs.

certificate     = $dir/cacert.pem       # The CA certificate
serial          = $dir/serial           # The current serial number
crlnumber       = $dir/crlnumber        # the current crl number
                                        # must be commented out to leave a V1 CRL
crl             = $dir/crl.pem          # The current CRL
private_key     = $dir/private/cakey.pem# The private key
RANDFILE        = $dir/private/.rand    # private random number file

x509_extensions = usr_cert              # The extentions to add to the cert
```

#### 4.生成CSR并签署
这一步是在测试、验证刚刚建立的内部CA。  
CA的一个重要用途是处理“证书签名请求”，生成签名后的证书。  
假设一个场景：利用nginx搭建https网站。  
首先，生成CSR:
```
$ openssl req -new -newkey rsa:2048 -nodes -keyout nginx0.key -out nginx0.csr -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=c7304.ambari.apache.org"
```
生成了私钥nginx0.key和证书签名请求nginx0.csr。nginx0.csr的开始一行是`-----BEGIN CERTIFICATE REQUEST-----`。  

下面利用刚创建的CA处理这个证书签名请求：
```
$ openssl ca -in nginx0.csr -out nginx0.crt
```
会提示输入CA的私钥密码。可以查看一下签名后的证书：
```
$ openssl x509 -noout -text -in nginx0.crt
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=CN, ST=Shan Dong, L=Ji Nan, O=Inspur, OU=SBG, CN=iMaiCA
        Validity
            Not Before: Aug  2 00:50:53 2017 GMT
            Not After : Aug  2 00:50:53 2018 GMT
        Subject: C=CN, ST=Shan Dong, O=Inspur, OU=SBG, CN=c7304.ambari.apache.org
```
已上信息省略了一部分。可以看出Issuer就是刚建立的CA，Subjcet是nginx的信息。  

## 附2、申请Let's Encrypt证书
[参考1](https://imququ.com/post/letsencrypt-certificate.html)，[参考2](https://bryanchain.com/2016/01/28/lets-encrypt-with-nginx/)

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
$ openssl req -new -sha256 -key domain.key -subj "/" -reqexts SAN -config <(cat /etc/pki/tls/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:dp.imaicloud.com,DNS:c7304.dp.imaicloud.com")) > domain.csr
```
Let's Encrypt可能是处于管理的要求让你生成两个密钥对：acccout和domain。证书签名申请用domain.key签名，而挑战请求用account签名。openssl的配置文件`/etc/pki/tls/openssl.cnf`在第五章提到过。上面的脚本是临时在openssl.cnf文件最后增加了下面的内容：
```
[SAN]
subjectAltName=DNS:dp.imaicloud.com,DNS:c7304.dp.imaicloud.com
```
生成证书签名请求(CSR)保存在文件domain.csr中。

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
        server_name .dp.imaicloud.com;
        root dp;
        index index.html index.htm;

        location ^~ /.well-known/acme-challenge/ {
                alias /opt/nginx/dp/challenge/;
                try_files $uri =404;
        }
```
注意`.dp.imaicloud.com`是个通配符server_name，下级域名(如c7304.dp.imaicloud.com)也可以直接用这个配置响应let's encrypt的挑战。  

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
Verifying c7304.dp.imaicloud.com...
c7304.dp.imaicloud.com verified!
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

## 附3、命令备忘
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