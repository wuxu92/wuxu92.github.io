---
layout: post
title: Golang 学习
description: 
category: post
tags: go golang lang
published: true
lastUpdate: 2015-10-22
---

## 定义函数 ##
定义函数很简单：

```golang
func message(name string) string {
	message := fmt.Sprintf("hello, %v", name)
	return message
]
```
这是最常见的声明方式，我们还可以这样：

```golang
func message(name string) (message string) {
	message = fmt.Sprintf("hello, %v", name)
	return
}
```
使用命名的返回值，可以在return部分生命变量名。

## 接口类型 ##
接口类型是`interface{}`，在声明函数时指定相应参数类型如下：

```golang
func print(v ...interface{}) {
	...
}
```
注意其中的大括号。这里还是用了可变参数，在参数类型前面的三个点号表示这是一个数量可变的参数，可变参数只能在参数列表的最后。这样接收到的v是一个切片，可以对其进行range之类的操作。

关于接口类型的类型判断，之前提到过golang的类型断言使用`x.(T)`的方式。

在golang中的强制类型转换，尤其是将interface{}转换为其他类型的时候，需要做类型断言，否则会报类似下面的错误：

```
vp := (Person)p  // p 是interface{}类型

报错:
.\variable_param_test.go:15: cannot convert p (type interface {}) to type Person: need type assertion
```
要把p转换为Person对象，应该如下：

```golang
func convert(v ...interface{}) {
	for _, p := range v {
		if t, ok := p.(Person); ok {
			// 现在t就是转换后的Person对象了
			fmt.Println("person", t.name)
		} else {
			fmt.Println("not person", p)
		}
	}
}
```

## 为类型绑定方法 ##
这部分也将过了，在对自定义类型绑定方法时的语法如下：

```golang
type Message struct {
	x string
	y *string
}

func (m *Message) Update(x, y string) {
	m.x = x
	m.y = &y
}

func (m Message) String() string {
	var y = "nil"
	if m.y != nil {
		y = *m.y
	}
	return fmt.Sprintf("%s, %s", m.x, y)
}
```

需要注意的是， Message和*Message是两种不同的类型。上面的代码为*Message类型绑定了方法`Update(x, y string)`，而为类型Message绑定了方法`String()`
当然也可以这样理解：对于需要修改类型内容，并且方法调用结束要保留这些修改的方法使用带指针的绑定。因为在调用时Go会自动对指针进行“非常乐观”的转换，实际上调用这些方法的类型并没有严格要求，也就是，对上面的绑定的方法可以如下调用：

```
m := Message{}  // 或者 m := &Message{} 也是一样的
m.Store("hello", "world")
s := m.String()
```

## Go的类型嵌入及其方法调用 ##
Go没有提供面向对象的继承机制，意味着熟悉的子类调用父类方法的思维不再适用了。
但是Go提供了一种类似于Javascript的原型继承很相似的方式，叫做"类型内嵌"，即A类型内嵌按一个B类型的变量，可以直接在A类型的变量调用B类型绑定的方法，只要方法没有冲突。

请看下面的例子：

```golang
type ColorMessage struct {
	Message
}

func TestTypeEmbed(t *testing.T) {
	cm := ColorMessage{Message{"colorful", nil}}
	cm.StorePtr("new", "Message")  // 直接调用Message绑定的方法
	fmt.Println(cm)     // 会调用Message的String()方法
}
```

更复杂一点，多个内嵌类型都有Stirng方法，会依次调用它们的String方法：

```golang
type Ring struct {
	ring string
}
func (r Ring) String() string {
	return r.ring
}

type Bell struct {
	bell string
}
func (b Bell) String() string {
	return b.bell
}

type ColorMessage struct {
	Message
	Ring
	Bell
}
func TestTypeEmbed(t *testing.T) {
	cm := ColorMessage{Message{"colorful", nil}, Ring{"morning"}, Bell{"dida, dida"}}
	cm.StorePtr("new", "Message")
	fmt.Println(cm)  // 输出: {new, Message morning dida, dida}
}
```

> Go allows **user-defined** types to declare methods on either a value type or a pointer to a value type. When methods operate on a value type the value manipulated remains **immutable** to the rest of the program (essentially the method operates on a copy of the value) whilst with a pointer to a value type any changes to the value are apparent throughout the program. 

注意其中的user-defined types,只有对用户定义类型是这样的，对于切片和数组并不适用。




