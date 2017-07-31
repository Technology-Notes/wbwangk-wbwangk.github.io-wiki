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