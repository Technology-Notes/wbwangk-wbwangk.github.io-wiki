先[下载rpm包](https://getkong.org/install/centos/#packages)，然后安装：
```
$ sudo yum install epel-release
$ sudo yum install kong-community-edition-0.11.0.*.noarch.rpm --nogpgcheck
```
然后安装postgreSQL9.4+。 一开始直接使用ambari带的postgres，后来才发现那是个9.2版本，不行，只好[手工装postgreSQL](https://www.postgresql.org/download/linux/redhat/)。  
```
$ yum install https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm
$ yum install postgresql96-server
$ /usr/pgsql-9.6/bin/postgresql96-setup initdb
$ systemctl enable postgresql-9.6
$ systemctl start postgresql-9.6
```
刚装上的时候，postgres默认策略是`ident`，即使用操作系统用户来认证，而我喜欢root，所以要改成`md5`。修改配置文件` /var/lib/pgsql/9.6/data/pg_hba.conf`为以下内容：
```
local   all             all                                     md5
host    all             all             127.0.0.1/32           md5
host    all             all             ::1/128                 md5
host all all 0.0.0.0 0.0.0.0 md5
```
最后一项是允许其他节点远程访问postgresql。  
修改完上述配置文件后需要重启服务：
```
$ systemctl restart postgresql-9.6
```
在postgres中为kong创建用户和数据库：
```
$ sudo -u postgres psql
postgres=# CREATE USER kong WITH PASSWORD '1';              (新建一个数据库用户hive，密码是1)
CREATE ROLE
postgres=# CREATE DATABASE kong OWNER kong;                       (创建用户数据库hive，并指定所有者为hive)
CREATE DATABASE
postgre=# \q    
```
配置kong的配置文件`/etc/kong/kong.conf`:
```
proxy_listen = 0.0.0.0:8000
admin_listen = 0.0.0.0:8001  
ssl = off
http2 = off
client_ssl = off 
admin_ssl = off
admin_http2 = off

database = postgres             # Determines which of PostgreSQL or Cassandra
pg_host = 127.0.0.1     # The PostgreSQL host to connect to.
pg_port = 5432                  # The port to connect to.
pg_user = kong                  # The username to authenticate if required.
pg_password = 1
pg_database = kong              # The database name to connect to.
pg_ssl = off                    # Toggles client-server TLS connections
```
对kong的数据库初始化：
```
$ kong migrations up -c /etc/kong/kong.conf
```
启动kong：
```
$ kong start -c /etc/kong/kong.conf
2017/09/26 07:35:44 [warn] ulimit is currently set to "1024". For better performance set it to at least "4096" using "ulimit -n"
```
编辑配置文件`/etc/security/limits.conf`，在文件的最后加上：
```
root soft nofile 4096
root hard nofile 4096
```
需要重新用root登录。然后停止kong后再启动就不出现ulimit的提示了。  
测试kong：
```
$ curl -i http://localhost:8001/
```