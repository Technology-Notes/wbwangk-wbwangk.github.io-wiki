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

8. 以ldap中的4个用户通过curl登录REST服务器并把访问令牌依次写入浏览器cookie，上传对应的业务网络卡片到REST服务器

9. 在浏览器中切换4个用户的cookie，测试其读写权限

注意：本测试中的参与者拥有两个身份文件（身份由CA维护）。如参与者importerA有`importer`和`importer0`两个身份，前者导入了composer中用于playground，后者导入composer-rest-server，用于通过REST访问composer。ldap中的用户(如importer0)用于客户端登录到REST服务器。

### 环境准备
#### 准备REST用户
将下列用户数据导入到openldap中。从而在ldap中建立了四个用户：进口商importer0、供应商supplier0、零售商retailer0、监管机构regulator0(为了省事直接放在了People这个OU下)：
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
导入ldap和测试：
```
ldapadd -x -D dc=example,dc=com -W -f food.ldif
ldapsearch -x -LLL -H ldap:/// -b dc=example,dc=com dn
```

#### 从playground创建和导出身份卡片
在playground右上角下拉菜单中点击`admin`(当前用户)进入身份管理页面。点击Issue New ID按钮颁发新的身份。参与者类型输入框有下拉提示。身份卡不导入到钱包，而是导出为文件。最后屏幕的样子：  
[(No Title)](https://github.com/wbwangk/wbwangk.github.io/raw/master/images/export-card.png)

#### 以多用户模式启动REST服务器
通过设置环境变量来告诉REST服务器使用LDAP作为认证服务：
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
                        "url": "ldap://ldap.example.com:389",
                        "bindDN": "cn=admin,dc=example,dc=com",
                        "bindCredentials": "1",
                        "searchBase": "ou=People,dc=example,dc=com",
                        "searchFilter": "(uid={{username}})"
                    }
                }
            }'
