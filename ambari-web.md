https://cwiki.apache.org/confluence/display/AMBARI/Coding+Guidelines+for+Ambari
```
$ sudo npm install -g brunch@1.7.20
npm: relocation error: npm: symbol SSL_set_cert_cb, version libssl.so.10 not defined in file libssl.so.10 with link time reference
```
google到[这个网页](https://bugzilla.redhat.com/show_bug.cgi?id=1481470)的办法：
```
$ yum-config-manager --enable cr && yum update
```
这导致yum可以安装未正式发布的测试包。  
之后虽然可以安装了，但很慢，创建一个使用淘宝NPM镜像的别名：
```
$ alias cnpm="npm --registry=https://registry.npm.taobao.org \
--cache=$HOME/.npm/.cache/cnpm \
--disturl=https://npm.taobao.org/dist \
--userconfig=$HOME/.cnpmrc"
$ sudo cnpm install -g brunch@1.7.20
```
运行`cnpm install`时提示phantomjs没有在路径上，手工下载phantomjs，并添加到PATH:
```
$ cd /opt
$ wget https://github.com/Medium/phantomjs/releases/download/v2.1.1/phantomjs-2.1.1-linux-x86_64.tar.bz2
$ tar -xjf phantomjs-2.1.1-linux-x86_64.tar.bz2
$ export PATH=$PATH:/opt/phantomjs-2.1.1-linux-x86_64/bin
```
根据社区文章：https://community.hortonworks.com/questions/26377/how-can-i-build-hdp-for-source.html
从github上下载HDP的ambari源码包：
https://github.com/hortonworks/ambari-release/archive/AMBARI-2.5.1.7-tag.zip

按[Ambari Web Frontend Development Environment](https://cwiki.apache.org/confluence/display/AMBARI/Coding+Guidelines+for+Ambari)的办法进行构建，发现少了`info.json`文件：
```
$ ambari/ambari-web/app/assets/data/clusters/info.json
$ ambari/ambari-web/public/data/clusters/info.json
```
导致登录ambari UI后报错，把上述两个文件从apache ambari项目复制到HDP ambari项目后重新启动：
```
$ brunch watch --server
```
问题解决！
