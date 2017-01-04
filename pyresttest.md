(使用虚机block2（ubuntu/xenial），启动命令vagrant up share2)
[pyresttest](https://github.com/svanoort/pyresttes)是一个用python实现的API自动化测试工具。使用YMAL/JSON格式的配置文件驱动，不用写代码。
### 安装pyresttest
```
$ apt-get install python-pycurl
$ pip install pyresttest
```
### json-server与httpbin.org
要测试pyresttest需要一个REST模拟服务器。可以自己部署[json-server](https://github.com/typicode/json-server)或直接使用[httpbin.org](http://httpbin.org/)。本测试使用的httpbin.org。

### 测试代码
参考了pyresttest项目的README文档中的样例代码，改写成了使用httpbin.org。
```
---
- config:
    - testset: "Basic tests"
    - timeout: 100  # Increase timeout from the default 10 seconds
- test: 
    - name: "返回ip"
    - url: "http://httpbin.org/ip"
- test: 
    - name: "Get测试"
    - url: "http://httpbin.org/get"
- test: 
    - name: "Delete测试"
    - url: "http://httpbin.org/delete"
    - method: 'DELETE'
- test: # create entity by PUT
    - name: "Create/update 测试"
    - url: "http://httpbin.org/put"
    - method: "PUT"
    - body: '{"first_name": "Gaius","id": 1,"last_name": "Baltar","login": "gbaltar"}'
    - headers: {'Content-Type': 'application/json'}
    - validators:  # This is how we do more complex testing!
        - compare: {header: content-type, comparator: contains, expected:'json'}
        - compare: {jsonpath_mini: 'login', expected: 'gbaltar'}  # JSON extraction
        - compare: {raw_body:"", comparator:contains, expected: 'Baltar' }  # Tests on raw response
- test: # create entity by POST
    - name: "Create person"
    - url: "/api/person/"
    - method: "POST"
    - body: '{"first_name": "William","last_name": "Adama","login": "theadmiral"}'
    - headers: {Content-Type: application/json}
```
上述例子中put测试较复杂，有比较(compare)逻辑。所以最好提前测试一下httpbin.org的功能：
```
$ curl -i -X PUT http://httpbin.org/put -d '{"first_name": "Gaius","id": 1,"last_name": "Baltar","login": "gbaltar"}' 
HTTP/1.1 200 OK
Server: nginx
Date: Wed, 04 Jan 2017 01:45:07 GMT
Content-Type: application/json
Content-Length: 437
Connection: keep-alive
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true

{
  "args": {},
  "data": "",
  "files": {},
  "form": {
    "{\"first_name\": \"Gaius\",\"id\": 1,\"last_name\": \"Baltar\",\"login\": \"gbaltar\"}": ""
  },
  "headers": {
    "Accept": "*/*",
    "Content-Length": "72",
    "Content-Type": "application/x-www-form-urlencoded",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.47.0"
  },
  "json": null,
  "origin": "60.208.111.201",
  "url": "http://httpbin.org/put"
}
```
