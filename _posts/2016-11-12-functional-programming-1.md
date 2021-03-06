---
layout: post
title: "函数式编程初探（一）"
date: 2016-11-12 17:35:13
categories: java
tags: 
- java
- RxJava
---

### 一、前言
　　作为一名从业以来一直在编写Java的程序媛，虽然项目里经常用java8的lambda，可是在引入[RxJava](http://zhangyuyu.github.io/2016/11/01/RxJava/)之后，函数作为参数传递更加普遍，故阅读《函数式编程思维》一书，以求了解函数式编程。

### 二、定义
　　`函数式编程`(Functional Programming)是一种`编程范式`（programming paradigm），也就是如何编写程序的方法论。
主要思想是把运算过程尽量写成一系列嵌套的函数调用。

### 三、背景
　　函数式编程（Functional Programming）其实相对于计算机的历史而言是一个非常古老的概念，甚至早于第一台计算机的诞生。函数式编程的基础模型来源于 λ 演算，而 λ 演算并非设计于在计算机上执行，它是由 Alonzo Church 和 Stephen Cole Kleene 在 20 世纪三十年代引入的一套用于研究函数定义、函数应用和递归的形式系统。

### 四、为什么？
　　在《函数式编程思维》这本书中，多处指出`函数式编程思维`的好处之一,是能够将低层次细节的控制权移交给运行时,从而消弭了一大批注定会发生的程序错误。这使得程序员得以在更高的抽象层次上工作，同时运行时也有了执行复杂优化的自由空间。

　　在阮一峰的[《函数式编程初探》](http://www.ruanyifeng.com/blog/2012/04/functional_programming.html)一文中，
函数式编程的五大特点如下：
1. 函数是"第一等公民"
2. 只用"表达式"，不用"语句"
3. 没有"副作用"
4. 不修改状态
5. 引用透明
　　函数的运行不依赖于外部变量或"状态"，只依赖于输入的参数，任何时候只要参数相同，引用函数所得到的返回值总是相同的。

　　函数式编程的好处如下：
1. 代码简洁，开发快速
2. 接近自然语言，易于理解
3. 更方便的代码管理(每一个函数都可以被看做独立单元，很有利于进行单元测试以及模块化组合)
4. 易于"并发编程"
5. 代码的热升级（函数式编程没有副作用，只要保证接口不变，内部实现是外部无关的）

### 五、转变思维
　　先大致说明一下函数式编程与命令式编程的区别吧！

　　命令式编程：按照"程序是一系列改变状态的命令"来建模的一种编程风格。  
　　函数式编程将程序描述为表达式和变换，以数学方程的形式建立模型，并且尽量避免可变的状态。

　　《Working with Lagacy Code》的作者Michael Feathers的用以下一句话说明了函数式抽象和面向对象抽象的关键区别：
>面向对象编程通过封装不确定因素来使代码能被人理解；函数式编程通过尽量减少不确定因素来使代码能被人理解。——Michael Feathers

　　在面向对象的命令式编程语言里面，重用的单元是类和类之间沟通用的消息。所以OOP的世界提倡开发人员针对具体问题**建立专门的数据结构**，相关的专门操作以"方法"的形式附加在数据结构上。
函数式编程语言重用的思路很不一样，它提倡在**有限的几种关键数据结构**（如list、set、map）上运用针对这些数据结构高度优化过的操作，以此构成基本运转结构。

### 六、基本概念及使用介绍
#### Hello World
　　程序员往往都是讨厌看文档而偏爱看代码的，上述那么多文字，你不一定看的下去，还是首先来一个hello world吧。
对于计算表达式（1+2）* 3，

传统的编程方式是：
int a = 1 + 2
int b = a * 3

而对于函数式编程方式是：
int result = multiply(add(1,2), 3)
这里函数作为参数进行传递。

#### 函数式编程的常见函数
　　从一个例子说起，假设我们有一个名字列表，其中一些条目由单个字符构成。现在的任务是：
* 除去单字符条目
* 放在一个逗号分隔的字符串中进行返回
* 每个名字的首字母都是大写
代码可参考《函数式编程思维》的[github](https://github.com/oreillymedia/functional_thinking/blob/master/functional_thinking_examples/java/trans/TheCompanyProcess.java)
传统的java实现
```
    public String cleanNames(List<String> listOfNames) {
        StringBuilder result = new StringBuilder();
        for(int i = 0; i < listOfNames.size(); i++) {
            if (listOfNames.get(i).length() > 1) {
                result.append(capitalizeString(listOfNames.get(i))).append(",");
            }
        }
        return result.substring(0, result.length() - 1).toString();
    }
    public String capitalizeString(String s) {
        return s.substring(0, 1).toUpperCase() + s.substring(1, s.length());
    }
```
命令式编程鼓励程序员将操作安排在循环内部执行，本例中的三个操作filter、transform、convert都必须依赖于相同的低层次机制。

Java8的实现：
```
    public String cleanNames(List<String> names) {
        if (names == null) return "";
        return names
                .stream()
                .filter(name -> name.length() > 1)
                .map(name -> capitalize(name))
                .collect(Collectors.joining(","));
    }
    private String capitalize(String e) {
        return e.substring(0, 1).toUpperCase() + e.substring(1, e.length());
    }
```
　　两者的对比，可以发现后者这种更高层次的抽象思考有一些好处：
* 促使我们换一种角度去归类问题，看到问题的共性。
* 让运行时有更大的余地去做只能的优化（有时候调整作业的先后顺序会更有效率）。
* 让埋头实现细节的开发者看到原本视野之外的解决方案（比如，如果改用多线程，传统的方式可能需要自己手动穿插一些线程相关的代码）

　　可以从上述例子中，看到三个函数filter、map、fold/reduce的影子。
1. filter——筛选
根据用户定义的条件来筛选列表中的条目，并由此产生一个较小的新列表。

2. map——映射
对原集合的每一个元素执行给定的函数，从而变换成一个新的集合。

3. fold/reduce——折叠/化约
用一个累积量来"收集"集合元素。

　　总而言之，函数式编程以参数传递和函数的复合作为主要的表现手段，我们不需要掌握太多作为"不确定因素"存在的其他语言构造之间的交互规则。

　　本篇只讲述了函数式编程的概念、背景和三个具有普遍意义的基本构造单元，[下一篇](http://zhangyuyu.github.io/2016/11/12/函数式编程初探2)我会讲述一些柯里化与部分施用、缓存、缓求值、 函数式的数据结构。

