DSP是Data Sharing Platform(数据共享平台)的简写。[Kong](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong)是一个基于Nginx的微服务网关软件。  
本文讲的是如何在DSP中利用Kong实现“服务”类型的数据资源共享问题。数据资源的共享有三种方式：数据库、文件、服务。  
Kong在DSP中的作用有：
- 反向代理(路由功能)，将外部系统请求路由到相应的服务器  
- 认证功能，对外部调用者的身份进行认证  
- 权限控制，对于“有条件共享”且为“接口”类型的数据资源进行访问控制  

## Kong的部署
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

### 认证
对于“有条件共享”的数据资源，用户需要在DSP中提出“共享申请”。用户id和口令在DSP中定义，用户需要登录DSP才能发出“共享申请”。  
在Kong中将开发者用户称为consumer。Kong的认证类插件可以为Kong提供各种认证consumer的能力，如基础认证、API-KEY认证、OAUTH认证等。  
下面利用Kong的`key-auth`插件对以服务方式访问数据资源提供认证能力。有关`key-auth`插件的更多细节参考[这个文档](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#key-auth%E6%8F%92%E4%BB%B6)。  
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
关于`middleman`的更多细节，可以参考[这个文档](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#middleman子请求插件)。`middleman`不是官方插件，需要手工下载和构建，安装办法参考[这个](https://github.com/wbwangk/wbwangk.github.io/wiki/Kong#%E6%8F%92%E4%BB%B6%E5%AE%89%E8%A3%85)。手工安装后，执行下列命令向  

为了利用`middleman`完成服务的授权检查，需要DSP提供一个REST API来接收来自Kong的子请求(Nginx叫subrequest)。