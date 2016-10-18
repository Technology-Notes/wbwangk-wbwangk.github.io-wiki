### 平台级认证
sandstorm实现了平台级别的认证，所以你只需要登录平台一次，而不用分别登录各个应用。当你打开一个应用，平台会通知应用你已经通过认证的id。  
我们计划实现一个outging OAuth代理，这样应用就不用处理OAuth请求，平台会统一处理这种OAuth请求。平台会能够安全地存储OAuth令牌。

### 免费动态域名xip.io
如域名127.0.0.1.xip.io（或xxx.127.0.0.1.xip.io）会自动解析到ip 127.0.0.1

### sandstorm的动态域名和https证书
[github.com/sandstorm-io/sandcats](https://github.com/sandstorm-io/sandcats)

### API token with

#####两种办法可以提交API token：OAuth2.0或HTTP基础认证

Recommended: OAuth 2.0-style Bearer header. You can pass an Authorization: Bearer foo header with the HTTP request, replacing foo with the API token. For example:   
```curl -H "Authorization: Bearer 49Np9sqkYV4g_FpOQk1p0j1yJlvoHrZm9SVhQt7H2-9" https://alpha-api.sandstorm.io/```   
HTTP Basic auth. You can use any username so long as you provide the API token as the password. For example, ```https://anything:DT5hkM18CejvQomjIM1AVT4zqQdOdoFCid898bP2hQS@api-d9bc3de0bed9cb9b321d3c491c10dbca.alpha.sandstorm.io/.```

##### websocket与token

websocket无法设置http头。sandstorm的做法：  
```/.sandstorm-token/<token>```
例如：  
```wss://api-qxJ58hKANkbmJLQdSDk4.oasis.sandstorm.io/.sandstorm-token/RfNqni4FEHXkWC5B8v6t/some/path```
