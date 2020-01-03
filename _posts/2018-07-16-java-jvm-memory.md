---
layout: post
title: "Java-JVM-内存结构"
date: 2018-07-16 15:39:08
categories: java
tags: 
- java
- jvm
---

## 一、前言
　　上一篇《[Java-JVM-类加载机制](http://zhangyuyu.github.io/java-jvm-classloader/)》讲述了 JVM 类加载的过程，类加载的最终产品是位于堆区中的Class对象，本篇主要针对于Java 虚拟机运行时数据区域展开：

* 程序计数器
* Java 虚拟机栈
* 本地方法栈
* Java 堆
* 方法区
* 运行时常量池
* 直接内存
<!-- more -->

## 二、背景
　　最近准备巩固已学的 Java 知识，同时《[面试为什么需要了解JVM](https://mp.weixin.qq.com/s/NsPNfNViujmjM_nzcCc0IA)》一文更加坚定了信念。

　　从《[Java - JVM内存结构 vs Java内存模型 vs Java对象模型](http://zhangyuyu.github.io/java-jvm-vs-memory-model-vs-java-object-model/)》开始，先准备对JVM相关的知识点进行回顾：

* [Java-JVM-类加载机制](http://zhangyuyu.github.io/java-jvm-classloader/)
* **Java-JVM-内存结构**（本篇）
* [Java-JVM-GC算法](http://zhangyuyu.github.io/java-jvm-gc/)
* [Java-内存模型](http://zhangyuyu.github.io/2018/07/22/java-memory-model/)

## 三、运行时数据区域

　　Java 虚拟机在执行Java 程序的过程中，会把它所管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途，以及创建个销毁的时间，有的区域随着虚拟机进程的启动而存在，有的区域则依赖于用户线程的启动和结束而建立和销毁。

![](/assets/img/java-jvm-memory-structure.png)

### 1. 程序计数器（Program Counter Register）
　　程序计数器是一个比较小的内存区域，用于指示当前线程所执行的字节码执行到了第几行， 可以理解为是**当前线程的行号指示器**。字节码解释器在工作时，会通过改变这个计数器的值来取下一条语句指令。

　　由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，**每条线程都需要有一个独立的程序计数器**，各条线程之间的计数器互不影响，独立存储，我们称这类内存区域为"线程私有"的内存。

　　如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是Natvie方法，这个计数器值则为空（Undefined）。

　　此内存区域是唯一一个在Java虚拟机规范中没有规定任何`OutOfMemoryError`情况的区域。

### 2. Java 虚拟机栈（Java Virtual Machine Stacks）
　　`Java虚拟机栈`，也是线程私有的，它的生命周期与线程相同。描述的是**Java方法执行的内存模型**：每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作栈、动态链接、方法出口等信息。每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

* 如果请求线程分配的容量超过JVM栈允许的最大容量，抛出`StackOverflowError`异常。
* 如果JVM栈可以动态扩展，扩展的动作也已经尝试过，但是没有申请到足够的内存，则抛出`OutOfMemoryError`异常。

### 3. 本地方法栈（Native Method Stacks）
　　`Java 虚拟机栈`为虚拟机执行Java方法（也就是字节码）服务，而`本地方法栈`则是**为虚拟机使用到的Native方法服务**。

　　虚拟机规范中对`本地方法栈`中的方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机（譬如Sun HotSpot虚拟机）直接就把本地方法栈和虚拟机栈合二为一。

　　与`Java虚拟机栈`一样，`本地方法栈`区域也会抛出`StackOverflowError`和``OutOfMemoryError异常。`

### 4. Java堆（Java Heap）
　　对于大多数应用来说，`Java堆`是Java虚拟机所管理的内存中最大的一块。Java堆是被所有**线程共享**的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是**存放对象实例**，几乎所有的对象实例都在这里分配内存。

　　`Java堆`是垃圾收集器管理的主要区域，因此很多时候也被称做`"GC堆"`。如果从内存回收的角度看，由于现在收集器基本都是采用的分代收集算法，所以Java堆中还可以细分为：堆被划分成两个不同的区域：年轻代 ( Young )、老年代 ( Tenured)。年轻代 ( Young ) 又被划分为三个区域：Eden、From Survivor、To Survivor。 这样划分的目的是为了使 JVM 能够更好的管理堆内存中的对象，包括内存的分配以及回收。详细可参考《下一篇[Java JVM GC算法]()》

　　如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出`OutOfMemoryError`异常。

### 5. 方法区（Method Area）
　　方法区（Method Area）与Java堆一样，是各个**线程共享**的内存区域，它用于**存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据**。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做Non-Heap（非堆），目的应该是与Java堆区分开来。

　　对于习惯在`HotSpot虚拟机`上开发和部署程序的开发者来说，很多人愿意把方法区称为`"永久代"（Permanent Generation）`，本质上两者并不等价，仅仅是因为HotSpot虚拟机的设计团队选择把GC分代收集扩展至方法区，或者说使用永久代来实现方法区而已。

　　当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。

### 6. 运行时常量池（Run-Time Constant Pool）
　　`运行时常量池`是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等信息外，还有一项是常量池（Constant Pool Table），用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。

　　运行时常量池具有动态性，并非只有Class文件中的内容才能进入运行时常量池，运行期间也能将新的常量放入池中。如String.intern（）方法。

　　存储区域不够用时候抛出OutOfMemoryError异常。

### 7. 直接内存（Direct Memory）
　　`直接内存`并不是虚拟机内存的一部分，也不是Java虚拟机规范中定义的内存区域。

　　在NIO中，引入了一种基于通道和缓冲区的I/O方式，它可以使用native函数直接分配堆外内存，然后通过一个存储在java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。这样能够避免Java堆和Native堆来回复制数据，在某些场景可以显著提高性能。

　　直接内存不受任何虚拟机参数控制，但很明显，你不能大于物理内存大小。如果超过，会出现`OutOfMemoryError`。

## 四、运行时内存区分类
　　上面在运行时数据区域的各部分介绍里，说明了线程共享情况和异常情况。

　　根据线程共享可以归类如下：
　　![](/assets/img/java-jvm-memory-thread.png){: .img-medium}

　　根据异常可以归类如下：
　　![](/assets/img/java-jvm-memory-error.png){: .img-large}

## 最后
　　本篇主要是针对JVM 内存结构的各个区域（程序计数器、Java 虚拟机栈、本地方法栈、Java 堆、方法区、运行时常量池）进行介绍，其中程序计数器、Java 虚拟机栈、本地方法栈是线程私有的，Java 堆、方法区、运行时常量池是线程共享的。程序计数器是唯一一个在Java虚拟机规范中没有规定任何`OutOfMemoryError`情况的区域。

　　下一篇将讲述《[Java-JVM-GC算法](http://zhangyuyu.github.io/java-jvm-gc/)》

## References
* [JVM内存结构, 纯洁的微笑](http://www.ityouknow.com/jvm/2017/08/25/jvm-memory-structure.html)
* [【JVM】Java内存模型, 风动静泉](https://www.cnblogs.com/z00377750/p/9180923.html#autoid-4-0-0)

