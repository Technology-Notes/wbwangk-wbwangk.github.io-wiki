## OpenLDAP
### 安装OpenLDAP
[安装部署OpenLDAP服务](https://github.com/fogray/fogray/blob/gh-pages/pages/openldap.MD)

注：

1. 修改OpenLDAP的域名为：
```
# vim /etc/openldap/slapd.d/cn\=config/olcDatabase\=\{2\}hdb.ldif

...
olcSuffix: dc=example,dc=com
olcRootDN: cn=admin,dc=example,dc=com
olcRootPW: 123456a?
...
```
2. 修改管理员配置
```
# vim /etc/openldap/slapd.d/cn\=config/olcDatabase\=\{1\}monitor.ldif

olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=extern
 al,cn=auth" read by dn.base="cn=admin,dc=example,dc=com" read by * none #修改管理员base dn
```
### 创建OpenLDAP用户
创建用户ldif文件：
```
vi add_user.ldif

dn: uid=sara,ou=People,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: sara
sn: yang
givenName: Sara
mail: sara@126.com   # mail属性为composer必须
cn: sara yang
displayName: Sara Yang
uidNumber: 10001
gidNumber: 5001
userPassword: sara
gecos: Sara geo
loginShell: /bin/bash
homeDirectory: /home/sara
```
执行ldapadd命令，添加用户：
```
# ldapadd -x -D cn=admin,dc=example,dc=com -W -f add_user.ldif
Enter LDAP Password:  (输入安装OpenLDAP过程中创建的密码)
```

## Composer-rest-server配置
执行以下命令，设置composer-rest-server环境变量：
```
$ export COMPOSER_AUTHENTICATION=true  # 启用身份验证
$ export COMPOSER_MULTIUSER=true  # 启用多用户
$ export COMPOSER_NAMESPACES=never # 是否使用命名空间
$ export COMPOSER_DATASOURCES={
    "db": {
        "name": "db",
        "connector": "couchdb",
        "host": "192.168.100.103",
        "port": "5984",
        "username": "admin",
        "password": "123456a?",
        "database": "db",
        "debug": "true"
    }
}
$ export COMPOSER_PROVIDERS={
            "ldap": {
                "provider": "ldap",
                "module": "passport-ldapauth",
                "authPath": "/auth/ldap",
                "callbackURL": "/auth/ldap/callback",
                "successRedirect": "/",
                "failureRedirect": "/",
                "authScheme": "ldap",
                "server": {
                    "url": "ldap://192.168.100.101:389",
                    "bindDN": "cn=admin,dc=example,dc=com",
                    "bindCredentials": "123456a?",
                    "searchBase": "dc=example,dc=com",
                    "searchFilter": "(uid={{username}})"
                }
            }
        }
```

> COMPOSER_PROVIDERS：
> > provider: 身份认证的提供者
>
> > module: 使用passport的插件类型
>
> > authPath: 认证url
>
> > callbackURL: 回调url
>
> > authScheme: 认证模式
>
> > server: 身份认证服务配置，如OpenLDAP Server的连接配置
> > > url: OpenLDAP的连接url
> >
> > > bindDN: 管理员dn
> >
> > > bindCredentials: 管理员密码
> >
> > > searchBase: 查询起始点
> >
> > > searchFilter: 查询过滤设置


启动composer-rest-server：
```
$ composer-rest-server -c alice@tn
```

## 创建用户身份卡片
创建ldap用户[sara]身份证书:
```
$ composer identity request -c PeerAdmin@fogray-test-org1-only -u admin -s adminpw -d sara
```
绑定用户[sara]到业务网络[tn]，并指定参与者:
```
$ composer identity bind -c alice@tn -a org.acme.test.SampleParticipant#p001 -e sara/admin-pub.pem
```
生成用户[sara]的身份卡片：
```
$ composer card create -p connection-org1.json -u sara -n tn -c sara/admin-pub.pem -k sara/admin-priv.pem 
```
生成用户[sara]的身份卡片后，将卡片发放给用户[sara]，sara就可以使用该身份卡片访问业务网络[tn]了

## 客户端测试

用于composer-rest-sever请求响应的cookie的HttpOnly属性为false，通过curl命令无法获取cookie，所以该测试通过Postman工具进行。
测试步骤：
1. 客户端导入用户身份卡片

地址栏输入：http://192.168.100.101:3000/api/wallet/import，请求类型选择POST，在请求的Body选项卡选择form-data，输入参数name和card，name为业务网络名称，card为用户身份卡片文件；点击Send发送请求，导入成功后，进行下一步。
    
2. 用户登录

使用ldap作为composer-reset-server的身份认证时，认证请求类型为POST请求；
地址栏输入：http://192.168.100.101:3000/auth/ldap，

请求类型选择POST，在请求的Body选项卡选择raw(JSON格式)，输入参数:
```
{
	"username": "sara",
	"password": "sara"
}
```
username为用户名，password为密码；点击Send发送请求，登录成功后，响应中会生成三个cookie：connect.sid、access_token、userId。
3. 用户连接业务网络
地址栏输入：http://192.168.100.101:3000/api/system/ping，

请求类型选择GET，点击Send发送请求，该请求会携带用户登录后的cookie信息，查看响应：
```
{
    "version": "0.16.2",
    "participant": "org.hyperledger.composer.system.NetworkAdmin#sara"
}
```
测试成功