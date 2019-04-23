---
layout: post
title: "Gradle VS Maven"
date: 2015-08-30
categories: tool
tags: 
- gradle
- maven
---

## 一、背景
创世之初，世上只有Make一种构建工具，后来，其发展为GNU Make。但是，由于需求的不断涌现，码农的世界里逐渐演化出了千奇百怪的构建工具。
当前，JVM生态圈由三大构建工具所统治：
* Apache Ant带着Ivy
* Maven
* Gradle

## 二、历史

### 1. Ant
`Ant`("Another Neat Tool")，是由Apache软件件基金会启动的一个开源项目。
2000年发布，在很短时间内成为Java项目上最流行的构建工具。它基于过程式编程的idea。在最初的版本之后，逐渐具备了支持插件的功能。
* 使用XML描述构建的步骤
* 只负责构建步骤管理，如果要添加依赖管理的功能，还需要引入`Ivy`

### 2. Maven
Maven发布于2004年。目的是解决码农使用Ant所带来的一些问题。

* `convention over configuration`的思想，无需配置或者仅需少量配置即可开始构建
* 和`Ant`对比增加了依赖库管理
Maven仍旧使用XML作为编写构建配置的文件格式，但是，文件结构却有巨大的变化。Ant需要码农将执行task所需的全部命令都一一列出，
然而Maven依靠约定（convention）并提供现成的可调用的目标（goal）。不仅如此，有可能最重要的一个补充是，Maven具备从网络上
自动下载依赖的能力（Ant后来通过Ivy也具备了这个功能），这一点革命性地改变了我们开发软件的方式。
但是，Maven也有它的问题。依赖管理不能很好地处理相同库文件不同版本之间的冲突（Ivy在这方面更好一些）。XML作为配置文件的格式
有严格的结构层次和标准，定制化目标（goal）很困难。因为Maven主要聚焦于依赖管理，实际上用Maven很难写出复杂、定制化的构建脚本，
甚至不如Ant。用XML写的配置文件会变得越来越大，越来越笨重。在大型项目中，它经常什么“特别的”事还没干就有几百行代码。
Maven的主要优点是生命周期。只要项目基于一定之规，它的整个生命周期都能够轻松搞定，代价是牺牲了灵活性。

在对DSL（Domain Specific Languages)的热情持续高涨之时，通常的想法是设计一套能够解决特定领域问题的语言。在构建这方面，DSL的一个成功案例就是Gradle。

### 3. Gradle
Gradle结合了前两者的优点，在此基础之上做了很多改进。它具有Ant的强大和灵活，又有Maven的生命周期管理且易于使用。最终结果就是一个工具
在2012年华丽诞生并且很快地获得了广泛关注。例如，Google采用Gradle作为Android OS的默认构建工具。
Gradle不用XML，它使用基于Groovy的专门的DSL，从而使Gradle构建脚本变得比用Ant和Maven写的要简洁清晰。Gradle样板文件的代码很少，
这是因为它的DSL被设计用于解决特定的问题：贯穿软件的生命周期，从编译，到静态检查，到测试，直到打包和部署。它使用Apache Ivy来处理Jar包的依赖。
Gradle的成就可以概括为：约定好，灵活性也高。
* 使用 Groovy DSL 替代繁琐的 XML
* 支持增量构建
* 项目结构更加灵活

## 三、Maven和Gradle主要特性对比

Maven的主要功能主要分为5点，分别是依赖管理系统、多模块构建、一致的项目结构、一致的构建模型和插件机制。我们可以从这五个方面来分析一下
Gradle比起Maven的先进之处。

### 1. 依赖管理
在Java世界中，groupId、artifactId、version组成的Coordination（坐标）唯一标识一个依赖。任何基于Maven构建的项目自身也必须定义这三项属性，
生成的包可以是Jar包，也可以是war包或者ear包。一个典型的依赖引用如下所示：
```groovy
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.1.3.RELEASE</version>
</dependency>
```
换成gradle：
```groovy
dependencies {
    compile 'org.springframework.boot:spring-boot-starter:2.1.3.RELEASE'
}
```

比较起来，gradle相对于maven：
* 引用依赖方面变得非常简洁
* 对依赖项的scope有所不同
* Gradle支持动态的版本依赖
* 解决依赖冲突方面Gradle的实现机制更加明确


### 2. 多模块构建


### 3. 一致的项目结构


### 4. 一致的构建模型

### 5. 插件机制
Maven和Gradle设计时都采用了插件机制。但显然Gradle更胜一筹。主要原因在于Maven是基于XML进行配置。所以其配置语法太受限于XML。即使实现
很小的功能都需要设计一个插件，建立其与XML配置的关联。比如想在Maven中执行一条shell命令，其配置如下：



## 参考
* [Java Build Tools: Ant vs Maven vs Gradle](http://technologyconversations.com/2014/06/18/build-tools/)
* [Java构建工具：Ant vs Maven vs Gradle](https://blog.csdn.net/napolunyishi/article/details/39345995)
* [Gradle官网Gradle vs Maven Comparison](https://gradle.org/maven-vs-gradle/)
* [Maven和Gradle对比](http://www.huangbowen.net/blog/2016/02/23/gradle-vs-maven/)
