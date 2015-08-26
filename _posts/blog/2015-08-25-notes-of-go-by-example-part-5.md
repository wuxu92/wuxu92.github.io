---
layout: post
title: Go by Example(五)：  常用package
description:  gobyexample.com 是一个非常好的入门教程，里面有很多基础的知识，这里主要记录一些比较有“新意”的点。这是第五部分。这些笔记记得越来越细了，本来只打算记一些比较有新意的东西的，现在想记就记了。
category: lang
tags: golang tricks
published: false
lastUpdate: 2015-08-26
---

## 排序包/sort ##
go内置的sort包提供了内置类型和用户自定义类型的排序功能。需要注意的sort包的方法对传入的参数排序后直接修改参数，而不是返回一个新的数组。可以这么理解：一般传入排序的是一个数组，数组一般作为引用传递，而对于引用传递的修改直接反映在原数组上。
sort包提供对数组排序的函数，函数名为待排序数组元素类型加s，参数为待排序数组，比如

```
strs := []string{"bcd", "abk", "opq", "hij"}
sort.Strings(strs)
// strs has been sorted
sort.StringsAreSorted(strs) // true
```
同样对于int数组的排序方法为```sort.Ints(ints)``` sort包还提供了一个判断数组是否排序的系列方法，其方法名为参数类型加s加AreSorted(); 比如 ```IntsAreSorted(ints), StringsAreSorted(strs)```

## 给自定义类型排序 ##
类似于java与其他面向对象语言可以为实现了sortable接口的对象数组进行排序，go也提供了为自定义类型数组进行排序的方法。```sort.Sort(myArray)```中myArray是一个自定义的类型，它是某种类型的数组类型。比如:

```golang
type myType []string
```
要实现排序，也需要为这种类型绑定一些方法，就像实现sortable接口的方法一样。实际上绑定这些方法也是实现go内置定义的 [sort.Interface](https://golang.org/pkg/sort/#Interface "https://golang.org/pkg/sort/#Interface"), 要实现Len（） int, Swap和Less() bool三个方法。
实际上Go内置的sort.Sort()方法使用了泛型实现，虽然go语言本身不支持泛型。


```
func (m myType) Len() int {
  return len(m)
}
func (m myType) Swap(i, j int) {
  m[i], m[j] = m[j], m[i]
}
func (m myType) Less(i, j int) bool {
  return len(m[i]) < len(m[j])
}
```
这样让myType类型实现了sort.Interface接口。可以用sort.Sort()来排序了。

```golang
fruits := []string{"kiwi", "apple", "plum"}
sort.Sort(myType(fruits))
```
这里也算是一种编程pattern，用来实现排序。

