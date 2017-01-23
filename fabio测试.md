
### consul安装
Consul是一个在国外流行的服务发现和配置共享的服务软件。fabio要依赖consul，或者说fabio增强了consul，使注册在consul的服务有了统一路由服务。  
这是[安装地址](https://www.consul.io/downloads.html)(含UI下载地址)。  
```
$ wget https://releases.hashicorp.com/consul/0.7.2/consul_0.7.2_linux_amd64.zip
$ unzip consul_0.7.2_linux_amd64.zip
$ mv consul /usr/local/bin            （把consul移动到可执行目录下）
$ consul agent -dev    (-dev表示开发模式)
```
屏幕显示一些日志，表示consul安装并启动。验证consul，需要进入另外一个终端（可参考[这个](https://github.com/wbwangk/wbwangk.github.io/wiki/virtualbox-vagrant-gitbash%E5%85%A5%E9%97%A8)），ssh登录到刚启动了consul的这台机器，执行：
```
$ consul members
$ curl localhost:8500/v1/catalog/nodes     （8500是consul的HTTP API端口）
```
会显示当前节点信息。如果上述命令正确执行就表示consul已经安装成功。上述命令行和HTTP API的功能相同。

### 样例服务安装

设置golang路径：
```
$ export GOPATH=/go
$ go get github.com/magiconair/fabio-example
```
要等一会儿到/go目录下去看，发现/go/bin目录下多了一个fabio-example的可执行程序。
（go语言的安装参考golangk.org）
```
$ ./fabio-example --help
Usage of ./fabio-example:
  -addr string
        host:port of the service (default "127.0.0.1:5000")
  -name string
        name of the service (default "fabio-example")
  -prefix string
        comma-sep list of host/path prefixes to register
$ ./fabio-example -name svc-a -prefix /foo
```
调用consul的HTTP API获取服务svc-a的元数据：
```
$ curl http://localhost:8500/v1/catalog/service/svc-a
[
    {
        "Node": "block3",
        "Address": "127.0.0.1",
        "TaggedAddresses": {
            "lan": "127.0.0.1",
            "wan": "127.0.0.1"
        },
        "ServiceID": "svc-a-127.0.0.1:5000",
        "ServiceName": "svc-a",
        "ServiceTags": [
            "urlprefix-/foo"
        ],
        "ServiceAddress": "127.0.0.1",
        "ServicePort": 5000,
        "ServiceEnableTagOverride": false,
        "CreateIndex": 7,
        "ModifyIndex": 8
    }
]
```
需要注意的是svc-a服务的tag：```urlprefix-/foo```，这个tag符合fabio的路由定义约定，会被当作fabio的路由定义。安装fabio后再测试上述路由定义约定。
### fabio安装
安装地址是：https://github.com/eBay/fabio/releases
```
$ wget https://github.com/eBay/fabio/releases/download/v1.3.7/fabio-1.3.7-go1.7.4-linux_amd64
$ chmod +x fabio-1.3.7-go1.7.4-linux_amd64
$ ln -s fabio-1.3.7-go1.7.4-linux_amd64 /usr/local/bin/fabio
```
用chmod将下载的程序修改为可执行，然后在可执行目录/usr/local/bin下定义了一个软符号链接fabio。用默认值启动fabio：
```
$ fabio
```
会提示consul: connection to "localhost:8500" in datacenter "dc1"。这表明fabio已经找到了本地的consul服务。
### 测试fabio路由
现在consul、fabio-excample、fabio各占了一个终端，现在打开第4个终端，执行：
```
$ curl localhost:9999/foo
Serving /foo from svc-a on 127.0.0.1:5000
```
9999端口是fabio的代理(proxy)端口。