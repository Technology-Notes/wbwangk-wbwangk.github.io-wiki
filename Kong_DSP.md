DSP是Data Sharing Platform(数据共享平台)的简写。[Kong](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong)是一个基于Nginx的微服务网关软件。  
本文讲的是如何在DSP中利用Kong实现“服务”类型的数据资源共享问题。数据资源的共享有三种方式：数据库、文件、服务。  
Kong在DSP中的作用有：
- 反向代理(路由功能)，将外部系统请求路由到相应的服务器  
- 认证功能，对外部调用者的身份进行认证  
- 权限控制，对于“有条件共享”且为“接口”类型的数据资源进行访问控制  
Kong具有丰富的插件系统，即有[官方的插件](https://konghq.com/plugins/)，也可以在github上找到一些“定制插件”。目前计划在DSP中使用两个Kong插件，一个官方的key-auth，一个定制的midlleman。随着项目的深入，可能还会引入其它的Kong插件。  

### Kong的部署
Kong的安装参考[这个文档](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#kong安装)。  
假定Kong安装在IP:192.168.73.102的VM上。DSP以集群部署，则在每台DSP部署的VM上配置`/etc/hosts`:
```
192.168.73.102 kong
```
然后DSP就可以类似通过下面的方式配置Kong了：
```
$ curl http://kong:8001/apis/
```
#### 保护Kong管理端口
可以利用Kong自己的能力对管理端口(默认是8001)进行保护：
```
$ curl http://kong:8001/apis \
  --data name=admin-api \
  --data uris=/admin-api \
  --data upstream_url=http://kong:8001
```
上述命令给Kong增加一个API，将对`/admin-api`上下文的请求转发到Kong管理端口：
```
$ curl http://kong:8000/admin-api/apis
```
上述命令等价于：` curl http://kong:8001/apis/`。   
然后就可以利用Kong的能力为API`admin-api`增加认证、IP限制等。

#### 启用key-auth插件
DSP对外提供的服务都需要认证后才能使用，为此部署Kong后需要手工为所有API（包括上面的`admin-api`）启用key-auth插件。key-auth插件可以为开发者生成一个apikey，要求对于DSP服务的调用带上apikey。有关`key-auth`插件的更多细节参考[这个文档](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#key-auth插件)。  
为Kong的所有API启用key-auth插件的方法是：
```
$ curl -X POST http://kong:8001/plugins/  \
    --data 'name=key-auth'
```

### 认证
对于“有条件共享”的数据资源，用户需要在DSP中提出“共享申请”。用户id和口令在DSP中定义，用户需要登录DSP才能发出“共享申请”。  
在Kong中将开发者用户称为consumer。Kong的认证类插件可以为Kong提供各种认证consumer的能力，如基础认证、API-KEY认证、OAUTH认证等。  
下面利用Kong的[key-auth](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#key-auth插件)插件对以服务方式访问数据资源提供认证能力。  
首先定义一个约定：Kong的consumer的username属性值为DSP的用户id。为了安全，DSP并不保存用户的`apikey`，需要在界面上提示用户自己保存好`apikey`的字符串。用户应用调用DSP服务时，需要提供`apikey`。  

#### consumer注册与apikey申请
DSP需要提供一个界面供用户申请apikey。在申请apikey的逻辑中，隐含着consumer的注册。当用户点击申请apikey的按钮后，DSP应执行下面的逻辑来向Kong增加consumer：
```
$ curl -X POST --url http://kong:8001/consumers/ \
  --data "username=webb"
```
将webb替换为DSP的当前用户id。如果kong提示这个用户id已经存在，说明他申请过apikey，忽略“用户id已存在”的错误，继续执行即可。因为同一个consumer可以创建多个apikey。  
  
当有了consumer后，DSP接着执行下面的逻辑来创建apikey：
```
$ curl -i -X POST --url http://localhost:8001/consumers/webb/key-auth/ \
  --data 'key=<apikey>'
```
apikey为自动生成的32位的随机数或UUID。上述逻辑成功后再屏幕上显示apikey，并提醒用户“系统不会保存这个apikey，要自己保存好”，然后提示apikey的用法：
```
$ curl http://kong:8000/<api-path>?apikey=<apikey>   (或)
$ curl -H "apikey: <apikey>" http://kong:8000/<api-path>
```

### 授权
DSP的有条件共享服务的授权比较特殊，它通过一个申请流程来完成服务的授权。这导致传统的基于角色的授权(RBAC)模型变得无效。因为用户是否有权限访问某个服务是“随机”的，无法用角色对服务进行分组。  

Kong的官方授权插件不能满足需求，选择了一个定制插件`middleman`来满足这个特殊的授权需求。`middleman`插件使得Kong在执行对API的反向代理前向一个指定URL发送请求，如果返回的HTTP状态码是2xx，反向代理会顺利执行，如果返回的状态码是401或402，反向代理会终止。`middleman`除了可以用于授权，还可用于认证或其他任意适合的用途。  

关于`middleman`的更多细节，可以参考[这个文档](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#middleman插件)。`middleman`不是官方插件，需要手工下载和构建，安装办法参考上文的文档。  
手工安装后，执行下列逻辑向指定API添加插件：
```
$ curl -X POST http://kong:8001/apis/example-api2/plugins \
    --data "name=middleman" \
    --data "config.url=http://dsp:8080/<DSP_hook>?api=example-api2"
```  
需要注意的是，只有“有条件共享”的服务才需要添加`middleman`插件，无条件共享的服务不需要添加该插件。  
上面的`<DSP_hook>`是需要DSP提供的REST API，用来接收来自Kong的子请求(Nginx叫subrequest)。DSP_hook收到的请求体是个json串(格式见[这里](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#测试子请求返回200状态码))，里面有请求的用户id，而URL参数中有API名称。根据用户id和API名称可以定位到“数据资源”，然后到共享申请结果表中查看这个用户是否允许访问这个服务，以决定返回200还是401状态码。  

至于API名称如何对应数据资源，见下文。  

#### 数据资源与API名称
数据资源分三种：数据库、文件、接口。接口又称服务，Kong中称为API，一般是SOAP（俗称web service）格式或REST风格。  
Kong的API名称和`uris`可以是不同的，为了简化DSP中服务的配置，约定将Kong应用在DSP时，总是让API名称与`uris`属性相同，并用“URI前缀”来称谓它。  
数据资源维护时，当用户选择“接口”类型时，需要渲染出服务定义区域，内容有：  
- URI前缀，就是例子中的`/my-path`，在Kong中同时作为API名称(例子中叫example-api2)和'uris'属性。约定API名称与`uris`相同。  
- 上游URL，就是将`/my-path`反向代理到的URL。也就是`upstream_url`的值。  

由于服务定义的属性很少，暂时不增加服务定义表，而是直接将服务的属性保存在“数据资源”表中。URI前缀保存到`TABLE_NAME`字段，上游URL保存到`TABLE_DESCRIPTION`字段。  

当用户按保存按钮后，DSP应执行下列逻辑，将这个服务定义到Kong中：
```
$ curl http://kong:8001/apis \
  --data name=<URI前缀> \
  --data uris=/<URI前缀> \
  --data upstream_url=<上游URL>
```
如果当前的数据资源是“有条件共享”的，还要执行下列逻辑，为这个API添加权限检查：
```
$ curl -X POST http://kong:8001/apis/<URI前缀>/plugins \
    --data "name=middleman" \
    --data "config.url=http://dsp:8080/<DSP_hook>?api=<URI前缀>"
```
当把一个数据资源从“有条件共享”修改成“无条件共享”时，需要删除权限检查逻辑：
```
$ curl kong:8001/apis/example-api2/plugins      (显示某API的插件清单，找到middleman的插件id)
$ curl -X DELETE kong:8001/apis/example-api2/plugins/<middleman plugin id>
```

### DSP授权API
DSP需要实现一个授权检查API，供Kong的middleman插件调用。这个授权API的调用参数参考[这个文档](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#子请求API的实现)。  
在DSP授权API的实现中，从URL的`api`参数中取出"URI前缀"，从请求体的JSON串的`x-consumer-username`属性中取出用户id。然后到数据资源申请表（OD_DATARESOURCE_APPLY）中查找，看是否有审批通过的记录(需要联查数据资源表)。如果已经审批通过，则返回200状态码，如果没有就返回401状态码。  
为了供Kong找到DSP的授权API应用的IP地址，也像Kong一样，在部署Kong的主机的`/etc/hosts`文件中添加类似的记录：  
```
192.168.73.3  dsp
192.178.73.2  dsp
```
可以定义多个ip，操作系统会自动选择一个(这貌似叫DNS轮询)。这种办法虽然可以定义一个简单的集群，但端口号只能配置成一样的。如果有条件，可以用Nginx或httpd配置出更完善的集群，然后将dsp配置到Nginx或httpd的ip。  

### 前端OAuth授权
文件上传可以通过前端js完成，也可以通过后端java应用完成。直接用前端js实现文件上传更简单，但apikey是后端用的，前端一般用OAuth实现认证和授权。  
利用Kong的oauth2插件实现OAuth授权很容易。尤其在DSP的场景下，只需要“客户端凭据”方式的授权即可。详细描述见文档"[Kong_OAuth2](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong_OAuth2)"。  

需要强调的是，添加API、创建消费者、创建oauth应用等步骤只需要手工完成。拿到客户端凭据后，根据客户端凭据换取令牌的动作才需要写程序。  
利用Kong对Webdav协议(文件服务)进行代理的测试见[这个文档](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#kong与webdav)

### CORS跨域资源共享
将来部署时，如果Kong所在的域与DSP所在的域不一样，需要设置CORS，以便让DSP域的网页js可以调用Kong域的API。  
在[这个WIKI](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#cors插件)中示范了Kong允许所有域的API调用。  
