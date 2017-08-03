- 一、安全知识  
- 二、[java访问https链接](https://github.com/wbwangk/wbwangk.github.io/wiki/https%E7%A0%94%E7%A9%B6#%E4%BA%8Cjava%E8%AE%BF%E9%97%AEhttps%E9%93%BE%E6%8E%A5)  
- 三、[建立https服务器](https://github.com/wbwangk/wbwangk.github.io/wiki/https%E7%A0%94%E7%A9%B6#%E4%B8%89%E5%BB%BA%E7%AB%8Bhttps%E6%9C%8D%E5%8A%A1%E5%99%A8)  
- 四、[双向SSL](https://github.com/wbwangk/wbwangk.github.io/wiki/https%E7%A0%94%E7%A9%B6#%E5%9B%9B%E5%8F%8C%E5%90%91ssl)  
- 五、[建立内部CA](https://github.com/wbwangk/wbwangk.github.io/wiki/https%E7%A0%94%E7%A9%B6#%E4%BA%94%E5%88%9B%E5%BB%BA%E5%86%85%E9%83%A8ca)  
- 六、[HDP的SSL证书](https://github.com/wbwangk/wbwangk.github.io/wiki/https%E7%A0%94%E7%A9%B6#%E5%85%ADhdp%E7%9A%84ssl%E8%AF%81%E4%B9%A6)  

## 一、安全知识
### (一)Java安全教程
[原文](http://docs.oracle.com/javase/tutorial/security/TOC.html)  

#### 私钥/公钥对
私钥对消息进行数字签名(sign)。公钥验证签名，确保签名是私钥签署的。（身份验证、不可否认）  

#### 证书(certificate)
证书包含公钥、公钥拥有者(subject)、签署者的签名、签署者(issuer)。可以简单认为证书包含公钥，以及证明公钥有效的签名。  
自签名证书的subject与issuer相同。根证书是自签名证书。

#### 密钥库(keystore)
将密钥和对应公钥保存在一个受密码保护的数据库中，就叫密钥库。这是个jdk的术语，不是通用的。  
密钥库中条目有两种：私钥/公钥对、可信公钥。或者叫私钥/证书对、可信证书。（记住，证书里只有公钥）  

#### 为公钥证书生成证书签名请求(CSR)
CSR是Certificate Signing Request的简写。  
当使用keytool生成公钥/私钥对时，它创建了一个密钥库条目。条目中包含公钥和公钥的自签名证书，即证书使用相应的私钥进行签名。  
如果想把签名换成CA签署的，需要生成“证书签名请求(CSR)”：
```
$ keytool -certreq -alias <alias> -file <csrFile> 
```
把生成的<csrFile>交给CA（如LetsEncrypt），CA用某种手段认可主体(subject)身份后，生成一个签名后的证书发回。  
有时CA发回多个证书(证书链)，其中的每个证书都是链中的前一个证书签署的。  

#### 导入CA的响应
要将密钥库中的自签名证书覆盖为CA签署的证书，首先需要密钥库中创建一个“可信证书”条目，可信证书可以被CA的公钥认证。通过这样一个条目，可以验证CA的签名。可信证书要么是CA的证书(CA签署)，要么是证书链中的最后一个证书。  

#### 将证书从CA导入为“受信任的证书”
在导入从CA回复的证书之前，您需要在密钥库或cacerts文件中存在一个或多个“受信任的证书” 。  
 - 如果CA返回的是证书链，只需要导入最顶层的证书。这个证书是“根”CA认证的某个下级CA的公钥证书。  
 - 如果CA返回的是单个证书，需要导入CA的证书。如果CA证书不是自签名的，需要导入其上级CA的证书，依次类推，直到自签名的根证书。  

#### 导入CA回复的证书
一旦象前文描述的那样建立了“可信证书”库，就可以导入从CA回复的证书了。  
导入CA回复证书的命令：
```
$ keytool -import -trustcacerts -keystore <storefile> -alias <alias> -file <certReplyFile>
```
#### 查看某服务器的公钥证书
```
$ openssl s_client -connect xxxxx.com:443 |tee logfile
```
可以显示某网站的公钥证书及其他内容。其中`BEGIN CERTIFICATE`与`END CERTIFICATE`之间的内容就是服务器公钥证书。经测试可以正确返回的有：
```
$ openssl s_client -connect mail.inspur.com:443 |tee logfile
$ openssl s_client -connect c7301.ambari.apache.org:8443 |tee logfile
```
上面端口8443是hadoop集群的apache knox的服务器。

### (二)Java安全套接字扩展（JSSE）
[参考](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html#Introduction)  

Java安全套接字扩展（JSSE）支持安全的Internet通信，它自1.4版本开始包含在JDK中。它为Java版本的SSL和TLS协议提供了框架和实现，并且包括数据加密，服务器身份验证，消息完整性和可选的客户端认证的功能。使用JSSE，开发人员可以通过TCP/IP为客户端和服务器数据安全通道，可运行任何应用程序协议（如超文本传输​​协议（HTTP），Telnet或FTP）。  

#### 为什么使用SSL?
目前广泛使用的是SSL3.0和TLS1.0，两者的差异很小。SSL解决的问题：  
 - 你不能总是确定与你进行沟通的实体真的是你认为的。  
 - 网络数据可能被拦截，因此有可能被未经授权的第三方（有时称为攻击者）读取。  
 - 拦截数据的攻击者可能会在将其发送到接收器之前进行修改。  

#### SSL如何工作
用于加密和解密通过网络传输的数据的算法通常分为两类：**密钥密码术**和**公钥密码术**。  
**密钥密码术**的加密和解密密码相同，又叫对称加密。密钥加密的算法有：Data Encryption Standard (DES), Triple DES (3DES), Rivest Cipher 2 (RC2), and Rivest Cipher 4 (RC4).  

**公钥密码术**的密钥分为私钥和公钥。公钥可以公开传播，私钥保存在安全的地方。公钥加密，只有对应的私钥能解密。私钥加密，只有对应的公钥能解密（这种特性还可以用来确认发送者身份）。  
公钥密码术也被称为非对称密码术，因为不同的密钥用于加密和解密数据。经常与SSL一起使用的公知密钥加密算法是Rivest Shamir Adleman（RSA）算法。使用专门用于密钥交换的SSL的另一种公钥算法是Diffie-Hellman（DH）算法。公钥密码学需要大量的计算，使其非常慢。因此，它通常仅用于加密小块数据，例如秘密密钥，而不是大量的加密数据通信。  

#### 公钥证书
公钥证书可以被认为是护照的数字等价物。它由可信任的组织颁发，可为持有人提供身份证明。发布公钥证书的受信任组织称为证书颁发机构（CA）。  
公钥证书包含以下字段：  

 - 发行方  
  发行证书的CA。如果用户信任颁发证书的CA，并且证书有效，则用户可以信任证书。  
 - 有效期  
  证书有有效期限。在验证证书的有效性时，应检查此日期。  
 - 主体  
  包括有关证书所代表的实体的信息。实体可以是个人或组织。  
 - 主体的公钥  
  证书提供的主要信息是主体的公钥。提供所有其他字段以确保此密钥的有效性。  
 - 签名  
  证书由颁发证书的CA进行数字签名。使用CA的私钥创建签名（签署），并确保证书的有效性。因为只有证书被签名，而不是SSL事务中发送的数据，SSL不提供不可否认性。  

多个证书可以链接成**证书链**。当使用证书链时，第一个证书始终是发件人的证书。接下来是颁发发件人证书的实体的证书。如果链中有更多证书，那么每个证书都是颁发先前证书的上级CA。链中的最终证书是根CA的证书。根CA是广泛信任的公共证书颁发机构。几个根CA的信息通常存储在客户端的Internet浏览器中。该信息包括CA的公钥。着名的CA包括VeriSign，Entrust和GTE Cyber​​Trust。  

#### HMAC(散列消息认证码)
利用散列算法（SSL中常用MD5或SHA）生成消息认证码，加密后发送给对方。对方解开HMAC后，与消息进行校验，防止消息被篡改。  

## 二、java访问https链接
现代浏览器都内嵌了一列可信CA的公钥证书。如果你访问一个不可信的https网站（一般是自建CA），浏览器会弹出警告，只有把要访问的网站加入“例外”目录，浏览器才运行继续访问。Java也实现了类似机制。但OpenJDK和OracleJDK的机制有差异。OracleJDK使用自带的可信证书库(文件名为cacerts)，而OpenJDK则使用linux系统的证书体系。  

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
$ keytool -list -keystore <cacerts文件> -storepass changeit
```
`changeit`是密钥库的默认密码。 把`<cacerts文件>`替换成`/usr/java/jdk1.8.0_131/jre/lib/security/cacerts`。  
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
       //初始化
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
$ java HttpsTest https://cn.bing.com
(返回一堆html代码)
$ java HttpsTest https://kyfw.12306.cn
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
(下略)
```
这是因为kyfw.12306.cn网址的服务器证书不是可信CA签署的。而“必应”bing.com的服务器证书是cacerts中某个可信CA签署的。  

#### cacerts的修改测试
为了测试cacerts的作用，现在把它改名，然后用一个空文件代替。实测中，下面的<cacerts file>被替换为`/usr/java/jdk1.8.0_131/jre/lib/security/cacerts`：
```
$ mv <cacerts file> <cacerts file>.old
$ echo "" > <cacerts file>
$ java HttpsTest https://cn.bing.com
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed:
(下略)
```
必应网站也不行了，是因为cacerts文件中的证书被清空了。  

### OpenJDK访问https链接
卸载OracleJDK，安装OpenJDK。再查找`cacerts`文件：
```
$ find / -name cacerts
/etc/pki/ca-trust/extracted/java/cacerts
/etc/pki/java/cacerts    （符号链接，指向第一个cacerts）
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64/jre/lib/security/cacerts
```
再运行`java HttpsTest <URL>`，发现`$JAVA_HOME/jre/lib/security/cacerts`这个文件不再管用，管用的是`/etc/pki/ca-trust/extracted/java/cacerts`。  
说明OpenJDK对于可信证书库的使用与OracleJDK不同，OpenJDK优先使用linux自带的可信证书库，而OracleJDK优先使用自带的。  

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
$ nginx           (刚装上nginx时要执行，之后就不用了)
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

### (二)CA签名证书https服务器
要获得一个CA签名的nginx公钥证书，除了通过免费`let's encrypt`网站或商用CA证书公司外，还可以自己搭建一个“内部CA”。搭建办法参见第五章。以下测试假定你已经搭建好了自己的CA。  
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
双向SSL(two-way SSL)又叫Mutual Authentication。第三章主要讲单向SSL。单向SSL是客户端验证服务器是否可信。而双向SSL增加了服务器对客户端的可信验证。  
客户端设定为c7302节点。在c7302的`/opt/twowayssl`为客户端用户webb创建签名请求：
```
$ openssl req -new -newkey rsa:2048 -nodes -keyout client.key -out client.csr -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=webb"
$ scp client.csr root@c7304:/opt/ca                (将客户端的证书签名请求发送到CA所在机器)
```
CA建立在c7304上（参见第五章），需要在c7304上对该CSR进行签名：
```
$ cd /opt/ca
$ openssl ca -in client.csr -out client.crt
```

配置nginx，增加两条配置：
```
server {
    listen       443;
    server_name  c7304.ambari.apache.org;
                ssl on;
                ssl_certificate      /opt/ca/nginx.crt;
                ssl_certificate_key  /opt/ca/nginx.key;

                ssl_client_certificate /opt/ca/client.crt;
                ssl_verify_client on;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
```
用`nginx -s reload`重启nginx。
```
$ curl https://c7304.ambari.apache.org  --cacert /opt/ca/nginx.crt --cert /opt/ca/client.crt
```
重新执行之前的java程序：
```
$ cd /opt/https
$ java -cp ".:/opt/https/httpcomponents-client-4.5.3/lib/*" HttpClientSSL https://c7304.ambari.apache.org
Executing request GET https://c7304.ambari.apache.org HTTP/1.1
----------------------------------------
HTTP/1.1 400 Bad Request
```
注意到返回值不再是`200 OK`。这说明nginx双向SSL的配置是起作用了。

## 五、创建内部CA
[参考](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_security/content/create-internal-ca.html)，如果对keytool不熟悉建议先读[这个](https://github.com/wbwangk/wbwangk.github.io/wiki/java%E7%BB%93%E5%90%88keytool%E5%AE%9E%E7%8E%B0%E5%85%AC%E7%A7%81%E9%92%A5%E7%AD%BE%E5%90%8D%E4%B8%8E%E9%AA%8C%E8%AF%81)。  

一般使用开源软件OpenSSL来创建CA。首先，生成CA根证书公钥和私钥。然后，将公私钥证书配置到OpenSSL的配置文件。之后，就可以使用内部CA来处理证书签名请求(CSR)，生成签名证书了。  

#### 1.生成密钥对和证书
将创建CA的根证书。
```
$ openssl req -new -x509 -keyout ca-key -out ca-cert -days 365  -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=iMaiCA"
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
$  openssl req -new -newkey rsa:2048 -nodes -keyout nginx.key -out nginx.csr -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=c7304.ambari.apache.org"
```
生成了私钥nginx.key和证书签名请求nginx.csr。nginx.csr的开始一行是`-----BEGIN PRIVATE KEY-----`。  

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