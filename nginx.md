## centos7.3下的安装
参考了nginx.org的[安装文档](http://nginx.org/en/linux_packages.html)。  
编辑`etc/yum.repos.d/nginx.repo`:
```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
```
然后执行：
```
$ yum update -y
$ yum install nginx -y
$ whereis nginx
nginx: /usr/sbin/nginx /usr/lib64/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz
```
### 配置Nginx以支持WebDav
根据whereis得知nginx的配置文件位于`/etc/nginx`。这个目录下有个nginx.conf，是nginx的主配置文件。下级目录`conf.d`的所有conf扩展名的文件都会被include进入nginx.conf。

编辑`conf.d/default.conf`，添加以下内容：
```nginx
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
            autoindex on;

            ## webdav config
            client_body_temp_path /tmp;
            dav_methods PUT DELETE MKCOL COPY MOVE;
            create_full_put_path  on;
            dav_access    group:rw  all:r;
    }
```
Nginx的root目录`/usr/share/nginx/html`默认是只读的，而上述配置文件启用了Nginx的webdav模块(ngx_http_dav_module)，这需要root目录的写权限，这样：
```
$ chmod 777 /usr/share/nginx/html
```
启动nginx：
```
$ nginx
```
下面还有两个nginx常用命令，`-T`是检测配置文件的语法，'-s reload`是重新加载nginx配置文件:
```
$ nginx -T
$ nginx -s reload
```
创建一个t.txt文件，并上传到localhost的根目录下（对应操作系统的`/usr/share/nginx/html`）:
```
$ echo "this is t.txt!" > t.txt
$ curl -T './t.txt' localhost
$ ll /usr/share/nginx/html      (可以看到t.txt上传到了这里)
$ curl localhost/t.txt
```
## Kong与nginx并存
Kong自带的Nginx的模块不全，没有webdav模块。可以象上面一样用yum安装nginx。但启动nginx要小心，最好直接指定启动目录，否则很容易起来Kong自带的Nginx。而且最好不要绑定默认的80端口，如改成8002端口。  
下面是个与Kong并存的Nginx配置文件(/etc/nginx/conf.d/default.conf)：
```nginx
server {
    listen       8002;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html;
            autoindex on;
            ## webdav config
            client_body_temp_path /tmp;
            dav_methods PUT DELETE MKCOL COPY MOVE;
            create_full_put_path  on;
            dav_access    group:rw  all:r;
    }
```
启动nginx(默认使用/etc/nginx下的配置文件)：
```
$ chmod 777 /usr/share/nginx/html
$ /usr/sbin/nginx
```
然后测试一下webdav协议：
```
$ curl -T './t.txt' localhost:8002
```
