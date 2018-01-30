进入fabric-ca容器：
```
docker exec -it ca.org1.example.com bash
```
在容器内，fabric-ca的配置文件位于目录`/etc/hyperledger/fabric-ca-server`下。

fabric-ca默认数据库是sqlite3。sqlite的数据存储在文件中，数据库文件也在这个目录下。

可以安装sqlite3命令行工具，以便查看ca数据库中的数据：
```
apt update
apt install sqlite3
cd /etc/hyperledger/fabric-ca-server
sqlite3 fabric-ca-server.db
> .tables
affiliations  certificates  users
> select * from users;
admin|$2a$10$c3NEj9o06ES7RpRbP9e04OZy6eKv7vidAjUw.hCjtLzAL9MqnHumy|client||[{"name":"hf.Registrar.Roles","value":"client,user,peer,validator,auditor"},{"name":"hf.Registrar.DelegateRoles","value":"client,user,validator,auditor"},{"name":"hf.Revoker","value":"1"},{"name":"hf.IntermediateCA","value":"1"}]|2|-1
webb8081|$2a$10$RAG8HarOk6qMzEQLT4mKbeQGmgur0IOf7UkMl1DJhIxCGJ9ZslftK|client|org1|[]|0|1
```
除了CA管理员admin外，webb8081是通过composer-playground界面创建的用户(Identity)。

### enroll和register

要向CA注册用户，必须先用admin登录（enroll）。下列命令在ca.org1.example.com的容器内执行：
```
$ fabric-ca-client enroll -u http://admin:adminpw@localhost:7054
054t@90dda54c2970:/# fabric-ca-client enroll -u http://admin:adminpw@localhost:7054
2018/01/30 02:08:50 [INFO] User provided config file: /etc/hyperledger/fabric-ca-server/fabric-ca-client-config.yaml
2018/01/30 02:08:50 [INFO] Created a default configuration file at /etc/hyperledger/fabric-ca-server/fabric-ca-client-config.yaml
2018/01/30 02:08:50 [INFO] generating key: &{A:ecdsa S:256}
2018/01/30 02:08:50 [INFO] encoded CSR
2018/01/30 02:13:22 [INFO] Stored client certificate at /etc/hyperledger/fabric-ca-server/msp/signcerts/cert.pem
2018/01/30 02:13:22 [INFO] Stored CA root certificate at /etc/hyperledger/fabric-ca-server/msp/cacerts/localhost-7054.pem
```
然后注册一个新用户webb2：
```
fabric-ca-client register --id.name webb2 --id.type user --id.affiliation org1.department1 --id.attrs hf.Revoker=true
2018/01/30 02:14:37 [INFO] User provided config file: /etc/hyperledger/fabric-ca-server/fabric-ca-client-config.yaml
2018/01/30 02:14:37 [INFO] Configuration file location: /etc/hyperledger/fabric-ca-server/fabric-ca-client-config.yaml
Password: DkNmmaqMDsGC
```
然后可以用新用户登录：
```
fabric-ca-client enroll -u http://webb2:DkNmmaqMDsGC@localhost:7054
```
可以用前文的sqlite3命令`select * from users;`查看新增加的用户。