---
layout: post
title: "Servlet Life"
date: 2015-10-25 22:19:46
categories: java
tags: 
- java
- servlet
---
### 一、整体工作流程
>1、Client对Web Server发出请求<br>
2、Web Server接收到请求后，将其发送给Servlet容器<br>
3、Servlet容器产生Servlet实例对象并调用ServletAPI中相应的方法来对Client的请求进行处理，然后将处理的相应结果返回给WEB Server。<br>
4、Web server将从Servlet实例中收到响应并发送回客户端。<br>

![](/assets/img/client-server.png)

### 二、阶段流程
　　为了细化上述过程，可分为application的deploy过程和request的发送过程来讲：
#### 1、deploy阶段：
　　以Tomcat的容器为例，真正管理Servlet的容器是Context容器。一个Context对应一个Web工程。Context容器直接管理Servlet在容器中的包装类Wrapper。
　　deploy过程中，首先会生成context对象，config对象。（listener、filter的初始化过程也是在此阶段先后完成的）。
　　![](/assets/img/servlet-container.png)

#### 2、发送request阶段
(1）在默认情况下，Servlet实例是在第一个请求到来的时候创建，以后复用。如果有的Servlet需要复杂的操作需要载初始化时完成，比如打开文件、初始化网络连接等，可以通知服务器在启动的时候创建该Servlet的实例，配置`<load-on-startup>1</load-on-startup>`。
(2）init()
　　一旦Servlet实例被创建，Web服务器会自动调用init(ServletConfig config)方法来初始化该Servlet。其中方法参数config中包含了Servlet的配置信息，比如初始化参数，该对象在deploy阶段创建。
　　init()方法只执行一次。
(3)service()
　　 service()方法为Servlet的核心方法，Client的业务逻辑应该在该方法内执行，解析Client请求-〉执行业务逻辑-〉输出响应页面到Client。
(4）destroy()
　　当Web Server认为Servlet实例没有存在的必要了，比如应用重新装载，或服务器关闭，以及Servlet很长时间都没有被访问过。Web Server可以从内存中销毁该实例。Web Server必须保证在销毁Servlet实例之前调用该实例的destroy()方法，以便回收Servlet申请的资源或进行其它的重要的处理。
>当先后发送多个request时候，init()只执行一次。servlet容器处理多个请求通过产生多个线程，每个线程执行servlet的单实例的service()方法。

![](/assets/img/servletLife.png)

### 三、Filter
　　Filter不是一个servlet,它不能产生一个response,但是它能够在一个request到达servlet之前预处理request,也可以在离开servlet时处理response。
>Filter常用来：<br>
* 日志文件记录request参数<br>
* 身份验证或认证request的resource<br>
* 在request发送给Servlet之前格式化request的header或者body<br>
* 压缩发回给Client的reponse数据

　　一个filter有三个阶段init()、doFilter()、destroy()。
#### 1、init()
　　filter的init()是在上述的deploy阶段完成的。
#### 2、doFilter()　
　　一个Servlet初始化之后，每次request到达该Servlet的service()方法之前，都会先做doFilter()。能够配置一个filter到一个或多个servlet；单个servlet或servlet组能够被多个filter使用。
#### 3、destroy()
　　　当Servlet进行destroy之后，对应的Filter也会destroy。
>当有多个Filter时，按照web.xml中声明的顺序执行filter。

### 四、Listener
　　Listener是Servlet的监听器，它可以监听客户端的请求、服务端的操作等。如监听context、session、application的create,destroy,可以监听到context、session、application
属性绑定的变化。通过监听器，可以自动激发一些操作，比如监听在线的用户的数量。
　　以实现ServletContextListener接口监听context的create和destroy为例：
　　contextInitialized()是在上述deploy阶段Filter的init()之前执行的。
　　contextDestroyed()是在Servlet的destroy()及Filter的destroy()之后执行的。
　　
### 五、执行顺序
　　在[ServletLife](https://github.com/zhangyuyu/ServletLife)的工程下，执行命令`gradle tomcatRunWar`。根据下面控制台打印信息，可以看出listen->filter->servlet的顺序：

	*************Context created on*************
	***********This is the filter init***********
	Started Tomcat Server
	The Server is running at http://localhost:8080/ServletLife
	**********This is the servlet init***********
	*****************do filter*****************
	***********This is the servlet service***********
	***********This is the servlet destroy***********
	***********This is the filter destroy***********
	*************Context destroy*************

### 六、对应一个故事
><h5 align = "center">T公司CX项目开发经历</h5>
1、T公司接到一个CX项目<br>
　　根据项目计划书划分职能（BA、开发A功能人员、开发B功能人员……）<br>
　　确定BA具体人员<br>
2、需求R1<br>
　　准备安排具体人员A、B<br>
　　BA分析问题<br>
　　A、B开始认真干活<br>
3、需求R2<br>
　　BA分析问题<br>
　　A、B继续干活<br>
　　……<br>
4、项目完成了，开发人员解散<br>
　　BA也role off<br>
   
　　故事中，T对应一个Web container, CX对应一个context, BA对应Filter，A\B分别对应一个Servlet，需求R1\R2分别对应两次request。
　　第一个阶段对应deploy阶段：划分职能可以代表生成context、config对象；确定BA的具体人员则代表着Filter的初始化。
　　第二个阶段对应发送request阶段：安排A、B的巨头人员代表Servlet的初始化；BA分析问题代表着Filter的doFilter(）操作；认真干活则代表Servlet的service()操作。
　　第三个阶段是发送另一个request，此过程中没有安排具体的人员（Servlet的初始化），对应的是servlet的多线程操作。
　　第四个阶段对应destroy过程，开发人员解散代表Servlet的destroy，BA的role off则代表着Filter的destroy。
