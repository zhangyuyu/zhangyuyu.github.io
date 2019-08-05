---
layout: post
title: "Docker Workshop（一）容器简介"
date: 2017-07-09 18:25:04
categories: 
- devops
tags: 
- devops
- docker
---
### 一、前言
　　“不想在自己电脑上装一堆东西，恩，弄个docker容器，随便安装，随便破坏，随便试验的玩吧！”，这是笔者了解docker的初衷。  
　　后来回到武汉弄起了《Docker in Production》的workshop，正好项目也用到了docker，这才有股动力push我系统的了解一下docker。

### 二、背景
　　该系列《Docker in Prodcution》内容包含如下部分：

* **容器简介**
* [Docker简介](http://zhangyuyu.github.io/2017/07/10/Docker-workshop-2-Docker%E7%AE%80%E4%BB%8B/)
* [Docker的基本操作](http://zhangyuyu.github.io/2017/07/11/Docker-workshop-3-Docker%E7%9A%84%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C/)
* [Docker数据存储](http://zhangyuyu.github.io/2017/07/13/Docker-workshop-4-Docker%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8/)
* [Docker网络](http://zhangyuyu.github.io/2017/07/17/Docker-workshop-5-Docker%E7%BD%91%E7%BB%9C/)
* [Docker安全](http://zhangyuyu.github.io/2017/07/20/Docker-workshop-6-Docker%E5%AE%89%E5%85%A8/)
* 多主机部署
* 服务发现
* 日志、跟踪、监控

　　本篇主要讲述容器相关的知识，比较容器和虚拟机，练习LXC容器的相关指令，最后引出Docker。

### 三、容器的定义
<!-- more -->
　　容器是一种轻量级、可移植、自包含的软件打包技术，使应用程序可以在几乎任何地方以相同的方式运行。开发人员在自己笔记本上创建并测试好的容器，无需任何修改就能够在生产系统的虚拟机、物理服务器或公有云主机上运行。

![](/assets/img/container-structure.png)
　　容器是轻量级的操作系统级虚拟化，可以让我们在一个资源隔离的进程中运行应用及其依赖项。运行应用程序所必需的组件都将打包成一个镜像并可以复用。执行镜像时，它运行在一个隔离环境中，并且不会共享宿主机的内存、CPU 以及磁盘，这就保证了容器内进程不能监控容器外的任何进程。

### 四、容器解决的问题
　　为什么需要容器？容器到底解决的是什么问题？
　　一句话回答是：容器使软件具备了超强的可移植能力。

#### 1. 从运输行业的集装箱说起
　　20世纪60年代以前，运输行业几乎所有的货物都是以*散件*方式运输。

　　每一次运输，一方面货主与承运方会担心因货物类型的不同而导致损失，如几个铁桶压在了一堆香蕉上。另外一方面，运输过程中需要使用不同的交通工具严重浪费了大量的时间和精力，如货物先搬上车，到了码头由码头工人卸货，然后装上船，到岸之后卸下船，再装上火车，到达目的地，最后卸货……中间可能还有更多种类的交通工具，涉及到更多次的装、卸货、转移过程。

![](/assets/img/container-in-transportation1.png)

　　幸运的是，集装箱的发现解决了这个难题。

![](/assets/img/container-in-transportation2.png)

　　无论货物的体积、形状差异有多大，最终都被装载进集装箱里。由于要实现标准尺寸集装箱的运输，堆场、码头、起吊、船舶、汽车乃至公路桥梁、隧道等，都必须适应它在全球范围内的应用而逐渐加以标准化，形成影响国际贸易的全球物流系统。由此带来的是系统效率大幅度提升，运输费大幅度下降，地球上任何一个地方生产的产品都可以快速而低廉地运送到有需求的地方。

| 比较        | 散货运输    |  集装箱运输  |
| --------    | -----:   | :----: |
| 货物装卸时间  | 几天到一个星期，分拣、合并、装卸过程繁琐     |   几个小时  |
| 货物装卸时间占整个运输过程的比例     | >50%   |   <10%    |
| 所需劳动力    |    大量的码头工人   |   少数的码头工人和起重机操作员   | 
| 自动化程度    |    低             |   高  |
| 安全性       |   低，装卸时常有物品损毁和盗窃   |高，货物被隔离在封闭的集装箱内|
|成本          |货物越多，成本越高，耗时越长|前期固定成本投入较高，随着集装箱规模、运货量的增加，分摊到运送每个集装箱的成本极低，总成本几乎不变|

#### 2. 应用程序中的容器
　　同样，我们看看今天的软件开发面临的挑战。

　　以前几乎所有的应用都采用三层架构（Presentation/Appliation/Data），系统部署到有限的几台物理服务器上。如今，开发人员通常使用多种服务（如MQ、cache、DB）构建和组装应用，此外应用很可能部署到不同的环境（如虚拟服务器、私有云和公有云）。一方面应用包含多种服务，这些服务有自己所依赖的库和软件包；另一方面存在多种部署环境，服务在运行时可能需要动态迁移到不同的环境中。如何让每种服务能够在所有的部署环境中顺利运行？  
　　“服务”和“应用环境”对应到前面运输行业的“货物类型”和“运输工具”，容器则对应“集装箱”。  
![](/assets/img/container-in-application.png)

### 五、为什么容器技术如此诱人？

#### 1. 封装、隔离
　　各个容器之间，资源独立、隔离，不同的应用以container为单位进行部署，环境互不影响。
#### 2. 轻量、可移植
　　相比于传统的虚拟化技术，容器在cpu、memory、network IO上的性能损耗都有同样水平甚至更优，容器的创建、启动、销毁都很快。
#### 3. 标准化、run on any hardware
　　开发人员只需为按照一定的标准为应用创建一次运行环境，然后打包成容器便可在其他机器上运行。
#### 4. 一致性、可重复性
　　标准化的容器消除了开发、测试、生产环境的不一致性。

### 六、容器与虚拟机
　　谈到容器，就不得不将它与虚拟机进行对比，因为两者都是为应用提供封装和隔离。  
　　一句话解释区别：容器能做的事少得多并且使用起来相当廉价。而虚拟机提供整个虚拟化硬件层，可以做更多的事情但是使用成本显著。

![](/assets/img/container-vs-vms.png)

主要区别如下：

1. 容器实例与主机共享操作系统内核，通过内核提供的运行时隔离能力为服务提供独立的用户域、文件系统、网络以及进程运行空间。
虚拟机的每个实例自带操作系统，因而是一种硬件级的虚拟化隔离。
2. 容器通常是专用于运行特定服务的，它的镜像通常只包含运行该服务所需的上下文内容，许多广泛使用的镜像都只有几十MB，甚至几MB大小。
虚拟机则需要提供包括内核在内的通用进程运行环境，它的镜像偏向于大而完整的全功能集合，即使一个最小的精简镜像的体积也有几百MB。
3. 容器的使用方式倾向于开箱即用，镜像提供的是一个『不可变的基础设施环境』。
虚拟机则倾向于让用户根据所用的系统，自定义初始化操作，使用Ansible、Puppet这样的配置工具来进行基础设施的管理
4. 容器在启动速度和运行性能上更有优势，虚拟机在安全性上更有优势。

### 七、容器的分类
#### 1. 操作系统容器
　　操作系统层虚拟化是一种计算机虚拟化技术，这种技术将操作系统内核虚拟化，可以允许多个独立用户空间的存在，而不是只有一个。这些实例有时会被称为容器、虚拟引擎、虚拟专用服务器或是 jails（FreeBSD jail 或者 chroot jail）。从运行在容器中的程序角度来看，这些实例就如同真正的计算机。

　　容器技术如 LXC，OpenVZ，Linux VServer，BSD Jails 和 Solaris 区域就是操作系统容器。

#### 2. 应用容器
　　应用程序虚拟化是从其所执行的底层操作系统封装计算机程序的软件技术。一个完全虚拟化的应用，尽管仍像原来一样执行，但是并不会进行传统意义上的安装。应用在运行时的行为就像它直接与原始操作系统以及操作系统所管理的所有资源进行交互一样，但可以实现不同程度的隔离或者沙盒化。

　　容器技术如 Docker 和 Rocket 就是应用程序容器的示例。

### 八、LXC的练习
　　LXC是Linux containers的简称，是一种基于容器的操作系统层级的虚拟化技术。

　　LXC 与虚拟机的不同之处在于，它是一个操作系统级别的虚拟化环境，而不是硬件虚拟化环境。他们都做同样的事情，但 LXC 是操作系统级别的虚拟化环境，虚拟环境有它自己的进程和网络空间，而不是创建一个完整成熟的虚拟机。因此，一个 LXC 虚拟操作系统具有最小的资源需求，并启动只需几秒钟。

#### 【练习1】创建LXC容器
1. Ubuntu系统上安装好lxc
2. 宿主机上操作
```
# 列出所有的容器:
sudo lxc-ls --fancy
# 启动一个后台运行的容器demo-container:
sudo lxc-start --name demo-container --daemon
# 查看运行的容器的相关信息:
sudo lxc-info --name demo-container
# 进入到容器:
sudo lxc-console -n demo-container
# 登陆时候输入:
Username: ubuntu
Password: ubuntu
```
3. LXC容器中操作
```
# 查看容器的hostname
hostname
# 在容器中创建一个文件
echo "hello" > my_text.txt
cat my_text.txt
```
4. 退出容器
先按下`ctrl+a`，然后再按`q`。

#### 【练习2】克隆LXC容器
1. 宿主机上操作
```
# 冻结运行的容器
sudo lxc-freeze -n demo-container
# 列出所有的容器:
sudo lxc-ls --fancy
# 克隆容器之前，必须要先停止容器
sudo lxc-stop --name demo-container
sudo lxc-clone -o demo-container -n cloned-container
# 列出所有的容器:
sudo lxc-ls --fancy
# 启动复制的容器
sudo lxc-start --name cloned-container --daemon
# 进入到复制的容器:
sudo lxc-console -n cloned-container
# 登陆时候输入:
Username: ubuntu
Password: ubuntu
```
2. 克隆的容器中操作
```
# 查看克隆的容器是否也一起克隆了之前创建的文件
ls
```
3. 退出容器
先按下`ctrl+a`，然后再按`q`。
4. 停止克隆的容器
```
sudo lxc-stop --name cloned-container
```
5. 销毁克隆的容器
```
sudo lxc-destroy --name cloned-container
```

### 最后
　　本篇文章主要是涉及容器，从what, why, how上讲述了容器的相关知识。
至于容器的历史，有兴趣的读者可以自己查看一下相关资料。 
[容器简史：从20世纪70年代的chroot到2016的Docker](http://www.dockone.io/article/1522)以及[【容器那些事儿】容器技术的前世今生](http://www.alauda.cn/2016/01/18/container-history/)  
　　下一篇将讲述Docker的出现以及相关的基本理论知识。

### References
* [为什么容器技术将主宰世界？](http://dockone.io/article/803)
* [容器技术概览](http://dockone.io/article/2442)
* [容器What, Why, How - 每天5分钟玩转容器技术 - IBM](https://www.ibm.com/developerworks/community/blogs/132cfa78-44b0-4376-85d0-d3096cd30d3f/entry/%E5%AE%B9%E5%99%A8_What_Why_How_%E6%AF%8F%E5%A4%A95%E5%88%86%E9%92%9F%E7%8E%A9%E8%BD%AC%E5%AE%B9%E5%99%A8%E6%8A%80%E6%9C%AF_6?lang=zh)
* [LXC：Linux 容器工具](https://www.ibm.com/developerworks/cn/linux/l-lxc-containers/)
* [LXC基础学习教程](http://17173ops.com/2013/11/14/linux-lxc-install-guide.shtml#toc2)
