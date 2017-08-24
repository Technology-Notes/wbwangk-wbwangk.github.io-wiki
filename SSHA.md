原文：[Java SSHA 匹配算法](http://blog.csdn.net/andy20050125/article/details/24985519)  

改造后的代码：  
```java
import java.io.IOException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

import sun.misc.BASE64Encoder;
import sun.misc.BASE64Decoder;

public class EncryptUtil {

    public static void main(String[] args) throws IOException {


        String original = "{SSHA}xn/c1xL9cEMeTi+aTIqI0X2ftJ1notFppwCk7A==";
        String password = "123456";

        System.out.println(original);
        boolean isSame = verifyPassword(original, password);

        System.out.println(isSame);
    }

    /**
     * Splits a byte array into two.
     *
     * @param src
     *            byte array to split
     * @param n
     *            location to split the array
     * @return a two dimensional array of the split
     */
    private static byte[][] split(byte[] src, int n) {
        byte[] l;
        byte[] r;

        if (src.length <= n) {
            l = src;
            r = new byte[0];

        } else {
            l = new byte[n];
            r = new byte[src.length - n];
            System.arraycopy(src, 0, l, 0, n);
            System.arraycopy(src, n, r, 0, r.length);
        }

        byte[][] lr = { l, r };

        return lr;

    }

    /**
     * Validates if a plaintext password matches a hashed version.
     *
     * @param digest
     *            digested version
     * @param password
     *            plaintext password
     * @return if the two match
     */
    public static boolean verifyPassword(String digest, String password) {
        String alg = null;
        int size = 0;

        if (digest.regionMatches(true, 0, "{SHA}", 0, 5)) {
            digest = digest.substring(5);
            alg = "SHA1";
            size = 20;

        } else if (digest.regionMatches(true, 0, "{SSHA}", 0, 6)) {
            digest = digest.substring(6);
            alg = "SHA1";
            size = 20;

        } else if (digest.regionMatches(true, 0, "{MD5}", 0, 5)) {
            digest = digest.substring(5);
            alg = "MD5";
            size = 16;

        } else if (digest.regionMatches(true, 0, "{SMD5}", 0, 6)) {
            digest = digest.substring(6);
            alg = "MD5";
            size = 16;

        }

        try {
            MessageDigest mDigest = MessageDigest.getInstance(alg);

            if (mDigest == null) {
                return false;
            }
            BASE64Encoder enc = new BASE64Encoder();
            BASE64Decoder decoder = new BASE64Decoder();

            byte[] decodeBase64 = decoder.decodeBuffer(digest);
            byte[][] hs =split(decodeBase64, size);
            byte[] hash = hs[0];
            byte[] salt = hs[1];
String str_hash = new String(hash);
String str_salt = new String(salt);

System.out.println("hash:");
for(int i=0;i<hash.length;i++)
        System.out.format("%02X",hash[i]);
System.out.println();
System.out.println("salt:");
for(int i=0;i<salt.length;i++)
        System.out.format("%02X",salt[i]);
System.out.println();
            mDigest.reset();
            mDigest.update(password.getBytes());
            mDigest.update(salt);

            byte[] pwhash = mDigest.digest();

            return MessageDigest.isEqual(hash, pwhash);

        } catch (NoSuchAlgorithmException nsae) {
            return false;
        } catch (IOException e) {
            e.printStackTrace();
        }

        return false;
    }
}
```
SSHA算法的总结：加密盐salt是8个字节。密码转换成字节，加上8个字节的salt，然后进行SHA-1散列。散列值base64后，最前面加上`{SSHA}`。  
解析：去掉`{SSHA}`,进行base64解码。得到28个字节，最后8个字节是salt。前面20个字节是SHA-1散列值。散列值是密码和salt连接后SHA-1计算得到的。  