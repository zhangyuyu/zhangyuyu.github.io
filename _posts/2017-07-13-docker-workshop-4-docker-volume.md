---
layout: post
title: "Docker Workshop（四）Docker数据存储"
date: 2017-07-13 08:18:24
categories: 
- devops
tags: 
- devops
- docker
---
## 一、前言

　　[上一篇 Docker的基本操作](http://zhangyuyu.github.io/docker-workshop-2-docker-brief/)按照构建流程练习了docker的一些常见基本指令。  
　　本篇主要讲述Docker数据管理相关的内容。

## 二、背景
　　该系列《Docker in Production》内容包含如下部分：

* [容器简介](http://zhangyuyu.github.io/docker-workshop-1-docker-container/)
* [Docker简介](http://zhangyuyu.github.io/docker-workshop-2-docker-brief/)
* [Docker的基本操作](http://zhangyuyu.github.io/docker-workshop-3-docker-operation/)
* **Docker数据存储**
* [Docker网络](http://zhangyuyu.github.io/docker-workshop-5-docker-network/)
* [Docker安全](http://zhangyuyu.github.io/docker-workshop-6-docker-security/)
* 多主机部署
* 服务发现
* 日志、跟踪、监控

本章主要通过练习`数据卷`和`数据卷容器`来理解Docker的数据管理。

## 三、Docker的存储方式
<!-- more -->
　　Docker存储驱动包含：AUFS、Device mapper、OverlayFS、Btrfs、ZF，它们提供了接口支持**镜像分层**与**写时复制机制Cow**，这两种技术满足了容器的核心价值，即极快的创建速度，极小的存储资源消耗以及容器迁移的便捷性。
容器的Root Image存储分为以下三类：
* AUFS，Overlay : 联合文件系统。
* DeviceMapper：CoW块存储。
* ZFS，btrfs: CoW文件系统。

　　Docker并不推荐采用Root Image的存储方式来存储应用数据。因为应用数据对安全，可用性，共享，性能等方面的要求和Root Image的要求是完全不一样的。
　　Docker采用了Volume这样一个独立的数据访问接口，应用通过Volume去访问相关的数据，Volume的实现和CoW的分层文件系统完全独立。
　　Volume通过Rancher Convoy或者Flocker这样的存储驱动去管理和访问具体的存储设备。

## 四、卷是什么
　　Docker的理念之一是将应用与其运行的环境进行打包，因此通常docker容器的生存周期与在容器中运行的程序的生存周期是一致的，当容器被销毁时，容器里的数据也会随之消失。

　　为了能够**保存（持久化）数据**以及**共享容器间的数据**，Docker提出了Volume的概念。
简单来说，Volume就是目录或者文件，它可以绕过默认的联合文件系统，而以正常的文件或者目录的形式存在于宿主机上。

## 五、管理数据的两种方式
在容器中管理数据主要有两种方式：

* 数据卷（Data volumes）
* 数据卷容器（Data volume containers）

### 1. 数据卷（Data volumes）
　　数据卷是容器内的一个特殊目录，该目录绕过UFS，不向顶层的可读写layer写入。数据卷用来保存、固化数据，独立于容器的生存周期，不会主动被回收。

数据卷特性：
* 处于UFS(Union File System)之外
* 主机文件系统中的普通目录
* 在卷上的I/O性能应与主机上的完全相同
* 卷的内容不包含在Docker镜像中
* 任何对卷内容的修改不是镜像的一部分
* 可被多个容器共享和重用
* 持久化数据（即使容器已被删除

>**注意**：数据卷的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会隐藏掉，能显示看的是挂载的数据卷。

可以使用以下两种方式创建：

* 在Dockerfile中指定VOLUME /some/dir
* 执行docker run -v /some/dir命令来指定
　　两种方式的区别在于`run的-v`可以指定挂载到宿主机的哪个目录，而`Dockerfile的VOLUME`不能，其挂载目录由docker随机生成。

#### 【练习1】用-v添加一个数据卷

1）启动一个基于busybox镜像的容器volume-example,并在其根目录下挂载一个data卷
```
$ docker run -d -it --name volume-example -v /data busybox
```
2）查看data是否挂载成功
```
$ docker exec volume-example ls | grep data
```

3) 查看data卷对应主机的目录；
```
$ docker inspect -f {{.Mounts}} volume-example
[{093c2cccfcc37e01d0ae4c145dcfb25a6c65a4b94617944efaa8f130463bcc6f /var/lib/docker/volumes/093c2cccfcc37e01d0ae4c145dcfb25a6c65a4b94617944efaa8f130463bcc6f/_data /data local  true}]
```

> 如果`步骤1）`，没有指定host的挂载目录，那么docker会自动创建一个挂载文件夹放在 
`/var/lib/docker/volumes/`下。上述例子中，在主机下的挂载目录是:
```
/var/lib/docker/volumes/093c2cccfcc37e01d0ae4c145dcfb25a6c65a4b94617944efaa8f130463bcc6f/_data
```

> 如果`步骤1）`，指定了主机目录：
```
$ docker run -d -it --name volume-example -v ~/volume/data:/data busybox
$ docker inspect -f {{.Mounts}} volume-example
[{ /home/vagrant/volume/data /data   true}]
```
那么在主机下挂载的目录则是自己指定的`~/volume/data`了。

4) 在主机对应的目录创建一个文件
```
$ sudo touch <paste the copied host directory location>/test-file
```

5) 在容器里面检查一下刚才创建的文件
```
$ docker exec volume-example ls data/
```

6) 查看所有volumn
```
$ docker volume ls
DRIVER              VOLUME NAME
local               bd0769d0b9a94e475b370d08ceafd9417499a8f3549a7b0f55f3be96f46b1b56
local               093c2cccfcc37e01d0ae4c145dcfb25a6c65a4b94617944efaa8f130463bcc6f
```

7） 查看volumn详细信息
```
$ docker volume inspect 093c2cccfcc37e01d0ae4c145dcfb25a6c65a4b94617944efaa8f130463bcc6f
[
    {
        "Name": "093c2cccfcc37e01d0ae4c145dcfb25a6c65a4b94617944efaa8f130463bcc6f",
        "Driver": "local",
        "Mountpoint": "/var/lib/docker/volumes/093c2cccfcc37e01d0ae4c145dcfb25a6c65a4b94617944efaa8f130463bcc6f/_data"
    }
]
```

#### 【练习2】用Dockerfile挂载数据卷
1） 创建一个`Dockerfile`
```yml
FROM busybox
RUN mkdir /test-dir
COPY test.yml /test-dir
VOLUME /test-dir
```
2) 创建一个`test.yml`文件
```
$ echo "This is a test file for testing volume from Dockerfile." > test.yml
```

3) 构建一个镜像
```
$ docker build -t volume-example:test .
Sending build context to Docker daemon 8.192 kB
Step 1 : FROM busybox
 ---> 103e96d345c0
Step 2 : RUN mkdir /test-dir
 ---> Running in 10aa732b4099
 ---> ef6b4c6bde95
Removing intermediate container 10aa732b4099
Step 3 : COPY test.yml /test-dir
 ---> 7fe861cb8aef
Removing intermediate container 50b4cec740b7
Step 4 : VOLUME /test-dir
 ---> Running in c20cfc350b8f
 ---> 8ea4d7630233
Removing intermediate container c20cfc350b8f
Successfully built 8ea4d7630233
```
4） 查看该镜像所包含的卷信息
```
$ docker inspect -f {{.Config.Volumes}} volume-example:test
map[/test-dir:{}]
```

