使用 pyenv 可以在一个系统中安装多个python版本。
This project used to be known as pythonbrew, but is now known as pyenv. To install it you need to clone a copy of it into your $HOME directory like so(下载pyenv):
```
$ git clone git://github.com/yyuu/pyenv.git ~/.pyenv
Cloning into .pyenv...
remote: Counting objects: 2207, done.
remote: Compressing objects: 100% (617/617), done.
remote: Total 2207 (delta 1489), reused 2172 (delta 1462)
Receiving objects: 100% (2207/2207), 358.75 KiB, done.
Resolving deltas: 100% (1489/1489), done.
```
如root用户安装到了```/root/.pyenv/```目录下。  
设置pyenv路径到~/.bashrc（.bashrc会在用户切换时自动执行）：
```
$ echo 'export PATH="$HOME/.pyenv/bin:$PATH"' >> .bashrc
$ echo 'eval "$(pyenv init -)"' >> .bashrc
```
查看pyenv帮助:
```
$ pyenv 
pyenv 0.4.0-20130613-17-ge1ea64b
Usage: pyenv <command> [<args>]

Some useful pyenv commands are:
   commands    List all available pyenv commands
   local       Set or show the local application-specific Python version
   global      Set or show the global Python version
   shell       Set or show the shell-specific Python version
   install     Install a Python version using the python-build plugin
   uninstall   Uninstall a specific Python version
   rehash      Rehash pyenv shims (run this after installing executables)
   version     Show the current Python version and its origin
   versions    List all Python versions available to pyenv
   which       Display the full path to an executable
   whence      List all Python versions that contain the given executable

See `pyenv help <command>' for information on a specific command.
For full documentation, see: https://github.com/yyuu/pyenv#readme
```
看看有哪些python版本可以切换。星号表示当前版本:
```
$ pyenv versions
* system (set by /home/saml/.pyenv/version)
```
让我们安装Python 3.2.5:
```
$ pyenv install 3.2.5
Downloading Python-3.2.5.tgz...
-> http://yyuu.github.io/pythons/ed8d5529d2aebc36b53f4e0a0c9e6728
Installing Python-3.2.5...
Installed Python-3.2.5 to /home/saml/.pyenv/versions/3.2.5

Downloading setuptools-0.9.5.tar.gz...
-> https://pypi.python.org/packages/source/s/setuptools/setuptools-0.9.5.tar.gz
Installing setuptools-0.9.5...
Installed setuptools-0.9.5 to /home/saml/.pyenv/versions/3.2.5

Downloading pip-1.3.1.tar.gz...
-> http://yyuu.github.io/pythons/cbb27a191cebc58997c4da8513863153
Installing pip-1.3.1...
Installed pip-1.3.1 to /home/saml/.pyenv/versions/3.2.5
```
为新的安装重建环境：
```
$ pyenv rehash
```
现在可以看到两个版本了，system仍然是默认的(*)：
```
$ pyenv versions
* system (set by /home/saml/.pyenv/version)
  3.2.5
```
让我们切换到3.2.5:
```
$ pyenv which python
/usr/bin/python

$ pyenv global 3.2.5

$ pyenv which python
/home/saml/.pyenv/versions/3.2.5/bin/python

$ pyenv versions
  system
* 3.2.5 (set by /home/saml/.pyenv/version)

$ python --version
Python 3.2.5
```
星号(*)现在3.2.5的前面，表示默认(全局)版本是3.2.5。  