PHP本身是一种弱类型的语言，可以在程序中改变变量存储的值的类型。那么这些变量在PHP的底层是如何实现的呢，理解内核变中变量的实现机制将有利于我们理解PHP的变量系统。

## 变量的类型 ##
对PHP有一定理解的同学应该都已经知道了PHP在内核中是使用zval这个结构体来存储变量的，也就是说不同的变量在底层都是一个zval结构体，它定义在Zend/zend.h中：

```
struct _zval_struct {
    zvalue_value value; /* 变量的值 */
    zend_uint refcount__gc;
    zend_uchar type;    /* 变量当前的数据类型 */
    zend_uchar is_ref__gc;
};
typedef struct _zval_struct zval;
 
//在Zend/zend_types.h里定义的：
typedef unsigned int zend_uint;
typedef unsigned char zend_uchar;
```

其中保存变量的值的value则是zvalue_value类型，它是一个union：

```
typedef union _zvalue_value {
    long lval;  /* long value */
    double dval;    /* double value */
    struct {
        char *val;
        int len;
    } str;
    HashTable *ht;  /* hash table value */
    zend_object_value obj;
} zvalue_value;
```
PHP使用zvalue_value实现8种数据类型，这些类型在内核中分别对应特定的常量。通过这个union可以理解常用的类型判断函数： ```is_null, is_bool, is_long, is_double, is_string, is_array, is_object, is_resource``` 他们的效率应该是很高的。同时也可以猜想gettype函数的实现大概使用了Z_TYPE_P宏，这个宏大概和zval结构的type有关
在Zend/zend_operators.h中定义的宏：

```
#define Z_TYPE(zval)        (zval).type
#define Z_TYPE_P(zval_p)    Z_TYPE(*zval_p)
#define Z_TYPE_PP(zval_pp)  Z_TYPE(**zval_pp)
```

> 注意，虽然在32位系统中，signed long的存储范围是-2147483648～2147483647 的整数，但是要注意，如果整数变量超出了这个范围并不会直接溢出，而是会转换程double类型继续计算。

PHP内核还同时在zval结构里保存着字符串的实际长度， 这个设计使PHP可以在字符串中嵌入‘\0’字符，也使PHP的字符串是二进制安全的， 可以安全的存储二进制数据

## 变量的值 ##
string型变量比较特殊，因为内核在保存String型变量时，不仅保存了字符串的值，还保存了它的长度， 所以它有对应的两种宏组合STRVAL和STRLEN，即：Z_STRVAL、Z_STRVAL_P、Z_STRVAL_PP与Z_STRLEN、Z_STRLEN_P、Z_STRLEN_PP。 前一种宏返回的是char *型，即字符串的地址；后一种返回的是int型，即字符串的长度。
Array型变量的值其实是存储在C语言实现的HashTable中的， 我们可以用ARRVAL组合宏（Z_ARRVAL, Z_ARRVAL_P, Z_ARRVAL_PP）这三个宏来访问数组的值。
对象是一个复杂的结构体（zend_object_value结构体），不仅存储属性的定义、属性的值，还存储着访问权限、方法等信息。 内核中定义了以下组合宏让我们方便的操作对象： OBJ_HANDLE：返回handle标识符， OBJ_HT：handle表， OBJCE：类定义， OBJPROP：HashTable的属性， OBJ_HANDLER：在OBJ_HT中操作一个特殊的handler方法。
资源型变量的值其实就是一个整数，可以用RESVAL组合宏来访问它，我们把它的值传给zend_fetch_resource函数，便可以得到这个资源的操作句柄，如mysql的链接句柄等。

### 创建PHP变量 ###
在内核中是如何创建zval的呢，PHP内核中提供了一个MAKE_STD_ZVAL(pzv)宏，它使用内核的方式类申请一块内存，并将其地址赋给pzv。这个宏能自动处理内存不足的问题。
获取空间后，就可以给这个zval赋值了。旧的做法是先确定zval的类型：```Z_TYPE_P(pzv)=ISNULL``` 来设置其为null类型，再通过Z_SOMEVAL_P的宏类赋值：

```
Z_TYPE_P(pzv)=IS_BOOL;
Z_BVAL(PZV)=1;
```
不过现在PHP内核提供了更多的宏可以更方便的操作zval的值。

新宏	其它宏的实现方法

ZVAL_NULL(pvz); **(注意这个Z和VAL之间没有下划线！)**	```Z_TYPE_P(pzv) = IS_NULL;```**(IS_NULL型不用赋值，因为这个类型只有一个值就是null)**
ZVAL_BOOL(pzv, b); **(将pzv所指的zval设置为IS_BOOL类型，值是b)**	```Z_TYPE_P(pzv) = IS_BOOL; Z_BVAL_P(pzv) = b ? 1 : 0;```
ZVAL_TRUE(pzv); **(将pzv所指的zval设置为IS_BOOL类型，值是true)**	ZVAL_BOOL(pzv, 1);
ZVAL_FALSE(pzv); **(将pzv所指的zval设置为IS_BOOL类型，值是false)**	ZVAL_BOOL(pzv, 0);
ZVAL_LONG(pzv, l); **(将pzv所指的zval设置为IS_LONG类型，值是l)**	```Z_TYPE_P(pzv) = IS_LONG;Z_LVAL_P(pzv) = l;```
ZVAL_DOUBLE(pzv, d); **(将pzv所指的zval设置为IS_DOUBLE类型，值是d)**	```Z_TYPE_P(pzv) = IS_DOUBLE; Z_DVAL_P(pzv) = d;```
ZVAL_STRINGL(pzv,str,len,dup);**(下面单独解释)**	```Z_TYPE_P(pzv) = IS_STRING;Z_STRLEN_P(pzv) = len;if (dup) {Z_STRVAL_P(pzv) =estrndup(str, len + 1);} else {Z_STRVAL_P(pzv) = str;}```
ZVAL_STRING(pzv, str, dup);	```ZVAL_STRINGL(pzv, str,strlen(str), dup);```
ZVAL_RESOURCE(pzv, res);	```Z_TYPE_P(pzv) = IS_RESOURCE;```
Z_RESVAL_P(pzv) = res;

## 变量的存储方式 ##

