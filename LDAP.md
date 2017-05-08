[来源](http://archive.oreilly.com/pub/a/perl/excerpts/system-admin-with-perl/ten-minute-ldap-utorial.html)  
LDAP条目数据结构  
![](http://figures.oreilly.com/tagoreillycom20090511oreillybooks296268I_book_d1e1/figs/I_mediaobject_d1e40679-web.png)  
 - 条目(entry)  
条目由多个属性组成，属性包含了条目的数据。条目都有一个名称，叫DN。如果使用数据库属于，属性就像一条数据库记录的字段。
 - DN(distinguished name)  
DN是由逗号分隔的多个RDN组成的字符串。DN是唯一标识符，有时称为目录条目的主键。
 - RDN(relative distinguished name)  
一个RDN是由一个或多个属性名称/值对组合。例如，```cn=Jay Sekora```可以是一个RDN(cn代表"通用名称common name")。属性名是```cn```，值是```Jay Sekora```。

遍历树形去生成一个DN：  
![](http://figures.oreilly.com/tagoreillycom20090511oreillybooks296268I_book_d1e1/figs/I_mediaobject_d1e40961-web.png)  
上图中左边树形的DN是：  
```
cn=Robert Smith, l=main campus, ou=CCS, o=Hogwarts School, c=US
```
右边树形的DN是：
```
uid=rsmith, ou=system, ou=people, dc=ccs, dc=hogwarts, dc=edu
```
几个缩写的解释：  
```
ou => organizational unit, o => organization, dc => “domain component”, c =>country 
```