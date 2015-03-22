---
layout: post
title: 集合
---

## 集合接口 ##
Java设计中 ，集合的接口与其实现分离开，所以在构建集合对象时，使用具体的类才有意义，但是可以使用接口类型存放集合的引用。

> java 类库中由一组Abstract开头的类，这些是为类库设计者而设计的，如果要实现自的对应类，可以将扩展这些abstract类。

集合类的基本接口是Collection接口，它有两个基本方法：
```java	
public interface Collection<E> {
	boolean add(E element);
	Iterator<E> iterator();
}
```
从代码中可以看出集合类的基本性质：通过add添加元素，通过迭代器遍历元素，并且它是一个泛型接口。

1. 添加元素时，如果添加的元素改变了集合就返回true，如果没有改变就返回false。
2. 集合中不允许存在重复的对象。

迭代器接口基本方法及其使用：

```java
public interface Iterator<E> {
	E next();
	boolean hasNext();
	void remove();
}

while (iter.hasNext()) {
	Type t = iter.next();
	...
}
```
Java5以后可以用for each来简练地遍历iterator。

```
for (Type t : inter ) { ... }
```

java的迭代器中查找一个元素的唯一方法就是调用next()方法，**应该认为Java迭代器在两个元素之间，当调用next()方法时，就越过下一个元素，并返回那个元素的引用**。

> 迭代器的remove()方法会删除上一次调用next()方法时返回的那个元素。所以**在调用remove()之前必须要先调用next()，否则会抛出IllegalStateException**


#### 集合类型及描 ####
1. ArrayList             可以动态增长和缩减的索引序列
2. LinkedList            可以在任何位置上进行高效的插入，删除操作的有序序列
3. ArrayDeque 	一种用循环数组实现的双端队列
3. HashSet               没有重复元素的无序集合
4. TreeSet               有序集
5. EnumSet               包含枚举类型元素集 
6. LinkedHashSet         可以记住元素被插入顺序的集合
7. PriorityQueue         可以高效的移除最小元素的集合
8. HashMap               一种存放键值对应关系的数据结构
9. TreeMap               键有序的map
10. EnumMap               键属于枚举类型的map集合  
11. LinkedHashMap	一种可以记住键值项添加次序的映射表
12. WeakHashMap	一种其值无用武之地后可以被立即回收的映射表
13. IdentityHashMap		一找用 == 而不是用equals比较简直的映射表

### LinkedList ###
LinkedList是链表的一种实现，比较特殊的是它的迭代器：
```java
ListIterator<Base> iter = baseList.listIterator();
    if (iter.hasPrevious()) {
    	Base lastBase = iter.previous();
	print(lastBase);
}
```
- ListIterator 是Iterator的子类，实现了previous方法，其add方法总是会修改对象。
- 上面的代码并不会输出内容，说明ListIterator的第一个元素的previous()并不会返回最后一个元素。
- 虽然LinkedList提供了get()方法可以随机访问list中的元素，但是这种用法并不符合LinkedList的设计，效率极低，不应该使用，这是应该考虑ArrayList

### 散列表 ###
Java中散列表使用链表数组实现，每个列表被称为桶（bucket）。要朝着表中对应的位置,要先计算元素的散列码,再与桶的总数取余,所得结果就是这个元素的在桶中的索引.
**通常将桶的大小设置为预计元素个数的75%--150%,并且最好设置为一个素数**

HashSet：实现了基于散列表的集,使用add()方法添加元素，使用contains()方法来快速查看某个元素是否已经添加。


TreeSet：树集是一个有序集合，在对集合进行遍历时，将会按照排序后的顺序输出。树集的排序是使用红黑树实现的。

### 对象的比较：Comparable接口 ###
Comparabel接口是一个非常重要的接口，几乎所有的类应该都扩展了这个类。
```java
public interface Comparable<T> {
	int compareTo(T other);
}
```
> 一般来说，a在b前面返回负数，a等于b返回0， a在b之后返回正数


对于自定义的类，需要自己实现Comparable接口的comparerTo()方法。

如果需要对一个类进行排序而这个类又没有实现Comparabel接口，或者需要按照某种新的方式排序，这时可以将一个Comparator对象传递给TreeSet的构造器来实现不同的比较方法。
```java
TreeSet<Base> baseTree = new TreeSet<Base>(new Comparator<Base>() {
   @Override
   public int compare(Base o1, Base o2) {
		return o1.getAge() - o2.getAge();
   }
});
```

### 队列和优先队列 ###
Queue类是Java的队列实现，常有的方法是：offer(E e), poll(), peek();
优先队列(priority queue)中的元素按照任意顺序插入，确总是按照顺序进行检索。

> 优先队列使用了一种优雅而且高效的数据结构：堆（heap）

### 映射表Map ###
Java类库为映射表（map）提供了两种通用的实现：HashMap和TreeMap。这是两个非常常用的类。

**集合框架本身并没有将映射表视为一个集合，当时可以获得映射表的视图，包括键集、值集合、键值对集合:**
```java
Set<K> KeySet()
Collection<K> values()
Set<Map.Entry<K, V>> entrySet<> 
```
Set接口扩展了Collection接口，所以可以像任何集合一样使用keySet

常用的遍历方法：
```java
HashMap<String, Base> baseMap = new HashMap<String, Base>();
baseMap.put("wee", new Base("wee", 33));
baseMap.put("wew", new Base("wew", 23));
baseMap.put("eew", new Base("wew", 13));
Set<String> keys = baseMap.keySet();
for (String name : keys) {
    print(baseMap.get(name));
}

for (Map.Entry<String, Base> entry : baseMap.entrySet()) {
    print(entry.getKey());
}
```
### 其他散列表 ###
1. 弱散列映射表：WeakHashMap。能及时回收不再使用的元素
2. 枚举集与映射表：EnumSet是一个枚举类型元素集的高效实现。EnumSet没有构造器，可以使用静态工厂方法构造集合。键类型为枚举类型的EnumMap映射，可以高效的使用一个值数组实现

## 集合框架 ##
Java集合框架的两个基本接口：Collection和Map.0

### 视图与包装器 ###
通过视图可以获得其他的实现了集合接口和映射表接口的对象。Map的keySet()方法返回一个实现Set接口的类对象，这个类的方法对原映射表进行操作，这种集合称为**视图（view）**

1. Arrays.asList()将一个数组转换成List包装器。也可以 
```
List<String> names = Arrays.asList("ww", "uu"); 
```

一些常用的操作：

```java
Collections.nCopies(n, obj);  //Collections是具体类，不是接口，如同Arrays也是具体类
Collections.singleton(anobj)；
list.subList(start, end);
collect.clear();
headSet();
tailSet();
subMap();
headMap();
```

同步视图，用于多线程访问集合，保证集不会被破坏。类库使用视图机制来确保常规集合的线程安全，而不是实现线程安全的集合类。
Collections类的静态方法synchronizedMap()方法就是将任何一个映射表转换成具有同步访问方法的Map。

```java
Map<String, Base> syncMap = Collections.synchronizedMap(baseMap);
```

### 算法 ###
待写
