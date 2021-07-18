---
title: C语言编译原理浅析
date: 2020-06-21 13:03:47
tags:
- Linux
- gcc
- C语言
categories:
- 工程构建
- 编译原理
---
我们平时所说的程序，在Windows系统上一般是后缀为.exe，双击后即可运行的程序文件；在类Unix系统上，可执行程序没有特定的后缀名，一般输入名称即可执行（shell命令认为输入命令的第一个单词是可执行文件的名字。内置命令在PATH目录下，shell可以直接找到，所以输入名称即可；系统一般无法找到私有可执行文件，所以需要指明路径，如当前目录下执行：./hello）。
<!-- more -->

可执行程序的内部是一系列二进制形式的计算机指令和数据的集合，CPU可以直接识别，而程序员所接触的一般是高级语言，如C语言。将高级语言转化为低级机器语言指令，并将这些指令按照一定的格式进行打包，并以二进制磁盘文件的格式组成可执行文件的过程称为编译，完成这些过程的工具称为编译器，[gcc](http://gcc.gnu.org/)就是Unix系统常用的编译器。本文针对C语言，并在Linux系统上进行操作演示，以简单阐述编译原理，主要参考书籍为《[深入理解计算机系统](https://baike.baidu.com/item/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%B3%BB%E7%BB%9F/4542223?fr=aladdin)》。

## 0 编译过程
在我们初学编程的时候，初次学习的都是HelloWorld程序，这里我们就以一个最简单的例子：hello.c来阐述编译原理，代码如下：
``` c
#include <stdio.h>

#define NAME "IguoChan"

int main()
{
    /* print */
    printf("Hello %s!\n", NAME);
    return 0;
}

```
在Linux系统中输入以下指令，即可得到可执行文件hello，执行后如下：
``` bash
$ gcc -o hello hello.c && ./hello
Hello IguoChan!
```
从上貌似可以看出，使用简单的一条指令就可以将高级语言转换为可执行文件，这个过程貌似很简单，其实执行了四个阶段的程序（预处理器、编译器、汇编器和链接器），这些程序一起构成了整个的编译系统，如下图所示：
![compilation system](/img/cpls.png)

可以看出，gcc实质上不是一个单独的程序，而是多个程序的集合，因此通常称为工具链。下面，我们将借助gcc工具链对以上过程进行详细描述。

## 1 预处理（Pre-Processing）
在Linux系统中输入以下指令，预处理器将根据字符#开头的命令，修改原始C程序，并得到另一个C程序，通常以.i作为文件拓展名。
``` bash
$ gcc -E hello.c -o hello.i && vim hello.i
```
可以观察到hello.i的内容为：
``` c
# 1 "hello.c"
# 1 "<built-in>"
# 1 "<command-line>"
# 1 "hello.c"
# 1 "/usr/include/stdio.h" 1 3 4
# 28 "/usr/include/stdio.h" 3 4
# 1 "/usr/include/features.h" 1 3 4
# 324 "/usr/include/features.h" 3 4
# 1 "/usr/include/x86_64-linux-gnu/bits/predefs.h" 1 3 4
# 325 "/usr/include/features.h" 2 3 4

...

typedef unsigned char __u_char;
typedef unsigned short int __u_short;
typedef unsigned int __u_int;
typedef unsigned long int __u_long;

...

extern int printf (__const char *__restrict __format, ...);

...

# 2 "hello.c" 2



int main()
{

 printf("Hello %s!\n", "IguoChan");
 return 0;
}

```
观察以上文件中的信息，大致可以分为以下三类：

1） # linenum filename flags（如1-10行，...表示中间有省略）

此类信息表征的是头文件包含的关系，被称为行标记（linemarkes），意思是：以下行起源于filename的linenum行，文件名后有0-4个标志，分别是1-4，含义如下：
'1'：表示新文件的开始，此时行号应该是1；
'2'：表示回到一个文件（在打开一个新文件后）；
'3'：表示以下文本来自系统头文件，应该禁止某些警告；
'4'：表示应将以下文本视为包含在隐式extern "C"块中；
具体可参考[gcc官方的解释](https://gcc.gnu.org/onlinedocs/gcc-4.3.6/cpp/Preprocessor-Output.html)。

2) 各种别名、结构体定义和函数声明等（如14-21行）

这些内容都是在各头文件中的各种定义和声明，1）中即是对头文件进行解析。

3） 源代码结构主体（25-34行）

在预处理器处理源文件时，会将所有的预处理指令（#开头），譬如上述的文件包含，还有宏定义、条件编译等。在上图第32行，NAME宏已经被替换掉，且宏定义行已经被删除（但空行仍然被保留）。除此外，预处理器还会删除所有的注释，如上图31行。

## 2 编译（Compiling）
在Linux系统中输入以下指令，即可将预处理后的源程序翻译为汇编语言程序，一般以.s作为文件拓展名。
``` bash
$ gcc -S hello.i -o hello.s && vim hello.s
```
可以看到hello.s的内容为：
``` c
        .file   "hello.c"
        .section        .rodata
.LC0:
        .string "Hello %s!\n"
.LC1:
        .string "IguoChan"
        .text
.globl main
        .type   main, @function
main:
.LFB0:
        .cfi_startproc
        pushq   %rbp
        .cfi_def_cfa_offset 16
        movq    %rsp, %rbp
        .cfi_offset 6, -16
        .cfi_def_cfa_register 6
        movl    $.LC0, %eax
        movl    $.LC1, %esi
        movq    %rax, %rdi
        movl    $0, %eax
        call    printf
        movl    $0, %eax
        leave
        ret
        .cfi_endproc
.LFE0:
        .size   main, .-main
        .ident  "GCC: (Ubuntu/Linaro 4.4.7-1ubuntu2) 4.4.7"
        .section        .note.GNU-stack,"",@progbits
```
编译器将预处理完的文件进行一系列的词法分析、语法分析、语义分析及优化后产生汇编代码，这个过程是程序构建的核心部分。上图中main函数包含多条低级的机器语言指令，这些机器语言是汇编语言。汇编语言非常有用，它为不同高级语言的不同编译器提供了通用的输出语言，能看懂汇编语言，也是深入理解计算机系统的基本要求。

## 3 汇编（Assembling）
接下来，汇编器将hello.s翻译成机器语言指令，把这些指令打包成一种叫做可重定位目标程序的格式，并将结果保存在hello.o中。hello.o是二进制文件，当我们用vim查看时，看到的将是一堆乱码。在Linux中，输入以下指令，可以得到hello.o文件。
``` bash
$ gcc -c hello.s -o hello.o
```

## 4 链接（Linking）
hello程序调用了printf函数，这个函数是标准C库提供的，其函数实现于一个名为printf.o的文件中，而这个文件必须以某种方式合并到我们的hello可执行文件中，链接器就负责这种合并。在Linux中执行以下命令，就可以得到最终的可执行文件。
``` bash
$ gcc hello.o -o hello
```
值得注意的是，gcc在链接到标准C库的时候不需要手动链接，而调用其他库时均需要手动链接，譬如调用了libm.so时，需要在命令后加-lm，如下：
``` bash
$ gcc hello.o -o hello -lm
```
有关链接的详细内容，可参考我的博客：[链接](https://iguochan.github.io/2020/06/21/linking/)