---
title: Golang微框架Gin
date: 2020-12-06 16:20:07
tags: Go
categories: 语言
---
框架是敏捷开发中的利器，更像是一些常用函数或者工具的集合。借助框架开发，不仅可以省去很多常用的封装带来的时间，也有助于团队的编码风格和形成规范。

框架能让开发者快速上手。Golang提供的net/http库对于http协议的实现非常好，基于此的Gin框架是一个封装优雅，API友好的web微框架。
<!-- more -->


## 0 Hello World
如下代码所示，简单的几行代码就能实现一个web服务。使用gin的Default方法创建一个路由handler。然后通过HTTP方法绑定路由规则和路由函数。不同于net/http库的路由函数，gin进行了封装，把request和response都封装到gin.Context的上下文环境。最后是启动路由的Run方法监听端口。麻雀虽小，五脏俱全。当然，除了GET方法，gin也支持POST,PUT,DELETE,OPTION等常用的restful方法。
``` golang
package main

import (
"github.com/gin-gonic/gin"
"net/http"
)

func main(){

	router := gin.Default()

	router.GET("/", func(c *gin.Context) {
		c.String(http.StatusOK, "Hello World")
	})
	router.Run(":8000")
}
```
我们在主机上运行代码并访问[http://localhost:8000](http://localhost:8000)，即可访问看到`Hello World`字样。

查看`gin.Default()`源代码，如下：
``` golang
// Default returns an Engine instance with the Logger and Recovery middleware already attached.
func Default() *Engine {
	debugPrintWARNINGDefault()
	engine := New()
	engine.Use(Logger(), Recovery())
	return engine
}
```
可以发现，`gin.Default()`函数内部调用New()函数来初始化一个gin实例，同时使用注册了`Logger`和`Recovery`两个中间件，所以我们也可以使用`gin.New()`函数来初始化一个无中间件启动的gin实例。

## 1 Gin框架的几个核心结构
开发一个HTTP服务，首先需要启动一个TCP监听，然后需要一系列的handler来处理具体的业务逻辑，最后在再将具体的业务逻辑通过HTTP协议约定和相关的Method和URL进行绑定，以此来对外提供具体功能的HTTP服务。Gin中通过以下几个重要的模型来实现以上功能。

### 1.1 Engine
用来初始化一个gin对象实例，在该对象实例中包含了一些框架的基础功能。如前代码所示，首先我们使用`gin.Default()`或者`gin.New()`初始化一个实例。
``` golang
type Engine struct {
	// 路由组，在实际开发过程中我们通常会使用路由组来组织和管理一些列的路由. 比如: /apis/,/v1/等分组路由
	RouterGroup

	// 开启自动重定向。如果当前路由没有匹配到，但是存在不带/开头的handler就会重定向. 比如: 用户输入/foo/但是存在一个/foo 就会自动重定向到该handler，并且会向客户端返回301或者307状态码(区别在于GET方法和其他方法)
	RedirectTrailingSlash bool

    // 如果开启该参数，没有handler注册时，路由会尝试自己去修复当前的请求地址.
    // 修复流程:
    // 1.首位多余元素会被删除(../ or //); 2.然后路由会对新的路径进行不区分大小写的查找;3.如果能正常找到对应的handler，路由就会重定向到正确的handler上并返回301或者307.(比如: 用户访问/FOO 和 /..//Foo可能会被重定向到/foo这个路由上)
	RedirectFixedPath bool

	// 如果开启该参数，当当前请求不能被路由时，路由会自己去检查其他方法是否被允许.在这种情况下会响应"Method Not Allowed"，并返回状态码405; 如果没有其他方法被允许，将会委托给NotFound的handler
	HandleMethodNotAllowed bool

	// 是否转发客户端ip
	ForwardedByClientIP    bool

	// 如果开启将会在请求中增加一个以"X-AppEngine..."开头的header
	AppEngine bool

	// 如果开启将会使用url.RawPath去查找参数(默认:false)
	UseRawPath bool

	// 如果开启，请求路径将不会被转义. 如果UseRawPath为false，该参数实际上就为true(因为使用的是url.Path)
	UnescapePathValues bool

	// maxMemory参数的值(http.Request的ParseMultipartForm调用时的参数)
	MaxMultipartMemory int64

	// 是否删除额外的反斜线(开始时可解析有额外斜线的请求)
	RemoveExtraSlash bool

	// 分隔符(render.Delims表示使用HTML渲染的一组左右分隔符,具体可见html/template库)
	delims           render.Delims

	// 设置在Context.SecureJSON中国的json前缀
	secureJsonPrefix string

	// 返回一个HTMLRender接口(用于渲染HTMLProduction和HTMLDebug两个结构体类型的模板)
	HTMLRender       render.HTMLRender

	// html/template包中的FuncMap map[string]interface{} ,用来定义从名称到函数的映射
	FuncMap          template.FuncMap

	// 以下是gin框架内部定义的一些属性
    // HandlersChain 是一个HandlerFunc 的数组(HandlerFunc其实就是一个Context的指针,Context会在下一节讲解)
	allNoRoute       HandlersChain
	allNoMethod      HandlersChain
	noRoute          HandlersChain
	noMethod         HandlersChain

	// 这里定义了一个可以临时存取对象的集合(sync.Pool是线程安全的，主要用来缓存为使用的item以减少GC压力，使得创建高效且线程安全的空闲队列)
	pool             sync.Pool

	// methodTrees是methodTree的切片(methodTree是一个包含请求方法和node指针的结构体,node是一个管理path的节点树)
	trees            methodTrees
}
```
如上，`New()`和`Default()`函数用于初始化`*Engine`实例。