分别搜索关键字：virtualbox、vagrant、git

 - virtualbox：虚拟机
 - vagrant：虚拟机管理（vagrantup.com）
 - git：终端（git-scm.com）

1.执行vagrant init获得一个基础的Vagrantfile  
2.将base换成ubuntu/xenial64
```
config.vm.box = "ubuntu/xenial64"
```  
3.多虚拟机的[Vagrantfile](https://github.com/wbwangk/wbwangk.github.io/wiki/Vagrantfile)

将Vagrantfile修改为：
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
   end
end
```
4.进入webb1  
```
$vagrant ssh webb1  
```
查看/etc/hosts、/etc/hostname

5.通过vbox增加硬盘  
   查看设备
(待补充)