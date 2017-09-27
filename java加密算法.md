## HMAC SHA256
[原文](http://www.cnblogs.com/rubekid/p/5989912.html)  
ApiSecurityExample.java源码如下：
```
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import org.apache.commons.codec.binary.Base64;

public class ApiSecurityExample {
  public static void main(String[] args) {
    try {
     String secret = "NCKDUmPQtBDocbqu6ZFo0juJlfGNJXvf";
     String message = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnNUwxQmFFbFdmcHhLalM4SUpsdWNFczk5VEZ0b2g4WiJ9";

     Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
     SecretKeySpec secret_key = new SecretKeySpec(secret.getBytes(), "HmacSHA256");
     sha256_HMAC.init(secret_key);

     String hash = Base64.encodeBase64String(sha256_HMAC.doFinal(message.getBytes()));
     System.out.println(hash);
    }
    catch (Exception e){
     System.out.println("Error");
    }
   }
}
```
源码中的message是一个JWT的头和载荷。  
需要下载apache的包：
```
$ wget http://www-us.apache.org/dist//commons/codec/binaries/commons-codec-1.10-bin.zip
$ unzip commons-codec-1.10-bin.zip
$ javac -cp ".:./commons-codec-1.10/commons-codec-1.10.jar" ApiSecurityExample.java
$ java -cp ".:./commons-codec-1.10/commons-codec-1.10.jar" ApiSecurityExample
wc0tE4XSb+iYxBs9a/XWgT0btABQM6JyWCHpSlleUlg=
```
unzip自动创建了目录`commons-codec-1.10`目录，解压出jar包。`-cp`参数指明了类路径。而输出的`wc0tE4XSb+iYxBs9a/XWgT0btABQM6JyWCHpSlleUlg=`是JWT的签名。
这个签名同`https://jwt.io/`计算出的一样。