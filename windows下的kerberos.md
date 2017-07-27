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

### Windows的高级话题
虽然Kerberos的Windows实现与RFC 1510中的规范兼容，但是Kerberos的Microsoft实现与MIT和Heimdal实现相差很大，以保证其自身的解释。为了提供Windows Active Directory所需的附加功能以及与旧版Windows NT工作站的向后兼容性，Windows Kerberos环境在与Unix对应的几个重要区域中不同。
#### 加密算法支持
Windows中使用的主要加密类型是基于RC4流密码，MD5-HMAC算法用于校验和字段。该加密类型被称为RC4-HMAC，并且具有可变密钥长度，以支持较弱的“导出”质量密钥长度以及更强的128位密钥长度。

Microsoft决定背后的原因是双重的：首先，与旧版Windows NT域兼容; 第二，出于政治原因。在Windows 2000的初始设计期间，DES和三重DES都不被批准从美国出口。微软希望鼓励部署Windows 2000; 因此，RC4-HMAC密码被选为默认Kerberos加密类型，因为它是用于生成较旧的NT4密码散列的相同密码。这样，当较旧的NT4域迁移到Active Directory域时，用户的密码将继续工作，无需人工干预。

Microsoft在其发行之前确实向Windows 2000添加了DES支持，而在Windows Active Directory中创建的用户具有与其帐户相关联的RC4和DES加密密钥。但是，当Active Directory中的帐户不能使用DES密钥时，有两种情况。第一种情况是上面讨论的情况，其中NT4域被转换为Windows Active Directory域。由于散列算法只能以一种方式工作，因此Windows无法将现有用户的RC4加密密钥转换为DES密钥。第二种特殊情况是创建新的Windows 2000域时。作为域创建过程的一部分，将创建管理员帐户作为新的域管理员。该帐户最初创建时只有一个RC4密钥。

为了在上述两种情况下向用户的帐户添加DES密钥，只需更改用户密码即可。当用户密码更改时，KDC将为该用户生成RC4和DES加密密钥。

请注意，即使没有为用户选中“对此帐户使用DES加密类型”复选框，则DES密钥确实存在于Active Directory数据库中（受前两种情况的限制），但KDC不会使用响应票据请求，除非复选框被激活。

更好的解决方案是其他Kerberos实现也将RC4-HMAC加密类型应用到其代码中。Heimdal最近在其发行版中增加了这种支持，MIT已经为其Kerberos 5 1.3版本引入了完整的RC4-HMAC支持。使用RC4加密类型支持，可以在Unix和Windows Kerberos领域之间实现互操作性，而不降级用于单个DES的加密密钥。

另请注意，Windows不支持标准Kerberos 5盐以外的盐。如果您的Unix KDC具有与其他盐类（如Kerberos 4盐或AFS盐）的加密密钥，以便与较旧的客户端进行互操作，则在尝试与Windows进行互操作时可能会遇到问题。在Unix KDC的配置文件中仔细重新排序加密类型可以帮助缓解此问题，但建议在与Windows进行互操作时仅使用Kerberos 5盐和加密类型。

#### 缓存登录证书

许多Windows机器是移动的，没有固定的网络连接。例如，笔记本电脑可以将大部分时间从网络中断开; 但是，即使没有网络连接，终端用户也希望登录自己的计算机。传统上，像Active Directory一样，需要成功进行本地访问Kerberos身份验证的系统需要稳定的网络连接才能运行，因为需要与KDC通信来授权登录。Microsoft通过缓存的凭据功能提供了断开登录到域帐户的功能。

当用户登录到Windows 2000，XP或2003主机时，本地安全机构（LSA）派生密码验证器，并将此验证者保存到本地计算机的注册表中。之后，如果用户与网络断开连接，或者域控制器不可用，则如果用户名和密码与存储在本地磁盘上的保存凭证相匹配，本地系统将授权访问。
缓存的凭据存储在本地机器的注册表中
HKEY_LOCAL_MACHINE\Security\Cache密钥，其中包含NL$1到NL$10的子密钥。最后10个用户的登录用户名和密码验证器存储为每个密钥的值。

请注意，我一直指的是密码验证器，而不是密码或密码哈希。密码验证器是密码哈希的散列，因此密码验证器不能用于派生原始密码或密码散列。因此，验证器包含足够的数据，可以使本地登录过程获取密码，在其上运行两次散列算法，并与验证器进行比较。
```
【小提示】  
出于安全考虑，Windows限制对本地SYSTEM帐户的此注册表项的访问。要以管理员身份查看这些密钥，请启动regedt32应用程序，导航到HKEY_LOCAL_MACHINE\Security，然后选择安全性→权限。设置管理员组的读取权限的复选框。
```
不幸的是，某些情况导致Windows系统通过缓存的凭证验证程序对用户进行身份验证时，不向最终用户提供任何指示。当用户对位于缓存中的密码验证者进行验证时，登录过程不能获取用户的任何初始票证。虽然Windows会注意到Windows域控制器再次可用并透明地获取票据时，缓存的登录凭据可能会特别成问题，当Windows设置为对非Microsoft KDC进行身份验证时，可以直接或通过与Windows的跨领域信任域。

#### 禁用缓存的凭据功能

当然，禁用缓存的凭据功能的缺点是，如果机器丢失其网络连接，并且无法再到达域控制器，则该域的任何本地登录请求都将失败。缓存登录数可以通过使用域安全策略强制为零。设置“先前登录到缓存的数量（如果域控制器不可用）”，可以在“计算机配置”→“安全设置”→“本地策略”→“安全选项”中找到）。此更改会影响域中每个成员计算机上的CachedLogonsCount注册表项。该注册表值可以在注册表项HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\WinLogon中找到。

### Windows和Unix互操作性
参考微软文档：[Kerberos 5互操作性分步指南](https://technet.microsoft.com/en-us/library/bb742433.aspx)  

