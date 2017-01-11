*[项目位于github](https://github.com/iron-io/functions)*
### 命令行fn
执行下列脚本，会下载和安装命令行工具fn。
```
$ curl -sSL http://get.iron.io/fn | sh
$ whereis fn
fn: /usr/local/bin/fn
$ mkdir /opt/iron && cd /opt/iron
$ vi func.go
```
func.go的文件内容：
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
然后构建docker镜像：
```
$ fn build
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