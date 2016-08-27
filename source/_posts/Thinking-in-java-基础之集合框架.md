---
title: Thinking in java 基础之集合框架
date: 2016-06-25 10:30:54
tags: [List,Set,Map,Collection,Iterator]
categories: java
description: "总结java.util包中的类作用！"
---
## Thinking in java 基础之集合框架
####大家都知道我的习惯，先上图说话。
![集合框架](Thinking-in-java-基础之集合框架/201606251033.gif)
<!--more-->
## 集合简介(容器)

把具有相同性质的一类东西，汇聚成一个整体，就可以称为集合，例如这里有20个苹果，我们把每一个苹果当成一个东西（一个对象），然后我们借用袋子把这20个苹果装起来，而这个袋子就是集合（也叫容器）。然后呢，我们按照不同的方法装，就是不同的框架。

> 换句话说，集合框架就是数据结构的实现。

## 链表(数据结构)

### LinkedList的结构

```
public class chain {
private class Data{
    private Object obj;
    private Data next = null;
    Data(Object obj){
        this.obj = obj;
    }
}
private Data first = null;
public void insertFirst(Object obj){
    Data data = new Data(obj);
    data.next = first;
    first = data;
}

public Object deleteFirst() throws Exception{  
    if(first == null)
        throw new Exception("empty!");  
    Data temp = first;
    first = first.next;
    return temp.obj;
}  

public Object find(Object obj) throws Exception{  
    if(first == null)  
        throw new Exception("LinkedList is empty!");  
    Data cur = first;  
    while(cur != null){  
        if(cur.obj.equals(obj)){  
            return cur.obj;  
        }  
        cur = cur.next;  
    }  
    return null;  
}  

public void remove(Object obj) throws Exception{  
    if(first == null)  
        throw new Exception("LinkedList is empty!");  
    if(first.obj.equals(obj)){  
        first = first.next;  
    }else{  
        Data pre = first;  
        Data cur = first.next;  
        while(cur != null){  
            if(cur.obj.equals(obj)){  
                pre.next = cur.next;  
            }  
            pre = cur;  
            cur = cur.next;  
        }  
    }  
}  

public boolean isEmpty(){  
    return (first == null);  
}
public void display(){
    if(first == null)
        System.out.println("empty");
    Data cur = first;
    while(cur != null){
        System.out.print(cur.obj.toString() + " -> ");
        cur = cur.next;
    }
    System.out.print("\n");
}
public static void main(String[] args) throws Exception {
    chain ll = new chain();
    ll.insertFirst(4);
    ll.insertFirst(3);
    ll.insertFirst(2);
    ll.insertFirst(1);
    ll.display();
    ll.deleteFirst();
    ll.display();
    ll.remove(3);
    ll.display();
    System.out.println(ll.find(1));
    System.out.println(ll.find(4));
}
}
```

## Collection

### List
保存输入的顺序，而且可以重复的存储相关元素。
#### ArrayList(随机访问)(数组线性表)
ArrayList数组线性表的特点为:类似数组的形式进行存储，因此它的随机访问速度极快。
ArrayList数组线性表的缺点为:不适合于在线性表中间需要频繁进行插入和删除操作。因为每次插入和删除都需要移动数组中的元素。可以这样理解ArrayList就是基于数组的一个线性表，只不过数组的长度可以动态改变而已。ArrayList线程不安全，
#### LinkedList(频繁删除添加)(链式线性表)
您要频繁的从列表的中间位置添加和除去元素，而只要顺序的访问列表元素，那么，LinkedList 实现更好。
可以这样理解LinkedList就是一种双向循环链表的链式线性表，只不过存储的结构使用的是链式表而已。
#### Vector(向量)
如果一定在多线程使用List的，您可以使用Vector，因为Vector和ArrayList基本一致，区别在于Vector中的绝大部分方法都使用了同步关键字修饰，这样在多线程的情况下不会出现并发错误哦，还有就是它们的扩容方案不同，ArrayList是通过原始容量*3/2+1,而Vector是允许设置默认的增长长度，Vector的默认扩容方式为原来的2倍。
切记Vector是ArrayList的多线程的一个替代品。
#### Stack(栈)
在各种List中，最好的做法是以ArrayList作为缺省选择。当插入、删除频繁时，使用LinkedList();Vector总是比ArrayList慢，所以要尽量避免使用。使用最多的是ArrayList。
### Set
Set子接口: 无序，不允许有重复的元素,最多允许有一个null元素对象。
#### HashSet(没有顺序)
您会使用 HashSet 存储重复自由的集合。考虑到效率，添加到 HashSet 的对象需要采用恰当分配哈希码的方式来实现hashCode()方法。虽然大多数系统类覆盖了Object中缺省的hashCode()和equals()实现，但创建您自己的要添加到HashSet的类时，别忘了覆盖 hashCode()和equals()。
#### LinkedHashSet(添加顺序会被记录)
如果想跟踪添加给HashSet的元素的顺序，LinkedHashSet实现会有帮助。 按照元素的插入顺序来访问各个元素。它提供了一个可以快速访问各个元素的有序集合。
#### TreeSet(按照比较器排序)
当您要从集合中以有序的方式插入和抽取元素时，TreeSet实现会有用处。
为了能顺利进行。添加到TreeSet的元素必须是可排序的。
在各种Set中，HashSet通常优于TreeSet（插入、查找）。只有当需要产生一个经过排序的序列，才用TreeSet。
TreeSet存在的唯一理由：能够维护其内元素的排序状态。
### Queue(队列)
### Map
Map接口用于维护键/值对(key/value pairs)。该接口描述了从不重复的键到值的映射。
#### HashMap
在Map 中插入、删除和定位元素，HashMap 是最好的选择。
#### LinkedHashMap(包含插入顺序)
以插入顺序将关键字/值对添加进链接哈希映像中
#### TreeMap(自定义顺序)
但如果您要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。
#### WeakHashMap
它使用WeakReference(弱引用)来存放哈希表关键字。使用这种方式时，当映射的键在 WeakHashMap 的外部不再被引用时，垃圾收集器会将它回收，但它将把到达该对象的弱引用纳入一个队列。WeakHashMap的运行将定期检查该队列，以便找出新到达的 弱应用。当一个弱引用到达该队列时，就表示关键字不再被任何人使用，并且已经被收集起来。然后WeakHashMap便删除相关的映射。
#### HashTable

