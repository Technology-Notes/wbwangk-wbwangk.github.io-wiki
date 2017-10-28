登录github，我的账号是wbwangk。
在wbwangk账号下创建新库git-webb，用于测试。
### 认证
在本地linux下生成ssh-key：
```
$ ssh-keygen -t rsa -C "javacup00@163"
```
需要按三次回车。后两次回车也可以输入密码。
查看生成的公钥：
```
$ cat ~/.ssh/id_rsa.pub
```
在github的头像下拉菜单中选“Settings/SSH and GPG Keys”。再点“new SSH key”按钮。title随便，将刚才cat出来的公钥内容粘贴到下面的输入框中。
### git基本功能测试
```
$ mkdir /opt/git-webb && cd /opt/git-webb
$ git init
$ git config --global user.email "javacup00@163.com"    （这两个git config如果不执行下面的git remote add会报错）
$ git config --global user.name "wbwangk"
$ git remote add origin git@github.com:wbwangk/git-webb.git  (上面新建的库git-webb在本地添加为orgin，象别名或简称)
$ git pull origin master
$ touch t.txt   (创建一个叫t.txt的空文件)
$ git add t.txt
$ git commit -m "add t.txt"   （提交到本地库）
$ git push -u orgin master     （推送到远程库）
```
可以用浏览器到github.com/wbwangk/git-webb库下查看，发现增加了一个t.txt文件。

克隆指定分支：
```
$ git clone -b branch-2.6 https://github.com/apache/ambari.git
```
### git与基础认证
gitlib与AD进行了集成，当对于私有库进行克隆时提示输入用户名口令，如：
```
git clone http://open.inspur.com/imaidata/docs.git
正克隆到 'docs'...
Username for 'http://open.inspur.com': wbwang
Password for 'http://wbwang@open.inspur.com':
remote: Counting objects: 32, done.
remote: Compressing objects: 100% (20/20), done.
remote: Total 32 (delta 12), reused 30 (delta 11)
Unpacking objects: 100% (32/32), done.
```
如果需要填入自动用户名口令，则编辑`~/.netrc`，输入下列内容：
```
machine <git host>
       login <user>
       password <password>
```
如：
```
machine open.inspur.com
       login wbwang
       password <password>
```
然后再执行克隆命令就不再提示输入用户名口令了。