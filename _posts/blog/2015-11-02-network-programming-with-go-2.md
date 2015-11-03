---
layout: post
title: Go 网络编程-2
description: 对于任何一门语言，网络编程都是重要的一部分。对于Go来说其天生的高并发网络编程更是充满魅力。所以今天开学学习Go网络编程部分，教材是 Jan Newmarch 的 Network programming with Go 的pdf文档。
category: post
tags: network golang go
published: true
lastUpdate: 2015-11-02
---
![](/images/golang/gopher-banner-small.jpg)

## UDP数据报 ##
关于UDP的部分在 [这篇](http://wuxu92.github.io/go-lang-notebook-2/) 文章里已经介绍了很多了，这里只记录在npwg中出现的内容。

## 监听多个socket ##
一个server一般式监听多个端口服务多个客户端的，这种情况一般会用到轮训的方式。在C中使用 select() 调用让内核去做这个工作。在Go中，为每一个端口分配一个Goroutine来实现。

## Conn, PacketConn, Listener 类型 ##
