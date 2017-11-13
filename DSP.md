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