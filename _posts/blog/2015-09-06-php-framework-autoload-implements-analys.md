---
layout: post
title: PHP框架中自动加载机制实现分析
description: 之前一片文章分析过PHP中实现自动加载的方法，这几天在看lavaral的代码，发现其自动加载机制和Yii 2 的自动加载时同样的。这里索性把这种常用的自动加载实现做个简单的总结。
category: post
tags: php framework
published: true
lastUpdate: 2015-09-06
---

{{ page.description }}

关于自动加载和命名空间，是很多PHP开发者绕不过去的领域，虽然很多旧的教程并不会提及这一块，但是可以看到随着PHP的不断发展，命名被用的越来越广泛，PSR-0 和 PSR-4 规范的通过也是业界的一种发展趋势。
关于自动加载和命名空间在项目中的使用，我还和勇哥进行过多次“和颜悦色”、“亲切友好”的交流，我是希望公司都能推广使用更好的自动加载机制（现在使用的自动加载确实有点麻烦），并且希望在项目中使用命名空间，最好符合PSR-0的规范。但是由于公司很多人都没有使用命名空间的习惯，所以讲过多次交流，勇哥没削我，我也没法说服他去推广。。。。。
但是希望所有的php开发者在尽早都使用命名空间。

废话不多说了，下面开始吧。

1. 自动加载机制
2. PSR-0和PSR-4规范
3. laravel中自动流程

## 自动加载机制 ##
关于php中的自动加载机制，在之前的[一篇文章](http://wuxu92.github.io/php-using-spl-autoloader/)已经简单的介绍了。php在5.3版本才引入命名空间，有一种松散凌乱的语言慢慢向标准化进化，自动加载机制就是重要的一部分。
在之前的文章中，介绍了实现自动加载的方法，通过```spl_autoload_register```函数给系统注册一下autoloader，这些加载器通过我们自定义的函数去寻找类对应的php文件，找到文件后使用`require`方法引入文件。
这样做可以实现基本的类加载，但是不同的人去注册自动加载方法会各不相同，导致不同的php项目的加载机制实现还是五花八门，这对于模块化的开发是灾难性的，因为不同模块的加载机制都不同，整合到一起更本没法工作。

## PSR-0 和 PSR-4 ##
鉴于这些问题的存在，[php-fig](http://www.php-fig.org/),即PHP Framework Interop Group/php框架互操作小组，推出了一系列的推荐规范。其中包括推荐的，也是已经被接受的自动加载规范。
其实PHP发展了这么多年，并缺少一个官方的语言规范(language specification)，知道最近才有一个[社区维护的spec](https://github.com/php/php-langspec)。php-fig就推出了一些列的recommendation，详情参见： [http://www.php-fig.org/psr/](http://www.php-fig.org/psr/)

其实在php-fig的一接受规范中，有两个是关于自动加载的，即[PSR-0](http://www.php-fig.org/psr/psr-0/)和[PSR-4](http://www.php-fig.org/psr/psr-4/)。为什么一个工作组推出的规范会有两个不同的版本呢？这还是因为php常年的不够规范导致的。

PSR-0是比较早的一个规范，在2014年10月21日已经是一个过时的规范了，现在官方推荐使用的是PSR-4规范。但是由于一些项目已经在使用PSR-0规范实现类加载了，而且又没有升级到PSR-4兼容的规范，所以在一般的php框架都会同时支持psr-0和psr-4的自动加载实现。

### PSR-0 ###
虽然PSR-0是一个deprecated的规范，在这里还是要介绍一下，PSR-0有如下的强制要求：
1. A fully-qualified namespace and class must have the following structure \<Vendor Name>\(<Namespace>\)*<Class Name>
1. Each namespace must have a top-level namespace ("Vendor Name").
1. Each namespace can have as many sub-namespaces as it wishes.
1. Each namespace separator is converted to a DIRECTORY_SEPARATOR when loading from the file system.
1. Each _ character in the CLASS NAME is converted to a DIRECTORY_SEPARATOR. The _ character has no special meaning in the namespace.
1. The fully-qualified namespace and class is suffixed with .php when loading from the file system.
1. Alphabetic characters in vendor names, namespaces, and class names may be of any combination of lower case and upper case.

其中要注意的几点：1. namespace的顶级名是Vendor Name。 2.类名中的下划线会被翻译成路径分隔符

PSR-0的一种实现： [http://gist.github.com/221634](http://gist.github.com/221634) 这在之前的文章中已经提过了。

### PSR-4 ###
按照php-fig的[一篇文章](http://www.php-fig.org/psr/psr-4/meta/)的说法，PSR-4是PSR-0的一种补充而不是为了代替PSR-0规范。PSR-4的提出主要考虑PSR-0会导致文件夹组织层级太深，具体在上面的链接有介绍。

> The purpose is to specify the rules for an interoperable PHP autoloader that maps namespaces to file system paths, and that can co-exist with any other SPL registered autoloader. This would be an addition to, not a replacement for, PSR-0.

PSR-4中的规范：
1. The fully qualified class name MUST have a top-level namespace name, also known as a "vendor namespace".
1. The fully qualified class name MAY have one or more sub-namespace names.
1. The fully qualified class name MUST have a terminating class name.
1. Underscores have no special meaning in any portion of the fully qualified class name.
1. Alphabetic characters in the fully qualified class name MAY be any combination of lower case and upper case.
1. All class names MUST be referenced in a case-sensitive fashion.

需要注意的就是PSR-4中下划线没有特殊意义了，不再被翻译为路径分隔符，显然这是更加优雅的，因为下划线是在php 5.2及之前还不支持命名空间时的一种妥协方法。
在PSR-4中添加一条就是自动加载的实现不能抛出异常或者导致error，也不要有返回值。

> Autoloader implementations MUST NOT throw exceptions, MUST NOT raise errors of any level, and SHOULD NOT return a value.

一种PSR-4的[实现](http://www.php-fig.org/psr/psr-4/examples/)


## Laravel中的自动加载实现 ##

## 待续 ##