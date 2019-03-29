---
layout: post
title: "Nexus - Jenkins pipeline Job构建、上传、部署"
date: 2018-01-13 13:56:23
categories: devops
tags:
- devops
- nexus
---
## 一、前言

　　上一篇里面，在Jenkins容器里面，创建了一个简单的gradle job，进行构建产物。
本篇则构建多个stage：构建、上传、形成一个stage view。

<!-- more -->

　　本打算把build docker image，publish docker image的stage加上，但是中间出现了一些问题，主要是在jenkins的docker容器里运行docker，就形成了docker in docker 的局面。这部分问题查阅了相关的资料，没找到非常满意的做法，因此暂不在本文涉及。

　　下面暂时列举一些参考链接：

* [katacoda上关于Jenkins - Building Docker Images using Jenkins的实验课程]
(https://www.katacoda.com/courses/jenkins/build-docker-images)
* [Running Docker in Jenkins (in Docker)](http://container-solutions.com/running-docker-in-jenkins-in-docker/)
* [The simple way to run Docker-in-Docker for CI](https://getintodevops.com/blog/the-simple-way-to-run-docker-in-docker-for-ci)

## 二、安装Pipeline插件

　　如果你是新建一个jenkins容器，只需要在安装的时候勾选Pipeline插件即可。

　　如果你是已经存在的jenkins容器，则进入Jenkins -> Plugin Manager -> [available](http://localhost:51001/pluginManager/available)，找到Pipeline插件，选择install without restart等待安装完成即可。

## 三、新建pipeline工程

### 1. 新建item，选择pipeline工程
　　![](/assets/img/nexus-jenkins-new-item-pipeline.png 800 400%}

### 2. 配置

* General，去一个项目名称
* Pipeline，选择Pipeline Script；在Script里面写下如下脚本：
```
node {
   stage 'Checkout'
   git([url: 'https://github.com/zhangyuyu/Simple-web.git', branch: 'master'])
   
   stage 'Build'
   sh './gradlew clean build'
   
   stage 'Upload'
   sh './gradlew uploadArchives'
}

```

### 3. Build Now
　　配置完成之后，即可手动触发了。
　　![](/assets/img/nexus-jenkins-pipeline-build.png 500 300%}

## 四、jenkins中涉及到的术语

1. Master
　　Master是Jenkins安装和运行的地方，它负责解析job脚本，处理任务，调度计算资源。

2. Agent 
　　Agent是负责处理从Master分发的任务。

3. Executor
　　Executor是执行任务的计算资源，它可以在Master或者Agent上运行。多个Executor也可以合作执行一些任务。

4. Job 任务
　　job用来定义具体的构建过程。一个新的job及时一个新的item（这一点区别于另外一个CI CD工具——GO CD，在Go里面，一个job只是一个pipeline的一个步骤）

5. Groovy
　　Groovy是一种基于JVM（Java虚拟机）的敏捷开发语言，它结合了Python、Ruby和Smalltalk的许多强大的特性，Groovy代码能够与Java代码很好地结合，也能用于扩展现有代码。由于其运行在 JVM 上的特性，Groovy可以使用其他Java语言编写的库。Jenkins用Groovy作为DSL。

6. Pipeline 
　　流水线即代码（Pipeline as Code），通过编码而非配置持续集成/持续交付（CI/CD）运行工具的方式定义部署。流水线使得部署是可重现、可重复的。
　　流水线包括节点（Node）、阶段（Stage）和步骤（Step）。流水线执行在节点上。节点是Jenkins安装的一部分。流水线通常包含多个阶段。一个阶段包含多个步骤。

    * node在Pipeline中的context中，node是job运行的地方。 node会给job创建一个工作空间。工作空间就是一个文件目录，这是为了避免跟资源相关的处理互相产生影响。工作空间是node创建的，在node里的所有step都执行完毕后会自动删除。

    * stage阶段，stage是一个任务执行过程的独立的并且唯一的逻辑块，Pipeline定义在语法上就是由一系列的stage组成的。 每一个stage逻辑都包含一个或多个step。

    * step步骤，一个step是整个流程中的一系列事情中的一个独立的任务，step是用来告诉Jenkins如何做。

7. Jenkinfile
　　Jenkins支持创建流水线。它使用一种基于Groovy的流水线领域特定语言（Pipeline DSL）的简单脚。而这些脚本，通常名字叫Jenkinsfile。它定义了一些根据指定参数执行简单或复杂的任务的步骤。流水线创建好后，可以用来构建代码，或者编排从代码提交到交付过程中所需的工作。Jenkins中的Jenkinsfile有点类似Docker中的Dockfile的感觉。

## 最后
　　本篇文章只是基础的记录了一下运用jenkins构建pipeline工程的做法，关于docker的复杂步骤，以后将会更新。

## References
* [试用Jenkins 2 的 Pipeline 项目](https://www.cnblogs.com/wzy5223/p/5554935.html)
* [Using a Jenkinsfile](https://jenkins.io/doc/book/pipeline/jenkinsfile/)
* [Jenkins与Docker的持续集成实践](http://dockone.io/article/2594)
