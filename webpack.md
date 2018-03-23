环境：ubuntu16.4桌面版 <sup>[1](vagrant9/ubuntu16.4/u1607)</sup>。

安装nodejs、npm<sup>[1](https://github.com/wbwangk/wbwangk.github.io/wiki/ubuntu16%E4%B8%8B%E5%B8%B8%E7%94%A8%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85#nodejs--npm)</sup>。

### 快速开始
[原文](https://webpack.js.org/guides/getting-started/)  
```
mkdir webpack-demo && cd webpack-demo
npm init -y
npm install webpack webpack-cli --save-dev
```
`npm init`会生成`package.json`。`--save-dev`会向`package.json`的`devDependencies`中写入依赖。`node_modules`目录下是安装后的webpack、webpack-cli及其依赖的组件。

原文首先给出一个index.html的例子，在里面引用`lodash.js`。然后说这样不好，因为这个依赖包含在代码中，没有被管理。

安装lodash：
```
npm install --save lodash
```
`--save`选项使依赖保存在package.json的`dependencies`中，即lodash会打包到运行时。而`devDependencies`表示仅在开发过程中依赖（运行时不依赖）。

修改`src/index.js`(加号表示增加行，减号表示删除这行)：
```
+ import _ from 'lodash';
+
  function component() {
    var element = document.createElement('div');

-   // Lodash, currently included via a script, is required for this line to work
+   // Lodash, now imported by this script
    element.innerHTML = _.join(['Hello', 'webpack'], ' ');

    return element;
  }

  document.body.appendChild(component());
```
import并不是传统的javascript的语法（它是ECMAScript 2015的标准）。它会被webpack构建后放入dist目录下的bundle.js文件中。

修改`dist/index.html`:
```
  <!doctype html>
  <html>
   <head>
     <title>Getting Started</title>
-    <script src="https://unpkg.com/lodash@4.16.6"></script>
   </head>
   <body>
-    <script src="./src/index.js"></script>
+    <script src="dist/bundle.js"></script>
   </body>
  </html>
```
看样子webpack把index.js和index.js依赖的lodash打包到了boudle.js中。

#### 执行构建
可以执行`npx webpack`或`./node_modules/.bin/webpack`。如果全局安装webpack，甚至可以直接执行`webpack`。

实测发现webpack构建生成的输出文件不是bundle.js，而是main.js。所以`dist/index.html`应该是：
```
-    <script src="dist/bundle.js"></script>
+    <script src="main.js"></script>
```
由于测试使用的是桌面版ubuntu，所以可以直接用图形界面中的火狐浏览器打开`dist/index.html`，也可以显示'Hello webpack'。

### Modules
`import`和`export`是[ES2015](https://babeljs.io/learn-es2015/)标准语法，但不被主流浏览器支持，通过webpack可以把它们编译成浏览器支持的语法。

除了`import`和`export`，webpack并不会转译其他的[ES2015](https://babeljs.io/learn-es2015/)语法，如果你想使用这种语法，需要你自己引入一个[转译器](https://webpack.js.org/loaders/#transpiling)，如[Babel](https://babeljs.io/)（parity教程使用的这个）。引入转译器需要用到webpack的[loader](https://webpack.js.org/concepts/loaders/)系统。

### 使用配置
webpack使用[配置文件](https://webpack.js.org/concepts/configuration)来控制其行为。它的配置文件叫`webpack.config.js`，位于项目的根目录下。

编辑`webpack.config.js`:
```javascript
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```
看到这里就明白为啥webpack生成的js文件不叫builde.js而是main.js，因为没有创建这个配置文件。

使用新创建的配置文件重新执行webpack构建：
```
npx webpack --config webpack.config.js
```
会发现dist目录下有了两个几乎一样的js文件：main.js和bundle.js，main.js是上次未指定配置文件时构建的。

> 其实webpack默认就会加载`webpack.config.js`，这里放在命令行参数中是为了解释而已。

