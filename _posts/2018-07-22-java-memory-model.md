---
layout: post
title: "Java-内存模型"
date: 2018-07-22 17:41:51
categories: java
tags: 
- java
- jvm

---
## 一、前言

　　前文《[Java - JVM内存结构 vs Java内存模型 vs Java对象模型](http://zhangyuyu.github.io/java-jvm-vs-memory-model-vs-java-object-model/)》中提到JVM内存结构和Java内存模型的区别。  
　　JVM内存结构，和Java虚拟机的运行时区域有关。而Java内存模型，和Java并发编程有关。《[Java-JVM-内存结构](http://zhangyuyu.github.io/Java-jvm-memory/)》一文中对JVM内存结构进行了说明，本篇将主要回顾Java的内存模型：

* 硬件的效率和一致性
* Java内存模型
* 内存模型的特性
* 重排序
* 内存屏障
* happens-before
<!-- more -->

## 二、背景
　　最近准备巩固已学的 Java 知识，同时《[面试为什么需要了解JVM](https://mp.weixin.qq.com/s/NsPNfNViujmjM_nzcCc0IA)》一文更加坚定了信念。

　　从《[Java - JVM内存结构 vs Java内存模型 vs Java对象模型](http://zhangyuyu.github.io/java-jvm-vs-memory-model-vs-java-object-model/)》开始，先准备对JVM相关的知识点进行回顾：

* [Java-JVM-类加载机制](http://zhangyuyu.github.io/java-jvm-classloader/)
* [Java-JVM-内存结构](http://zhangyuyu.github.io/Java-jvm-memory/)
* [Java-JVM-GC算法](http://zhangyuyu.github.io/java-jvm-gc/)
* **Java-内存模型**（本篇）

## 三、硬件的效率与一致性
　　在介绍Java虚拟机并发相关的内存模型之前，我们先了解一下物理计算机的并发问题，物理机遇到的并发问题与虚拟机中的情况有不少相似之处，物理机对于并发的处理方案对于虚拟机的并发实现也有着相当大的参考意义。

### 1. 高速缓存
　　由于计算机的存储设备与处理器的运算能力之间有几个数量级的差距，所以现代计算机系统都不得不加入一层读写速度尽可能接近处理器运算速度的`高速缓存（cache）`来作为内存与处理器之间的缓冲：将运算需要使用到的数据复制到缓存中，让运算能快速进行，当运算结束后再从缓存同步回内存之中没这样处理器就无需等待缓慢的内存读写了。

### 2. 缓存一致性
　　基于高速缓存的存储交互很好地解决了处理器与内存的速度矛盾，但是引入了一个新的问题`：缓存一致性（Cache Coherence）`。在多处理器系统中，每个处理器都有自己的高速缓存，而他们又共享同一主存（Main Memory）。为了保障数据的一致性，需要处理器访问缓存时候遵循一些协议，比如MSI、MESI、MOSI及Dragon Protocol等。

![](/assets/img/java-jvm-hardware-model.png)

　　我们所说的"内存模型"，可以理解为在特定的操作协议下，对特定的内存或者高速缓存进行读写访问的过程抽象。不同架构的物理机器可以拥有不同的内存模型，而Java虚拟机也有自己的内存模型。

### 3. 乱序执行优化
　　为了使得CPU内部运算单元能尽量被充分利用，CPU可能会对输入代码进行`乱序执行（Out-Of-Order Execution）`优化，CPU会在计算之后将乱序执行的结果重组，保证该结果与顺序执行的结果是一致的。  
　　与CPU的乱序执行优化类似，JVM的即时编译器中也有类似的指令重排序`（Instruction Recorder）`优化。

## 四、Java内存模型

　　Java虚拟机规范中试图定义一种Java内存模型来屏蔽掉各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台上都能达到一致的内存访问效果。

#### 1. 主内存与工作内存
　　Java内存模型的主要目标：定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样的底层细节。注意，此处的变量与Java编程语言中所说的变量有所区别，它包括实例字段、静态字段和构成数组对象的元素，但不包括局部变量与方法参数，因为后者是线程私有的，不会被共享，自然就不会存在竞争问题。

　　Java内存模型中规定了所有的变量都存储在`主内存`中，每条线程还有自己的`工作内存`（可以与前面将的处理器的`高速缓存`类比），线程的`工作内存`中保存了该线程使用到的变量到主内存副本拷贝，线程对变量的所有操作（读取、赋值）都必须在`工作内存`中进行，而不能直接读写`主内存`中的变量。不同线程之间无法直接访问对方工作内存中的变量，线程间变量值的传递均需要在`主内存`来完成，`线程`、`主内存`和`工作内存`的交互关系如下图所示，和上图很类似:

![](/assets/img/java-jvm-memory-model.png)

* 这里的`主内存`、`工作内存`与Java内存区域中的`Java堆`、`栈`、`方法区`等并不是同一个层次的内存划分
* `主内存`主要对应`java堆`中的对象实例数据部分，而`工作内存`则对应于`虚拟机栈`中的部分数据

#### 2. 内存间的交互操作
　　关于主内存与工作内存之间具体的交互协议，即一个变量如何从主内存拷贝到工作内存、如何从工作内存同步回主内存等的细节，Java内存模型定义了8种操作来完成，虚拟机实现时必须保证下面提及的每一种操作都是原子操作。  

![](/assets/img/java-jvm-memory-interaction.png)

* lock(锁定)：作用于主内存的变量，它把一个变量标志为一条线程独占的状态。
* unlock(解锁)：作用于主内存中的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
* read(读取)：作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用。
* load(载入)：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
* use(使用)：作用于工作内存的变量，它把工作内存中一个变量的值传递给执行引擎。
* assign(赋值)：作用于工作内存的变量，它把一个从执行引擎接受到的值赋给工作内存的变量。
* store(存储)：作用于工作内存的变量，它把工作内存中一个变量的值传送到主内存中，以便随后的write操作使用。
* write(写入)：作用于主内存中的变量，它把store操作从主内存中得到的变量值放入主内存的变量中。


## 五、内存模型的特性

　　Java内存模型是围绕着并发编程中原子性、可见性、有序性这三个特征来建立的，那我们依次看一下这三个特征：

### 1. 原子性（Atomicity）
　　由Java内存模型来直接保证的原子性变量操作包括read、load、assign、use、store、write。我们大致可以认为基本数据类型的访问读写是具备原子性的（例外就是long和double的非原子协定）。  
　　Java内存模型还提供了lock和unlock操作来满足这种需求，尽管虚拟机未把lock与unlock操作直接开放给用户使用，但是却提供了更高层次的字节码指令monitorenter和monitorexit来隐匿地使用这两个操作，这两个字节码指令反映到Java代码中就是同步块—synchronized关键字，因此在`synchronized`块之间的操作也具备原子性。

### 2. 可见性（Visibility）
　　可见性就是指当一个线程修改了线程共享变量的值，其它线程能够立即得知这个修改。

#### 2.1 volatile
　　Java内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值这种依赖主内存作为传递媒介的方法来实现可见性的，无论是普通变量还是volatile变量都是如此，普通变量与volatile变量的区别是volatile的特殊规则保证了新值能立即同步到主内存，以及每使用前立即从内存刷新。因为我们可以说volatile保证了线程操作时变量的可见性，而普通变量则不能保证这一点。

#### 2.2 synchronized
　　同步块的可见性是由"对一个变量执行unlock操作之前，必须先把此变量同步回主内存中(执行store和write操作)"这条规则获得的。

#### 2.3 final
　　final关键字的可见性是指：被final修饰的字段是构造器一旦初始化完成，并且构造器没有把"this"引用传递出去，那么在其它线程中就能看见final字段的值。

### 3. 有序性（Ordering）
　　Java内存模型中的程序天然有序性可以总结为一句话：`如果在本线程内观察，所有操作都是有序的；如果在一个线程中观察另一个线程，所有操作都是无序的。`前半句是指"线程内表现为串行语义"（Within-Thread As-If-Serial Semantics ），后半句是指"指令重排序"现象和"工作内存主主内存同步延迟"现象。

　　Java语言提供了`volatile`和`synchronized`两个关键字来保证线程之间操作的有序性，`volatile`关键字本身就包含了禁止指令重排序的语义，而`synchronized`则是由"一个变量在同一时刻只允许一条线程对其进行lock操作"这条规则来获得的，这个规则决定了持有同一个锁的两个同步块只能串行地进入。


## 六、重排序

　　指令重排是指JVM在编译Java代码的时候，或者CPU在执行JVM字节码的时候，对现有的指令顺序进行重新排序。指令重排的目的是为了在不改变程序执行结果的前提下，优化程序的运行效率。需要注意的是，这里所说的不改变执行结果，指的是不改变单线程下的程序执行结果。重排序分成三种类型：

* 编译器优化的重排序。编译器在不改变单线程程序语义放入前提下，可以重新安排语句的执行顺序。
* 指令级并行的重排序。现代处理器采用了指令级并行技术来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
* 内存系统的重排序。由于处理器使用缓存和读写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

　　从Java源代码到最终实际执行的指令序列，会经过下面三种重排：

![](/assets/img/java-jvm-reorder.png)

* 对于编译器重排序，JMM的编译器重排序规则会禁止特定类型的编译器重排序。
* 对于处理器重排序，JMM的处理器重排序规则会要求java编译器在生成指令序列时，插入特定类型的内存屏障（Memory barriers，intel称之为内存栅栏Memory Fence）指令，通过内存屏障指令来禁止特定类型的处理器重排序。


## 七、内存屏障

　　内存屏障也称为内存栅栏或栅栏指令，是一种屏障指令，它使CPU或编译器对屏障指令之前和之后发出的内存操作执行一个排序约束。  
　　硬件层的内存屏障分为两种：Load Barrier 和 Store Barrier即读屏障和写屏障。内存屏障有两个作用：

* 阻止屏障两侧的指令重排序；
* 强制把写缓冲区/高速缓存中的脏数据等写回主内存，让缓存中相应的数据失效。

　　Java内存模型把内存屏障分为LoadLoad、LoadStore、StoreLoad和StoreStore四种：

|屏障类型|指令示例|说明|
|--- |--- |--- |
|LoadLoad Barriers|Load1; LoadLoad; Load2|确保Load1数据的装载，之前于Load2及所有后续装载指令的装载。|
|StoreStore Barriers|Store1; StoreStore; Store2|确保Store1数据对其他处理器可见（刷新到内存），之前于Store2及所有后续存储指令的存储。|
|LoadStore Barriers|Load1; LoadStore; Store2|确保Load1数据装载，之前于Store2及所有后续的存储指令刷新到内存。|
|StoreLoad Barriers|Store1; StoreLoad; Load2|确保Store1数据对其他处理器变得可见（指刷新到内存），之前于Load2及所有后续装载指令的装载。StoreLoad Barriers会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令。|

　　StoreLoad Barriers是一个"全能型"的屏障，它同时具有其他三个屏障的效果。现代的多处理器大都支持该屏障（其他类型的屏障不一定被所有处理器支持）。执行该屏障开销会很昂贵，因为当前处理器通常要把写缓冲区中的数据全部刷新到内存中（Buffer Fully Flush）。


## 八、happens-before
　　从JDK 5开始，JMM就使用happens-before的概念来阐述多线程之间的内存可见性。happens-before原则非常重要，它是判断数据是否存在竞争、线程是否安全的主要依据，依靠这个原则，我们解决在并发环境下两操作之间是否可能存在冲突的所有问题。
<center>
如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在happens-before关系。
</center>

　　这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！happens-before仅仅要求前一个操作（执行的结果）对后一个操作可见。

　　下面是Java内存模型下一些"天然的"happens-before关系，这些happens-before关系无须任何同步器协助就已经存在，可以在编码中直接使用。如果两个操作之间的关系不在此列，并且无法从下列规则推导出来的话，它们就没有顺序性保障，虚拟机可以对它们进行随意地重排序。

### 1. 程序次序规则(Pragram Order Rule)
　　在一个线程内，按照程序代码顺序，书写在前面的操作先行发生于书写在后面的操作。准确地说应该是控制流顺序而不是程序代码顺序，因为要考虑分支、循环结构。

### 2. 监视器锁规则(Monitor Lock Rule)
　　一个unlock操作先行发生于后面对同一个锁的lock操作。这里必须强调的是同一个锁，而"后面"是指时间上的先后顺序。

### 3. volatile变量规则(Volatile Variable Rule)
　　对一个volatile变量的写操作先行发生于后面对这个变量的读取操作，这里的"后面"同样指时间上的先后顺序。

### 4. 线程启动规则(Thread Start Rule)
　　Thread对象的start()方法先行发生于此线程的每一个动作。

### 5. 线程结束规则(Thread Termination Rule)
　　线程中的所有操作都先行发生于对此线程的终止检测，我们可以通过Thread.join()方法结束，Thread.isAlive()的返回值等作段检测到线程已经终止执行。

### 6. 线程中断规则(Thread Interruption Rule)
　　对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过Thread.interrupted()方法检测是否有中断发生。

### 7. 对象终结规则(Finalizer Rule)
　　一个对象初始化完成(构造方法执行完成)先行发生于它的finalize()方法的开始。

### 8. 传递性(Transitivity)
　　如果操作A先行发生于操作B，操作B先行发生于操作C，那就可以得出操作A先行发生于操作C的结论。

## 最后
　　本文先从硬件的内存模型说起，引出Java内存模型，介绍了内存间的交互操作以及内存模型的特点，最后对`重排序`、`内存屏障`、`happens-before`进行了简单的解释说明。

## References
* [深入理解Java内存模型（一）——基础, 程晓明](http://www.infoq.com/cn/articles/java-memory-model-1)
* [【JVM】Java内存模型, 风动静泉](https://www.cnblogs.com/z00377750/p/9180923.html)
* [Java内存模型，朱小厮](http://www.importnew.com/20086.html)