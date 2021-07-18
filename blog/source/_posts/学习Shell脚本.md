---
title: 学习Shell脚本
date: 2020-07-12 10:39:00
tags:
- Linux
- Shell
categories:
- Linux
- Shell
---
操作系统是一组软件，这组软件是控制整个硬件与管理系统的监测，不会让用户随意操作，因此需要一种统一的、可以调用其它命令（也就是程序）的界面应用程序，用于调用内核提供的功能，为了和内“核”相区分，这种程序被命名为“壳”程序，即Shell，也就是命令行模式。Shell有很多版本，Linux默认是bash（Bourne Again Shell）。

但是在面对许多重复性、同质性的工作，在命令行上一次次的敲击命令显得太低效，这个时候就需要通过Shell脚本（Shell Script）去实现了。Shell脚本就是利用Shell的功能所写的一个程序，这个程序是纯文本文件，它将一些Shell的语法与命令写在里面，搭配正则表达式、管道命令与数据流重定向等功能，最终达到处理目的。
<!-- more -->

我的工作重心是Linux环境编程，代码开发是我主要的工作，但是在处理碰到的一些难题时，还是会接触到Shell脚本，因此，能读懂、修改、撰写一些简单的Shell脚本也是必要的。我系统性学习Shell脚本参照的是《[鸟哥的LINUX私房菜——基础学习篇](http://cn.linux.vbird.org/linux_basic/0340bashshell-scripts.php)》，以下很多例子也直接来源于本书，大家可以直接参考原书学习。

## 0 一个简单的例子
如下，一个简单的程序，输入名字后，程序会和你打招呼。
``` bash
vim hello.sh && chmod 755 hello.sh
#!/bin/bash
# Program:
#       Input your name, program will say hello to you.
# History:
#   2020/07/12/     |   IguoChan    |   First release

read -p "Please input your first name: " NAME
echo -e "Hello ${NAME}!"
```
以#!/bin/bash声明这个文件内使用bash的语法，这样以【#!】开头的行被称为Shebang行。如果没有设置好这行，程序可能无法执行，因为系统可能无法判断用什么Shell来执行这个程序。除此，其它以【#】开头的都是注释，一般注释最好标示功能、历史版本等内容，这也是良好的编程习惯。剩下的就是程序的代码了，和命令行模式基本相同，也很好理解，执行如下：
``` bash
$ ./hello.sh 
Please input your first name: IguoChan
Hello IguoChan!
```

## 1 脚本的执行方式
### 1.1 直接执行
使用```./hello.sh```或者```bash hello.sh```的方式执行，该脚本都会使用一个新的bash环境来执行脚本内的命令，即，脚本是在子进程的bash内执行的，执行结束后即退出子进程。当子进程完成后，在子进程内的各项变量或操作将会结束而不会传回父进程，如下，在父进程中没有NAME变量。
``` bash
$ bash hello.sh 
Please input your first name: IguoChan
Hello IguoChan!
$ echo ${NAME}

$
```
我们知道，在父进程中```export AGE=18```一个变量，```AGE```将会成为环境变量，子进程将会继承父进程的环境变量，可以打印出```AGE```的大下，但是父进程却无法使用子进程定义的环境变量，即使使用了```export NAME```，如下。
``` bash
$ export AGE=18
$ vim hello.sh 
#!/bin/bash
# Program:
#       Input your name, program will say hello to you.
# History:
#   2020/07/12/     |   IguoChan    |   First release
echo ${AGE}
read -p "Please input your first name: " NAME
echo -e "Hello ${NAME}!"
export NAME
```
执行结果如下：
``` bash
$ ./hello.sh 
18
Please input your first name: IguoChan
Hello IguoChan!
$ echo ${NAME}

$ 
```
### 1.2 利用source执行脚本
source命令只是简单地读取脚本的语句并在当前Shell里面执行，没有建立新的Shell，脚本里的所有变量也都会保存下来，我们针对第0章的脚本执行以下命令，可以看出，```NAME```变量可以打印出来。
``` bash
$ source hello.sh 
Please input your first name: IguoChan
Hello IguoChan!
$ echo ${NAME}
IguoChan
```

## 2 Shell脚本默认变量
脚本和命令一样，后面也可以带一些参数，而Shell脚本已经为这些参数设置好了一些变量名称，对应如下：
``` bash
/path/script_name opt1 opt2 opt3
        $0         $1   $2   $3 
```
执行脚本的文件名为$0，其后参数依次为$1 $2 $3... 除此之外，还有一些特殊的变量定义如下：
* $#：代表接收的参数个数，在上例中即为3；
* $@：代表【"$1""$2""$3"】，每个变量是独立的；
* $\*：代表【"$1 $2 $3"】，中间有空格，变量之间非独立的；
* $?：代表上一次命令的执行返回值；

下面这个例子用于验证以上的变量。
``` bash
$ vim test_paras.sh && chmod 755 test_paras.sh
#!/bin/bash
# Program:
#       Programs show script name, parameters, and nums of parameters...
# History:
#   2020/07/12/     |   IguoChan    |   First release
echo "The scripts name is $0"
echo "Total parameter num is $#"
echo "Whole parameter is '$@'"
echo "The first parameter is $1"
```
执行结果如下：
``` bash
$ ./test_paras.sh a b c d
The scripts name is ./test_paras.sh
Total parameter num is 4
Whole parameter is 'a b c d'
The first parameter is a
```

## 3 判断式
在Shell脚本中，我们既可以使用Shell内置命令test进行判断，也可以使用“[]”进行判断，两者的用法基本相同，这里介绍“[]”进行判断的方法。下面关于测试参数的表格截图于《[鸟哥的Linux私房菜](http://cn.linux.vbird.org/linux_basic/0340bashshell-scripts.php#dis)》。

判断符号[]的需注意以下规则：
* 在中括号[]内的每个组件都需要有空格来分隔；
* 在中括号内的变量，最好都以双引号括号起来；
* 在中括号内的常数，最好都以单或双引号括号起来；


### 3.1 文件判断
文件类型的测试的参数和意义如下图所示：
![list](/img/test_file.JPG)

如下，可测试文件是否存在。首先需要认识一下在Shell中的逻辑“与”运算符&&，它表示当前命令执行成功后才会执行后面的命令；逻辑“或”运算符||，表示当前命令失败后才会执行后面的命令；逻辑“非”运算符是!，表示把条件测试的判断结果取反。以下测试hello.sh文件是否存在，以及是否为文件路径，大家可以自己试试。


``` bash
$ [ -e "hello.sh" ] && echo "File exist"
File exist
$ [ -d "hello.sh" ] || echo "Not directory"
Not directory
```

### 3.2 整数之间的判定
整数之间的判定，测试的参数和意义如下图所示：
![list](/img/test_num.JPG)
``` bash
$ [ 3 -eq 4 ] && echo "Equal"
$ [ 3 -eq 4 ] || echo "Not equal"
Not equal
$ [ 4 -eq 4 ] && echo "Equal"
Equal
```

### 3.3 判定字符串
字符串的判定，测试的参数和意义如下图所示：
![list](/img/test_string.JPG)
``` bash
$ [ ! -z "${HAHA}" ] && echo ${HAHA}
$ HAHA="haha"
$ [ ! -z "${HAHA}" ] && echo ${HAHA}
haha
```
值得注意的是，在判断式时使用“==”和“=”的含义是相同的，在C语言中，“=”代表的是赋值，而“==”则代表的是逻辑判断之意，因此为了表达清楚，建议使用“==”为佳。

### 3.4 多重条件判断
多重条件的判定，测试的参数和意义如下图所示：
![list](/img/test_multi.JPG)
``` bash
$ [ -f hello.sh -a ! -z "${HAHA}" ] && echo "File hello.sh exist and HAHA is ${HAHA}"
File hello.sh exist and HAHA is haha
$ [ -f hello.sh -o -d /etc/fstab ] && echo "hello.sh is file or /etc/fstab is directory"
hello.sh is file or /etc/fstab is directory
$ [ -f hello.sh -a -d /etc/fstab ] && echo "hello.sh is file and /etc/fstab is directory"
```

## 4 条件判断式
### 4.1 if...then...fi
类似于C语言中的if...else，该条件式的基本形式如下：
``` bash
if [ condition1 ]; then
	command1
elif [ condition2 ]; then
	command2
else
	command3
fi
```
如下例子，通过判断用户输入，选择输出不同内容。
``` bash
$ vim ifthen.sh && chmod 755 ifthen.sh
#!/bin/bash
# Program:
#       Input your choice, program will print it
# History:
#   2020/07/12/     |   IguoChan    |   First release

read -p "Please input (Y/N): " yn
if [ "${yn}" == "Y" -o "${yn}" == "y" ]; then
    echo "OK, continue"
elif [ "${yn}" == "N" -o "${yn}" == "n" ]; then
    echo "Oh, interrupt!"
else
    echo "I don't know what your choice is"
fi
```
执行结果如下：
``` bash
$ ./ifthen.sh 
Please input (Y/N): m
I don't know what your choice is
$ ./ifthen.sh 
Please input (Y/N): y
OK, continue
```

### 4.2 case...in...esac
类似于C语言中的case，该条件式有如下格式，其中pattern内容右边是关键字“)”，“;;”表示该段落结束。
``` bash
case word1 in
	pattern1 )
		command1
		;;
	pattern2 )
		command2
		;;
esac
```
改写4.1中的例子如下，虽然例子改写成这个样子会更麻烦，但是有些其它情况下，使用case...esac会更方便。“\*”表示通配符，表示一切其它的输入，相当于C语言中default。
``` bash
$ cp ifthen.sh case.sh && vim case.sh
#!/bin/bash
# Program:
#       Input your choice, program will print it
# History:
#   2020/07/12/     |   IguoChan    |   First release

read -p "Please input (Y/N): " yn
case ${yn} in
    "y")
        echo "OK, continue"
        ;;
    "N")
        echo "OK, continue"
        ;;
    "n")
        echo "Oh, interrupt!"
        ;;
    "N")
        echo "Oh, interrupt!"
        ;;
    *)
        echo "I don't know what your choice is"
        ;;
esac
```
执行结果如下：
``` bash
$ ./case.sh 
Please input (Y/N): n
Oh, interrupt!
$ ./case.sh 
Please input (Y/N): N
OK, continue
$ ./case.sh 
Please input (Y/N): v
I don't know what your choice is
```

## 5 循环
关于循环我就不举太多例子了，大家可以自己试试，Shell脚本写起来还是比较简单的。
### 5.1 while...do...done
其一般表达式如下所示，表示当condition条件成立时，就进行循环，直到condition条件不成立。
``` bash
while [ condition ]; do
	#statements
done
```

### 5.2 until...do...done
其表达式如下，和5.1相反，表示当condition条件成立时，才终止循环，否则一直循环。
``` bash
until [ condition ]; do
	#statements
done
```

### 5.3 for...do...done
其表达式有两种形式，一种是变量在一群变量之间遍历，如下。
```bash
for i in words; do
	#statements
done
```
参照鸟哥，我们可以写一个显示userid的脚本，如下：
``` bash
$ vim userid.sh && chmod 755 userid.sh 
#!/bin/bash
# Program:
#       Input your choice, program will print it
# History:
#   2020/07/12/     |   IguoChan    |   First release

users=$(cut -d ":" -f1 /etc/passwd)
for username in ${users}
do
    id ${username}
done
```
执行结果如下：
``` bash
$ ./userid.sh 
uid=0(root) gid=0(root) groups=0(root)
uid=1(daemon) gid=1(daemon) groups=1(daemon)
...
```

另外一种就是数值处理的形式，和C语言很相似。
``` bash
for (( i = 0; i < 10; i++ )); do
	#statements
done
```

## 6 Shell脚本的括号使用
到了这里，就不得不讲一下Shell脚本中的括号运用了，不然有时候看到许多括号运用会有些疑问，详见[shell脚本中的各种括号](https://blog.csdn.net/niu91/article/details/105078818/)。
### 6.1 ()
* 放置命令：格式```$(cmd)```和``` `cmd` ```一样，即表示先执行命令，再得到结果输出；
* 初始化数组：array=(1 2 3 4)

### 6.2 (())
* 算数运算：```$((3*3+4*4))```
* 重定义

### 6.3 []
* 条件判断，可用test替代，如第3章；
* 正则表达式的一部分；
* 数组

### 6.4 [[]]
条件判断，和[]的区别是：
* &&、||、< 和 > 只能在[[]]中正常使用，如果放到[]中会报错；
* [[]] 支持算术扩展；
* [[]] 支持字符串模式匹配，[]不支持；

### 6.5 {}
1、代码块，和()是有区别的：
* {}中不会开新进程，()会开，所有脚本里的变量在()是用不了的，但{}可以；
* 格式问题，{ cmd;cmd;} (cmd;cmd) ,{}左边必须空格开头，并且cmd后面必须加;

2、字符串的替换和截断

## 7 总结
Shell脚本和许多解释性语言、Makefile等都有相似之处，易于上手，最重要的还是要多摸索，多练习，能够熟练应用就好！

另外，还有许多有用的东西，如正则表达式、管道命令和sed工具等，这里没有介绍，在写Shell脚本的时候是很有用的，也需要了解一下。