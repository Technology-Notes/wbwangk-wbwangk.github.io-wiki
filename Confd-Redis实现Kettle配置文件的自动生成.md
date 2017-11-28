## Confd+Redis实现Kettle配置文件的自动生成
通过confd监听redis数据变化，根据定义的Kettle配置模板自动生成配置文件<br>
以drive_job.kjb模版为例（drive_trans.ktr同）
### Confd安装
下载：
```
curl -L https://github.com/kelseyhightower/confd/releases/download/v0.11.0/confd-0.11.0-linux-amd64 
```
将下载的 confd-0.11.0-linux-amd64 文件复制到/usr/local/bin/目录，并重命名为confd：
```
cp confd-0.11.0-linux-amd64 /usr/local/bin/confd
```
创建confd工作目录：
```
mkdir -p /etc/confd/{templates,conf.d}
```

### 模板配置
1、创建模板源文件<br>
/etc/confd/conf.d/redis_kettle.toml:
```
[template]
src = "redis_kettle_module.tmpl"
dest = "/etc/confd/redis_kettle_list.conf"
mode = "0777"
keys = [
"/kettle"
]
check_cmd = ""
reload_cmd = "cd /etc/confd/conf.d & java ConfdTemplateSplit /etc/confd/redis_kettle_list.conf" 
```
src: 指定配置模板文件<br>
dest：根据模板文件生成的配置模板列表文件，生成的该文件内容是根据数据key不同生成的若干个配置集合<br>
mode: 生成文件的权限<br>
keys：etcd中存放数据的目录<br>
check_cmd: 检查配置文件脚本，如:nginx -t -c {{.src}}，验证nginx配置文件正确性<br>
reload_cmd: 重新加载配置文件脚本，如:nginx -s reload，重启nginx;  该例中，此处指定配置模板表文件(/etc/confd/redis_kettle_list.conf)的处理脚本<br>

2、创建配置文件模板文件<br>
/etc/confd/templates/redis_kettle_module.tmpl:
```
{{range gets "/kettle/*"}}
    {{$data := json .Value}}
    [template]
    src = "redis_drive_job.kjb"
    dest =  "/tmp/{{$data.resource_id}}_{{$data.created_user}}.conf"
    keys = [
    "/kettle/{{$data.resource_id}}_{{$data.created_user}}"
]
check_cmd = ""  #此处可以指定nginx或者kettle的配置检查脚本命令
reload_cmd = ""  #此处可以指定nginx或者kettle的重新加载脚本命令
{{end}}
```
说明：循环读取etcd数据库的/kettle目录下的所有key，key的值均为json字符串数据，解析出json的所需字段，填入模板中，生成配置文件<br>

3、上述两个步骤的目的是生成步骤[1]中dest指定的配置模板列表文件：/etc/confd/redis_kettle_list.conf，该文件格式为：
```
[template]
src = "redis_drive_job.kjb"
dest = "/tmp/1000_fogray.conf"
keys = [
"/kettle/1000_fogray"
]
check_cmd = ""
reload_cmd = ""

[template]
src = "redis_drive_job.kjb"
dest = "/tmp/1001_fogray1.conf"
keys = [
"/kettle/1001_fogray1"
]
check_cmd = ""
reload_cmd = ""
......
```
该文件生成后，confd就会自动执行步骤[1]的reload_cmd指定的脚本命令，解析该文件的[template]标签，每个[template]标签的内容，均生成一个独立的配置文件，该配置文件即为最终的能够被第三方服务使用的配置文件<br>
4、Kettle模板
/etc/confd/templates/redis_drive_job.kjb：
```
{{range gets "/kettle/*"}}
{{$data := json .Value}}
<job>
  <name>use_{{$data.resource_id}}_{{$data.created_user}}</name>   <!-- 作业名（需按照约定格式来，不可重复） -->
  <description></description >          <!-- 作业描述 （可选）-->
  <extended_description ></extended_description > 
  <job_version></job_version >            
  <job_status>0</job_status>
  <directory>/</directory>
  <created_user>{{$data.created_user}}</created_user> <!-- 创建者（部门id可选） -->
  <created_date>{{$data.created_date}}</created_date><!-- 创建时间 -->
  <modified_user></modified_user><!-- 修改者（可选） -->
  <modified_date></modified_date><!-- 修改时间 -->
  ......
</job>
{{end}}
```

5、测试<br>
启动confd：
```
confd -config-file /etc/confd/conf.d/redis_kettle.toml -interval 10 -backend redis -node http://localhost:2379
```
指定配置文件、confd后端为redis、redis服务url启动confd，并每隔10s监听一次redis的数据变化更新配置文件<br>
向redis中存入数据:
```
redis-cli set /kettle/1000_fogray "{\"resource_id\": \"1000\", \"created_user\": \"fogray\", \"target_source_name\": \"v6db\", \"target_source_ip\": \"10.10.10.105\", \"target_source_type\": \"postgresql\", \"target_source_access\": \"jdbc\", \"target_source_database\": \"v6db\", \"target_source_port\": \"5432\", \"target_source_userid\": \"postgres\", \"target_source_password\": \"123456a?\", \"target_source_dtp\": \"td_base\", \"target_source_itp\": \"td_base_index\"}"
```
然后，分别检查目录：<br>
/etc/confd/conf.d/目录下是否生成1000_fogray.toml配置模板文件<br>
/tmp/目录下是否生成1000_fogray.conf配置文件
