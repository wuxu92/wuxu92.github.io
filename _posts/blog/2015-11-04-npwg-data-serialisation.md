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
func Unmarshal(b []byte, val interface{}) (rest []byte, err error)
```
第一个方法将数据(val)整理为一个序列化的字节数组；第二个方法就是反序列化了。注意由于Go是静态类型的语言，与PHP和JS中的反序列化不太一样，Unmarshal的第一个参数的类型是interface{}，在实际反序列化时需要做类型检查，因为要把反序列化得到一个序列化时输入的对象相同的类型的变量中。其一个示例如下：

```
import (
	"fmt"
	"encoding/asn1"
)
func main() {
	serialed, e := asn1.Marshal(12)
	CheckErr(e)
	fmt.Println("serialed data", serialed)   // [2 1 12]
	var n int
	_, err := asn1.Unmarshal(serialed, &n)
	CheckErr(err)
	fmt.Println("After unmarshal", n)  // 12
}
```
要注意对于基础类型的Unmarshal需要取地址符，否则会panic。
要知道序列化支持的数据类型跟Go对其的实现程度有关。asn定义了很多的字符集和类型，但是Go只支持 PrintableString 和 IA5String(ASCII)。注意它不能实现unicode字符的序列化。

asn1可以序列化结构体(struct)，但是**要求结构体的所有字段都是可导出的**（Exportable）即首字母都是大写的，如果有非导出的小写字母field则会unmarshal的时候panic。

```
type Person struct {
	Name string
	Age int
	gender int  // cause unmarshal panic
}

func main() {
	me := Person{"wuxu", 23, 1}
	serialMe, err := asn1.Marshal(me)
	CheckErr(err)
	
	var anotherMe = new(Person)
	_, err = asn1.Unmarshal(serialMe, anotherMe) // panic: reflect: reflect.Value.SetInt using value obtained using unexported field

	CheckErr(err)
	
	fmt.Println("unmarshal me", anotherMe)
}
```
还有要注意的是序列化的时候只考虑属性的类型，而不考虑属性的变量名。所以对于只是属性名不同的结构体的序列化字节数组可以互相反序列化操作的。

## JSON ##
JSON 应该都比较熟悉了，它是一种很方便的JavaScript对象标记，本是用来在js系统之间传递数据的轻量级方法，它使用基于文本的格式且具有足够的一般性，被许多的编程语言用作数据序列化方法。
关于json标记的具体语法可以看[官方的文档](http://www.json.org/)，有很多非常好理解的图示说明。
下面是一个序列化json的示例

```

```