---
title: Go go.mod包管理工具
date: 2020-10-24 15:44:13
tags: Go
categories: 语言
---
近来入职旷厂转行做后端开发，开始接触Go语言，刚入手时学习的是[Go语言圣经](http://shouce.jb51.net/gopl-zh/index.html)，其中利用```GOPATH```这个变量来管理Go工程，但是这样会很麻烦，不管是在Linux环境中还是Windows中，都需要在不同的工程中设置该变量。

go modules是Golang 1.11新加的特性，是Go包的集合。modules是源代码交换和版本控制的单元。 go命令直接支持使用modules，包括记录和解析对其他模块的依赖性。modules替换旧的基于```GOPATH```的方法来指定在给定构建中使用哪些源文件。

Modules和传统的GOPATH不同，不需要包含例如src，bin这样的子目录，一个源代码目录甚至是空目录都可以作为Modules，只要其中包含有go.mod文件。
<!-- more -->

## 0 使用go.mod条件
1）首先将go的版本升级到1.11以上

2）设置GO111MODULE（off/on/auto）
* GO111MODULE=off，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找；
* GO111MODULE=on，go命令行会使用modules，而一点也不会去GOPATH目录下查找；
* GO111MODULE=auto，默认值，go命令行将会根据当前目录来决定是否启用module功能。这种情况下可以分为两种情形：
    * 当前目录在GOPATH/src之外且该目录包含go.mod文件；
    * 当前文件在包含go.mod文件的目录下面。

## 1 go.mod初探
go为用户提供了`go mod`命令来管理包，其子命令有以下几种：
![go mod command](/img/go-mod-commond.png)

首先，我们在GOPATH/src外新建一个工程，或者将老工程从GOPATH/src拷贝出来。go.mod文件一旦创建，它的内容将会被go toolchain全面掌控。

* go.mod文档中有module, require、replace和exclude四个命令：
	* module语句指定包的名字（路径）
	* require语句指定的依赖项模块
	* replace语句可以替换依赖项模块
	* exclude语句可以忽略依赖项模块

首先我们在新建一个文件夹，名为hellomod，然后在文件夹下新建名为main的文件夹，然后我们在main文件夹中添加main.go文件，内容如下：
``` golang
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello world!")
}
```

然后在hellomod文件夹下执行`go mod init hello`指令，得到如下所示的go.mod文件（子目录里是不需要init的，所有的子目录里的依赖都会组织在根目录hellomod的go.mod文件里）：
```
module hello

go 1.15
```
后续可以直接执行运行命令等:
``` bash
$ go run main.go
Hello world!
```
这个时候go.mod的作用并没有体现出来，我们接着往下看。


## 2 私有包
在hellomod文件夹中新建aaa文件夹，然后在aaa文件加中添加任意名字的`.go`文件，如`aaa.go`，内容如下（注意package名称一定要和文件夹相同，但是go文件的名称无所谓）：
``` golang
package aaa

import (
	"fmt"
)

func AaaPrint() {
	fmt.Println("Hello AAA")
}
```
修改main.go如下：
``` golang
package main
 
import (
	"fmt"
	"hello/aaa"
)
 
func main() {
    fmt.Println("Hello, world!")
    aaa.AaaPrint()
}
```
注意在导入包的时候，一定要用模块名充当路径名，即第5行的`"hello/aaa"`，这样才能找到aaa包。这是再次运行代码，得到：
``` bash
$ go run main.go
Hello, world!
Hello AAA
```
这时候可以发现go.mod文件内容并没有改变，但是我们不需要和之前一样必须在GOPATH路径下才能找到依赖包了，这大大方便了go工程管理。

## 3 github上的包
我们修改main.go代码如下，随意找一个github上的包名，如下：
``` golang
package main
 
import (
	"fmt"
	"hello/aaa"
	_ "github.com/gohouse/gorose"
)
 
func main() {
    fmt.Println("Hello, world!")
    aaa.AaaPrint()
}
```
然后再次运行代码，得到：
``` bash
$ go run main.go
go: finding module for package github.com/gohouse/gorose
go: found github.com/gohouse/gorose in github.com/gohouse/gorose v1.0.5
go: finding module for package github.com/gohouse/converter
go: found github.com/gohouse/converter in github.com/gohouse/converter v0.0.3
Hello, world!
Hello AAA
```
这时候可以发现go.mod文件内容变成了
```
module hello

go 1.15

require (
	github.com/gohouse/converter v0.0.3 // indirect
	github.com/gohouse/gorose v1.0.5 // indirect
)
```
对应上面的log，可以发现go工具自动去github上下载了文件，放在了`$GOPATH/pkg/mod`路径下，如下
```
C:\Users\Chen\go\pkg\mod\github.com\gohouse
$ ls
'converter@v0.0.3'/  'gorose@v1.0.5'/
```

## 4 使用其它版本的包
上例中是自动使用github上最新版本的包，这里我们可以使用旧版本的包，譬如我们想用gorose 1.0.4的包，可以执行指令`go mod edit -require=github.com/gohouse/gorose@v1.0.4`，然后运行程序：
``` bash
$ go run main\main.go
go: downloading github.com/gohouse/gorose v1.0.4
Hello, world!
Hello AAA
```
这时候发现，go.mod中gorose的版本变成了v1.0.4：
```
module hello

go 1.15

require (
	github.com/gohouse/converter v0.0.3 // indirect
	github.com/gohouse/gorose v1.0.4 // indirect
)
```
还有其它的一些`go mod`指令，大家也可以试试。

除了go.mod之外，go命令还维护一个名为go.sum的文件，其中包含特定模块版本内容的预期加密哈希 
go命令使用go.sum文件确保这些模块的未来下载检索与第一次下载相同的位，以确保项目所依赖的模块不会出现意外更改，无论是出于恶意、意外还是其他原因。 go.mod和go.sum都应检入版本控制。