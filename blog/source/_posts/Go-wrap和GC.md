---
title: Go-wrap和GC
date: 2021-03-07 11:49:39
tags: Go
categories: Go
---
## golang的wrap
在golang中，我们有时候会对对象包装一层wrap，当我们通过调用子对象的方法时，系统持有的是子对象，而非wrap。
<!-- more -->
如下的代码中：
``` golang
package main

import (
    "fmt"
    "runtime"
    "time"
)

var ch = make(chan bool)

type wrapper struct {
    core *core
}

type core struct {
}

func entry() {
    c := &core{
    }
    w := &wrapper{
        c,
    }
    runtime.SetFinalizer(w, func(ww *wrapper) {
        Finalize(ww.core)
    })
    go w.f()
}

func Finalize(ap *core) {
    fmt.Println("finalize")
}


func (w *wrapper) f() {
    w.core.f()
}

func (c *core) f() {
    select {
    case <-ch:
        fmt.Println("call core.f")
    }
    fmt.Println("finish loop")
}

func main() {
    entry()
    for i := 0; i < 10; i++{
        time.Sleep(1*time.Second)
        runtime.GC()
    }
    ch <- true
    time.Sleep(1*time.Second)
}
```
此时运行结果是：
``` bash
$ go run main.go
finalize
call core.f
finish loop
```

`go w.f()`这行代码实际运行的是运行的是`go w.core.f()`，实际上系统持有的对象是w.core，不是w，所以在gc的时候会被回收，如果我们添加如下的代码：
``` golang
package main

import (
    "fmt"
    "runtime"
    "time"
)

type wrapper struct {
    core *core
}

var ch = make(chan bool)

type core struct {
}

func entry() {
    c := &core{
    }
    w := &wrapper{
        c,
    }
    runtime.SetFinalizer(w, func(ww *wrapper) {
        Finalize(ww.core)
    })
    go w.f()
}

func Finalize(ap *core) {
    fmt.Println("finalize")
}

func (w *wrapper) keepAlive() {
}

func (w *wrapper) f() {
    defer w.keepAlive()
    w.core.f()
}

func (c *core) f() {
    select {
    case <-ch:
        fmt.Println("call core.f")
    }
    fmt.Println("finish loop")
}

func main() {
    entry()
    for i := 0; i < 10; i++{
        time.Sleep(1*time.Second)
        runtime.GC()
    }
    ch <- true
    time.Sleep(1*time.Second)
}
```
此时运行结果如下，说明不停的gc，也不会回收掉w。而这就是因为增加了一个`defer w.keepAlive()`，虽然是空函数，但是还是能够保持w对象到select结束。
``` bash
$ go run main.go
call core.f
finish loop
```

