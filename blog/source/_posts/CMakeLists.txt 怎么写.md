---
title: CMakeLists.txt 怎么写
date: 2020-07-06 23:12:30
tags:
- Linux
- cmake
categories:
- 工程构建
- cmake
---
写程序的大体步骤就是：首先用编辑器编写源代码，如.c文件；然后经过[预处理、编译和汇编](https://iguochan.github.io/2020/06/21/compile-principle/)生成可重定位目标文件，也就是.o（Unix下）文件；最后通过链接器将所有的.o以及用到的库文件[链接](https://iguochan.github.io/2020/06/21/linking/)成可执行文件。

但是当源文件越来越多时，一个个地编译就会特别麻烦，于是人们设计了一种批量处理源文件的工具，也就是Make，一种自动化编译工具，可以通过一条命令实现完全编译。为了实现以上功能，需要编写一个规则文件，也就是[Makefile](https://iguochan.github.io/2020/06/17/Makefile/)。
<!-- more -->

而对于一个大型程序，编写Makefile也是一件复杂的事情，于是人们设计了一个工具，自动生成Makefile，也就是CMake工具。另外，CMake不仅可以生成Unix环境下的GNU Make，也可以生成QT的qmake，微软的MS nmake，BSD Make（pmake），Makepp，等等。也就是说，CMake是一种跨平台的Make maker，它根据平台无关的CMakeLists.txt文件来指定整个编译过程，再根据平台生成所需的Makefile或project文件，本文就简单介绍一下Linux下CMakeLists.txt文件的编写。

## 1 一个简单的例子
假设我们只有一个源文件，如下所示：
``` c
#include <stdio.h>

int main()
{
    printf("Hello World!\n");
    return 0;
}
~                                                                               
                                                                             
"main.c"
```
我们在源文件所在的目录下写一个CMakeLists.txt文件，如下所示，其中：
* cmake_minimum_required(VERSION 3.3)：指定此文件运行的所需的cmake最低版本是3.3；
* project(test)：表示该项目的名字为test；
* add_executable(test main.c)：表示将源文件main.c编译生成test；

``` c
# Check the version of cmake
cmake_minimum_required(VERSION 3.3)

# Project name
project(test)

# Generate project
add_executable(test main.c)
```
这个时候，我们执行以下指令（新建一个build目录用于生成一系列产物很有必要，因为cmake的执行过程中会生成很多的中间产物和文件，这些文件并不属于工程，放在build目录下便于管理）：
``` bash
$ mkdir build && cd build && cmake ../
-- The C compiler identification is GNU 4.4.7
-- The CXX compiler identification is GNU 4.4.7
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/……/build
```
这个时候我们发现build目录下生成了多个文件，其中就有Makefile，这个时候我们执行以下指令，可以看到输出了打印结果。可以看到，通过在CMakeLists.txt中简单的几行代码，就可以构建代码工程。
``` bash
$ make && ./test
Scanning dependencies of target test
[ 50%] Building C object CMakeFiles/test.dir/main.c.o
[100%] Linking C executable test
[100%] Built target test
Hello World!
```
值得注意的是，在cmake中，所有的内置命令是不区分大小写的；但是cmake的变量是区分大小写的！

## 2 多个源文件
### 2.1 指定源文件
当同一目录下有多个源文件时，我们可以有多种方式，譬如均写在add_executable后：
``` bash
add_executable(test main.c a.c b.c)
```
但是当源文件太多时，这样写明显不方便，这时有多种方式解决。
#### 2.1.1 aux_source_directory命令
aux_source_directory命令形式如下，表示收集指定目录下（不包括子目录）所有源文件的名称，并将链表存储在\<variable\>中，我测试的时候发现，后缀为.cpp、.c和.cc的源文件都会被收录进来。
``` bash
aux_source_directory(<dir> <variable>)
```
这个时候，我们只需要将CMakeLists.txt修改成以下即可：
``` bash
# Check the version of cmake
cmake_minimum_required(VERSION 3.3)

# Project name
project(test)

# Collect all source file in current directory
aux_source_directory(. SRC)

# Generate project
add_executable(test ${SRC})
```
“.”表示当前目录，即CMakeLists.txt所在的目录，也可以用CMAKE_SOURCE_DIR（cmake定义了很多的常用变量，大家可以去cmake的文档中查找，很好用）；和shell脚本、Makefile规则一样，“$”符号用作取变量值。

#### 2.1.2 file命令
file命令用于文件操作，功能强大，大家可以通过查看[cmake官方手册](https://cmake.org/cmake/help/v3.18/command/file.html?highlight=file)查看。在这里，我们用到file命令查找的功能，即：
``` bash
file(GLOB <variable>
     [LIST_DIRECTORIES true|false] [RELATIVE <path>] [CONFIGURE_DEPENDS]
     [<globbing-expressions>...])
file(GLOB_RECURSE <variable> [FOLLOW_SYMLINKS]
     [LIST_DIRECTORIES true|false] [RELATIVE <path>] [CONFIGURE_DEPENDS]
     [<globbing-expressions>...])
```
file命令以上的用法是生成match正则表达式\<globbing-expressions\>的文件链表，并将其储存在变量\<variable\>中。LIST_DIRECTORIES、RELATIVE和CONFIGURE_DEPENDS都是标志，这里不表。我们可以使用以下的命令生成在该文件夹下后缀为.c的源文件list（如果还想要子文件夹下的源文件，将关键字替换为GLOB_RECURSE），并存入SRC变量中。
``` bash
# Check the version of cmake
cmake_minimum_required(VERSION 3.3)

# Project name
project(test)

# Collect all source file in current directory
file(GLOB SRC *.c)

# Generate project
add_executable(test ${SRC})
```

值得注意的是，在file的文档中，有以下说明：
``` bash
Note We do not recommend using GLOB to collect a list of source files from your source tree. If no CMakeLists.txt file changes when a source is added or removed then the generated build system cannot know when to ask CMake to regenerate.
```
大意就是官方不建议使用file的GLOB指令来收集工程的源文件，因为当收集的源文件增加或删除，且CMakeLists.txt没有发生修改时，CMake不能识别这些文件。其实，在aux_source_directory命令的说明中，也有如下同样的一段，表达相同的意思，如下。所以我们使用这两者进行文件收集时，应当注意在文件发生增删时，需重新使用cmake命令。
``` bash
aux_source_directory(<dir> <variable>)
...
It is tempting to use this command to avoid writing the list of source files for a library or executable target. While this seems to work, there is no way for CMake to generate a build system that knows when a new source file has been added. Normally the generated build system knows when it needs to rerun CMake because the CMakeLists.txt file is modified to add a new source. When the source is just added to the directory without modifying this file, one would have to manually rerun CMake to generate a build system incorporating the new file.
```

#### 2.1.3 set命令
set命令是cmake最常用的命令之一，用于将变量设置为给定值，常用的原型如下：
``` bash
set(<variable> <value>... [PARENT_SCOPE])
```
即用于将\<value\>值赋给当前\<variable\>，作用域默认在当前函数或目录，但是当PARENT_SCOPE标志设置后，作用域将扩大到上一级目录或范围。我们可以改写CMakeLists.txt如下：
``` bash
# Check the version of cmake
cmake_minimum_required(VERSION 3.3)

# Project name
project(test)

# Collect all source file in current directory
set(SRC ./main.c ./a.c ./b.c)

# Generate project
add_executable(test ${SRC})
```

以上三种方式其实大同小异，都是利用变量存储源文件，最后利用add_executable函数生成可执行文件。

### 2.2 多目录下引用、链接路径
大多数时候工程源代码都会分布在不同的目录下，这个时候的源文件的查找下使用2.1中方法进行查找，但是切不可在最高目录（一般CMakeLists.txt在此）使用GLOB_RECURSE关键字的file命令，这会将build目录下由CMake生成的源文件也包含进去，其中在```build/CMakeFiles/3.18.0-rc3/CompilerIdC/CMakeCCompilerId.c```文件中，还包含main函数，会造成```multiple definition of `main'```的错误。

在多目录的工程下，需要指定头文件和库文件的引用目录，分别类似于Makefile文件中“-I”和“-L”的用法。

#### 2.2.1 include_directories命令
在CMake中，使用include_directories命令来指定头文件的路径，类似于Makefile中使用```-I```指定路径。命令将给定目录添加到编译器用于搜索包含文件的目录中，相对路径被解释为相对于当前源目录。
``` bash
include_directories([AFTER|BEFORE] [SYSTEM] dir1 [dir2 ...])
```
这里需要注意，CMake中的include命令类似于C语言的include命令，用于载入其它模块的CMake代码，和include_directories的区别需厘清。

#### 2.2.2 link_directories命令
和include_directories一样，link_directories命令用于指定链接库文件的搜索路径，类似于Makefile中使用```-L```，用于添加链接器应在其中搜索库的路径。赋予此命令的相对路径被解释为相对于当前源目录。
``` bash
link_directories([AFTER|BEFORE] directory1 [directory2 ...])
```

## 3 库文件的链接
### 3.1 link_libraries命令
如果说link_directories类似于gcc中的“-L”，那么link_libraries就像gcc中的“-L... -l...”，不仅指定了链接文件的路径，还指定了链接文件名。命令应该在add_executable命令前执行。
``` bash
link_libraries([item1 [item2 [...]]]
               [[debug|optimized|general] <item>] ...)
```

### 3.2 target_link_libraries命令
其形式如下：
``` bash
target_link_libraries(<target> ... <item>... ...)
```
相比于link_libraries，CMake更加推荐使用target_link_libraries，如下，大意就是target_link_libraries不需要指定路径，除非是自己定义的私有库。
``` bash
Note The target_link_libraries() command should be preferred whenever possible. Library dependencies are chained automatically, so directory-wide specification of link libraries is rarely needed.
```

如果我们需要链接某个库文件，可能有以下两种方法，如果是标准库，使用target_link_libraries就显得更方便了。
``` bash
cmake_minimum_required(VERSION 3.3)

project(test)

link_libraries("/libpath/libhello.so")

add_executable(test main.c)
```
``` bash
cmake_minimum_required(VERSION 3.3)

project(test)

link_directories("/libpath/")

add_executable(test main.c)

target_link_libraries(test libhello.so)
# or target_link_libraries(test hello)
```

## 4 添加子目录的方法
在大型工程中，如果按照以上方法一一添加源文件会显得很麻烦，我们也可以将各个子文件夹中的内容编译成子模块，并利用add_subdirectory命令来添加该子模块。
``` bash
add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])
```
而在子模块中，我们将源文件编译为静态库，这时无需指定链接路径。如下例子，在主文件夹下（即最高目录）的CMakeLists.txt如下：
``` bash
cmake_minimum_required(VERSION 3.3)

project(test)

include_directories(
    ./source_Dir1
    ./source_Dir2
)

add_subdirectory(source_Dir1)
add_subdirectory(source_Dir2)

add_executable(test main.c)

target_link_libraries(test aaa bbb m)
```
在source_Dir1和source_Dir2的子文件夹下，其CMakeLists.txt分别如下：
``` bash
cmake_minimum_required(VERSION 3.3)

aux_source_directory(. DIR_LIB_SRCS)

add_library(aaa STATIC ${DIR_LIB_SRCS})
```
``` bash
cmake_minimum_required(VERSION 3.3)

aux_source_directory(. DIR_LIB_SRCS)

add_library(bbb STATIC ${DIR_LIB_SRCS})
```
基本按照以上的形式构建一个工程就差不多搞定了。编译动态库时将以上STATIC关键字改成SHARED即可。

值得注意的是，对于有些需要库文件（譬如libaaa.a）需要完全编译到最后的可执行代码中的情况，需要在target_link_libraries命令下做如下处理（一行写也行，我是为了看起来直观），-Wl,--whole-archive和-Wl,--no-whole-archive是gcc的链接选项，可以在CMake中直接使用。
``` bash
target_link_libraries(test
	-Wl,--whole-archive
	aaa
	-Wl,--no-whole-archive
	bbb
	m)
```

## 5 小结
CMakeLists.txt的简单写法就介绍到这里，还有关其很多的关键字和内置变量的用法这里就不一一介绍了，百度的资料良莠不齐，因此去[官网](https://cmake.org/cmake/help/v3.18/search.html?q=)学习和搜索是一种很好的学习习惯。