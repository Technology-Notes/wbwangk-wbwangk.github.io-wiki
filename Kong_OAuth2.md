上面使用key-auth插件对webdav协议附加了apikey认证。如果总是从后台应用访问webdav服务器，apikey认证是满足需求的。但如果想支持从网页上用js直接访问webdav服务器，则面临着apikey的泄密问题。这时候，OAUTH认证才是个更好的选择。  

```
curl -X POST http://kong:8001/apis/example-api2/plugins \
    --data "name=oauth2" \
    --data "config.enable_authorization_code=true" \
    --data "config.scopes=db,file,api" \
    --data "config.mandatory_scope=true"
{
  "created_at": 1510109689000,
  "config": {
    "token_expiration": 7200,
    "mandatory_scope": true,
    "hide_credentials": false,
    "enable_authorization_code": true,
    "enable_implicit_grant": false,
    "global_credentials": false,
    "scopes": [
      "db",
      "file",
      "api"
    ],
    "enable_password_grant": false,
    "accept_http_if_already_terminated": false,
    "anonymous": "",
    "enable_client_credentials": false,
    "provision_key": "dNYLur1pJPKAvEKhAyQsBeyrO2YdYm96"
  },
  "id": "d26ab1d0-07a1-42e8-a972-89770ef4d0aa",
  "name": "oauth2",
  "api_id": "828981ba-9d83-486d-842b-696b0eb5417d",
  "enabled": true
}
```
`example-api2`API启用了oauth2插件后，再访问这个API就报错了：
```
$ curl localhost:8000/my-path
{"error_description":"The access token is missing","error":"invalid_request"}
```
#### Resource Owner Password Credentials
```
$ curl http://kong:8000/oauth2/token \
     --data "client_id=client1" \
    --data "client_secret=1" \
    --data "grant_type=password" \
    --data "scope=file" \
    --data "provision_key=dNYLur1pJPKAvEKhAyQsBeyrO2YdYm96" \
    --data "authenticated_userid=webb" \
    --data "username=webb" \
    --data "password=1"
```

#### Kong启用SSL

```
$ cd /opt/https
$ openssl req -new -newkey rsa:2048 -nodes -x509 -keyout nginx.key -out nginx.crt -subj "/C=CN/ST=Shan Dong/L=Ji Nan/O=Inspur/OU=SBG/CN=c7302.ambari.apache.org"
$ curl -i -X POST http://localhost:8001/certificates \
    -F "cert=@/opt/https/nginx.crt" \
    -F "key=@/opt/https/nginx.key"
$ curl -i -X POST --url http://localhost:8001/apis/ \
  --data 'name=ssl-api' \
  --data 'uris=/ssl-api' \
  --data 'upstream_url=http://webdav.imaicloud.com/'
$ curl https://localhost:8443/ssl-api
curl: (7) Failed connect to localhost:8443; Connection refused
```
刚安装后Kong默认没有启用8443端口，所以上面报错。修改Kong默认配置文件`/etc/kong/kong.conf`:
```
proxy_listen_ssl = 0.0.0.0:8443
ssl = on
```
重启Kong：
```
$ kong restart
```
为了确认8443端口已经成功监听，可以打开Kong自动生成的Nginx配置文件`/usr/local/kong/nginx-kong.conf`，可以看到下列内容：
```
listen 0.0.0.0:8443 ssl;
```
Kong的底层是Nginx，使用Kong的管理接口(默认8001)对Kong所做的所有配置，大多会反映到上面的`nginx-kong.conf`配置文件中。  
测试一下8443端口：
```
$ curl -k https://localhost:8443/ssl-api
```


#### 创建应用
```
curl -i -X POST http://kong:8001/consumers/webb/oauth2 \
    --data "name=gxb-webdav" \
    --data "client_id=client1" \
    --data "client_secret=1" \
    --data "redirect_uri=http://webdav.imaicloud.com/"
{"client_id":"client1","created_at":1510110463000,"id":"e45921d3-76a9-4da8-ab50-a3557b4b6685","redirect_uri":["http:\/\/webdav.imaicloud.com\/"],"name":"gxb-webdav","client_secret":"1","consumer_id":"9c270f20-f3e0-4af1-a3a1-91b58f11072c"}
```
#### 获取访问令牌
```
$ curl -X POST http://kong:8001/oauth2_tokens \
    --data "credential_id=e45921d3-76a9-4da8-ab50-a3557b4b6685" \
    --data "token_type=bearer" \
    --data "access_token=access_token1" \
    --data "refresh_token=refresh_token1" \
    --data "expires_in=3600"
{"refresh_token":"refresh_token1","token_type":"bearer","access_token":"access_token1","created_at":1510110884000,"expires_in":3600,"credential_id":"e45921d3-76a9-4da8-ab50-a3557b4b6685","id":"9802fe24-bb3a-4a01-8de3-a09d2fad1715"}
```
### OAuth 2.0 Hello World for Kong
这是一个利用Kong实现OAuth2的Hello World示范。界面以Node.js实现。  
安装：
```
$ git clone https://github.com/Kong/kong-oauth2-hello-world.git kong-oauth2
$ cd kong-oauth2
$ npm install
```
设置环境变量：
```
export PROVISION_KEY="dNYLur1pJPKAvEKhAyQsBeyrO2YdYm96"
export KONG_ADMIN="http://c7302:8001"
export KONG_API="https://c7302:8443"
export API_PATH="/ssl-api"
export SCOPES="{ \
  \"db\": \"Grant permissions to db\", \
  \"file\": \"Grant permissions to file\", \
  \"api\": \"Grant permissions to api\" \
}"
```