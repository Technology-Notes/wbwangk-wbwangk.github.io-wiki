### consul安装
Consul是一个在国外流行的服务发现和配置共享的服务软件。fabio要依赖consul，或者说fabio增强了consul，使注册在consul的服务有了统一路由服务。  
这是[安装地址](https://www.consul.io/downloads.html)(含UI下载地址)。  
先下载这个[二进制包](https://releases.hashicorp.com/consul/0.7.2/consul_0.7.2_linux_amd64.zip)，解压：
```
$ unzip consul_0.7.2_linux_amd64.zip
$ mv consul /usr/local/bin            （把consul移动到可执行目录下）
$ consul agent -dev    (-dev表示开发模式)
```
屏幕显示一些日志，表示consul安装并启动。验证consul，需要进入另外一个终端（可参考[这个](https://github.com/wbwangk/wbwangk.github.io/wiki/virtualbox-vagrant-gitbash%E5%85%A5%E9%97%A8)），ssh登录到刚启动了consul的这台机器，执行：
```
$ consul members
$ curl localhost:8500/v1/catalog/nodes
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