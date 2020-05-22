---
title: Thinking in java 高级之对象大小模型
date: 2020-05-19 10:50:29
tags:
categories: java
description: "Java对象模型，对象大小，对象头，数组对象，如何计算，怎么验证，(OOP)Ordinary Object Pointer"
---


## JOL工具

```
    <dependency>
        <groupId>org.openjdk.jol</groupId>
        <artifactId>jol-core</artifactId>
        <version>0.10</version>
    </dependency>
```

    print(ClassLayout.parseClass(Object.class).toPrintable());
    print(ClassLayout.parseInstance(new Object()).toPrintable());
    print(ClassLayout.parseInstance(Object.class).toPrintable());//这个实际上是class对象

```

-------------------------
java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0    12        (object header)                           N/A
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

-------------------------
java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

-------------------------
java.lang.Class object internals:
 OFFSET  SIZE                                              TYPE DESCRIPTION                               VALUE
      0     4                                                   (object header)                           01 e8 bf 80 (00000001 11101000 10111111 10000000) (-2134906879)
      4     4                                                   (object header)                           1e 00 00 00 (00011110 00000000 00000000 00000000) (30)
      8     4                                                   (object header)                           df 03 00 f8 (11011111 00000011 00000000 11111000) (-134216737)
     12     4                     java.lang.reflect.Constructor Class.cachedConstructor                   null
     16     4                                   java.lang.Class Class.newInstanceCallerCache              null
     20     4                                  java.lang.String Class.name                                (object)
     24     4                                                   (alignment/padding gap)                  
     28     4                       java.lang.ref.SoftReference Class.reflectionData                      (object)
     32     4   sun.reflect.generics.repository.ClassRepository Class.genericInfo                         null
     36     4                                java.lang.Object[] Class.enumConstants                       null
     40     4                                     java.util.Map Class.enumConstantDirectory               null
     44     4                    java.lang.Class.AnnotationData Class.annotationData                      (object)
     48     4             sun.reflect.annotation.AnnotationType Class.annotationType                      null
     52     4                java.lang.ClassValue.ClassValueMap Class.classValueMap                       null
     56    32                                                   (alignment/padding gap)                  
     88     4                                               int Class.classRedefinedCount                 0
     92   404                                                   (loss due to the next object alignment)
Instance size: 496 bytes
Space losses: 36 bytes internal + 404 bytes external = 440 bytes total

```
可以回头再过来看，先往下继续，
## 大小

### _markword

markword 
第一部分markword,用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，这部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit，官方称它为“MarkWord”。
klass 
对象头的另外一部分是klass类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例. 32位4字节，64位开启指针压缩或最大堆内存<32g时 4字节，否则8字节
数组长度（只有数组对象有） 4字节
如果对象是一个数组, 那在对象头中还必须有一块数据用于记录数组长度.int最大值2g，2^31，java数组（包含字符串）最长2g

### KlassOop

### 实际数据大小

#### 基本类型

字节
1 2 4 8
byte short int lang
boolean char float double
ref 4

#### 对象类型
String =char[] + hash
Object = ref 4
上一个对象添加引用，本对象计算

#### 数组类型

int[] 
基本类型 添加数组长度4 加上基本类型大小
Object[]
对象数组  添加数组长度 + ref

## 计算

### 本身大小

```
package io.github.muxiaobai.java.objectsize;


import org.openjdk.jol.info.ClassLayout;

public class JOLPeople {
    int age = 20;
    String name = "Xiaoming";
    public static void main(String[] args) {
        print(ClassLayout.parseInstance(new JOLPeople()).toPrintable());

        print(ClassLayout.parseInstance(new String("Xiaoming")).toPrintable());
        char[] chars = new char[8];
        chars[0] = 'X';
        chars[1] = 'i';
        chars[2] = 'a';
        chars[3] = 'o';
        chars[4] = 'm';
        chars[5] = 'i';
        chars[6] = 'n';
        chars[7] = 'g';

        print(ClassLayout.parseInstance(chars).toPrintable());
    }

    static void print(String message) {
        System.out.println(message);
        System.out.println("-------------------------");
    }
}
```

/***************************************************************************/

JOLPeople 本身
 -------------------------
 io.github.muxiaobai.java.objectsize.JOLPeople object internals:
 OFFSET  SIZE               TYPE DESCRIPTION                               VALUE
 0     4                    (object header)                           05 00 00 00 (00000101 00000000 00000000 00000000) (5)
 4     4                    (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
 8     4                    (object header)                           af f3 00 f8 (10101111 11110011 00000000 11111000) (-134155345)
 12     4                int JOLPeople.age                             20
 16     4   java.lang.String JOLPeople.name                            (object)
 20     4                    (loss due to the next object alignment)
 Instance size: 24 bytes
 Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

 -------------------------
 _mark +oop  + 4(age的value)+ 4(name引用)
 8     + 4   +  4            +4          + 4(lose) =  24



### ShallowSize 

 String 是一个char[] 数组 和hash 两个属性

 The value is used for character storage.
private final char value[];

 Cache the hash code for the string
private int hash; // Default to 0

String

 -------------------------
 java.lang.String object internals:
 OFFSET  SIZE     TYPE DESCRIPTION                               VALUE
 0     4          (object header)                           05 00 00 00 (00000101 00000000 00000000 00000000) (5)
 4     4          (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
 8     4          (object header)                           da 02 00 f8 (11011010 00000010 00000000 11111000) (-134216998)
 12     4   char[] String.value                              []
 16     4      int String.hash                               0
 20     4          (loss due to the next object alignment)
 Instance size: 24 bytes
 Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

 -------------------------
 _mark +oop  + 引用(char[])+ hash
 8     + 4   +  4            4 + lose =  24



 char[]数组


 -------------------------
 [C object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
 0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
 4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
 8     4        (object header)                           41 00 00 f8 (01000001 00000000 00000000 11111000) (-134217663)
 12     4        (object header)                           08 00 00 00 (00001000 00000000 00000000 00000000) (8)
 16    16   char [C.<elements>                             N/A
 Instance size: 32 bytes
 Space losses: 0 bytes internal + 0 bytes external = 0 bytes total

 -------------------------

 _mark +oop + 数组长度 + 实际数据
 8     + 4   +  4     + 8(arrLength) * 2(char) =  32


 我们可以手工计算一下JOLPeople obj = new JOLPeople()的大小：
 JOLPeople的Shallow size = 8(_mark) + 4(oop指针) +  4(age的value)+ 4(name引用) + 4(lose) = 24
 String对象的长度 = 8(_mark) + 4(oop指针) + 4(char[8]引用) +4(hash) +4(lose) =  24
 char[]对象长度 =  8(_mark) + 4(oop指针) +  4(数组长度占4个字节) + 8*2(value) = 32
 所以JOLPeople实际占用的空间 = 24 + 24 + 32 = 80



验证：参考:[github ShallowSize.java]()