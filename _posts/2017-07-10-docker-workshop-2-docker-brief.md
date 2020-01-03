---
layout: post
title: "Docker Workshop（二）Docker简介"
date: 2017-07-10 08:59:59
categories: 
- devops
tags: 
- devops
- docker
---
## 一、前言

　　容器技术已经出现了相当一段时间了，例如LXC, BSD Jails, Solaris Zones... 那为什么出现了Docker，中间缺失了什么呢？  
　　[上一篇](http://zhangyuyu.github.io/docker-workshop-1-docker-container/)讲述了容器相关的知识，练习了LXC容器的相关指令，
本篇接着讲述Docker的出现以及Docker相关的理论基础。

## 二、背景
　　该系列《Docker in Production》内容包含如下部分：

* [容器简介](http://zhangyuyu.github.io/docker-workshop-1-docker-container/)
* **Docker简介**
* [Docker的基本操作](http://zhangyuyu.github.io/docker-workshop-3-docker-operation/)
* [Docker数据存储](http://zhangyuyu.github.io/docker-workshop-4-docker-volume/)
* [Docker网络](http://zhangyuyu.github.io/docker-workshop-5-docker-network/)
* [Docker安全](http://zhangyuyu.github.io/docker-workshop-6-docker-security/)
* 多主机部署
* 服务发现
* 日志、跟踪、监控

## 三、从Linux容器到Docker
<!-- more -->

>尽管有着光辉的历史，容器仍未得到广泛的认可。一个很重要的原因就是容器技术的复杂性：容器本身就比较复杂，不易安装 管理和自动化也很困难。
>而Docker就是为了改变这一切而生 ——《第一本Docker书》

　　[上一篇](http://zhangyuyu.github.io/docker-workshop-1-docker-container/)练习里，我们可以明显感受到，LXC存在的问题是
难以移动——难以通过标准化的模板制作、重建、复制和移动container。

　　在LXC的基础上，Docker进一步优化了容器的使用体验。Docker提供了各种容器管理工具（如：**分发、版本、移植**等）让用户无需关注
底层的操作，可以更简单的管理和使用容器。

　　早期的Docker代码实现是直接基于LXC的。自0.9版本开始，Docker开发了libcontainer项目，作为更广泛的容器驱动实现，从而替换掉
了LXC的实现。

>Docker官网上对于[Docker在单纯的LXC之上做了哪些事情](https://docs.docker.com/engine/faq/#what-open-source-license-are-you-using)的解释：
* Portable deployment across machines(跨主机可移植性部署)
* Application-centric(以应用为中心)
* Automatic build(自动构建)
* Versioning(版本)
* Component re-use(组件重用)
* Sharing(共享)
* Tool ecosystem(工具生态系统)


## 四、Docker是什么？

　　Docker是一个开源的引擎，可以轻松的为任何应用创建一个**轻量级的、可移植的、自给自足的**容器。

　　Docker可以让你将所有应用软件以及它的依赖打包成软件开发的标准化单元。

　　Docker容器将软件以及它运行安装所需的一切文件（代码、运行时、系统工具、系统库）打包到一起，这就保证了不管是在什么样的运行环境，
总是能以相同的方式运行。就好像 Java 虚拟机一样，"一次编写，到处运行（Write once, run anywhere）"，而 Docker 是"一次构建，
到处运行（Build once，run anywhere）"。

## 五、Docker与Microservices的关系

　　Microservices（微服务）依赖于"基础设施自动化"，而Docker正是"基础设施自动化"的利器。可以说Docker的火爆，一定程度上也带动
了微服务架构的兴起，而微服务的广泛应用也促进了Docker 繁荣，可以说两者相辅相成。

## 六、为什么使用Docker？

### 1. Why Docker?
　　按照当前官网[Why Docker?](https://docs.docker.com/engine/#why-docker)的解释，主要有四个方面：

#### 1.1更快速的交付应用（Faster delivery of your applications）
* 借助于Docker容器，程序开发员，系统管理员甚至QA和版本控制工程师都可以进行协同工作。Docker创建了一套标准的容器数据格式，
在这套标准数据格式基础上，当系统管理员管理操作容器时，程序开发员不需要关心容器的变化，这样就可以更专心的关注自己的应用代码。
而这种管理和开发的任务隔离，大大的简化了开发和部署的成本。
* Docker容器的创建非常容易，这样应用程序就可以进行快速迭代开发，从而缩短产品的上市周期。
* Docker的容器属于轻量级的容器，因此启动和停止都特别快。容器启动只需要毫秒级的时间，因此在进行开发、测试和部署各个环境之间
切换时几乎感受不到时间的流失。

#### 1.2 更轻松的部署和扩展（Deploy and scale more easily）
* Docker容器几乎可以在任何地方执行，至少在理论层面是可以再任意地方执行。Dokcer可以在桌面操作系统，物理服务器，虚拟机，
数据中心或者共有/私有云上面执行。
* 因为Docker容器可以在各种环境下运行，因此容器之间的迁移也非常方便。你可以非常方便的将容器从测试环境迁移到云环境中。

#### 1.3 更多的工作负载（Get higher density and run more workloads）
* Docker的容器本身不需要额外创建虚拟机管理系统，因此你可以启动多套Docker容器，这样就可以充分发挥主机服务器的物理资源，
也可以降低因为采购服务器licenses而带来的额外成本。

#### 1.4 更简单的管理（Faster deployment makes for easier management）
* Docker快速部署和轻量级的特性，可以使应用快速迭代。同时所有的修改都以增量的方式被分发和更新，从而实现自动化并且高效的管理。
    
### 2. Docker的使用场景

* 简化配置
* 代码流水线（Code Pipeline）管理
* 提高开发效率
* 隔离应用
* 整合服务器
* 调适能力
* 多租户环境
* 快速部署

## 七、Docker的组件

Docker主要包括两个大组件：

* The Docker Engine  
　　Docker Engine是基于虚拟化技术的一个轻量级的并且功能非常强大的开源容器管理工具。它可以将不同的work flow组合起来，
然后构建或者管理你的容器。

* Docker Hub  
　　一个分享和管理你所创建的image的SAAS（Software-as-a-Service）平台。

## 八、Docker的架构
　　Docker 采用的是 Client/Server 架构。客户端向服务器发送请求，服务器负责构建、运行和分发容器。客户端和服务器可以运行
在同一个 Host 上，客户端也可以通过 socket 或 REST API 与远程的服务器通信。

![](/assets/img/docker-architecture.svg)

　　Docker的核心组件：

* Docker 客户端 - Client
* Docker 服务器 - Docker daemon
* Docker 镜像 - Image
* Docker 容器 - Container
* Registry

### 1. Docker 客户端

　　最常用的 Docker 客户端是 docker 命令。通过 docker 我们可以方便地在 Host 上构建和运行容器。  
　　除了 docker 命令行工具，用户也可以通过 REST API 与服务器通信。

### 2. Docker 服务器
　　Docker daemon 是服务器组件，以 Linux 后台服务的方式运行，所谓"运行 docker"，指的就是运行 Docker daemon。  
　　Docker daemon 运行在 Docker host 上，负责创建、运行、监控容器，构建、存储镜像。

### 3.Docker镜像
　　可将 Docker 镜像看着只读模板，通过它可以创建 Docker 容器。  
镜像有多种生成方法：

* 可以从无到有开始创建镜像
* 也可以下载并使用别人创建好的现成的镜像
* 还可以在现有镜像上创建新的镜像

在[下一篇 Docker的基本操作](http://zhangyuyu.github.io/docker-workshop-3-docker-operation/)里会有对镜像的详细介绍。

### 4. Docker容器
　　Docker 容器就是 Docker 镜像的运行实例。用户可以通过 CLI（docker）或是 API 启动、停止、移动或删除容器。可以这么认为，
对于应用软件，镜像是软件生命周期的构建和打包阶段，而容器则是启动和运行阶段。  
　　总结起来，Docker容器就是：

* 一个镜像格式
* 一系列标准的操作
* 一个执行环境

### 5.Registry
　　Docker用Registry保存用户构建的镜像。    
　　Registry 分私有和公有两种：

* [Docker Hub](https://hub.docker.com/)是默认的 Registry，由 Docker公司维护，上面有数以万计的镜像，用户可以自由下载和使用。
* 出于对速度或安全的考虑，用户也可以创建自己的私有Registry

## 九、Docker核心技术

![](/assets/img/docker-core-tech.png){: .img-medium}

　　Docker核心是一个操作系统级虚拟化方法, 理解起来可能并不像VM那样直观。我们从虚拟化方法的四个方面：隔离性、可配额/可度量、
便携性、安全性来详细介绍Docker的技术细节。

![](/assets/img/docker-core-engine.png){: .img-medium}

### 1. 隔离性: namespace资源隔离
* pid：每个容器都运行在自己的进程环境中；
* user：在container内部用container内部的用户执行程序而非Host上的用户。
* net：容器间的虚拟网络接口和IP地址都是分开的；
* mnt：允许不同namespace的进程看到的文件结构不同；
* uts：UNIX Time-sharing System,使其在网络上可以被视作一个独立的节点而非Host上的一个进程；
* ipc：进程间交互；

　　例如：在主机和docker容器里，都可以拥有自己的init进程（PID=1），init进程是所有其他进程的祖先进程，docker容器其实是
主机里的一个进程。
![](/assets/img/docker-namespace-pid.png){: .img-medium}

### 2. 可配额/可度量：cgroups资源限制
　　`cgroups`是Linux内核提供的一种机制，用来限制、控制与分离一个进程组群的资源（如CPU、Memory、IO等）

主要提供了如下功能:

* 资源限制：限制任务使用的资源总额，并在超过这个配额时发出提示
* 优先级分配：分配CPU时间片数量及磁盘IO带宽大小、控制任务运行的优先级
* 资源统计：统计系统资源使用量，如CPU使用时长、内存用量等
* 任务控制：对任务执行挂起、恢复等操作

　　在实践中，系统管理员一般会利用cgroup做下面这些事（有点像为某个虚拟机分配资源似的）：

* 隔离一个进程集合（比如：nginx的所有进程），并限制他们所消费的资源，比如绑定CPU的核。
* 为这组进程分配其足够使用的内存
* 为这组进程分配相应的网络带宽和磁盘存储限制
* 限制访问某些设备（通过设置设备的白名单)

### 3. 便携性: 文件系统
　　最初docker仅能在支持Aufs文件系统的Linux发行版上运行，但是由于Aufs未能加入linux内核，为了寻求兼容性、扩展性，
Docker在内部通过graphdriver机制这种可扩展的方式来实现对不同文件系统的支持。目前，Docker支持Aufs，Devicemapper，Btrfs和Vfs
四种文件系统。

　　典型的Linux文件系统由bootfs和rootfs两部分组成，bootfs(boot file system)主要包含 bootloader和kernel，
bootloader主要是引导加载kernel，当kernel被加载到内存中后 bootfs就被umount了。 rootfs (root file system) 包含的就是典型Linux
系统中的/dev，/proc，/bin，/etc等标准目录和文件。
![](/assets/img/docker-aufs1.png){: .img-medium}

　　对于不同的linux发行版，bootfs基本是一致的，但rootfs会有差别，因此不同的发行版可以公用bootfs:
![](/assets/img/docker-aufs2.png){: .img-medium}

### 4. 安全性: AppArmor, SELinux, GRSEC
　　安全永远是相对的，这里有三个方面可以考虑Docker的安全特性:

* 由kernel namespaces和cgroups实现的Linux系统固有的安全标准;
* Docker Deamon的安全接口;
* Linux本身的安全加固解决方案,类如AppArmor, SELinux;

## 十、内核机制的练习

### 【练习1】命名空间的练习
```
# 创建一个Docker容器
$ docker run -it --rm busybox

# 列出所有的namespace
$ ls -l /proc/$$/ns
lrwxrwxrwx 1 vagrant vagrant 0 Jul 11 13:04 ipc -> ipc:[4026531839]
lrwxrwxrwx 1 vagrant vagrant 0 Jul 11 13:04 mnt -> mnt:[4026531840]
lrwxrwxrwx 1 vagrant vagrant 0 Jul 11 13:04 net -> net:[4026531957]
lrwxrwxrwx 1 vagrant vagrant 0 Jul 11 13:04 pid -> pid:[4026531836]
lrwxrwxrwx 1 vagrant vagrant 0 Jul 11 13:04 user -> user:[4026531837]
lrwxrwxrwx 1 vagrant vagrant 0 Jul 11 13:04 uts -> uts:[4026531838]

# 查看 pid namespace:
$ ps aux

# 查看 mnt namespace:
$ cat /proc/mounts

# 查看 uts namespace
$ hostname

# 查看 ipc namespace
$ ipcs

# 在宿主机里使用ipcmk -Q命令创建一个message queue
$ ipcmk -Q
Message queue id: 0

# 再次查看宿主机ipc namespace
$ ipcs

# 在容器里查看ipc namespace
$ ipcs
```

退出容器，按`ctrl + d`

### 【练习2】cgroup的练习
```
# 列出所有的subsystem
$ ls /sys/fs/cgroup/ -l
dr-xr-xr-x 4 root root  0 Jul 11 12:47 blkio
drwxr-xr-x 2 root root 60 Jul 11 12:47 cgmanager
dr-xr-xr-x 4 root root  0 Jul 11 12:47 cpu
dr-xr-xr-x 4 root root  0 Jul 11 12:47 cpuacct
dr-xr-xr-x 4 root root  0 Jul 11 12:47 cpuset
dr-xr-xr-x 4 root root  0 Jul 11 12:47 devices
dr-xr-xr-x 4 root root  0 Jul 11 12:47 freezer
dr-xr-xr-x 4 root root  0 Jul 11 12:47 hugetlb
dr-xr-xr-x 4 root root  0 Jul 11 12:47 memory
dr-xr-xr-x 4 root root  0 Jul 11 12:47 net_cls
dr-xr-xr-x 4 root root  0 Jul 11 12:47 net_prio
dr-xr-xr-x 4 root root  0 Jul 11 12:47 perf_event
dr-xr-xr-x 3 root root  0 Jul 11 12:47 systemd
```

　　默认情况下，Docker 启动一个容器后，会在`/sys/fs/cgroup`目录下的各个资源目录下生成以容器ID为名字的目录（group）:
```
$ ls /sys/fs/cgroup/cpu/docker/654bd6413abb005ccb2e93e096151650e022132b7186171fd871cc8d5692e356
cgroup.clone_children  cpu.cfs_period_us  cpu.shares  notify_on_release
cgroup.procs           cpu.cfs_quota_us   cpu.stat    tasks
$ cat /sys/fs/cgroup/cpu/docker/654bd6413abb005ccb2e93e096151650e022132b7186171fd871cc8d5692e356/cpu.cfs_quota_us
-1
```
　　此时 cpu.cfs_quota_us 的内容为-1，表示默认情况下并没有限制容器的CPU使用。在容器被 stopped 后，该目录被删除。

## 最后
　　本篇文章主要是涉及docker的基础理论知识，讲述了docker的出现，docker与LXC以及Microservice的关系，同时从what、why上理解了docker，
最后涉及了`cgroup`和`namespaces`两个Linux内核机制。

　　[下一篇](http://zhangyuyu.github.io/docker-workshop-3-docker-opearation/)将讲述Docker的基本操作。

## References
* [what-does-docker-technology-add-to-just-plain-lxc](https://docs.docker.com/engine/faq/#what-does-docker-technology-add-to-just-plain-lxc)
* [简述Docker](https://waylau.com/ahout-docker/)
* [什么是Docker](https://www.qcloud.com/community/article/597293)
* [八个Docker的真实应用场景](http://dockone.io/article/126)
* [Namespace isolation](https://www.toptal.com/linux/separation-anxiety-isolating-your-system-with-linux-namespaces)
* [Docker背后的内核知识——cgroups资源限制](http://www.infoq.com/cn/articles/docker-kernel-knowledge-cgroups-resource-isolation)
* [Docker资源配额](http://blog.opskumu.com/docker.html#16-资源配额cgroups)

