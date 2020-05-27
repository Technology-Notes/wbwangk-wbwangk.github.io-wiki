## 部署
#### 安装nginx
安装Nginx，并配置Webdav，[参考这里](https://github.com/wbwangk/wbwangk.github.io/wiki/nginx)。 
配置nginx的配置文件(`/etc/nginx/conf.d/default.conf`)：  
```nginx
server {
    listen       8002;
    server_name  webdav;
    location / {
        root   /usr/share/nginx/html;
            autoindex on;
            ## webdav config
            client_body_temp_path /tmp;
            dav_methods PUT DELETE MKCOL COPY MOVE;
            create_full_put_path  on;
            dav_access    group:rw  all:r;
    }
...
```
设置webdav根目录权限和启动nginx：
```
$ chmod 777 /usr/share/nginx/html
$ /usr/sbin/nginx
```
机器上可能装了多个nginx，如Kong自带的，启动Nginx时最好带上全路径，以防止误启动了其它路径的Nginx。下面的方式可以查找nginx的所在目录，已经现有的Nginx进程：
```
$ whereis nginx 
nginx: /usr/sbin/nginx /usr/lib64/nginx /etc/nginx /usr/share/nginx /usr/local/openresty/nginx/sbin/nginx /usr/share/man/man8/nginx.8.gz
$ ps -ef | grep nginx
```
`/usr/local/openresty`目录下的nginx是Kong自带的。其它目录的是手工安装的nginx所在的目录。  

#### 安装Kong
安装Kong，[这里](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#kong%E5%AE%89%E8%A3%85)。 
  Nginx和Kong都安装在10.10.250.249上。编辑249的`/etc/hosts`文件，添加：
```
10.10.250.249 kong webdav
```
这样做是为了约定文件服务器域名是webdav，网关服务器的域名是kong。  

### Webdav API
利用Kong代理Webdav文件上传服务，创建`webdav`API:
```
$ curl -i -X POST --url http://kong:8001/apis/ \
  --data 'name=webdav' \
  --data 'uris=/webdav' \
  --data 'upstream_url=http://webdav:8002/'
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
$ curl -i -X POST --url http://kong:8001/consumers/webdav/key-auth/ \
    --data 'key=XXXX'
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
$ git clone --depth 1 https://github.com/pantsel/kong-middleman-plugin middleman
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
$ curl -i -X GET --url http://kong:8000/webdav \
  --header "apikey: XXXX"
```

### 服务注册
数据资源分三种类型，数据库、文件和接口(服务)。接口类型的数据资源又区分为有条件共享和无条件共享。  
当接口类型的数据资源保存时，需要向服务网关(Kong)注册API。注册逻辑类似于：
```
$ $ curl -i -X POST --url http://kong:8001/apis/ \
  --data 'name=<API名称>' \
  --data 'uris=<API的URI前缀>' \
  --data 'upstream_url=<上游URL>'
```
可以约定<API名称>与<API的URI前缀>相同，通称为<URI前缀>。

### 有条件共享服务注册
对于“有条件共享”的数据资源，除了普通的注册逻辑外，还要额外注册middleman插件。注册逻辑：
```
$ curl -X POST http://kong:8001/apis/<URI前缀>/plugins \
    --data "name=middleman" \
    --data "config.url=http://dsp:8080/<middleman钩子URI>?api=<URI前缀>"
```
### consumer注册与apikey申请
[更多细节](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong_DSP#consumer%E6%B3%A8%E5%86%8C%E4%B8%8Eapikey%E7%94%B3%E8%AF%B7)  
开发者要调用共享接口(服务)，必须要进行consumer注册，并申请API-KEY。  
consumer注册逻辑：
```
$ curl -X POST --url http://kong:8001/consumers/ \
  --data "username=<DSP中的用户ID>"
```
API-KEY的生成逻辑：
```
$ curl -X POST --url http://kong:8001/consumers/<DSP中的用户ID>/key-auth/ \
   --data 'key=XXXX'
```
### middleman钩子
需要DSP实现一个middleman钩子服务，对于添加了middleman插件的API，Kong会向上游服务器代理HTTP流之前调用钩子服务。调用的参数：  
- <DSP中的用户ID>  
- URL参数`api=<URI前缀>`  
根据这两个参数需要计算出当前的有条件共享API是否允许当前用户调用。[更多细节](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong_DSP#%E6%8E%88%E6%9D%83)  

## 测试
使用自动REST测试工具[pyresttest](https://github.com/imaidev/imaidev.github.io/wiki/REST%E8%87%AA%E5%8A%A8%E6%B5%8B%E8%AF%95(pyresttest))对DSP+Kong进行自动化测试。  
dsp_kong.yaml:
```
---
- config:
    - testset: "测试DSP与Kong的集成环境"
    - variable_binds: {apikey: '1'}

- test:
  - group: "webdav"
  - name: "GET /webdav/"
  - url: "/webdav/"
  - headers: {'apikey': "$apikey"}

- test: 
  - group: "webdav"
  - name: "PUT /webdav/test.txt"
  - url: "/webdav/test.txt"
  - method: "PUT"
  - body: "this is content of file test.txt"
  - headers: {'apikey': "$apikey"}

- test: 
  - group: "webdav"
  - name: "DELETE /webdav/test.txt"
  - url: "/webdav/test.txt"
  - method: "DELETE"
  - headers: {'apikey': "$apikey"}

- test: 
  - name: "GET /admin-api/，上游是kong:8001"
  - url: "/admin-api/"

```