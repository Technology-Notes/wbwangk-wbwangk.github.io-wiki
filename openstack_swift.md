## VM安装
在(https://app.vagrantup.com/boxes/search)中搜索`openstack`发现了`stackinabox/openstack`这个vagrant box。它号称将这个openstack装入了这一个盒子中，操作系统是ubuntu16。  
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
登录操作系统并切换为root，通过`/etc/hosts`文件发现它的主机名是`openstack`，参照之前写的[这个文档](https://github.com/wbwangk/wbwangk.github.io/wiki/openstack-keystone)，编辑一个环境变量文件`admin-openrc`:
```
export OS_USERNAME=admin \
export OS_PASSWORD=labstack \
export OS_PROJECT_NAME=admin \
export OS_USER_DOMAIN_NAME=default \
export OS_PROJECT_DOMAIN_NAME=default \
export OS_AUTH_URL=http://openstack:35357/v3 \
export OS_IDENTITY_API_VERSION=3
```
执行环境变量设置，获取token：
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
  -H "X-Auth-Token: $ADMIN_TOKEN" | jq
```
