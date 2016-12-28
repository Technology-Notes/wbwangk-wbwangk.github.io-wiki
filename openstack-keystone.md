下文的openstack最新newton版keystone测试环境是按[文档](http://docs.openstack.org/newton/install-guide-ubuntu/keystone.html)部署的。目前（2016-11-28）在ubuntu上仅支持ubuntu16的xenial版（实测trusty版不行）。
vagrant box 是"ubuntu/xenial64"，provider是virtualbiox。在公司下载这个box花费了3个小时。  

安装keystone的机器域名是controller，在/etc/hosts文件中增加了一条记录：```192.168.1.116 controller```
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
$ . demo-openrc           (小数点+空格+脚本表示执行它)
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
在这个[官方API文档](http://developer.openstack.org/api-ref/identity/v3/?expanded=password-authentication-with-unscoped-authorization-detail)中，写是POST方法，实测必须用GET方法才能正确返回。
#### 取unscopted token
```
$ curl -i -H 'Content-Type: application/json' http://controller:5000/v3/auth/tokens \
    -d '{"auth": {"identity": {"methods": ["password"],"password": {"user": {"name": "admin", 
   "domain": {"name": "default" },"password": "vagrant"}}}}}'     （实测发现 curl的-d参数中不能用反斜杠）

X-Subject-Token: (header的其它部分略)  gAAAAABYYwbwwGx3IektpWI_QpvfibRCTsbBKPM2RuVZLzs9xI9Bkiw1Fhpn23osGS1QsSSqWWyqytirIRtwNfW9CuqnYxzHhlW2HipxJMObCRuCNWjaZ4YZEyozS1rMZROfzSh5i5TsvKNNoKW6IVADikoXLOj13w

{"token": {"issued_at": "2016-12-28T00:27:28.000000Z", "audit_ids": ["vOzeX48sRruswQm6guKwyw"], "methods": ["password"], "expires_at": "2016-12-28T01:27:28.000000Z", "user": {"domain": {"id": "default", "name": "Default"}, "id": "c0f5c13dc43a455d81647fd49f2b1798", "name": "admin"}}}
```
响应中的X-Subject-Token就是要取的token，调用openstack REST API都需要使用这个token,方法是在设置在http头中的X-Auth-Token。

#### 取scoped token
```
ADMIN_TOKEN=$(\
curl -i -s http://controller:5000/v3/auth/tokens \
    -H "Content-Type: application/json" \
    -d '{
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
#### 取domain
```
ID_ADMIN_DOMAIN=$(\
curl http://controller:5000/v3/domains \
    -s \
    -H "X-Auth-Token: $ADMIN_TOKEN" \
    -H "Content-Type: application/json" \
    -d '
{
    "domain": {
    "enabled": true,
    "name": "default"
    }
}' | jq .domain.id | tr -d '"' )
```
（jq是个格式化显示json的工具，类似的还有jshon）
## 概念
 - project
    一个对服务或认证对象进行分组或隔离的容器。可以映射到客户、账号、组织或租户。
 - domain 
    项目和用户的集合。一个域下可能包含多个项目（租户）。一个用户可以指定为某域的管理员角色。
 - endpoint
    一个网络访问地址，一般是个URL，通过它可以访问某个服务。
 - region
    一个认证API的v3版实体，用于表示openstack部署的分布，如可以用地域表示。