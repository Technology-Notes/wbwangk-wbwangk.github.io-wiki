[.](/e/vagrant10/ambari-vagrant/centos7.3，c7302)  
kong是微服务API网关。kong底层是nginx，利用lua插件进行扩展实现而达成。nginx的逻辑只能用配置文件配置，而kong可以通过REST API来配置，更灵活和更容易扩展。  
## kong安装
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
上面的配置中，注意8000和8001端口。8000端口是个运行时端口，是实际的网关工作端口。8001端口是配置端口，或者叫admin端口，用于配置Kong。  
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
## kong快速开始

### 添加API
在添加API时，有三个参数很关键：`hosts`、`uris`、`methods`，三个参数至少需要一个，还可以是两个或三个。  
#### hosts参数测试
`hosts`参数的含义是：将发往虚拟主机`{hosts}`的请求转发到`{upstream_url}`。  
用[Mockbin API](https://mockbin.com/)充当后台API服务器，添加配置：
```
$ curl -i -X POST --url http://localhost:8001/apis/ \
  --data 'name=example-api' \
  --data 'hosts=c7302.ambari.apache.org' \
  --data 'upstream_url=http://httpbin.org'
```
解释：上述命令向kong的管理API发送请求，创建了一个名叫`example-api`的API，在该API中监听发往主机`c7302.ambari.apache.org`的请求，并代理到地址`http://httpbin.org`。  
kong将nginx改造成了通过REST API来配置，更容易扩展。但其管理API的与nginx的配置文件从思想上很一致。  
测试以上配置：
```
$ curl -i -X GET --url http://localhost:8000/ \
  --header 'Host: c7302.ambari.apache.org'
```
按HTTP协议，标头中的`Host`代表了虚拟主机的域名。一个IP上可能有多个虚拟主机域名。一般来说，如果不指定Host，它会被自动设置为URL中的主机域名。  
Kong会将上述请求转发到`http://httpbin.org`，然后将响应发回给curl，并现在屏幕上。  

#### uri参数测试
`uris`参数的含义是：将发往路径`{uris}`的请求转发到`{upstream_url}`。  
下面的例子中创建的API(`example-api2`)将发送到`/my-path`路径的请求转发到`http://webdav.imaicloud.com/`。如何设置`strip_uri=false`则发送到`/my-path`的请求会转发到`http://webdav.imaicloud.com/my-path`。  
新增API:
```
$ curl -i -X POST --url http://localhost:8001/apis/ \
  --data 'name=example-api2' \
  --data 'uris=/my-path' \
  --data 'upstream_url=http://webdav.imaicloud.com/'
$ curl -X GET --url http://localhost:8000/my-path
(显示了http://webdav.imaicloud.com/目录下的文件清单)
```
#### 三个参数的组合
假如将`hosts`和`uris`进行组合的含义是：将发往`{hosts}`虚拟主机的`{uris}`路径的请求转发到`{upstream_url}`
#### API删除、修改
现在的Kong修改API有bug。  
修改API相比增加API，需要在请求体中增加id参数，以便精确定位API。  
可以用下面的办法查询API清单(里面有id)：
```
$ curl http://localhost:8001/apis/ | jq  
```
查到API的id后就可以删除它了：
```
$ curl -X DELETE http://localhost:8001/apis/920579a0-e6d1-4479-bf9f-f221da81f083
```
## Kong插件系统
Kong支持插件扩展。可以到[插件库](https://konghq.com/plugins/)查看Kong的可用插件清单， 其中的企业版插件需要付费，这与Nginx的商业策略类似。  
插件可以添加到全局(所有API)、到某API、到某个消费者、到某个API与某个消费者的组合。当进行组合配置时，最终的执行优先顺序从最高到最低：  
1. 插件应用于API和消费者的组合（如果请求被认证）。  
2. 应用于消费者的插件（如果请求被认证）。  
3. 插件应用于API。  
4. 插件配置为全局运行。  
当插件被添加多次时，每次的配置可能不一样，则只有优先级高的被执行。  

您可以通过四种不同的方式添加插件：  
- 对于每个API和消费者。不要设置api_id和consumer_id。  
- 对于每个API和特定的消费者。只有设置consumer_id。  
- 对于每个消费者和特定的API。只有设置api_id。  
- 对于特定的消费者和API。同时设置api_id和consumer_id。  
请注意，并非所有插件都允许指定consumer_id。检查插件文档。
为特定API添加插件的语法：
```
POST /apis/{API-name or API-id}/plugins/
```
为全部API(全局)添加插件的语法：
```
POST /plugins/
```
插件名称、消费者等通过“请求正文”传递：  
属性 | 描述
-----|------
name | 将要添加的插件的名称。目前插件必须分别安装在每个Kong实例中
consumer_id(可选) | 消费者的唯一标识符，覆盖传入请求中此特定消费者的现有设置
config.{property} | 插件的配置属性，可以在具体插件的文档页面找到

### 基础认证插件
向`example-api`API添加`basic-auth`插件：
```
$ curl -X POST http://localhost:8001/apis/example-api/plugins \
    --data "name=basic-auth"
{"created_at":1509703102000,"config":{"hide_credentials":false,"anonymous":""},"id":"b191aec9-5170-4e47-80de-6a6ab2c0735a","name":"basic-auth","api_id":"a52568c1-50fc-4b63-a49e-aa77b2080be6","enabled":true}
```
如果想为所有API都启用"基础认证"插件，则需要这样：
```
$ curl -X POST http://localhost:8001/plugins \
    --data "name=basic-auth"
```
#### 创建凭据
`webb`消费者添加两个用户：`tom`和`jack`：
```
 curl -X POST http://localhost:8001/consumers/webb/basic-auth \
>     --data "username=tom" \
>     --data "password=1"
{"created_at":1509703515000,"id":"848ad6fc-0176-4460-a367-6591a7f79e21","username":"tom","password":"3ae99f595e0397d0a2982e24001e47c3aaa6c624","consumer_id":"9c270f20-f3e0-4af1-a3a1-91b58f11072c"}
[root@c7302 kong]# curl -X POST http://localhost:8001/consumers/webb/basic-auth     --data "username=jack"     --data "password=1"
{"created_at":1509703533000,"id":"07147481-87c6-4e47-940a-75eaec42476f","username":"jack","password":"3ae99f595e0397d0a2982e24001e47c3aaa6c624","consumer_id":"9c270f20-f3e0-4af1-a3a1-91b58f11072c"}
```
`example-api`API添加了基础认证插件后，如果再向其发送请求，则会提示401错误：
```
$ curl -i -X GET --url http://localhost:8000/ 
         --header 'Host: c7302.ambari.apache.org'
{"message":"Unauthorized"}
```
下面在请求中增加基础认证：
```
$ curl http://localhost:8000/ \
    --header 'Host: c7302.ambari.apache.org' \
    -H 'Authorization: Basic dG9tOjEK'
```
curl -X POST http://localhost:8001/consumers/webb/basic-auth \
    --data "username=Aladdin" \
    --data "password=OpenSesame"
{"created_at":1509704641000,"id":"98466068-c0ff-483e-8baa-b0fdceb6ef23","username":"Aladdin","password":"e3faa8bcbf417f3aabf364eeba0f5ca2ee5a3cff","consumer_id":"9c270f20-f3e0-4af1-a3a1-91b58f11072c"}
 curl http://localhost:8000/ \
    -H 'Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l'
### key-auth插件
key-auth可以给API添加“基础认证” 。为kong API添加插件的语法是：
```
POST /apis/{name or id}/plugins/
```
下面的命令为`example-api`这个API增加`key-auth`插件：
```
$ curl -i -X POST \
  --url http://localhost:8001/apis/example-api/plugins/ \
  --data 'name=key-auth'
```
当`example-api`这个API启用了key-auth之后，再象之前那么访问它就会报401错误。关于“kong插件API”可参考[这个文档](https://getkong.org/docs/0.11.x/admin-api/#plugin-object)。  

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

先列出example-api的所有插件，找到key-auth插件的id，然后删除：
```
$ curl -X GET http://localhost:8001/apis/example-api/plugins/
{"total":1,"data":[{"created_at":1506468313000,"config":{"key_names":["apikey"],"anonymous":"","hide_credentials":false,"key_in_body":false},"id":"30f745fd-67c3-4fbb-bf09-e031d2a087f1","enabled":true,"api_id":"a52568c1-50fc-4b63-a49e-aa77b2080be6","name":"key-auth"}]}
$ curl -X DELETE http://localhost:8001/apis/example-api/plugins/30f745fd-67c3-4fbb-bf09-e031d2a087f1
```
### JWT插件       
[原文](https://getkong.org/plugins/jwt/)  

为Kong API`example-api`增加jwt插件：
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
HS256是对称加密算法，所以签名和验证都需要secret。[这篇文章](https://github.com/wbwangk/wbwangk.github.io/wiki/java%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95)是上述伪代码的java实现，而且使用了本文的JWT头和载荷为算法的输入。  
可以使用[https://jwt.io](https://jwt.io/)的JWT调试器来生成或验证这个JWT，生成的签名与java生成的一样。  
在上述网页中，注意把`VERIFY SIGNATURE`域的secret替换成`NCKDUmPQtBDocbqu6ZFo0juJlfGNJXvf`，即替换成自己的密钥。HEADER和PAYLOAD也替换成前文。然后网页会自动计算出下面的JWT：
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnNUwxQmFFbFdmcHhLalM4SUpsdWNFczk5VEZ0b2g4WiJ9.wc0tE4XSb-iYxBs9a_XWgT0btABQM6JyWCHpSlleUlg
```
上面第二个点后面的就是签名，与java程序生成的签名(其实是个散列结果)只差了一个`=`，而`=`是个填充字符，不影响校验。  

验证JWT:
```
$ curl http://localhost:8000   --header "Host: c7302.ambari.apache.org" -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnNUwxQmFFbFdmcHhLalM4SUpsdWNFczk5VEZ0b2g4WiJ9.wc0tE4XSb-iYxBs9a_XWgT0btABQM6JyWCHpSlleUlg'
$ curl --header "Host: c7302.ambari.apache.org" http://localhost:8000?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnNUwxQmFFbFdmcHhLalM4SUpsdWNFczk5VEZ0b2g4WiJ9.wc0tE4XSb-iYxBs9a_XWgT0btABQM6JyWCHpSlleUlg
```
上述两种使用JWT的方法都可以通过验证，前者将JWT令牌放在标头，后者放在URL参数中。可以故意把签名输入错，如删除最后的`g`，再测试：
```
$ curl --header "Host: c7302.ambari.apache.org" http://localhost:8000?0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnNUwxQmFFbFdmcHhLalM4SUpsdWNFczk5VEZ0b2g4WiJ9.wc0tE4XSb-iYxBs9a_XWgT0btABQM6JyWCHpSlleUl
{"message":"Invalid signature"}
```
kong API的创建需要`hosts`、`uris`或`methods`三个参数的组合。在`example-api`这个例子API中的使用`host`定义的，所以在验证JWT时必须在请求的标头中指定`Host: c7302.ambari.apache.org`。为了验证这一点，可以故意把前文的Host输入错误(`org`改成了`com`)，如：
```
$ curl --header "Host: c7302.ambari.apache.com" http://localhost:8000?0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnNUwxQmFFbFdmcHhLalM4SUpsdWNFczk5VEZ0b2g4WiJ9.wc0tE4XSb-iYxBs9a_XWgT0btABQM6JyWCHpSlleUl
```  
然后发现验证通过了，因为没有定义这个kong API，JWT插件自然不起作用，所以没有JWT验证，即使签名错误也可以通过。  

#### 验证声明(claim)
Kong还可以对[RFC 7519](https://tools.ietf.org/html/rfc7519)中定义的注册声明进行验证。要对声明执行验证，请将其添加到`config.claims_to_verify`属性。下面是相关语法：
```
$ curl -X PATCH http://kong:8001/apis/{api}/plugins/{jwt plugin id} \
    --data "config.claims_to_verify=exp,nbf"
```
为了查到{jw plugin id}，可以先试用GET方法查询到插件id，然后再执行PATCH方法进行设置(插件id是`3931733a-149c-4577-aa16-426cf5f92b48`)：
```
$ curl -X GET http://localhost:8001/apis/example-api/plugins/
...
      "id": "3931733a-149c-4577-aa16-426cf5f92b48",
...
$ curl -X PATCH http://localhost:8001/apis/example-api/plugins/3931733a-149c-4577-aa16-426cf5f92b48 \
     --data "config.claims_to_verify=exp,nbf" 
{
  "created_at": 1506495552000,
  "config": {
    "claims_to_verify": [
      "exp",
      "nbf"
    ],
    "key_claim_name": "iss",
    "anonymous": "",
    "secret_is_base64": false,
    "uri_param_names": [
      "jwt"
    ]
  },
  "id": "3931733a-149c-4577-aa16-426cf5f92b48",
  "name": "jwt",
  "api_id": "a52568c1-50fc-4b63-a49e-aa77b2080be6",
  "enabled": true
}
```

设置了`exp`和`nbf`声明验证后，再执行之前的API:
```
$  curl http://localhost:8000   --header "Host: c7302.ambari.apache.org" -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnNUwxQmFFbFdmcHhLalM4SUpsdWNFczk5VEZ0b2g4WiJ9.wc0tE4XSb-iYxBs9a_XWgT0btABQM6JyWCHpSlleUlg'
{"nbf":"must be a number","exp":"must be a number"}
```
这是因为之前定义的JWT中的声明只有一个`iss`，没有定义`exp`和`nbf`(`exp`是令牌的过期时间，令牌不能在`nbf`定义的时间之前使用)。现在重新定义载荷为：
```
{
    "iss": "g5L1BaElWfpxKjS8IJlucEs99TFtoh8Z",
    "nbf":1506562106,
    "exp":1506565706
}
```
将上面的载荷复制到`jwt.io`网页重新生成JWT。需要说明的是，上面exp有效期只有一个小时，你无法直接使用，可以用[这个在线工具](http://kjur.github.io/jsjws/mobile/tool_jwt.html#signer)自己生成一个(需要base64解码)。  

用新的JWT重新请求API，可以正常返回响应了：
```
curl http://localhost:8000   --header "Host: c7302.ambari.apache.org" -H ization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnNUwxQmFFbFdmcHhLalM4SUpsdWNFczk5VEZ0b2g4WiIsIm5iZiI6MTUwNjU2MjEwNiwiZXhwIjoxNTA2NTY1NzA2fQ.EBsK5FPDEKcC34BuKLdM5l3LDk9rAtAgVkZIFPUXLts'
```
#### 用公钥/私钥（RS256或ES256）制作JWT
要创建一对全新的公钥/私钥，可以运行以下命令：
```
$ openssl genrsa -out private.pem 2048
```
这个私钥必须保密。要生成与私钥对应的公钥，请执行以下命令：
```
$ openssl rsa -in private.pem -outform PEM -pubout -out public.pem
```
如果您运行上面的命令，公钥将被写入`public.pem`，而私钥将被写入`private.pem`。

与之前的HS256格式JWT凭据相比，非对称算法JWT凭据在创建时需要上传公钥(webb是消费者)：
```
$ curl -X POST http://localhost:8001/consumers/webb/jwt \
      -F "rsa_public_key=@public.pem"

{
  "rsa_public_key": "-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAt8inE3+rJMtppDKfY42r\ncc/bVJveGkgYfI9C/+oQjonwMAUYnRvs78znCnCHHjdTo8orso2NvqSAm1WPC6h6\nXR3X9R2WNGtmIuGdIKhBifgq3QKLjBS0H0wRbx2ME4zXG/Zk/umBhbBMOoSeHhnn\nOM0L4xa9twCmdknn48JwBqasLxQv9jvoA1yfy8QDGRkWaciaskUCWqLYPMCNI0Sw\nmQXbBcCuDx4WLIaqLm+Eli4mm+kpZPZ2Vz0hcpbx9ykuGQl277WjfQ59P0RA7/ms\n+T47VGi2DX+tfwWdj3P5lCTHr3ABppUEoHKr3LjZUZNbcP52ZmRlt/RWMGMBYC+b\n2wIDAQAB\n-----END PUBLIC KEY-----",
  "created_at": 1506564261000,
  "id": "82360c6b-f4a3-420d-97d9-52a8ff43deee",
  "algorithm": "HS256",
  "key": "JL8mNC7PZjrQiJpmBqy3xwP4SIvYm43v",
  "secret": "Jy0WTzITJieAvvh1RZ6l8QRZvKnIqPHZ",
  "consumer_id": "9c270f20-f3e0-4af1-a3a1-91b58f11072c"
}
```
当上述命令成功后，webb用户已经有两个JWT凭据：`HS256`和`RS256`各一个，可以用下列命令查看：
```
$ curl -X GET http://localhost:8001/consumers/webb/jwt
{
  "total": 2,
  "data": [
    {
      "created_at": 1506495780000,
      "id": "f07eecbb-0b9e-48e4-aeff-d19a81bd5a07",
      "algorithm": "HS256",
      "key": "g5L1BaElWfpxKjS8IJlucEs99TFtoh8Z",
      "secret": "NCKDUmPQtBDocbqu6ZFo0juJlfGNJXvf",
      "consumer_id": "9c270f20-f3e0-4af1-a3a1-91b58f11072c"
    },
    {
      "rsa_public_key": "-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAt8inE3+rJMtppDKfY42r\ncc/bVJveGkgYfI9C/+oQjonwMAUYnRvs78znCnCHHjdTo8orso2NvqSAm1WPC6h6\nXR3X9R2WNGtmIuGdIKhBifgq3QKLjBS0H0wRbx2ME4zXG/Zk/umBhbBMOoSeHhnn\nOM0L4xa9twCmdknn48JwBqasLxQv9jvoA1yfy8QDGRkWaciaskUCWqLYPMCNI0Sw\nmQXbBcCuDx4WLIaqLm+Eli4mm+kpZPZ2Vz0hcpbx9ykuGQl277WjfQ59P0RA7/ms\n+T47VGi2DX+tfwWdj3P5lCTHr3ABppUEoHKr3LjZUZNbcP52ZmRlt/RWMGMBYC+b\n2wIDAQAB\n-----END PUBLIC KEY-----",
      "created_at": 1506564261000,
      "id": "82360c6b-f4a3-420d-97d9-52a8ff43deee",
      "algorithm": "RS256",
      "key": "JL8mNC7PZjrQiJpmBqy3xwP4SIvYm43v",
      "secret": "Jy0WTzITJieAvvh1RZ6l8QRZvKnIqPHZ",
      "consumer_id": "9c270f20-f3e0-4af1-a3a1-91b58f11072c"
    }
  ]
}
```
需要强调的是：Kong的jwt插件负责检验JWT，但不负责生成JWT。为了测试jwt插件检验RS256算法JWT的能力，需要手工生成JWT。仍可以使用在线工具[https://jwt.io](https://jwt.io/)。    
通过[https://jwt.io](https://jwt.io/)创建JWT时，要保证header是：
```
{
    "typ": "JWT",
    "alg": "RS256"
}
```
在载荷中需要指定`iss`为JWT凭据id(`"key": "JL8mNC7PZjrQiJpmBqy3xwP4SIvYm43v"`):
```
{
    "iss": "JL8mNC7PZjrQiJpmBqy3xwP4SIvYm43v"
}
```
在[https://jwt.io](https://jwt.io/)的下拉框中选择RS256，然后把私钥(private.pem)的全部文本复制到私钥输入框，把上面的header和payload(载荷)分别复制到相应输入框。这时JWT已经生成，如何要验证，就再把公钥(public.pem)的全部文本复制到公钥输入框。如果一切正常，签名应该可以验证通过。
生成了JWT后，用下列方式测试：
```
$ curl http://localhost:8000 -H "Host: c7302.ambari.apache.org" -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJKTDhtTkM3UFpqclFpSnBtQnF5M3h3UDRTSXZZbTQzdiJ9.SfVOXuk-0-VELm4QV1nPteJC9toXtH2xqqhClBueO-zrGn5K2lCsPjku2TwGMiLYFVLqfzFSjNlLuCNZIjpzSW3Hq7fjeEcVRyVhrdwAMOJm0SWRL9x5BOQIK_anLvoQA2ONTKFzD61wLlnEpnGK3vSh6sFquqd8rTOxii7DK9nHGP1svpymetB0AO5ma-wQuK_6fVOzuxdqK3bwigyRjByHrsZaQIS6U8Rx_KTtAE-5T-siBkGRdlSq7wxNvaYvsnMykfJs6b-hXkEb4ErOT-qfX4KVjG9JvgjD3dGYcuDYxMgbIfPat9b1913_exgt9UtjklRpJaWy5iVa0UtSGQ'

{"message":"Invalid algorithm"}
```
可以通过命令查看该JWT凭据明细，发现算法类似是`HS256`，正确的应该是`RS256`。使用HTTP `PATCH`方法将算法类型改成`RS256`。（说明创建该JWT凭据时参数不全，这是原文的BUG）
```
$ curl -X GET http://localhost:8001/consumers/webb/jwt/JL8mNC7PZjrQiJpmBqy3xwP4SIvYm43v
$ curl -X PATCH http://localhost:8001/consumers/webb/jwt/JL8mNC7PZjrQiJpmBqy3xwP4SIvYm43v \
  --data "algorithm=RS256"
```
重新执行前文的curl测试，发现提示JWT声明中`exp`、`nbf`没有定义。运行下列命令将jwt插件的`exp`、`nbf`声明验证去掉：
```
$ curl -X PATCH http://localhost:8001/apis/example-api/plugins/3931733a-149c-4577-aa16-426cf5f92b48 \
     --data "config.claims_to_verify="  
$ curl http://localhost:8000 -H "Host: c7302.ambari.apache.org" -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJKTDhtTkM3UFpqclFpSnBtQnF5M3h3UDRTSXZZbTQzdiJ9.SfVOXuk-0-VELm4QV1nPteJC9toXtH2xqqhClBueO-zrGn5K2lCsPjku2TwGMiLYFVLqfzFSjNlLuCNZIjpzSW3Hq7fjeEcVRyVhrdwAMOJm0SWRL9x5BOQIK_anLvoQA2ONTKFzD61wLlnEpnGK3vSh6sFquqd8rTOxii7DK9nHGP1svpymetB0AO5ma-wQuK_6fVOzuxdqK3bwigyRjByHrsZaQIS6U8Rx_KTtAE-5T-siBkGRdlSq7wxNvaYvsnMykfJs6b-hXkEb4ErOT-qfX4KVjG9JvgjD3dGYcuDYxMgbIfPat9b1913_exgt9UtjklRpJaWy5iVa0UtSGQ'
```
发现终于正常显示了网页信息。

#### 删除JWT插件
先查看插件清单，找到jwt插件的id，然后调用DELETE方法删除jwt插件：
```
$ curl http://localhost:8001/apis/example-api/plugins/
...
    "id": "3931733a-149c-4577-aa16-426cf5f92b48",
      "enabled": true,
      "api_id": "a52568c1-50fc-4b63-a49e-aa77b2080be6",
      "name": "jwt"
...
$ curl -X DELETE http://localhost:8001/apis/example-api/plugins/3931733a-149c-4577-aa16-426cf5f92b48
```
### IP限制插件
为了测试IP限制插件，首先要删除JWT插件等影响测试插件，具体方法见上文。  
为`example-api`API启用"IP限制"插件：
```
$ curl -X POST http://localhost:8001/apis/example-api/plugins \
    --data "name=ip-restriction" \
    --data "config.whitelist=192.168.73.102,127.0.0.1"
```
启用IP限制插件，同时告诉了插件IP白名单。这个插件貌似没有提供“更新”API，所以如果要更改白名单，只能先删除再添加。删除方法见上一节的“删除JWT插件”。
Kong所在VM的IP就是`192.168.73.102`，所以执行下列调用是可以的：
```
$ curl -i -X GET --url http://c7302.ambari.apache.org:8000/   --header 'Host: c7302.ambari.apache.org'
```
如果换到另一个IP为`192.168.73.103`的虚拟机，再执行上述调用，则显示：
```
{"message":"Your IP address is not allowed"}
```
### middleman子请求插件
在调用API前，[middleman](https://github.com/pantsel/kong-middleman-plugin)插件使得Kong向一个地址发送额外的HTTP请求。似乎是把Nginx的[ngx_http_auth_request_module](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html)模块的发送认证“子请求”(subrequest)的功能搬运到了Kong中。  
middleman插件可用于授权检查。在调用真正的Kong API前，利用middleman向某个权限检查的地址发送子请求。如果子请求返回2xx的响应，Kong API会被继续调用。如果子请求返回401或403，向API发出的请求会终止。  

#### 插件安装
middleman不是官方插件，而是一个定制插件。使用前需要先安装。假定kong安装目录是`/usr/local/kong`。  
```
$ cd /usr/local/kong  && mkdir plugins && cd plugins
$ git clone https://github.com/pantsel/kong-middleman-plugin middleman
$ cd middleman
$ uarocks make *.rockspec
```
然后配置kong以便加载定制插件。编辑`/etc/kong/kong.conf`，添加下列配置（如果有多个定制插件需要配置，就逗号隔开）：
```
custom_plugins = middleman 
```
重启Kong。
#### 插件测试（首先是子请求状态码401）
http://httpbin.org是一个免费的http调试云服务（首页是帮助）。测试用到下面的两个URI，一个返回200状态码，另一个返回401状态码：
```
$ curl -i http://httpbin.org/status/200
HTTP/1.1 200 OK
$ curl -i http://httpbin.org/status/401
HTTP/1.1 401 UNAUTHORIZED
```
在下面的测试中，向API `example-api2`添加middleman插件。首先让httpbin.org返回401状态码，然后测试让httpbin.org返回200状态码。由于当前版本的Kong的插件更新(PATCH方法)存在bug，只能删除后重新添加middleman插件来修改`config.url`。
```
$ curl -X POST http://localhost:8001/apis/example-api2/plugins \
    --data "name=middleman" \
    --data "config.url=http://httpbin.org/status/401"
```
`example-api2`API的路径(`uris`)是`/my-path`，下面是测试该路径：
```
$ curl http://localhost:8000/my-path
{"message":""}
```
如果`example-api2`API正常执行应返回`http://webdav.imaicloud.com`的网页，现在只显示了个空的`message`，说明401的状态码阻止了API的正常执行。 

#### 测试子请求返回200状态码
参考下列命令查询出middleman的插件id，删除它：
```
$ curl localhost:8001/plugins/     (找到插件middleman的id)
$ curl -X DELTE localhost:8001/plugins/4e19f07c-328f-454d-9d03-4ca66bad63fc
```
重新向`example-api2`添加middleman插件，让httpbin.org返回的子请求状态码是200：
```
$ curl -X POST http://localhost:8001/apis/example-api2/plugins \
    --data "name=middleman" \
    --data "config.url=http://httpbin.org/status/200"
```
再次测试`example-api2`API:
```
$ curl http://localhost:8000/my-path
```
可以看到正常返回了`http://webdav.imaicloud.com`网页的内容。  

上述测试说明middleman插件可以用于Kong外部的认证、权限检查等功能。  

## 管理命令备忘
查询API清单，并删除一种一个：
```
$ curl -X GET http://localhost:8001/apis          (查询出API的id)
$ curl -X DELETE http://localhost:8001/apis/{API id}
```
查询某消费者的JWT凭据清单：
```
$ curl -X GET http://localhost:8001/consumers/{consumer}/jwt
$ curl -X GET http://localhost:8001/consumers/webb/jwt
```
查询某消费者的某JWT凭据明细：
```
$ curl -X GET http://localhost:8001/consumers/{consumer}/jwt/{id}
$ curl -X GET http://localhost:8001/consumers/webb/jwt/JL8mNC7PZjrQiJpmBqy3xwP4SIvYm43v
```
删除一个JWT凭据：
```
$ curl -X DELETE http://localhost:8001/consumers/{consumer}/jwt/{id}
$ curl -X DELETE http://localhost:8001/consumers/webb/jwt/SJqJnm1cPXpqlmT9QRp8srcKZtHR36Yx
```
更新一个JWT凭据属性：
```
$ curl -X POST http://kong:8001/consumers/{consumer}/jwt/{id} \
  --data "algorithm=RS256"
$ curl -X PATCH http://localhost:8001/consumers/webb/jwt/JL8mNC7PZjrQiJpmBqy3xwP4SIvYm43v \
  --data "algorithm=RS256"
```

[KONG API DOC](https://getkong.org/docs/0.11.x/admin-api/)  