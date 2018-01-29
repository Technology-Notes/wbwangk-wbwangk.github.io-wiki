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
/etc/hyperledger/fabric-ca-server
sqlite3 fabric-ca-server.db
> .tables
affiliations  certificates  users
> select * from users;
admin|$2a$10$c3NEj9o06ES7RpRbP9e04OZy6eKv7vidAjUw.hCjtLzAL9MqnHumy|client||[{"name":"hf.Registrar.Roles","value":"client,user,peer,validator,auditor"},{"name":"hf.Registrar.DelegateRoles","value":"client,user,validator,auditor"},{"name":"hf.Revoker","value":"1"},{"name":"hf.IntermediateCA","value":"1"}]|2|-1
webb8081|$2a$10$RAG8HarOk6qMzEQLT4mKbeQGmgur0IOf7UkMl1DJhIxCGJ9ZslftK|client|org1|[]|0|1
```
除了CA管理员admin外，webb8081是通过composer-playground界面创建的用户(Identity)。

