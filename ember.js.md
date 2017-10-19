## quick-start
[原文](https://guides.emberjs.com/v2.15.0/getting-started/quick-start/)  
安装Ember.js:  
```
$ npm install -g ember-cli@2.15
```
#### 创建一个新应用
```
$ ember new ember-quickstart
```
new命令会以创建一系列的目录和文件，包括npm的包依赖配置文件package.json。然后执行`npm install`。也可以先安装安装依赖包，然后手工执行：
```
$ cd /opt
$ ember new --skip-npm ember-quickstart
$ cd ember-quickstart
$ npm install
```
也可以把npm换成cnpm，这样会使用国内的npm源，也许能让依赖包下载快点。  
启用开发服务器：
```
$ ember serve --host c7302.ambari.apache.org
```
`ember serve`默认会监听localhost主机，用`--host`参数可以让它监听其它ip。这对于ember运行在linux虚拟机，而用windows浏览器去访问ember服务时很有用。  
用浏览器访问`http://c7302.ambari.apache.org:4200`就可以看到刚创建的Ember应用的欢迎页。  
打开另外的终端窗口(如git bash)，编辑`app/templates/application.hbs`为下列内容：
```
<h1>PeopleTracker</h1>
{{outlet}}
<p>end...</p>
```
ember会自动检测到模板文件变化，并自动加载。可以看到浏览器中欢迎页变成了`PeopleTracker`的大字和`end...`的小字。  
`{{outlet}}`的位置会渲染嵌入式路由的内容，到目前为止还没有定义嵌入式路由，所以这个位置并没有内容。  

#### 定义一个路由
在`/opt/ember-quickstart`目录下执行下列命令来创建一个路由：
```
$ ember generate route scientists
installing route
  create app/routes/scientists.js
  create app/templates/scientists.hbs
updating router
  add route scientists
installing route-test
  create tests/unit/routes/scientists-test.js
```
根据上述提示也可以看出，命令在`app/router.js`中添加了一个条目：
```
Router.map(function() {
  this.route('scientists');
});
```
创建了一个模板文件(`scientists.hbs`)和一个对应的Route对象(`scientists.js`)，还创建了一个单元测试程序文件(`scientists-test.js`)。`.hbs`是[handlebars](https://github.com/wycats/handlebars.js)(一种模板语言)的简写。

向新建的模板文件`app/templates/scientists.hbs`中加点页面元素：
```
<h2>List of Scientists</h2>
```
ember会自动加载这个模板文件。现在用浏览器访问地址`http://c7302.ambari.apache.org:4200/scientists`，可以看到两个模板文件`app/templates/application.hbs`和`app/templates/scientists.hbs`的内容合并后现在了屏幕上。而`scientists.hbs`的内容被渲染到了`application.hbs`模板中`{{outlet}}`的位置，这正是前面说的嵌入式路由。即：
```
PeopleTracker
List of Scientists
end...
```  
在实际使用中，模板要显示动态数据。数据通过`app/routes/scientists.js`提供，这就是所谓的模型。向`scientists.js`中增加`model()`方法：
```
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return ['Marie Curie', 'Mae Jemison', 'Albert Hofmann'];
  }
});
```
`model()`方法名是一个约定，会被ebmer框架调用。如果需要异步获取数据，`model()`支持[JavaScript Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。  
现在数据有了，修改`scientists.hbs`模板文件，以便在模板中展示模型中的数据：
```
<h2>List of Scientists</h2>
{{outlet}}
<ul>
  {{#each model as |scientist|}}
    <li>{{scientist}}</li>
  {{/each}}
</ul><h2>List of Scientists</h2>
```
同原文相比保留了`{{outlet}}`，这为渲染更下级嵌入式路由内容指定了位置。  
vue.js和angular2的模板语言使用了html的自定义元素(即自定义tag)风格，而handlebars则采用了[Mustache](http://mustache.github.io)模板语法风格。前者兼容html，后者个人感觉更容易阅读。  

## Test
#### Ember's test helpers
- **visit** - loads a given URL
- **click** - pretends to be a user clicking on a specific part of the screen
- **andThen** - waits for our previous commands to run before executing our function. In our test below, we want to wait for our page to load after click is called so that we can double-check that the new page has loaded
 - **currentURL** - returns the URL of the page we're currently on

### Ember Addon
https://emberobserver.com/