5) Dockerfile中声明卷会占用镜像体积么？
```
$ docker history volume-example:test
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
8ea4d7630233        39 seconds ago      /bin/sh -c #(nop) VOLUME [/test-dir]            0 B
7fe861cb8aef        39 seconds ago      /bin/sh -c #(nop) COPY file:6d145c083d0383b1b   56 B
ef6b4c6bde95        39 seconds ago      /bin/sh -c mkdir /test-dir                      0 B
103e96d345c0        4 weeks ago         /bin/sh -c #(nop)  CMD ["sh"]                   0 B
f16f9e1c2f42        4 weeks ago         /bin/sh -c #(nop) ADD file:aa56bc8f2fea9c0c81   1.106 MB
```

6) 运行容器
```
$ docker run -d --name volume-example-container volume-example:test
c063baa2d2bd07373a563316a05a0c2793bca41883bc0964039dd96a8887cc3b

$ docker inspect -f {{.Mounts}} volume-example-container
[{054802484b2f99171b7d96be0b1e8028662d151b5491c80296d9b92e7d7d392b /var/lib/docker/volumes/054802484b2f99171b7d96be0b1e8028662d151b5491c80296d9b92e7d7d392b/_data /test-dir local  true}]

$ docker exec volume-example-container ls /test-dir
test.yml
```

