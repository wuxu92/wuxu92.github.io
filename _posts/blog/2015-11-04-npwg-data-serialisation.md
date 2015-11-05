---
layout: post
title: Go 网络编程-数据序列化
description: 这一篇主要是关于Go序列化数据。数据序列化在网络编程中非常重要，在网络传输数据时，如果数据是对象，数组等等，需要先把对象序列化为才能传递，同时接收方需要进行反序列化才能使用这些数据。一个比较常见的是json数据。
category: post
tags: network golang go
published: true
lastUpdate: 2015-11-05
---
![](/images/golang/gopher-banner-small.jpg)

{{ page.description }} 序列化和反序列化操作又成为 marshalling 和 unmarshalling 。

## 序列化方法 ##
序列化其实简单地说就是一种把变量似乎用字节流表示出来的规范，我们可以定义自己的规范，只要接收方能使用相应的方法反序列化出来原来的对象就可以了。但是我们要考虑序列化方法的通用性，希望对任意的数据可以使用同样的序列化和反序列化方法实现。
早期很著名的系列化方法是 XDR(external data representation),它最早用于Sun的RPC中。Go没有对不透数据的marshalling 和 unmarshalling 有明确的支持。RPC包中没有使用XDR，而是使用`gob`。
要实现对数据统一的序列化方法，需要实现一套 **自描述数据**的规范。像xml就是一类自描述的数据，能通过节点属性对数据进行描述。xml还有很强的扩展性。这也是为什么很多项目中会使用xml作为数据传输的格式。


## ASN ##
ASN 是 abstract Syntax notation 的缩写，也就是抽象格式标记。这是1984年为电信行业设计的。ASN.1非常复杂，在Go的 'ans.1' 包中实现了一个它的一个子集。它可以从复杂的数据结构中建立自描述的序列化数据。在现在的网络系统中用的最多的是作为 X.509 认证的编码，我们知道 X.509 在认证（SSL）中用途非常广。
下面两个方法可以用来marshal和unmarshal数据。

```
func Marshal(val interface{}) ([]byte, os.Error)
func Unmarshal(val interface{}, b []byte) (rest []byte, err os.Error)
```
 