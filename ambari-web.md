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

## Ember.js
[原文](https://guides.emberjs.com/v2.15.0/getting-started/quick-start/)  
安装Ember.js。用npm很慢，cnpm快很多。
```
$ cnpm install -g ember-cli@2.15
```
创建一个新应用，按原文的办法使用npm安装依赖包很慢。改进后的办法是：
```
$ ember new --skip-npm ember-quickstart
$ cd ember-quickstart
$ cnpm install
```
这样就是使用cnpm安装依赖包，怀疑能快10倍。  
启用开发服务器：
```
$ ember serve
```
用浏览器访问`http://c7302.ambari.apache.org:4200`就可以看到刚创建的Ember应用的欢迎页。  
编辑`app/templates/application.hbs`为下列内容：
```
<h1>PeopleTracker</h1>
{{outlet}}
```
重新执行`ember serve`，然后可以看到欢迎页变了。  

## Test
#### Ember's test helpers
- **visit** - loads a given URL
- **click** - pretends to be a user clicking on a specific part of the screen
- **andThen** - waits for our previous commands to run before executing our function. In our test below, we want to wait for our page to load after click is called so that we can double-check that the new page has loaded
 - **currentURL** - returns the URL of the page we're currently on

### Ember Addon
https://emberobserver.com/

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
