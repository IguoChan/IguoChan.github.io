---
title: Go-闭包时的for-range
date: 2021-03-07 10:00:19
tags: Go
categories: 语言
---
for-range关键字可以用来遍历array、slice、map和string，可以返回一个值或两个值，对于array、slice和string，返回一个值时返回的时索引，两个值时返回的是索引和索引对应位置的值；对于map而言，返回一个值时是key值，双值时是key和value。
<!-- more -->

## 1 正常用法
在循环时，并不是每一次循环都申请不同的临时变量，而是在整个循环中都只声明一个临时变量，在循环结束后这个变量会被gc回收，每次循环都会把容器中的索引和值赋值给临时变量。
如下，一个最简单的for-range用法如下：
``` golang
package main

import "fmt"

func main() {
    var slice = []int{100, 200, 300, 400, 500}
    for i, s := range slice {
        fmt.Println("[", i, "]", ":", s)
    }
}
```
这时候打印出来的结果如下，输出很符合想象
``` bash
$ go run main.go
[ 0 ] : 100
[ 1 ] : 200
[ 2 ] : 300
[ 3 ] : 400
[ 4 ] : 500
```

## 2 闭包的坑
闭包中，闭包持有的是对捕获变量的引用，如下是并发操作时常用的操作。
``` golang
package main

import (
    "fmt"
    "sync"
)

func main() {
    var slice = []int{100, 200, 300, 400, 500}
    wg := sync.WaitGroup{}
    for i, s := range slice {
        wg.Add(1)
        go func() {
            defer wg.Done()
            fmt.Println("[", i, "]", ":", s)
        }()
    }
    wg.Wait()
}
```
其输出是：
```bash
$ go run main.go
[ 4 ] : 500
[ 4 ] : 500
[ 4 ] : 500
[ 4 ] : 500
[ 4 ] : 500
```
可以发现，结果并非如预计那样（如果数据量很大的话，输出的值就是混乱的，而不是一致性的最后一个），这是因为闭包函数所持有的是`i`和`s`的引用，所以最后打印出来的就是其协程执行时`i`和`s`的值。

## 3 使用临时变量避免以上错误
如果我们将代码改成如下，就能使得并发操作是正确的。
```golang
package main

import (
    "fmt"
    "sync"
)

func main() {
    var slice = []int{100, 200, 300, 400, 500}
    wg := sync.WaitGroup{}
    for i, s := range slice {
        wg.Add(1)
        tempI := i
        tempS := s
        go func() {
            defer wg.Done()
            fmt.Println("[", tempI, "]", ":", tempS)
        }()
    }
    wg.Wait()
}
```
输出结果如下（并发操作，所以是乱的），这是因为每次循环都会创建一个临时变量`tempI`和`tempS`，所以闭包对该变量的引用是独一份的。
``` bash
$ go run main.go
[ 4 ] : 500
[ 0 ] : 100
[ 1 ] : 200
[ 2 ] : 300
[ 3 ] : 400
```

## 4 函数传参
我们也可以使用函数传参，避免以上问题。
``` golang
package main

import (
    "fmt"
    "sync"
)

func main() {
    var slice = []int{100, 200, 300, 400, 500}
    wg := sync.WaitGroup{}
    for i, s := range slice {
        wg.Add(1)
        go func(paramI, paramS int) {
            defer wg.Done()
            fmt.Println("[", paramI, "]", ":", paramS)
        }(i, s)
    }
    wg.Wait()
}
```
结果如下：
```bash
$ go run main.go
[ 4 ] : 500
[ 0 ] : 100
[ 1 ] : 200
[ 2 ] : 300
[ 3 ] : 400
```


## 5 小结
其实避免for-range的错误只需注意以下两点：
* range返回值在整个循环中都只声明一个临时变量；
* 闭包持有的是捕获变量的引用，而不是复制。