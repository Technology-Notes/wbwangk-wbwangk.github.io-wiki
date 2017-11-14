有几个云服务可用于HTTP调试。  
### httpbin.org
如果需要一些简单的http响应模拟，httpbin.org无疑是个很好的选择。  
httpbin.org首页就是帮助，下面通过几个例子体验一下httpbin.org的用法。  

1. 模拟POST   
```json
$ curl -X POST httpbin.org/post -d "dddd"
{
  "args": {},
  "data": "",
  "files": {},
  "form": {
    "dddd": ""
  },
  "headers": {
    "Accept": "*/*",
    "Connection": "close",
    "Content-Length": "4",
    "Content-Type": "application/x-www-form-urlencoded",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.29.0"
  },
  "json": null,
  "origin": "112.224.67.169",
  "url": "http://httpbin.org/post"
}
```
2. 返回请求的所有信息  
```json
$ curl -X DELETE httpbin.org/anything
{
  "args": {},
  "data": "",
  "files": {},
  "form": {},
  "headers": {
    "Accept": "*/*",
    "Connection": "close",
    "Content-Length": "0",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.29.0"
  },
  "json": null,
  "method": "DELETE",
  "origin": "112.224.67.169",
  "url": "http://httpbin.org/anything"
}
```
3. 返回一个401状态码  
```
$ curl -i httpbin.org/status/401
HTTP/1.1 401 UNAUTHORIZED
```

### requestb.in
如果希望服务器把请求都缓存起来，以便统一查看请求和响应的信息，那么requestb.in更好用。  
到requestb.in的首页上点击` Create a RequestBin`按钮就跳转到一个新页面，并提示了新建Bin的URL，如：`https://requestb.in/w9uxqaw9`。旁边是个红色的实心圆圈。。  
发送下列请求来测试requestb.in的功能：
```
$ curl -X DELETE https://requestb.in/w9uxqaw9
$ curl -X POST https://requestb.in/w9uxqaw9
```
然后点击红色的实心圆圈，网页进入了列表页面，页面上显示了上面两个请求的信息。  

### mockbin.org
不大稳定，可能是网络的原因  