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
$ ip addr (查看网卡,可以看到enp0s3等)
$ justniffer -i enp0s3 -r 
```
#### openjdk
```
$ apt install default-jdk
$ java -version
```
#### python-pip
unbuntu自带了python2.7
```
$ apt-get install python-pip
$ pip install --upgrade pip
```
#### nodejs & npm
```
$ curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
$ apt-get install -y nodejs
```
会自动安装nodejs和npm，并创建了node的符号链接。
npm的自我升级:
```
$ npm install npm@latest -g
```
#### golang
$ cd /usr/local
$ wget https://storage.googleapis.com/golang/go1.7.4.linux-amd64.tar.gz
$ tar -C /usr/local -xzf go1.7.4.linux-amd64.tar.gz
$ export GOROOT=/usr/local/go
$ export PATH=$PATH:$GOROOT/bin
如果要编译go源码,还要设置GOPATH，GOPATH下的src目录下放源码。
可以把上面的两个export添加到/root/.bashrc文件中（root用户）。
#### htpasswd
```
$ apt-get install apache2-utils
```
#### jq
jq是一个将json格式化显示的工具。
```
$ apt-get install jq
$ echo '{"name": "webb"}' | jq
{
  "name": "webb"
}
```
或直接下载运行包：
```
$ wget https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64
$ chmod +x jq-linux64
$ echo '{"name": "webb"}' | ./jq-linux64
```
#### pyresttest
```
$ apt-get install python-pycurl
$ pip install pyresttest
```
#### wscat
```
$ npm install -g wscat
$ wscat -c ws://10.0.7.107:8086 
```