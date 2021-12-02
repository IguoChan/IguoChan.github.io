---
title: Go语言实现继承与多态
date: 2020-11-28 15:00:44
tags: Go
categories: 语言
---
Go被称为“互联网时代的C语音”，严格意义上不是面向对象编程的语言，但是Go语言结构体和接口的特性可以轻松地实现继承和多态。
<!-- more -->

## 0 接口
Go语言有着非常灵活的接口概念，Go语言的接口是隐式实现的，只需简单地定义一些方法就足够了，无需对于给定的具体类型定义所有满足的接口类型。具体可以参考[《Go语言圣经——接口》](http://shouce.jb51.net/gopl-zh/ch7/ch7.html)。

## 1 Go语言实现继承
如下例子，定义一个结构体为`Person`，有成员`name`和`age`，这相当于C++中的**成员变量**；然后我们定义了方法`Speak`，相当于**成员函数**。结构体`Student`包含结构体`Person`，其可以继承后者的成员和方法，比C++中的类还要方便。
``` golang
package main

import (
	"fmt"
)

type Person struct {
	name	string
	age		int
}

func (p Person) Speak() {
	fmt.Printf("I am a Person, My name is %s, is %d years old!\n", p.name, p.age)
}

type Student struct {
	Person
	stuId	int
}

func main() {
	p := Person {"Tom", 16}
	p.Speak()

	s := Student {Person{"Jerry", 15}, 1001}
	s.Speak()
}
```

执行结果如下：
``` bash
$ go run main.go 
I am a Person, My name is Tom, is 16 years old!
I am a Person, My name is Jerry, is 15 years old!
```
当然我们也可以定义`Student`自己的`Speak`方法，如下：
``` golang
func (s Student) Speak() {
	fmt.Printf("I am a Student, My name is %s, is %d years old, my stuentId is %d!\n",s.name, s.age, s.stuId)
}
```
执行结果如下：
``` bash
$ go run main.go 
I am a Person, My name is Tom, is 16 years old!
I am a Student, My name is Jerry, is 15 years old, my stuentId is 1001!
```
从上可以看出，利用Go语言实现继承是一件很方便的事情。

## 2 Go语言实现多态
如下例子，我们定义一个名为`People`的接口，只需定义`Speak`和`Working`方法，无需实现该方法。然后对`Student`和`Teacher`对象都实现了该接口，即实现接口下的所有方法。
``` golang
package main

import "fmt"

type People interface {
	Speak()
	Working()
}

type Student struct {
	name		string
	age			int
	stuId		int
}

type Teacher struct {
	name		string
	age 		int
	teacherId	int
}

func (s Student) Speak() {
	fmt.Printf("I am a Student, My name is %s, is %d years old, my stuentId is %d!\n",s.name, s.age, s.stuId)
}

func (s Student) Working() {
	fmt.Println("I am a Student, My work is reading book!")
}

func (t Teacher) Speak() {
	fmt.Printf("I am a Teacher, My name is %s, is %d years old, my teacherId is %d!\n",t.name, t.age, t.teacherId)
}

func (t Teacher) Working() {
	fmt.Println("I am a Teacher, My work is teaching book!")
}

func SetPeople(p People) {
	p.Speak()
	p.Working()
}

func main() {
	s := Student {"Jerry",15,1001}
	SetPeople(s)

	t := Teacher {"Tom", 16, 2001}
	SetPeople(t)
}
```
`SetPeople`函数接收一个`People`接口类型的值，如果一个实体类型实现了该接口，`SetPeople`函数会根据不同的类型实体实现不同的行为，这相当于实现了多态。也就是针对一套方法接口，对于不同的类型可以实现不同的行为，执行结果如下：
``` bash
$ go run main.go 
I am a Student, My name is Jerry, is 15 years old, my stuentId is 1001!
I am a Student, My work is reading book!
I am a Teacher, My name is Tom, is 16 years old, my teacherId is 2001!
I am a Teacher, My work is teaching book!
```

因为任何用户定义的类型都可以实现任何接口，所以通过不同实体类型对接口值方法的调用就是多态。

## 3 小结
Go语言虽然没有将自己定位为面向对象的语言，但是Go语言却可以轻松实现面向对象编程的三大基本特征:封装、继承、多态。