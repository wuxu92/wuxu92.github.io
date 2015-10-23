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
当程序在命令行中运行的时候，一般使用`Ctrl+c`可以终结程序运行。对于操作系统有一定了解的话应该知道，`Ctrl+c`向运行的进程发送了一个结束进程的signal。这种方式很有用，但是有点时候暴力结束进程执行不是我们想要的逻辑，我们可以捕获这个**termination signal**，然后对他做一些自己的处理。注意，windows下不能捕获这些信号，只在*nix下有效。

使用os/signal包提供的封装。

```golang
package main

import (
	"fmt"
	"os"
	"os/signal"
	"time"
)
var i int

func init() {
	go SignalHandler(make(chan os.Signal, 1))
}
func main() {
	i = 0
	for {
		i++
		time.Sleep(time.Second)
	}
}

func SignalHandler(c chan os.Signal) {
	signal.Notify(c, os.Interrupt)
	for s:= <-c;; s= <-c {
		switch s {
			case os.Interrupt:
				fmt.Println("interrupt received", i)
				os.Exit(0)  // 去掉这一行，那么Ctrl+c的快捷键将不能中断进程运行
			default:
				fmt.Println("default case")
		}
	}
}
```
运行上面的代码，在Ctrl-c时可以看到输出 interrupt received 字符串。

除了接收系统的信号，还可以向其他的进行发送信号。

```golang
syscall.Kill(syscall.Getpid(), syscall.SIGABRT)
```
这一部分可能暂时用不上，只要知道有这种用法就可以了。

## TCP/IP ##
使用net提供的TCP/IP封装可以很方便的监听与请求端口。

server(src/wuxu/tcpserver/main.go):

```golang
package main

import (
	"net"
	"fmt"
)
var i int
func main() {
	i = 0
	if listener, e := net.Listen("tcp", ":1024"); e == nil {
		for {
			if conn, e := listener.Accept(); e==nil {
				go func(c net.Conn) {
					defer c.Close()
					fmt.Fprintln(c, "hihi", i)
					i++
				}(conn)
			}
		}
	} else {
		fmt.Println("error")
	}
}
```

client(src/wuxu/tcpclient/main.go)

```golang
package main

import (
	"fmt"
	"net"
	"bufio"
)

func main() {
	if conn, e := net.Dial("tcp", "localhost:1024"); e == nil {
		defer conn.Close()
		if text, e := bufio.NewReader(conn).ReadString('\n'); e ==nil {
			fmt.Println(text)
		}
	} else {
		fmt.Println("error", e)
	}
}
```

### 加密的TCP通信 ###
还可以很方便地简历加密的连接，为了加密连接我们需要两套密钥对，即服务器的公钥与私钥，客户端的公钥与私钥，使用之前的方法再在服务器和客户端的目录下，各生成一对密钥。`go run $GOROOT/src/crypto/tls/generate_cert.go -ca=true -host="localhost"`
修改代码如下：

server:

```golang
package main

import (
	"fmt"
	"crypto/tls"
	"crypto/rand"
)
var i int
func main() {
	i = 0
	var config tls.Config
	if certificate, e := tls.LoadX509KeyPair("cert.pem", "key.pem"); e ==nil {
		config = tls.Config{
			Certificates: []tls.Certificate{certificate},
			Rand: rand.Reader,
		}
	}
	if listener, e := tls.Listen("tcp", ":1024", &config); e == nil {
		for {
			if conn, e := listener.Accept(); e==nil {
				go func(c *tls.Conn) {
					defer c.Close()
					fmt.Fprintln(c, "secure hihi", i)
					i++
				}(conn.(*tls.Conn))
			}
		}
	} else {
		fmt.Println("error")
	}
}
```
可以看出，引入tls包后不需要net包了，tls实现了相关得了协议的加密版本。

client:

```golang
package main

import (
	"fmt"
	"bufio"
	"crypto/tls"
)

func main() {
	var config tls.Config
	if certificate, e := tls.LoadX509KeyPair("cert.pem", "key.pem"); e == nil {
		config = tls.Config{
			Certificates: []tls.Certificate{certificate},
			InsecureSkipVerify: true,
		}
	}
	if conn, e := tls.Dial("tcp", "localhost:1024", &config); e == nil {
		defer conn.Close()
		if text, e := bufio.NewReader(conn).ReadString('\n'); e ==nil {
			fmt.Println(text)
		}
	} else {
		fmt.Println("error", e)
	}
}
```
和使用net包的整体流程差不多，使用tls包的实现替换net包。
这样我们就实现了一个使用tls密钥加密的传输通道了。

## UDP ##