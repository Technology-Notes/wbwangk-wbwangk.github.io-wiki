下文的openstack最新newton版keystone测试环境是按openstack官方的[install tutorial](http://docs.openstack.org/newton/install-guide-ubuntu/keystone.html)部署的。目前（2016-11-28）在ubuntu上仅支持ubuntu16的xenial版（实测trusty版不行）。
vagrant box 是"ubuntu/xenial64"，provider是virtualbiox。在公司下载这个box花费了3个小时。  

安装keystone的机器域名是controller，在/etc/hosts文件中增加了一条记录：  
```192.168.1.116 controller```  
也可以在Vagrantfile中直接定义hostname为"controller"。利用vagrant启动controller:  
```
$ vagrant up wbwang1
```
## keystone安装
初始化命令：
```
keystone-manage bootstrap --bootstrap-password vagrant \
  --bootstrap-admin-url http://controller:35357/v3/ \
  --bootstrap-internal-url http://controller:35357/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```

向keystone的mysql数据库添加授权，官方安装说明中授权的是'keystone'@'localhost',实测需要增加'keystone'@'controller'的授权(该观点未确认是否正确):
```
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'controller' \
  IDENTIFIED BY 'vagrant';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
  IDENTIFIED BY 'vagrant';
```
## 安装后测试keystone
环境脚本admin-openrc和demo-openrc位于/home/webb/目录下。
 - admin-openrc:
```
export OS_USERNAME=admin \
export OS_PASSWORD=vagrant \
export OS_PROJECT_NAME=admin \
export OS_USER_DOMAIN_NAME=default \
export OS_PROJECT_DOMAIN_NAME=default \
export OS_AUTH_URL=http://controller:35357/v3 \
export OS_IDENTITY_API_VERSION=3
```
 - demo-openrc:
```
export OS_PROJECT_DOMAIN_NAME=default \
export OS_USER_DOMAIN_NAME=default \
export OS_PROJECT_NAME=demo \
export OS_USERNAME=demo \
export OS_PASSWORD=vagrant \
export OS_AUTH_URL=http://controller:5000/v3 \
export OS_IDENTITY_API_VERSION=3 \
export OS_IMAGE_API_VERSION=2
```
获取认证token：
```
$ . admin-openrc           (小数点+空格+脚本表示执行它)
$ openstack token issue
+------------+----------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                              |
+------------+----------------------------------------------------------------------------------------------------+
| expires    | 2016-12-27 10:36:43+00:00                                                                          |
| id         | gAAAAABYYjYrngrJNOEEaGMdFhD-LJZDtFHcR54w-                                                          |
|            | mur9vFtPdOnlhehP80CfkkJEH3bKeHsnXMhMs1vnvTND3t5s4coiS9Jv1iBjAI75_ln-                               |
|            | BFZxM6lGh_bq5GpZXSgxsSCeiLGpfc515zZyAnoCLJMPH0HTMfJ2QhCxDlbQ8FfLwOZ2EibH0c                         |
| project_id | 6c0647b73b5e4c43888779b32446507a                                                                   |
| user_id    | c0f5c13dc43a455d81647fd49f2b1798                                                                   |
+------------+----------------------------------------------------------------------------------------------------+
```
## API测试
本测试参考了openstack[官方API文档](http://developer.openstack.org/api-ref/identity/v3/)

#### 密码换token(scoped)
```
ADMIN_TOKEN=$(\
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
                    "name": "admin",
                    "password": "vagrant"
                }
            }
        },
        "scope": {
            "project": {
                "domain": {
                    "name": "default"
                },
                "name": "admin"
            }
        }
    }
}' | grep ^X-Subject-Token: | awk '{print $2}' )
```
返回的token存在于响应头的X-Subject-Token中，然后 保存到环境变量$ADMIN_TOKEN中。查看token：
```
$ echo $ADMIN_TOKEN
gAAAAABYYxwRhlpHQqw5-HqSRpJN4tsPaG_F5fdIwzRyqC4Tvetq9eIBU4Nf3AZZLGO7gpOF5iwGfyAGiWZhyM_W6GfklKknUEb6K6SctH_TZP87M7NLIC91MN_0-gj1XvigHvoRx8qKCmSPBlQcsg7dkosE0Pr8jQ
```
认证token的默认有效期是一个小时。  

#### 取用户清单
```
$ curl http://controller:5000/v3/users \
  -H "X-Auth-Token: $ADMIN_TOKEN" | jq
```
（jq是个格式化显示json的工具，类似的还有jshon。不用jq只是显示的json串难读一些）  
使用git bash进入ubuntu16，执行上述curl命令失败，提示：
```
{"error": {"message": "The request you have made requires authentication.", "code": 401, "title": "Unauthorized"}}
```
当把$ADMIN_TOKEN替换为token的值本身时可以正常返回用户清单。而用justniffer监控keystone收到的请求，两种curl执行方式（使用环境变量与否）的请求是完全一样的。在git bash下（windows模拟linux）下执行两种curl方式都可以成功。

#### 取具体用户信息
```
curl http://controller:5000/v3/users/cbf4cff4cb874ce5a9ca075fd96649c4 \
  -H "X-Auth-Token: $ADMIN_TOKEN" | jq
```
上面请求会返回“manila”这个用户的信息，而manila这个用户的id(cbf4cff4cb874ce5a9ca075fd96649c4)是“取用户清单”这个API返回的。

#### 创建用户
```
$ curl -X POST http://controller:5000/v3/users \
    -H "X-Auth-Token: $ADMIN_TOKEN" \
    -H "Content-Type: application/json" \
    -d \
'{
    "user": {
        "enabled": true,
        "name": "James Doe",
        "password": "secret"
    }
}'
```
创建用户后可以重新调用“取用户清单”的API，可以看到新增加的James Doe用户。

### 另一种密码换token(unscoped)
```
ADMIN_TOKEN=$(\
curl http://controller:5000/v3/auth/tokens \
    -i \
    -H "Content-Type: application/json" \
    -d '
{ "auth": {
    "identity": {
      "methods": ["password"],
      "password": {
        "user": {
          "name": "admin",
          "domain": { "id": "default" },
          "password": "vagrant"
        }
      }
    }
  }
}' | grep ^X-Subject-Token: | awk '{print $2}' )
```
使用这种不确定scope的令牌来执行“取用户清单”报错如下：
```
{"error": {"message": "You are not authorized to perform the requested action: identity:list_users", "code": 403, "title": "Forbidden"}}
```
打开文件/etc/keystone/policy.json，发现identity:list_users的默认设置是：
```
 "identity:list_users": "rule:admin_required",
```
如果修改为：
```
 "identity:list_users": "",
```
则“取用户清单”的请求就可以正常执行了。而admin明明是admin角色，却被keystone认为没有权限。这可能是个keystone的bug。用scoped令牌，没有问题。

## 概念
 - **project**  
    一个对服务或认证对象进行分组或隔离的容器。可以映射到客户、账号、组织或租户。
 - **domain**   
    项目和用户的集合。一个域下可能包含多个项目（租户）。一个用户可以指定为某域的管理员角色。
 - **endpoint**  
    一个网络访问地址，一般是个URL，通过它可以访问某个服务。
 - **region**  
    一个认证API的v3版实体，用于表示openstack部署的分布，如可以用地域表示。
 - **fernet**
    /etc/keystone/keystone.conf中配置的token默认provider