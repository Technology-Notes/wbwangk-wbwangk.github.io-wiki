环境：ubuntu16.4桌面版 <sup>[#](vagrant9/ubuntu16.4/u1607)</sup>。

安装nodejs、npm<sup>[#](https://github.com/wbwangk/wbwangk.github.io/wiki/ubuntu16%E4%B8%8B%E5%B8%B8%E7%94%A8%E8%BD%AF%E4%BB%B6%E5%AE%89%E8%A3%85#nodejs--npm)</sup>。
## 官方快速开始
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

### NPM Scripts
命令`webpack`（或`npx webpack`）会触发webpack的构建。如果把这个命令放入package.json的scripts，如：
```
  {
    "name": "webpack-demo",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1",
+     "build": "webpack"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
      "webpack": "^4.0.1",
      "webpack-cli": "^2.0.9",
      "lodash": "^4.17.5"
    }
  }
```
则当执行`npm run build`的时候，会自动触发webpack的构建执行。注意，在`scripts`中指定的包，就像用`npx`执行它一样的效果。

## 另一篇入门
[中文原文](https://segmentfault.com/a/1190000006178770)  

里面的这段话把webpack的功能描述的比较到位：

Webpack的工作方式是：把你的项目当做一个整体，通过一个给定的主文件（如：index.js），Webpack将从这个文件开始找到你的项目的所有依赖文件，使用loaders处理它们，最后打包为一个（或多个）浏览器可识别的JavaScript文件。

#### 解释一下entry和output
```
$ webpack --help
Usage without config file: webpack <entry> [<entry>] --output [-o] <output>
...
```
例如，本地安装的webpack（非全局安装）可以这样执行：
```
$ node_modules/.bin/webpack app/main.js public/bundle.js
```
上面第一个参数是entry，第二个就是输出了。

假如main.js是：
```javascript
const greeter = require('./Greeter.js');
document.querySelector("#root").appendChild(greeter());
```
而Greeter.js是：
```javascript
module.exports = function() {
  var greet = document.createElement('div');
  greet.textContent = "Hi there and greetings!";
  return greet;
};
```
则执行上面的webpack命令后就输出了bundle.js，将两个输入js文件的内容合并到了bundle.js中。

### 通过配置文件
如果把上面的命令行方式变成配置文件的方式，需要定义一个配置文件`webpack.config.js`:
```
module.exports = {
  entry:  __dirname + "/app/main.js",//已多次提及的唯一入口文件
  output: {
    path: __dirname + "/public",//打包后的文件存放的地方
    filename: "bundle.js"//打包后输出文件的文件名
  }
}
```
entry和output都在上面的配置文件中了。

> 注：“__dirname”是node.js中的一个全局变量，它指向`webpack.config.js`所在的目录。

### webpack-dev-server
webpack自带了一个web服务器，默认通过8080端口提供网页服务。

安装：
```
npm install --save-dev webpack-dev-server
```
执行可以通过`npx webpack-dev-server`或`node_modules/.bin/webpack-dev-server`。也可以放入到package.json中：
```json
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack",
    "server": "webpack-dev-server --open"
  },
```
然后通过`npm run server`来启动上面配置的命令。

### loaders
`Loaders`是webpack提供的最激动人心的功能之一了。

通过使用不同的loader，webpack有能力调用外部的脚本或工具，实现对不同格式的文件的处理，比如说分析转换scss为css，或者把下一代的JS文件（ES6，ES7)转换为现代浏览器兼容的JS文件，对React的开发而言，合适的Loaders可以把React的中用到的JSX文件转换为JS文件。

Loaders在webpack.config.js中的`module.rules`关键字下进行配置。其配置包括以下几方面：

- **test**：一个用以匹配loaders所处理文件的扩展名的正则表达式（必须）

- **loader**：loader的名称（必须）

- **include/exclude**:手动添加必须处理的文件（文件夹）或屏蔽不需要处理的文件（文件夹）（可选）；

- **query**：为loaders提供额外的设置选项（可选）

#### Babel
Babel其实是一个编译JavaScript的平台，它可以编译代码帮你达到以下目的：

- 让你能使用最新的JavaScript代码（ES6，ES7...），而不用管新标准是否被当前使用的浏览器完全支持；

- 让你能使用基于JavaScript进行了拓展的语言，比如React的JSX；

在webpack中配置Babel的方法如下:
```javascript
module.exports = {
    entry: __dirname + "/app/main.js",//已多次提及的唯一入口文件
    output: {
        path: __dirname + "/public",//打包后的文件存放的地方
        filename: "bundle.js"//打包后输出文件的文件名
    },
    devtool: 'eval-source-map',
    devServer: {
        contentBase: "./public",//本地服务器所加载的页面所在的目录
        historyApiFallback: true,//不跳转
        inline: true//实时刷新
    },
    module: {
        rules: [
            {
                test: /(\.jsx|\.js)$/,
                use: {
                    loader: "babel-loader",
                    options: {
                        presets: [
                            "env", "react"
                        ]
                    }
                },
                exclude: /node_modules/
            }
        ]
    }
};
```