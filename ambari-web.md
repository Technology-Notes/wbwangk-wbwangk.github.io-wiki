原文地址：`https://cwiki.apache.org/confluence/display/AMBARI/Coding+Guidelines+for+Ambari`  
### 安装和启动过程
```
$ git clone https://git-wip-us.apache.org/repos/asf/ambari.git
$ cd ambari/ambari-web
$ sudo npm install -g brunch@1.7.20
$ rm -rf node_modules public
$ npm install
$ brunch build
``
启动前端服务：
```
$ brunch watch --server
```

### 安装中碰到的问题
用手机上网问题少，用公司网络上网问题多。手机上网只出现了`phantomjs-prebuilt@2.1.15`安装失败。  

#### 1.phantomjs安装失败
如果内网安装报错，可以用手机联网试试。用手机联网报过错：`phantomjs-prebuilt@2.1.15`安装失败。直接手工装安装上面的包，再执行`npm install`:   
```
$ npm install phantomjs-prebuilt@2.1.15
$ npm install
```
#### npm relocation错误
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
#### 3. phantomjs错误的另一个办法
运行`cnpm install`时提示phantomjs没有在路径上，手工下载phantomjs，并添加到PATH:
```
$ cd /opt
$ wget https://github.com/Medium/phantomjs/releases/download/v2.1.1/phantomjs-2.1.1-linux-x86_64.tar.bz2
$ tar -xjf phantomjs-2.1.1-linux-x86_64.tar.bz2
$ export PATH=$PATH:/opt/phantomjs-2.1.1-linux-x86_64/bin
```
#### 4. 缺少info.json
根据社区文章：https://community.hortonworks.com/questions/26377/how-can-i-build-hdp-for-source.html
从github上下载HDP的ambari源码包：
https://github.com/hortonworks/ambari-release/archive/AMBARI-2.5.1.7-tag.zip

按[Ambari Web Frontend Development Environment](https://cwiki.apache.org/confluence/display/AMBARI/Coding+Guidelines+for+Ambari)的办法进行构建，并启动：
```
$ brunch watch --server
```
登录ambari UI后报错，发现少了`info.json`文件。自己造一个：
```
$ cd app/assets/data/clusters
$ vi info.json
{
  "items" : [
    {
      "Clusters" : {
        "cluster_name" : "cc",
        "provisioning_state" : "INSTALLED",
        "version" : "HDP-2.4.3"
      }
    }
  ]
}
```
重新启动`brunch watch --server`后问题解决！

## api-docs
ambari的api文档以swagger.json的方式提供。在ambari-web目录看到一个api-docs目录，这个目录是swagger显示引擎的目录。可以将nginx的根目录设置到这里。
```
$ nginx -t 
nginx: the configuration file /usr/local/openresty/nginx/conf/nginx.conf syntax is ok
```
编辑上面的nginx.conf文件：
```
   server {
        listen       80;
        server_name  localhost;
        location / {
            root /opt/ambari/ambari-web/api-docs;
            index  index.html index.htm;
        }
   }
```
然后找到ambari的swagger.json文件：
```
$ find /opt/ambari -name swagger.json
/opt/ambari/ambari-server/docs/api/generated/swagger.json
$ cp /opt/ambari/ambari-server/docs/api/generated/swagger.json /opt/ambari/ambari-web/api-docs
$ nginx -t
$ nginx
$ curl localhost
```
假设这个linux虚拟机的ip是：192.168.73.102。则在windows下用浏览器访问地址`http://192.168.73.102`就可以用swagger阅读ambari API文档了。

## Ember.js
[原文](https://guides.emberjs.com/v2.15.0/getting-started/quick-start/)   
整理后中Ember.js的快速开始和教程在[这里](https://github.com/wbwangk/wbwangk.github.io/wiki/ember.js)。  

## Mustache
Mustache是一个无逻辑模板语言。称无逻辑，因为它的定义中没有循环、判断等语法，只有标签。  
官网：http://mustache.github.io。  
官方只提供了RUBY实现，但大量爱好者提供了各种语言的实现。其[javascript的实现](https://github.com/janl/mustache.js)在github上有一万多颗星。  
Mushtache的语法在官网的[这个链接](http://mustache.github.io/mustache.5.html)。  

## handlebars.js
[github home](https://github.com/wycats/handlebars.js)  
handlebars.js是Mustache的超集，和javascript实现。  
handlebars.js的增强部分：  
- 内部路径，可以访问json串内部属性，如`{{author.name}}`
- helpers。在mustache的体系下，可以在json串中嵌入代码(Lambda)，把数据和代码放在一起有些怪。handerbars.js以helper代替，helper对代码的调用显式定义在模板中，格式类似`{fullName author}}`。`fullName`就是一个helper。  
- 块表达式(mustache称为`Section`)。增强了mustache的section，使块也支持helper。格式类似`{{#fillName author}}`。   

#### Ember Simple Auth Token
https://github.com/jpadilla/ember-simple-auth-token