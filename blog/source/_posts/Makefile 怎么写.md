---
title: Makefile 怎么写
date: 2020-06-17 20:15:28
categories:
- 工程构建
- Makefile
tags: Makefile
top: 100
---
在接触Linux之前，我曾经做过一段时间的裸机嵌入式开发，那个时候我并不知道什么是Makefile，因为Windows的IDE帮我做好了构建整个工程的工作，我要做的就像是往搭好框架的房子里码砖。当接触Linux后，我就不得不自己编写Makefile来构建工程了，会不会编写Makefile也就从一个侧面说明一个工程师是否有构建大型工程的能力。这里记录一下本人的学习Makefile的过程，本文主要参考陈皓大神的《[跟我一起写Makefile](https://blog.csdn.net/haoel/article/details/2886)》，和[GNU Make](https://www.gnu.org/software/make/manual/make.html#toc-Overview-of-make)文档。
<!-- more -->

## 0 前言
Makefile所做的工作就是“自动化编译”，一旦写好只需一个make命令，整个工程即完全自动编译。本文默认编译器为Unix下gcc，以C语言的源码作为基础。在这里，我并不想长篇叙述Makefile的写法，如果想了解更详细的知识的，还请移步[陈皓大神的博客](https://blog.csdn.net/haoel)。

首先我们需要知道程序的编译和链接的相关知识，这个可以参考我的博客《[C语言编译原理浅析](https://iguochan.github.io/2020/06/21/compile-principle/)》和《[链接](https://iguochan.github.io/2020/06/21/linking/)》。我们知道，源文件通过编译工具链（如gcc）编译、链接可以生成可执行文件。而巨大的软件工程可能拥有数量庞大的源文件，这使得在命令行中输入指令生成可执行文件的方法变得不那么容易接受。而make就是用来完成这项工作的工具，make指令更像是Makefile文件的解释器，会根据Makefile文件的编写逻辑生成可执行文件。

当初我学习Makefile编写的时候，参照的是陈皓大神的《跟我一起写Makefile》，他从基础处讲起，一点点深入，逐步介绍Makefile的各种特性和编写规则，使我一个小白收益匪浅，也写出了人生第一份Makefile。今天我准备以一种倒叙的逻辑写这篇博客，首先我会给一个可以使用的Makefile例子，然后一步步解析这个例子中所用的Makefile规则。

## 1 一个例子
在下面这个例子中，生成一个名为test的可执行文件，基本涉及Makefile的基本规则、隐晦规则、变量、函数应用等，将在接下来的内容中一一介绍。
``` bash
# define the target
TARGET = test

# define the Build Directory
BUILD_DIR = build
OBJ_DIR := $(BUILD_DIR)/objs
DEP_DIR := $(BUILD_DIR)/deps

# define PATH
LOCAL_PATH = $(shell pwd)

# define the sources and objects
SOURCES := $(shell find $(LOCAL_PATH)/ -name "*.c")
OBJS := $(addprefix $(OBJ_DIR)/, $(patsubst %.c, %.o, $(notdir $(SOURCES))))
DEPS := $(addprefix $(DEP_DIR)/, $(patsubst %.c, %.d, $(notdir $(SOURCES))))

# define VPATH
VPATH = $(LOCAL_PATH):$(LOCAL_PATH)/source_Dir1/:$(LOCAL_PATH)/source_Dir2/

# define the includes, compile and link flags
INCLUDES := -I$(LOCAL_PATH) -I$(LOCAL_PATH)/source_Dir1/ -I$(LOCAL_PATH)/source_Dir2/
CC_FLAGS := -g $(INCLUDES) 
LK_FLAGS := -L$(LOCAL_PATH)/lib_Dir
LK_FLAGS += -ltest -lm

# define the compiler
CC = gcc

# define the phony target
.PHONY : all clean

# build the target 
all: $(BUILD_DIR)/$(TARGET)
$(BUILD_DIR)/$(TARGET): $(OBJS)
	@if [ ! -d $(BUILD_DIR) ]; then mkdir -p $(BUILD_DIR); fi;\
	$(CC) $^ $(LK_FLAGS) -o $@

# build the objects
$(OBJ_DIR)/%.o : %.c
	@if [ ! -d $(OBJ_DIR) ]; then mkdir -p $(OBJ_DIR); fi;\
	$(CC) -c $(CC_FLAGS) -o $@ $<

# build the dependencies
$(DEP_DIR)/%.d : %.c
	@if [ ! -d $(DEP_DIR) ]; then mkdir -p $(DEP_DIR); fi;\
	set -e; rm -f $@;\
	$(CC) -MM $(CC_FLAGS) $< > $@.$$$$;\
	sed 's,\($*\)\.o[ :]*,$(OBJ_DIR)/\1.o $@ : ,g' < $@.$$$$ > $@;\
	rm -f $@.$$$$

# when *.h file changes, remake the project
-include $(DEPS)

# clean all products
clean:
	-rm -r $(BUILD_DIR)
```

## 2 Makefile的基本规则
make是一个解释器，其会根据Makefile的内容，调用编译器等Linux命令，最终生成编译产物。Makefile的基本规则如下：
``` bash
target ... : prerequisites ...
	command
	...
	...
```
* target：目标文件，可以是可重定位目标文件、可执行文件或动态库文件，还可以是标签（后续伪目标中会讲到）；可以是一个或多个文件；
* prerequisites：即生成target所需的文件，可以是一个或多个；
* command：是make需执行的命令，前需要空一个制表符（Tab键）；

以上描述的是一个文件的依赖关系，target这目标文件依赖于prerequisites中的文件，生成规则定义在command中。即当prerequisites中有一个以上的文件要比target中的文件要新的话，command命令就会被执行，这也是Makefile最核心的规则：
* 当我们执行make命令时，它会首先找到文件中的第一个目标文件（target），在上面的例子里，第一个文件就是伪目标all，伪目标all代表了```$(BUILD_DIR)/$(TARGET)```，根据变量定义，即test，也就是最终的可执行文件，后面会根据依赖关系找到生成目标文件所需的各级依赖文件；
* test依赖于$(OBJS)（也就是.o）文件，.o文件依赖于源文件；在上述例子的第34-36行，test是目标文件，.o文件是依赖文件；在39-41行，.o文件目标文件，.c源文件是依赖文件；而是否重新生成目标文件取决于依赖文件是否要比目标文件新；在首次编译时（或执行完make clean后），根据依赖关系，.o文件不存在，即.c源文件的改动要比对应的.o新，那么会生成所有的.o文件，而test文件也不存在，那么所有生成的.o文件都会比test文件新，故而会生成test文件，那么最终构造可执行文件的过程也就完成了。在其后，如果我们修改了某一个所依赖的a.c文件，再次执行make命令时，会检查到a.c比a.o新，所以会生成a.o文件，其后再检查到a.o比test要新，所以也会重新链接生成test文件。

也许有小伙伴会有疑问，为什么不使用```gcc -o test *.c```这种形式一步就生成可执行文件呢？原因在编译过程中，编译器会事先将每个源文件编译成可重定位目标文件（.o），然后链接器将所有的可重定位目标文件链接为可执行文件，那么根据以上规则，每一个.c的改动都会完完全全执行一遍所有的编译链接过程，而第1章例子中的写法可以在修改了某个模块后只编译此模块并重新链接到可执行文件，这在编译大型工程的时候有利于节省资源。

还有就是，上述例子中，只有33-41行的语法是无法做到头文件修改后，引用此头文件的源文件生成的模块重新编译的，因此需要44-52行的语法，这个后续会详细讲。
在大致了解了make在解析Makefile时候的工作机制后，后面我将一句第一章的例子涉及到的要素进行讲解。

make在工作时的执行有以下步骤（摘自陈皓的《[跟我一起写Makefile](https://blog.csdn.net/haoel/article/details/2886)》）：
* 1、将所有的Makefile和include包含的Makefile读入，包括include包含的依赖文件；
* 2、初始化所有变量；
* 3、推导隐晦规则，并分析所有规则，依据规则为所有目标文件创建依赖关系链；
* 4、根据依赖关系，确定哪些目标需要重新生成，并执行生成命令。 

值得注意的是，针对变量，如果定义的变量被使用了，那么，make会把其展开在使用的位置。但make并不会完全马上展开，make使用的是拖延战术，如果变量出现在依赖关系的规则中，那么仅当这条依赖被决定要使用了，变量才会在其内部展开。



## 3 伪目标
我们注意到在第33行，有```.PHONY : all clean```的定义，表示all和clean都是伪目标。伪目标并不是一个文件，只是一个标签，先然make不需要也无法根据依赖关系去生成这个标签，一般显式地通过关键字```.PHONY```去指明这是伪目标。
make执行的时候，如果不指定生成目标，会默认生成第一个可执行文件，如果需要生成多个可执行文件，但是不想敲过多的命令，可以定义一个名为all的的伪目标，指向几个可执行文件，这样直接执行```make all```的指令即可，本文例子中虽然只有一个可执行文件，但是还是定义了all。

对于clean这个伪目标也是一样，我们需要一个标签来删除生成的目标文件（包括中间产物），clean后不需要跟依赖文件，直接跟command指令即可。


## 4 变量
形如```TARGET = test```的形式是变量定义，在Makefile中定义的变量，就像是C语言中的宏一样，代表了一个文本字串，在Makefile中执行的时候会展开在所使用的地方，譬如在第33行，```$(BUILD_DIR)/$(TARGET)```就会被展开为```build/test```。不同于宏的是，变量可以在Makefile中改变值。在变量中，我们使用```$```符号取变量值，在使用时，最好用“（）”或“{}”将变量括起来，这样会更安全。

变量的赋值符号除了“=”号还有几种，以下是它们的区别：
* “=”：变量的值是整个Makefile中最后被指定的值；
* “:=”：变量的值是当前位置的值；
* “?=”：如果该变量没有被赋值，则赋予等号后的值；
* “+=”：追加变量值，将等号后面的值添加到前面的变量上；

### 4.1 VPATH
VPATH是Makefile中的特殊变量，不同于一般变量是用户自定义并在执行命令或解析依赖关系时再展开一样，VPATH是给Makefile用做寻找文件的依赖关系时的路径。如果没有设置此变量，那么make只会在当前的目录中去寻找依赖文件和目标文件；只有定义了这个变量，make会在当前目录找不到的情况下去指定的目录中寻找。

另一个设置文件搜索路径的方法是使用make的“vpath”关键字（注意，它是全小写的），这不是变量，这是一个make的关键字，这和上面提到的那个VPATH变量很类似，但是它更为灵活。它可以指定不同的文件在不同的搜索目录中。这是一个很灵活的功能。它的使用方法有三种：
* 1、vpath \<pattern\> \<directories\>：为符合模式\<pattern\>的文件指定搜索目录\<directories\>，\<pattern\>需要包含“%”通配符；
* 2、vpath \<pattern\>：清除符合模式\<pattern\>的文件的搜索目录；
* 3、vpath：清除所有已被设置好了的文件搜索目录；

### 4.2 隐含规则下的变量
在隐含规则下，基本会使用一些预先设置的变量，我们既可以使用这些变量，也可以重新定义这些变量，在编译时，可以利用make的“-R”或“--no–builtin-variables”参数来取消你所定义的变量对隐含规则的作用。

#### 4.2.1 命令相关的变量
* AR：函数库打包程序，默认命令是“ar”；
* AS：汇编语言编译程序，默认命令是“as”；
* CC：C语言编译程序，默认命令是“cc”，这里被重构为gcc，其实是一个命令：
``` bash
$ ll /etc/alternatives/cc /usr/bin/cc 
lrwxrwxrwx 1 root root 12 Nov 12  2014 /etc/alternatives/cc -> /usr/bin/gcc*
lrwxrwxrwx 1 root root 20 Nov 12  2014 /usr/bin/cc -> /etc/alternatives/cc*
```
* RM：删除文件命令，默认是“rm -f”；
还有许多就不一一列了。

#### 4.2.2 命令参数的变量
* CFLAGS：C语言编译器参数；
* LDFLAGS：链接器参数；
* ……

## 5 函数
在Makefile中使用函数来处理变量会使得命令更加的智能。函数调用的语法如下：
```bash
$(<function> <arguments>)
```
```<function>```指的是函数名，```<arguments>```指的是参数，参数间用“,”隔开，函数名和参数之间用空格隔开；函数调用以```$```符号开始，用“（）”或“{}”将函数名和参数括起来，用法类似于变量，如果前面有赋值，则将会将函数返回值赋值给变量。如例子中的
``` bash
LOCAL_PATH = $(shell pwd)
SOURCES := $(shell find $(LOCAL_PATH)/ -name "*.c")
OBJS := $(addprefix $(OBJ_DIR)/, $(patsubst %.c, %.o, $(notdir $(SOURCES))))
DEPS := $(addprefix $(DEP_DIR)/, $(patsubst %.c, %.d, $(notdir $(SOURCES))))
```
都用到了函数。下面就针对这几个函数讲讲，更多的函数定义还参考《[跟我一起写Makefile](https://blog.csdn.net/haoel/article/details/2886)》。

### 5.1 shell函数
shell函数的参数是操作系统的shell命令，如
``` bash
LOCAL_PATH = $(shell pwd)
SOURCES := $(shell find $(LOCAL_PATH)/ -name "*.c")
```
第一行中，pwd指令会返回当前Makefile所在目录的路径，即将此路径指定为LOCAL_PATH，第二行中，表示在此路径及子路径下查找所有的源文件，并将其赋值给SOURCES。

### 5.2 addprefix函数
addprefix属于文件名操作函数，其基本格式如下，功能是把前缀```<prefix>```添加到```<name>```中的每个单词前面，并返回加过前缀的文件名序列。
``` bash
$(addprefix <prefix>,<names...>)
```
### 5.3 patsubst函数
patsubst函数属于字符串处理函数，基本格式如下，功能是查找\<text\>中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式\<pattern\>，如果匹配的话，则以\<replacement\>替换。这里，\<pattern\>可以包括通配符“%”， 表示任意长度的字串。 如果\<replacement\>中也包含“%”， 那么， \<replacement\>中的这个“%”将是\<pattern\>中的那个“%”所代表的字串。（可以用“\”来转义， 以“\%”来表示真实含义的“%”字符）。并返回被替换后的字符串。
``` bash
$(patsubst <pattern>,<replacement>,<text>)
```

### 5.4 notdir函数
notdir函数用于取出文件名称中的非目录部分，并返回此部分，基本格式如下：
``` bash
$(notdir <names...>)
```
所以结合以上三个函数的含义，就能解析以下语句的定义，即取出每个源文件替换成.o(.d)文件并添加上```$(OBJ_DIR)```（```$(DEP_DIR)```）的路径前缀，并返回给```OBJS```（```DEPS```）变量。
``` bash
OBJS := $(addprefix $(OBJ_DIR)/, $(patsubst %.c, %.o, $(notdir $(SOURCES))))
DEPS := $(addprefix $(DEP_DIR)/, $(patsubst %.c, %.d, $(notdir $(SOURCES))))
```

## 6 符号定义
### 6.1 通配符
在Makefile中，% 表示的是通配符，和Unix系统中的 * 通配符有着不同的含义，我在网上找到的描述我觉得都不是很好理解，下面是我的理解：
* % 是Makefile的规则通配符，它会对后面的集合进行二次定义，当make准备生成test时，会发现其依赖于许多.o，当看到类似于```%.o : %.c```的语句时，会将前面所依赖的.o逐一展开，并将.c前的%替换为相应名称前缀的.c，如下：
``` bash
%.o : %.c
	gcc -c $< -o $@

等价于

a.o : a.c
	gcc -c a.c -o a.o
b.o : b.c
	gcc -c b.c -o b.o
……
```
* * 符号是Unix系统的通配符，表示所有。

### 6.2 特殊符号
* $@：目标的名字；
* $<：第一个依赖目标；
* $^：依赖目标集；
* $?：依赖目标集中更新过的文件；
* -command：忽略当前命令行所遇到的错误；
* @command：command指令将不会回显。

### 6.3 gcc选项
在以上例子中还有一些gcc的选项符号，譬如：
``` bash
# -I 表示去以下路径寻找头文件，一般头文件的查找在本目录以及编译器自带的目录下寻找，这是指定私有头文件目录
INCLUDES := -I$(LOCAL_PATH) -I$(LOCAL_PATH)/source_Dir1/ -I$(LOCAL_PATH)/source_Dir2/

# -L 表示去此路径下寻找链接库文件，一般标准库文件会在标准路径下，如果用到私有库，应指定 
LK_FLAGS := -L$(LOCAL_PATH)/lib_Dir

# -l 表示链接此库，ltest表示库名称为libtest.so或libtest.a，lm是libm.so，标准math库
LK_FLAGS += -ltest -lm
```

## 7 自动生成依赖性
源文件中包含了头文件，即源文件依赖于头文件，当工程较大时，一一写出依赖性是不合适的。大多数提供了“-M”选项，用于自动寻找源文件包含的头文件，并生成依赖关系，但是GNU C的编译器中，“-M”选项会将标准头文件也包含进来，我们使用“-MM”参数，只包含自定义的头文件，如下：
```bash
$ cc -M main.c 
main.o: main.c /usr/include/stdio.h /usr/include/features.h \
 /usr/include/x86_64-linux-gnu/bits/predefs.h \
 /usr/include/x86_64-linux-gnu/sys/cdefs.h \
 /usr/include/x86_64-linux-gnu/bits/wordsize.h \
 /usr/include/x86_64-linux-gnu/gnu/stubs.h \
 /usr/include/x86_64-linux-gnu/gnu/stubs-64.h \
 /usr/lib/gcc/x86_64-linux-gnu/4.4.7/include/stddef.h \
 /usr/include/x86_64-linux-gnu/bits/types.h \
 /usr/include/x86_64-linux-gnu/bits/typesizes.h /usr/include/libio.h \
 /usr/include/_G_config.h /usr/include/wchar.h \
 /usr/lib/gcc/x86_64-linux-gnu/4.4.7/include/stdarg.h \
 /usr/include/x86_64-linux-gnu/bits/stdio_lim.h \
 /usr/include/x86_64-linux-gnu/bits/sys_errlist.h list.h

$ cc -MM main.c 
main.o: main.c list.h
```
要让Makefile自动检测依赖头文件有些困难，不过GNU组织建议把编译器为每一个源文件的自动生成的依赖关系放到一个文件中，为每一个“name.c”的文件都生成一个“name.d”的Makefile文件，[.d]文件中就存放对应[.c]文件的依赖关系。于是，我们可以写出[.c]文件和[.d]文件的依赖关系，并让make自动更新或自成[.d]文件，并把其包含在我们的主Makefile中，这样，我们就可以自动化地生成每个文件的依赖关系了。以下语句体现了这个规则。
``` bash
$(DEP_DIR)/%.d : %.c
	@if [ ! -d $(DEP_DIR) ]; then mkdir -p $(DEP_DIR); fi;\
	set -e; rm -f $@;\
	$(CC) -MM $(CC_FLAGS) $< > $@.$$$$;\
	sed 's,\($*\)\.o[ :]*,$(OBJ_DIR)/\1.o $@ : ,g' < $@.$$$$ > $@;\
	rm -f $@.$$$$

# when *.h file changes, remake the project
-include $(DEPS)
```
这个规则的意思是，所有的[.d]文件依赖于[.c]文件，“rm -f $@”的意思是删除所有的目标，也就是[.d]文件，第二行的意思是，为每个依赖文件“$<”，也就是[.c]文件生成依赖文件，“$@”表示模式“%.d” 文件，如果有一个 C 文件是 name.c，那么“%”就是“name”，“$$$$”意为一个随机编号，第二行生成的文件有可能是“name.d.12345”，第三行使用 sed 命令做了一个替换，关于sed命令的用法请参看相关的使用文档。第四行就是删除临时文件。

以上语句可以保证每次生成新的依赖文件，要用include命令包含进Makefile，这样在头文件更新后，也会编译相应的目标。

## 8 总结
以上就是我对于Makefile的一些学习的笔记和总结，虽然没有针对Makefile进行十分详细且深入的研究，但是对于平时工作中对于Makefile的编写和阅读，应该也是足够应付的。想对Makefile的编写有更深入了解的，请移步陈皓大神的《[跟我一起写Makefile](https://blog.csdn.net/haoel/article/details/2886)》。