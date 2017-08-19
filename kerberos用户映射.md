[参考](https://hortonworks.com/blog/fine-tune-your-apache-hadoop-security-settings/)  
kerberos用户到本地用户的配置是`core-site.xml`中的“hadoop.security.auth_to_local”。例子：
```
           hadoop.security.auth_to_local

                 RULE:[1:$1@$0](.*@YOUR.REALM)s/@.*//
                 RULE:[2:$1@$0](hdfs@.*YOUR.REALM)s/.*/hdfs/
                 DEFAULT
```
`DEFAULT`是默认值，表示把主体的第一部分直接映射为UNIX用户。  
RULE有三部分组成：**基础**、**过滤**和**替换**。
#### 基础
基础部分由数字和模板组成。数字(1或2)表示主体由几部分组成(不包括领域)。模板定义了主体各部分的转换规则：`$0`表示领域，`$1`表示主体的第1部分，`$2`表示主体的第2部分。  
例如：
```
[1:$1@$0] 将 “username@APACHE.ORG” 转换为 “username@APACHE.ORG”
[2:$1] 将 “username/admin@APACHE.ORG” 转换为 “username”
[2:$1%$2] 将 “username/admin@APACHE.ORG” 转换为 “username%admin”
```
#### 过滤
过滤是括号中的正则表达式，必须是要应用规则的生成字符串。  
例如:
```
“(.*%admin)” 将匹配以“%admin”结尾的字符串
“(.*@SOME.DOMAIN)” 将匹配以“@SOME.DOMAIN”结尾的字符串
```
#### 替换
最后，替换是将正则表达式转换为固定字符串的sed规则。
例如:
```
“s/@ACME.COM//” 表示将“@ACME.COM”删除(替换为空串)。
“s/@[A-Z]*.COM//” 表示删除以“@”开始紧跟一个大写字符串，并以“.COM”结尾的字符串。
“s/X/Y/g” 将所有的 “X”替换为“Y”
```
sed常见命令由字符`s`和三个斜杠(`/`)组成，表示将前两个斜杠之间的内容替换为后两个斜杠之间的内容。`/g`表示替换到行尾，如果仅是`/`表示替换一次。  