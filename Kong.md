[.](/e/vagrant10/ambari-vagrant/centos7.3，c7302)  
kong是微服务API网关。kong底层是nginx，利用lua插件进行扩展实现而达成。nginx的逻辑只能用配置文件配置，而kong可以通过REST API来配置，更灵活和更容易扩展。  
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

先列出example-api的所有插件，找到key-auth插件的id，然后删除：
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
上述两种使用JWT的方法都可以通过验证。可以故意把签名输入错，如删除最后的`g`，再测试：
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

### 管理命令备忘
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