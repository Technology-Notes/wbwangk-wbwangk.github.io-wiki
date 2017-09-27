先[下载rpm包](https://getkong.org/install/centos/#packages)，然后安装：
```
$ sudo yum install epel-release
$ sudo yum install kong-community-edition-0.11.0.*.noarch.rpm --nogpgcheck
```
然后安装postgreSQL9.4+。 一开始直接使用ambari带的postgres，后来才发现那是个9.2版本，不行，只好[手工装postgreSQL](https://www.postgresql.org/download/linux/redhat/)。  
```
$ yum install https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm
$ yum install postgresql96-server
$ /usr/pgsql-9.6/bin/postgresql96-setup initdb
$ systemctl enable postgresql-9.6
$ systemctl start postgresql-9.6
```
刚装上的时候，postgres默认策略是`ident`，即使用操作系统用户来认证，而我喜欢root，所以要改成`md5`。修改配置文件` /var/lib/pgsql/9.6/data/pg_hba.conf`为以下内容：
```
local   all             all                                     md5
host    all             all             127.0.0.1/32           md5
host    all             all             ::1/128                 md5
host all all 0.0.0.0 0.0.0.0 md5
```
最后一项是允许其他节点远程访问postgresql。  
修改完上述配置文件后需要重启服务：
```
$ systemctl restart postgresql-9.6
```
在postgres中为kong创建用户和数据库：
```
$ sudo -u postgres psql
postgres=# CREATE USER kong WITH PASSWORD '1';              (新建一个数据库用户hive，密码是1)
CREATE ROLE
postgres=# CREATE DATABASE kong OWNER kong;                       (创建用户数据库hive，并指定所有者为hive)
CREATE DATABASE
postgre=# \q    
```
配置kong的配置文件`/etc/kong/kong.conf`:
```
proxy_listen = 0.0.0.0:8000
admin_listen = 0.0.0.0:8001  
ssl = off
http2 = off
client_ssl = off 
admin_ssl = off
admin_http2 = off

database = postgres             # Determines which of PostgreSQL or Cassandra
pg_host = 127.0.0.1     # The PostgreSQL host to connect to.
pg_port = 5432                  # The port to connect to.
pg_user = kong                  # The username to authenticate if required.
pg_password = 1
pg_database = kong              # The database name to connect to.
pg_ssl = off                    # Toggles client-server TLS connections
```
对kong的数据库初始化：
```
$ kong migrations up -c /etc/kong/kong.conf
```
启动kong：
```
$ kong start -c /etc/kong/kong.conf
2017/09/26 07:35:44 [warn] ulimit is currently set to "1024". For better performance set it to at least "4096" using "ulimit -n"
```
编辑配置文件`/etc/security/limits.conf`，在文件的最后加上：
```
root soft nofile 4096
root hard nofile 4096
```
需要重新用root登录。然后停止kong后再启动就不出现ulimit的提示了。  
测试kong。8001是管理端口，8000是反向代理端口。在没有后台应用可以反向代理的情况下，8000端口不能用。可以访问8001管理端口来测试：
```
$ curl -i http://localhost:8001/
```
kong的底层是nginx，所以可以通过命令查看kong监听的端口：
```
$ netstat -anp | grep nginx
```
### 测试kong
用[Mockbin API](https://mockbin.com/)充当后台API服务器，添加配置：
```
$ curl -i -X POST --url http://localhost:8001/apis/ \
  --data 'name=example-api' \
  --data 'hosts=c7302.ambari.apache.org' \
  --data 'upstream_url=http://httpbin.org'
```
kong将nginx改造成了通过REST API来配置，更容易扩展。但其管理API的与nginx的配置文件从思想上很一致。  
测试以上配置：
```
$ curl -i -X GET --url http://localhost:8000/ \
  --header 'Host: c7302.ambari.apache.org'
```
按HTTP协议，标头中的`Host`代表了虚拟主机的域名。一个IP上可能有多个虚拟主机域名。一般来说，如果不指定Host，它会被自动设置为URL中的主机域名。  
Kong会将上述请求转发到`http://httpbin.org`，然后将响应发回给curl，并现在屏幕上。  

#### 启用key-auth插件
key-auth可以给API添加认证，可以理解为颁发给开发者的API key。下面给`example-api`这个API启用认证：
```
$ curl -i -X POST \
  --url http://localhost:8001/apis/example-api/plugins/ \
  --data 'name=key-auth'
```
当`example-api`这个API启用了key-auth之后，再象之前那么访问它就会报401错误。

#### 创建消费者
利用管理API创建消费者`webb`:
```
$ curl -i -X POST --url http://localhost:8001/consumers/ \
  --data "username=webb"
{"created_at":1506494530000,"username":"webb","id":"9c270f20-f3e0-4af1-a3a1-91b58f11072c"}
```
为消费者创建凭据(key)：
```
$ curl -i -X POST --url http://localhost:8001/consumers/webb/key-auth/ \
  --data 'key=1'
{"id":"c0ca1b8c-0d38-4b18-a77e-f69c35974c03","created_at":1506494666000,"key":"1","consumer_id":"9c270f20-f3e0-4af1-a3a1-91b58f11072c"}
```
验证一下新建的key：
```
$ curl -i -X GET --url http://localhost:8000 \
  --header "Host: c7302.ambari.apache.org" \
  --header "apikey: 1"
```
屏幕上显示了来自`httpbin.org`的html格式内容。如果故意输错apikey，会返回`403 Forbidden`。

先删除key-auth插件，先找到key-auth插件的id，然后删除：
```
$ curl -X GET http://localhost:8001/apis/example-api/plugins/
{"total":1,"data":[{"created_at":1506468313000,"config":{"key_names":["apikey"],"anonymous":"","hide_credentials":false,"key_in_body":false},"id":"30f745fd-67c3-4fbb-bf09-e031d2a087f1","enabled":true,"api_id":"a52568c1-50fc-4b63-a49e-aa77b2080be6","name":"key-auth"}]}
$ curl -X DELETE http://localhost:8001/apis/example-api/plugins/30f745fd-67c3-4fbb-bf09-e031d2a087f1
```
### JWT插件       
[原文](https://getkong.org/plugins/jwt/)  
为`example-api`启用jwt插件：
```
$ curl -X POST http://localhost:8001/apis/example-api/plugins \
    --data "name=jwt"
{"created_at":1506495552000,"config":{"key_claim_name":"iss","anonymous":"","secret_is_base64":false,"uri_param_names":["jwt"]},"id":"3931733a-149c-4577-aa16-426cf5f92b48","name":"jwt","api_id":"a52568c1-50fc-4b63-a49e-aa77b2080be6","enabled":true}
```
为之前创建的消费者webb创建JWT凭据(使用默认的HS256算法)：
```
$ curl -X POST http://localhost:8001/consumers/webb/jwt -H "Content-Type: application/x-www-form-urlencoded"
{"created_at":1506495780000,"id":"f07eecbb-0b9e-48e4-aeff-d19a81bd5a07","algorithm":"HS256","key":"g5L1BaElWfpxKjS8IJlucEs99TFtoh8Z","secret":"NCKDUmPQtBDocbqu6ZFo0juJlfGNJXvf","consumer_id":"9c270f20-f3e0-4af1-a3a1-91b58f11072c"}
```
可以不停调用上述方法创建多个JWT凭据。下面用`g5L1BaElWfpxKjS8IJlucEs99TFtoh8Z`这个key和`NCKDUmPQtBDocbqu6ZFo0juJlfGNJXvf`这个secret来讲解和测试JWT。  
JWT被两个点(`.`)分隔成三段：第一段(HEADER)是下列json串的BASE64编码：
```
{
    "typ": "JWT",
    "alg": "HS256"
}
```
第二段(PAYLOAD)是下列json串的BASE64编码：
```
{
    "iss": "g5L1BaElWfpxKjS8IJlucEs99TFtoh8Z"
}
```
上面的`iss`值是之前JWT凭据的key。  
第三段是签名，签名段算法伪代码：
```
HMACSHA256(base64UrlEncode($header) + "." +
  base64UrlEncode($payload),$secret)
```
HS256是对称加密算法，所以签名和验证都需要secret。