[一篇深入浅出的PAM文章](https://wenku.baidu.com/view/40c039fe04a1b0717fd5dd28.html)  [1](http://wpollock.com/AUnix2/PAM-Help.htm)  
老版本PAM的配置文件是`/etc/pam.conf`的格式：
```
service module_type control_flag module_path
------- ----------- ------------ -----------
login   auth        required     pam_unix_auth.so
```
在PAM新版本中，配置文件放在`/etc/padm.d/`目录下，文件名是service名，文件内容只留下了module_type、control_flash、module_path三部分：
```
#%PAM-1.0
auth            sufficient      pam_rootok.so
```
#### LDAP
要使用LDAP，您可以：  
- 使用pam _ldap模块。  
- 使用pam_unix模块并配置NSS(name service switch)以使用LDAP。  
- 使用pam _sssd模块并配置SSSD使用LDAP。   
- 使用pam_unix模块，并将NSS配置为使用SSSD，并将SSSD其配置为使用LDAP。  
- 直接使用一些LDAP库，并绕过PAM。  
- 使用标准系统调用（绕过PAM），并将NSS配置为使用LDAP（或使用SSSD，SSSD又使用LDAP）。  

#### PAM的例子
[2](https://www.linux.com/news/understanding-pam)  
```
auth       required     /lib/security/pam_securetty.so
auth       required     /lib/security/pam_env.so
auth       sufficient   /lib/security/pam_ldap.so
auth       required     /lib/security/pam_unix.so try_first_pass
```
由于模块被按顺序调用，所以会发生什么：

1. 'pam_securetty'模块将检查其配置文件`/etc/securetty`，并查看该文件中是否列出了用于此登录的终端。如果不是，root登录将不被允许。如果您尝试以root身份登录到“坏”终端，则此模块将失败。由于它是“必需”，它仍将调用堆栈中的所有模块。但是，即使其中每一个都成功，登录也将失败。有趣的是，如果模块被列为“必需”，则操作将立即终止，并且不调用任何其他模块，而不管其状态如何。
2. “pam_env”模块将根据管理员在/etc/security/pam_env.conf中设置的内容来设置环境变量。在Redhat 9，Fedora Core 1和Mandrake 9.2 的默认设置下，此模块的配置文件实际上并没有设置任何变量。一个很好的用处可能是自动为通过SSH登录的用户设置一个DISPLAY环境变量，如果他们想要将“xterm”回到远程桌面，那么它们不必自己设置（尽管可以采取这种方式关心OpenSSH自动化）。
3. 'pam_ldap'模块将提示用户输入密码，然后检查/etc/ldap.conf中指定的ldap目录来验证用户。如果失败，如果'pam_unix'成功验证用户，操作仍然可以成功。如果pam_ldap成功，将不会调用“pam_unix”。
4. 在这种情况下，'pam_unix'模块不会提示用户输入密码。'try_first_pass'参数将告诉模块使用前面的模块给出的密码（在这种情况下是pam_ldap）。它将尝试使用标准的`getpw*`系统调用来验证用户。如果pam_unix失败，并且pam_ldap失败，操作将失败。如果pam_ldap失败，但是pam_unix成功，则操作将成功（这在root不在ldap目录中但仍在本地/etc/ passwd文件中的情况下非常有用）。