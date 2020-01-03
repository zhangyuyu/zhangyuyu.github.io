---
layout: post
title: "Docker Workshop（六）Docker安全"
date: 2017-07-20 08:49:15
categories: 
- devops
tags: 
- devops
- docker
---
## 一、前言
　　[上一篇 Docker网络](http://zhangyuyu.github.io/docker-workshop-5-docker-network/)主要讲述了网络模型、网络模式。  
　　本篇将讲述Docker安全性体现在哪些方面，并探讨每个方面的最佳实践。

## 二、背景
　　该系列《Docker in Production》内容包含如下部分：

* [容器简介](http://zhangyuyu.github.io/docker-workshop-1-docker-container/)
* [Docker简介](http://zhangyuyu.github.io/docker-workshop-2-docker-brief/)
* [Docker的基本操作](http://zhangyuyu.github.io/docker-workshop-3-docker-operation/)
* [Docker数据存储](http://zhangyuyu.github.io/docker-workshop-4-docker-volume/)
* [Docker网络](http://zhangyuyu.github.io/docker-workshop-5-docker-network/)
* **Docker安全**
* 多主机部署
* 服务发现
* 日志、跟踪、监控

## 三、Docker访问主机系统
　　很多时候，我们启动Docker容器时都以root用户权限在运行。那么，如果有ROOT权限可以做什么呢？当然可以做很多的事情，比如：访问所有信息、修改任何内容、关闭机器、结束进程以及安装各种软件等。
<!-- more -->

### 【练习】访问主机系统
1）主机上访问文件系统

```
$ cat /etc/shadow  # 使用普通用户访问
cat: /etc/shadow: Permission denied
$ who am i; groups;  # 查看当前的用户和组
vagrant  pts/0        2017-09-14 08:08 (10.0.2.2)
vagrant docker
$ sudo cat /etc/shadow  # 使用root用户访问
root:$6$HmunRCSU$YXNgfbnj2AQVtJS8DWFqb2ZxXIFGp3eRXMbXtuF9XUfgkAD1X8o2cQxBK6SHEH/K6D77PYMuv9p7rVtjAFfmh0:16744:0:99999:7:::
daemon:*:16744:0:99999:7:::
bin:*:16744:0:99999:7:::
sys:*:16744:0:99999:7:::
sync:*:16744:0:99999:7:::
games:*:16744:0:99999:7:::
man:*:16744:0:99999:7:::
...
```

2) Docker容器里访问文件系统

```
$ docker run -v /:/hostfs busybox cat /hostfs/etc/shadow
root:$6$HmunRCSU$YXNgfbnj2AQVtJS8DWFqb2ZxXIFGp3eRXMbXtuF9XUfgkAD1X8o2cQxBK6SHEH/K6D77PYMuv9p7rVtjAFfmh0:16744:0:99999:7:::
daemon:*:16744:0:99999:7:::
bin:*:16744:0:99999:7:::
sys:*:16744:0:99999:7:::
sync:*:16744:0:99999:7:::
games:*:16744:0:99999:7:::
man:*:16744:0:99999:7:::
...

```
　　可以看出只有root用户才能访问上面的文件。而Docker容器是用root权限运行的，所以在docker group下的用户间接地就有了root权限，也就可以访问到上述文件了。

3）Docker容器里修改主机文件系统

```
$ ls / | grep threat
$ docker run -it -v /:/hostfs busybox touch /hostfs/threat-on-the-way
$ ls / | grep threat
threat-on-the-way
```
　　可以看到，在Docker容器中创建的文件出现在了宿主机中，即在Docker容器中能够修改主机文件系统。

　　更严重的时，如果是 Privileged 容器，即在运行容器时指定--privileged=true参数，则能够允许容器所有设备执行任意操作，能够读写内核内存/proc/kcore，使用参数--net=host可以嗅探主机所有网络流量。

4）查看Docker容器和Metadata在宿主机上存储的地址

```
$ ll /var/lib/docker/
total 84
drwxr-xr-x   9 root root  4096 Sep 14 12:24 ./
drwxr-xr-x  52 root root  4096 Nov 22  2015 ../
drwxr-xr-x   5 root root  4096 Nov 22  2015 aufs/
drwx------   8 root root  4096 Sep 14 12:24 containers/
drwx------ 259 root root 36864 Sep 14 08:10 graph/
-rw-r--r--   1 root root  5120 Sep 14 12:24 linkgraph.db
drwxr-x---   3 root root  4096 Nov 22  2015 network/
-rw-------   1 root root  2116 Sep 14 08:14 repositories-aufs
drwx------   2 root root  4096 Sep 14 08:10 tmp/
drwx------   2 root root  4096 Nov 22  2015 trust/
drwx------   3 root root  4096 Sep 14 08:08 volumes/
```

　　运行态容器默认都是使用/var/lib/docker目录，容器内部写日志、产生运行时数据等都会影响该目录，并且产生的文件越来越多，占用空间越来越大，因此需要定期清理无用的镜像和容器。可以用Logical Volume Manager (LVM)为Docker挂载点/var/lib/docker创建单独的分区，最好是SSD盘。

## 四、Docker的安全性

　　Docker的安全性主要体现在如下几个方面：
* [Docker容器的安全性](4-1.Docker容器的安全性)
* [镜像的安全性](4-2.镜像的安全性)
* [Docker daemon的安全性](4-3.Docker daemon的安全性)

### 4-1.Docker容器的安全性
　　指容器是否会危害到宿主机或其他容器。  
　　容器的安全性问题的根源在于容器和宿主机共用内核，因此受攻击的面特别大，另外，如果容器里的应用导致Linux内核崩溃，那么毫无疑问，整个系统哥都会崩溃。这一点与虚拟机是不同的，虚拟机与宿主机的接口非常有限，而且虚拟机崩溃一般不会导致宿主机崩溃。

### 4-2.镜像的安全性
　　用户如何确保下载下来的镜像是可信的、未被篡改过的；

### 4-3.Docker daemon的安全性
　　如何确保发送给daemon的命令是由可信用户发起的。用户通过CLI或者REST API向daemon发送命令已完成对容器的各种操作，例如通过docker exec命令删除容器里的数据，因此需要保证client与daemon的连接时可信的。

## 五、Docker安全最佳实践

### 主机
* 保持内核及时更新，防止黑客利用未修复的漏洞进行攻击
* 增强主机安全保护，如果主机不安全了，容器也就谈不上安全了
* 保持Docker及时更新，特别要关注Docker安全相关方面的更新

### 镜像
* 在Dockerfile中为容器创建一个非root用户
* 以非root用户运行容器进程，最大程度控制用户的权限范围
* 只使用受信的基础镜像，可由最小基础镜像开始(Busybox, Alpine)，最好是建立本地仓库镜像
* 仅安装必要的包，因为可能有些包会有漏洞，一定程度上降低风险
* 重新构建镜像时需要包含安全补丁，防止黑客利用漏洞实施攻击

### 守护进程
* 只允许受信用户控制Docker守护进程，保证与Daemon的连接是可信的
* 不使用不受信的镜像仓库，因为镜像可能会被篡改过
* 必要时请为Docker守护进程应用 TLS 认证网络
* 限制容器之间的网络通信，如果两个容器之间没有通信的必要就限制其网络通信功能

### 容器运行时
* 不要到产品环境中使用任何开发者工具(boot2docker, kinematic)
* 限制容器使用Linux内核能力和资源使用，为守护进程设置受限的控制资源权限(–ulimit)
* 不要使用 Privileged 容器，如果使用--privileged参数将授予容器与主机几乎相同的权限
* 指定容器重启策略为on-failure，重启策略(Restart policies)有四种：`no`、`on-failure`、`always`、`unless-stopped`。
* 使用强制权限控制系统 AppArmor 和 Linux安全增强工具 SELinux 保证额外的安全层

　　更多详情，可以参考[Security Benchmarks](https://learn.cisecurity.org/benchmarks)制定的容器的安全基准。下面截取了其中的一部分目录：
![](/assets/img/docker-security-host.png)
<center>主机</center>

![](/assets/img/docker-security-images.png)
<center>镜像</center>

![](/assets/img/docker-security-daemon.png)
<center>守护进程</center>

![](/assets/img/docker-security-container-runtime.png)
<center>容器运行时</center>

### 六、构建容器平台时候的安全问题
　　上面说到的安全问题基本上是容器自身的，就是单独去考虑一个容器时，容器自身的一个安全性的问题。那么，当我们构建一个云平台的时候，这时容器是大量的，也可能是多租户的，特别是公有云，用户上传上来的应用是否安全，在这种情况下，怎么来去考虑一个容器平台的安全问题。可以从四个方面：

* 基础架构层安全
* 容器调度层安全
* 容器调度层安全
* 容器调度层安全

　　我们可以采取的一些措施：
* 建立容器云平台的安全基线
* 容器 CI/CD 过程加密验证
* 加强平台的权限访问控制或者 API 密钥管理
* 加强容器的安全测试及渗透测试
* 加强安全漏洞扫

## 最后
　　本篇文章主要列举了一些Docker安全的最佳实践，并没有做过多的解释，具体情况还需要结合具体的项目实践。

## References
* [浅谈Docker隔离性和安全性](http://www.freebuf.com/articles/system/69809.html)
* [Docker安全](https://segmentfault.com/a/1190000005794220)
* [绝不避谈 Docker 安全](https://mp.weixin.qq.com/s/IN_JJhg_oG7ILVjNj-UexA)
* [如何打造安全的容器云平台](http://securityer.lofter.com/post/1d0f3ee7_d4e69b1)
* [从自身漏洞与架构缺陷，谈Docker安全建设](http://www.10tiao.com/html/167/201707/2650762256/2.html)
