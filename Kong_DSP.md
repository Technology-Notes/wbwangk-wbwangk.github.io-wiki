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

