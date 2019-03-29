---
layout: post
title: "Docker Workshop（三）Docker的基本操作"
date: 2017-07-11 22:08:27
categories: 
- devops
tags: 
- devops
- docker
---
### 一、前言

　　[上一篇 Docker简介](http://zhangyuyu.github.io/2017/07/10/Docker-workshop-2-Docker%E7%AE%80%E4%BB%8B/)讲述了Docker相关的理论知识，了解了Docker的场景及优势，练习了内核的namespace以及cgroup。
　　本篇将开始实际动手操作，熟悉docker基本的指令。

### 二、背景
　　该系列《Docker in Prodcution》内容包含如下部分：

* [容器简介](http://zhangyuyu.github.io/2017/07/09/Docker-workshop-1-%E5%AE%B9%E5%99%A8%E7%AE%80%E4%BB%8B/)
* [Docker简介](http://zhangyuyu.github.io/2017/07/10/Docker-workshop-2-Docker%E7%AE%80%E4%BB%8B/)
* **Docker的基本操作**
* [Docker数据存储](http://zhangyuyu.github.io/2017/07/13/Docker-workshop-4-Docker%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8/)
* [Docker网络](http://zhangyuyu.github.io/2017/07/17/Docker-workshop-5-Docker%E7%BD%91%E7%BB%9C/)
* [Docker安全](http://zhangyuyu.github.io/2017/07/20/Docker-workshop-6-Docker%E5%AE%89%E5%85%A8/)
* 多主机部署
* 服务发现
* 日志、跟踪、监控

本章通过以下系列过程，来熟悉Docker的基本指令：
* 构建镜像；
* 搭建私有registry；
* 上传镜像、获取镜像；
* 创建容器、运行容器；
* 连接容器
* 使用Docker-compose工具

### 三、Docker镜像
<!-- more -->

#### 1. 镜像是什么
　　Docker镜像是一个特殊的**文件系统**，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

镜像通常包含：

* 一个轻量级的操作系统发行版
* 相关依赖
* 单个应用或服务

#### 2. 镜像的分层存储
　　传统的Linux加载bootfs时会先将rootfs设为read-only，然后在系统自检之后将rootfs从read-only改为read-write，然后我们就可以在rootfs上进行写和读的操作了。
　　但Docker的镜像却不是这样，它在bootfs自检完毕之后并不会把rootfs的read-only改为read-write。而是利用union mount（UnionFS的一种挂载机制）将一个或多个read-only的rootfs加载到之前的read-only的rootfs层之上。

![](/assets/img/docker-image-layer1.png)

可以通过`docker info`查看宿主机上docker的文件系统方式：

    $ docker info
    Containers: 3
    Images: 267
    Server Version: 1.9.1
    Storage Driver: aufs
     Root Dir: /var/lib/docker/aufs
     Backing Filesystem: extfs
     Dirs: 273
     Dirperm1 Supported: true
    Execution Driver: native-0.2
    Logging Driver: json-file
    Kernel Version: 3.19.0-32-generic
    Operating System: Ubuntu 14.04.3 LTS
    CPUs: 2
    Total Memory: 3.431 GiB
    Name: workshop
    id=: 27OE:SE73:WCPT:366X:CCGS:G2NQ:RB2A:UVJ2:A3OC:CZUJ:RYCB:XDDE
    WARNING: No swap limit support

#### 3. 练习构建镜像
　　[上一篇](http://zhangyuyu.github.io/2017/07/10/Docker-workshop-2-Docker%E7%AE%80%E4%BB%8B/)讲到获取镜像有三种方式，其中自己从无到有地创建镜像，Docker有两种方式：

* 创建一个容器，运行若干命令，再使用`docker commit`来生成一个新的镜像。以这种方式创建的镜像不具备再生产能力且无法实现版本控制性，因此绝对**不值得提倡**。
* 创建一个`Dockerfile`然后再使用`docker build`来创建一个镜像。大多人会使用`Dockerfile`来创建镜像。

#### 【练习1】通过Dockerfile构建镜像
1）创建一个Dockerfile
```yml
FROM debian:8
MAINTAINER John Citizen
RUN apt-get update && \
apt-get install -y nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
![](/assets/img/docker-image-layers-history.png)

2）生成镜像
` docker build -t nginx-demo .`其中`.`是当前目录，包含`Dockerfile`文件


    $ docker build -t nginx-demo .
    Sending build context to Docker daemon 7.168 kB
    Step 1 : FROM debian:8
    8: Pulling from library/debian
    aaec12cbddb4: Pull complete
    a4231e14d761: Pull complete
    Digest: sha256:64682f1a6d256b358b10dba3669b22f3594c69ec790548f5ba6362276ac9d4ca
    Status: Downloaded newer image for debian:8
     ---> a4231e14d761
    Step 2 : MAINTAINER John Citizen
     ---> Running in 35d7b813f3ec
     ---> 3abd0b877321
    Removing intermediate container 35d7b813f3ec
    Step 3 : RUN apt-get update && apt-get install -y nginx
     ---> Running in f75d9c834ee9
     ---> 352a72a3b8a8
    Removing intermediate container f75d9c834ee9
    Step 4 : EXPOSE 80
     ---> Running in c9939361ce4d
     ---> e55ea9a20dd4
    Removing intermediate container c9939361ce4d
    Step 5 : CMD nginx -g daemon off;
     ---> Running in bdac893db4aa
     ---> 7bd0c6e98644
    Removing intermediate container bdac893db4aa
    Successfully built 7bd0c6e98644

可以看到Docker会创建（create -> commit -> destroy）一些临时的容器，如`Step 2`的`35d7b813f3ec`，生成的新的镜像作为中间镜像会被保存在cache中。可以通过下面两种方式查看该临时容器：

    $ docker inspect 3abd0b877321
    $ docker images -a |grep 3abd0b877321


3）查看所有镜像

    $ docker images

4）查看`nginx-demo`镜像信息

    $ docker inspect nginx-demo
    $ docker inspect -f '{{ .Config.ExposedPorts }}' nginx-demo


5) 查看创建镜像的历史

    $ docker history nginx-demo:latest
    IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
    7bd0c6e98644        5 minutes ago       /bin/sh -c #(nop) CMD ["nginx" "-g" "daemon o   0 B
    e55ea9a20dd4        5 minutes ago       /bin/sh -c #(nop) EXPOSE 80/tcp                 0 B
    352a72a3b8a8        5 minutes ago       /bin/sh -c apt-get update && apt-get install    71.8 MB
    3abd0b877321        6 minutes ago       /bin/sh -c #(nop) MAINTAINER John Citizen       0 B
    a4231e14d761        3 weeks ago         /bin/sh -c #(nop)  CMD ["bash"]                 0 B
    aaec12cbddb4        3 weeks ago         /bin/sh -c #(nop) ADD file:9c48682ff75c756544   123.5 MB


* 容器镜像包括元数据和文件系统，其中文件系统是指对基础镜像的文件系统的修改，元数据不影响文件系统，只是会影响容器的配置;
* 每个步骤都会生成一个新的镜像，新的镜像与上一次的镜像相比，要么元数据有了变化，要么文件系统有了变化而多加了一层;
* Docker在需要执行指令时通过创建临时镜像，运行指定的命令，再通过`docker commit`来生成新的镜像;
* Docker会将中间镜像都保存在缓存中，这样将来如果能直接使用的话就不需要再从头创建了。
* 由于每一行指令，都会产生一个镜像，因此可以使用链式指令，有助于构建较小的镜像。(更多内容可以阅读《高性能Docker》)

### 四、Docker仓库
#### 【练习2】使用Docker public仓库

    $ docker search busybox
    NAME                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
    busybox                         Busybox base image.                             1052      [OK]
    progrium/busybox                                                                65                   [OK]
    radial/busyboxplus              Full-chain, Internet enabled, busybox made...   13                   [OK]
    container4armhf/armhf-busybox   Automated build of Busybox for armhf devic...   8                    [OK]
    ofayau/busybox-jvm              Prepare busybox to install a 32 bits JVM.       2                    [OK]
    azukiapp/busybox                This image is meant to be used as the base...   2                    [OK]
    multiarch/busybox               multiarch ports of ubuntu-debootstrap           2                    [OK]
    ofayau/busybox-libc32           Busybox with 32 bits (and 64 bits) libs         1                    [OK]
    skomma/busybox-data             Docker image suitable for data volume cont...   1                    [OK]
    prom/busybox                    Prometheus Busybox Docker base images           1                    [OK]
    elektritter/busybox-teamspeak   Leightweight teamspeak3 container based on...   1                    [OK]
    clover/busybox                  BusyBox base image                              1                    [OK]
    getblank/busybox                Docker container busybox for Blank              1                    [OK]
    zanner/busybox                  https://github.com/sergej-kucharev/zanner-...   1                    [OK]
    cucy/busybox                    aouto  build busybox                            0                    [OK]
    adamant/busybox                 Busybox base image and debian package inst...   0                    [OK]
    sdurrheimer/prom-busybox        Moved to https://hub.docker.com/r/prom/bus...   0                    [OK]
    ggtools/busybox-ubuntu          Busybox ubuntu version with extra goodies       0                    [OK]
    ddn0/busybox                    fork of official busybox                        0                    [OK]
    hongtao12310/busybox            for busybox image based on the gcr.io/goog...   0                    [OK]
    freenas/busybox                 Simple Busybox interactive Linux container      0                    [OK]
    kakaximeng/busybox              busybox                                         0                    [OK]
    jahroots/busybox                Busybox containers                              0                    [OK]
    jiangshouzhuang/busybox         busybox                                         0                    [OK]
    stubbornrock/busybox            busybox:latest                                  0                    [OK]

    $ docker pull busybox


#### 【练习3】搭建Docker private仓库
1) 需要提前创建证书文件cert，并放到相应的地方。
通过` registry:2`镜像运行起了一个私有registry如下：

    docker run -d \
      -p 5000:5000 \
      --restart=always \
      --name registry \
      -v /home/vagrant/topics/tmp/registry-certs:/certs \
      -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry_hostname.crt \
      -e REGISTRY_HTTP_TLS_KEY=/certs/registry_hostname.key \
      registry:2

2) 查看所有docker容器

    $ docker ps -a
3) 访问私有registry接口

    $ sudo curl -is --cacert /etc/docker/certs.d/registry:5000/registry.crt https://registry:5000/v2/
    HTTP/1.1 200 OK
    Content-Length: 2
    Content-Type: application/json; charset=utf-8
    Docker-Distribution-Api-Version: registry/2.0
    Date: Wed, 12 Jul 2017 13:30:17 GMT
    HTTP/1.1 200 OK

### 五、发布镜像到仓库
#### 【练习4】上传镜像到registry
1）给镜像打标签

    $ docker tag nginx-demo registry:5000/nginx-demo

2) 查看所有镜像

    $ docker images
    REPOSITORY                      TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
    nginx-demo                      latest              8ef43fa97b67        About a minute ago   195.3 MB
    registry:5000/nginx-demo        latest              8ef43fa97b67        About a minute ago   195.3 MB

3）上传镜像

    $ docker push registry:5000/nginx-demo
    The push refers to a repository [registry:5000/nginx-demo] (len: 1)
    8ef43fa97b67: Pushed
    5982f0a481be: Pushed
    c329ad2e9994: Pushed
    cdea4d036852: Pushed
    aaec12cbddb4: Pushed
    latest: digest: sha256:399e271209f61771afc03896bea5cd107d2da64985b7bea6733b9380b181b31b size: 8470

4）访问registry接口查看镜像是否存在

    $ export CERT_PATH=/etc/docker/certs.d/registry:5000/registry.crt
    $ sudo curl -s --cacert ${CERT_PATH} https://registry:5000/v2/_catalog
    {"repositories":["nginx-demo"]}

5) 删除本地镜像

    $ docker rmi nginx-demo
    Untagged: nginx-demo:latest
    Deleted: 7bd0c6e986444a5855665a4b746130088904bc5b7626d5bb5c0dc3ab52fc1142
    Deleted: e55ea9a20dd4b5d4ede4a4873e18e94f139df646085c79c83d12f71815559dda
    Deleted: 352a72a3b8a8ad51f8a84a357fcb3af0a127c8cefc34b6d5839fdcd8c06ececd
    Deleted: 3abd0b8773212881d3e73babe2304d5b760c0b156095bd9496ceb2fedf71fd62

6）从私有registry上拉取镜像

    $ docker pull registry:5000/nginx-demo
    Using default tag: latest
    latest: Pulling from nginx-demo
    31f1e1f22679: Pull complete
    51c8e8d08671: Pull complete
    04ec693a6b9c: Pull complete
    ae06ee9e3b16: Pull complete
    59c395799b5f: Pull complete
    26d4e0d847a6: Pull complete
    Digest: sha256:399e271209f61771afc03896bea5cd107d2da64985b7bea6733b9380b181b31b
    Status: Downloaded newer image for registry:5000/nginx-demo:latest

### 六、启动Docker容器
#### 【练习5】启动Docker容器
1) 启动一个容器

    $ docker run -d -p 80:80 --name nginx-app registry:5000/nginx-demo
    a56d494e8af22247eccbd7e99f108501684b52395a50f479cca09caa8ec18b19
    $ docker ps
    CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS              PORTS                    NAMES
    a56d494e8af2        registry:5000/nginx-demo   "nginx -g 'daemon off"   3 seconds ago       Up 2 seconds        0.0.0.0:80->80/tcp       nginx-app
    7d01136fd367        registry:2                 "/bin/registry /etc/d"   22 minutes ago      Up 22 minutes       0.0.0.0:5000->5000/tcp   registry

此处用`docker run`是创建并启动，`docker create`是单纯创建容器。
2）查看容器log

    $ docker logs nginx-app

### 七、连接Docker容器
#### 【练习6】连接多个Docker容器
假设我们有如下三个应用，且都已经publish到了registry中：
* shop-app，需要连接review和catalogue
* review，微服务
* catalogue，微服务

1）先启动`review`和`catalogue`

    $ docker run -d -p 8082:8082 --name review registry:5000/review
    $ docker run -d -p 8084:8084 --name catalogue registry:5000/catalogue

2）连接`shop-app`到`review`和`catalogue`

    $ docker run -d -p 8080:8080 --name shop-app --link review:review --link catalogue:catalogue registry:5000/shop-app

3）查看`shop-app`容器

    $ docker exec shop-app cat /etc/hosts

4）确认`shop-app`可以连接到`review`

    $ docker exec -t shop-app ping review

　　以下是Docker link的一些限制：
* Docker link 只在同一宿主机内可以使用
* 重新创建容器将会移除之前的链接
* 被连接的容器必须是一个已经启动的容器

　　在Docker发布1.9版本的时候推荐使用`Docker Networks`代替`Docker link`，后面的[Docker Workshop（五）Docker网络]()会讲到Docker Networks。

### 八、使用Docker Compose
　　Compose是用于定义和运行复杂Docker应用的工具。你可以在一个文件中定义一个多容器的应用，然后使用一条命令来启动你的应用，然后所有相关的操作都会被自动完成。

#### 【练习7】使用Docker compose工具
1）创建一个`docker-compose.yml`文件
```yml
shop-app:
  image: registry:5000/shop-app
  ports:
    - "8080:8080"
  links:
    - review
    - catalogue

review:
  image: registry:5000/review

catalogue:
  image: registry:5000/catalogue
```
2) 启动三个容器

    $ docker-compose up
3） 查看运行的容器

    $ docker-compose ps
3）查看log

    $ docker-compose logs

`docker-compose`可以查看所有在`docker-compose.yml`中声明的容器的日志，不用切换tab查看单独的docker容器日志。

### 九、使用容器时要避免的做法
* 不要在容器中保存数据（Don’t store data in containers）
* 将应用打包到镜像再部署而不是更新到已有容器（Don’t ship your application in two pieces）
* 不要产生过大的镜像 （Don’t create large images）
* 不要使用单层镜像 （Don’t use a single layer image）
* 不要从运行着的容器上产生镜像 （Don’t create images from running containers ）
* 不要只是使用 “latest”标签 （Don’t use only the “latest” tag）
* 不要在容器内运行超过一个的进程 （Don’t run more than one process in a single container ）
* 不要在容器内保存 credentials，而是要从外面通过环境变量传入 （ Don’t store credentials in the image. Use environment variables）
* 不要使用 root 用户跑容器进程（Don’t run processes as a root user ）
* 不要依赖于IP地址，而是要从外面通过环境变量传入 （Don’t rely on IP addresses ）

### 最后
　　本篇文章主要是按照构建流程练习了docker的一些常见基本指令，指令并不全面，详细的指令可以参考[官网](https://docs.docker.com/engine/reference/commandline/docker/#parent-command)进行练习。
　　[下一篇](http://zhangyuyu.github.io/2017/07/11/Docker-workshop-3-Docker%E7%9A%84%E5%9F%BA%E6%9C%AC%E6%93%8D%E4%BD%9C/)将讲述Docker的数据存储。

### References
* [Docker笔记](http://feisky.xyz/docker/index.html)
* [10 things to avoid in docker containers](https://developers.redhat.com/blog/2016/02/24/10-things-to-avoid-in-docker-containers/)
