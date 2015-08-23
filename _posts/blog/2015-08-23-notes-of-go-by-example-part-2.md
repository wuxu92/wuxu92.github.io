---
layout: post
title: Go by Example 中的goroutine和channel(二)
description:  gobyexample.com 是一个非常好的入门教程，里面有很多基础的知识，这里主要记录一些比较有“新意”的点。这是第二部分。这些笔记记得越来越细了，本来只打算记一些比较有新意的东西的，现在想记就记了。
category: lang
tags: golang tricks
published: true
lastUpdate: 2015-08-22
---

## Goroutines ##
goroutines就是可以方便的并行执行函数，本身并没有太多东西要注意的，主要的内容在于goroutine和channel结合的并发程序

## Channel ##
channel相当于联系并发的goroutine的管道，我们常使用linux中的管道pipe做类比：

```bash
ls -l | grep .go
```
上面的命令把当前目录下的文件列出，每个文件占一行，用管道把输出结果送到grep程序的标准输入流，grep把当前目录下的go源程序文件筛选出来。管道吧ls程序和grep程序连接起来，goroutine的作用也差不多，把一个goroutine的输出，可以在另外一个goroutine读出来。
当然channel要比linux的管道功能更加强大一些，因为是可编程的，它可以传递任意规定的类型的变量，使用也很方便。

```golang
msgChannel := make(chan string)

go func() {msgChannel <- "hello"}()
msg := <-msgChannel // receive a msg from channel
```
上面的代码即声明了一个可传递string类型的channel。第3行代码则使用goroutine执行一个匿名函数，函数的作用是向msgChannel传入一个字符串。

> goroutine执行一个匿名还是得语法是很有用的，其格式为```go func() { ... }()```注意最后的括号表示执行匿名函数。

```channel <-```通常叫做发送操作，即向channel发送一个值。``` <- channel```通常叫做接受操作，即从channel中取出一个值。
channel的发送和接受是一个同步的过程，只有sender和receiver都准备就绪的时候才能发送/接受，否则会阻塞等待，这在后面的Channel Buffering 中会讲到。

### Channel Buffering ###
默认的channel是不带缓冲的（unbuffered）如果像上面的方式定义channel的话，只能往里面添加一个元素，并且需要有从channel取数据的receiver，否则再往里面添加元素则会导致无限等待。
所以更常见的是使用带缓冲的channel，这样可以一次性往channel里面添加多个元素。

```golang
msgChannel := make(chan string, 2)

msgChanenl <- "hello"
msgChannel <- "world"

fmt.Println(<-msgChannel)
fmt.Println(<-msgChannel)
```
这样就channel就可以缓存两个变量了。在实际使用中，常会使用更大的缓存channel。

### 使用channel同步goroutine ###
我们知道channel的发送和接受操作只有在channel准备就绪时才会进行，否则会阻塞等待，也就是说在向channel发送数据时，需要channel有空余空间，从channel接受数据时，需要channel不能为空，否则也会阻塞等待。如下的代码展示了一个使用不带缓冲的channel实现同步的功能:

```golang
func working(flagChan chan bool) {
  fmt.Println("working on it")
  time.Sleep(2 * time.Second)
  fmt.Println("work done")            

  flagChan <- true
}

func chanSync() {
  flagChan := make(chan bool, 1)
  go working(flagChan)

  // 下面的接受操作会阻塞等待goroutine执行结束
  done := <- flagChan 
  fmt.Println("sync goroutine output", done)
}
```

### 指定channel的方向（操作） ###
我们可以在方法的定义的参数列表处指定channel参数的操作方式为发送或者接受，因为一般发送是一个逻辑，接受是一个逻辑，通常不会把两个操作放在一起，指定某个函数的操作模式可以起到“代码即文档”的作用。比如上面的working函数：

```go
func working (flafChan chan<- bool) { ... }

func recv (flagChan <-chan bool) { ... }
```
即表明了在函数中channel的数据流向（directions）
网页中的代码示例：

```
package main
import "fmt"
func pong(pings <-chan string, pongs chan<- string) {
    msg := <-pings
    pongs <- msg
}
func ping(pings chan<- string, msg string) {
    pings <- msg
}
func main() {
    pings := make(chan string, 1)
    pongs := make(chan string, 1)
    ping(pings, "passed message")
    pong(pings, pongs)
    fmt.Println(<-pongs)
}
```

## 多路goroutine: select ##
