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
### Webdav API
利用Kong代理Webdav，对于PUT/DELETE方法需要认证；对于GET方法不认证(js可以直接访问)。需要创建两个API：
- webdav，PUT/DELETE方法  
- webdav_get，GET方法
```
$ curl -i -X POST --url http://kong:8001/apis/ \
  --data 'name=webdav' \
  --data 'uris=/webdav' \
  --data 'methods=PUT,DELETE' \
  --data 'upstream_url=http://webdav/'
$ curl -i -X POST --url http://kong:8001/apis/ \
  --data 'name=webdav_get' \
  --data 'uris=/webdav_get' \
  --data 'methods=GET' \
  --data 'upstream_url=http://webdav/'
```
测试一下：
```
$ curl -X PUT http://kong:8000/webdav/test.txt -d "this is a test.txt"
$ curl -X GET --url http://kong:8000/webdav/test.txt
```
### key-auth插件
为`webdav`API添加key-auth插件：
```
$ curl -i -X POST \
  --url http://kong:8001/apis/webdav/plugins/ \
  --data 'name=key-auth'
```
创建文件上传专用消费者`webdav`，然后创建API-KEY：
```
$ curl -i -X POST --url http://kong:8001/consumers/ \
  --data "username=webdav"
$ curl -i -X POST --url http://kong:8001/consumers/webdav/key-auth/
```
响应的json串中有自动生成的API-KEY:
```
"key":"XXXXX"
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
### key-auth认证
在java程序中访问文件服务器，需要带上API-KEY，逻辑示例：
```
curl -i -X GET --url http://kong:8000/webdav \
  --header "apikey: XXXX"
```

### DSP授权API
