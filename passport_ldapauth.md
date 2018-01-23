## 测试一：仅认证
测试使用的LDAP服务器是[这篇](https://github.com/wbwangk/wbwangk.github.io/wiki/LDAP)文章中搭建的。地址是u1601.ambari.apache.org。LDAP服务器启动后在Composer所在的主机测试一下：
```
$ ldapsearch -x -LLL -H ldap://u1601.ambari.apache.org:389 -b dc=ambari,dc=apache,dc=org 'uid=john' cn gidNumber
dn: uid=john,ou=People,dc=ambari,dc=apache,dc=org
cn: John Doe
gidNumber: 5000
```
测试使用的业务网络档案是food-supply.bna，来自[这个库](https://github.com/wbwangk/BlockchainPublicRegulationFabric-Food)。并提前使用[这个文章](https://wbwangk.github.io/ComposerDocs/tutorials_developer-tutorial/#_9)讲到的部署好。  

#### composer-rest-server的相关环境变量设置
```
export COMPOSER_PROVIDERS='{
                "ldap": {
                    "provider": "ldap",
                    "module": "passport-ldapauth",
                    "authPath": "/auth/ldap",
                    "callbackURL": "/auth/ldap/callback",
                    "successRedirect": "/?success=true",
                    "failureRedirect": "/?failure=true",
                    "authScheme": "ldap",
                    "server": {
                        "url": "ldap://u1601.ambari.apache.org:389",
                        "bindDN": "cn=admin,dc=ambari,dc=apache,dc=org",
                        "bindCredentials": "1",
                        "searchBase": "dc=ambari,dc=apache,dc=org",
                        "searchFilter": "(uid={{username}})"
                    }
                }
            }'
```
#### 安装passport-ldapauth
passport-ldapauth的github官方库[地址](https://github.com/vesse/passport-ldapauth)。  
```
npm install -g passport-ldapauth
```
碰到错误可以加上`--unsafe-perm`参数试试。  

#### 启动composer-rest-server
用交互式启动不容易出错：
```
$ composer-rest-server
? Enter the name of the business network card to use: admin@food-supply
? Specify if you want namespaces in the generated REST API: never use namespaces
? Specify if you want to enable authentication for the REST API using Passport: Yes
? Specify if you want to enable multiple user and identity management using wallets: No
? Specify if you want to enable event publication over WebSockets: Yes
? Specify if you want to enable TLS security for the REST API: No

To restart the REST server using the same options, issue the following command:
   composer-rest-server -c admin@food-supply -n never -a true -w true
...
```
#### 测试启用认证后的composer-rest-server
在windows下用浏览器，页面能出来，但如果选择某个资产（如Supplier）后点`GET`来查询供应商清单，会报401错误。说明启用认证管用了。
下面用curl发送登录请求：
```
$ curl -i -r -X POST http://localhost:3000/auth/ldap -H "Content-Type:application/json" -d '{"username": "john", "password":"johnldap"}
{"error":{"statusCode":500,"message":"email is missing from the user profile"}}
```
john用户没有mail属性，所以要重新建一个带mail属性的用户。  
创建用户`add_user.ldif`文件：
```
dn: uid=sara,ou=People,dc=ambari,dc=apache,dc=org
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: sara
sn: yang
givenName: Sara
mail: sara@126.com
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
# ldapadd -x -D cn=admin,dc=ambari,dc=apache,dc=org -W -f add_user.ldif
Enter LDAP Password: 1 (输入安装OpenLDAP过程中创建的密码)
```
重新用curl发出登录请求：
```bash
$ curl -i -r -X POST http://localhost:3000/auth/ldap -H "Content-Type:application/json" -d '{"username": "sara", "password":"sara"}
...
set-cookie: access_token=s%3AUC0DCiT11RHjSp1eQGzlNNZgCtF27GTCu3b7mQsdHOhAmZopvZYWdaaUj06NNnpE.KSh6JKT7VTapL4vrm%2BjSgftlDTdxqCFS9TYhd9quwr0; Max-Age=1209600; Path=/; Expires=Thu, 25 Jan 2018 20:23:50 GMT
...
```
passport-ldapauth用cookie的形式返回了令牌。  
如果你的Chrome装了EditThisCookies插件，可以把上面的access_token编辑到localhost域的cookie中，然后再访问链接`localhost:3000/api/Supplier`将可以通过认证返回供应商清单了。  

也可以用curl直接带上cookie发送请求：
```
$ curl --cookie "access_token=s%3AUC0DCiT11RHjSp1eQGzlNNZgCtF27GTCu3b7mQsdHOhAmZopvZYWdaaUj06NNnpE.KSh6JKT7VTapL4vrm%2BjSgftlDTdxqCFS9TYhd9quwr0" http://localhost:3000/api/Supplier
[{"$class":"composer.food.supply.Supplier","supplierId":"string","countryId":"string","orgId":"string","firstName":"string","lastName":"string","middleName":"string","contactDetails":{"$class":"composer.base.ContactDetails","email":"string","mobilePhone":"string","office":"string","address":{"$class":"composer.base.Address","city":"string","country":"string","locality":"string","region":"string","street":"string","street2":"string","street3":"string","postalCode":"string","postOfficeBoxNumber":"string"}}}]
```

## 测试二：同时启用认证和多用户模式
### 安装OpenLDAP
[安装部署OpenLDAP服务](https://github.com/fogray/fogray/blob/gh-pages/pages/openldap.MD)  
[LDAP入门](https://imaidata.github.io/blog/ldap/)  
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
mail: sara@126.com
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
Enter LDAP Password: 123456a? (输入安装OpenLDAP过程中创建的密码)
```

## Composer-rest-server配置

执行以下命令，设置composer-rest-server环境变量：
```
export COMPOSER_AUTHENTICATION=true  # 启用身份验证
export COMPOSER_MULTIUSER=true  # 启用多用户
export COMPOSER_NAMESPACES=never # 是否使用命名空间
export COMPOSER_DATASOURCES='{
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
}'
export COMPOSER_PROVIDERS='{
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
        }'
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

地址栏输入：http://192.168.100.101:3000/api/wallet/import，<br>请求类型选择POST，在请求的Body选项卡选择form-data，输入参数name和card，name为业务网络名称，card为用户身份卡片文件；点击Send发送请求，导入成功后，进行下一步。
    
2. 用户登录

使用ldap作为composer-reset-server的身份认证时，认证请求类型为POST请求；
地址栏输入：http://192.168.100.101:3000/auth/ldap，<br>请求类型选择POST，在请求的Body选项卡选择raw(JSON格式)，输入参数:
```
{
	"username": "sara",
	"password": "sara"
}
```
username为用户名，password为密码；点击Send发送请求，登录成功后，响应中会生成三个cookie：connect.sid、access_token、userId。

3. 用户连接业务网络

地址栏输入：http://192.168.100.101:3000/api/system/ping，<br>请求类型选择GET，点击Send发送请求，该请求会携带用户登录后的cookie信息，查看响应：
```
{
    "version": "0.16.2",
    "participant": "org.hyperledger.composer.system.NetworkAdmin#sara"
}
```
测试成功

## 测试三：多用户+ACL
仍使用默认的LoopBack数据源(内存)保存用户数据（导入的卡片仅保存在内存中，即rest服务器重启后卡片会自动删除）。业务网络使用IBM提供的[一个食品工业的公共监管Fabric例子](https://github.com/wbwangk/BlockchainPublicRegulationFabric-Food)。  

#### 测试过程
1. 启动fabric-tools（自己添加了一个cli容器）

2. 在BlockchainPublicRegulationFabric-Food中构建形成food-supply.bna

3. 启动composer-playground，导入food-supply.bna

4. 根据BlockchainPublicRegulationFabric-Food的[README](https://github.com/wbwangk/BlockchainPublicRegulationFabric-Food)初始化数据，并执行基本测试

5. 在ldap中创建4个开发者用户(importer0等)

6. 在playground中新建4个身份(也叫importer0等)，并导出4个业务网络卡片

7. 启动composer-rest-server，启用身份认证和多用户

8. 
#### 准备REST用户
将下列用户数据导入到openldap中。从而在ldap中建立了四个用户：进口商importer0、供应商supplier0、零售商retailer0、监管机构regulator0：
```
dn: uid=importer0,ou=People,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: importer0
sn: Doe
givenName: importer0
mail: importer0@example.com
cn: importer0
displayName: importer0
uidNumber: 10001
gidNumber: 5001
userPassword: 1
gecos: importer0
loginShell: /bin/bash
homeDirectory: /home/importer0

dn: uid=regulator0,ou=People,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: regulator0
sn: regulator0
givenName: regulator0
mail: regulator0@example.com
cn: regulator0
displayName: regulator0
uidNumber: 10001
gidNumber: 5001
userPassword: 1
gecos: regulator0
loginShell: /bin/bash
homeDirectory: /home/regulator0

dn: uid=retailer0,ou=People,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: retailer0
sn: retailer0
givenName: retailer0
mail: retailer0@example.com
cn: retailer0
displayName: retailer0
uidNumber: 10001
gidNumber: 5001
userPassword: 1
gecos: retailer0
loginShell: /bin/bash
homeDirectory: /home/retailer0

dn: uid=supplier0,ou=People,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: supplier0
sn: supplier0
givenName: supplier0
mail: supplier0@example.com
cn: supplier0
displayName: supplier0
uidNumber: 10001
gidNumber: 5001
userPassword: 1
gecos: supplier0
loginShell: /bin/bash
homeDirectory: /home/supplier0
```
导入方式：
