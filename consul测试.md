
### consul安装
Consul是一个在国外流行的服务发现和配置共享的服务软件。fabio要依赖consul，或者说fabio增强了consul，使注册在consul的服务有了统一路由服务。  
这是[安装地址](https://www.consul.io/downloads.html)(含UI下载地址)。  
```
$ wget https://releases.hashicorp.com/consul/0.7.2/consul_0.7.2_linux_amd64.zip
$ unzip consul_0.7.2_linux_amd64.zip            (如果没有unzip工具，则先apt-get install unzip)
$ rm consul_0.7.2_linux_amd64.zip
$ mv consul /usr/local/bin            （把consul移动到可执行目录下）
$ consul agent -dev    (-dev表示开发模式)
```
屏幕显示一些日志，表示consul安装并启动。验证consul，需要进入另外一个终端（可参考[这个](https://github.com/wbwangk/wbwangk.github.io/wiki/virtualbox-vagrant-gitbash%E5%85%A5%E9%97%A8)），ssh登录到刚启动了consul的这台机器，执行：
```
$ consul members
$ curl localhost:8500/v1/catalog/nodes     （8500是consul的HTTP API端口）
```
会显示当前节点信息。如果上述命令正确执行就表示consul已经安装成功。上述命令行和HTTP API的功能相同。

### consul集群搭建
-dev选项用于启动单机的开发模式。集群部署不使用这个选项。  
每个节点须有唯一的id，默认使用hostname，可以用-node选项手工设置。  
使用-bind选项设置绑定的ip，这个ip必须能够被其他节点访问到。  
第一个节点会被当作集群的server，可以用-server选项来手工指定。-server选项可以指定agent工作在服务器还是客户端模式下。  
-bootstrap-expect选择的指定了consul集群的节点数，如果小于这个节点数，集群应不能工作（无法选举出领导？）。  
-config-dir选项指定了配置文件的目录，目录里定义了服务和健康检查策略。  
启动consul agent的命令：
```
$ consul agent -server -bootstrap-expect=1 \
    -data-dir=/tmp/consul -node=block1 -bind=10.10.11.85 \
    -config-dir=/etc/consul.d
   ...
```
启动另一个终端进入第二个节点，第二节点的节点名字是block2，ip是10.10.11.86。这个节点不是consul server，不能加server选项。启动consul agent的命令：
```
$ consul agent -data-dir=/tmp/consul -node=block2 \
    -bind=10.10.11.86 -config-dir=/etc/consul.d
   ...
```