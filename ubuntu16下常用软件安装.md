#### docker
```
$ apt-key adv  --keyserver hkp://ha.pool.sks-keyservers.net:80 \
               --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
$ echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" | sudo tee /etc/apt/sources.list.d/docker.list
$ apt-get update
$ apt-get install docker-engine
$ service docker start
$ docker ps (看是否正确执行)
```
#### nginx
```
$ echo "deb http://nginx.org/packages/ubuntu/ xenial nginx" | sudo tee /etc/apt/sources.list.d/nginx.list
$ echo "deb-src http://nginx.org/packages/ubuntu/ xenial nginx" | sudo tee /etc/apt/sources.list.d/nginx.list
$ apt-get update  (会报错： public key is not available: NO_PUBKEY ABF5BD...BF62，将下面的$key替换为提示的key:ABF5BD...BF62)
$ apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $key
$ apt-get update
$ apt-get install nginx
$ nginx -t (显示有ok就是成功了)
```
#### justniffer
```
$ add-apt-repository ppa:oreste-notelli/ppa
$ apt-get update
$ apt-get install justniffer
$ ip addr (查看网卡,可以看到enp0s03等)
$ justniffer -i enp0s03 -r 
```
#### openjdk
```
$ apt install default-jdk
$ java -version
```
#### python-pip
```
apt-get install python-pip
```