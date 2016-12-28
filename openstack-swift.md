openstack swift proxy安装在controller(192.168.1.116)上，存储节点是object1和object2。启动swift3个节点的的vagrant命令是:  
```vagrant up wbwang1 object1 object2```。  
在controller上/home/webb/目录下测试swift的部署情况：
```
$ . demo-openrc
$ swift stat 
$ openstack object list container1          (列出容器container1下所有对象)
```
**[这里]**是swift API官方文档的地址。  
## API测试
下面这个请求不需要认证：
```
$ curl http://controller:8080/info | jq
```
(jq是个json格式化显示的工具，类似的还有jshon)
后续的API测试需要认证，首先要获取token。openstatck认证服务的相关测试参考[openstack keystone](openstack keystone)。
token存放于环境变量$ADMIN_TOKEN中。可以通过demo-openrc看到demo用户的相关信息：
```
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=vagrant
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```
后续测试都使用这个demo用户换取的token进行测试。