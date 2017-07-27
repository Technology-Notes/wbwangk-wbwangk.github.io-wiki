[参考](http://docs.oracle.com/javase/tutorial/security/toolfilex/step2.html)  

签名方生成密钥对（自动创建密钥库）、导出公钥。验证方将公钥导入密钥库。  
keytool是JDK自带的一个密钥库管理工具。  
## 签名方的操作
签名方是个叫Stan的人。
#### 生成密钥对
```
$ keytool -genkey -alias signLegal -keystore examplestanstore -validity 1800
```
生成别名为signLegal的密钥对，存放在密钥库examplestanstore中，证书的有效期是1800天(默认是90天)。  
输入一系列的参数。输入的参数遵循了LDAP的风格和标准。可以想象，生成的密钥对可以看成LDAP的一个条目。  
命令执行成功后会在当前目录下创建一个叫`examplestanstore`的文件。  
#### 查看密钥对
```
$ keytool -list -keystore examplestanstore -v
```
列出了examplestanstore密钥库的中所有密钥对。`-v`参数表示详细信息，详细信息中有证书的失效时间。  

#### 导出公钥证书
```
$ keytool -export -keystore examplestanstore -alias signLegal -file StanSmith.cer
```
导出的公钥存放在当前目录的StanSmith.cer文件中，是个二进制文件。  

#### 生成jar并签名
新建一个文本文件hello.txt，打成jar包：
```
$ jar cvfM hello.jar hello.txt
```
对jar包进行签名：
```
$ jarsigner -tsa http://tsa.starfieldtech.com -keystore examplestanstore -signedjar shello.jar hello.jar signLegal 
```
## 验证方的操作
验证方是个叫Ruch的人。

#### 导入公钥
```
$ keytool -import -alias stan -file StanSmith.cer -keystore exampleruthstore
```
如果密钥库exampleruthstore不存在，keytool会自动创建(新库会让输入两次口令)。  
可以利用前文的`-list`参数查看一下新导入的公钥证书：
```
$ keytool -list -alias stan -keystore exampleruthstore
stan, 2017-7-27, trustedCertEntry,
证书指纹 (SHA1): 38:EB:50:A4:1D:87:E5:1B:9A:41:B0:9E:92:0D:8C:B7:41:BD:C4:AF
```
#### 验证jar
```
$ jarsigner -verify -verbose -keystore exampleruthstore shello.jar 
(前略)
- 由 "CN=Stan Smith, OU=Legal, O=Example2, L=New York, ST=NY, C=US" 签名
    摘要算法: SHA-256
    签名算法: SHA1withDSA, 1024 位密钥
  由 "CN=Starfield Timestamp Authority - G1, O="Starfield Technologies, Inc.", L=Scottsdale, ST=Arizona, C=US" 于 星期四 七月 27 02:32:12 UTC 2017 加时间戳
    时间戳摘要算法: SHA-256
    时间戳签名算法: SHA1withRSA, 2048 位密钥

jar 已验证。
```
## 导出keystore中私钥
需要编程实现：
```java
import java.io.File;  
import java.io.FileInputStream;  
import java.io.FileWriter;  
import java.security.Key;  
import java.security.KeyPair;  
import java.security.KeyStore;  
import java.security.KeyStoreException;  
import java.security.NoSuchAlgorithmException;  
import java.security.PrivateKey;  
import java.security.PublicKey;  
import java.security.UnrecoverableKeyException;  
import java.security.cert.Certificate;  
import sun.misc.*;  
public class ExportPrivateKey {  
private File keystoreFile;  
private String keyStoreType;  
private char[] password;  
private String alias;  
private File exportedFile;  
public static KeyPair getPrivateKey(KeyStore keystore, String alias, char[] password) {  
try {  
Key key=keystore.getKey(alias,password);  
if(key instanceof PrivateKey) {  
Certificate cert=keystore.getCertificate(alias);  
PublicKey publicKey=cert.getPublicKey();  
return new KeyPair(publicKey,(PrivateKey)key);  
}  
} catch (UnrecoverableKeyException e) {  
} catch (NoSuchAlgorithmException e) {  
} catch (KeyStoreException e) {  
}  
return null;  
}  
public void export() throws Exception{  
KeyStore keystore=KeyStore.getInstance(keyStoreType);  
BASE64Encoder encoder=new BASE64Encoder();  
keystore.load(new FileInputStream(keystoreFile),password);  
KeyPair keyPair=getPrivateKey(keystore,alias,password);  
PrivateKey privateKey=keyPair.getPrivate();  
String encoded=encoder.encode(privateKey.getEncoded());  
FileWriter fw=new FileWriter(exportedFile);  
fw.write("-----BEGIN RSA PRIVATE KEY-----\n");  
fw.write(encoded);  
fw.write("\n");  
fw.write("-----END RSA PRIVATE KEY-----");  
fw.close();  
}  
public static void main(String args[]) throws Exception{  
ExportPrivateKey export=new ExportPrivateKey();  
export.keystoreFile=new File("examplestanstore");  
export.keyStoreType="JKS";  
export.password="123456".toCharArray();  
export.alias="signLegal";  
export.exportedFile=new File("signLegal.key");  
export.export();  
}  
}
```
看看新生成的signLegal.key文件的内容：
```
—–BEGIN PRIVATE KEY—–
MIIBSwIBADCCASwGByqGSM44BAEwggEfAoGBAP1/U4EddRIpUt9KnC7s5Of2EbdSPO9EAMMeP4C2
USZpRV1AIlH7WT2NWPq/xfW6MPbLm1Vs14E7gB00b/JmYLdrmVClpJ+f6AR7ECLCT7up1/63xhv4
O1fnxqimFQ8E+4P208UewwI1VBNaFpEy9nXzrith1yrv8iIDGZ3RSAHHAhUAl2BQjxUjC8yykrmC
ouuEC/BYHPUCgYEA9+GghdabPd7LvKtcNrhXuXmUr7v6OuqC+VdMCz0HgmdRWVeOutRZT+ZxBxCB
gLRJFnEj6EwoFhO3zwkyjMim4TwWeotUfI0o4KOuHiuzpnWRbqN/C/ohNWLx+2J6ASQ7zKTxvqhR
kImog9/hWuWfBpKLZl6Ae1UlZAFMO/7PSSoEFgIUNp1DXIvT+7wyyZ02tUXwt0xV3Ag=
—–END PRIVATE KEY—–
```
## openssl
#### 摘要计算
创建一个内容是`1234`的文本文件hello.txt。用openssl计算它的SHA256摘要（SHA256是jarsigner的默认摘要算法）：
```
$ cat hello.txt
1234
$ openssl dgst -SHA256 -out hello.sha256 hello.txt
SHA256(hello.txt)= 03ac674216f3e15c761ee1a5e255f067953623c8b388b4459e13f978d7c846f4
```