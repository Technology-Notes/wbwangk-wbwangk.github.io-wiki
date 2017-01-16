项目地址：https://github.com/atarola/pyjojo
## 准备工作 ##
使用ubunt14TLS做的验证，它已经默认装了python2.7.6和ython3.4.0。
安装pip，是python应用的部署工具：
```
apt-get install python-pip
```
如果是centos，则先安装```yum install epel-release```再执行```yum install python-pip```
然后用pip装pyjojo：
```
pip install pyjojo
```
安装后测试一下：
```
pyjojo --help
```
会显示很多命令行说明，表明安装成功。
运行pyjojo：
```
pyjojo -d --dir /srv/pyjojo
```
pyjojo默认监听3000端口，可以用浏览器访问```localhost:3000```测试一下。
我要用apache htpasswd工具来生成密码文件，要先安装它。执行：
```
apt-get install apache2-utils
```
安装成功后执行一个命令测试一下：```htpasswd --help```
再生成一个密码测试一下：```htpasswd -nb wbwang 123```。系统会显示```wbwang:$apr1$GfIUY8Yf$OuMYbvtJ.gDasIn3wvGvn1```
这就是生成的密码文件。这个文件可以直接被apache httpd或nginx解析，实现http基础认证。用户名wbwang，密码是123。

## 正式验证pyjojo ##
```
cd /srv/pyjojo
vi htpasswd.sh
```
输入以下内容：
```
#!/bin/bash

# -- jojo --
# description: httpasswd密码文件生成
# param: user - 用户id
# param: pwd - 密码
# filtered_param: pwd - 不要把密码输出到日志
# -- jojo --

echo "jojo_return_value user=$USER"
pwdfile=$(htpasswd -nb $USER $PWD); echo "jojo_return_value htpasswd=$pwdfile"
exit 0
```
第一行是是说明这是一个bash脚本，需要/bin目录下bash应用去解释这个脚本文件。被```-- jojo ---```包裹的区域是元数据定义区。根据定义的元数据，这个服务接受两个参数，用户id和密码。

刚编辑生成的htpasswd.sh脚本是不可执行的，需要用linux命令赋予它执行的权限：
```
chmod +x htpasswd.sh
```
测试一下这个脚本文件。注意这个脚本的文件名htpasswd.sh将自动变成URL的一部分。
```
curl -XPOST http://localhost:3000/scripts/htpasswd -H "Content-Type: application/json" -d '{"user": "wbwang", "pwd": "123"}'
```
响应如下：
```
{"retcode": 0, "return_values": 
    {"htpasswd": "wbwang:$apr1$Q3sWEa2V$6de019fnluH.v9N0kV50M0", "user": "wbwang"}, 
    "stderr": [], 
    "stdout": ["jojo_return_value user=wbwang", "jojo_return_value htpasswd=wbwang:$apr1$Q3sWEa2V$6de019fnluH.v9N0kV50M0"]}
```
注意，即使用户名口令不变，每次调用htpasswd生成的密码文件也会不同。因为有个随机数当加密盐，但并不影响密码的解析。

至此，htpasswd工具就被包装成了一个API，API地址是：```localhost:3000/scripts/htpasswd```。这个API的输入输出都是json字符串。
