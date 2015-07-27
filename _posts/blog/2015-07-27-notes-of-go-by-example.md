---
layout: post
title: Go by Example 中值得注意的笔记
description:  gobyexample.com 是一个非常好的入门教程，里面有很多基础的知识，这里主要记录一些比较有“新意”的点
category: lang
tags: golang tricks
published: true
---

## 不定参数 ##
不定参数的用法已经知道了，函数可以定义如下：

```golang
func sum (nums ...int){
	total := 0
	for _, num := range nums {
		total += num
	}
	return total
}
```

这样可以计算任意个传入的int参数的和，但是有下面的用法：

```
nums := []int{1,2,3,4,5}
t := sum(nums...)
```
```nums...``` 这样的用法在之前没有遇到过，它可以用来表示， 将一个slice的元素作为函数的实参列表。

## 闭包 ##
对于使用过js的人，闭包是一个非常熟悉的概念了。js使用闭包实现内部变量的私有化，go中也有同样的目的？
与js不同的是在定义函数的地方要把返回值写明返回的函数的签名。

```golang
func intSeq() func() int {
	i := 0
	return func() int {
		i += 1;
		return i
	}
}
```
这样定义的intSeq函数调用后的返回值就是一个函数。

```golang
next := intSeq()
// next是一个命名函数了
next()
```
相比js的闭包，功能不是那么丰富，不过本质差不多。



