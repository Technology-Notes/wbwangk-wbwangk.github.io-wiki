
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
这时，你就有了两个consul agent，一个server一个客户端。现在两个agent还不知道彼此的存在，都是独立在运行。可以分别通过命令consul members命令查看节点清单：
```
$ consul members       （在节点1上运行）
Node    Address           Status  Type    Build  Protocol  DC
block1  10.10.11.85:8301  alive   server  0.7.2  2         dc1
$ consul members        （在节点2上运行）
Node    Address           Status  Type    Build  Protocol  DC
block2  10.10.11.86:8301  alive   client  0.7.2  2         dc1
```
现在登录第一个节点，让节点1加入节点2：
```
$ consul join 10.10.11.86
Successfully joined cluster by contacting 1 nodes.
$ consul members
Node    Address           Status  Type    Build  Protocol  DC
block1  10.10.11.85:8301  alive   server  0.7.2  2         dc1
block2  10.10.11.86:8301  alive   client  0.7.2  2         dc1
```
要加入consul集群，agent只要join集群的任何一个成员都可以。  
通过contrl+C将block1的agent进程终止，然后进入block2：
```
$ vagrant ssh block2
$ consul members
Node    Address           Status  Type    Build  Protocol  DC
block1  10.10.11.85:8301  failed  server  0.7.2  2         dc1
block2  10.10.11.86:8301  alive   client  0.7.2  2         dc1
```
看到节点block1的状态是failed。  
重新启动节点1的consul agent，然后再执行consul members，两个节点都显示为alive状态，consul集群回复正常。  

