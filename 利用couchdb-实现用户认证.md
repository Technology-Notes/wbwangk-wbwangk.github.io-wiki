token的检验依赖nginx，参考[使用NGINX的secure_link模块实现token验证](使用NGINX的secure_link模块实现token验证)。  
token的生成过程可分成两大步：用户登录和生成token。其中用户登录的检验依靠couchdb。另，使用couchdb管理角色，完成授权。
## 用户注册
couchdb内置了一个_users数据库存放所有用户，创建用户的服务如下，其中'org.couchdb.user'是couchdb规定的前缀。
```
curl -X PUT https://dev.imaicloud.com/couchdb/_users/org.couchdb.user:jan \
     -H "Accept: application/json" \
     -H "Content-Type: application/json" \
     -d '{"name": "jan", "password": "apple", "roles": [], "type": "user"}'
```
## 用户登录
couchdb支持多种认证方式，这里使用它的cookie认证。  
```
curl -vX POST https://dev.imaicloud.com/couchdb/_session \
       -H 'Content-Type:application/x-www-form-urlencoded' \
       -d 'name=anna&password=secret'
```
登录后couchdb会在用户cookie中写入会话标识（类似jsessionid）。登录响应相关内容：  
```
Set-Cookie: AuthSession=amFuOjU3ODY0NzU1Otyly0ka4T1Y5FGB0Q8yfZGfmvbq;
{"ok":true,"name":"root","roles":["_admin"]}
```
## 取用户信息
couchdb已经在用户浏览器中写入了会话标识，就可以发请求取用户信息了：
```
curl -vX GET https://dev.imaicloud.com/couchdb/_session \
      --cookie AuthSession=amFuOjU3ODY0NzU1Otyly0ka4T1Y5FGB0Q8yfZGfmvbq
```
响应：
```
{ok: true,userCtx: {name: "anna",roles: ["_admin"]}, \
info: {authentication_db: "_users",authentication_handlers: [\
"oauth","cookie","default"],authenticated: "cookie"}}
```
上述响应中的userCtx（User Context Object）就是用户上下文信息。

## 生成token
将userCtx发给后台的token生成程序来生成token。算法见：[使用NGINX的secure_link模块实现token验证](使用NGINX的secure_link模块实现token验证)
将token写入cookie中：
```imaicloud_expires=2147483647; imaicloud_md5=TX7ZNGJJNqtHviTmMr-DDQ; imaicloud_payload=payload; imaicloud_role=_admin,compdev,appdev
```

## NGINX检验token
见[使用NGINX的secure_link模块实现token验证](使用NGINX的secure_link模块实现token验证)