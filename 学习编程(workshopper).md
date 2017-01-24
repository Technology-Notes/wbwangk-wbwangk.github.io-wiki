项目地址：https://github.com/therebelrobot/awesome-workshopper
这个库包含了一系列学习编程的应用。应用提供了一些“挑战”（考试）。通过完成挑战的方式学习编程知识。这种实战学习方式效果很好。
以[javascripting](https://github.com/sethvincent/javascripting)这个应用为例说明workshopper的理念。  
安装应用，并运行：
```
$ npm install --global javascripting
$ javascripting
```
应用打开一个全屏菜单：  
![](https://github.com/workshopper/javascripting/raw/master/screenshot.png)  
每个菜单项都是一个“挑战”，用方向键可移动光标，回车选择要挑战的菜单项。现在选择第一个菜单项“introductions”后按回车键，屏幕显示以下内容后返回到系统命令行：
```
# JAVASCRIPTING

 ## INTRODUCTION (Exercise 1 of 19)

  To keep things organized, let's create a folder for this workshop.

  Run this command to make a directory called javascripting (or something
  else if you like):

     mkdir javascripting

  Change directory into the javascripting folder:

     cd javascripting

  Create a file named introduction.js:

     touch introduction.js

  Or if you're on Windows:

     type NUL > introduction.js

  (type is part of the command!)

  Open the file in your favorite editor, and add this text:

     console.log('hello');:::

  Save the file, then check to see if your program is correct by running
  this command:

     javascripting verify introduction.js

  By the way, throughout this tutorial, you can give the file you work with
  any name you like, so if you want to use something like catsAreAwesome.js
  file for every exercise, you can do that. Just make sure to run:

     javascripting verify catsAreAwesome.js

 ─────────────────────────────────────────────────────────────────────────────
  Need help? View the README for this workshop:
  [http://github.com/sethvincent/javascripting](http://github.com/sethvincen
  t/javascripting)

 ─────────────────────────────────────────────────────────────────────────────

   » To print these instructions again, run: javascripting print
   » To execute your program in a test environment, run: javascripting run                
     program.js
   » To verify your program, run: javascripting verify program.js
   » For help run: javascripting help

```
按上述文字提示：
```
$ mkdir javascripting
$ cd javascripting
$ echo "console.log('hello');" > introduction.js        (也可用vi编辑生成introduction.js)
$ javascripting verify introduction.js
```
提示```# YOU DID IT!```，表明这次挑战成功。执行js：
```
$ javascripting run introduction.js
hello
```
再进入全屏菜单，发现INTRODUCTION这个练习已经是COMPLETED状态：
```
   JAVASCRIPTING
   Select an exercise and hit Enter to begin
   ─────────────────────────────────────────────────────────────────────────
   » INTRODUCTION                                                [COMPLETED]
   » VARIABLES                                                   [COMPLETED]
   » STRINGS
   » STRING LENGTH
   » REVISING STRINGS
   » NUMBERS
   » ROUNDING NUMBERS
   » NUMBER TO STRING
```
可以模仿着完成其他练习。