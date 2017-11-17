## Kettle安装
[官方安装说明](http://wiki.pentaho.com/display/EAI/01.+Installing+Kettle)，[docker镜像](https://hub.docker.com/r/schoolscout/pentaho-kettle/)  

下载一个800多M的zip文件。解压后生成了一个`data-integration`的目录，里面有多个.bat和.sh文件。启动kettle执行spoon.bat(windows)或spoon.sh(linux/unix)。kettle是java实现的，所以可以跨平台。Kettle是一个桌面应用，而不是一个Web应用。所以需要桌面系统支持。  

在ubuntu14桌面下测试出问题，ubuntu14桌面下的java只有7，看错误提示kettle是java8编译的，所以ubuntu14下运行不了。window10下，运行没有问题。 
在菜单Help-Option中可以切换语言，切换后需要重启kettle工作台。   

## Hello World  
[官方Hello World](http://wiki.pentaho.com/display/EAI/03.+Hello+World+Example)  

创建一个Tutorial目录，里面新建一个list.csv文件，内容是(第一行是标题)：
```
last_name,name
Suarez,Maria
Guimaraes,Joao
Rush,Jennifer
Ortiz,Camila
Rodriguez,Carmen
da Silva,Zoe
```
在`data-integration`目录下运行Spoon.bat启动kettle工作台。通过菜单File->New->Transformation新建一个转换，保存为hello.ktr，就保存在Tutorial目录下。  
kettle利用图形化界面的方式定义数据处理逻辑。在界面中两种关键元素是“步骤”(Step)和“跳转”(Hop)。步骤象流程图的节点，而跳转是带方向的连线。  
1. 在工作台的左侧的Design面板中，选择Input种类，将“CSV file input”拖到右侧工作区。  
2. 选择Scripting种类，将“Modified JavaScript Value”拖到右侧工作区。  
3. 选择Output种类，将“Text file output”拖到右侧工作区。（原文是XML Output，但我的环境中这种类型报错）  

现在工作区中有了三个步骤，下面用跳转(Hop)将三个步骤连接起来：  
1. 在右侧工作区中，选中步骤一  
2. 按住Shift键，将步骤一拖动到步骤二，这样就创建了两个步骤之间的跳转(Hop)    
3. 同样的办法，将步骤二和步骤三连接起来  

#### 设置步骤一
1. 双击“CSV file input”图标，打开步骤一的配置窗口
2. 将Step name命名为`name list`
3. 在Filename中点浏览按钮选择上文中创建的`list.csv`  
4. 取消选中Lazy conversion
5. 点Get Fields按钮，将列名加载到下面的表格中。Header row present单选框默认选中，表示csv的第一行是列名。  
6. 点Preview预览
7. 点Ok完成

#### 配置步骤二
1. 双击“Modified Java Script Value”图标，打开步骤二的配置窗口  
2. 将Step name命令为`Greetings`  
3. Java script窗口中输入转换脚本：  
```javascript
var msg = 'Hello, ' + name + "!";
```
这会在Java script functions面板的转换脚本分类下创建了`Script 1`脚本(可以重命名)。  
4. 点Get variables按钮，在下面的表格中添加msg变量。  
5. 点OK完成配置  

#### 配置步骤三
1. 双击“Text file output”图标打开步骤三的配置窗口  
2. 通过浏览按钮定义Filename为Hello  
3. 将Step name命名为`File with Greetings`  
4. 点OK按钮  
5. 重新保存整个工作区(hello.ktr)  

#### 验证、预览和运行
在Action菜单，分别选择verify、preview、Run，可以验证、预览和运行当前转换。   
执行结束后，在执行结果的Logging选项卡中，可以看到：
```
Spoon - The transformation has finished!!
```
然后到Tutorial目录下，可以看到生成的Hello.txt文件。Hello.txt文件的内容是：
```
msg
Hello, Maria!
Hello, Joao!
Hello, Jennifer!
Hello, Camila!
Hello, Carmen!
Hello, Zoe!
```

### 在linux下的无头运行
尝试在windows下的kettle工作台中定义`.ktr`文件，然后复制到linux，让kettle在linux下执行`.ktr`文件中定义的转换逻辑。  
将centos7下解压kettle，也创建Tutorial目录，把`hello.ktr`和`list.csv`复制到Tutorial目录下。  
在windows下，hello.ktr中定义了两个文件路径，分别是list.csv和Hello.txt的路径。这个路径都是windows的写法，类似：
```xml
<filename>E:\data-integration\Tutorial\list.csv</filename>
<name>E:\data-integration\Tutorial\Hello</name>
```
将上述路径修改成linux的格式，类似：
```xml
<filename>/opt/data-integration/Tutorial/list.csv</filename>
<name>/opt/data-integration/Tutorial/Hello</name>
```
路径要与linux的实际路径一致。  

执行下列命令：
```
$  ./pan.sh /file /opt/data-integration/Tutorial/hello.ktr /norep
...
2017/11/10 09:57:12 - Pan - Processing ended after 1 seconds.
2017/11/10 09:57:12 - hello -
2017/11/10 09:57:12 - hello - Step name list.0 ended successfully, processed 6 lines. ( 6 lines/s)
2017/11/10 09:57:12 - hello - Step Greetings.0 ended successfully, processed 6 lines. ( 6 lines/s)
2017/11/10 09:57:12 - hello - Step File with Greetings.0 ended successfully, processed 6 lines. ( 6 lines/s)
```
`/file`和`/norep`都是pan.sh的参数，参数格式有些怪。  
`/norep`参数的含义是告诉pan不连接repository。kettle的转换定义可以保存到repository，也可以保存到.ktr文件。  

## Hello World 2
[官方原文](http://wiki.pentaho.com/display/EAI/04.+Refining+Hello+World)  
引入了作业(Job)。作业由作业条目(Job Entry)组成。作业条目可能是转换或另一个作业。  
首先在kettle配置文件**kettle.properties**中定义一个变量`FILES`，`FILES`变量指出输入输出文件的地址。对于win10，配置文件位于`C:\Users\<用户>\.kettle`目录，对于linux位于`kettle.properties`目录。打开kettle.properties，在文件的最后加上：
```
FILES=E:\data-integration\Tutorial\Logs
```
要使上述配置生效，必须重启kettle工作台。  
### 创建转换  
创建一个新的转换，命名为`get_file_name`。保存的文件名为`get_file_name.ktr`。  
新转换配置成下图的样子：  
![](http://wiki.pentaho.com/download/attachments/8291384/get_file_name_transform_diagram.png?version=1&modificationDate=1243014147000)   
各个步骤分别是：  
- Input类型：Get System Info  
- Flow类型： Filter rows  
- Flow类型: Abort  
- Job类型：Set Variable  
画从Filter rows引出的跳转线会出现下拉框，随便选个值即可。   
#### 设置Get System Info步骤  
1. 双击Get System Info步骤  
2. 在表格第一行，Name列下输入**my_file**  
3. 在表格第一行，Type列下选择**command line argument 1**  
4. 点**OK**  
#### 设置Filter rows步骤  
1. 双击Filter rows步骤  
2. 输入条件：**<field>**中选择**my_file**，等号`=`替换成**IS NULL**  
3. 下游步骤选择，'true'时选**Abort**  
4. 下游步骤选择，'false'时选**Set Variable**  
5. 点**OK**  
#### 设置Abort步骤  
默认值即可。  
#### 设置Set Variable步骤  
1. 双击Set Variable步骤  
2. 点击**Get Fields**按钮，唯一的字段**my_file**显示出来，其值是**MY_FILE**。  
3. 点**OK**  

### 运行  
1. 点击Action->Run菜单  
2. 在Run对话框中，点击Arguments(legacy)按钮，输入**list**为第一个参数的值，其它参数有就删除。  
3. 点Run按钮  
在日志面板中看到下列内容：
```
Set Variables.0 - Set variable MY_FILE to value [list]
```
4. 点击Action->Run菜单，重新运行转换  
5. 在Run对话框中，点击Arguments(legacy)按钮，删除第一个参数。  
6. 点Run按钮  
日志面板中看到一些报错信息，Kettle执行到了Abort步骤。  

### 修改转换(hello.ktr)
原来的hello转换写死了输入文件名和输出文件名。现在想把hello转换修改成可以根据参数处理不同的文件。如果作业参数设为`foo`，则会读入文件`foo.csv`，输出文件是`foo_with_greetings.xml`。  
1. 用kettle打开hello.ktr文件
2. 双击第一个步骤(CSV File Input)
3. 删除之前设定的filename。按原文的说明，按**Ctrl+空格**可以调出之前在kettle.propertys 配置的**FILES**变量。但中文windows将**Ctrl+空格**占用了，无法调出变量。只能在filename输入框中输入下面的内容：
```
${FILES}/${MY_FILE}.csv
```
`FILES`是kettle.propertys中设置的工作目录，`MY_FILE`是输入文件名。  
4. 点OK  
5. 双击第3个步骤  
6. 删除之前设置的filename，在filename输入框中输入：  
```
${FILES}/${MY_FILE}_with_greetings
```
这定义了输出文件的名字。  
7. 点击`Show Filename(s)`按钮，`${FILES}`会被替换为kettle.propertys中设定的值。  
8. 点OK  
9. 拖动一个`Filter Rows`步骤到转换
10. 拖动`Filter Rows`到步骤`CSV Input`和步骤`Modified Javascript Script Value`之间的连线(Hop)上，直到连线高亮(加粗)再释放鼠标按钮。这样就将步骤`Filter Rows`串联到步骤`CSV Input`和步骤`Modified Javascript Script Value`之间。  
11. 在<field>输入框中选择`name`，在等号输入框中选择`IS NOT NULL`。  
12. 输入框`Send 'true' data to Step`和`Send 'false' data to Step`保持为空   
13. 点OK
14. 通过菜单File->Save as将当前转换保存到`Hello_with_parameters`。  
### 运行转换
新建的这个转换依赖一个环境变量`MY_FILE`，在运行转换前必须要手工定义这个值。  
1. 通过菜单Edit->Set Environment Variables，打开环境变量定义窗口
2. 定义变量`MY_FILE`的值，如list。这是一个文件名，list.csv存在。    
3. 点OK  
4. 点Run  
5. 在运行窗口的variables面板中可以看到之前涉及到的FILES和MY_FILE两个变量。点击Run。  
6. 最后看到生成的文件(list_with_greetings.txt)  

## 将kettle日志保存到数据库
[官方原文](https://help.pentaho.com/Documentation/6.0/0P0/0U0/0A0/000)  

### 创建日志数据库
为保存kettle日志创建数据库（postgresql位于c7301节点）：
```
$ sudo -u postgres psql
postgres=# CREATE USER kettle WITH PASSWORD 'vagrant';              (新建一个数据库用户hive，密码是vagrant)
CREATE ROLE
postgres=# CREATE DATABASE pdi_logging OWNER kettle;                       (创建用户数据库hive，并指定所有者为hive)
CREATE DATABASE
postgre=# \q    
```
远程连接测试：
```
$ psql -h 192.168.73.101 -U kettle -d pdi_logging
Password for user kettle: vagrant
```
### 转换日志配置
通过kettle菜单Edit->Settings->Logging->Transformation打开配置窗口。  
1. 对于Log Connection输入框，点击New按钮。输入：  
主机名称：c7301.ambari.apache.org(或ip：192.168.73.101)  
数据库名称：pdi_logging  
端口号：5432（默认）  
用户名：kettle  
密码：vagrant  

2. Log table name数入库输入pdi_log  
3. 选择写入到日志的字段  
4. 点SQL按钮，显示建表语句，并点执行按钮，从而在数据库中创建日志表  、
5. 点OK完成配置

### 完成
重新执行转换。连接到数据库查看日志数据：
```
$ psql -h 192.168.73.101 -U kettle -d pdi_logging
Password for user kettle: vagrant
pdi_logging=>\dt          (查看所有表)
         List of relations
 Schema |  Name   | Type  | Owner
--------+---------+-------+--------
 public | pdi_log | table | kettle

pdi_logging=>select * from pdi_log;
```