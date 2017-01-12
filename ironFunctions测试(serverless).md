*[项目位于github](https://github.com/iron-io/functions)*  
go/python/nodejs等语言的安装参考:[ubuntu16下常用软件安装](ubuntu16下常用软件安装)。本测试中使用的各种语言hello world程序位于github项目的[example/hello](https://github.com/iron-io/functions/tree/master/examples/hello)目录下。

首先启动一个ironFunctions的服务器，后面执行curl会用到：
```
docker run --rm -it --name functions --privileged -v ${pwd}/data:/app/data -p 8080:8080 iron/functions
```
### 命令行fn
执行下列脚本，会下载和安装命令行工具fn。
```
$ curl -sSL http://get.iron.io/fn | sh
$ whereis fn
fn: /usr/local/bin/fn
$ mkdir /opt/iron && cd /opt/iron
```
### go语言测试
需要准备go语言的运行环境。
在当前目录下创建一个func.go文件：
```
package main

import (
    "encoding/json"
    "fmt"
    "os"
)

type Person struct {
    Name string
}

func main() {
    p := &Person{Name: "World"}
    json.NewDecoder(os.Stdin).Decode(p)
    fmt.Printf("Hello %v!", p.Name)
}
```
初始化，我的$USERNAME=wbwang，执行下列脚本会创建func.yaml文件。
```
$ fn init $USERNAME/hello
```
查看func.yaml的文件内容，可以看到fn自动识别语言为go。
然后构建docker镜像：
```
$ fn build
Running prebuild command: docker run --rm -v /opt/iron:/go/src/github.com/x/y -w /go/src/github.com/x/y iron/go:dev go build -o func
```
在构建时，fn先启动一个镜像为iron/go:dev的docker容器来编译func.go，然后把编译后的可执行程序打包为docker镜像wbwang/hello:0.0.1。
执行func.go：
```
$ fn run
Hello World!
```
也可以直接执行构建出来的docker镜像：
```
$ docker run wbwang/hello:0.0.1
Hello World!
```
### python语言测试
删除func.go，创建一个func.py的文件，文件内容：
```
print ("python hello world!!")
```
需要额外创建一个空文件requirements.txt，否则报错。需要先安装pip（apt-get install python-pip）。
重新初始化、构建、执行：
```
$ fn init wbwang/hello -f
$ fn build
Running prebuild command: docker run --rm -v /opt/iron:/worker -w /worker iron/python:2-dev pip install -t packages -r requirements.txt
(略)
$ fn run
python hello world!!
```
### nodejs测试
删除func.py，创建func.js:
```
name = "World";
fs = require('fs');
try {
	obj = JSON.parse(fs.readFileSync('/dev/stdin').toString())
	if (obj.name != "") {
		name = obj.name
	}
} catch(e) {}
console.log("Hello", name, "from Node!");
```
构建、运行：
```
$ fn build
Sending build context to Docker daemon 4.608 kB
Step 1 : FROM iron/node
 ---> 9ca501065d18
Step 2 : WORKDIR /function
 ---> Using cache
 ---> 78d86313bd0c
Step 3 : ADD . /function/
 ---> Using cache
 ---> ea27340f45dc
Step 4 : ENTRYPOINT node func.js
 ---> Using cache
 ---> ad07428e2aaf
Successfully built ad07428e2aaf
Function wbwang/hello:0.0.1 built successfully.
$ fn fun
Hello World from Node!
```
### push镜像
```fn push```把构建出的docker镜像推送到docker hub中。在执行push命令前应使用docker login登录docker hub：
```
$ docker login -u wbwang
Password:
Login Succeeded
$ fn push
The push refers to a repository [docker.io/wbwang/hello]
1c39755406e8: Pushed
fe696dc00a19: Pushed
e67f7ef625c5: Mounted from iron/python
321db514ef85: Mounted from iron/python
6102f0d2ad33: Mounted from iron/python
0.0.1: digest: sha256:d880817fc614a072e2333150e0f2670c230c7185c52f375c6995e9515291ae1d size: 1364
Function wbwang/hello:0.0.1 pushed successfully to Docker Hub.
```
### 创建应用和路由
```
$ fn apps create myapp2
myapp2 created
$ fn routes create myapp2 /hello2
/hello2 created with wbwang/hello:0.0.1
$ curl http://localhost:8080/r/myapp2/hello2
Hello World from Node!
root@block3:/opt/iron# curl http://localhost:8080/r/myapp2/hello2
Hello World from Node!
```
利用fn创建的docker镜像的默认tag是0.0.1，而不是docker默认的latest，要注意。  
创建路由前，可以直接通过docker run命令来运行wbwang/hello:0.0.1这个镜像，但无法通过浏览器来启动它。创建路由后，就可以通过http协议来启动了。serverless应用的典型运行方式启动、处理数据、数据结果、停止。所以前面用了“启动”而不是一般说的“请求”。  