[参考](https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.6.1/bk_security/content/create-internal-ca.html)  
如果对keytool的参数阅读起来吃力，建议先读[这个](https://github.com/wbwangk/wbwangk.github.io/wiki/java%E7%BB%93%E5%90%88keytool%E5%AE%9E%E7%8E%B0%E5%85%AC%E7%A7%81%E9%92%A5%E7%AD%BE%E5%90%8D%E4%B8%8E%E9%AA%8C%E8%AF%81)。  

## 内部CA
一般使用开源软件OpenSSL来创建CA。  

#### 1.生成密钥对和证书
```
$ openssl req -new -x509 -keyout ca-key -out ca-cert -days 365
Generating a 2048 bit RSA private key
........................................................................................+++
...................................+++
writing new private key to 'ca-key'
Enter PEM pass phrase: vagrant
Verifying - Enter PEM pass phrase: vagrant
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:cn
State or Province Name (full name) []:shandong
Locality Name (eg, city) [Default City]:jinan
Organization Name (eg, company) [Default Company Ltd]:sbg
Organizational Unit Name (eg, section) []:ec
Common Name (eg, your name or your server's hostname) []:webbca
Email Address []:wbwang@inspur.com
```
生成的CA只是一个公钥 - 私钥对和证书，旨在签署其他证书。  
当前目录下多了两个文件ca-key和ca-cert。  
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
crl             = $dir/crl.pem              # The current CRL
private_key     = /root/CA/private/ca-key   # The private key
RANDFILE        = /root/CA/private/.rand     # private random number file

x509_extensions = usr_cert              # The extensions to add to the cert
```
保存配置文件并重启OpenSSL。  

## CA使用(未完)
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
## Java安全教程
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

## Java安全套接字扩展（JSSE）
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

### JSSE类和接口
![](http://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/classes1.jpg)  
#### SSLEngine类
![](http://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/sslengine.jpg)  

SSLEngine该类的一个实例可以是以下状态之一：  
 - 创建：准备配置
 - 初步握手：执行认证和协商通信参数
 - 应用数据：准备应用交换
 - 重新握手：重新协商通信参数/认证; 握手数据可能与应用程序数据混合
 - 关闭：准备关闭连接

例1: 创建一个SSLEngine对象：
```java
import javax.net.ssl.*;
import java.security.*;

// Create and initialize the SSLContext with key material
char[] passphrase = "passphrase".toCharArray();

// First initialize the key and trust material
KeyStore ksKeys = KeyStore.getInstance("JKS");
ksKeys.load(new FileInputStream("testKeys"), passphrase);
KeyStore ksTrust = KeyStore.getInstance("JKS");
ksTrust.load(new FileInputStream("testTrust"), passphrase);

// KeyManagers decide which key material to use
KeyManagerFactory kmf = KeyManagerFactory.getInstance("SunX509");
kmf.init(ksKeys, passphrase);

// TrustManagers decide whether to allow connections
TrustManagerFactory tmf = TrustManagerFactory.getInstance("SunX509");
tmf.init(ksTrust);

sslContext = SSLContext.getInstance("TLS");
sslContext.init(kmf.getKeyManagers(), tmf.getTrustManagers(), null);

// Create the engine
SSLEngine engine = sslContext.createSSLengine(hostname, port);

// Use as client
engine.setUseClientMode(true);
```
#### 生成和处理SSL数据
两个主要的SSLEngine方法是 wrap()和unwrap()。他们分别负责生成和使用网络数据。根据SSLEngine对象的状态，此数据可能是握手或应用程序数据。  
例2: 使用非阻塞SocketChannel：
```java
// Create a nonblocking socket channel
SocketChannel socketChannel = SocketChannel.open();
socketChannel.configureBlocking(false);
socketChannel.connect(new InetSocketAddress(hostname, port));

// Complete connection
while (!socketChannel.finishedConnect()) {
    // do something until connect completed
}

// Create byte buffers to use for holding application and encoded data
SSLSession session = engine.getSession();
ByteBuffer myAppData = ByteBuffer.allocate(session.getApplicationBufferSize());
ByteBuffer myNetData = ByteBuffer.allocate(session.getPacketBufferSize());
ByteBuffer peerAppData = ByteBuffer.allocate(session.getApplicationBufferSize());
ByteBuffer peerNetData = ByteBuffer.allocate(session.getPacketBufferSize());

// Do initial handshake
doHandshake(socketChannel, engine, myNetData, peerNetData);

myAppData.put("hello".getBytes());
myAppData.flip();

while (myAppData.hasRemaining()) {
    // Generate SSL/TLS encoded data (handshake or application data)
    SSLEngineResult res = engine.wrap(myAppData, myNetData);

    // Process status of call
    if (res.getStatus() == SSLEngineResult.Status.OK) {
        myAppData.compact();

        // Send SSL/TLS encoded data to peer
        while(myNetData.hasRemaining()) {
            int num = socketChannel.write(myNetData);
            if (num == 0) {
                // no bytes written; try again later
            }
        }
    }

    // Handle other status:  BUFFER_OVERFLOW, CLOSED
    ...
}
```
示例3：从非阻塞SocketChannel读取数据  
```java
// Read SSL/TLS encoded data from peer
int num = socketChannel.read(peerNetData);
if (num == -1) {
    // The channel has reached end-of-stream
} else if (num == 0) {
    // No bytes read; try again ...
} else {
    // Process incoming data
    peerNetData.flip();
    res = engine.unwrap(peerNetData, peerAppData);

    if (res.getStatus() == SSLEngineResult.Status.OK) {
        peerNetData.compact();

        if (peerAppData.hasRemaining()) {
            // Use peerAppData
        }
    }
    // Handle other status:  BUFFER_OVERFLOW, BUFFER_UNDERFLOW, CLOSED
    ...
}
```
示例4：处理BUFFER_UNDERFLOW和BUFFER_OVERFLOW
```java
SSLEngineResult res = engine.unwrap(peerNetData, peerAppData);
switch (res.getStatus()) {

case BUFFER_OVERFLOW:
    // Maybe need to enlarge the peer application data buffer.
    if (engine.getSession().getApplicationBufferSize() > peerAppData.capacity()) {
        // enlarge the peer application data buffer
    } else {
        // compact or clear the buffer
    }
    // retry the operation
    break;

case BUFFER_UNDERFLOW:
    // Maybe need to enlarge the peer network packet buffer
    if (engine.getSession().getPacketBufferSize() > peerNetData.capacity()) {
        // enlarge the peer network packet buffer
    } else {
        // compact or clear the buffer
    }
    // obtain more inbound network data and then retry the operation
    break;

    // Handle other status: CLOSED, OK
    ...
}
```
示例5：检查和处理握手状态和总体状态
```java
void doHandshake(SocketChannel socketChannel, SSLEngine engine,
        ByteBuffer myNetData, ByteBuffer peerNetData) throws Exception {

    // Create byte buffers to use for holding application data
    int appBufferSize = engine.getSession().getApplicationBufferSize();
    ByteBuffer myAppData = ByteBuffer.allocate(appBufferSize);
    ByteBuffer peerAppData = ByteBuffer.allocate(appBufferSize);

    // Begin handshake
    engine.beginHandshake();
    SSLEngineResult.HandshakeStatus hs = engine.getHandshakeStatus();

    // Process handshaking message
    while (hs != SSLEngineResult.HandshakeStatus.FINISHED &&
        hs != SSLEngineResult.HandshakeStatus.NOT_HANDSHAKING) {

        switch (hs) {

        case NEED_UNWRAP:
            // Receive handshaking data from peer
            if (socketChannel.read(peerNetData) < 0) {
                // The channel has reached end-of-stream
            }

            // Process incoming handshaking data
            peerNetData.flip();
            SSLEngineResult res = engine.unwrap(peerNetData, peerAppData);
            peerNetData.compact();
            hs = res.getHandshakeStatus();

            // Check status
            switch (res.getStatus()) {
            case OK :
                // Handle OK status
                break;

            // Handle other status: BUFFER_UNDERFLOW, BUFFER_OVERFLOW, CLOSED
            ...
            }
            break;

        case NEED_WRAP :
            // Empty the local network packet buffer.
            myNetData.clear();

            // Generate handshaking data
            res = engine.wrap(myAppData, myNetData);
            hs = res.getHandshakeStatus();

            // Check status
            switch (res.getStatus()) {
            case OK :
                myNetData.flip();

                // Send the handshaking data to peer
                while (myNetData.hasRemaining()) {
                    socketChannel.write(myNetData);
                }
                break;

            // Handle other status:  BUFFER_OVERFLOW, BUFFER_UNDERFLOW, CLOSED
            ...
            }
            break;

        case NEED_TASK :
            // Handle blocking tasks
            break;

        // Handle other status:  // FINISHED or NOT_HANDSHAKING
        ...
        }
    }

    // Processes after handshaking
    ...
}
```
#### 处理阻塞任务
```java
if (res.getHandshakeStatus() == SSLEngineResult.HandshakeStatus.NEED_TASK) {
    Runnable task;
    while ((task = engine.getDelegatedTask()) != null) {
        new Thread(task).start();
    }
}
```
#### 关闭
示例6：关闭SSL / TLS连接:
```java
// Indicate that application is done with engine
engine.closeOutbound();

while (!engine.isOutboundDone()) {
    // Get close message
    SSLEngineResult res = engine.wrap(empty, myNetData);

    // Check res statuses

    // Send close message to peer
    while(myNetData.hasRemaining()) {
        int num = socketChannel.write(myNetData);
        if (num == 0) {
            // no bytes written; try again later
        }
        myNetData().compact();
    }
}

// Close transport
socketChannel.close();
```
## java访问https链接
现代浏览器都内嵌了一列可信CA的公钥证书。如果你访问一个不可信的https网站（一般是自建CA），浏览器会弹出警告，只有把要访问的网站加入“例外”目录，浏览器才运行继续访问。  
Java也实现了类似机制。JDK自带的`$JAVA_HOME/jre/lib/security/cacerts`是个JKS格式的keystore文件，里面是默认的可信CA证书。本机装了多个JDK，查了一下发现了多个cacerts：  
```
$ find / -name cacerts
/etc/pki/ca-trust/extracted/java/cacerts
/etc/pki/java/cacerts    （符号链接，指向第一个cacerts）
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64/jre/lib/security/cacerts
/usr/jdk64/jdk1.8.0_112/jre/lib/security/cacerts
```
经测试，第一个管用。这与[JSSE官方文档](http://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html#X509TrustManager)中说的文件位置不符。系统属性javax.net.ssl.trustStore可以定义另一个文件来代替cacerts。  
  
可以用`keytool`命令查看该文件内容：  
```
$ keytool -list -keystore <cacerts文件> -storepass changeit
```
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
为了测试cacerts的作用，现在把它改名，然后用一个空文件代替。实测中，下面的<cacerts file>被替换为`/etc/pki/java/cacerts`：
```
$ mv <cacerts file> <cacerts file>.old
$ echo "" > <cacerts file>
$ java HttpsTest https://cn.bing.com
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed:
(下略)
```
必应网站也不行了，是因为cacerts文件中的证书被清空了。  