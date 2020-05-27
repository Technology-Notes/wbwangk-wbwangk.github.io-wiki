[1](https://unix.stackexchange.com/questions/246888/how-do-i-convert-linux-man-pages-to-html-without-using-groff) 将linux手册转化为html网页，方面阅读和翻译成中文。  

### 转换工具roffit
下载perl脚本，形成转换管道，生成html文件：
```
$ cd /opt
$ git clone --depth 1 git://github.com/bagder/roffit.git
$ cd roffit
$ mkdir -p /var/www/html/man
$ zcat /usr/share/man/man1/scp.1.gz \
    | perl roffit \
    | sudo tee /var/www/html/man/scp.1.html
$ ls /var/www/html/man/scp.1.html
```
### 用httpd搭建web服务器
```
$ yum -y install httpd
$ service httpd start
$ curl localhost
```
虚机IP是192.168.73.101，在windows下用浏览器访问地址：`http://192.168.73.101/man/scp.1.html`即可。

### 更简单的办法
```
$ man scp > /var/www/html/man/scp.txt
```
然后再浏览器下用浏览器访问地址：`http://192.168.73.101/man/scp.txt`即可。只是纯文本，格式差一些。  