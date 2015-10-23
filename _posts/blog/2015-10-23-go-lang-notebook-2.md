---
layout: post
title: Golang 学习笔记2
description: 最近开始看golang的一个网上教程，名字叫做 A Go Developer’s Notebook，挺不错的
category: post
tags: go golang lang
published: true
lastUpdate: 2015-10-23
---

## HTTP ##
我们知道，go内置有http server 支持，我们只需要在代码中启动这个server就可以启动一个类似于apache的服务器。同时可以很方便的监听多个端口等等。

```golang
import (
	. "fmt"
	"net/http"
)

const (
	PORT = ":1024"
	MSG = "hello, gopher"
)
func main() {
	http.HandleFunc("/hello", Hello)
	go func() {
		http.ListenAndServe(PORT, nil)
	}()
	
	go func() {
		http.ListenAndServeTLS(":443","cert.pem", "key.pem", nil)
	}()
	for {}
}
func Hello(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "text/plain")
	Fprintf(w, MSG)
}
```
这段代码启动了一个server，开启了http和https支持；分别监听在1024端口和443端口。
有几个需要注意的地方。

1. http.HandleFunc(string, func(http.ResponseWriter, *http.Request)) 绑定某个请求路径的处理方法
2. 使用goroutine执行监听与服务动作。启动了两个goroutine
3. 最后使用一个for循环保证主线程不会退出。因为主线程退出的话goroutine也会结束。
4. 其中的TLS支持的HTTPS服务的cert和key文件可以使用Go源码中带的文件生成`go run $GOROOT/src/crypto/tls/generate_cert.go -ca=true -host="localhost"
`

最后一个无限循环可以使用channel代替，可以使用sync.WaitGroup实现， sync.WaitGroup持有一个计数器，其`Add(int)`方法增加计数器，其`Done()`方法减小计数器，其`Wait()`方法会让线程阻塞等待计数器恢复0。其实现如下。

```golang
import (
	. "fmt"
	"net/http"
	"sync"
)

const (
	PORT = ":1024"
	MSG = "hello, gopher"
)
var servers sync.WaitGroup

func main() {
	http.HandleFunc("/hello", Hello)
	Launch(func() {
		http.ListenAndServe(PORT, nil)
	})
	
	Launch(func() {
		http.ListenAndServeTLS(":443","cert.pem", "key.pem", nil)
	})
	
	servers.Wait()
}

func Launch(f func()) {
	servers.Add(1)
	go func() {
		defer servers.Done()
		f()
	}()
}

func Hello(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "text/plain")
	Fprintf(w, MSG)
}
```
使用上面的方法可以添加任意多的监听端口。可以通过Server做更多的配置。

```golang
s := &http.Server{
	Addr:           ":8080",
	Handler:        myHandler,
	ReadTimeout:    10 * time.Second,
	WriteTimeout:   10 * time.Second,
	MaxHeaderBytes: 1 << 20,
}
log.Fatal(s.ListenAndServe())
```

### HTTP与Go程结合 ###
我们对Goroutine和HTTP都比较熟悉了，我们可以设置一个任务队列，让每一个请求都进入这个队列然后使用一定数量的Goroutine去执行逻辑。这样效率比上面的实现要高很多。

### 发送GET/POST请求 ###
http保重提供了非常方便的发发送get请求和post请求的封装。

```golang
resp, err := http.Get("http://example.com/")
...
resp, err := http.Post("http://example.com/upload", "image/jpeg", &buf)
...
resp, err := http.PostForm("http://example.com/form",
	url.Values{"key": {"Value"}, "id": {"123"}})

defer resp.Body.Close()  // 不要忘记defer关闭资源
body, err := ioutil.ReadAll(resp.Body)  // 读取资源
strCont := string(body)
```

## 环境变量 ##
使用os包中的`Getenv(string)`和`Setenv(string, string)`方法获得和设置环境变量。要注意Setenv的作用域只是当前运行进程，结束后或者重新运行后环境变量会重置为系统设置值。

```golang
Println(os.Getenv("PATH"))
os.Setenv("GOOO", "gofast")
os.Setenv("PATH", ".")
Println(os.Getenv("PATH"))
```

## 操作系统信号 ##
当程序在命令行中运行的时候，一般使用`Ctrl+c`可以终结程序运行。对于操作系统有一定了解的话应该知道，`Ctrl+c`向运行的进程发送了一个结束进程的signal。这种方式很有用，但是有点时候暴力结束进程执行不是我们想要的逻辑，我们可以捕获这个**termination signal**，然后对他做一些自己的处理。

使用os/signal包提供的封装。
