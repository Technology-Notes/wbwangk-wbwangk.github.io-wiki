### vagrant虚机之间可以ssh
使用vagrant up启动的虚机，虽然可以用vagrant ssh登录虚拟，但进入虚拟操作系统后，却不可以用类似：
```
$ ssh root@<另一个虚机ID>
```
来登录。可能是ubuntu默认的ssh不支持root的远程登录。这对于习惯使用root用户的人很不方便。为了让虚机之间可以ssh，可以这样做，首先用vagrant ssh登录虚机，然后：
```
$ sudo su -     (切换为root用户)
$ passwd root    (为root用户创建口令)
$ vi /etc/ssh/sshd_config     （改变ssh的默认设置PermitRootLogin为yes）
  PermitRootLogin yes
$ service ssh restart   (重启ssh服务)
$ ssh root@localhost   （测试一下root ssh）
```
假定有两个vagrant创建的虚机big1和big2，通过上述操作两台VM之间可以用类似：
```
$ ssh root@big1
```
互相登录。
### 无密码ssh

假定big1已经可以用密码方式登录big2，则在big1中执行：
```
# ssh-keygen   (敲几次回车)
# cd ~/.ssh  && ls
id_rsa  id_rsa.pub  known_hosts
```
发现产生了两个密码文件，其中id_rsa.pub是公钥。现在利用命令把公钥复制到big2：
```
# ssh-copy-id root@big2         (会提示输入big2的root口令)
Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@big2'"
and check to make sure that only the key(s) you wanted were added.
# ssh root@big2   (发现不再提示密码了)
root@big2:~#   （这个提示表示已经进入了big2）
```