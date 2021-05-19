---
layout: post
title: "Nexus - 构建jenkins容器、配置gradle job从nexus获取依赖"
date: 2018-01-12 12:04:13
categories: devops
tags:
- devops
- nexus
---
## 一、前言
　　前面都是在手动的构建、上传，那么接下来我们会熟悉持续集成与持续交付的相关过程：
利用jenkins生成并上传构建产物、构建并上传应用程序的docker image以及部署应用。  
　　本篇文章主要是先构建一个jenkins容器，然后基本实现利用jenkins生成构建产物。
<!-- more -->

## 二、构建jenkins容器

### 1. 通过nexus获取jenkins/jenkins:lts镜像

#### 1.1 启动nexus container
　　创建container的部分，可以参考前文[Nexus - Sonatype Nexus搭建maven私服](http://zhangyuyu.github.io/nexus-sonatype-nexus-maven-server/)。
启动nexus container，可直接在docker for mac里面，点击start即可。

#### 1.2 获取镜像
```
$ docker pull localhost:50000/jenkins/jenkins:lts
```

#### 1.3 运行容器
```
$ docker run --name jenkins-container -p 51001:8080 -p 51002:50000 -v ~/Documents/jenkins_home:/var/jenkins_home localhost:50000/jenkins/jenkins:lts
```

#### 1.4 初始化jenkins配置
* 上述运行容器的命令执行之后，会出现如下信息：
  
```
*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

97b8f494cbc340c4818083XXXXXXXXXX

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************
```

* kitematic的右侧也会出现如下页面：
　　![](/assets/img/2018/nexus-jenkins-kitematic-web-view.png){: .img-large}

* 浏览器访问`http://localhost:51001/`，输入上述命令行的password
　　![](/assets/img/2018/nexus-jenkins-unlock.png){: .img-large}

* 自定义安装插件
　　建议在下图中，选择"select plugins to install"，然后不要选择太多的插件，等到要用的时候再去安装。笔者在本次使用中
只选择了`Git plugin`和`Gradle Plugin`。

    * Git插件:用于从git仓库获取代码
    * Gradle插件:用于打包gradle项目

![](/assets/img/2018/nexus-jenkins-customize-plugins.png){: .img-large}

* 创建admin user
　　为了不忘记，笔者设置的是`admin/admin`。

![](/assets/img/2018/nexus-jenkins-admin-user.png){: .img-large}

* 配置插件
　　依次选择Jenkins -> Manage Jenkins -> Global Tool Configuration：  

![](/assets/img/2018/nexus-jenkins-configure-tools.png)

## 三、配置gradle job

### 1. 新建Freestyle project
　　![](/assets/img/2018/nexus-jenkins-new-item.png)

### 2. 配置source code management
　　![](/assets/img/2018/nexus-jenkins-configure-source-code.png)

　　选择Git：
* Repository URL 填写你的项目的git地址
* Credentials 选择你的git仓库的账户密码(如果没有，请点击Add)
* Branches to build => Branch Specifier (blank for 'any') 这里填写构建项目时将要拉取的分支的名称 例如(*/master)

### 3. 配置build
　　![](/assets/img/2018/nexus-jenkins-configure-build.png)

　　选择Invoke Gradle script：
* 选择 Invoke Gradle , Gradle Version 中选择前面自动安装的gradle
* Tasks 填写 `clean build`
* 点击高级，勾上Force GRADLE_USER_HOME to use workspace

### 4. 手动在容器里进行build
#### 4.1 登录到jenkins container
```
$ bash -c "clear && docker exec -it jenkins-container sh"
```

#### 4.2 在jenkins container里面
```
$ cd /var/jenkins_home/workspace/simple-web
$ ./gradlew clean build
```

#### 4.3 可能出现的错误
##### 4.3.1 具体错误
```
$ ./gradlew clean build

FAILURE: Build failed with an exception.

* What went wrong:
A problem occurred configuring root project 'simple-web'.
> Could not resolve all files for configuration ':classpath'.
   > Could not resolve org.akhikhl.gretty:gretty:2.0.0.
     Required by:
         project :
      > Could not resolve org.akhikhl.gretty:gretty:2.0.0.
         > Could not get resource 'http://localhost:32768/repository/maven-central/org/akhikhl/gretty/gretty/2.0.0/gretty-2.0.0.pom'.
            > Could not HEAD 'http://localhost:32768/repository/maven-central/org/akhikhl/gretty/gretty/2.0.0/gretty-2.0.0.pom'.
               > Connect to localhost:32768 [localhost/127.0.0.1] failed: Connection refused (Connection refused)
```

##### 4.3.2 原因
　　由于应用simple-web的代码`gradle.properties`里面的nexus是localhost，而实际上期望的是我们自己搭建的nexus repo的地址，
因此会报错，连接失败，下载不了依赖。
```
nexusUrl=http://localhost:32768/repository/
```

##### 4.3.3 解决办法
　　这里先提供粗暴的直接解决办法，下文会自动化整个过程。一般docker之间的互连有如下三种方式：
* 通过端口公开（port exposure）连接
* 将宿主机端口绑定（bind）至容器端口
* 通过链接（link）选项去连接两个容器

　　对于第一种方式，由于我们的容器已经正在运行了，不好再次公开端口；对于第二种方式，应该是连接
jenkins container -> host -> nexus container，但是nexus container已经将32768端口号和宿主机绑定，
jenkins就无法再次绑定该端口；第三种方式可以一试。

##### 4.3.4 具体解决步骤
* 先构建nexus3-container容器和jenkins-container的网络连接
用Docker for mac，在两个container的network设置处，都要配置Links：
![](/assets/img/2018/nexus-jenkins-link-container.png){: .img-large}
之后登陆到`jenkins-container`，就可以ping通`nexus3-container`了。

* 将gradle.properties文件里的nexusUrl设置为`nexus3-container`的ip地址及端口号`510001`。

　　上述方式缺点很明显，每次构建都要更改代码里的nexusUrl，依赖于一个动态ip配置；此外，用link必须要求容器是正在运行的，因此每次都必须先启动两个容器，再去配置link container。可以有几种解决方案，比如动态设置环境变量、比如设置静态子网ip（下文将讲述该方式）。

### 5. 通过jenkins UI进行build
　　点击左侧的`Build Now`，即可看到`Build History`里面有Build的次数和时间信息。点击进入，可看到详细的console构建信息：
　　![](/assets/img/2018/nexus-jenkins-build-console.png)

## 四、自动化

### 1. 删除已有container
　　手动删除本地的nexus3-container和jenkins-container，不删除其对应的容器数据。

### 2. 创建docker-compose.yml
　　下面的docker-compose文件基本是按照现有的容器配置进行编写的，唯一不同的是，增加了关于网络的部分：
```
version: '3'
services:
  nexus-service:
    image: sonatype/nexus3:latest
    container_name: nexus3-container
    hostname: wanzi
    ports:
      - "32768:8081"
      - "50000:50000"
      - "50001:50001"
      - "50002:50002"
    volumes:
      - ~/Documents/Kitematic/nexus3-container/nexus-data:/nexus-data
    networks:
      nexus-net:
        ipv4_address: 172.16.238.10
    restart: always

  jenkins-service:
    image: localhost:50000/jenkins/jenkins:lts
    container_name: jenkins-container
    hostname: wanzi
    volumes:
      - ~/Documents/jenkins_home:/var/jenkins_home
    networks:
      nexus-net:
        ipv4_address: 172.16.238.11
    ports:
      - "51001:8080"
      - "51002:50000"
    depends_on:
      - nexus-service
    restart: always

networks:
  nexus-net:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.16.238.0/24
```
#### 4. 更改nexusUrl
* 更改`simple-web`中`gradle.properties`文件的nexusUrl
```
nexusUrl=http://172.16.238.10:8081/repository/
```
* 然后push代码到github

### 3. 依次启动`nexus3-container`和`jenkins-container`
　　由于`jenkins-container`要依赖于从`nexus3-container`下载依赖，所以要先启动`nexus3-container`。
    ```
    $ docker-compose up nexus-service
    ```

　　然后启动`jenkins-container`
    ```
    $ docker-compose up jenkins-service
    ```

## 最后
　　本篇文章是以jenkins搭建持续集成、持续交付应用的首篇，主要介绍了一些基本的操作。包括搭建jenkens容器，
与前文的nexus环境进行结合，创建第一个gradle job构建产物。  
　　接下来，会逐步搭建jenkins上其他的job。

Github代码地址：https://github.com/zhangyuyu/Simple-web

## References
* [基于 git + jenkins + gradle + docker 搭建自动化集成环境](http://www.jdkhome.com/article/build-automation-by-git-jenkins-gradle-docker)
* [如何对docker-compose配置静态ip](https://deepzz.com/post/docker-compose-file.html)
* [Docker容器互联方法-篇一](http://dockone.io/article/1155)
