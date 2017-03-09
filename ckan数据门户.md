一个使用ckan(中文)的网站：http://data.sjtu.edu.cn  
用163邮箱注册了一个用户。这个网站可以体会ckan的功能。  

一个新闻：[微软开放技术（中国）携 CKAN 和 OData 技术引入基于 Azure 的开放数据平台](http://blog.csdn.net/azurechina/article/details/41078143)  

[百度文库中繁体的ckan介绍](http://wenku.baidu.com/link?url=VssJ4Y9nGsePhzBy9fLst6at4ZqndP4bLMnRy1whHjvQWqXCsJec6CwcTLBTuaM8dbMwmLKsz-krO_aBrUFGRvaHBtuDBlKz0ht6sv0_ab7)  

ckan的github库地址  
https://github.com/ckan/ckan  

ckan官网？  
ckan.org

[ckan的docker安装](https://github.com/ckan/ckan/blob/4a3b375/doc/maintaining/installing/install-using-docker.rst)

[另一个docker镜像](https://hub.docker.com/r/cygri/ckan/):
docker run -d --rm --name db ckan/postgresql
docker run -d --rm --name solr ckan/solr
docker run -d -p 82:80 --link db:db --link solr:solr cygri/ckan:2.2.1


