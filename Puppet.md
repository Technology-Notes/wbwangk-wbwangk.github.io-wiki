[ibm developerworks的puppet入门](https://www.ibm.com/developerworks/cn/opensource/os-cn-puppet/index.html)  
[Puppet官方手册](https://docs.puppet.com/puppetserver/5.0/install_from_packages.html#red-hat-enterprise-linux)  
本文环境是ubuntu14.4(/e/vagrant11/ambari-vagrant/ubuntu14.4)。环境有两个虚拟机：
```
u1401.ambari.apache.org                   (充当puppet server)
u1402.ambari.apache.org                   (充当puppet agent)
```
## Puppet安装
### server安装
在节点u1401上添加源、安装server、启动服务：
```
$ wget https://apt.puppetlabs.com/puppetlabs-release-pc1-trusty.deb
$ sudo dpkg -i puppetlabs-release-pc1-trusty.deb
$ sudo apt-get update
$ sudo apt-get install puppetserver
$ sudo service puppetserver start                            (原文有误)
```
### agent安装
在节点u1402上添加源、安装agent：
```
$ wget https://apt.puppetlabs.com/puppetlabs-release-pc1-trusty.deb
$ sudo dpkg -i puppetlabs-release-pc1-trusty.deb
$ sudo apt-get update
$ sudo apt-get install puppet-agent 
```
编辑配置文件/etc/puppetlabs/puppet/puppet.conf，设置该agent的puppet server的地址：
```
[main]
server = u1401.ambari.apache.org
```
### agent启动
在u1402上执行测试：
```
$  /opt/puppetlabs/bin/puppet agent --test
```
提示找不到证书。(网上查到的[解决方案](https://www.slideshare.net/AshwinPawar/puppet-agent))  
猜测是客户端的CSR(证书签名请求)没有被签署。到u1401上执行：
```
$ /opt/puppetlabs/puppet/bin/puppet  cert sign u1402.ambari.apache.org
Signing Certificate Request for:
  "u1402.ambari.apache.org" (SHA256) E6:3F:DA:8E:D7:26:B7:6B:2D:D5:9F:3C:B0:D8:5E:AB:04:C9:51:11:B3:02:0F:3F:BE:9E:9F:FA:02:94:18:3C
Notice: Signed certificate request for u1402.ambari.apache.org
Notice: Removing file Puppet::SSL::CertificateRequest u1402.ambari.apache.org at '/etc/puppetlabs/puppet/ssl/ca/requests/u1402.ambari.apache.org.pem'
```
完成上面的步骤后，服务器对u1402的CSR进行了签署。(原文中server和agent是同一台机器，所以不需要签署证书)。再回到u1402上执行：
```
$ /opt/puppetlabs/bin/puppet agent --test
Info: Creating a new SSL key for u1402.ambari.apache.org
Info: Caching certificate for ca
Info: csr_attributes file loading from /etc/puppetlabs/puppet/csr_attributes.yaml
Info: Creating a new SSL certificate request for u1402.ambari.apache.org
Info: Certificate Request fingerprint (SHA256): E6:3F:DA:8E:D7:26:B7:6B:2D:D5:9F:3C:B0:D8:5E:AB:04:C9:51:11:B3:02:0F:3F:BE:9E:9F:FA:02:94:18:3C
Info: Caching certificate for ca
Exiting; no certificate found and waitforcert is disabled
```

## 编写第一个配置文件
### 第一个HelloWorld配置文件
在u1401(puppet server)的`/etc/`文件夹下面创建一个文件`helloworld.txt`，文件的内容是`”hello world from puppet!\n”`。
然后再u1401上进入`/etc/puppetlabs/code/environments/production/manifests`文件夹，创建`site.pp`文件：
```
node u1402.ambaria.apache.org {
file { 'helloworld':
    path => '/etc/helloworld.txt',
    owner  => 'root',
    group  => 'root',
    mode   => '655',
    content => "hello world from puppet!\n",
    }
}
```
`site.pp`就是节点的配置文件，里面可以包含对各个节点的配置描述。在实例配置文件中，`u1402.ambaria.apache.org`就是节点的主机名。包含在 `u1402.ambaria.apache.org`中的配置描述就是该节点的资源集合的描述。
配置文件创建好后，agent节点会周期性地查询 PuppetServer 来获取自己的配置文件并在本地应用。当然 Puppet 也支持手动获取自己的配置。在本例中，我们通过手动的方式来进行配置更新。我们在 u1402(PuppetAgent)上手动执行命令：
```
$ /opt/puppetlabs/bin/puppet agent --test
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Caching catalog for u1402.ambari.apache.org
Info: Applying configuration version '1503317552'
Notice: /Stage[main]/Main/Node[u1402.ambari.apache.org]/File[helloworld]/ensure: defined content as '{md5}c3aa68786c58c94ef6f3e2399920f268'
Notice: Applied catalog in 0.02 seconds
$ cat /etc/helloworld.txt 
hello world from puppet!
```
我们看到节点成功从Puppet Server获取配置文件，并且在本地应用，对应的文件成功创建。

### 进阶：执行脚本任务
作为进阶的任务，我们希望节点可以执行一些更加复杂一点的任务。我们希望节点可以从 PuppetServer 获取一个命令脚本，并且执行该脚本。
我们首先在u1401的`/etc/puppetlabs/code/environments/production/modules`中创建一个名叫”test”的模块，在`test`模块下面创建一个”files”文件夹。在这个文件夹里的文件是可以被节点获取的。然后我们在这个”files”文件夹里创建一个shell脚本`test.sh`(/etc/puppetlabs/code/environments/production/modules/test/files/test.sh):
```
touch /etc/helloworld.log
echo "helloworld" >> /etc/helloworld.log
```
该脚本会在`/etc/`目录下创建`helloworld.log`文件，然后在文件里添加”hello world”内容。  
仍是在u1401上，进入目录`/etc/puppetlabs/code/environments/production/manifests`，然后我们再来编辑 site.pp 文件：
```
node u1402.ambari.apache.org {
file { 'test.sh':
  path => '/etc/test.sh',
  owner  => 'root',
  group  => 'root',
  mode   => '655',
  source => 'puppet:///modules/test/test.sh',
  }
exec { 'execute ':
  command => 'bash /etc/test.sh',
  require => File['test.sh'],
  path => ["/bin/"],
  }
}
```
其中，我们定义了两个资源：一个文件资源和一个执行命令资源。同时这两个资源有依赖关系，命令执行资源依赖于文件资源，所以Puppet会优先处理文件资源。执行命令资源会在文件资源存在后再执行。
我们看下客户端的执行结果：
```
$ /opt/puppetlabs/bin/puppet agent --test
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Caching catalog for u1402.ambari.apache.org
Info: Applying configuration version '1503318737'
Notice: /Stage[main]/Main/Node[u1402.ambari.apache.org]/File[test.sh]/ensure: defined content as '{md5}4a7fef6d906b79559966eef350f380a6'
Notice: /Stage[main]/Main/Node[u1402.ambari.apache.org]/Exec[execute ]/returns: executed successfully
Notice: Applied catalog in 0.09 seconds
$ cat /etc/helloworld.log 
helloworld
```
我们可以看到，helloworld.log 文件被正确的创建，说明脚本文件被正确地执行。

## 结束语
Puppet 是基于 Ruby 的开源系统配置和管理工具，它提供的独特的系统配置语言极大程度地简化了系统管理员管理和配置系统的过程。本文首先介绍了 Puppet 的系统架构和工作流程，并且介绍了 Puppet 独特的系统配置语言，之后我们简单介绍了安装和配置 Puppet 的具体步骤。最后，本文以两个实例介绍了如何在 Puppet 中为节点编写配置文件，来达到创建文件和执行命令的效果。希望本文能对系统管理员，Puppet 初学者有所帮助。