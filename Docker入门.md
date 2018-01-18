## vagrant启动linux VM
vagrant的入门参考《[virtualbox vagrant gitbash入门](virtualbox vagrant gitbash入门)》。  
新建一个docker文件夹，在文件夹下创建一个名为Vagrantfile的文本文件：
```
Vagrant.configure("2") do |config|
        config.vm.box = "ubuntu/xenial64"
        config.vm.define "node0" do |node0|
                node0.vm.network "private_network", ip: "192.168.1.110"
                node0.vm.hostname = "node0"
        end
        config.vm.define "node1" do |node1|
                node1.vm.network "private_network", ip: "192.168.1.111"
                node1.vm.hostname = "node1"
        end
end
```
在docker文件夹下利用git bash启动虚拟机：
```
$ vagrant up docker0 docker1
$ vagrant ssh docker0
```
## 安装docker
**使用ubuntu默认源安装**：
```
$ apt-get install docker.io
$ docker -v  （查看docker引擎版本）
$ apt-get remove docker.io  (卸载docker)
```
这种方式方便，但安装的docker版本略旧。

**使用docker官方源安装**：
```
$ sudo apt-get install -y --no-install-recommends \
    apt-transport-https  ca-certificates \
    curl software-properties-common
$ curl -fsSL https://apt.dockerproject.org/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb https://apt.dockerproject.org/repo/ \
       ubuntu-$(lsb_release -cs) main"
$ sudo apt-get update
$ sudo apt-get -y install docker-engine
$ docker -v  (查看docker引擎版本)
$ systemctl status docker  或 service docker status
```
这种方式安装的docker版本是最新的，甚至可以选装测试版。docker官方文档地址：[docs.docker.com](https://docs.docker.com)(有时不翻墙很慢)。
## 运行docker容器
两种容器：短任务和长任务。
####短任务
```
$ docker run hello-world  (也可先执行docker pull hello-world)
$ docker ps -a   (-a表示全部容器,包括停止的)
$ docker logs <容器id>
$ docker run -d -e HAHA='wbwang' busybox echo $HAHA   (-d 表示后台执行, 镜像id+空格之后是容器启动后执行的命令和参数)
$ docker run -it busybox /bin/sh   (-it表示开启标准输入输出)
$ docker stop <容器id> && docker rm <容器id>
```
#### 长任务
长任务往往执行网络服务：
```
$ docker run -d -p 9000:9000 -v "/var/run/docker.sock:/var/run/docker.sock" portainer/portainer
$ docker run -d -p 8080:8080 swaggerapi/swagger-ui
$ docker exec -it <container id> /bin/sh
```
查看容器日志文件路径：
```
docker inspect --format='{{.LogPath}}' containername
```
## docker构建
可以本地构建，也可以使用Docker Hub的自动构建。本地构建，可以使用命令行构建，更多的使用Dockerfile构建。
####利用命令行构建镜像
```
$ docker run -d -p 80:80 nginx
$ curl localhost:80
$ docker exec -it <nginx的容器id> /bin/sh
# apt-get update && apt-get install vim  (然后用vim修改一下/usr/share/nginx/html/index.html)
# exit
$ docker commit <nginx的容器id> nginx:test
$ docker run -d -p 81:80 nginx:test
$ curl localhost:81
```
####利用Dockerfile构建镜像
新建一个目录，在目录下创建一个叫Dockerfile的文本文件：
```
FROM nginx
COPY index.html /usr/share/nginx/html/index.html
```
创建一个index.html:
```
<!DOCTYPE html>
<html>
<body>
<h1>======= Welcome to nginx!  =======</h1>
<p>haha</p>
</body>
</html>
```
执行构建(命令最后有个点)：
```
$ docker build -t nginx:2 .
$ docker run -d -p 82:80 nginx:2
```
####镜像push到docker hub
```
$ docker tag nginx:2 wbwang/nginx:2
$ docker images       (查看镜像列表)
$ docker login -u wbwang    (登录Docker Hub)
$ docker push wbwang/nginx:2
$ docker run -d -p 83:80 wbwang/nginx:2
```
####利用dockerhub自动构建

 1. 在github上建个库，库的根目录下建个叫Dockerfile的文件
 2. 登录hub.docker.com，在create下拉菜单中选择Create Automated Build，弹出页点选github
 3. 选择刚创建的库，保存
 4. 在Build Settings选项卡中点击Trigger按钮
 5. 在Build Details选项卡中等待构建成功

#### docker端口
[1](https://docs.docker.com/engine/swarm/networking/)  
Port 7946 TCP/UDP for container network discovery.  
Port 4789 UDP for the container overlay network.  
[2](https://github.com/docker/docker/blob/master/docs/swarm/swarm-tutorial/index.md)  
TCP port 2377 for cluster management communications  
TCP and UDP port 7946 for communication among nodes  
TCP and UDP port 4789 for overlay network traffic  
[3](https://github.com/docker/docker/blob/master/docs/swarm/ingress.md)  
Port 7946 TCP/UDP for container network discovery.  
Port 4789 UDP for the container ingress network.

#### 远程访问docker引擎
[官方文档](https://success.docker.com/article/Using_systemd_to_control_the_Docker_daemon)  [一个博文](http://blog.csdn.net/csde12/article/details/70240721)  
下面的配置方式仅针对ubuntu16，其他版本慎用。  

处于安全的原因，docker引擎只能被本机访问。如果想远程访问docker引擎，需要修改服务的启动参数。首先查看一下当前的启动参数：
```
sudo systemctl cat docker | grep ExecStart
ExecStart=/usr/bin/docker daemon -H fd://
```
修改配置：
```
sudo systemctl edit docker
```
然后输入以下的内容：
```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H fd://
```
ctrl+O是保存，然后按回车创建一个文件。

重启docker服务：
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### 将docker日志写入syslog
参考[docker相关官方文档](https://docs.docker.com/engine/admin/logging/overview/#configure-the-default-logging-driver)  
ubuntu16内置了rsyslog。在ubuntu下（其他linux貌似相同）要将docker日志写入syslog，需要在目录`/etc/docker`创建一个叫`daemon.json`的文件：
```
{
  "log-driver": "syslog"
}
```
重启docker服务后，去看syslog：
```
tail /var/log/syslog -n 100
```

#### docker清理
参考[这个文档](https://hub.docker.com/r/martin/docker-cleanup-volumes/)  
