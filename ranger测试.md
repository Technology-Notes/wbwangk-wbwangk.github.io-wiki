ranger官网地址：ranger.apache.org  
使用vagrant VM + ubuntu1-6.10 + openjdk-8-jdk测试。
### 构建
```
$ git clone https://git-wip-us.apache.org/repos/asf/ranger.git
$ cd ranger
$ mvn clean
$ mvn -DskipTests=false clean compile package install assembly:assembly 
```
执行Maven命令前需要设置JAVA_HOME。