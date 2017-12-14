
## 寻找管理员的证书和私钥

从`/opt/fabric-samples/basic-network`中查找，想找到系统通道的管理员私钥和证书。先说结论：  
```
$ cd /opt/fabric-samples/basic-network
$ find . -name 'Admin@*' -print
```
这样查找是因为知道管理员的证书的文件的命名习惯。结果：
```
./crypto-config/ordererOrganizations/example.com/msp/admincerts/Admin@example.com-cert.pem
 Serial Number:  54:c5:f3:d5:c7:a3:af:7b:06:80:3b:17:b8:80:4a:df
./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp/admincerts/Admin@example.com-cert.pem
./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/admincerts/Admin@example.com-cert.pem
./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/signcerts/Admin@example.com-cert.pem

./crypto-config/peerOrganizations/org1.example.com/msp/admincerts/Admin@org1.example.com-cert.pem
 Serial Number:  15:2c:67:2c:01:ac:bb:4e:33:ac:59:00:13:0c:e7:eb
./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/admincerts/Admin@org1.example.com-cert.pem
./crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/admincerts/Admin@org1.example.com-cert.pem
./crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem
```
利用openssl命令分别查看以上这些证书文件的内容：
```
$ openssl x509 -noout -text -in <证书文件>
```
可以看到前4个证书的序列号相同，是：
```
Serial Number:  
    54:c5:f3:d5:c7:a3:af:7b:06:80:3b:17:b8:80:4a:df
```
后4个证书的序列号相同，是：
```
Serial Number:  
     15:2c:67:2c:01:ac:bb:4e:33:ac:59:00:13:0c:e7:eb
```
### 寻找私钥，并与证书配对
知道Fabric私钥的文件是以`-sk`结尾的，这样查找：
```
$ cd /opt/fabric-samples/basic-network
$ find . -name '*_sk' -print
```
结果如下：
```
./crypto-config/ordererOrganizations/example.com/ca/a0606a4a860a1e31c90a23788da6f3b6b74925ed0d23061af4899409ba46ae6a_sk
./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp/keystore/4d2f776c0fef8eac3f460a7c3558dc7859c4fe458e262e674a6c23f242ea33d1_sk
./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/keystore/1deeab5433fa6e5f045eb763109d6165268fba153211af1281f00d45f54b1022_sk
./crypto-config/peerOrganizations/org1.example.com/ca/4239aa0dcd76daeeb8ba0cda701851d14504d31aad1b2ddddbac6a57365e497c_sk
./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp/keystore/46be1d569fe68f33e517c9e0072a0ccfbfb42727480fb8c8d0223af321a7893d_sk
./crypto-config/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/keystore/c75bd6911aca808941c3557ee7c97e90f3952e379497dc55eb903f31b50abc83_sk
./crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/cd96d5260ad4757551ed4a5a991e62130f8008a0bf996e4e4b84cd097a747fec_sk
```
上面的这些私钥文件中，只有一个包含了字符串`Admin@example.com`，怀疑它是orderer管理员的私钥。为了证明这一点，可以找个文件用私钥签名一下，然后用公钥去验证，如果验证通过，就说明它们一对匹配的私钥和证书。
工作的目录：
```
$ cd /opt/fabric-samples/basic-network
```
#### 1. 在证书中提取提取公钥
```
$ openssl x509 -pubkey -noout -in /opt/fabric-samples/basic-network/crypto-config/ordererOrganizations/example.com/msp/admincerts/Admin@example.com-cert.pem  > public.pem
```
#### 2. 用私钥对某文件签名
```
$ openssl dgst -ecdsa-with-SHA1 -sign ./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/keystore/1deeab5433fa6e5f045eb763109d6165268fba153211af1281f00d45f54b1022_sk README.md > signature.bin
```
#### 3. 用公钥验证签名
```
$ openssl dgst -ecdsa-with-SHA1 -verify public.pem -signature signature.bin README.md
Verified OK
```
说明假设是成立的，这个文件`./crypto-config/ordererOrganizations/example.com/users/Admin@example.com/msp/keystore/1deeab5433fa6e5f045eb763109d6165268fba153211af1281f00d45f54b1022_sk`是管理员私钥。