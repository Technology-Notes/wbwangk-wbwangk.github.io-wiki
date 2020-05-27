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
$ git clone --depth 1 -b branch-2.6 https://github.com/apache/ambari.git
```
### git与基础认证
gitlib与AD进行了集成，当对于私有库进行克隆时提示输入用户名口令，如：
```
git clone --depth 1 http://open.inspur.com/imaidata/docs.git
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

## git入门
[原文](https://git-scm.com/book/zh/v2/)  
git工作区中文件可处于两种状态：tracked 或 untracked。跟踪状态又可细分为：unmodified, modified, or staged(暂存)。  
![](https://git-scm.com/book/en/v2/images/lifecycle.png)  

git add导致文件被暂存。（暂存后的文件再修改就又有了一个modified版本，同一个文件的两个版本出现在两个状态中）
git pull 通常会从最初克隆的服务器上抓取数据并自动尝试合并到当前所在的分支。  
git checkout testing 切换分支
git checkout -b testing 创建并切换到分支

## mkdocs
[mkdocs中文文档](http://markdown-docs-zh.readthedocs.io/zh_CN/latest/)  
mkdocs是一个文档构建工具，可以将markdown文章转换为html。然后可以将html发布到github.io，也就成为一个在线文档系统。

### 安装mkdocs
本地环境是ubuntu16，已经预装了python2.7，但需要自己装pip。  
pip的安装参考[这个文档](http://pip.readthedocs.io/en/latest/installing/)。  
用pip安装mkdocs并安装必要依赖：
```
pip install mkdocs  
pip install python-markdown-math
```

### 文档编辑和构建
`https://github.com/HyperledgerCN/hyperledgerDocs`是Hyperledger Fabric的中文文档库，以这个为范例描述一下用mkdocs建立在线文档的过程。  
#### fork到自己的库
将`https://github.com/HyperledgerCN/hyperledgerDocs`分叉(fork)到自己的库:`https://github.com/wbwangk/hyperledgerDocs`
#### clone到本地
```
git clone --depth 1 https://github.com/wbwangk/hyperledgerDocs.git
cd hyperledgerDocs
```
#### 编辑文档
为了测试，将`docs/index.md`做适当的修改，增加好辨识的标记，如加几个字。  
如果在文档中心的根目录增加文档，需要编辑配置文件`mkdocs.yml`。  

#### 提交到github
由于刚clone过来，当前分支是master，可以用下列命令查看当前分支：
```
$ git status
On branch master
```
下面将index.md提交到github：
```
$ git add docs/index.md
$ git commit -m "{add index.md}"
$ git push
(输入github的用户名和口令)
```
按一般的软件工程习惯，应该在本地测试通过后再提交到github，这里只是提前说明。

#### mkdocs构建和本地测试
下面命令会根据`mkdocs.yml`文件定义构建这个库。结果是输出到site目录下：
```
$ mkdocs build
```
下列命令会运行一个web服务将site目录下的内容代理到8000端口：
```
$ mkdocs serve  --dev-addr 192.168.16.103:8000
```
由于是在虚拟机中运行，而且不是NAT模式，所以需要加上`--dev-addr`参数确保绑定的IP不是默认的127.0.0.1。  
在windows(宿主机)下打开浏览器访问地址`192.168.16.103:8000`就可以看到文档内容了。注意自己对index.md的修改内容是否生效。  

#### 提交到github
由于要使用github pages展现静态网站内容，所以需要将site目录的内容提交到`gh-pages`分支。而markdown源码的分支是master。  
现在切换到`gh-pages`分支，然后把site目录的内容复制库的根目录下。github pages象一般web服务器一样，会找根目录下的index.html作为首页：
```
$ git checkout gh-pages
$ cp -R site/* .
$ git add .
$ git commit -m "{mkdocs commit}"
$ git push
(按提示输入github的用户名和口令)
```
#### 提交到github(补充)
后发现mkdocs可以直接发布到github pages。
```
$ mkdocs build
$ mkdocs gh-deploy
INFO    -  Cleaning site directory
INFO    -  Building documentation to directory: /opt/ComposerDocs/site
INFO    -  Copying '/opt/ComposerDocs/site' to 'gh-pages' branch and pushing to GitHub.
Username for 'https://github.com': wbwangk
Password for 'https://wbwangk@github.com':
INFO    -  Your documentation should shortly be available at: https://wbwangk.github.io/ComposerDocs/
```
也就是直接把site目录发布到了同一个库的gh-pages分支。太方便了。
#### 查看结果
首先，到`https://github.com/wbwangk/hyperledgerDocs/tree/gh-pages`查看提交的结果。  
然后到 https://wbwangk.github.io/hyperledgerDocs/ 去查看github pages解析后的结果。

## Restructured Text Markup, Sphinx
[原文](https://ethereum-homestead.readthedocs.io/en/latest/about.html#restructured-text-markup-sphinx)  
与Markdown类似的标记语言，格式更丰富。例如，下面以太坊homestead版本文档就是用rst格式编写的。编译办法：
```
git clone --depth 1 https://github.com/ethereum/homestead-guide
cd homestead-guide
make html
```
提示找不到`sphinx-build`命令。参考[这个官方文档](http://www.sphinx-doc.org/en/master/usage/installation.html)后，在ubuntu下安装：
```
sudo apt-get install python-sphinx
make html
```
就将这个项目构建出来了。构建出来的html文档存放在了`build`目录下。
