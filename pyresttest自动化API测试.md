(使用虚机block2（ubuntu/xenial），启动命令vagrant up share2，ip：192.168.1.142或127.0.0.1)
[pyresttest](https://github.com/svanoort/pyresttes)是一个用python实现的API自动化测试工具。使用YMAL/JSON格式的配置文件驱动，不用写代码。
### 安装pyresttest
```
$ apt-get install python-pycurl
$ pip install pyresttest
```
### 安装测试REST服务器
pyresttest库自带了测试用的REST服务器。
```
cd /opt
git clone https://github.com/svanoort/pyresttest.git
pip install 'django >=1.6, <1.7' django-tastypie==0.12.1
cd pyresttest/pyresttest/testapp
python manage.py testserver test_data.json &
```
如果运行成功，进程会监听8000端口。如果不加&符号，就需要另外再启动一个终端访问block2虚拟机。用下面的命令测试一下刚启动的REST服务：
```
curl -s http://localhost:8000/api/person/2/ | python -m json.tool
```
如果执行成功会返回响应：
```
{
    "first_name": "Leeroy",
    "id": 2,
    "last_name": "Jenkins",
    "login": "jenkins",
    "resource_uri": "/api/person/2/"
}
```
## 配置文件的基本语法
配置文件使用YAML格式。  
有5个顶级的测试语法元素：  
 - **url**: 简单测试，用给定的GET请求来检测响应状态码是否正确。
 - **test**: 完整测试定义。
 - **benchmark**: 定义基准测试。
 - **config或configuration**: 全部测试设置(超时是最常见选项)
 - **import**: 导入其它配置文件

### 配置文件范例
一个典型的PyRestTest配置文件：
```
---
- config:
    - testset: "Basic tests"
    - timeout: 100  # Increase timeout from the default 10 seconds
- test: 
    - name: "Basic get"
    - url: "/api/person/"
- test: 
    - name: "Get single person"
    - url: "/api/person/1/"
- test: 
    - name: "Delete a single person, verify that works"
    - url: "/api/person/1/"
    - method: 'DELETE'
- test: # create entity by PUT
    - name: "Create/update person"
    - url: "/api/person/1/"
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
大致的解释：每个"- test:"代表一个测试，会发起一次REST请求；紧跟的对象往往是定义请求的内容；validators定义更复杂的验证逻辑，否则仅能通过响应状态码来确定测试是否通过；expected定义预期值；jsonpath_mini是提取语法用于从响应json串中提取值。

###测试1：首个冒烟测试
在/opt/pyresttest/pyresttes目录下建立test.yaml文件，文件内容：
```
---
- config:
    - testset: "Quickstart app tests"

- test:
    - name: "Basic smoketest"
    - url: "/api/people/"
```
然后执行：
```
resttest.py http://localhost:8000 test.yaml
```
发现报错，提示“测试组Default失败”。改正错误：
```
---
- config:
    - testset: "Quickstart app tests"

- test:
    - group: "Quickstart"
    - name: "Basic smoketest"
    - url: "/api/person/"
```
执行成功！

### 测试2：功能测试 - 创建/更新/删除
创建test2.yaml:
```
---
- config:
    - testset: "Quickstart app tests"

- test:
    - group: "Quickstart"
    - name: "Basic smoketest"
    - url: "/api/person/"

- test:
    - group: "Quickstart"
    - name: "Create a person"
    - url: "/api/person/10/"
    - method: "PUT"
    - body: '{"first_name": "Gaius","id": 10,"last_name": "Baltar","login": "baltarg"}'
```
发现报错，这是因为没有指定Content-Type导致的。修复的代码如下：
```
- test:
    - group: "Quickstart"
    - name: "Create a person"
    - url: "/api/person/10/"
    - method: "PUT"
    - body: '{"first_name": "Gaius","id": 10,"last_name": "Baltar","login": "baltarg"}'
    - headers: {'Content-Type': 'application/json'}
```
测试通过。然后增加一个测试检测刚刚添加的10号(Baltar)用户：
```
---
- config:
    - testset: "Quickstart app tests"

- test:
    - group: "Quickstart"
    - name: "Basic smoketest"
    - url: "/api/person/"

- test:
    - group: "Quickstart"
    - name: "Create a person"
    - url: "/api/person/10/"
    - method: "PUT"
    - body: '{"first_name": "Gaius","id": 10,"last_name": "Baltar","login": "baltarg"}'
    - headers: {'Content-Type': 'application/json'}

- test:
    - group: "Quickstart"
    - name: "Make sure Mr Baltar was added"
    - url: "/api/person/10/"
```
如果在最开始“希望”10号用户不存在：
```
---
- config:
    - testset: "Quickstart app tests"

- test:
    - group: "Quickstart"
    - name: "Make sure Mr Baltar ISN'T there to begin with"
    - url: "/api/person/10/"
    - expected_status: [404]

- test:
    - group: "Quickstart"
    - name: "Basic smoketest"
    - url: "/api/person/"

- test:
    - group: "Quickstart"
    - name: "Create a person"
    - url: "/api/person/10/"
    - method: "PUT"
    - body: '{"first_name": "Gaius","id": 10,"last_name": "Baltar","login": "baltarg"}'
    - headers: {'Content-Type': 'application/json'}

- test:
    - group: "Quickstart"
    - name: "Make sure Mr Baltar is there after we added him"
    - url: "/api/person/10/"
```
expected_status表示预期状态是404（表示该URL不存在），但10号用户(Baltar)已经存在，所以第一个测试不通过（Error）。
进一步改进测试代码，在最后增加删除10号用户(Baltar)的测试：
```
---
- config:
    - testset: "Quickstart app tests"

- test:
    - group: "Quickstart"
    - name: "Make sure Mr Baltar ISN'T there to begin with"
    - url: "/api/person/10/"
    - expected_status: [404]

- test:
    - group: "Quickstart"
    - name: "Basic smoketest"
    - url: "/api/person/"

- test:
    - group: "Quickstart"
    - name: "Create a person"
    - url: "/api/person/10/"
    - method: "PUT"
    - body: '{"first_name": "Gaius","id": 10,"last_name": "Baltar","login": "baltarg"}'
    - headers: {'Content-Type': 'application/json'}

- test:
    - group: "Quickstart"
    - name: "Make sure Mr Baltar is there after we added him"
    - url: "/api/person/10/"

- test:
    - group: "Quickstart"
    - name: "Get rid of Gaius Baltar!"
    - url: "/api/person/10/"
    - method: 'DELETE'

- test:
    - group: "Quickstart"
    - name: "Make sure Mr Baltar ISN'T there after we deleted him"
    - url: "/api/person/10/"
    - expected_status: [404]
```
现在6个测试全部通过了。这基本上是一个完整的测试，包括创建、查询、删除用户。

### 高阶向导
高阶向导的原始文档是[这里](https://github.com/svanoort/pyresttest/blob/master/advanced_guide.md)。  
PyRestTest可用于基准测试，而基准测试往往要运行多次来取平均值。PyRestTest提供Generators来产生数据。生成器(Generator)和模板本次测试忽略。
#### 提取器：jsonpath_mini
响应的例子：
```
{
    "thing":{"foo":"bar"},
    "link_ids": [1, 2, 3, 4],
    "person":{
        "firstname": "Bob",
        "lastname": "Smith",
        "age": 17
    }
}
```
在上述响应下使用提取器jsonpath_mini:
 - jsonpath_mini: 'person.lastname' 返回 "Smith"
 - jsonpath_mini: 'person.is_a_ninja' 返回 NOTHING (None object) ；因为person对象下没有这个key
 - jsonpath_mini: 'link_ids.1' 返回 2
 - jsonpath_mini: 'thing' 返回 {"foo":"bar"}
 - jsonpath_mini: '.'返回整个响应
 - jsonpath_mini: 'thing.0' 返回 None；因为thing不是数组
#### 提取器：header
从响应头中提取数据。不区分大小写。如果响应头中存在多个值(如cookie)则返回列表。
例子：
```
header: 'content-type'
```
例子2：
```
compare: {header: 'content-type', expected: 'application/json'}
```
#### 提取器：raw_body
返回整个响应体。

#### 验证器：extract_test
例子：
```
- validators:
    # Test key does not exist
    - extract_test: {jsonpath_mini: "key_should_not_exist",  test: "not_exists"}
```
检查值是否存在。

#### 验证器：compare
例子：
```
- validators:
     # Check the user name matches
     - compare: {jsonpath_mini: "user_name", comparator: "eq", expected: 'neo'}

     # Check the total_count key has value over 10
     - compare: {jsonpath_mini: "total_count", comparator: "gt", expected: 10}

     # Check the user's login
     - compare: {jsonpath_mini: "total_count", comparator: "gt", expected: }
```
参数有3个：提取器、比较函数、预期值。  
预期值也可以用提取器，这时可用于比较响应中包含的两个值。  
比较函数清单：  
 Name(s)                             |                Description                | Details for comparator(A, B)                
--------------------------------------------- | --------------------------------------- | -----------------------------  
 'count_eq','length_eq'                       | Check length of body/str or count of elements equals value      | ength(A)   == B   or -1 if cannot obtain length   
 'lt', 'less_than':                           | Less Than                                 | A < B                                                                  
 'le', 'less_than_or_equal'                   | Less Than Or Equal To                     | A <= B                                                                 
 'eq', 'equals'                               | Equals                                    | A == B                                                                 
 'str_eq'                                     | Values are Equal When Converted to String | str(A) == str(B) -- useful for comparing templated numbers/collections   
 'ne', 'not_equals'                           | Not Equals                                | A != B                                                               
 'ge', 'greater_than_or_equal'                | Greater Than Or Equal To                  | A >= B                                                                 
 'gt', 'greater_than'                         | Greater Than                              | A > B                                                                  
 'contains'                                   | Contains                                  | B in A                                                                 
 'contained_by'                               | Contained By                              | A in B                                                                 
 'type'                                       | Type of variable is                       | A instanceof (at least one of)   B  
 'regex'                                      | Regex Equals                              | A matches regex B                                                 

### json-server与httpbin.org
如果测试pyresttest需要一个REST模拟服务器。可以自己部署[json-server](https://github.com/typicode/json-server)或直接使用云服务[httpbin.org](http://httpbin.org/)。

