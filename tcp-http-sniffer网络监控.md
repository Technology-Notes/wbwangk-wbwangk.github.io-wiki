工作的原因，经常需要跟踪tcp/http请求的详细信息，如http头中内容。纯从nginx的访问日志中获取的信息有限。后来发现有tcp嗅探(sniffer)软件可以用。  
就网络监控(嗅探)来说，tcpdump最流行，但不好掌握。最终选择了[httpry](https://github.com/jbittel/httpry)和[justniffer](http://justniffer.sourceforge.net/)。两者貌似都师从tcpdump，参数都有点像。
在unbuntu16下两个软件的安装都容易：
```
# apt-get install httpry
# add-apt-repository ppa:oreste-notelli/ppa 
$ apt-get update
$ apt-get install justniffer
```
两个软件都监控网卡的tcp流。

### httpry
首先要确定要监控的网卡，可以使用ip addr命令：
```
# ip addr
```
根据命令的显示，找到网卡的id是enp0s8。启动httpry命令：
```
# httpry -i enp0s8
```
然后所有通过网卡enp0s8的http请求和响应信息就都输出屏幕上了。httpry的缺点是不能显示http协议头中内容。

### justniffer
同httpry一样，先用ip addr命令找到要监控的网卡，如enp0s8。然后执行：
```
# justniffer -i enp0s8 -r
```
-r参数会使http请求和响应的完整信息输出到屏幕上，包括http头的信息。这就是justniffer比httpry强的地方。