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
使用ubuntu默认源安装
```
$ apt install docker.io
```
使用docker官方源安装
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