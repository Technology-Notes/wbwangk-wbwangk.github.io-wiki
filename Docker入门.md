### vagrant启动linux VM
Vagrantfile：
```
Vagrant.configure("2") do |config|
   config.vm.define :"docker0" do |os|
      config.vm.box = "ubuntu/xenial64"
      config.vm.network "private_network", ip: "192.168.1.160"
      os.vm.hostname = "docker0"
      os.vm.synced_folder ".", "/vagrant", disabled: true
      os.vm.provider "virtualbox" do |v|
         v.customize ["modifyvm", :id, "--memory", "1024"]
         v.name = "docker0"
         v.gui = false
      end
   end
   config.vm.define :"docker1" do |os|
      config.vm.box = "ubuntu/xenial64"
      config.vm.network "private_network", ip: "192.168.1.161"
      os.vm.hostname = "docker1"
      os.vm.synced_folder ".", "/vagrant", disabled: true
      os.vm.provider "virtualbox" do |v|
         v.customize ["modifyvm", :id, "--memory", "1024"]
         v.name = "docker1"
         v.gui = false
      end
   end
end
```
启动虚拟机：
```
$ vagrant up docker0 docker1
$ vagrant ssh docker0
```
### 安装docker
使用ubuntu默认源安装：
```
$ apt install docker.io
```
使用docker官方源安装：
```
$ sudo apt-get install -y --no-install-recommends \
    apt-transport-https  ca-certificates \
    curl software-properties-common
$ curl -fsSL https://apt.dockerproject.org/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb https://apt.dockerproject.org/repo/ \
       ubuntu-$(lsb_release -cs) main"
$ sudo apt-get update
$ sudo apt-get -y install docker-engine
$ systemctl status docker  或 service docker status
```
### docker run
两种容器：短任务和长任务。短任务：
```
$ docker run hello-world  (也可先执行docker pull hello-world)
$ docker ps -a   (-a表示全部容器,包括停止的)
$ docker logs <hello-world container id>
$ docker run -d -e HAHA='wbwang' busybox echo $HAHA   (-d 表示后台执行, 镜像id+空格之后是容器启动后执行的命令和参数)
$ docker run -it busybox /bin/sh   (-it表示开启标准输入输出)
```
长任务往往执行网络服务：
```
$ docker run -d -p 9000:9000 -v "/var/run/docker.sock:/var/run/docker.sock" portainer/portainer
$ docker run -d -p 8080:8080 swaggerapi/swagger-ui
```
查看容器日志文件路径：
```
docker inspect --format='{{.LogPath}}' containername
```
运行一个容器
浏览docker hub
构建镜像并运行
docker hub自动构建镜像
push镜像到docker hub