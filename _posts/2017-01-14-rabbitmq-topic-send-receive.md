---
layout: post
title: "基于RabbitMQ topic的消息发送与接收"
date: 2017-01-14 16:19:46
categories: java
tags: 
- rabbitmq
---
### 一、前言

#### 1.项目结构：
![](/assets/img/2017/rabbit-1.png){: .img-medium}
　　A是消息的生产者，B、C则是消息的consumer。A会通过queue（不关心其产品类型）发送消息给B、C。

#### 2.需求
　　近期项目上，新增的D应用程序需要监听原有A应用程序的消息。按照以往的做法，我需要做如下两件事：
* 向某个团队，给每该D应用的每个region（dev、intg、sys、prod等），申请新的queue A.D。（这中间可能需要两个星期时间）
* 改变应用A的代码，让其每次发消息时候还需要再给D发消息。
* 给应用A准备一次deploymen plan，并做相应的回归测试。
* 改变应用D的代码，让其从新建的queue里面接收处理消息。

#### 3.痛点
　　这一做法，主要有两大痛点：
* 每增加一个新应用，都需要申请queue
* 每增加一个新应用，都需要改动应用A的代码，还需要单独安排一次上线。生产者和消费者之间的耦合非常大。

#### 4.解决方案
　　因此，在兼容以前的基础上，我试图用下面的结构图解决上述痛点：
![](/assets/img/2017/rabbit-2.png){: .img-medium}
保持原有应用程序B和C的接收消息方式不变，新的应用程序开始改用topic的方法，这样可以兼容既有的应用程序，也可以将新应用程序的生产者和消费者解耦。但是还是存在一些问题：
* A的消息那么多，全部放在一个topic可能负载过大，那么应该采取分布式的方法？
* 如果消费者之一中途down了一段时间，该消息的等待时间又如何处理？是一直停留在topic中进行等待吗？

　　最后，考虑到上述因素以及实际项目的其他因素，我将结构变成了如下：
![](/assets/img/2017/rabbit-3.png){: .img-medium}

　　虽然把最开始的痛点1又引入了，但是解耦的好处还是非常有意义的。

　　本文旨在spike上述想法，证明该想法的可能性。

### 二、搭建外部环境
　　笔者采用vagrant + VirtualBox搭建一个含有rabbitmq server的虚拟机：
总体说来，所有的命令如下：

#### 1.创建虚拟机
```
$ mkdir rabbit
$ cd rabbit
$ vagrant box list
$ vagrant init yungsang/coreos
```
然后注意取消注释`config.vm.network "private_network", ip: "192.168.33.10"`
 
#### 2.登陆到虚拟机
```
vagrant ssh
```

#### 3.外部访问rabbitmq的管理页面
* URL：`http://192.168.33.10:15672/#/`  
* 用户名：`guest`  
* 密码：`guest`  

### 三、Spike
　　在编写本文时候，笔者顺便了解了一下Rabbit MQ的相关基础模式与用法，可以参见上一篇文章[RabbitMQ初探](http://zhangyuyu.github.io/rabbitmq/)

　　再回过头来看这个spike，发现其实本文要实现的就是[RabbitMQ初探](http://zhangyuyu.github.io/rabbitmq/)中的Topic Exchange模式。

　　其实说起来，就是JMS和AMQP的一个较大的区别：
- JMS有队列(Queues)和主题(Topics)两种形式，发送到JMS队列的消息最多只能被一个Client消费，发送到JMS主题的消息可能会被多个Clients消费；
- AMQP只有队列(Queues)，队列的消息只能被单个接受者消费，发送者并不直接把消息发送到队列中，而是发送到Exchange中，该Exchage会与一个或多个队列绑定，能够实现与JMS队列和主题同样的功能。

### 四、结语
　　本文只是针对其中的一点，进行了可行性的spike，但是实际应用中往往还涉及到很多复杂因素，比如技术上消息的事务处理和消息负载，另外还有项目进度、人员安排以及后续维护等问题。笔者不再过多阐述，谨以此文的小demo探究一些不一样的架构。

