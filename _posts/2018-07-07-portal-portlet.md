---
layout: post
title: "Portal 和 Portlet"
date: 2018-07-07 09:32:04
categories: java
tags: 
- portal
- germany
---

## 一、前言
　　本文是初识Portal 和Portlet，并对相关的概念进行扫盲，主要包括：

* [Portal](#Portal)
* [Portlet](#Portlet)
* [Portlet容器](#Portlet容器)
* [JSR168](#JSR168)
* [认识 Portal 和 Portlet](#认识Portal和Portlet)
* [Portlet和Widget](#Widget)、[Servlet的对比](#Servlet)
* [Portlet的生命周期](#Portlet的生命周期)

<!-- more -->

## 二、背景
　　最近加入敏捷咨询的团队，客户是一个保险公司，我们的任务是对他们现有的技术架构进行分析，了解他们的痛点，对应做出敏捷转型的方案。
　　了解到他们目前是采用 IBM 的 Websphere服务器，采用Portal的架构模式。各个 portlet 之间相互调用，相互依赖，并不是完全的独立。因此他们的主要痛点在于，每次部署都需要很长时间的准备，此外，很难进行集成测试。当然，还有其他一些问题，但是经过小组讨论之后，进行敏捷转型，第一步是要帮助其构建持续集成、持续交付。
　　在此之前，笔者对Portal 这一套门户系统，并不了解，因此特意先做点准备工作。

## 三、一些概念
　　先简单解释下`Portal`、`Portlet`、`Portlet 容器`和`JSR168`，然后再详细展开：

![](/assets/img/portal-portlet-architecture.png)

　　`Portal`是一种web应用，`Portlet`是一种Web组件。通俗来说，`Portlet`就是一个`Portal`上的子窗口。`Portlet容器`是`Portlet`的运行时环境。`JSR168`是规范，为创建Portlet建立标准的API。

### <span id="Portal">1. 什么是Portal？</span>

　　Portal，英文直译是“大门、入口”，这里指“门户”，现多用于互联网的门户网站和企业应用系统的门户系统。

　　狭义上，门户网站，指通向某类综合性互联网信息资源并提供有关信息服务的应用系统。门户网站最初提供搜索引擎、目录服务，后来由于市场竞争日益激烈，门户网站不得不快速地拓展各种新的业务类型，希望通过门类众多的业务吸引和留驻互联网用户。在全球范围中，最为著名的门户网站则是谷歌以及雅虎，而在中国，最著名的门户网站有中国四大门户网站（新浪、网易、搜狐、腾讯），其他也有百度、新华网、人民网、凤凰网等也较为著名。

　　广义上，这里是一个Web应用框架，它将各种应用系统、数据资源和互联网资源集成到一个信息管理平台之上，并以统一的用户界面提供给用户，并建立企业对客户、企业对内部员工和企业对企业的信息通道，使企业能够释放存储在企业内部和外部的各种信息。

　　本文的Portal主要指广义上的概念，更多解释，可以查看[门户网站的百度百科](https://baike.baidu.com/item/%E9%97%A8%E6%88%B7%E7%BD%91%E7%AB%99#2)

### <span id="Portlet">2. 什么是Portlet？</span>

　　Portlet，门户组件，它是基于Java的Web组件，由Portlet容器管理，并由容器处理请求，生产动态内容。Portals使用Portlets作为可插拔用户接口组件，提供信息系统的表示层。Portlets实现了Web应用的模块化和用户中心化。

　　从用户的角度来看，`Portlet`是门户网站站点中提供特定服务或信息（例如，新闻、广告）的窗口；
从应用程序开发者的角度来看，Portlet 是可插入的模块，它们被设计成在门户网站服务器的`Portlet 容器`中运行。

　　更多解释，可以查看[Portlet的百度百科](https://baike.baidu.com/item/Portlet/1069487?fr=aladdin)

### <span id="Portlet容器">3. 什么是Portlet容器?</span>

　　Portlet容器提供Portlet需求的运行时环境并运行Portlet。它包含Portlets并控制它们的生命周期。容器提供Portlet参数的持久存储机制，它接受来自Portal的request，并在其持有的Portlet上执行request。容器不负责Portlet产生内容的聚合，Portal自己处理内容聚合。

### <span id="JSR168">4. 什么是JSR168？</span>

　　JSR168是Java规范要求（Java Specification Request，JSR）的缩写，开发符合JSR168规范的Portlet将可以顺利移植到符合该规范的不同Portal平台上！

　　随着企业级Portal的大量涌现，不同提供商创建了不同的Portal组件API，即Portlet。不同的不兼容给应用服务商，Portal用户和Portal Server提供商都造成了问题。为了消除这些问题，JSR168，即Portlet规范提出，从而提供Portlet和Portal间的互操作性。

![](/assets/img/portal-portlet-protocol.png) 

　　JSR168的目标:

* 定义了portlet运行环境 - portlet容器 
* 定义了portlet容器和portlet之间的API 
* 提供了portlet存储持久性和非持久性数据的机制 
* 提供了portlet包含servlet和JSP的机制 
* 定义了portlet打包，方便部署 
* 保证了portlet在JSR 168门户中的二进制移植 
* 能够以WSRP（Web Service for Remote Portlet）协议把JSR 168 portlet作为远程portlet运行。

## <span id="认识Portal和Portlet">三、认识 Portal 和 Portlet</span>

　　`Portlet` 在 `Portlet 容器`中运行，容器接收`Porlet`生成的内容，并传递给`Portal`。由 `Portal`服务器组织成`Portal 页面`，并送交客户端设备（如浏览器）显示。

### 1. Portal页面

　　`Portal`可以被视为一系列具有不同区域的网页。这些区域包含不同的窗口，每个窗口包含一个`Portlet`：
![](/assets/img/portal-portlet-page.png)

　　Portlet 生成片段（Fragment），而 Portal 通常要添加上标题（Title）、控制按钮和其他装饰性元素（Decorations and Controls），而得到 Portlet 窗口（Portlet Window）。最后 Portal 将多个 Portlet 窗口聚合而成一个完整的文档，即 Portal 页面（Portal Page）。

### 2. Portlet Rendering Modes

　　`Portlet`有不同的视图模式，规范定义了3种模式：

* VIEW（查看）: 生成反映Portlet当前状态的标记。
* EDIT（编辑）: 应该允许用户自定义Portlet的行为。
* HELP（帮助）: 应该向用户提供有关如何使用Portlet的一些信息。

### 3. Window States（窗口状态）

　　窗口状态是指示Portlet在任何给定页面上应占用多少页面空间的指示器。规范定义了3种状态：

* NORMAL（正常）
* MINIMIZED（最小化）
* MAXIMIZED（最大化）

## 四、对比

### <span id="Widget">1. `Portlet` 和 `Widget`</span>

　　`Widget`是嵌入在较大网页中的网页，通常使用iFrame——内容来自单独的HTTP连接，并具有自己的CSS样式表，cookie等。最终合成发生在用户的浏览器中。
　　`Portlet`是软件模块，用来生成组合HTML页面的标记片段（Markup Fragment），这些Fragment共享通用的CSS样式表，cookie等。最终组合在`Portlet Server`上进行，然后页面再被传递到客户端浏览器。

　　`Portlet`和`Widget`都提供了UI组件模型，因此具有很多相似之处:

* 它们都提供用户和后端服务的交互;
* 都可以将信息和上下文传递给其他的`Portlet`或者`Widget`;
* 最终用户和管理员可以将`Portlet`或`Widget`放到页面上并在页面上重新排列它们。

　　上述可以看出`Portlet` 和 `Widget`最终合成页面的地方不一样，因此主要不同之处也是围绕这个展开：

* `Portlet`是服务端组件模型，`Widget`是客户端组件模型;
* `Widget`的源码是已经下载了的，所以在浏览器中是可见的;
* `Widget`比标准的`Portlet`更具响应式，因为`Widget` 对服务器的调用是独立的浏览器调用。但是，也许有时候呈现页面的某一个部分并不是期望行为。
* `Widget`通常直接与信息系统交互，来获取它们需要的内容，`Portal`则是聚合很多`Portlet`生成的内容

![](/assets/img/portal-portlet-widget.png)

### <span id="Servlet">2. `Portlet` 和 `Servlet`</span>

![](/assets/img/portal-portlet-servlet.png)

　　相似之处：
* 都是基于Java技术的web组件
* 都是被专门的容器管理，`Portlet`被`portlet 容器`管理，`Servlet`被`Servlet容器`管理
* 都可以使用 客户端/服务器端 模式
* 都是与web客户端通过request/response方式交互

　　不同之处：
* `Servlet`生成整个web页面，而`Portlet`只生成内容片断，而`Portal`来负责将这些片断组装到同一个页面
* `Servlet`可以被映射为url，但是`Portlet`不可以被直接映射为url
* Web客户端可以直接同`Servlet`交互，但是如果Web客户端要和`Portlet`交互需要通过`Portal` 系统
* `Portlet`定义了`Portlet Mode`和`Window State`
* `Portlet`可以持久化存储和访问数据
* `Portlet`可以在两种范围上存/取数据到session里: Portlet私有域和application域上
* `Portlet`的response都无法设置字符编码，而Servlet 可以。

## <span id="Portlet的生命周期">五、Portlet的生命周期</span>

　　`Portlet API`包含一个`GenericPortlet`类，它实现`Portlet`，`EventPortlet`和`ResourceServingPortlet`接口并提供默认功能。开发人员通常应直接或间接扩展`GenericPortlet`类以实现其portlet。
![](/assets/img/portal-portlet-interface-diagram.png)

　　基本的`Portlet接口`，生命周期包括下面四个方法：

* init
* processAction
* render
* destroy

　　使用`EventPortlet接口`时，额外的生命周期操作：

* processEvent，当事件被触发的时候，该方法会被调用

　　使用`ResourceServingPortlet`时，额外的生命周期操作：

* serveResource，由portlet容器调用，以允许portlet根据其当前状态生成资源内容。

## 最后
　　下图是基于IBM WebSphere Portal的企业门户示例，选择门户平台产品将帮助企业更好的建立各种企业级的集成标准。
![](/assets/img/portal-portlet-ibm.png)

　　本文只是对Portal门户系统的初步了解，至于它的优势劣势、以及它的具体使用场景、使用的注意事项，甚至portlet的开发，不在此做过多描述。要有实际使用过之后，才有更深刻的体会。

## References
* [IBM门户部署与管理简明教程](https://www.ibm.com/developerworks/community/files/basic/anonymous/api/library/c144a765-c982-4ab5-93ff-cfdcb9140298/document/1ae8a0f1-e5e8-4430-9b55-73c1f6e72e91/media)
* [Making Sense of Portlets and Widgets - IBM WebSphere Portal Blog](https://www.ibm.com/developerworks/mydeveloperworks/blogs/WebSpherePortal/entry/making_sense_of_portlets_and_widgets1?lang=en)
* [Portlet and Servlet](http://jsr286tutorial.blogspot.com/p/portlet-and-servlet.html)
* [Portlet Development Resources - Red Hat](https://access.redhat.com/documentation/en-US/Red_Hat_JBoss_Portal/6.2/html/Development_Guide/chap-Portlet_Development_Resources.html)
* [Jboss Portal Quickstarts](https://github.com/jboss-developer/jboss-portal-quickstarts)
