---
layout: post
title: Go by Example(五)：杂烩2
description:  gobyexample.com 是一个非常好的入门教程，这一部分是个大杂烩，包括随机数，字符串与数字类型转换等。
category: lang
tags: golang tricks
published: true
lastUpdate: 2015-08-30
---


## 随机数/Rand ##
在之前的联系中，已经使用过随机数相关的函数了，比如随机休眠N毫秒：

```
time.Sleep(time.Milliseond * time.Duration(rand.Intn(500) ))
```
math/rand包提供了很多方法，上面的```Intn(n int)```方法就是放回n以内整数这是最常用的函数之一。另外常用的方法包括：

```
rand.Float64()  // 返回0-1之间的一个浮点数
rand.Int31()
rand.Uint32()
```
和其他平台的随机数生成器一样，rand包提供的随机数序列也是伪随机数，如果不传入一个不同的种子/seed，得到的随机数序列将是确定的。我们常把时间作为随机数种子传入方法：

```
rs := rand.NewSource(time.Now().UnixNano())
r := rand.New(rs)
// r此时是一个新的随机数生成器
// 正式使用应该这样
r.Intn(100)
```

## strconv/数字与字符串转换 ##
strconv包提供了字符串与数字类型的转换，这是一些很常用的功能，比如从"123"到123的转换，"1.23"转换到1.23;特别是在与客户端通信，使用json，或者处理从数据库查询的数据的时候，经常需要这类转换。

```golang
// parseFloat原型
func ParseFloat(s string, bitSize int) (f float64, err error)
f,_ := strconv.ParseFloat("1.23", 64)   // 转换到64位Float，不处理转换的错误

// parseInt的原型， base从2-36
func ParseInt(s string, base int, bitSize int) (i int64, err error)

// 更常用的
k,_ := strconv.Atoi("123")  // k=123
_,e := strconv.Atoi("wrongFormat")  // will return error
```
其中最常用的是```Atoi(s tring)```和```Itoa(i int)```;strconv包提供的方法大部分会有两个返回值，第一个是转换的结果，第二个是是否转换成功的error。

## URL相关 ##
