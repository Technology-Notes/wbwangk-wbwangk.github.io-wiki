本文参考了《[Apache_Shiro_reference(中文版)](http://www.java1234.com/a/javabook/javaweb/2013/1107/1027.html)》一书。  

### 快速开始
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
1. 我们使用Shiro的`IniSecurityManager`实现来提取我们的shiro.ini文件，它位于classpath的根目录。该实现反映了Shiro对工厂设计模式的支持。`classpath:`前缀是一个资源定位符，用来告诉shiro去哪加载ini件（其他前缀，如`url:`和`file:`也同样被支持）。  
2. `factory.getInstance()`方法被调用，它来解析INI文件并返回反映该配置的`SecurityManager`实例。  
3. 在这个简单的例子中，我们把`SecurityManager`设置为一个静态的（memory）单例，能够跨JVM访问。但请注意，这是不可取的，如果你在单个的JVM只中会有不只一个启用Shiro的应用程序。对于这个简单的例子而言，这是没有问题的，但更为复杂的应用程序环境通常将`SecurityManager`置于应用程序特定的存储中（如在Web应用中的`ServletContext`或Spring，Guice后JBoss DI容器实例）。  

几乎在所有的环境中，你可以通过下面的调用获取当前正在执行的用户：
```java
   Subject currentUser = SecurityUtils.getSubject();
```
使用`SecurityUtils.getSubject()`，我们可以获得当前正在执行的`Subject`。`Subject`是一个安全术语，它基本上的意思是“当前正在执行的用户的特定的安全视图”。它并没有被称为"User"是因为"User"一词通常和人类相关联。在安全界，术语"Subject"可以表示为人类，而且可是第三方进程，cron job，daemon account，或其他类似的东西。它仅仅意味着“该事物目前正与软件交互”。对于大多数的意图和目的，你可以把 Subject 看成是 Shiro 的"User"概念。  

现在你拥有了一个`Subject`，你能拿它来获得当前`Subject`的会话：
```java
    Session session = currentUser.getSession();
    session.setAttribute("someKey", "aValue");
```
`Session`时Shiro的特定实现，功能与JDK的`HttpSessoins`，除了一些额外的好处以及一个巨大的区别：它不需要一个HTTP环境！  
如果在一个Web应用程序内部部署，默认的`Session`将会是基于`HttpSession`的。但，在一个非Web环境中，像这个简单的教程应用程序，Shiro将会默认自动地使用它的`Enterprise Session Management`。这意味着应用程序不再被强制使用`HttpSession`。并且，任何客户端技术现在能够共享会话数据。  
下面的代码演示了用户登录：
```java
    if (!currentUser.isAuthenticated()) {
            UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
            token.setRememberMe(true);
            currentUser.login(token);
    }
```
如果登录失败，`QuickStart.java`中还示范了登录失败的处理：
```java
        try {
            currentUser.login(token);
        } catch (UnknownAccountException uae) {
            log.info("There is no user with username of " + token.getPrincipal());
        } catch (IncorrectCredentialsException ice) {
            log.info("Password for account " + token.getPrincipal() + " was incorrect!");
        } catch (LockedAccountException lae) {
            log.info("The account for username " + token.getPrincipal() + " is locked.  " +
                    "Please contact your administrator to unlock it.");
        }
```
现在用户已经登录了，我们想知道他是谁：
```java
    log.info("User [" + currentUser.getPrincipal() + "] logged in successfully.");
```
也可以测试他们是否有特定的角色:
```java
        if (currentUser.hasRole("schwartz")) {
            log.info("May the Schwartz be with you!");
        } else {
            log.info("Hello, mere mortal.");
        }
```
我们还可以判断他们是否有权限在一个确定类型的实体上进行操作：
```java
        //test a typed permission (not instance-level)
        if (currentUser.isPermitted("lightsaber:wield")) {
            log.info("You may use a lightsaber ring.  Use it wisely.");
        } else {
            log.info("Sorry, lightsaber rings are for schwartz masters only.");
        }
```
当然，我们可以执行极其强大的实例级权限检查——判断用户是否有能力访问某一类型的特定实例的能力：
```java
        //a (very powerful) Instance Level permission:
        if (currentUser.isPermitted("winnebago:drive:eagle5")) {
            log.info("You are permitted to 'drive' the winnebago with license plate (id) 'eagle5'.  " +
                    "Here are the keys - have fun!");
        } else {
            log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
        }
```
最后，当用户完成了对应用程序的使用，他们可以注销：
```java
   //all done - log out!
   currentUser.logout();
```
[这里](https://raw.githubusercontent.com/apache/shiro/master/samples/quickstart/src/main/java/Quickstart.java)是上面`QuickStart.java`的完整代码。  

### 体系架构
在最高的概念层次，Shiro的架构有3个主要的概念：Subject，SecurityManager和Realms。  
- **Subject**：Subject实质上是一个当前执行用户的特定的安全“视图”。所有Subject实例都被绑定到（且这是必须的）一个SecurityManager上。当你与一个Subject交互时，那些交互作用转化为与SecurityManager交互的特定subject的交互作用。  
- **SecurityManager**：SecurityManager是Shiro架构的心脏，并作为一种“保护伞”对象来协调内部的安全组件共同构成一个对象图。然而，一旦SecurityManager和它的内置对象图已经配置给一个应用程序，那么它单独留下来，且应用程序开发人员几乎使用他们所有的时间来处理Subject API。  
- **Realms**：Realms担当Shiro和你的应用程序的安全数据(保存了用户和密码)之间的“桥梁”或“连接器”。Shiro可以从多个Realm中寻找用户身份或授权数据。  
Realm本质上是一个特定安全的DAO：它封装了数据源的连接详细信息。当配置Shiro时，你必须指定至少一个Realm用来进行身份验证和/或授权。SecurityManager可能配置多个 Realms，但至少有一个是必须的。    
Shiro提供了立即可用的Realms来连接一些安全数据源（即目录），如LDAP、关系数据库（JDBC）、文本配置源，像INI及属性文件，以及更多。你可以插入你自己实现的Realm实现来代表自定义的数据源。  

### 配置
[快速开始]()一章种

