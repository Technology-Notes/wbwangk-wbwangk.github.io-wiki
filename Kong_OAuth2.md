在GXB项目上使用Nginx的webdav模块配置出了“文件上传”的功能。并使用Kong的key-auth插件为所有API（包括文件上传）附加了apikey认证。如果总是从后台应用访问webdav文件服务器，apikey认证是满足需求的。但如果想支持从网页上用js直接访问webdav服务器，则面临着apikey泄密的问题。这时候，OAuth2认证才是个更好的选择。  
Kong的官方OAuth2插件的文档在[这里](https://getkong.org/plugins/oauth2-authentication/)。  
### Kong启用SSL
OAuth2的底层依赖SSL通信，要使用Kong的OAuth2插件，必须为Kong的代理端口启用SSL。Kong的SSL代理端口默认是8443。    
在Kong的默认配置文件`/etc/kong/kong.conf`中，SSL配置默认是关闭的，修改成下列的样子:
```
proxy_listen_ssl = 0.0.0.0:8443
ssl = on
```
然后用`kong restart`命令重启Kong。  
为了确认Kong对8443端口已经成功监听，打开Kong自动生成的Nginx配置文件`/usr/local/kong/nginx-kong.conf`，可以看到下列内容：
```
listen 0.0.0.0:8443 ssl;
```
Kong的底层是Nginx，使用Kong的管理接口(默认8001)对Kong所做的所有管理配置，大多会反映到上面的`nginx-kong.conf`配置文件中。  
测试一下8443端口：
```
$ curl -k https://localhost:8443
{"message":"no API found with those values"}
```
上述返回值说明SSL已经正常工作。由于Kong使用的SSL公钥证书不是权威CA发放的，不存在于curl的可信证书库中，所以要用-k参数让curl忽略可信证书检测。关于SSL的更多知识，参考《[SSL研究](https://github.com/wbwangk/wbwangk.github.io/wiki/SSL研究)》。  

利用Kong可以为多个虚拟主机反向代理API，但利用上面的办法，所有的虚拟主机共享同一个公钥证书。如果需要为每个虚拟主机配置单独的公钥证书，需要用到Kong的[certificate-object](https://getkong.org/docs/0.11.x/admin-api/#certificate-object)和[sni-objects](https://getkong.org/docs/0.11.x/admin-api/#sni-objects)功能。  

certificate-object命令示范：
```
$ curl -X POST http://localhost:8001/certificates \
    -F "cert=@/opt/https/nginx.crt" \
    -F "key=@/opt/https/nginx.key"
```
### 授权方式：客户端凭据
在Kong的官方[OAuth2插件](https://getkong.org/plugins/oauth2-authentication/)文档中，对于客户端凭据([Client Credentials](https://tools.ietf.org/html/rfc6749#section-4.4))授权方式描述很少，而是主要讲的更普遍的授权码([Authorization Code Grant](https://tools.ietf.org/html/rfc6749#section-4.1))方式。  

在GXB项目中，DSP-OD应用作为OAuth2客户端是完全可信的，可以使用更简单的客户端凭据方式来完成OAuth授权，所以本章主要讲如何使用Kong的OAuth2插件实现客户端凭据方式授权。  

#### 1. 为某API启用oauth2插件
首先，创建一个叫`cats2`的API，它会将URI前缀为`/cats2`的请求代理到`http://mockbin.org`：
```
$ curl -X POST http://127.0.0.1:8001/apis/  \
      -d "name=cats2" \
      -d "uris=/cats2" \
      -d "upstream_url=http://mockbin.org/" 
```
然后，为API`cats2`启用oauth2插件：
```
$ curl -X POST http://127.0.0.1:8001/apis/cats2/plugins/  \
     -d "name=oauth2" \
     -d "config.enable_client_credentials=true" 
   
{"created_at":1510133627000,"config":{"enable_client_credentials":true,"mandatory_scope":false,"hide_credentials":false,"enable_implicit_grant":false,"global_credentials":false,"accept_http_if_already_terminated":false,"enable_password_grant":false,"provision_key":"LNeLjurIrzYDJlczl6psClGCC0lbPYMs","enable_authorization_code":false,"anonymous":"","token_expiration":7200},"id":"c8606a93-6570-4d06-83ff-17b139db16eb","name":"oauth2","api_id":"18413f29-a432-4020-8f5e-0d4efc52dc67","enabled":true}
```
oauth2插件默认是禁止客户端证书授权方式的，所以启用插件时要增加参数`config.enable_client_credentials=true`。在返回值中有个`provision_key`后面会用到。在我的测试中，`provision_key`的值：
```
"provision_key":"LNeLjurIrzYDJlczl6psClGCC0lbPYMs"
```
令牌的有效期默认是7200(两个小时)，可以在启用oauth2插件时设置`config.token_expiration`参数来调整这个值得大小。  

#### 2. 创建oauth应用
首先，创建API消费者`thefosk2`，一般是某个应用开发者：
```
$ curl -X POST http://127.0.0.1:8001/consumers/  \
     -d "username=thefosk2" 
```
为开发者`thefosk2`创建一个需要OAuth认证的应用：
```
curl -X POST http://127.0.0.1:8001/consumers/thefosk2/oauth2/ \
     -d "name=Hello World App2" \
     -d "redirect_uri=http://getkong.org/" 

{"client_id":"eRFRLEtx5uW1qvYfX9a4aPTWC8EIoD5d","created_at":1510133726000,"id":"5e49377c-8a62-4163-8094-30e1286c5ae3","redirect_uri":["http:\/\/getkong.org\/"],"name":"Hello World App2","client_secret":"ik69jv2dk4sj5V8FAHmY0S0ddegUEBl9","consumer_id":"c29bee8a-2139-4432-8dd9-563f7635f980"}
```
可以看到返回值中有`client_id`和`client_secret`，这就是传说中的“客户端凭据”。在客户端凭据授权方式下，访问令牌(access token)是用客户端凭据换取的。   
除了让插件自动帮你生成`client_id`和`client_secret`，你也可以在创建应用时主动附加上`client_id`和`client_secret`参数，这时插件就采用参数中的`client_id`和`client_secret`，而不会再自动生成了。  

#### 3. 使用客户端凭据获取访问令牌
```
$ curl --silent --insecure -X POST --url https://localhost:8443/cats2/oauth2/token \
--data "client_secret=ik69jv2dk4sj5V8FAHmY0S0ddegUEBl9" \
--data "client_id=eRFRLEtx5uW1qvYfX9a4aPTWC8EIoD5d" \
--data "grant_type=client_credentials"
{"token_type":"bearer","access_token":"Xjh29oE8UBCx0KouGDEW1pVt4a27Q1XH","expires_in":7200}
```
#### 4. 授权令牌的使用
DSP_OD应用需要把`client_id`和`client_secret`的值写入程序中。当需要上传文件时，利用客户端凭据向Kong换取访问令牌，然后用令牌访问Kong代理的文件上传API，类似于：
```
$ curl -k https://localhost:8443/cats2?access_token=Xjh29oE8UBCx0KouGDEW1pVt4a27Q1XH
$ curl -k -H "Authorization: token Xjh29oE8UBCx0KouGDEW1pVt4a27Q1XH" https://localhost:8443/cats2
```
访问令牌可以用URL参数的方式或HTTP标头的方式传递，就是上面示范的那样。  

### 授权方式：授权码方式
授权码方式是互联网上常见的OAuth授权场景使用的方式。典型特点是会有一个弹出窗口，供最终用户授权，当用户授权后，申请的应用就会获得一个授权码，然后用授权码换取访问令牌。授权窗口上会显示权限清单，这在OAuth中叫`scope`。  
确保你的Kong已经参照上一个章节启用了SSL。  

#### 安装示范应用kong-oauth2-hello-world
github上有个利用Kong的OAuth2插件做的[Hello World示范](https://github.com/Kong/kong-oauth2-hello-world)，以这个为入门很好。可以再结合插件的[官方文档](https://getkong.org/plugins/oauth2-authentication/)一起看。  
这个Hello World是个用Node.js应用，有前端有后端。安装很简单：
```
$ git clone https://github.com/Kong/kong-oauth2-hello-world.git kong-oauth2
$ cd kong-oauth2
$ npm install
```
npm会根据kong-oauth2目录下的package.json安装依赖包。安装完毕后执行`node app.js`就可运行。只是需要先配置Kong。  

#### 1. 为某API启用oauth2插件 
首先，创建一个叫`cats`的API，它会将URI前缀为`/cats`的请求代理到`http://mockbin.org`：
```
$ curl -X POST http://127.0.0.1:8001/apis/  \
      -d "name=cats" \
      -d "uris=/cats" \
      -d "upstream_url=http://mockbin.org/" 
```
然后，为API`cats`启用oauth2插件：
```
$ curl -X POST http://127.0.0.1:8001/apis/cats/plugins/  \
     -d "name=oauth2" \
     -d "config.scopes=email, phone, address" \
     -d "config.mandatory_scope=true" \
     -d "config.enable_authorization_code=true" 
```
`config.mandatory_scope=true`的意思是在申请授权的时候至少要选择一个scope。在返回值中有个`provision_key`后面会用到。在我的测试中，`provision_key`的值：
```
"provision_key":"G5uoPlnq2D0N4gBDnopRiZPAi6XbGVEE"
```
#### 2. 创建oauth应用
首先，创建API消费者`thefosk`，一般是某个应用开发者：
```
$ curl -X POST http://127.0.0.1:8001/consumers/  \
     -d "username=thefosk" 
```
为开发者`thefosk`创建一个需要OAuth认证的应用：
```
$ curl -X POST http://127.0.0.1:8001/consumers/thefosk/oauth2/ \
     -d "name=Hello World App" \
     -d "redirect_uri=http://getkong.org/" 

{"client_id":"HaTqElw2bOsLMtxPGJI0uHGYspyIA0OB","created_at":1510127846000,"id":"7e30d382-e337-4481-8d87-ab3e899824bf","redirect_uri":["http:\/\/getkong.org\/"],"name":"Hello World App","client_secret":"E8ue3mb9jyTDrvb6sn7Of5vPB2cLu1Hz","consumer_id":"5ff62661-0de7-4d69-8a50-e52834a8d34c"}
```
可以看到返回值中有`client_id`和`client_secret`，这个下面会用到。  

#### 3. 运行Hello world Web应用
首先设置环境变量，将Hello World应用需要的Kong参数告诉它。
```
export PROVISION_KEY="G5uoPlnq2D0N4gBDnopRiZPAi6XbGVEE"
export KONG_ADMIN="http://127.0.0.1:8001"
export KONG_API="https://127.0.0.1:8443"
export API_PATH="/cats"
export SCOPES="{ \
  \"email\": \"Grant permissions to read your email address\", \
  \"address\": \"Grant permissions to read your address information\", \
  \"phone\": \"Grant permissions to read your mobile phone number\" \
}"
export LISTEN_PORT=3301
```
如果不设置`LISTEN_PORT`，则HelloWorld应用默认使用3000端口。  
现在可以启动HelloWorld应用了：
```
$ node app.js
PROVISION_KEY is G5uoPlnq2D0N4gBDnopRiZPAi6XbGVEE
KONG_ADMIN is http://127.0.0.1:8001
KONG_API is https://127.0.0.1:8443
API_PATH is /cats
SCOPES is {   "email": "Grant permissions to read your email address",   "address": "Grant permissions to read your address information",   "phone": "Grant permissions to read your mobile phone number" }
Running at Port 3000
```
#### 4.测试授权码方式OAuth2
我应用是跑在虚拟机里的，而浏览器是宿主机windows下的，虚拟机的IP是`192.168.73.102`。用浏览器访问HelloWorld应用应该访问`192.168.73.102`这个地址，而不是`127.0.0.1`。  
在浏览器中输入：
```
http://192.168.73.102:3000/authorize?response_type=code&scope=email%20address&client_id=HaTqElw2bOsLMtxPGJI0uHGYspyIA0OB
```
浏览器弹出了授权页面，页面上显示的scope为email和address。在浏览器中点击“Auhorize”（授权）按钮后，页面重定向到了地址：
```
https://getkong.org/?code=qaZnTA8wlqutux7064JsT0oxBoi1FXVo
```
上面的code就是授权码。如果实际使用时，`redirect_uri`会被设置成Web应用的某个URI，这个URI会接收到授权码，然后用下面的逻辑换取访问令牌：  
```
$ curl https://127.0.0.1:8443/cats/oauth2/token \
     -d "grant_type=authorization_code" \
     -d "client_id=HaTqElw2bOsLMtxPGJI0uHGYspyIA0OB " \
     -d "client_secret=E8ue3mb9jyTDrvb6sn7Of5vPB2cLu1Hz" \
     -d "redirect_uri=http://getkong.org/" \
     -d "code=qaZnTA8wlqutux7064JsT0oxBoi1FXVo" --insecure
```
获得访问令牌后，访问具体API就与上一章的客户端凭据方式一样了。  

