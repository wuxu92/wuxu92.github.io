---
layout: post
title: Go 网络编程
description: 对于任何一门语言，网络编程都是重要的一部分。对于Go来说其天生的高并发网络编程更是充满魅力。所以今天开学学习Go网络编程部分，教材是 Jan Newmarch 的 Network programming with Go 的pdf文档。
category: post
tags: network golang go
published: true
lastUpdate: 2015-10-29
---
![](/images/golang/gopher-banner-small.jpg)

{{ page.description }} 这份文档在 [这里](https://jan.newmarch.name/go/) 可以找到，由 Astaxie 翻译的中文版也在[这个网站的这个目录](https://jan.newmarch.name/go/zh/index.html)， 当然也可以在[astaxie的github上](https://github.com/astaxie/NPWG_zh)找到。

我使用英文的pdf，因为没有找到中午的pdf，而使用pdf比网页要方便些，方便做笔记和添加备注，并且这类文档的英文应该也不会太难。

根据pdf的结构，一节一节往后走。各章节组成如下：

1. 架构
2. Go语言概括
3. Socket编程
4. 数据序列化
5. 应用层协议
6. 管理字符集与编码
7. 安全
8. HTTP
9. 模板
10. 一个完整的Web服务
11. HTML
12. XML
13. 远程过程调用RPC
14. 网络 Channels
15. Web Sockets

其中前两张只简单的过一下，重要的是这几节： Socket, 序列化，应用层协议，安全，HTTP。 RPC与Web Sockets。其他章节看起来简单过一下就可以了。

## 基础/架构 ##
不同领域的程序员会有不同的技能树～批处理程序，Gui开发，游戏开发，分布式系统开发等不同的领域必然各不相同的。干什么活呢需要不同的工具，有各自的方法和模式。我们现在要考虑的就是高层的分布式系统的架构。

分布式系统的相关概念已经比较熟悉了，但是还没有真正的开发过一个分布式系统。因为光知道原理就觉得这个东西很难。网络通信需要协议来为高层应用提供支持，关于ISO/OSI协议栈也比较熟悉了就不多说了。现在用的最多的是TCP/IP协议栈，比OSI简单很多，它只有4层。(注意计算机网络中学的5层协议是OSI和TCP/IP为了教学混合出来的)。分别是： 硬件层，网络层，传输层，应用层。

**面向连接和无连接**。这是两个很重要的概念。面向连接是在会话时会建立一条连接，交互的通信在这个连接上进行。当会话结束是连接断开，比如TCP；而无连接的通信是每一个消息都是独立发送的，就像发邮件一样，比如IP协议，通常面向连接的服务是基于无连接服务。要注意区分面向连接和持久连接。面向连接是通信中复用一条链路，而持久连接是多个会话公用一次连接，参考http的keep-alive。

### 消息传递 ###
在网络服务中消息传递是很重要很基础的部分。其实Unix 管道通信就可以看作是一类消息传递，它直接传输二进制流。消息传递是很基础的功能，一般通过消息传递来共享。一个很重要的准则就是

> 不要通过共享来通信，而是通过通信来共享


![](/images/golang/messages.png)
消息传递用在大部分cs架构中，X window system(Linux的图形界面服务)就是这样的机制。server等待用户发送的消息（鼠标键盘信息），然后处理消息，返回结果。

### 远程过程调用(RPC) ###
