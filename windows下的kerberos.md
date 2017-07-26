如果Windows 2000/XP/2003主机是Windows域的一部分，那么实现基于Kerberos的登录和使用Kerberos身份验证的Windows服务不需要额外的工作。但是，对于Windows客户端也将连接到在Unix主机上运行的Kerberized服务的混合环境，您需要在Windows客户端上安装MIT Kerberos for Windows发行版。  
稍后将介绍混合Windows/Unix Kerberos环境的可能性和实现细节。现在，我们来看看如何从麻省理工学院安装Kerberos for Windows发行版，以及一些与之捆绑的应用程序。  
首先，从位于的MIT Kerberos主页检索最新的Kerberos for Windows发行版
(http://web.mit.edu/kerberos/www/)。本文撰写的当前非测试版是Kerberos for Windows 2.1.2。Kerberos for Windows发行版打包为几个Zip文件，因此请确保安装了WinZip或其他一些unarchiver。  
唯一必需的分布是主要的二进制分配。可能需要安装的另一个发行版是额外的可重新分发组件 - 如果您收到有关缺少DLL的错误，则需要安装Microsoft redistributable components distribution。每个分发包含一个相当深的目录结构，从以分发名称命名的目录开始。如果要安装多个发行版，则必须将其他发行版安装的二进制文件和DLL手动复制到主配件的rel目录。  
要使Kerberos库可用于所有Windows应用程序，请将Kerberos DLL复制到Windows SYSTEM32目录中：
```
C:\krb5>copy comerr32.dll gssapi32.dll kclnt32.dll krb5_32.dll krb5cc32.dll％SystemRoot％\ SYSTEM32
```
(在windows10下需要用资源管理器复制，并进行授权。我复制了所有DLL过去，有32位也有64位)  
您还需要创建一个krb5.ini配置文件。该文件与 Unix系统上的/etc/krb5.conf文件具有相同的语法，因此 从Kerberos领域的Unix系统复制krb5.conf文件就足够了。确保krb5.ini文件在SYSTEM32目录中，以便所有应用程序都可以访问它。请注意，如果应用程序目录中有一个krb5.ini文件，它将优先于系统范围的文件，并可能会导致问题。  
默认情况下，MIT Kerberos for Windows发行版默认启用其基于内存的凭据缓存，但也保留对旧版本的基于文​​件的凭据缓存的支持。内存中缓存支持的优点是不会将用户的Kerberos凭据暴露给文件系统。请注意，Kerberos凭据缓存，而现在以内存为基础，与内部Windows凭据缓存分开，由LSA（本地安全机构）维护。除了别的以外，LSA缓存还包含当前登录的用户的Kerberos TGT以及通过Windows SSPI接口获得的任何服务票证。为弥合麻省理工学院和LSA凭证缓存之间的差距，MIT Kerberos现在包括一个程序，`ms2mit`，将TGT从LSA缓存中复制到MIT凭据缓存中。这个程序运行时，
但是，仍然存在手动Kerberos登录的情况，MIT提供了一个名为leash32的图形Kerberos凭据缓存管理实用程序，允许用户从MIT凭据缓存中获取和删除Kerberos票证。Leash32还允许用户以图形方式查看他们当前的Kerberos票证。  
MIT分发还包含传统的命令行实用程序kinit ，klist 和kdestroy ，它们的行为就像他们的Unix对等体。此外，使用MIT Kerberos for Windows的所有应用程序都会荣誉KRB5CCNAME环境变量来指定备用凭据缓存位置。