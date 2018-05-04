## 安装avataaars-geneator
(VM:u1601)  
```
$ git clone https://github.com/fangpenlin/avataaars-geneator
$ cd avataaars-geneator
$ sudo npm i yarn -g --verbose
```
之后执行`sudo npm install`报错，改成用yarn安装：
```
$ sudo yarn install
$ yarn start
Compiled successfully!

You can now view avataaars-geneator in the browser.

  Local:            http://localhost:3000/
  On Your Network:  http://10.0.2.15:3000/
```
u1601虚拟机已经通过NAT将3000映射到了宿主机上。然后在宿主机windows下用浏览器访问地址`http://192.168.16.101:3000`就可以看到与`https://getavataaars.com`类似的头像定制页面。

