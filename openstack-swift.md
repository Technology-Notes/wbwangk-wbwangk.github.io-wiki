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
创建一个测试容器：
```
$ openstack container create webb
+---------------------------------------+-----------+------------------------------------+
| account                               | container | x-trans-id                         |
+---------------------------------------+-----------+------------------------------------+
| AUTH_43694a2ef90f4a22af23552aa6836b4e | webb      | tx7860038a5a664a518b85b-0058647bd2 |
+---------------------------------------+-----------+------------------------------------+
```
创建这个容器的目的是为了确认当前账号的id(AUTH_43694a2ef90f4a22af23552aa6836b4e)。
#### 获取token(scoped)
```
DEMO_TOKEN=$(\
curl -X POST http://controller:5000/v3/auth/tokens \
    -s \
    -i \
    -H "Content-Type: application/json" \
    -d '
{
    "auth": {
        "identity": {
            "methods": [
                "password"
            ],
            "password": {
                "user": {
                    "domain": {
                        "name": "default"
                    },
                    "name": "demo",
                    "password": "vagrant"
                }
            }
        },
        "scope": {
            "project": {
                "domain": {
                    "name": "default"
                },
                "name": "demo"
            }
        }
    }
}' | grep ^X-Subject-Token: | awk '{print $2}' )
```
#### 显示账号明细和容器清单
AUTH_43694a2ef90f4a22af23552aa6836b4e是为测试创建的webb容器所属的账号。这个curl命令中使用了变量，如果测试出错就把变量替换为token值。
```
$ curl http://controller:8080/v1/AUTH_43694a2ef90f4a22af23552aa6836b4e \
  -H "X-Auth-Token: $DEMO_TOKEN"
container1
container12
owncloud
webb

$  curl http://controller:8080/v1/AUTH_43694a2ef90f4a22af23552aa6836b4e?format=json \
  -H "X-Auth-Token: $DEMO_TOKEN" | jq
[
  {
    "count": 2,
    "bytes": 15196,
    "name": "container1"
  },
  {
    "count": 0,
    "bytes": 0,
    "name": "container12"
  },
  {
    "count": 4,
    "bytes": 16934,
    "name": "owncloud"
  },
  {
    "count": 0,
    "bytes": 0,
    "name": "webb"
  }
]
```
#### 取对象清单
```
$ curl http://controller:8080/v1/AUTH_43694a2ef90f4a22af23552aa6836b4e/container1?format=json \
  -H "X-Auth-Token: $DEMO_TOKEN" | jq
[
  {
    "hash": "4940ee7233574154944ef23f78699f8c",
    "last_modified": "2016-11-29T07:28:25.403170",
    "bytes": 7598,
    "name": "/etc/swift/swift.conf",
    "content_type": "application/octet-stream"
  },
  {
    "hash": "4940ee7233574154944ef23f78699f8c",
    "last_modified": "2016-11-29T07:27:47.624300",
    "bytes": 7598,
    "name": "swift.conf",
    "content_type": "application/octet-stream"
  }
]
```
#### 取对象
```
$ curl http://controller:8080/v1/AUTH_43694a2ef90f4a22af23552aa6836b4e/container1/swift.conf \
  -H "X-Auth-Token: $DEMO_TOKEN" 
[swift-hash]

# swift_hash_path_suffix and swift_hash_path_prefix are used as part of the
# hashing algorithm when determining data placement in the cluster.
(下面的略)
```
响应的Content-Type: application/octet-stream。直接输出文件swift.conf的内容到响应体中。