```
```
composer-rest-server
```
业务网络卡片输入`admin@food-supply`(这是playground部署的)、不用命名空间、启用认证、启用多用户、启用事件、不用TLS。  

#### 模拟登录和写入cookie
composer-rest-server默认会监听3000端口，3000端口已经被NAT映射到了windows，用浏览器访问localhost:3000端口可以查看food-supply的各种REST API。

由于启用了认证，随便点一个API，如[Get /Supplier](http://localhost:3000/explorer/#!/Supplier/Supplier_find)，都会提示401错误。屏幕上会提示用curl访问同一个API的方式，如：
```
curl -X GET --header 'Accept: application/json' 'http://localhost:3000/api/Supplier'
```
如果这样用curl访问上述API(相当于playground的Test页面中点最左边的Supplier选项卡)也会报告401错误。下面改造这个请求，加上supplier0的凭据信息来获取访问令牌：
```
curl -i -X POST http://localhost:3000/auth/ldap -H "Content-Type:application/json" -d '{"username": "supplier0", "password":"1"}'
set-cookie: access_token=s%3AhkrH3pn3ho5ZK9D6Lswpd1BeSFl0BYZwhfcAKTBbI9XBCjauEaFrwxSSl0m4wuAe.CDWuE%2FW0xBj93s3N9enWh84l3gswmqPvpM03r9oyhUA; Max-Age=1209600; Path=/; Expires=Tue, 06 Feb 2018 02:37:57 GMT
```
利用chrome的EditThisCookie插件将上述access_token添加到当前域(localhost)中。如下图：  
[(No Title)](https://github.com/wbwangk/wbwangk.github.io/raw/master/images/access_token_editthiscookie.png)  

#### 导入身份卡片
access_token写入cookie后就可以访问钱包API了。在浏览器中点击最下面的`GET /wallet`和`Try it out!`按钮，之前会报告401错误，现在显示一个空的钱包：
```
Response Body
[]
Response Code
200
```
下面上传身份卡片supplier0.card并与将卡片与ldap用户supplier0进行绑定。这种绑定是隐含的，因为cookie中是supplier0的令牌，所以上传的卡片就和它绑定了。  
点击`POST /wallet/import`，点击`选择文件`按钮，选择之前导出的`supplier0.card`。点击`Try it out!`上传并绑定。显示204状态码表示成功。

### 业务测试
这个业务过程如下(括号中是绑定的身份卡片id)：

1. 供应商A(supplier0)创建产品列表，然后将产品列表移交给进口商A(importer0)

2. 进口商A(importer0)对产品执行豁免检查，然后将产品移交给零售商A(retailer0)

3. 零售商A(retailer0)查看自己名下的商品

为了模拟上述过程，需要用访问`auth/ldap`三次，分别获取三个用户的令牌，并三次使用EditThisCookie写入cookie，分别三次上传身份卡片。

这个测试过程使用的数据仍是[BlockchainPublicRegulationFabric-Food](https://github.com/wbwangk/BlockchainPublicRegulationFabric-Food/blob/master/README.md)文档中的数据。只是现在是通过REST发出交易。

#### 创建产品列表
通过在cookie中写入访问令牌并绑定身份卡片，当前用户可以通过REST访问Composer业务网络了。

在浏览器中点击`POST /createProductListing`来新建`createProductListing`交易。在data参数中输入下列数据：
```json
{
  "$class": "composer.food.supply.createProductListing",
  "products": ["prodA,5","prodB,2"],
  "user": "resource:composer.food.supply.Supplier#supplierA"
}
```
如果正常将返回200状态码。这时到playground的测试屏幕上点最左侧`All Transactions`选项卡可以看到刚刚提交的这个交易。
产品列表id是程序自动生成的，要执行后续操作需要取得上面交易的ProductListingContractID。方法是通过API`GET /ProductListingContract`，这个API可以返回：
```
[
  {
    "$class": "composer.food.supply.ProductListingContract",
    "listingtId": "rtlh64cd2k",
(略)
```
`rtlh64cd2k`就是composer模型中的ProductListingContract.listingtId。

#### 产品列表移交给进口商

在浏览器点击API`POST /transferListing`，输入下列数据后点`Try it out!`按钮：
```json
{
  "$class": "composer.food.supply.transferListing",
  "ownerType": "supplier",
  "newOwner": "resource:composer.food.supply.Importer#importerA",
  "productListing": "resource:composer.food.supply.ProductListingContract#rtlh64cd2k"
}
```
上述交易执行后，可以通过API`GET /ProductListingContract`(即`http://localhost:3000/api/ProductListingContract`)来查询产品列表`rtlh64cd2k`目前的状态和所有者：
```
...
    "listingtId": "rtlh64cd2k",
    "status": "EXEMPTCHECKREQ",
...
 "owner": "resource:composer.food.supply.Importer#importerA",
...
```


#### 产品检验

现在产品的所有权在进口商手里。进口商需要将产品交付给监管机构进行豁免检查。切换当前用户到进口商：
```
curl -i -X POST http://localhost:3000/auth/ldap -H "Content-Type:application/json" -d '{"username": "importer0", "password":"1"}'
set-cookie: access_token=s%3AZwG4EPy5oTBpfU3kxrTh5Kr9a3z8lCSgPgX35g0TjAFQde48l19Qs3TVSEDg2LGf.4bKTV07rNeaFvUSFSfcdmFOq%2FIYM3y8oEIV%2FhbBAzZc; Max-Age=1209600; Path=/; Expires=Tue, 06 Feb 2018 05:42:32 GMT
```
将访问令牌用EditThisCookie写入浏览器的cookie。用浏览器点击`POST /wallet/import`，点击`选择文件`按钮，选择之前导出的`importer0.card`。点击`Try it out!`上传并绑定。显示204状态码表示成功。

点击API`POST /checkProducts`，输入下列数据进行商品检验：
```
{
  "$class": "composer.food.supply.checkProducts",
  "regulator": "resource:composer.food.supply.Regulator#regulatorA",
  "productListing": "resource:composer.food.supply.ProductListingContract#rtlh64cd2k"
}
```
交易完成后，可以通过API`GET /ProductListingContract`查看产品列表信息，会看到下列内容：
```
[
  {
    "$class": "composer.food.supply.ProductListingContract",
    "listingtId": "rtlh64cd2k",
    "status": "CHECKCOMPLETED",
    "products": [
      {
        "$class": "composer.food.supply.Product",
        "productId": "prodA",
        "quantity": "5",
        "countryId": "UK"
      },
      {
        "$class": "composer.food.supply.Product",
        "productId": "prodB",
        "quantity": "2",
        "countryId": "UK"
      }
    ],
    "owner": "resource:composer.food.supply.Importer#importerA",
    "supplier": "resource:composer.food.supply.Supplier#supplierA"
  }
]
```
这显示产品列表已经是检验完成状态，所有权在importerA手中。

#### 移交给零售商
当前用户仍是进口商(importer0)。现在访问API`POST /transferListing`，输入参数data是下面的内容：
```json
{
  "$class": "composer.food.supply.transferListing",
  "ownerType": "importer",
  "newOwner": "resource:composer.food.supply.Retailer#retailerA",
  "productListing": "resource:composer.food.supply.ProductListingContract#rtlh64cd2k"
}
```
提交后产品列表就移交到零售商A手中。

#### 零售商查看产品清单
参与者`retailerA`的身份ID和ldap用户都是`retailer0`。现在切换用户到该零售商。
```
curl -i -X POST http://localhost:3000/auth/ldap -H "Content-Type:application/json" -d '{"username": "retailer0", "password":"1"}'
```
把返回的access_token(`s%3ARReruFVC0JxD14O3l6b7XKbnUIvn4JwKZ6h2CG6VbIQEQi48nkQMmQVRAElJuRyz.LGInnJk1YvQ50SJRYXHTpofU8awyRoCXgklT3SdqUnc`)写入浏览器cookie。通过API[POST /wallet/import](http://localhost:3000/explorer/#!/Wallet/Card_importCard)导入`retailer0.card`。

下面通过API [GET /Retailer](http://localhost:3000/explorer/#!/Retailer/Retailer_find)来查看其名下的产品清单，返回的响应如下：
```
[
  {
    "$class": "composer.food.supply.Retailer",
    "retailerId": "retailerA",
    "products": [
      {
        "$class": "composer.food.supply.Product",
        "productId": "prodA",
        "quantity": "5",
        "countryId": "UK"
      },
      {
        "$class": "composer.food.supply.Product",
        "productId": "prodB",
        "quantity": "2",
        "countryId": "UK"
      }
    ]
  }
]
```
上面的两个产品就是通过之前的一系列交易交到零售商A手中的。

#### 权限测试

将cookie中的access_token修改成进口商A(importer0)的，即`s%3AZwG4EPy5oTBpfU3kxrTh5Kr9a3z8lCSgPgX35g0TjAFQde48l19Qs3TVSEDg2LGf.4bKTV07rNeaFvUSFSfcdmFOq%2FIYM3y8oEIV%2FhbBAzZc`，然后再访问[GET /Retailer](http://localhost:3000/explorer/#!/Retailer/Retailer_find)
