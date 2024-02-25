---
title: linux里source、sh、bash、./有什么区别(转)
categories:
  - 读书
tags:
  - 历史
date: 2024-02-25 17:23:16
---

在linux里，source、sh、bash、./都可以执行shell script文件，那它们有什么不同吗？

1、source
```
source a.sh
```
在当前shell内去读取、执行a.sh，而a.sh不需要有"执行权限"

source命令可以简写为"."
```
. a.sh
```
注意：中间是有空格的。

 

2、sh/bash
```
sh a.sh
bash a.sh
```
都是打开一个subshell去读取、执行a.sh，而a.sh不需要有"执行权限"

通常在subshell里运行的脚本里设置变量，不会影响到父shell的。

 

3、./
```
./a.sh
#bash: ./a.sh: 权限不够
chmod +x a.sh
./a.sh
```
打开一个subshell去读取、执行a.sh，但a.sh需要有"执行权限"

可以用chmod +x添加执行权限

另外，使用./来执行的文件里有alias语句的话，shell并不会把alias别名扩展成对应的命令，要解决的话，得使用shopt命令来开启alias扩展选项
```
shopt -s expand_aliases
```

4、fork、source、exec

使用fork方式运行script时， 就是让shell(parent process)产生一个child process去执行该script，当child process结束后，会返回parent process，但parent process的环境是不会因child process的改变而改变的。
使用source方式运行script时， 就是让script在当前process内执行， 而不是产生一个child process来执行。由于所有执行结果均于当前process内完成，若script的环境有所改变， 当然也会改变当前process环境了。
使用exec方式运行script时， 它和source一样，也是让script在当前process内执行，但是process内的原代码剩下部分将被终止。同样，process内的环境随script改变而改变。
通常如果我们执行时，都是默认为fork的。

为了实践下，我们可以先建立2个sh文件，以下代码来自ChinaUnix的網中人：

1.sh

```
#!/bin/bash
A=B
echo "PID for 1.sh before exec/source/fork:$$"
export A
echo "1.sh: \$A is $A"
case $1 in
    exec)
        echo "using exec..."
        exec ./2.sh ;;
    source)
        echo "using source..."
        . ./2.sh ;;
    *)
        echo "using fork by default..."
        ./2.sh ;;
esac
echo "PID for 1.sh after exec/source/fork:$$"
echo "1.sh: \$A is $A"
```
2.sh

```
#!/bin/bash
echo "PID for 2.sh: $$"
echo "2.sh get \$A=$A from 1.sh"
A=C
export A
echo "2.sh: \$A is $A"
```
 

自己运行下，观看结果吧 :)
```
chmod +x 1.sh
chmod +x 2.sh
./1.sh fork
./1.sh source
./1.sh exec
```

参考：[linux里source、sh、bash、./有什么区别](https://www.cnblogs.com/pcat/p/5467188.html)