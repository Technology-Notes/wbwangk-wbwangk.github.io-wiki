## 文档编号
当多sheet的excel被上传到ethercalc后，文档编号会带有等号前缀，如http://ethercalc.imaicloud.com/=yy71z8e6eb 

真正的编号是等号后面的。如果要单独引用某个sheet，则文档编号后面加上点和序号，如http://ethercalc.imaicloud.com/yy71z8e6eb.1

如果仅想引用文档框架，则把等号去掉，如http://ethercalc.imaicloud.com/yy71z8e6eb，这时会显示sheet清单

## 引用其他文档中的单元格
格式是='文档编号'!单元格，如='yy71z8e6eb.1'!A2。需要注意的是，由于引用的文档有多个sheet，所以加上了'.1'表示引用哪个sheet的单元格。如果文档只有一个sheet，则直接写如='yy71z8e6eb'!B2

## 引用同文档中其他sheet的单元格
格式是='文档编号.sheet序号'!单元格，如='yy71z8e6eb.1'!A2。第二个sheet就写类似='yy71z8e6eb.2'!A2

## 导出为excel文件
除了按A1单元格旁边的下拉箭头可以导出文件外。还可以直接在URL后面增加.xlsx来导出excel文件，如在浏览器地址栏输入http://ethercalc.imaicloud.com/=yy71z8e6eb.xlsx
