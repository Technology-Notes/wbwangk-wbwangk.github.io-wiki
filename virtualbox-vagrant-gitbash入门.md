分别搜索关键字：virtualbox、vagrant、git

 - virtualbox：虚拟机
 - vagrant：虚拟机管理（vagrantup.com）
 - git：终端（git-scm.com）
0.安装上述3个软件。
1.建个目录(如C:/U)，在鼠标右键菜单中选择“Git Bash”。然后执行vagrant init，发现目录下新建了一个Vagrantfile文件。
2.修改Vagrantfile文件将base换成ubuntu/xenial64
```
config.vm.box = "ubuntu/xenial64"
```  
执行（如果是第一次执行vagrant up，由于需要从网上下载镜像会很慢）：
```
$ vagrant up (启动虚拟机）
$ vagrant ssh （进入虚拟机）
```
3.多虚拟机的配置文件参考[Vagrantfile](https://github.com/wbwangk/wbwangk.github.io/wiki/Vagrantfile)

可以手工将Vagrantfile修改为：
```
Vagrant.configure("2") do |config|
   config.vm.define :"wang1" do |os|
      os.vm.box = "ubuntu/xenial64"
      os.vm.network :private_network, ip: "192.168.1.130"
      os.vm.hostname = "wang1"
      os.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--memory", "1024"]
          v.name = "wang1"
          v.gui = false
      end
   end
   config.vm.define :"wang2" do |os|
      os.vm.box = "ubuntu/xenial64"
      os.vm.network :private_network, ip: "192.168.1.131"
      os.vm.hostname = "wang2"
      os.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--memory", "1024"]
          v.name = "wang2"
          v.gui = false
      end
   end
end
```
上面的这个配置文件支持两个虚机，而且设定了两个虚机的ip、主机名、内存大小等。
4.进入webb1  
```
$ vagrant up wang1 wang2 （启动两个虚机）
$ vagrant ssh wang1  （登录wang1）
```
退出虚拟机执行```$ exit```
其他的vagrant命令还有：
```
$ vagrant halt wang1   (wang1关机)
$ vagrant destroy wang1 (将wang1虚拟机删除）
```

当虚机上安装了很多软件，想打包自己的虚机镜像，则：
```
$ vagrant package wang1 --output webb-wang1
```
webb-wang1是输出的镜像文件名。