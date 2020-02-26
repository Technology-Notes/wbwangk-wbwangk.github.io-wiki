$$ x^{y^z}=(1+{\rm e}^x)^{-2xy^w} $$

阮一峰的博客文章[跨域资源共享CORS详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)对CORS的原理讲的很到位。本文主要针对如何配置Nginx以便支持CORS。  
首先创建一个将来被include的nginx配置文件片段`cors-include.conf`，然后在nginx的配置文件中引用这个文件：
```nginx
    location / {
        include cors-include.conf;
        ...
    }
```
cors-include.conf的内容如下：
```nginx
#
# Wide-open CORS config for nginx
#
     if ($request_method = 'OPTIONS') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, PUT, DELETE, MKCOL, COPY, MOVE';
        #
        # Custom headers and headers various browsers *should* be OK with but aren't
        #
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
        #
        # Tell client that this pre-flight info is valid for 20 days
        #
        add_header 'Access-Control-Max-Age' 1728000;
        add_header 'Content-Type' 'text/plain charset=UTF-8';
        add_header 'Content-Length' 0;
        return 204;
     }
     if ($request_method ~* 'POST|GET|PUT|DELETE|MKCOL|COPY|MOVE') {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, PUT, DELETE, MKCOL, COPY, MOVE';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
     }
```
下面是一个更简单的例子：
```nginx
server {
  listen        80;
  server_name   api.test.com;

  location / {

    # Simple requests
    if ($request_method ~* "(GET|POST)") {
      add_header "Access-Control-Allow-Origin"  *;
    }

    # Preflighted requests
    if ($request_method = OPTIONS ) {
      add_header "Access-Control-Allow-Origin"  *;
      add_header "Access-Control-Allow-Methods" "GET, POST, OPTIONS, HEAD";
      add_header "Access-Control-Allow-Headers" "Authorization, Origin, X-Requested-With, Content-Type, Accept";
      return 200;
    }

    ....
    # Handle request
    ....
  }
}
```