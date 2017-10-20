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
```bash
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
```handlebars
<h1>PeopleTracker</h1>
{{outlet}}
<p>end...</p>
```
ember会自动检测到模板文件变化，并自动加载。可以看到浏览器中欢迎页变成了`PeopleTracker`的大字和`end...`的小字。  
`{{outlet}}`的位置会渲染嵌入式路由的内容，到目前为止还没有定义嵌入式路由，所以这个位置并没有内容。  

#### 定义一个路由
在`/opt/ember-quickstart`目录下执行下列命令来创建一个路由：
```shell
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
```javascript
Router.map(function() {
  this.route('scientists');
});
```
创建了一个模板文件(`scientists.hbs`)和一个对应的Route对象(`scientists.js`)，还创建了一个单元测试程序文件(`scientists-test.js`)。`.hbs`是[handlebars](https://github.com/wycats/handlebars.js)(一种模板语言)的简写。

向新建的模板文件`app/templates/scientists.hbs`中加点页面元素：
```handlebars
<h2>List of Scientists</h2>
```
ember会自动加载这个模板文件。现在用浏览器访问地址`http://c7302.ambari.apache.org:4200/scientists`，可以看到两个模板文件`app/templates/application.hbs`和`app/templates/scientists.hbs`的内容合并后现在了屏幕上。而`scientists.hbs`的内容被渲染到了`application.hbs`模板中`{{outlet}}`的位置，这正是前面说的嵌入式路由。即：
```
PeopleTracker
List of Scientists
end...
```  
在实际使用中，模板要显示动态数据。数据通过`app/routes/scientists.js`提供，这就是所谓的模型。向`scientists.js`中增加`model()`方法：
```javascript
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return ['Marie Curie', 'Mae Jemison', 'Albert Hofmann'];
  }
});
```
`model()`方法是一个约定的钩子函数(hook)，会被ebmer框架调用。如果需要异步获取数据，`model()`支持[JavaScript Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。  
现在数据有了，修改`scientists.hbs`模板文件，以便在模板中展示模型中的数据：
```handlebars
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
```handlebars
<h2>{{title}}</h2>

<ul>
  {{#each people as |person|}}
    <li>{{person}}</li>
  {{/each}}
</ul>
```
从名称上看scientist是person的一种，这暗示了这个组件的通用性。从文件内容上看，与`scientists.hbs`很像。下面会用这个组件来完成模板`scientists.hbs`类似的功能，从而体现出复用。  
下面改写`scientists.hbs`为以下内容：
```handlebars
{{people-list title="List of Scientists(use component)" people=model}}
```
上面就是在模板中引用UI组件的语法，与html的标签有点像，只是把尖括号换成了两个花括号。  
通过上述定义，指定了引用组件的id(`people-list`)，定义了变量title的值，为people这个变量赋值了路由`model()`方法的返回值。   
回到浏览器，可以看到显示的文字中多了`(use component)`这些字符。  
为了体现组件的复用，可以把`scientists.hbs`改成：
```handlebars
{{people-list title="List of Scientists(use component)" people=model}}
{{people-list title="List of Scientists(use component2)" people=model}}
```
可以看到浏览器中的科学家列表显示了两次。为了体现复用，还可以自己定义一个新的程序员(`programmers`)路由，然后显示与科学家列表类似的界面，会发现重用UI组件使编码量大大减少，切提高了一致性。    

#### 点击事件
在`people-list`组件模板文件(`app/templates/components/people-list.hbs`)的`li`标签中添加一个`action`的帮助器。(帮助器是handlebar的概念，有点像函数)：
```handlebars
<h2>{{title}}</h2>
<ul>
  {{#each people as |person|}}
    <li {{action "showPerson" person}}>{{person}}</li>
  {{/each}}
</ul>
```
`action`帮助器允许你向元素中添加事件监听器，并调用指定的函数。默认`action`帮助器会添加鼠标的`click`事件监听器，但也可以监听其它元素事件。通过上面的定义，`li`元素的点击事件会调用函数`showPerson`，类似于调用`this.actions.showPerson(person)`。  
`actions`事件的showPerson函数需要定义在`people-list`组件的js文件(app/components/people-list.js)中：
```javascript
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
```shell
$ ember build --env production
```
构建命令会创建`dist`目录，把需要像web服务器发布的内容打包进去。如果有兴趣以快速可靠的方式将应用程序部署到生产环境中，请查看[Ember CLI Deploy](http://ember-cli-deploy.com/)插件。    

## 教程
本教程试图用Emberjs创建一个“网上房屋租赁”的web应用程序。完成教程代码位于`https://github.com/ember-learn/super-rentals`。  
### 创建应用程序
Ember CLI，Ember的命令行界面提供了一个标准的项目结构，一组开发工具和一个插件系统。这允许Ember开发人员专注于构建应用程序，而不是构建使它们运行的​​支持结构。可通过`ember --help`显示Ember CLI提供的命令，或通过`ember help <command-name>`查看特定命令的信息。  

#### 创建应用程序
```
$ ember new super-rentals
```
上述`ember new`命令会创建一个叫`super-rentals`的目录，还有一些骨架程序，然后调用`npm install`安装必要的依赖包。进入应用目录并开始工作：  
```
$ cd super-rentals
```
`new`命令生成的项目目录和文件如下：
```
|--app
|--config
|--node_modules
|--public
|--tests
|--vendor

<other files>

ember-cli-build.js
package.json
README.md
testem.js
```
**app**：这是存储模型，组件，路由，模板和样式的文件夹和文件的地方。您在Ember项目中的大部分编码都发生在此文件夹中。

**config**：`config`目录包含environment.js，在这里可以配置应用程序设置。

**node_modules/package.json**：此目录和文件来自npm。npm是Node.js的包管理器。Ember使用Node构建，并使用各种Node.js模块进行操作。该`package.json`文件维护该应用程序的当前npm依赖关系的列表。您安装的任何Ember CLI插件也将显示在此处。列出的软件包`package.json`安装在node_modules目录中。

**public**：此目录包含图像和字体等资源。

**vendor**：此目录是由Bower不管理的前端依赖关系（如JavaScript或CSS）。

**tests/testem.js**：我们的应用程序的自动化测试会进入tests文件夹，并配置了Ember CLI的测试运行器testemtestem.js。

**ember-cli-build.js**：该文件描述了Ember CLI如何构建我们的应用程序。

#### ES6模块
看一下`app/router.js`文件的内容；
```javascript
import Ember from 'ember';
import config from './config/environment';

const Router = Ember.Router.extend({
  location: config.locationType,
  rootURL: config.rootURL
});

Router.map(function() {
});

export default Router;
```
Ember CLI使用ECMAScript 2015（简称ES2015或以前称为ES6）模块来组织应用程序代码。例如，该行`import Ember from 'ember';`允许我们访问实际的Ember.js库作为变量`Ember`。该`import config from './config/environment';`行可让我们访问我们的应用程序的配置数据作为变量`config`。`const`是一种声明只读变量的方式，以确保它不会在其他地方重新分配。在文件的最后，`export default Router;`使`Router`此文件中定义的变量可用于应用程序的其他部分。

#### 开发服务器
当我们生成一个新项目，可以通过启动开发服务器来检验一切是否正常：
```
$ ember server --host c7302.ambari.apache.org
```
如果不加`--host`参数则使用`localhost`主机名。  
现在用浏览器打开地址`http://c7302.ambari.apache.org:4200`，可以看到默认欢迎页面。我们可以编辑`app/templates/application.hbs`文件来将欢迎页面修改成自己的内容。  
打开另外的终端窗口，编辑文件`app/templates/application.hbs`。删除组件`{{welcome-page}}`，修改为：
```
<h1> This my webcome page .... </h1>
{{outlet}}
```
浏览器自动加载了新的模板，欢迎页变了。  

### 规划你的应用
为了展示如何搭建Ember应用程序，我们将搭建一个资产租赁的应用，叫`Super Rentals`。我们将开始于一个home页面，一个about页面和一个联系我们页面。
完成后的应用大约是下面的样子：  
![](https://guides.emberjs.com/v2.15.0/images/service/style-super-rentals-maps.png)  
应用程序的构成是：
- 在home页面上显示租赁清单
- 链接到关于公司的页面
- 链接到“联系我们”的页面
- 列出有效的租赁清单
- 按城市过滤租赁清单
- 显示一个选中租赁的详细信息

####用We Go测试应用程序
验收测试与我们的应用程序进行交互，就像实际的人一样，但是是自动化的，有助于确保我们的应用程序在将来不会中断。  
当我们使用Ember CLI创建一个新的Ember项目时，它使用[QUnit](https://qunitjs.com/)JavaScript测试框架来定义和运行测试。  
我们将首先使用Ember CLI生成新的验收测试：
```
$ ember g acceptance-test list-rentals
installing acceptance-test
  create tests/acceptance/list-rentals-test.js
```
可以打开生成的`tests/acceptance/list-rentals-test.js`的文件看看其内容。发现生成的测试代码的第一个测试是请求`/list-rentals`路由。这时我们的应用还没有定义任何路由，所以可以把文件中的三个`/list-rentals`都改成`/`来测试应用的基础URL`http://c7302.ambari.apache.org:4200/`。修改后的`tests/acceptance/list-rentals-test.js`如下：
```javascript
import { test } from 'qunit';
import moduleForAcceptance from 'super-rentals/tests/helpers/module-for-acceptance';

moduleForAcceptance('Acceptance | list rentals');

test('visiting /', function(assert) {
  visit('/');

  andThen(function() {
    assert.equal(currentURL(), '/');
  });
});
```
对于这个简单测试，需要注意的几点是：  
- 验收测试通过调用函数`moduleForAcceptance`来建立，此函数确保您的Ember应用程序在每次测试之间启动和关闭。  
- QUnit传到一个叫assert的对象到每个测试函数。assert含有函数，如equal()，允许检查测试环境中的条件。一个测试必须有一个通过断言才能成功。  
- Ember验收试验使用一组测试帮助函数，如visit，andThen和上面使用的currentURL函数。我们将在本教程的后面更详细地讨论这些功能。  

现在启动测试：
```
$ ember test --server --host c7302.ambari.apache.org
```
默认情况下，当执行上述命令时，Ember CLI 会运行 [Testem test runner](https://github.com/testem/testem), 它会在Chrome和[PhantomJS](http://phantomjs.org/)中运行Qunit。  

按屏幕的提示，让用浏览器访问地址`http://c7302.ambari.apache.org:7357/`。   
![](https://guides.emberjs.com/v2.15.0/images/acceptance-test/initial-tests.png)   
浏览器上现在显示10次成功测试。如果取消选中`Hide passed tests`，应该看到我们的验收测试成功，以及9次通过的[ESLint](http://eslint.org/)测试。  

### 路由和模板
对于Super Rentals，我们希望首先到达home页面，在上面显示租赁列表，然后可以跳转到`about`页面和`contact`页面。(下列测试在`/opt/super-rentals`目录下进行)  
#### about路由
生成about路由：
```
$ ember generate route about
installing route
  create app/routes/about.js
  create app/templates/about.hbs
updating router
  add route about
installing route-test
  create tests/unit/routes/about-test.js
```
一个Ember路由由三部分组成：
- 一个在Ember路由(`app/router.js`)中的条目，它会将路由名称映射到一个特定URI(或者反过来映射)  
- 一个路由处理文件(如`app/routes/about.js`)，定义路由被加载时发生什么  
- 一个路由模板(如`app/templates/about.hbs`)，定义显示的内容
打开`app/router.js`看看，发现多了下面的代码：
```
Router.map(function() {
  this.route('about');
});
```
上述代码告诉Ember路由，当访问URI`/about`时运行`app/routes/about.js`文件。  
然后打开模板文件`app/templates/about.hbs`，修改为下列内容：
```handlebars
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>About Super Rentals</h2>
  <p>
    The Super Rentals website is a delightful project created to explore Ember.
    By building a property rental site, we can simultaneously imagine traveling
    AND building Ember applications.
  </p>
</div>
```
用浏览器访问地址`http://c7302.ambari.apache.org:4200/about`测试一下`about`页面。  

#### 联系我们路由
生成联系我们路由：
```
$ ember g route contact
```
Ember CLI再一次向`app/router.js`中添加了contact条目，生成了一个路由处理器`app/routes/contact.js`，以及模板文件`app/templates/contact.hbs`。  
向模板文件中添加下列内容：
```handlebars
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>Contact Us</h2>
  <p>Super Rentals Representatives would love to help you<br>choose a destination or answer
    any questions you may have.</p>
  <p>
    Super Rentals HQ
    <address>
      1212 Test Address Avenue<br>
      Testington, OR 97233
    </address>
    <a href="tel:503.555.1212">+1 (503) 555-1212</a><br>
    <a href="mailto:superrentalsrep@emberjs.com">superrentalsrep@emberjs.com</a>
  </p>
</div>
```
用浏览器访问地址`http://c7302.ambari.apache.org:4200/contact`可以看到新增加的“联系我们”页面。  

#### 链接导航和{{link-to}}帮助器
为了方便在页面间跳转，需要在“关于”页面增加一个链接到“联系我们”页面，同样需要在“联系我们”页面增加一个链接到“关于”页面。  
为了做到这一点，我们将使用`{{link-to}}`这个Ember提供的帮助器，这样可以轻松地在我们的路由之间进行链接。我们来调整我们的`about.hbs`文件：
```
(略)
  {{#link-to 'contact' class="button"}}
    Contact Us
  {{/link-to}}
</div>
```
在这种情况下，我们告诉`{{link-to}}`帮助器我们要链接到的路由的名称：`contact`。用浏览器访问地址`http://c7302.ambari.apache.org:4200/about`，看到页面多了一个链接到“联系我们”页面。  
同样，修改`app/templates/contact.hbs`文件添加到`about`路由的链接：
```
(略)
  {{#link-to 'about' class="button"}}
    About
  {{/link-to}}
</div>
```
#### 租赁清单路由
下面添加一个显示租赁清单的路由(`rentals`):
```
$ ember g route rentals
```
先往租赁清单模板文件(`app/templates/rentals.hbs`)添加点初始内容，之后再进一步补充：
```
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>Welcome!</h2>
  <p>We hope you find exactly what you're looking for in a place to stay.</p>
  {{#link-to 'about' class="button"}}
    About Us
  {{/link-to}}
</div>
```
#### index路由
index路由用于处理对于网站根URI(`/`)请求。我们想用租借列表页面(URI是`/rentals`)充当应用的主页面。因此，我们希望我们的index路由简单地转发到已经创建的rentals路由。  
创建index路由：
```
$ ember g route index
installing route
  create app/routes/index.js
  create app/templates/index.hbs
installing route-test
  create tests/unit/routes/index-test.js
```
从上面的提示中可以看出，index路由比较特殊，不需要向路由映射(`app/router.js`)中添加条目。  
我们的需求是当用户访问根URI(`/`)转向到`/rentals`。为了实现这个需要需要在index路由的处理程序(`index.js`)中实现一个叫`beforeModel`路由生命周期钩子函数。  
每个路由处理程序都有一组“生命周期钩子”，它们是在加载页面时在特定时间被调用的函数。在[beforeModel](http://emberjs.com/api/classes/Ember.Route.html#method_beforeModel)钩子在数据从模型钩子取出之前执行，也在页面被渲染之前。  
在index处理程序中，我们调用[replaceWith](http://emberjs.com/api/classes/Ember.Route.html#method_replaceWith)函数。该`replaceWith`函数类似于路由的`transitionTo`函数，区别在于`replaceWith`将替换浏览器历史中的当前URL，而`transitionTo`将添加到历史记录中。由于我们希望我们的`rentals`路由作为我们的主页，我们将使用该`replaceWith`函数。  
将index处理程序(`app/routes/index.js`)修改成如下的样子：
```javascript
import Ember from 'ember';

export default Ember.Route.extend({
  beforeModel() {
    this.replaceWith('rentals');
  }
});
```
现在访问根路由`/`将导致`/rentals`URL的加载。  

#### 添加顶部导航
我们希望在所有页面的顶部添加一个通用区域显示应用标题和导航条。  
为了完成这个需求，需要修改应用模板(`/app/templates/application.hbs`)。先看原来的内容：
```
<h1> This my webcome page .... </h1>
{{outlet}}
```
在上面的模板下，应用所有页面的顶部都会显示`<h1>This my webcome page .... </h1>`，而`{{outlet}}`的位置会显示当前路由的网页内容。  
为了完成本节需求，需要将应用模板修改成下列的样子：
```handlebars
<div class="container">
  <div class="menu">
    {{#link-to 'index'}}
      <h1>
        <em>SuperRentals</em>
      </h1>
    {{/link-to}}
    <div class="links">
      {{#link-to 'about'}}
        About
      {{/link-to}}
      {{#link-to 'contact'}}
        Contact
      {{/link-to}}
    </div>
  </div>
  <div class="body">
    {{outlet}}
  </div>
</div>
```

## 参考
[ECMAScript 6 入门](http://es6.ruanyifeng.com/)