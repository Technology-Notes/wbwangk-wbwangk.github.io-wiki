openstack最新版是newton。目前（2016-11-28）在ubuntu上仅支持ubuntu16的xenial版（实测trusty版不行）。
vagrant box 是"ubuntu/xenial64"，provider是virtualbiox。在公司下载这个box花费了3个小时。  

安装keystone的机器域名是controller，在/etc/hosts文件中增加了一条记录：```192.168.1.116 controller```
也可以在Vagrantfile中直接定义hostname为"controller"。利用vagrant启动controller:  
```
$ vagrant up wbwang1
```
### keystone安装
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
### 测试keystone
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
xport OS_PROJECT_DOMAIN_NAME=default \
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
$ . demo-openrc
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