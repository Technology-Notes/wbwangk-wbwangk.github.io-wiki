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