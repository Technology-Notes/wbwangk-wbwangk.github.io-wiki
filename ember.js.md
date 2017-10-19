## 快速开始
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
`model()`方法是一个约定的钩子函数(hook)，会被ebmer框架调用。如果需要异步获取数据，`model()`支持[JavaScript Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。  
现在数据有了，修改`scientists.hbs`模板文件，以便在模板中展示模型中的数据：
```
<h2>List of Scientists</h2>
<ul>
  {{#each model as |scientist|}}
    <li>{{scientist}}</li>
  {{/each}}
</ul><h2>List of Scientists</h2>
```
vue.js和angular2的模板语言使用了html的自定义元素(即自定义tag)风格，而handlebars则采用了[Mustache](http://mustache.github.io)模板语法风格。前者兼容html，后者个人感觉更容易阅读。  

模板文件`scientists.hbs`保存后，浏览器自动刷新为显示了科学家列表：
```
PeopleTracker

List of Scientists

 . Marie Curie
 . Mae Jemison
 . Albert Hofmann

List of Scientists

end...
```
#### 创建一个UI组件
UI组件可以在多个页面之中复用，或在同一个页面中复用多次。  
创建组件的命令行(组件id是people-list)：
```
$ ember generate component people-list
installing component
  create app/components/people-list.js
  create app/templates/components/people-list.hbs
installing component-test
  create tests/integration/components/people-list-test.js
```
从上述提示上可以看出，UI组件由对象和模板构成。现在编辑模板文件`app/templates/components/people-list.hbs`为以下内容：
```
<h2>{{title}}</h2>

<ul>
  {{#each people as |person|}}
    <li>{{person}}</li>
  {{/each}}
</ul>
```
从名称上看scientist是person的一种，这暗示了这个组件的通用性。从文件内容上看，与`scientists.hbs`很像。下面会用这个组件来完成模板`scientists.hbs`类似的功能，从而体现出复用。  
下面改写`scientists.hbs`为以下内容：
```
{{people-list title="List of Scientists(use component)" people=model}}
```
上面就是在模板中引用UI组件的语法，与html的标签有点像，只是把尖括号换成了两个花括号。  
通过上述定义，指定了引用组件的id(`people-list`)，定义了变量title的值，为people这个变量赋值了路由`model()`方法的返回值。   
回到浏览器，可以看到显示的文字中多了`(use component)`这些字符。  
为了体现组件的复用，可以把`scientists.hbs`改成：
```
{{people-list title="List of Scientists(use component)" people=model}}
{{people-list title="List of Scientists(use component2)" people=model}}
```
可以看到浏览器中的科学家列表显示了两次。为了体现复用，还可以自己定义一个新的程序员(`programmers`)路由，然后显示与科学家列表类似的界面，会发现重用UI组件使编码量大大减少，切提高了一致性。    

#### 点击事件
在`people-list`组件模板文件(`app/templates/components/people-list.hbs`)的`li`标签中添加一个`action`的帮助器。(帮助器是handlebar的概念，有点像函数)：
```
<h2>{{title}}</h2>
<ul>
  {{#each people as |person|}}
    <li {{action "showPerson" person}}>{{person}}</li>
  {{/each}}
</ul>
```
`action`帮助器允许你向元素中添加事件监听器，并调用指定的函数。默认`action`帮助器会添加鼠标的`click`事件监听器，但也可以监听其它元素事件。通过上面的定义，`li`元素的点击事件会调用函数`showPerson`，类似于调用`this.actions.showPerson(person)`。  
`actions`事件的showPerson函数需要定义在`people-list`组件的js文件(app/components/people-list.js)中：
```
import Ember from 'ember';

export default Ember.Component.extend({
  actions: {
    showPerson(person) {
      alert(person);
    }
  }
});
```
浏览器的`http://c7302.ambari.apache.org:4200/scientists`网页会自动加载修改的网页。尝试点击一下科学家的名字，浏览器会弹出了一个alert模式小窗口，小窗口中显示了点击科学家的名字。

#### 生产构建