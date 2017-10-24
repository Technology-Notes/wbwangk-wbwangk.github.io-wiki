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
在`people-list`组件模板文件(`app/templates/components/people-list.hbs`)的`li`标签中添加一个`action`的助手。(助手是handlebar的概念，有点像函数)：
```handlebars
<h2>{{title}}</h2>
<ul>
  {{#each people as |person|}}
    <li {{action "showPerson" person}}>{{person}}</li>
  {{/each}}
</ul>
```
`action`助手允许你向元素中添加事件监听器，并调用指定的函数。默认`action`助手会添加鼠标的`click`事件监听器，但也可以监听其它元素事件。通过上面的定义，`li`元素的点击事件会调用函数`showPerson`，类似于调用`this.actions.showPerson(person)`。  
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
```handlebars
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

上面列出了6个路由，在下文的验收测试中提到“应用目标”，就是指将这6个路由分别测试一遍。如果再加上根路由`/`，则共有7个路由。  

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
- Ember验收试验使用一组测试助手函数，如visit，andThen和上面使用的currentURL函数。我们将在本教程的后面更详细地讨论这些功能。  

现在启动测试：
```
$ ember test --server --host c7302.ambari.apache.org
```
默认情况下，当执行上述命令时，Ember CLI 会运行 [Testem test runner](https://github.com/testem/testem), 它会在Chrome和[PhantomJS](http://phantomjs.org/)中运行Qunit。  

按屏幕的提示，让用浏览器访问地址`http://c7302.ambari.apache.org:7357/`。   
![](https://guides.emberjs.com/v2.15.0/images/acceptance-test/initial-tests.png)   
浏览器上现在显示10次成功测试。如果取消选中`Hide passed tests`，应该看到我们的验收测试成功，以及9次通过的[ESLint](http://eslint.org/)测试。  

#### 添加应用的完整目标到验收测试

虽然现在只有一个根路由`/`可以测试，但可以先创建完整的测试文件。编辑`tests/acceptance/list-rentals-test.js`，如下列内容：
```javascript
import { test } from 'qunit';
import moduleForAcceptance from 'super-rentals/tests/helpers/module-for-acceptance';

moduleForAcceptance('Acceptance | list-rentals');

test('should show rentals as the home page', function (assert) {
});

test('should link to information about the company.', function (assert) {
});

test('should link to contact information.', function (assert) {
});

test('should list available rentals.', function (assert) {
});

test('should filter the list of rentals by city.', function (assert) {
});

test('should show details for a selected rental', function (assert) {
});

test('visiting /', function(assert) {
  visit('/');

  andThen(function() {
    assert.equal(currentURL(), '/');
  });
});
```
然后运行`ember test --server`。则在linux屏幕上显示：
```
Chrome 61.0 
    1/7 ✘ 
```
而浏览器上显示：
```
7 tests completed in 18000 milliseconds, with 6 failed, 0 skipped, and 0 todo.
1 assertions of 7 passed, 6 failed.
```
这是因为除了根路由`/`，其它路由还没有创建，所以测试出错。共7个测试，通过1个，失败了6个。

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
```javascript
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

#### 链接导航和{{link-to}}助手
为了方便在页面间跳转，需要在“关于”页面增加一个链接到“联系我们”页面，同样需要在“联系我们”页面增加一个链接到“关于”页面。  
为了做到这一点，我们将使用`{{link-to}}`这个Ember提供的助手，这样可以轻松地在我们的路由之间进行链接。我们来调整我们的`about.hbs`文件：
```handlebars
(略)
  {{#link-to 'contact' class="button"}}
    Contact Us
  {{/link-to}}
</div>
```
在这种情况下，我们告诉`{{link-to}}`助手我们要链接到的路由的名称：`contact`。用浏览器访问地址`http://c7302.ambari.apache.org:4200/about`，看到页面多了一个链接到“联系我们”页面。  
同样，修改`app/templates/contact.hbs`文件添加到`about`路由的链接：
```handlebars
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
```handlebars
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
```handlebars
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
现在通过浏览器访问地址`http://c7302.ambari.apache.org:4200/contact`可以看到在页面的顶部显示了标题`SuperRentals`以及`about`和`contact`两个链接。  

#### 实现验收测试
首先，我们要测试访问`/`是否正确重定向到`/rentals`。我们将使用Ember visit助手，然后确保我们当前的URL是`/rentals`重定向发生的。  
打开之前创建的验收创建文件`/tests/acceptance/list-rentals-test.js`，修改为：
```handlebars
（前面的6个test省略）
test('should show rentals as the home page', function (assert) {
  visit('/');
  andThen(function() {
    assert.equal(currentURL(), '/rentals', 'should redirect automatically');
  });
});
```
原来的`currentURL()`会等于`/`，而现在被`replaceWith`为`/rentals`，所以测试代码要改成上面
运行测试程序：
```
$ ember test --server
```
用浏览器访问`http://c7302.ambari.apache.org:7357`，然后发现7个测试成功了1个，6个失败。如果没有把`/`修改成`/rentals`，则7个都失败。  

#### Ember测试助手
Ember提供各种验收测试助手，使常见任务更容易，如访问路由，填写字段，点击链接/按钮，等待页面显示。

我们常用的一些助手是：
- visit - 加载给定的URL
- click - 假装是用户点击屏幕的特定部分
- andThen - 等待之前的命令执行结束，然后执行指定函数。在下面的测试中，我们等待click后页面的加载，然后检查页面是否加载正确
- currentURL - 返回我们当前所在页面的URL

#### 测试about和contact页面
如果`ember test`已经在执行，可以打开另外的终端窗口来编辑`tests/acceptance/list-rentals-test.js`文件，将之前的about和contact两个测试替换成下列代码：
```handlebars
test('should link to information about the company.', function (assert) {
  visit('/');
  click('a:contains("About")');
  andThen(function() {
    assert.equal(currentURL(), '/about', 'should navigate to about');
  });
});

test('should link to contact information', function (assert) {
  visit('/');
  click('a:contains("Contact")');
  andThen(function() {
    assert.equal(currentURL(), '/contact', 'should navigate to contact');
  });
});
```
`Ember test`检测到了测试文件的变化，并自动加载。现在7个测试成功了3个，分别是`/`、`/about`和`/contact`。  

### 模型钩子
Ember将一个页面的数据保存在一个名为`model`的对象中。为了简单起见，我们将使用硬编码的JavaScript对象数组填写我们的租赁列表页面的模型。之后，我们将切换到使用[Ember Data](https://github.com/emberjs/data)，这是一个在应用程序中管理数据的库。  
在Ember中，路由处理器(handler)负责为页面加载模型。`model`是一个钩子函数，这意味着Ember会在应用中约定的时间调用它。  
现在，让我们打开`rentals`路由处理器文件`app/routes/rentals.js`，在`model`函数中返回租赁对象数组:
```javascript
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return [{
      id: 'grand-old-mansion',
      title: 'Grand Old Mansion',
      owner: 'Veruca Salt',
      city: 'San Francisco',
      propertyType: 'Estate',
      bedrooms: 15,
      image: 'https://upload.wikimedia.org/wikipedia/commons/c/cb/Crane_estate_(5).jpg',
      description: 'This grand old mansion sits on over 100 acres of rolling hills and dense redwood forests.'
    }, {
      id: 'urban-living',
      title: 'Urban Living',
      owner: 'Mike TV',
      city: 'Seattle',
      propertyType: 'Condo',
      bedrooms: 1,
      image: 'https://upload.wikimedia.org/wikipedia/commons/0/0e/Alfonso_13_Highrise_Tegucigalpa.jpg',
      description: 'A commuters dream. This rental is within walking distance of 2 bus stops and the Metro.'

    }, {
      id: 'downtown-charm',
      title: 'Downtown Charm',
      owner: 'Violet Beauregarde',
      city: 'Portland',
      propertyType: 'Apartment',
      bedrooms: 3,
      image: 'https://upload.wikimedia.org/wikipedia/commons/f/f7/Wheeldon_Apartment_Building_-_Portland_Oregon.jpg',
      description: 'Convenience is at your doorstep with this charming downtown rental. Great restaurants and active night life are within a few feet.'

    }];
  }
});
```
请注意，这里使用了ES6速记法定义语法：`model()`相当于`model: function()`。  
然后，就是在模板文件中显示模型中的数据。编辑模板文件`app/templates/rentals.hbs`:
```handlebars
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>Welcome!</h2>
  <p>
    We hope you find exactly what you're looking for in a place to stay.
  </p>
  {{#link-to 'about' class="button"}}
    About Us
  {{/link-to}}
</div>

{{#each model as |rental|}}
  <article class="listing">
    <h3>{{rental.title}}</h3>
    <div class="detail owner">
      <span>Owner:</span> {{rental.owner}}
    </div>
    <div class="detail type">
      <span>Type:</span> {{rental.propertyType}}
    </div>
    <div class="detail location">
      <span>Location:</span> {{rental.city}}
    </div>
    <div class="detail bedrooms">
      <span>Number of bedrooms:</span> {{rental.bedrooms}}
    </div>
  </article>
{{/each}}
```
在这里，我们使用了另一个常用的Handlebars助手`{{each}}`。这个助手将让我们循环遍历我们的模型中的每个租赁对象。  

#### 租赁列表的验收测试
要自动测试检查租赁列表是否正常，我们将创建一个测试来访问索引路线，并检查是否显示3个列表。  
在`app/templates/rentals.hbs`中，每个租赁展示包装在一个`article`元素中，并赋予它一个类型`listing`。我们将使用`listing`类型来查看页面上显示的租赁数量。  
要找到有一个类型是`listing`的元素，我们使用一个名为`find`的测试助手。该`find`函数返回与给定[CSS选择器](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors)的元素。在这种情况下，它将返回一个类型是`listing`的所有元素的数组。  
编辑测试代码`tests/acceptance/list-rentals-test.js`，将测试'should list available rentals.'的测试内容修改下面的样子：
```javascript
test('should list available rentals.', function (assert) {
  visit('/');
  andThen(function() {
    assert.equal(find('.listing').length, 3, 'should see 3 listings');
  });
});
```
再次运行`ember t -s`命令来启动验收测试。  

### 安装插件
Ember拥有丰富的插件生态系统，可轻松添加到项目中。插件可以为项目提供广泛的功能，通常可以节省时间并让您专注于项目。  
浏览插件，请访问[Ember Observer](https://emberobserver.com/)网站。它列出并分类了已经发布到NPM的Ember插件，并根据各种标准为他们分配了一个分数。  
对于超级租赁，我们将利用两个插件:[ember-cli-tutorial-style](https://github.com/toddjordan/ember-cli-tutorial-style)和[ember-cli-mirage](http://www.ember-cli-mirage.com/)。  
#### ember-cli-tutorial-style
代替复制粘贴CSS样式到超级租赁应用，安装ember-cli-tutorial-style插件可以立即为应用添加CSS样式。ember-cli-tutorial-style会生成一个叫`ember-tutorial.css`的文件并放置在应用的`vendor`目录中。  
`vendor`目录是Ember的一个特殊目录，其中可以包括被编译到应用程序中的内容。当Ember CLI执行构建时，会把`ember-tutorialCSS`文件的内容复制`vendor.css`文件中。该`vendor.css`文件被`app/index.html`引用，使得样式在运行时可用。可以自己打开`app/index.html`看看其中对于`vendor.css`的引用。    
安装`ember-cli-tutorial-style`插件：
```
$ ember install ember-cli-tutorial-style 
```
由于Ember插件是npm软件包，`ember install`请将它们安装在`node_modules`目录中，并在`package.json`中添加了条目。插件安装成功后，需要重新启动服务器。重新启动服务器(执行`ember s`)，并用浏览器访问`http://c7302.ambari.apache.org:4200`将显示下面的样子：  
![](https://guides.emberjs.com/v2.15.0/images/installing-addons/styled-super-rentals-basic.png)  

#### ember-cli-mirage
[Mirage](http://www.ember-cli-mirage.com/)是一个客户端HTTP骨架库，经常用于Ember验收测试。对于本教程，我们将使用mirage作为我们的数据源而不是传统的后端服务器。mirage将允许我们在开发应用程序时创建假数据和模拟API。  
安装mirage插件：  
```
$ ember install ember-cli-mirage
```
需要注意mirage的`config.js`文件，这是定义API端点和数据的地方。我们将遵循[JSON-API](http://jsonapi.org/)规范，这要求我们以某种方式格式化数据。我们修改`mirage/config.js`文件，以便mirage发回之前定义的租赁清单：
```javascript
export default function() {
  this.namespace = '/api';

  this.get('/rentals', function() {
    return {
      data: [{
        type: 'rentals',
        id: 'grand-old-mansion',
        attributes: {
          title: 'Grand Old Mansion',
          owner: 'Veruca Salt',
          city: 'San Francisco',
          "property-type": 'Estate',
          bedrooms: 15,
          image: 'https://upload.wikimedia.org/wikipedia/commons/c/cb/Crane_estate_(5).jpg'
        }
      }, {
        type: 'rentals',
        id: 'urban-living',
        attributes: {
          title: 'Urban Living',
          owner: 'Mike Teavee',
          city: 'Seattle',
          "property-type": 'Condo',
          bedrooms: 1,
          image: 'https://upload.wikimedia.org/wikipedia/commons/0/0e/Alfonso_13_Highrise_Tegucigalpa.jpg'
        }
      }, {
        type: 'rentals',
        id: 'downtown-charm',
        attributes: {
          title: 'Downtown Charm',
          owner: 'Violet Beauregarde',
          city: 'Portland',
          "property-type": 'Apartment',
          bedrooms: 3,
          image: 'https://upload.wikimedia.org/wikipedia/commons/f/f7/Wheeldon_Apartment_Building_-_Portland_Oregon.jpg'
        }
      }]
    };
  });
}
```
mirage会覆盖发送网络清单的javascript代码，并代替它们返回你指定的JSON串。这意味着你的开发工具中不会看到任何网络请求，而是会看到控制台中记录的JSON。我们修改`mirage/config.js`来配置Mirage，以便每当Ember Data发出GET请求`/api/rentals`时，Mirage将返回此JavaScript对象作为JSON，并且实际上没有网络请求。在mirage配置中，我们还定义了`/api`的命令空间。如果没有这个修改，应用程序导航到`/rentals`时会与Mirage冲突。  
为了使这个起作用，需要应用程序来默认向`/api`命名空间发送请求。为此，我们要生成一个应用程序适配器。[适配器](https://guides.emberjs.com/v2.15.0/models/customizing-adapters)是一个对象，[Ember Data](https://guides.emberjs.com/v2.15.0/models)用它来确定我们如何与后台进行通信。我们将在本教程的后面更详细地介绍Ember Data。现在，我们为应用程序生成一个适配器：   
```
$ ember generate adapter application
```
这个适配器(`app/adapters/application.js`)扩展了Ember Data的基础类[JSONAPIAdapter](http://emberjs.com/api/data/classes/DS.JSONAPIAdapter.html):  
```javascript
import DS from 'ember-data';

export default DS.JSONAPIAdapter.extend({
  namespace: 'api'
});
```
请注意，在本教程的这一节上，`app/routes/rentals.js`文件仍然提供数据。我们将在“使用Ember Data”一节中使用这里设置的mirage数据。  

### 创建简单组件
当用户查看我们的租借列表时，他们可能希望有一些交互式选项来帮助他们作出决定。让我们添加切换每个租赁图像大小的功能。为此，我们将使用一个组件。  
我们生成一个`rental-listing`组件来管理我们每个租赁的行为。每个组件名称中都要有一个破折号，来避免与可能的HTML元素冲突，因此命名`rental-listing`允许，受但命名`rental`不允许。  
生成一个组件：
```
$ ember g component rental-listing
installing component
  create app/components/rental-listing.js
  create app/templates/components/rental-listing.hbs
installing component-test
  create tests/integration/components/rental-listing-test.js
```
组件由两部分组成：  
- 一个定义它外观的模板（`app/templates/components/rental-listing.hbs`）
一个JavaScript源文件（`app/components/rental-listing.js`），用于定义它的行为方式。

我们的新`rental-listing`组件将管理用户看到的样子和与租赁交互。首先，我们将租赁显示详细信息从`rentals.hbs`模板中移到`rental-listing.hbs`(`app/templates/components/rental-listing.hbs`)并添加图像字段：  
```handlebars
<article class="listing">
  <img src="{{rental.image}}" alt="">
  <h3>{{rental.title}}</h3>
  <div class="detail owner">
    <span>Owner:</span> {{rental.owner}}
  </div>
  <div class="detail type">
    <span>Type:</span> {{rental.propertyType}}
  </div>
  <div class="detail location">
    <span>Location:</span> {{rental.city}}
  </div>
  <div class="detail bedrooms">
    <span>Number of bedrooms:</span> {{rental.bedrooms}}
  </div>
</article>
```
对比看看原来的`app/templates/rentals.hbs`，会发现基本上是把`each model`循环中内容定义的内容移到了组件模板中。那么现在修改`rentals.hbs`，在`each model`循环中引用新建立的组件：
```handlebars
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>Welcome!</h2>
  <p>
    We hope you find exactly what you're looking for in a place to stay.
  </p>
  {{#link-to 'about' class="button"}}
    About Us
  {{/link-to}}
</div>

{{#each model as |rentalUnit|}}
  {{rental-listing rental=rentalUnit}}
{{/each}}
```
这里我们调用名称为`rental-listing`的组件，并将其`rentalUnit`赋予组件的`rental`属性。  
用`ember s`运行超级租赁应用，通过浏览器访问`http://c7302.ambari.apache.org:4200`，会发现每个租赁附加了图像：  
![](https://guides.emberjs.com/v2.15.0/images/simple-component/app-with-images.png)  

#### 隐藏和显示图片
现在添加按用户请求显示图片的功能。  
我们使用`{{if}}`助手检查`isWide`是否为true来决定是否显示租赁图像。我们还将添加一些文本来指示可以单击图像，并使用一个锚点元素包含它，给它一个`image`类名，以便我们的测试可以找到它。  
修改组件模板文件`app/templates/components/rental-listing.hbs`：
```handlebars
<article class="listing">
  <a class="image {{if isWide "wide"}}">
    <img src="{{rental.image}}" alt="">
    <small>View Larger</small>
  </a>
  <h3>{{rental.title}}</h3>
(下略)
```
`isWide`的值来自于组件的JavaScript文件(`app/components/rental-listing.js`)。由于我们希望以小图像开始，因此把属性设置为`false`：
```javascript
import Ember from 'ember';

export default Ember.Component.extend({
  isWide: false
});
```
为了让用户可以放大图像，我们添加一个动作(action)来切换`isWide`的值，我们定义这个动作叫`toggleImageSize`。组件模板`app/templates/components/rental-listing.hbs`修改为：
```handlebars
<article class="listing">
  <a {{action 'toggleImageSize'}} class="image {{if isWide "wide"}}">
    <img src="{{rental.image}}" alt="">
    <small>View Larger</small>
  </a>
...
```
单击锚点元素将发送动作到组件。然后，Ember将进入`actions`散列并调用`toggleImageSize`函数。  
[动作散列](https://guides.emberjs.com/v2.15.0/templates/actions/)是一个包含多个函数的组件对象。当用户与UI进行交互（例如点击）时，将调用这些函数。  

我们创建`toggleImageSize`函数并切换组件上的`isWide`属性(`app/components/rental-listing.js`)：  
```javascript
import Ember from 'ember';

export default Ember.Component.extend({
  isWide: false,
  actions: {
    toggleImageSize() {
      this.toggleProperty('isWide');
    }
  }
});
```
现在，当我们在浏览器中点击图形或`View Larger`链接时，会看到大图像显示。当我们再次点击放大的图像，我们看到它变小。  

#### 集成测试
Ember组件通常通过[组件集成测试](https://guides.emberjs.com/v2.15.0/testing/testing-components/)进行测试。组件集成测试在Ember渲染引擎的上下文中验证组件的行为。当在集成测试中运行时，组件将经历其常规[渲染生命周期](https://guides.emberjs.com/v2.15.0/components/the-component-lifecycle/)，访问依赖对象，并且通过Ember解析器加载。  

我们的组件集成测试将测试两种不同的行为：  
- 组件应显示有关租赁的详细信息  
- 该组件应该在点击时切换`isWide`，以及扩展和缩小租赁照片。  
  
对于测试，我们将会向组件传递具有租赁模型所有属性的假对象。我们给变量命名为`rental`，在每个测试中，我们用`this`对象，将`rental`设置到本地范围(scope)。渲染模板可以访问本地范围内的值。
最终的`tests/integration/components/rental-listing-test.js`:  
```javascript
import { moduleForComponent, test } from 'ember-qunit';
import hbs from 'htmlbars-inline-precompile';
import Ember from 'ember';

let rental = Ember.Object.create({
  image: 'fake.png',
  title: 'test-title',
  owner: 'test-owner',
  propertyType: 'test-type',
  city: 'test-city',
  bedrooms: 3
});

moduleForComponent('rental-listing', 'Integration | Component | rental listing', {
  integration: true
});

test('should display rental details', function(assert) {
  this.set('rentalObj', rental);
  this.render(hbs`{{rental-listing rental=rentalObj}}`);
  assert.equal(this.$('.listing h3').text(), 'test-title', 'Title: test-title');
  assert.equal(this.$('.listing .owner').text().trim(), 'Owner: test-owner', 'Owner: test-owner')
});

test('should toggle wide class on click', function(assert) {
  this.set('rentalObj', rental);
  this.render(hbs`{{rental-listing rental=rentalObj}}`);
  assert.equal(this.$('.image.wide').length, 0, 'initially rendered small');
  Ember.run(() => document.querySelector('.image').click());
  assert.equal(this.$('.image.wide').length, 1, 'rendered wide after click');
  Ember.run(() => document.querySelector('.image').click());
  assert.equal(this.$('.image.wide').length, 0, 'rendered small after second click');
});
```
运行集成测试`ember t -s`，浏览器访问7357端口，显示为7个测试3个失败。  

### 创建handlebars助手
到目前为止，我们的应用程序直接显示了我们的Ember Data模型中的用户数据。随着我们的应用程序的发展，我们将会在将数据提供给用户之前进一步操纵数据。为此，Ember提供Handlebars模板助手来装饰模板中的数据。让我们使用一个Handlebars助手来让用户快速看到一个属性是“独立”还是“社区”的一部分。

生成一个rental-property-type助手：  
```
$ ember g helper rental-property-type
installing helper
  create app/helpers/rental-property-type.js
installing helper-test
  create tests/integration/helpers/rental-property-type-test.js
```
更新我们的`rental-listing`组件模板以使用新助手并传入rental.propertyType。新的模板文件(`app/templates/components/rental-listing.hbs`)：  
```handlebars
<article class="listing">
  <a {{action 'toggleImageSize'}} class="image {{if isWide "wide"}}">
    <img src="{{rental.image}}" alt="">
    <small>View Larger</small>
  </a>
  <h3>{{rental.title}}</h3>
  <div class="detail owner">
    <span>Owner:</span> {{rental.owner}}
  </div>
  <div class="detail type">
    <span>Type:</span> {{rental-property-type rental.propertyType}}
      - {{rental.propertyType}}
  </div>
  <div class="detail location">
    <span>Location:</span> {{rental.city}}
  </div>
  <div class="detail bedrooms">
    <span>Number of bedrooms:</span> {{rental.bedrooms}}
  </div>
</article>
```
下面实现`rental-property-type`助手。首先定义了一个房产社区类型数组常量(`communityPropertyTypes`)，然后定义了一个函数`rentalPropertyType`。在函数中检查传入的参数，然后参数与数组中的值匹配，说明是个社区房产(`Community`)，不匹配说明是个独立庄园(`Standalone`)。  
`app/helpers/rental-property-type.js`:
```javascript
import Ember from 'ember';

const communityPropertyTypes = [
  'Condo',
  'Townhouse',
  'Apartment'
];

export function rentalPropertyType([propertyType]) {
  if (communityPropertyTypes.includes(propertyType)) {
    return 'Community';
  }

  return 'Standalone';
}

export default Ember.Helper.helper(rentalPropertyType);
```
助手中的每个参数将被添加到一个数组中，并传递给我们的助手。例如，`{{my-helper "foo" "bar"}}`会导致`myHelper(["foo", "bar"])`。使用数组ES2015解析赋值，我们可以在数组中命名预期的参数。在上面的示例中，模板中的第一个参数将被分配给`propertyType`。这为您的助手提供了灵活的表达式界面，包括可选参数和默认值。  
通过`ember s`启动Ember服务器，然后用浏览器访问4200端口。应该看到第一个出租物业被列为“独立”，而另外两个被列为“社区”。  

#### 集成测试
修改测试代码以包含本节的内容。
`tests/integration/helpers/rental-property-type-test.js`:  
```javascript
import { moduleForComponent, test } from 'ember-qunit';
import hbs from 'htmlbars-inline-precompile';

moduleForComponent('rental-property-type', 'helper:rental-property-type', {
  integration: true
});

// Replace this with your real tests.
test('it renders', function(assert) {
  this.set('inputValue', '1234');

  this.render(hbs`{{rental-property-type inputValue}}`);

  assert.equal(this.$().text().trim(), 'Standalone');
});
```
### 使用Ember Data
目前，我们的应用程序使用硬编码的数据作为租赁列表中定义的rentals路由处理程序。随着应用程序的发展，我们希望在服务器上保留我们的租用数据，并且更容易地对数据进行高级操作，如查询。  
Ember提供了一个名为[Ember Data](https://github.com/emberjs/data)的数据管理的库来帮助处理持久的应用程序数据。  
Ember Data要求你通过扩展[DS.Model](http://emberjs.com/api/data/classes/DS.Model.html)来定义要提供给应用程序的数据的结构。  
可以使用Ember CLI生成Ember数据模型。下面生成叫`rental`的模型：
```
$ ember g model rental
installing model
  create app/models/rental.js
installing model-test
  create tests/unit/models/rental-test.js
```
我们修改`rental.js`代码来定义租赁对象的结构，与之前的硬编码的JavaScript对象数组（ 标题，所有者，城市，属性类型，图像，卧室和描述）相同的属性。通过函数[DS.attr()](http://emberjs.com/api/data/classes/DS.html#method_attr)的返回值定义属性。有关Ember数据属性的更多信息，请参阅指南中的[Defining Attributes](https://guides.emberjs.com/v2.15.0/models/defining-models/#toc_defining-attributes)一节。  
`app/models/rental.js`:  
```javascript
import DS from 'ember-data';

export default DS.Model.extend({
  title: DS.attr(),
  owner: DS.attr(),
  city: DS.attr(),
  propertyType: DS.attr(),
  image: DS.attr(),
  bedrooms: DS.attr(),
  description: DS.attr()
});
```
我们现在有一个可以用于我们的Ember Data实现的模型对象。

#### 更新模型钩子
要使用新的Ember Data Model对象，我们需要更新之前在“[模型钩子](https://github.com/wbwangk/wbwangk.github.io/wiki/ember.js#%E6%A8%A1%E5%9E%8B%E9%92%A9%E5%AD%90)”一节中定义的`model`函数。删除硬编码的JavaScript数组，并将其替换为调用[Ember Data Store服务](https://guides.emberjs.com/v2.15.0/models/#toc_the-store-and-a-single-source-of-truth)。该[存储服务](http://emberjs.com/api/data/classes/DS.Store.html)被注入到Ember的所有路线和组件。它是用于与Ember Data进行交互的主要接口。在这种情况下，需要调用存储的[findAll](http://emberjs.com/api/data/classes/DS.Store.html#method_findAll)函数，并向其提供新创建租赁模型类的名称作为参数。
`app/routes/rentals.js`:  
```javascript
import Ember from 'ember';

export default Ember.Route.extend({
  model() {
    return this.get('store').findAll('rental');
 }
});
```
当我们调用`findAll`函数时，Ember Data将尝试从`/api/rentals`获取租赁数据。回想一下，在“ [安装插件](https://github.com/wbwangk/wbwangk.github.io/wiki/ember.js#%E5%AE%89%E8%A3%85%E6%8F%92%E4%BB%B6) ”一节中，我们设置了一个适配器通过`/api`来路由数据请求。

由于我们已经在我们的开发环境中设置了Ember Mirage，所以Mirage将返回我们所要求的数据，而不会实际发出网络请求。

当我们将应用程序部署到生产服务器时，我们可能希望用Ember Data的远程服务器替换Mirage，以便与存储和检索持久化数据进行通信。远程服务器将允许在用户之间共享和更新数据。

### 构建复杂组件
当用户搜索租赁时，他可能还想将搜索范围缩小到特定城市。虽然我们的初始租赁列表组件仅显示租赁信息，但此新的过滤器组件还将允许用户以过滤条件的形式提供输入。  
首先，让我们生成新的组件`list-filter`。我们的需求是希望组件根据用户输入过滤租赁列表。  
```
$ ember g component list-filter
installing component
  create app/components/list-filter.js
  create app/templates/components/list-filter.hbs
installing component-test
  create tests/integration/components/list-filter-test.js
```
生成了一个组件模板、一个JavaScript文件和一个组件集成测试文件。

#### 为组件提供标记
在`app/templates/rentals.hbs`模板文件中，我们将添加对新`list-filter`组件的引用。  
请注意，在下面的模板中，我们用`list-filter`的开始和结束标记“包裹”了之前的租赁列表标记(`each`)。这是一个组件的[块形式](https://guides.emberjs.com/v2.15.0/components/wrapping-content-in-a-component)示例，这允许在组件内部渲染handlebars模板。  

在下面的代码中，我们将过滤器数据作为变量`rentals`传入到内部模板中。
`app/templates/rentals.hbs`:  
```handlebars
<div class="jumbo">
  <div class="right tomster"></div>
  <h2>Welcome!</h2>
  <p>
    We hope you find exactly what you're looking for in a place to stay.
  </p>
  {{#link-to 'about' class="button"}}
    About Us
  {{/link-to}}
</div>

{{#list-filter
   filter=(action 'filterByCity')
   as |rentals|}}
  <ul class="results">
    {{#each rentals as |rentalUnit|}}
      <li>{{rental-listing rental=rentalUnit}}</li>
    {{/each}}
  </ul>
{{/list-filter}}
```
#### 接受组件的输入

我们希望组件简单地提供一个输入域，和显示过滤结果的区域(`yield results`)，因此我们的模板很简单：  
`app/templates/components/list-filter.hbs`:  
```handlebars
{{input value=value
        key-up=(action 'handleFilterEntry')
        class="light"
        placeholder="Filter By City"}}
{{yield results}}
```
该模板包含一个`{{input}}`助手用于渲染输入框，用户可以在其中输入过滤使用的城市。输入框的`value`属性将与组件的`value`属性保持同步。  
以另一种说法是，输入框的`value`属性[绑定](https://guides.emberjs.com/v2.15.0/object-model/bindings/)到组件的`value`属性。如果属性更改，无论是用户输入，还是通过程序为其分配一个新值，该属性的新值将渲染到网页和体现在代码中。  
`key-up`属性将绑定到`handleFilterEntry`动作。  
这是组件的JavaScript的代码(`app/components/list-filter.js`)：  
```javascript
import Ember from 'ember';

export default Ember.Component.extend({
  classNames: ['list-filter'],
  value: '',

  init() {
    this._super(...arguments);
    this.get('filter')('').then((results) => this.set('results', results));
  },

  actions: {
    handleFilterEntry() {
      let filterInputValue = this.get('value');
      let filterAction = this.get('filter');
      filterAction(filterInputValue).then((filterResults) => this.set('results', filterResults));
    }
  }

});
```
#### 基于输入过滤数据

在上面的例子中，我们使用`init`钩子来初始化租赁列表，具体做法是以空值为过滤条件调用`filter`动作。在`handleFilterEntry`动作中，调用`filter`函数，函数参数是输入助手的value值。

该`filter`函数由调用对象传入。这是一种被称为[关闭动作](https://guides.emberjs.com/v2.15.0/components/triggering-changes-with-actions/#toc_passing-the-action-to-the-component)的模式。

注意对`then`函数的调用使用了`filter`函数的调用结果。该代码期望`filter`函数返回一个promise。[promise](http://emberjs.com/api/classes/RSVP.Promise.html)是JavaScript对象，它表示一个异步函数的结果。promise在收到时可能已经执行，也可能没有执行。为了解决这个问题，它提供了一些函数，如`then`函数可以你在promise返回结果时运行一些代码。  

要实现这个`filter`函数来实现城市租赁的实际过滤器，我们将创建一个`rentals`控制器。 控制器包含可用于其对应路由的模板的操作和属性。在我们的例子中，我们要生成一个名为`rentals`的控制器。Ember知道名称为`rentals`的控制器将应用于相同名称的路由。  

下列命令为`rentals`路由生成控制器：  
```
$ ember g controller rentals
installing controller
  create app/controllers/rentals.js
installing controller-test
  create tests/unit/controllers/rentals-test.js
```
现在定义新的控制器(`app/controllers/rentals.js`)：  
```javascript
import Ember from 'ember';

export default Ember.Controller.extend({
  actions: {
    filterByCity(param) {
      if (param !== '') {
        return this.get('store').query('rental', { city: param });
      } else {
        return this.get('store').findAll('rental');
      }
    }
  }
});
```
当用户在组件中的文本框输入时，控制器中的`filterByCity`动作会被调用。此动作取得`value`属性(来自用户的输入)，并过滤`rental`数据存储中与`value`匹配的数据。查询的结果返回给调用者。  

#### 伪造查询结果
为了使此动作正常工作，我们需要用下来代码覆盖Mirage`config.js`文件，以便它可以响应我们的查询。`rentals`的Mirage HTTP GET处理程序不再简单地返回租借列表，而是根据URL中的`city`参数中返回匹配的租赁清单。  
`mirage/config.js`:
```javascript
export default function() {
  this.namespace = '/api';

  let rentals = [{
      type: 'rentals',
      id: 'grand-old-mansion',
      attributes: {
        title: 'Grand Old Mansion',
        owner: 'Veruca Salt',
        city: 'San Francisco',
        "property-type": 'Estate',
        bedrooms: 15,
        image: 'https://upload.wikimedia.org/wikipedia/commons/c/cb/Crane_estate_(5).jpg',
        description: "This grand old mansion sits on over 100 acres of rolling hills and dense redwood forests."
      }
    }, {
      type: 'rentals',
      id: 'urban-living',
      attributes: {
        title: 'Urban Living',
        owner: 'Mike Teavee',
        city: 'Seattle',
        "property-type": 'Condo',
        bedrooms: 1,
        image: 'https://upload.wikimedia.org/wikipedia/commons/0/0e/Alfonso_13_Highrise_Tegucigalpa.jpg',
        description: "A commuters dream. This rental is within walking distance of 2 bus stops and the Metro."
      }
    }, {
      type: 'rentals',
      id: 'downtown-charm',
      attributes: {
        title: 'Downtown Charm',
        owner: 'Violet Beauregarde',
        city: 'Portland',
        "property-type": 'Apartment',
        bedrooms: 3,
        image: 'https://upload.wikimedia.org/wikipedia/commons/f/f7/Wheeldon_Apartment_Building_-_Portland_Oregon.jpg',
        description: "Convenience is at your doorstep with this charming downtown rental. Great restaurants and active night life are within a few feet."
      }
    }];

  this.get('/rentals', function(db, request) {
    if(request.queryParams.city !== undefined) {
      let filteredRentals = rentals.filter(function(i) {
        return i.attributes.city.toLowerCase().indexOf(request.queryParams.city.toLowerCase()) !== -1;
      });
      return { data: filteredRentals };
    } else {
      return { data: rentals };
    }
  });
}
```
修改了mirage配置后，在应用的首页上显示了输入框，可以按输入过滤城市。  
![](https://guides.emberjs.com/v2.15.0/images/autocomplete-component/styled-super-rentals-filter.png)  

#### 
在我们的示例中，您可能会注意到，如果快速输入结果可能与输入的当前过滤器文本不同步。这是因为我们的数据过滤功能是异步的，这意味着函数中的代码将被调度为稍后执行，而调用该函数的代码将继续执行。通常，可能使网络请求的代码设置为异步，因为服务器可能会在不同的时间返回其响应。

让我们添加一些保护代码，以避免查询结果与过滤器输入不同步。为此，我们将简单地将过滤器文本提供给过滤器函数，以便当结果返回时，我们可以将原始过滤器值与当前过滤器值进行比较。只有原始过滤器值和当前过滤器值相同，我们才会在屏幕上更新结果。
`app/controllers/rentals.js`(注释掉的是原来的代码):  
```javascript
import Ember from 'ember';

export default Ember.Controller.extend({
  actions: {
    filterByCity(param) {
      if (param !== '') {
//        return this.get('store').query('rental', { city: param });
        return this.get('store')
          .query('rental', { city: param }).then((results) => {
            return { query: param, results: results };
          });
      } else {
//        return this.get('store').findAll('rental');
        return this.get('store')
          .findAll('rental').then((results) => {
            return { query: param, results: results };
          });
      }
    }
  }
});
```
在上述`filterByCity`租赁控制器的函数中，我们添加了一个新的属性`query`，而不是像以前一样返回一系列租赁。  
`app/components/list-filter.js`:
```javascript
import Ember from 'ember';

export default Ember.Component.extend({
  classNames: ['list-filter'],
  value: '',

  init() {
    this._super(...arguments);
//    this.get('filter')('').then((results) => this.set('results', results));
    this.get('filter')('').then((allResults) => {
      this.set('results', allResults.results);
    });
  },

  actions: {
    handleFilterEntry() {
      let filterInputValue = this.get('value');
      let filterAction = this.get('filter');
//      filterAction(filterInputValue).then((filterResults) => this.set('results', filterResults));
      filterAction(filterInputValue).then((resultsObj) => {
        if (resultsObj.query === this.get('value')) {
          this.set('results', resultsObj.results);
        }
      });
    }
  }
});
```
在我们的列表过滤器组件JavaScript中，我们使用该`query`属性来比较组件的`value`属性。该`value`属性表示输入字段的最新状态。因此，我们现在检查结果与输入字段是否匹配，确保结果与用户输入保持同步。

虽然这种方法将使我们的结果顺序保持一致，但在处理多个并发任务时还需要考虑其他问题，例如[限制对服务器发出的请求数量](https://emberjs.com/api/classes/Ember.run.html#method_debounce)。为了为应用程序创建有效和强大的自动完成行为，我们建议您考虑使用[ember-concurrencyaddon](http://ember-concurrency.com/#/docs/introduction)项目。

您现在可以继续执行下一个功能，或继续测试我们新创建的过滤器组件。

#### 集成测试
`tests/integration/components/list-filter-test.js`:
```javascript
import { moduleForComponent, test } from 'ember-qunit';
import hbs from 'htmlbars-inline-precompile';
import wait from 'ember-test-helpers/wait';
import RSVP from 'rsvp';

moduleForComponent('list-filter', 'Integration | Component | filter listing', {
  integration: true
});

const ITEMS = [{city: 'San Francisco'}, {city: 'Portland'}, {city: 'Seattle'}];
const FILTERED_ITEMS = [{city: 'San Francisco'}];

test('should initially load all listings', function (assert) {
  // we want our actions to return promises, since they are potentially fetching data asynchronously
  this.on('filterByCity', () => {
    return RSVP.resolve({ results: ITEMS });
  });

  // with an integration test,
  // you can set up and use your component in the same way your application will use it.
  this.render(hbs`
    {{#list-filter filter=(action 'filterByCity') as |results|}}
      <ul>
      {{#each results as |item|}}
        <li class="city">
          {{item.city}}
        </li>
      {{/each}}
      </ul>
    {{/list-filter}}
  `);

  return wait().then(() => {
    assert.equal(this.$('.city').length, 3);
    assert.equal(this.$('.city').first().text().trim(), 'San Francisco');
  });
});
```
由于我们的组件期望过滤器过程是异步的，我们使用[Ember的RSVP库](http://emberjs.com/api/classes/RSVP.html)从我们的过滤器返回promise。 
`this.on`将提供的`filterByCity`函数添加到测试本地范围，我们可以使用它来提供给组件。  
`filterByCity`函数将被装扮成为我们组件的动作函数，实际过滤出租列表。  

测试的最后，添加了一个`wait`调用来检验返回结果。  

Ember的[wait助手](https://guides.emberjs.com/v2.15.0/testing/testing-components/#toc_waiting-on-asynchronous-behavior) 在运行给定的函数回调之前等待所有异步任务完成。它返回一个从测试返回的promise。

第一测试模拟了空值过滤，返回了所有城市的租赁列表。下面是第二个测试，它将模仿用户输入过滤条件，检测返回的租赁列表是否符合输入的城市。  
我们将为`filterByCity`动作添加一些附加功能，以返回单个租赁，`FILTERED_ITEMS`变量就是设置的过滤条件。  

我们通过`keyUp`在输入字段上生成一个事件来强制执行该操作，然后检测确保只渲染一个项目。  
`tests/integration/components/list-filter-test.js`:
```javascript
test('should update with matching listings', function (assert) {
  this.on('filterByCity', (val) => {
    if (val === '') {
      return RSVP.resolve({
        query: val,
        results: ITEMS });
    } else {
      return RSVP.resolve({
        query: val,
        results: FILTERED_ITEMS });
    }
  });

  this.render(hbs`
    {{#list-filter filter=(action 'filterByCity') as |results|}}
      <ul>
      {{#each results as |item|}}
        <li class="city">
          {{item.city}}
        </li>
      {{/each}}
      </ul>
    {{/list-filter}}
  `);

  // The keyup event here should invoke an action that will cause the list to be filtered
  this.$('.list-filter input').val('San').keyup();

  return wait().then(() => {
    assert.equal(this.$('.city').length, 1);
    assert.equal(this.$('.city').text().trim(), 'San Francisco');
  });
});
```
现在两个集成测试场景都应该能通过。您可以通过`ember t -s`命令来启动我们的测试套件来验证这一点。  

#### 验收测试
现在我们已经测试了`list-filter`组件的行为与预期的一样，让我们​​来测试一下，页面本身也可以正常地进行验收测试。我们会验证访问租借页面的用户可以在搜索字段中输入文字，并按城市缩小租赁列表。

打开我们现有的验收测试，`tests/acceptance/list-rentals-test.js`并实施标签为“should filter the list of rentals by city”的测试。
`tests/acceptance/list-rentals-test.js`:
```javascript
test('should filter the list of rentals by city.', function (assert) {
  visit('/');
  fillIn('.list-filter input', 'Seattle');
  keyEvent('.list-filter input', 'keyup', 69);
  andThen(function() {
    assert.equal(find('.listing').length, 1, 'should show 1 listing');
    assert.equal(find('.listing .location:contains("Seattle")').length, 1, 'should contain 1 listing with location Seattle');
  });
});
```
我们在测试中引入了两个新的帮手，`fillIn`和`keyEvent`。

- `fillIn`助手“填写”给定的文本到给定的选择相匹配的输入字段。  
- `keyEvent`助手发送键击事件的UI，模拟用户输入一个按键。  

在`app/components/list-filter.js`中，我们有一个被类型是`list-filter`的组件渲染出来的顶层元素。我们使用选择器在组件内定位搜索输入`.list-filter input`，因为我们知道列表过滤器组件中只有一个输入元素。

我们的测试填写“Seattle”作为搜索字段中的搜索条件，然后`keyup`使用`69`（字母`e`的按键值）的代码将事件发送到同一个字段，以模拟用户输入。

在测试中通过查找类型是`listing`的元素，定位出在本教程的“构建简单组件”部分中定义的`rental-listing`组件。

由于我们的数据在Mirage中是硬编码的，所以我们知道只有一个城市名称为“Seattle”的租金，所以我们断定数量是一个，它显示的位置被命名为“Seattle”。

测试验证在填写“Seattle”搜索输入后，租赁列表从3减少到1，显示的项目显示“Seattle”作为位置。

按原文，到现在应只剩下2个验收测试失败，但我测试3个失败。  





## 参考
[ECMAScript 6 入门](http://es6.ruanyifeng.com/)。  
[ESlint](https://eslint.org/)是一个javascript代码审查框架。  

### Testem
A test runner that makes Javascript unit testing fun.  
创建一个testem的测试目录，然后安装testem：
```
$ cd /opt && mkdir testem && cd /opt/testem
$ npm install testem -g
$ testem
```
屏幕会提示testem已经运行，监听端口是7357。这时，testem已经准备好，等待接受浏览器的访问。屏幕显示：
```
waiting for browsers...
```
用浏览器访问地址`http://c7302.ambari.apache.org:7357`。这时屏幕提示：
```
chrome 61.0
0/0 √
```
这表明测试已经完成，只是工作目录下没有任何`.js`文件，所以测试数量是0。
现在打开另外的终端窗口，进入`/opt/testem`目录，在目录下编辑一个`hello_spec.js`的文件，内容是：
```javascript
describe('hello', function(){
  it('should say hello', function(){
    expect(hello()).toBe('hello world');
  });
});
```
这是一个[Jasmine](https://jasmine.github.io/edge/introduction.html)格式的测试脚本。Testem会自动加载工作目录下的js文件。所以屏幕显示：
```
chrome 61.0
0/1 ×

hello returns "hello world"
  × ReferenceError: hello is not defined
```
这说明已经运行了测试，但被测试的对象`hello`没有定义。现在编辑一个`hello.js`文件，内容是：
```javascript
function hello(){
  return "hello world";
}
```
Testem又一次自动检测到了目录下的文件变化，并运行了测试。这次显示：
```
chrome 61.0
1/1 √

√ 1 tests complete.
```
### QUnit
[QUnit](https://qunitjs.com/): A JavaScript Unit Testing framework. 下面是快速入门。  

#### 在浏览器中
windows下编辑一个`qunit.html`：
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width">
  <title>QUnit Example</title>
  <link rel="stylesheet" href="https://code.jquery.com/qunit/qunit-2.4.0.css">
</head>
<body>
  <div id="qunit"></div>
  <div id="qunit-fixture"></div>
  <script src="https://code.jquery.com/qunit/qunit-2.4.0.js"></script>
  <script src="tests.js"></script>
</body>
</html>
```
同一目录下编辑一个js文件`tests.js`，内容是：
```
QUnit.test( "hello test", function( assert ) {
  assert.ok( 1 == "1", "Passed!" );
});
```
然后用浏览器打开`qunit.html`，可以看到浏览器中显示：
```
1 tests completed in 5 milliseconds, with 0 failed, 0 skipped, and 0 todo.
1 assertions of 1 passed, 0 failed.
```
#### 在Node中
linux下安装：
```
$ npm install -g qunitjs
$ mkdir -p /opt/qunit/test 
```
在`/opt/qunit/test`目录下创建`qunit.html`和`tests.js`，内容如上一节。然后运行：
```
$ cd /opt/quinit
$ qunit
TAP version 13
ok 1 hello test
1..1
# pass 1
# skip 0
# todo 0
# fail 0
```
qunit默认加载`test`目录下的测试。也可以用下面的方式指定测试的文件：
```
$ qunit 'test/*.js'
```
#### 断言(assert)
[cookbook原文](https://qunitjs.com/cookbook/)  
任何单元测试的基本要素都是断言。测试的作者需要表达预期的结果，并将单元测试框架与实现产生的实际值进行比较。  
QUnit内置了三种断言：ok、equal、deepEqual。  
- ok( truthy [, message ] )， 判断`truthy`的值是否为`true`
- equal( actual, expected [, message ] ) ，判断`actual == expected`是否为`true`
- deepEqual( actual, expected [, message ] )，判断`actual === expected`是否为`true`

`equal`断言的例子：
```javascript
QUnit.test( "equal test", function( assert ) {
  assert.equal( 0, 0, "Zero, Zero; equal succeeds" );
  assert.equal( "", 0, "Empty, Zero; equal succeeds" );
  assert.equal( "", "", "Empty, Empty; equal succeeds" );
  assert.equal( 0, false, "Zero, false; equal succeeds" );
 
  assert.equal( "three", 3, "Three, 3; equal fails" );
  assert.equal( null, false, "null, false; equal fails" );
});
```