---
layout: post
title: "Java-JVM内存结构 vs Java内存模型 vs Java对象模型"
date: 2018-07-10 12:57:13
categories: java
tags: 
- java
- jvm
---

## 一、前言
　　本文主要参考于 Hollis 的文章[《【JVM】JVM内存结构 VS Java内存模型 VS Java对象模型》](http://www.cnblogs.com/z00377750/p/9277836.html)
主要对`JVM内存结构`、`Java内存模型`、`Java对象模型`这三个概念进行概念区分和简单的介绍，之后再补充其他文章进行详细介绍。

* [JVM内存结构](#JVM内存结构)  
* [Java内存模型](#Java内存模型)    
* [Java对象模型](#Java对象模型)    

<!-- more -->

## 二、背景
　　很久没写 Java 代码了，感觉水平还停留在最初始的那个阶段。
一直以来都很想深入学习 Java，了解 JVM，了解并发编程，往往都是看了很多，没有用到，然后就忘记了。
最近项目初始阶段，不太忙，所以好好利用下时间，再次回顾一下相关知识，然后记录下来，印象更加深刻一些。

## <span id="JVM内存结构">三、JVM内存结构</span>

　　Java 虚拟机在执行Java 程序的过程中，会把它所管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途，
以及创建个销毁的时间，有的区域随着虚拟机进程的启动而存在，有的区域则依赖于用户线程的启动和结束而建立和销毁。
《Java虚拟机规范（Java SE 8）》中描述了JVM运行时内存区域结构如下：

![](/assets/img/java-jvm-memory-structure.png)

　　具体的各个区域的介绍，会在后续的文章中进行详细阐述。

## <span id="Java内存模型">四、Java内存模型</span>

　　在前面的JVM内存结构图中，我们可以看到Java堆和方法区的区域是多个线程共享的数据区域。也就是说，多个线程可能可以
操作保存在堆或者方法区中的同一个数据。这也就是我们常说的"Java的线程间通过共享内存进行通信"。

　　Java内存模型，Java Memory Model，简称JMM。JMM 是一个抽象的概念，并不像JVM内存结构一样真实存在。它描述的是一组
规则或规范，通过这组规范定义了程序中各个变量的访问方式。线程之间的共享变量存储在主内存中，每个线程都有一个私有的本地内存，
本地内存中存储了改线程对共享变量的操作副本。

![](/assets/img/java-memory-model.png){: .img-large}

　　由于采用共享内存进行通信，在通信过程中会存在一系列如可见性、原子性、顺序性等问题，而JMM就是围绕着多线程通信以及
与其相关的一系列特性而建立的模型。JMM定义了一些语法集，这些语法集映射到`Java`语言中就是`volatile`、`synchronized`等关键字。

## <span id="Java对象模型">五、 Java对象模型</span>
　　在JVM的内存结构中，对象保存在堆内存中，而我们在对对象进行操作时，其实操作的是对象的引用。
　　Java对象在JVM中的存储也是有一定的结构的，这个就是Java 对象模型。对象在堆内存的布局分为三个区域：对象头（Header）、
实例数据（Instance Data）和对齐填充（Padding）.

![](/assets/img/java-object-layout.png){: .img-large}

* 对象头（Header）  
　　包括Mark Word和元数据指针。如果对象是一个数组，那么对象头还需要有额外的空间用于存储数组的长度。  
　　Mark Word用于存储对象自身的运行时数据，例如HashCode、GC分代年龄等信息。  
　　元数据指针用于存储对象的类型指针，该指针指向它的类元数据，JVM通过这个指针确定对象是哪个类的实例。  
* 实例数据（Instance Data）  
　　实例数据部分是对象真正存储有效信息的区域，存储了代码中定义的各种字段的内容，包括从父类继承下来的字段和子类中定义的字段。  
* 对齐填充（Padding）  
　　对齐填充这部分不是必须存在的，这部分仅仅是起着占位符的作用。由于HotSpot虚拟机的自动内存管理系统要求对象的起始地址
必须是8字节的整数倍，因此当对象实例部分数据没有对齐时，就需要对剩余的部分进行填充。

　　HotSpot虚拟机中，设计了一个OOP-Klass Model。OOP（Ordinary Object Pointer）指的是普通对象指针，而Klass用来描述
对象实例的具体类型。

## 最后

　　JVM内存结构，和Java虚拟机的运行时区域有关；  
　　Java内存模型，和Java并发编程有关；  
　　Java对象模型，和Java对象在虚拟机中的表现形式有关。  

　　这三个概念，很多时候会被人混淆。先初步了解清楚这些概念，便于后面的深入的学习。

## References
* [【JVM】JVM内存结构 VS Java内存模型 VS Java对象模型](http://www.cnblogs.com/z00377750/p/9277836.html)  
* [JVM内存结构——运行时数据区](http://www.cnblogs.com/zhengbin/p/5617023.html)  
* [Java内存区域与Java内存模型](https://blog.csdn.net/calledwww/article/details/79368966)  
* [深入理解Java内存模型（一）——基础， 程晓明](http://www.infoq.com/cn/articles/java-memory-model-1)  
* [Java对象内存布局解析](https://segmentfault.com/a/1190000007652363)  
* [Java对象模型, 张小凡](http://www.cnblogs.com/qingshanli/p/9250491.html)  

