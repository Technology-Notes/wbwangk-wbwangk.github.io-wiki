## 部署
1. 安装Nginx，并配置Webdav，[这里](https://github.com/wbwangk/wbwangk.github.io/wiki/nginx)。 
   nginx配置webdav时，注意server_name增加webdav域名(这是一个约定)：  
```nginx
server {
    listen       80;
    server_name  webdav;

```
2. 安装Kong，[这里](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#kong%E5%AE%89%E8%A3%85)。 
  Nginx和Kong都安装在10.10.250.249上。编辑249的`/etc/hosts`文件，添加：
```
10.10.250.249 kong webdav
```
这样做是为了约定文件服务器域名是webdav，网关服务器的域名是kong。  

3. 利用Kong代理Webdav：
```
$ curl -i -X POST --url http://localhost:8001/apis/ \
  --data 'name=webdav' \
  --data 'uris=/webdav' \
  --data 'upstream_url=http://webdav/'
```
测试一下：
```
$ curl -X PUT http://kong:8000/webdav/test.txt -d "this is a test.txt"
$ curl -X GET --url http://kong:8000/webdav/test.txt
```

### CORS
```
curl -X POST http://kong:8001/plugins \
    --data "name=cors" \
    --data "config.origins=*" \
    --data "config.methods=GET, POST, HEAD, PUT, DELETE, PATCH" \
    --data "config.headers=DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type" \
    --data "config.exposed_headers=X-Auth-Token" \
    --data "config.credentials=true" \
    --data "config.max_age=1728000"
```

### 转换插件
创建一个API专门处理发往webdav的POST请求
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

### OAUTH2之客户端凭据

#### Kong启用SSL
为Kong的代理端口8000启用SSL。  
修改Kong配置文件`/etc/kong/kong.conf`，将下面两行配置的注释`#`去掉：
```conf
proxy_listen_ssl = 0.0.0.0:8443
ssl = on
```
重启Kong，然后用curl测试启用SSL的Kong（端口8443）：
```
$ kong restart
$  curl -k https://kong:8443
{"message":"no API found with those values"}
```
上面表示Kong的代理端口已经启用SSL。不启用SSL就无法使用OAuth2插件。  

#### 启用OAUTH2插件
需要为`webdav和`webdav_post`两个API启用OAUTH2认证(认证方式为客户端凭据)。
```  
$ curl -X POST http://kong:8001/apis/webdav/plugins/  \
     -d "name=oauth2" \
     -d "config.enable_client_credentials=true" 
$ curl -X POST http://kong:8001/apis/webdav_post/plugins/  \
     -d "name=oauth2" \
     -d "config.enable_client_credentials=true" 
```
#### 创建消费者和应用
约定文件上传消费者为webdav，约定文件服务应用名为webdav。  
```
$ curl -X POST http://kong:8001/consumers/  \
     -d "username=webdav" 
$ curl -X POST http://kong:8001/consumers/webdav/oauth2/ \
     -d "name=webdav" \
     -d "redirect_uri=http://webdav" 
("client_id":"FRsC7XCAWW0PQcKczr2ZqjjQCSeGXeq7","client_secret":"R5O3IZLnPs219X1vvYsgWerpp5Gs7ekA")
```
### middleman插件
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

## 编程
