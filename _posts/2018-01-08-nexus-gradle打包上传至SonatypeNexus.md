---
layout: post
title: "Nexus - Gradle打包上传至Sonatype Nexus"
date: 2018-01-08 18:48:36
categories: devops
tags:
- devops
- nexus
---
## 一、前言
　　[前一篇](http://zhangyuyu.github.io/2018/01/07/Nexus-SonatypeNexus%E6%90%AD%E5%BB%BAmaven%E7%A7%81%E6%9C%8D/)介绍了用nexus搭建一个maven私服，并尝试创建了一个proxy仓库。本篇将主要创建一个hosted仓库，上传gradle生成的构建产物。

<!-- more -->
## 二、搭建Nexus hosted 仓库

### 1. 创建单独的blob

　　![](/assets/img/nexus-create-local-blob.png){: .img-medium}

　　完成创建之后，可以在宿主机上看到路径`nexus-data/blobs/mvn-local-blob`的存在。

### 2. 创建hosted仓库
　　选择maven2(hosted)的仓库，进行创建：
　　![](/assets/img/nexus-create-maven2-hosted.png){: .img-medium}

　　注意选择：

* version policy：Mixed（此处笔者并不进行release和snapshot的区分，所以选择Mixed）
* blob store：刚刚创建的mvn-local-blob
* deployment policy：allow redeploy

## 三、配置gradle
　　在上一篇的基础上，build.gralde里面需要增加：

1. 定义group version，方便下面用到
```
group 'wanzi'
version '1.0.0'
```

2. maven插件
```
apply plugin: 'maven'
```

3. uploadArchives的task
```
uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "${nexusUrl}/mvn-local/") {
                authentication(userName: nexusUsername, password: nexusPassword)
            }

            pom.version = "${project.version}"
            pom.artifactId = "${project.name}"
            pom.groupId = "${project.group}"
        }
    }
}
```

* maven会以`groupId:artifactId:packaging:version`定位某一个输出物。上述指定了version、artifactId和groupId。packaging为War包。
* 注意上述repository配置的为`${nexusUrl}/mvn-local/`，拼起来就是上述创建的hosted仓库的URL。

## 四、运行

1. 执行命令`./gradlew uploadArchives`上传war包到nexus hosted repo。
2. 查看nexus上[mvn local](http://localhost:32768/#browse/browse:mvn-local)的hosted仓库，可以看到simple-web-1.0.0.war
![](/assets/img/nexus-browse-simple-web.png)

## 最后
　　本篇利用gralde生成程序构建产物，并在nexus上创建一个hosted仓库，将构建产物打包上传到nexus。

Github代码地址：https://github.com/zhangyuyu/Simple-web

## References
* [maven/gradle 打包后自动上传到nexus仓库](http://www.cnblogs.com/yjmyzz/p/auto-upload-artifact-to-nexus.html)
* [nexus-book-examples](https://github.com/sonatype/nexus-book-examples/blob/master/gradle/another-project/build.gradle)
* [Gradle系列七：依赖管理](http://timebridge.space/2016/05/21/gradle-advaced-dependency-management/)
* [maven构建产物介绍](https://www.cnblogs.com/bigtall/archive/2011/03/23/1993253.html)