7) 查看挂载的`test.yml`文件
```
$ sudo ls /var/lib/docker/volumes/054802484b2f99171b7d96be0b1e8028662d151b5491c80296d9b92e7d7d392b/_data
test.yml
```
>Dockerfile中每一句指令，都会生成一个临时的容器，如:
```
        Step 4 : VOLUME /test-dir
        ---> Running in c20cfc350b8f
        ---> 8ea4d7630233
```
>首先，`Step 4`里面生成了一个临时容器`c20cfc350b8f`;
>然后，commit容器得到了镜像`8ea4d7630233`;
>因此，`VOLUME /test-dir`是通过`是通过docker run -v  /test-dir`来实现的，随后由于容器的commit，该配置保存到了镜像`8ea4d7630233`里，可以通过如下质量查看
```
$ docker inspect -f {{.Config.Volumes}} 8ea4d7630233
map[/test-dir:{}]
```
>由于没有指定挂载到的宿主机目录，因此会默认挂载到宿主机的/var/lib/docker/volumes下的一个随机名称的目录下，因此Dockerfile里面的VOLUME不能指定主机挂载目录。

### 2. 数据卷容器（Data volume containers）
　　如果你有一些持续更新的数据需要在容器之间共享，最好创建数据卷容器。
数据卷容器，其实就是一个正常的容器，是一个挂载了数据卷但是不执行任何命令的容器，其目的只是为其他容器提供数据卷，方便数据在多容器之间共享、复用。

使用数据容器的两个注意点：

* 不要运行数据容器，这纯粹是在浪费资源。
* 不要为了数据容器而使用"最小的镜像"，如busybox或scratch，只使用数据库镜像本身就可以了。你已经拥有该镜像，所以并不需要占用额外的空间。

#### 【练习3】创建数据卷容器
练习2里面已经创建了一个挂载数据卷的镜像了，这里我们只需要用该镜像创建(create，不用run)一个数据卷容器
```
$ docker create --name volume-data-container volume-example:test
90cd6d51d364e6054c715f554c9a7921519ef4e4cddb2f5a54e1f86d8d202840
$ docker run -d --volumes-from volume-data-container --name volume-other volume-example:test
db57d89a10ee101df15e5fbbfa2b084526382a6ba5c15d133728011865f09a67
$ docker inspect -f {{.Mounts}} volume-data-container
[{2ce7145b0c5c286b13ef653349979a0f9f22048d6d80113b7edf98ceb61fc264 /var/lib/docker/volumes/2ce7145b0c5c286b13ef653349979a0f9f22048d6d80113b7edf98ceb61fc264/_data /test-dir local  true}]
$ docker inspect -f {{.Mounts}} volume-other
[{2ce7145b0c5c286b13ef653349979a0f9f22048d6d80113b7edf98ceb61fc264 /var/lib/docker/volumes/2ce7145b0c5c286b13ef653349979a0f9f22048d6d80113b7edf98ceb61fc264/_data /test-dir local  true}]
```
* 可以使用超过一个的 --volumes-from 参数来指定从多个容器挂载不同的数据卷。 
* 使用 --volumes-from 参数所挂载数据卷的容器自己并不需要保持在运行状态。
* 也可以从其他已经挂载了数据卷的容器来级联挂载数据卷。

