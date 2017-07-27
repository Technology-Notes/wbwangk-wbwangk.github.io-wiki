[参考](http://docs.oracle.com/javase/tutorial/security/toolfilex/step2.html)  

## keytool的使用
keytool是JDK自带的一个密钥库管理工具。这里只用到了keytool的部分功能，包括生成密钥对，导出公钥等。keytool生成的公钥/私钥对存放到一个到了一个文件中，这个文件有密码保护，叫keystore。    

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

## java签名和验证
参考了[java安全官方教程](http://docs.oracle.com/javase/tutorial/security/apisign/index.html)。    
在该官方教程中，[GenSig.java](http://docs.oracle.com/javase/tutorial/displayCode.html?code=http://docs.oracle.com/javase/tutorial/security/apisign/examples/GenSig.java)类生成密钥对，对输入的文件进行签名，输出了一个签名结果文件sig和公钥suepk。  
[VerSig.java](http://docs.oracle.com/javase/tutorial/security/apisign/examples/VerSig.java)类接受三个参数：公钥文件名(suepk)、签名文件(sig)、被签名的源文件名(hello.txt)。  
该教程解释了两个类的原理，并附加有源码。将源码下载并编译。创建一个hello.txt的文件作为被签名的目标文件，里面随便放点字符串。然后执行：
```
$ java GenSig hello.txt                        (生成文件sig和suepk)
$ java VerSig suepk sig hello.txt
signature verifies: true
```
在实际使用时，密钥对不可能每次在程序中重新生成。而keytool恰好可以生成并相对安全保存密钥对。所以下面结合了keytool和java实现的功能。  

## 结合keytool与java签名/验证
[参考](http://docs.oracle.com/javase/tutorial/security/apisign/enhancements.html)  
密钥对由keytool生成并保存到keystore中保护起来(keystore有密码)。公钥也从keystore中导出。GenSig.java类只需要从keystore中取得私钥进行签名即可。  
VerSig.java也要做适当的修改。貌似因为从keystore中导出的是证书而不是公钥，两者的封装格式估计有差异。  
#### 具体步骤
1. 利用`keytool -genkey`生成密钥对保存在keystore中
2. 利用`keytool -export'从keystore中导出公钥证书
3. 利用新类GenSig2.java生成签名，GenSig2.java会从keystore中取私钥
4. 将公钥、签名、被签名文件发给验证方
5. 验证方利用VerSig2.java进行验证
#### GenSig2.java
```java
import java.io.*;
import java.security.*;

class GenSig2 {

    public static void main(String[] args) {

        if (args.length != 1) {
            System.out.println("Usage: java GenSig2 <nameOfFileToSign>");
            }
        else try{

                /*create key paire use keytool:
                $ keytool -genkey -alias signLegal -keystore examplestanstore -validity 1800*/
                // read keystore file
                KeyStore ks = KeyStore.getInstance("JKS");
                FileInputStream ksfis = new FileInputStream("examplestanstore");
                BufferedInputStream ksbufin = new BufferedInputStream(ksfis);

                // open keystore and get private key
                // alias is 'signLeal', kpasswd/spasswd is 'vagrant'
                ks.load(ksbufin, "vagrant".toCharArray());
                PrivateKey priv = (PrivateKey) ks.getKey("signLegal", "vagrant".toCharArray());

            /* Create a Signature object and initialize it with the private key */

            Signature dsa = Signature.getInstance("SHA1withDSA", "SUN");

            dsa.initSign(priv);
            /* Update and sign the data */

            FileInputStream fis = new FileInputStream(args[0]);
            BufferedInputStream bufin = new BufferedInputStream(fis);
            byte[] buffer = new byte[1024];
            int len;
            while (bufin.available() != 0) {
                len = bufin.read(buffer);
                dsa.update(buffer, 0, len);
                };

            bufin.close();

            /* Now that all the data to be signed has been read in,
                    generate a signature for it */
            byte[] realSig = dsa.sign();

            /* Save the signature in a file */
            FileOutputStream sigfos = new FileOutputStream("sig");
            sigfos.write(realSig);

            sigfos.close();

            /* public key file can export from keystore use keytool:
            $  keytool -export -keystore examplestanstore -alias signLegal -file StanSmith.cer */

        } catch (Exception e) {
            System.err.println("Caught exception " + e.toString());
        }
    };
```
编译后，这样运行：  
```
$ java GenSig2 hello.txt
```
会生成签名文件sig。 
 
#### VerSig2.java
```java
import java.io.*;
import java.security.*;
import java.security.spec.*;

class VerSig2 {

    public static void main(String[] args) {

        /* Verify a DSA signature */

        if (args.length != 3) {
            System.out.println("Usage: VerSig publickeyfile signaturefile datafile");
            }
        else try{

            /* import encoded public cert */
            FileInputStream certfis = new FileInputStream(args[0]);
            java.security.cert.CertificateFactory cf =
                java.security.cert.CertificateFactory.getInstance("X.509");
            java.security.cert.Certificate cert =  cf.generateCertificate(certfis);
            PublicKey pubKey = cert.getPublicKey();

            /* input the signature bytes */
            FileInputStream sigfis = new FileInputStream(args[1]);
            byte[] sigToVerify = new byte[sigfis.available()];
            sigfis.read(sigToVerify );

            sigfis.close();

            /* create a Signature object and initialize it with the public key */
            Signature sig = Signature.getInstance("SHA1withDSA", "SUN");
            sig.initVerify(pubKey);

            /* Update and verify the data */

            FileInputStream datafis = new FileInputStream(args[2]);
            BufferedInputStream bufin = new BufferedInputStream(datafis);
            byte[] buffer = new byte[1024];
            int len;
            while (bufin.available() != 0) {
                len = bufin.read(buffer);
                sig.update(buffer, 0, len);
                };

            bufin.close();

            boolean verifies = sig.verify(sigToVerify);

            System.out.println("signature verifies: " + verifies);

        } catch (Exception e) {
            System.err.println("Caught exception " + e.toString());
        };
    }
}
```
编译后，这样运行(StanSmith.cer是利用keytool导出的公钥证书，见前文)：
```
$ java VerSig2 StanSmith.cer sig hello.txt
signature verifies: true
```

## openssl
虽然也研究了一下openssl，但发现与java难以结合，难度也很大。例如它的教程中采用的是RSA，而上面的java使用的是DSA。所以只是贴在这里备忘，可以忽略。  
[参考](http://users.dcc.uchile.cl/~pcamacho/tutorial/crypto/openssl/openssl_intro.html)  
#### 生成私钥
```
$ openssl genrsa -out key.pem 1024
$ cat key.pem
-----BEGIN RSA PRIVATE KEY-----
MIICXQIBAAKBgQCzVDmu6Cf2QF7cERCGYU3B8Epm6pkkpMZFgotphXMgAmBBNJbh
Si7qPH4R5JlEm1ZXPr5DZH/pyJBWQhiiHGeUAOve+GOgvt9Rk25r7OEWYvn/GCr/
JBfLBGqwtlzn/t2s2x04IooshsGkOd6YpZoztkEDtu2gKHedFczF607IvwIDAQAB
AoGAMdbIqUmwQYomUvcTJqXIXIwRwYSVx09cI1lisZL7Kfw/ECAzhq19WHAzgXmM
9zpMxraTXluCCVFKfA6mlfda+ZoBlKSYdOecwNB+TSAumf9XK8uHW/g8C+Ykq9OG
g9Uiy8rKnl12Zaiu9H8L82ud0CkTFW2636/PuKgtp+4YbXECQQDhKdh8lwgumg7H
YIw5476QOHnPL7c3OFPGtaOZMZJkjMPfRzgR4B5PjcGnOLDoTlkATcBPmXtLwwJJ
SzaBdaRjAkEAy+NwdOzC1yQrTrkZQx1brNjO3iytfkl3t1xAWyz5Sy1IB7+4fsod
Eh3br5E1o5YRipY2GJZvp2OAAt3tz6iS9QJASvIYwu+qo4hX3vk9847gwTRrJxFk
1JaFHCEdgUJEzf8ku08DVL/alvRCPxzZlZluenFmz5fwuDkCq87DJ7g2rQJBAMDM
+SnIPdMeA8n0pRvfJjLD7pMP4pu6M3fzx3Owiqj5T9TsCjXzQBxCmdxizzs7DKll
tA/6Kek64PFVFa25tgUCQQCTM1VwfNKjFbd+0HuF6WAs3Odjuo0gKk/QIjdn7M5/
I0kxEApKxTto3oiuCQGeYL/sqy3WjM0476w48+xUsQeF
-----END RSA PRIVATE KEY-----
```
#### 导出公钥
```
$ openssl rsa -in key.pem -pubout -out pub-key.pem
$ cat pub-key.pem
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCzVDmu6Cf2QF7cERCGYU3B8Epm
6pkkpMZFgotphXMgAmBBNJbhSi7qPH4R5JlEm1ZXPr5DZH/pyJBWQhiiHGeUAOve
+GOgvt9Rk25r7OEWYvn/GCr/JBfLBGqwtlzn/t2s2x04IooshsGkOd6YpZoztkED
tu2gKHedFczF607IvwIDAQAB
-----END PUBLIC KEY-----
```
#### 摘要计算
创建一个内容是`1234`的文本文件hello.txt。用openssl计算它的SHA256摘要（SHA256是jarsigner的默认摘要算法）：
```
$ cat hello.txt
1234
$ openssl dgst -SHA256 -out hello.sha256 hello.txt
$ cat hello.sha256
SHA256(hello.txt)= a883dafc480d466ee04e0d6da986bd78eb1fdd2178d04693723da3a8f95d42f4
```
#### 签名和验证
对摘要文件hello.sha256进行签名：
```
$ openssl rsautl -sign -in hello.sha256 -out hello.sign -inkey key.pem
```
用公钥对签名进行验证：
```
$ openssl rsautl -verify -in hello.sign -inkey pub-key.pem -pubin
SHA256(hello.txt)= a883dafc480d466ee04e0d6da986bd78eb1fdd2178d04693723da3a8f95d42f4
```
用公钥验证必须加上`-pubin`参数。
用私钥对签名进行验证：
```
$ openssl rsautl -verify -in hello.sign -inkey key.pem
SHA256(hello.txt)= a883dafc480d466ee04e0d6da986bd78eb1fdd2178d04693723da3a8f95d42f4
```
验证的STD输出与摘要文件hello.sha256的内容一样，说明验证可以通过。  