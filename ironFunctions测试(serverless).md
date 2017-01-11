*[项目位于github](https://github.com/iron-io/functions)*
go/python/nodejs等语言的安装参考:[ubuntu16下常用软件安装](ubuntu16下常用软件安装)。
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