### 服务定义
[官方原文](https://www.consul.io/intro/getting-started/services.html)  
想consul注册服务可以通过提供一个[服务定义](https://www.consul.io/docs/agent/services.html)，或通过调用[HTTP API](https://www.consul.io/api/index.html)。

#### 通过服务定义注册服务
首先，为Consul配置创建一个目录。Consul从配置目录中加载所有配置文件，一般惯例使用的配置目录是`/etc/consul.d`：
```
$ sudo mkdir /etc/consul.d
```
下面，我们要编写一个服务定义配置文件。假设我们有一个叫"wbb"的服务运行在80端口。在定义中，我们设置了一个标签用于查询：
```
$ echo '{"service": {"name": "web", "tags": ["rails"], "port": 80}}' \
    | sudo tee /etc/consul.d/web.json
```
现在，重启consul代理，在参数中指定配置目录：
```
$ consul agent -dev -config-dir=/etc/consul.d
==> Starting Consul agent...
...
    [INFO] agent: Synced service 'web'
...
```
你注意到了在输出中显示consul “synced” 了这个web服务。这意味着consul代理加载了配置文件中的服务定义，并成功将它注册到了服务目录中。

如果你想定义多个服务，你可以在Consul配置目录中创建多个服务定义文件。

#### 通过HTTP API定义服务
[原文](https://www.consul.io/api/catalog.html)
通过HTTP API创建服务需要的参数比较多：
```
$ curl -X PUT http://127.0.0.1:8500/v1/catalog/register \
-d '{
  "Datacenter": "dc1",
  "ID": "40e4a748-2192-161a-0510-9bf59fe950b5",
  "Node": "foobar",
  "Address": "192.168.10.10",
  "TaggedAddresses": {
    "lan": "192.168.10.10",
    "wan": "10.0.10.10"
  },
  "NodeMeta": {
    "somekey": "somevalue"
  },
  "Service": {
    "ID": "redis1",
    "Service": "redis",
    "Tags": [
      "primary",
      "v1"
    ],
    "Address": "127.0.0.1",
    "Port": 8000
  },
  "Check": {
    "Node": "foobar",
    "CheckID": "service:redis1",
    "Name": "Redis health check",
    "Notes": "Script based health check",
    "Status": "passing",
    "ServiceID": "redis1",
    "Definition": {
      "TCP": "localhost:8888",
      "Interval": "5s",
      "Timeout": "1s",
      "DeregisterCriticalServiceAfter": "30s"
    }
  },
  "SkipNodeUpdate": false
}'
```
执行完上述命令后，重新查看节点目录，发现多了一个`foobar`：
```
curl localhost:8500/v1/catalog/nodes
```
查看具体某节点的服务目录：
```
curl localhost:8500/v1/catalog/node/foobar
```

### 查询服务
当consul代理启动和服务被同步后，我们就可以使用DNS或HTTP API来查询服务。

#### DNS API
让我们首先使用DNS API查询服务。对于DNS API，服务的DNS名称是`NAME.service.consul`。默认情况下，所有DNS名称总是在`consul`命名空间中，但这是可以配置的。`service`子域名告诉consul我们正在查询服务，而`NAME`是服务的名称。

对于我们注册的web服务，这些惯例和设置产生了一个全限定域名`web.service.consul`：
```
$ dig @127.0.0.1 -p 8600 web.service.consul
...

;; QUESTION SECTION:
;web.service.consul.        IN  A

;; ANSWER SECTION:
web.service.consul. 0   IN  A   172.20.20.11
```
就像你看到的，返回了一个A记录，里面是服务所在节点的IP地址。A记录只能包含IP地址。

你也可以使用DNS API以一个SRV记录的方式获取一对完整的地址和端口：
```
$ dig @127.0.0.1 -p 8600 web.service.consul SRV
...

;; QUESTION SECTION:
;web.service.consul.        IN  SRV

;; ANSWER SECTION:
web.service.consul. 0   IN  SRV 1 1 80 vagrant.node.dc1.consul.

;; ADDITIONAL SECTION:
vagrant.node.dc1.consul. 0      IN      A       127.0.0.1
vagrant.node.dc1.consul. 0      IN      TXT     "consul-network-segment="
```
SRV记录告诉我们，web服务运行在端口80，运行的节点是`vagrant.node.dc1.consul.`。DNS还通过附加部分返回了这个节点的A记录。

最后，我们还可以使用DNS API来通过标签过滤服务。基于标签的服务查询的格式是`TAG.NAME.service.consul`。在下面的例子中，我们向Consul查询所有"rails"标签的服务。我们得到了一个成功的响应，因为之前注册服务时指定了这个标签：
```
$ dig @127.0.0.1 -p 8600 rails.web.service.consul
...

;; QUESTION SECTION:
;rails.web.service.consul.      IN  A

;; ANSWER SECTION:
rails.web.service.consul.   0   IN  A   172.20.20.11
```
#### HTTP API
除了使用DNS API，还可以使用HTTP API来查询服务：
```
$ curl http://localhost:8500/v1/catalog/service/web
[
    {
        "ID": "0755b91c-a3d9-bb41-a163-ce5c88d86af7",
        "Node": "vagrant",
        "Address": "127.0.0.1",
        "Datacenter": "dc1",
        "TaggedAddresses": {
            "lan": "127.0.0.1",
            "wan": "127.0.0.1"
        },
        "NodeMeta": {
            "consul-network-segment": ""
        },
        "ServiceID": "web",
        "ServiceName": "web",
        "ServiceTags": [
            "rails"
        ],
        "ServiceAddress": "",
        "ServicePort": 80,
        "ServiceEnableTagOverride": false,
        "CreateIndex": 6,
        "ModifyIndex": 6
    }
]
```
catalog API给出了托管一个指定服务的所有节点。

### 健康检查
consul支持对服务进行健康检查。consul支持的健康检查方法有5种：

- 脚本+时间间隔

- HTTP +时间间隔

- TCP +时间间隔

- 生存时间（TTL）

- Docker + Interval

一个含有健康检查的consul服务定义文件`/etc/consul.d/playground.json`：
```
{
  "service": {
    "name": "playground",
    "node": "vagrant",
    "tags": [
      "urlprefix-/playground"
    ],
    "address": "localhost",
    "port": 8080,
    "check": {
      "id": "api",
      "name": "playground health check",
      "http": "http://localhost:8080/config.json",
      "method": "GET",
      "interval": "10s",
      "timeout": "2s"
    }
  }
}
```

