安装三个软件：
 - virtualbox：虚拟机  
 - vagrant：虚拟机管理（[vagrantup.com]）  
 - git：终端（[git-scm.com]）  

### vagrant入门　
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
$ vagrant reload wang1 (重新加载配置文件，有时可以解决一些未知问题）
```

当虚机上安装了很多软件，想打包自己的虚机镜像，则：
```
$ vagrant package wang1 --output webb-wang1
```
webb-wang1是输出的镜像文件名。  
在Vagrantfile中直接引用镜像文件名来定义新的虚拟机：
```
   config.vm.define :"wang2" do |os|
         os.vm.box = "webb-wang1"
         (略)   
```
### git bash入门
打通了windows和linux，可以在windows文件系统下使用linux命令。  
可以将文件在windows和linux之间用scp命令。为了演示scp先建个用户：
```
$ useradd -d /usr/webb -m webb
$ passwd webb 为webb用户创建密码
```
在windows利用鼠标右键菜单进入git bash：
```
$ echo "12345" > t.tt
$ scp t.tt webb@10.10.11.86:/usr/webb/    （按提示输入webb用户的密码）
```
上述命令会把windows当前目录下新建的t.tt文件远程复制到linux的/usr/webb/目录下。  
如果把linux下的文件复制到windows下则：
```
$ scp webb@10.10.11.86:/usr/webb/t.tt .   （t.tt后面跟空格和点）
```
### 常用linux命令
```
$ sudo su - root 切换用户到root
$ rm -rf <dir name>   删除目录及下属
$ alias ll='ls -l' 定义别名
$ export PATH=$PATH:/usr/webb    定义环境变量
$ echo {\"path\": \"$PATH\"} | jq
$ chmod +x run.sh 或 chmod 777 run.sh    修改文件权限
$ netstat -an | grep 8080 查看谁占用了8080端口

```
```~```表示用户的home位置，如```cd ~```  
```$PWD```表示当前目录，如```echo $PWD```    
```.``` 当前目录，但有时```. run.sh```表示执行脚本文件run.sh。与```./xxx.sh```类似，有时让人困扰。  
```|```管道符。如```echo '{"name": "webb"}' | jq```  
```\\```转义符，除了前面的echo转义引号外，还用于把长命令拆成多行（忽略回车）。如：
```
$ curl -X POST http://127.0.0.1:5984/demo \
            -d '{"company": "Example, Inc."}'
```

### 解压命令
 1. 对于.tar结尾的文件 　　tar -xf all.tar  
 2. 对于.gz结尾的文件　　gzip -d all.gz 　　gunzip all.gz   
 3. 对于.tgz或.tar.gz结尾的文件 　　tar -xzf all.tar.gz 　　tar -xzf all.tgz   
 4. 对于.bz2结尾的文件 　　bzip2 -d all.bz2 　　bunzip2 all.bz2   
 5. 对于tar.bz2结尾的文件 　　tar -xjf all.tar.bz2   
 6. 对于.Z结尾的文件 　　uncompress all.Z  
 7. 对于.tar.Z结尾的文件 　　tar -xZf all.tar.z  
 8. 对于.zip结尾文件，apt-get install unzip，然后unzip xxx.zip  

### 进程命令
```
$ ps -ef | grep docker   查看docker服务的进程号
```
ps的参数：  
-a ：all，表示列出所有的连接，服务监听，Socket资料  
-t ：tcp，列出tcp协议的服务  
-u ：udp，列出udp协议的服务  
-n ：port number， 用端口号来显示  
-l ：listening，列出当前监听服务  
-p ：program，列出服务程序的PID  