在各种Map中HashMap用于快速查找，使用的最多。
### Array
我们都知道，由于Array(数组)通常意义上讲只是一个单纯的线性序列，又基于Native(本地方法)，凭此它的效率历来便号称Java中最高。所以通常我们也都承认Java中效率最高的存储方式就是使用数组。但是，由于数组初始化后大小固定，索引不能超出下标，缺少灵活的扩展功能等原因，使得很多人放弃了数组的使用， 转而使用Collection,List,Map,Set等接口处理集合操作。

> 当元素个数固定，用Array，因为Array效率是最高的。

比较器(Comparator和Comparable接口)
在“集合框架”中有两种比较接口：Comparable接口和Comparator接口。像String和Integer
等Java内建类实现 Comparable接口以提供一定排序方式，但这样只能实现该接口一次。对于那些没有实现Comparable接口的类、或者自定义的类，您可以通过 Comparator接口来定义您自己的比较方式。
### Comparable接口
在java.lang包中，Comparable接口适用于一个类有自然顺序的时候。假定对象集合是同一类型，该接口允许您把集合排序成自然顺序。

(1) int compareTo(Object o): 比较当前实例对象与对象o，如果位于对象o之前，返回负
值，如果两个对象在排序中位置相同，则返回0，如果位于对象o后面，则返回正值
在 Java 2 SDK版本1.4中有二十四个类实现Comparable接口。下表展示了8种基本类型的自然排序。
虽然一些类共享同一种自然排序，但只有相互可比的类才能排序。类排序 BigDecimal,BigInteger,Byte, Double, Float,Integer,Long,Short 按数字大小排序
Character 按 Unicode 值的数字大小排序
String 按字符串中字符 Unicode 值排序
利用Comparable接口创建您自己的类的排序顺序，只是实现compareTo()方法的问题。通常就是依赖几个数据成员的自然排序。同时类也应该覆盖equals()和hashCode()以确保两个相等的对象返回同一个哈希码。
### Comparator接口
若一个类不能用于实现java.lang.Comparable，或者您不喜欢缺省的Comparable行为并想提供自己的排序顺序(可能多种排序方式)，你可以实现Comparator接口，从而定义一个比较器。

(1)int compare(Object o1, Object o2): 对两个对象o1和o2进行比较，如果o1位于o2
的前面，则返回负值，如果在排序顺序中认为o1和o2是相同的，返回0，如果o1位于o2的
后面，则返回正值“与Comparable相似，0返回值不表示元素相等。一个0返回值只是表示两个对象排在同一位置。由Comparator用户决定如何处理。如果两个不相等的元素比较的结果为零，您首先应该确信那就是您要的结果，然后记录行为。”

(2)boolean equals(Object obj): 指示对象obj是否和比较器相等。
“该方法覆写Object的equals()方法，检查的是Comparator实现的等同性，不是处于比较
状态下的对象。”
## Iterator(迭代模式)

调用iterator()方法，返回Iterator<T>对象，Iterator<T>对象有hasnext();next();方法提供循环

Collection接口有iterator()方法。Map.entrySet()返回Set<Map.Entry<K,V>>,然后调用Collection对应的iterator();方法。
```
Iterator iterator = Collection.iterator();
while(iterator.hasNext()) {
Object iter=iterator.next();
System.out.println("object=" +object);
}
Iterator iterator = Map.entrySet().iterator();
while (iterator .hasNext()) {
Map.Entry entry = (Map.Entry) iterator .next();
Object key = entry.getKey();
Object value = entry.getValue();
System.out.println("key=" + key + " value=" + value);
}
```

## 工具类Collections and Arrays(静态方法)

### Collections(常用方法)

addAll添加

shuffle混排

binarySearch二分查搜索法

reverse反转

fill 替换

max/min 找出最大/最小(根据默认的自然排序或者自定义排序规则)

sort排序(根据默认的自然排序或者自定义排序规则)

### Arrays

binarySearch二分搜索法

sort排序

copyOf复制

equals判断相等

fill指定分配、替换

toString 返货字符串

hashCode哈希吗

详情参考[中文API](http://tool.oschina.net/apidocs/apidoc?api=jdk-zh)  [oracle官网API](https://docs.oracle.com/javase/8/docs/api/)
## 总结
在实际工作中，若用到集合框架，最常用的是ArrayList,HashSet,HashMap。这三者也是首先考虑的。而且，因为TreeXXX继承SortedXXX，所以用TreeXXX都是排序的。
## 参看文献
[java集合框架的讲解](http://www.cnblogs.com/xiohao/p/4309462.html)  
[JAVA中关于链表的操作和基本算法](http://blog.csdn.net/kerryfish/article/details/24043099)  
[java的集合框架最全详解（图）](http://doc.okbase.net/DavidIsOK/archive/94766.html)  
[集合_java集合框架](http://blog.csdn.net/zsw101259/article/details/7570033)  
《Thinking in java》  
《算法与数据结构》-------java语言描述 清华大学出版社
