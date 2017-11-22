## VM安装
在(`https://app.vagrantup.com/boxes/search`)中搜索`openstack`发现了`stackinabox/openstack`这个vagrant box。它号称将这个openstack装入了这一个盒子中，操作系统是ubuntu16。  
为了加快本地安装速度，将该box手工下载后上传到了地址`http://repo.imaicloud.com/stackinabox_openstack.box`。  

翻阅它的说明文档，它在github上的源码库：`https://github.com/stackinabox/stackina-base-box`。  
下面安装github上的[readme说明](https://github.com/stackinabox/stackina-base-box)进行安装。进入git bash命令行:
```
$ cd /e/vagrant10/ambari-vagrant/
$ git clone https://github.com/tpouyer/stackina-base-box.git
$ cd stackina-base-box/vagrant
$ cp Personalization.dict Personalization
```
编辑文件`Personalization`，令`$use_nfs = false`，将`box_url`修改为：
```
$box_url = http://repo.imaicloud.com/stackinabox_openstack.box
```
然后启动和登录虚拟机：
```
$ vagrant up --provider=virtualbox
$ vagrant ssh
```
### 测试keystone
登录操作系统并切换为root，通过`/etc/hosts`文件发现它的主机名是`openstack`，安装之前写的[这个keystone文档](https://github.com/wbwangk/wbwangk.github.io/wiki/openstack-keystone)中的步骤，编辑一个环境变量文件`admin-openrc`(如在/opt目录下):
```
export OS_USERNAME=admin \
export OS_PASSWORD=labstack \
export OS_PROJECT_NAME=admin \
export OS_USER_DOMAIN_NAME=default \
export OS_PROJECT_DOMAIN_NAME=default \
export OS_AUTH_URL=http://openstack:35357/v3 \
export OS_IDENTITY_API_VERSION=3
```
执行环境变量设置，获取token，根据token访问keystone API获取用户清单：
```
$ . admin-openrc
$ openstack token issue
+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| expires    | 2017-11-21 08:11:53+00:00        |
| id         | d8ce9c91fa77473b9d89378f96ad68d0 |
| project_id | 719860ba274f48329021404f219aca80 |
| user_id    | 5e778aa892d643bdadf8304e2eff970e |
+------------+----------------------------------+
$ export ADMIN_TOKEN=d8ce9c91fa77473b9d89378f96ad68d0
$ curl http://openstack:5000/v3/users \
  -H "X-Auth-Token: $ADMIN_TOKEN"
```
## Swift安装
可以用下列命令查看一下当前部署的openstack的端点清单：
```
$ openstack endpoint list
```
悲催的发现，当前VM中没有安装对象存储服务(swift)，只能手工安装。计划安装单节点swift服务。  
下面的swift安装参考了[openstack swift官方安装文档](https://docs.openstack.org/swift/pike/install/controller-install-ubuntu.html)和一篇文章《[swift(Object Storage对象存储服务)(单节点)](http://www.jianshu.com/p/c1da0b5f5668)》。  
### 准备
#### 创建swift用户
```
$ . admin-openrc
$ openstack user create --domain default --password-prompt swift
User Password: labstack
```
创建了一个`default`域下的用户`swift`。  
为`swift`用户添加`admin`角色：
```
$ openstack role add --project service --user swift admin
```
创建`swift`服务实体：
```
$ openstack service create --name swift \
  --description "OpenStack Object Storage" object-store
```
#### 创建对象存储服务API端点
```
$ openstack endpoint create --region RegionOne \
  object-store public http://openstack:8080/v1/AUTH_%\(project_id\)s
$ openstack endpoint create --region RegionOne \
  object-store internal http://openstack:8080/v1/AUTH_%\(project_id\)s
$ openstack endpoint create --region RegionOne \
  object-store admin http://openstack:8080/v1
```
openstack中的project就是租户，不同租户的公开和私有端点URI不同，而管理端点相同。  

### 安装和配置组件
#### 安装软件包
```
$ apt-get install swift swift-proxy python-swiftclient \
  python-keystoneclient python-keystonemiddleware memcached
```
创建`/etc/swift`目录。  
从对象存储源码库获取代理服务的配置文件：
```
$ curl -o /etc/swift/proxy-server.conf https://git.openstack.org/cgit/openstack/swift/plain/etc/proxy-server.conf-sample?h=stable/pike
```
编辑配置文件`/etc/swift/proxy-server.conf`以包括下列内容：  
```
[DEFAULT]
...
bind_port = 8080
user = swift
swift_dir = /etc/swift
```
在`[pipeline:main]`小节，删除`tempurl`和`tempauth`模块，添加`authtoken`和`keystoneauth`模块：
```
[pipeline:main]
pipeline = catch_errors gatekeeper healthcheck proxy-logging cache container_sync bulk ratelimit authtoken keystoneauth container-quotas account-quotas slo dlo versioned_writes proxy-logging proxy-server
```
不要改变上述各模块的顺序。  
在[app:proxy-server]小节，启用账号自动创建：
```
[app:proxy-server]
use = egg:swift#proxy
...
account_autocreate = True
```
在`[filter:keystoneauth]`小节，配置操作者角色：
```
[filter:keystoneauth]
use = egg:swift#keystoneauth
...
operator_roles = admin,user
```
在`[filter:authtoken]`小节，配置认证服务访问权限：
```
[filter:authtoken]
paste.filter_factory = keystonemiddleware.auth_token:filter_factory
...
auth_uri = http://openstack:5000
auth_url = http://openstack:35357
memcached_servers = openstack:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = swift
password = labstack
delay_auth_decision = True
```
在`[filter:cache]`小节，配置`memcached`地址：
```
[filter:cache]
use = egg:swift#memcache
...
memcache_servers = openstack:11211
```
### 测试swift
按照之前写的文档[openstack-swift测试](https://github.com/imaidev/imaidev.github.io/wiki/openstack-swift%E6%B5%8B%E8%AF%95)中操作来测试刚刚安装的VM中swift。  

