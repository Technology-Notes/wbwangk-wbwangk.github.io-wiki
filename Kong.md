[.](/e/vagrant10/ambari-vagrant/centos7.3，c7302)  
kong是微服务API网关。kong底层是nginx，利用lua插件进行扩展实现而达成。nginx的逻辑只能用配置文件配置，而kong可以通过REST API来配置，更灵活和更容易扩展。  
## kong安装
参考官方[安装文档](https://getkong.org/install/centos/#packages)

在Packages一节中有个centos7图标，图标链接：`https://bintray.com/kong/kong-community-edition-rpm/download_file?file_path=dists/kong-community-edition-0.11.1.el7.noarch.rpm`。  
下载rpm包：
```
$ wget -O kong-community-edition-0.11.1.el7.noarch.rpm https://bintray.com/kong/kong-community-edition-rpm/download_file?file_path=dists/kong-community-edition-0.11.1.el7.noarch.rpm
```
然后安装：
```
$ sudo yum install epel-release
$ sudo yum install kong-community-edition-0.11.1.el7.noarch.rpm
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
(250.249: passwd postgres, diadiadia3D)  
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

#### uris参数测试
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
请求应该可以正常返回响应了。`dG9tOjEK`是`tom:1`的base64编码。  

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
为消费者创建apikey：
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
参见[Kong_JWT](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong_JWT)  

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
### middleman插件
在调用API前，[middleman](https://github.com/pantsel/kong-middleman-plugin)插件使得Kong向一个地址发送额外的HTTP请求。似乎是把Nginx的[ngx_http_auth_request_module](http://nginx.org/en/docs/http/ngx_http_auth_request_module.html)模块的发送认证“子请求”(subrequest)的功能搬运到了Kong中。  
middleman插件可用于授权检查。在调用真正的Kong API前，利用middleman向某个权限检查的地址发送子请求。如果子请求返回2xx的响应，Kong API会被继续调用。如果子请求返回401或403，向API发出的请求会终止。  

#### 插件安装
middleman不是官方插件，而是一个定制插件。使用前需要先安装。假定kong安装目录是`/usr/local/kong`。  
```
$ cd /usr/local/kong  && mkdir plugins && cd plugins
$ git clone https://github.com/pantsel/kong-middleman-plugin middleman
$ cd middleman
$ luarocks make *.rockspec
```
然后配置kong以便加载定制插件。编辑`/etc/kong/kong.conf`，添加下列配置（如果有多个定制插件需要配置，就逗号隔开）：
```
custom_plugins = middleman 
```
重启Kong。
#### 插件测试（首先是子请求状态码401）
[httpbin.org](http://httpbin.org)是一个免费的http调试云服务（首页是帮助）。测试用到下面的两个URI，一个返回200状态码，另一个返回401状态码：
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
    --data "config.url=http://httpbin.org/status/200?api=example-api2"
```
再次测试`example-api2`API:
```
$ curl http://localhost:8000/my-path
```
可以看到正常返回了`http://webdav.imaicloud.com`网页的内容。  

上述测试说明middleman插件可以用于Kong外部的认证、权限检查等功能。  

#### 子请求API的实现
Kong向指定URL(如前文的`http://httpbin.org/status/200`)发送请求的格式如下：
```
POST /status/200?api=example-api2 HTTP/1.1
Content-Length: 208
TE: trailers
Connection: close, TE
User-Agent: LuaSocket 3.0-rc1
Content-Type: application/json
Host: httpbin.org

 {"headers":{"accept":"*/*","host":"localhost:8000","user-agent":"curl/7.29.0","x-consumer-id":"9c270f20-f3e0-4af1-a3a1-91b58f11072c","x-consumer-username":"webb"},"uri_args":{"apikey":"1"},"body_data":null} 
```
(上述内容是用justniffer抓取的)  
middleman并没有将API请求的原始URL信息发送到子请求中（这应是一个缺陷），如果用middleman用于授权，则必须知道API的原始URL。上面用了一种变通的方法将原始API的URL以参数（`?api=example-api2`）的形式发送给子请求的实现。  

### OAuth2插件
OAuth2认证插件的篇幅较长，单独写了[这个文档](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong_OAuth2)。  

### CORS插件
[CORS插件官方文档](https://getkong.org/plugins/cors/)，[CORS的Nginx配置](https://github.com/wbwangk/wbwangk.github.io/wiki/CORS)  

将Kong的所有API启用全域CORS：
```
$ curl -X POST http://kong:8001/plugins \
    --data "name=cors" \
    --data "config.origins=*" \
    --data "config.methods=GET, POST, HEAD, PUT, DELETE, PATCH" \
    --data "config.headers=DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type" \
    --data "config.exposed_headers=X-Auth-Token" \
    --data "config.credentials=true" \
    --data "config.max_age=1728000"
```
如果放入了不支持的方法，提示错误信息：
```
{"config.methods":"\"OPTIONS\" is not allowed. Allowed values are: \"HEAD\", \"GET\", \"POST\", \"PUT\", \"PATCH\", \"DELETE\""}
```
### 请求转换插件
该插件可以修改HTTP请求的参数。  
Nginx配置出的Webdav服务不支持POST方法。利用Kong的请求转换插件，可以把POST方法转换成PUT。  
首先，定义创建一个API专门处理发往webdav的POST请求
```
$ curl -i -X POST --url http://localhost:8001/apis/ \
  --data 'name=webdav_post' \
  --data 'uris=/webdav' \
  --data 'methods=POST' \
  --data 'upstream_url=http://webdav/'
```
为这个API启用请求转换插件，将POST方法变成PUT。
```
$ curl -X POST http://kong:8001/apis/webdav_post/plugins \
    --data "name=request-transformer" \
    --data "config.http_method=PUT" 
```
测试一下这个API：
```
$ curl -X POST http://kong:8000/webdav/t.txt -d "this is t.txt"
```

### HTTP日志插件
这个插件可以将Kong日志以HTTP协议实时发送到某个URI。  
我用requestb.in模拟接收日志的服务器。[关于requestb.in的更多细节](https://github.com/wbwangk/wbwangk.github.io/wiki/HTTP%E6%A8%A1%E6%8B%9F#requestbin)  
```
$ curl -X POST http://kong:8001/apis/example-api2/plugins \
    --data "name=http-log" \
    --data "config.http_endpoint=https://requestb.in/r9x8mlr9" \
    --data "config.method=POST" \
    --data "config.timeout=1000" \
    --data "config.keepalive=1000"
$ curl http://kong:8000/my-path
```
`/my-path`是API`example-api2`的路径。  
用浏览器到地址`https://requestb.in/r9x8mlr9?inspect`去查看Kong的日志，可以看到的信息：
```json
{
  "consumer": {
    "created_at": 1506494530000,
    "username": "webb",
    "id": "9c270f20-f3e0-4af1-a3a1-91b58f11072c"
  },
  "api": {
    "created_at": 1510620873000,
    "strip_uri": true,
    "id": "7a2ab4e9-325b-4922-9ad1-7c53c5e0ec1e",
    "name": "example-api2",
    "http_if_terminated": false,
    "https_only": false,
    "upstream_url": "http://webdav.imaicloud.com/",
    "uris": [
      "/my-path"
    ],
    "preserve_host": false,
    "upstream_connect_timeout": 60000,
    "upstream_read_timeout": 60000,
    "upstream_send_timeout": 60000,
    "retries": 5
  },
  "request": {
    "querystring": {},
    "size": "90",
    "uri": "/my-path",
    "request_uri": "http://kong:8000/my-path",
    "method": "GET",
    "headers": {
      "host": "kong:8000",
      "x-consumer-username": "webb",
      "user-agent": "curl/7.29.0",
      "accept": "*/*",
      "x-consumer-id": "9c270f20-f3e0-4af1-a3a1-91b58f11072c",
      "apikey": "1"
    }
  },
  "client_ip": "192.168.73.102",
  "authenticated_entity": {
    "consumer_id": "9c270f20-f3e0-4af1-a3a1-91b58f11072c",
    "id": "c0ca1b8c-0d38-4b18-a77e-f69c35974c03"
  },
  "latencies": {
    "request": 45,
    "kong": 0,
    "proxy": 45
  },
  "response": {
    "headers": {
      "content-type": "text/html; charset=UTF-8",
      "date": "Tue, 14 Nov 2017 09:28:19 GMT",
      "connection": "close",
      "via": "kong/0.11.0",
      "access-control-allow-headers": "DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,X-Auth-Token",
      "x-kong-proxy-latency": "0",
      "server": "nginx/1.10.3",
      "transfer-encoding": "chunked",
      "x-kong-upstream-latency": "45",
      "access-control-allow-origin": "*",
      "access-control-allow-methods": "GET, POST, OPTIONS,DELETE, PUT, COPY, MOVE,MKCOL"
    },
    "status": 200,
    "size": "1991"
  },
  "tries": [
    {
      "balancer_latency": 0,
      "port": 80,
      "ip": "60.216.42.102"
    }
  ],
  "started_at": 1510651699463
}
```
需要说明的是，API`example-api2`添加了key-auth认证插件，所以日志中有`consumer`的信息。对于没有添加任何认证插件的API，则不会在日志中输出消费者信息。  
## Kong与Webdav
上文中`example-api2`这个API，将发向`/my-path`的请求代理到了`http://webdav.imaicloud.com/`这个地址。这个地址是用Nginx的[ngx_http_dav_module](http://nginx.org/en/docs/http/ngx_http_dav_module.html)模块实现的一个webdav协议测试虚拟主机。[这个文章](https://github.com/imaidev/imaidev.github.io/wiki/WebDav%E6%B5%8B%E8%AF%95)是讲以前做的webdav协议测试。  

下面测试一下Kong对Webdav协议的支持情况。 
为了减少干扰，删除middleman插件： 
```
$ curl localhost:8001/apis/example-api2/plugins | jq      (查询出middleman的插件id)
$ curl -XDELETE localhost:8001/apis/example-api2/plugins/b39fcfcf-f2ba-4d2c-a568-30f6313b4aff
```
测试一下通过Kong代理的删除文件webdav协议：
```
$ curl -XDELETE -H "apikey:1" localhost:8000/my-path/t1.txt
```
key-auth插件仍在，所以需要在标头发送apikey。  
测试一下反面上传：
```
$ curl -T './t1.txt' -H "apikey:1" localhost:8000/my-path/
```
用浏览器看一下，发现t1.txt已经传送到了`http://webdav.imaicloud.com/t1.txt`地址。  

## 定制Nginx配置文件
Kong的底层是Nginx，Kong利用Lua语言自动生成Nginx配置文件。自动生成的配置文件是：
```
/usr/local/kong/nginx.conf
/usr/local/kong/nginx-kong.conf
```
其中nginx.conf中include了nginx-kong.conf。所谓定制Nginx配置文件，就是想办法把自己的个性化配置放到nginx.conf中。  
在[Custom Nginx configuration](https://getkong.org/docs/0.11.x/configuration/#custom-nginx-configuration)中，Kong给出了定制Nginx配置文件的方案。  
方案就是定制一个模板文件(`custom_nginx.template`)，包含下列内容：
```
# ---------------------
# custom_nginx.template
# ---------------------

worker_processes ${{NGINX_WORKER_PROCESSES}}; # can be set by kong.conf
daemon ${{NGINX_DAEMON}};                     # can be set by kong.conf

pid pids/nginx.pid;                      # this setting is mandatory
error_log logs/error.log ${{LOG_LEVEL}}; # can be set by kong.conf

events {
    use epoll; # custom setting
    multi_accept on;
}

http {
    # include default Kong Nginx config
    include 'nginx-kong.conf';

    # custom server
    server {
        listen 8002;
        server_name kong;

        location / {
            root html;
        }
    }
}
```
为了与上面配置吻合，创建下面的目录和文件：
```
$ cd /usr/local/kong
$ mkdir html
$ echo "This is a index.html" > html/index.html
```
然后用下列方式重新启动Kong：
```
$ kong restart --nginx-conf custom_nginx.template
```
测试一下：
```
$  curl kong:8002/index.html
This is a index.html
```
这样kong这个主机在8000端口上进行API转发，在8002端口上进行静态网页服务。  

### Kong提供WebDav服务
将`custom_nginx.template`按下面的内容配置：
```
# ---------------------
# custom_nginx.template
# ---------------------

worker_processes ${{NGINX_WORKER_PROCESSES}}; # can be set by kong.conf
daemon ${{NGINX_DAEMON}};                     # can be set by kong.conf

pid pids/nginx.pid;                      # this setting is mandatory
error_log logs/error.log ${{LOG_LEVEL}}; # can be set by kong.conf

events {
    use epoll; # custom setting
    multi_accept on;
}

http {
    # include default Kong Nginx config
    include 'nginx-kong.conf';

    # custom server
    server {
        listen 8002;
        server_name kong;
        client_max_body_size  30m;
        root html;

        location / {
            #开启文件列表功能
            autoindex on;

            ## webdav config
            include cors-include.conf;
            client_body_temp_path /var/tmp;
            dav_methods PUT DELETE MKCOL COPY MOVE;
            create_full_put_path  on;
            dav_access    group:rw  all:r;
        }
    }
}
```
文件`cors-include.conf`的内容参考文章[CORS](https://github.com/wbwangk/wbwangk.github.io/wiki/CORS)。

**实测发现Kong没有加载webdav模块！**   
目前只能另外部署一个Nginx，然后用Kong代理Nginx的WebDav服务，就是象[Kong与Webdav](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#kong%E4%B8%8Ewebdav)一章中讲到的那样。  




[KONG API DOC](https://getkong.org/docs/0.11.x/admin-api/)  