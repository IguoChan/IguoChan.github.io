---
title: Go语言断言注意事项
date: 2020-12-09 23:31:35
tags: Go
categories: 语言
---
类型断言（Type Assertion）是一个使用在接口值上的操作，用于检查接口类型变量所持有的值是否实现了期望的接口或者具体的类型。
在使用断言的时候，我们最好**接收第二个参数，或者确认该断言必定成功，否则可能引起系统的Panic！**
<!-- more -->

## 1 正确的使用方式
在Golang中，断言的语法格式如下所示：
``` golang
value, ok := x.(T)
```
其中，x表示一个接口的类型，T表示一个具体的类型（也可为接口类型）。

该断言表达式会返回x的值（也就是value）和一个布尔值（也就是ok），可根据该布尔值判断x是否为T类型：
* 如果T是具体某个类型，类型断言会检查x的动态类型是否等于具体类型T：如果检查成功，类型断言返回的结果是x的动态值，其类型是T，ok=true；如果检查失败，value为类型T的零值，ok=false；
* 如果T是接口类型，类型断言会检查x的动态类型是否满足T。如果检查成功，x的动态值不会被提取，返回值是一个类型为T的接口值，ok=true；如果检查失败，value为类型接口的零值nil，ok=false；
* 无论T是什么类型，如果x是nil接口值，类型断言都会失败。

如下代码：
``` golang
package main

import (
    "fmt"
)

func main() {
    var x interface{}
    x = 10
    value, ok := x.(int)
    fmt.Println(value, ok)
}
```
结果是：
``` bash
$ go run main.go
10 true
```
如果我们屏蔽`x = 10`那行，那么运行结果就如下：
``` bash
$ go run main.go
0 false
```

## 2 错误的使用方式
**如果不接收第二个参数也就是上面代码中的 ok，断言失败时会直接造成一个 panic！！！**
如下：
``` golang
package main

import (
    "fmt"
)

func main() {
    var x interface{}
    x = 10
    v1 := x.(int)
    fmt.Println(v1)

    v2 := x.(string)
    fmt.Println(v2)
}
```
运行结果如下，第一次断言（v1）是成功的，虽然没有接收第二个参数，是可以接收到正确的值的；第二次断言（v2）明显类型不对，断言失败，直接导致panic。
``` bash
$ go run main.go
10
panic: interface conversion: interface {} is int, not string
```
当然，在interface是nil的时候，如果没有接收第二个参数，一定会造成panic，譬如我们将`x = 10`屏蔽，那么第一次断言的时候就会panic。


这是我在工作中遇到的一个bug，同事写的代码中直接对interface进行断言却没有判断interface是否为nil（或者接收第二个参数），导致了系统的panic。Go语言虽然相比于C语言可以让程序员犯更少的错（毕竟指针是C程序员的Bug之源），但是书写的时候还是要注意。