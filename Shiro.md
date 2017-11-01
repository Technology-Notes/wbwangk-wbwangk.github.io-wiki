本文参考了《[Apache_Shiro_reference(中文版)](http://www.java1234.com/a/javabook/javaweb/2013/1107/1027.html)》一书。  
将Apache Shiro源码克隆到本地(centos7):
```
$ git clone https://github.com/apache/shiro.git
```
shiro带有一些示范，如QuickStart，下面进入QuickStart目录，并利用mvn下载好依赖包。
```
$ cd shiro/samples/quickstart              （这个目录下有个pom.xml是maven配置文件）
$ mvn compile
```
maven会根据pom.xml中的配置下载一系列依赖包。  
进入quickstart目录后，尝试编辑`src/main/java/Quickstart.java`文件。该文件中已经示范了典型的shiro编程写法，现在将main方法的最开始放入以下代码：
```java
  public static void main(String[] args) {
    log.info("My First Apache Shiro Application");
    System.exit(0);
  }
```
然后在quickstart目录下执行：
```
$ mvn compile exec:java 
```
屏幕上会显示很多提示信息，但最后会显示：
```
2017-11-01 05:52:29,537 INFO [Quickstart] - My First Apache Shiro Application
```
shiro.ini是shiro配置文件，可以找一下它的位置。在quickstart目录下执行：
```
$ find . -name shiro.ini
./src/main/resources/shiro.ini
```
可以查看一下`src/main/java/Quickstart.java`看看shiro的基本用法。其中三行代码是最关键的：
```java
        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        SecurityManager securityManager = factory.getInstance();
        SecurityUtils.setSecurityManager(securityManager);
```
在上面的代码中：  
1. 我们使用Shiro的`IniSecurityManager`实现来提取我们的shiro.ini文件，它位于classpath的根目录。该实现反
映了Shiro对工厂设计模式的支持。`classpath:`前缀是一个资源定位符，用来告诉shiro去哪加载ini件（其
他前缀，如`url:`和`file:`也同样被支持）。
2. `factory.getInstance()`方法被调用，它来解析INI文件并返回反映该配置的`SecurityManager`实例。
3. 在这个简单的例子中，我们把`SecurityManager`设置为一个静态的（memory）单例，能够跨JVM访问。但请
注意，这是不可取的，如果你在单个的JVM只中会有不只一个启用Shiro的应用程序。对于这个简单的例子
而言，这是没有问题的，但更为复杂的应用程序环境通常将`SecurityManager`置于应用程序特定的存储中（如
在 Web 应用中的 ServletContext 或 Spring，Guice 后 JBoss DI 容器实例）。
