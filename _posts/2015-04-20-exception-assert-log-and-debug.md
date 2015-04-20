---
layout: post
title: 异常、断言、日志和调试
---
包含的内容：
- 处理错误
- 捕获异常
- 使用断言
- 日志
- 调试技巧


## 处理错误/捕获异常 ##
在Java中异常和错误都是派生于**Throwable类**（不是接口）的实例。
Java的异常层次结构分为两支：RuntimeException和其他（IOException）。

> 如果程序出现了RuntimeException异常，那么就一定是你的问题。

应该通过异常检查来避免RE。

Java语言规范将所有派生于Error或者RE的异常称为未检查（unchecked）的异常。其他的异常被称受检的/已检查的（checked）异常。

> 编译器将检查是否所有受检的异常都提供了异常处理器。


常见的未受检异常（RE）有：
- 错误的类型转换
- 数组访问越界
- 访问空指针

在下面的情况下应该抛出异常
1. 调用一个抛出已受检异常的方法
2. 程序运行过程中发生错误，并且利用throw语句抛出一个已受检差异常
3. 程序出现错误
4. Java虚拟机和运行时库出现的内部错误

对于那些可能被他人使用的Java方法，应该根据异常规范，在方法的首部声明这个方法可能抛出的异常。

> 方法只应该抛出受检的异常，不需要声明Java的内部错误，即从Error继承的错误，同样也不应该声明从RE继承的那些未受检异常。
> 虽然任何代码都能抛出这些异常，但是我们对这些异常没有任何控制能力。

对于未受检的异常，大部分是我们的程序逻辑的错误，我们应该花时间去修复程序的错误，而不是说明这些错误发生的可能性。

子类覆盖超类的方法时不能抛出比超类中声明的更通用的异常。

从Java SE 7开始可以在一个catch语句中捕获多个异常类型 catch(EType | Etype2 exception) {}

> try catch finally结构中，不管什么情况下，finally中的代码都会执行，即使在try中break/return了。
> 如果在try代码块中return了然后在finally中又return一次，finally中的return会覆盖之前的return的值。

> 使用异常的技巧：早抛出，晚捕获

## 断言 ##
断言机制允许在测试期间向代码中插入一些检查语句，当代码发布时，这些插入的检测语句会被自动的移除。
Java中断言的形式为：
```java
assert condition;
assert condition:string; // add report string in the end
```
可以在程序运行时通过参数设置启动或者关闭断言，启动或者禁用断言是类加载器的功能。当断言被禁用时类加载器会自动跳过断言代码。

> 断言失败是致命的、不可恢复的错误
> 断言检查只用于开发和测试阶段


## 记录日志 ##
[log4j2](http://logging.apache.org/log4j/2.x/)
[slf4j](http://www.slf4j.org/)

java.util.Logger也是不错的日志记录器

## 调试技巧 ##
Bye

