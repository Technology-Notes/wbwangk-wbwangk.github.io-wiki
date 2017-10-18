node.js是运行在服务器端的javascript容器。  
1. 它利用chrome V8引擎来运行javascript。不是解释执行，而是编译成服务器端代码，所以速度很快。  
2. 采用事件驱动、非阻塞I/O模型。什么事非阻塞模型？类似于浏览器端的AJAX调用，当语言处理挂起操作(如I/O)时并不等待，而是继续执行。当挂起操作完成时，发出一个“事件”，会有一个回调函数来继续处理。

*（browserling.com是一个用node.js实现的浏览器模拟器）*   

node.js是在服务器端运行的，它的javascript程序不能被浏览器直接解释。如：
```
var http = require('http');
```
这是加载node.js的http模块，这个模块只存在于服务器端，浏览器不认这个。

### node.js文件目录

node.js的目录中可能存在多个.js文件，但需要分清哪些是在服务器端运行的，哪些是在浏览器端运行的。  
按node.js的约定，lib目录是服务器端目录(ember中app目录是服务器目录)，存放服务器端的js代码文件和其它服务器端文件；public是客户端目录，保存运行在浏览器的js文件和其它文件如html、css、png等(ember中dist目录是客户端目录)。上述目录貌似可以配置。   
package.json是依赖定义文件，当运行下列命令时npm会自动解析这个json文件并安装依赖的包：
```
$ npm install
```