#### 【练习4】备份数据
```
$  sudo docker run  --volumes-from volume-data-container -v ~/backup:/backup --name volume-backup volume-example:test tar cvf /backup/backup.tar /test-dir
test-dir/
test-dir/test.yml
```

* 首先利用`volume-example:test`镜像创建了一个叫做`volume-backup`的容器。
* 使用`--volumes-from volume-data-container`来让`volume-backup`容器挂载`volume-data-container`的数据卷。
* 使用-v参数挂载本地的`~/backup`目录到容器的`/backup`目录下。
* 容器启动之后，使用了`tar cvf /backup/backup.tar /test-dir`来将数据容器`/test-dir`下的内容备份为`/backup/backup.tar`(对应到宿主机下~/bakup.tar)。
```
$ ls ~/backup/
backup.tar
```

#### 【练习5】恢复数据
1）创建一个带有数据卷的容器
```
$ sudo docker run -v /test-dir --name volumn-data-container2 volume-example:test /bin/sh 
```

2）创建另外一个新的容器，挂载上面的数据卷容器，并使用untar解压备份文件到所挂载的容器卷
```
$ sudo docker run --volumes-from volumn-data-container2 -v ~/backup:/backup busybox tar xvf /backup/backup.tar
```

#### 【练习6】删除数据卷
如果你已经使用docker rm来删除你的容器，那可能有很多的孤立的Volume仍在占用着空间。

Volume只有在下列情况下才能被删除：
* 该容器是用docker rm －v命令来删除的（-v是必不可少的）。
* docker run中使用了--rm参数
* 手动去`/var/lib/docker/volumes/`删除

### 3. 数据卷和数据卷容器的比较
![](/assets/img/2017/docker-volumn-vs-container.png)

## 六、卷插件
数据卷  
优点：
* 跟主机磁盘性能一样
* 容器删除后依然保留

缺点：
* 仅限本地磁盘
* 不能随容器迁移

　　Docker推出了Volume plugin接口机制，让第三方的存储厂商来支持Docker Volume并且在此基础上进行功能拓展。

![](/assets/img/2017/docker-volumn-plugin.png)

* Rancher Convoy：Convoy是Rancher Labs用go开发的支持Device Mapper、NFS、EBS、GlusterFS多种后端存储的Docker Volume plugin driver. Convoy还提供了一个存储拓展功能（如快照、备份恢复等）的接口框架。

* Flocker：Flocker volume plugin driver主要用于多主机环境Docker数据卷的迁移，从而支持数据库应用等stateful有状态应用的主机间迁移。

## 最后
　　本篇文章主要是讲述了Docker的数据存储以及数据管理。  
[下一篇](http://zhangyuyu.github.io/docker-workshop-5-docker-network/)将讲述Docker的网络。

## References
* [Docker容器对存储的定义](http://dockone.io/article/1257)
* [Docker存储方式选型建议](http://dockone.io/article/1729)
* [Why Docker Data Containers (Volumes!) are Good](https://medium.com/@ramangupta/why-docker-data-containers-are-good-589b3c6c749e)
* [Data Volumes](https://rominirani.com/docker-tutorial-series-part-7-data-volumes-93073a1b5b72)
* [Understanding Volumes in Docker](http://container-solutions.com/understanding-volumes-docker/)
* [Docker Volumes vs Docker Volumes with Flocker](http://clusterhq.com/2015/12/09/difference-docker-volumes-flocker-volumes/)
* [Docker数据管理](http://feisky.xyz/docker/data_management/volume.html)
* [Docker：容器数据管理](http://www.ywnds.com/?p=7015)
