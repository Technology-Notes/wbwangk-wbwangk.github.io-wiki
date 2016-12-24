两台虚拟机：192.168.1.141, 192.168.1.142。
141主机名block1，充当集群管理节点；142主机名是block2，充当工作节点。

使用justniffer检测block1的enp0s8网卡（即IP192.168.1.141所在的网卡）：
```
# justniffer -i enp0s8 -r -p "port 2375"
```
2375端口是dockerd（docker守护进程）监听的端口。
首先在block2上执行一个docker ps命令来测试一下justniffer：
```
# docker -H tcp://192.168.1.141:2375 ps
```
justniffer在屏幕上显示：
```
# justniffer -i enp0s8 -r -p "port 2375"
GET /v1.24/containers/json HTTP/1.1
Host: 192.168.1.141:2375
User-Agent: Docker-Client/1.12.5 (linux)
Accept-Encoding: gzip

HTTP/1.1 200 OK
Content-Type: application/json
Server: Docker/1.12.5 (linux)
Date: Sat, 24 Dec 2016 03:08:55 GMT
Content-Length: 3

[]
```
由于block上没有正在运行的容器，所以返回的json串只是一个“[]”。如果执行的命令加上-a参数，则会看到一个很乱的json串。