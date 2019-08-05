---
layout: post
title: "Maven中snapshot与release"
date: 2015-08-30
categories: tool
tags: 
- maven
---

## 一、从实际问题说起

### 1. 背景
　　在开发应用的时候，将应用程序分为了不同的模块，模块之间采用jar包的形式依赖。

![](/assets/img/maven-structure.png){: .img-medium}
   
### 2. 手动操作
>每次更改common时候，都需要：
* 升级common包
* 上传升级之后的common包到nexus
* 更改引用处的common的包版本

### 3. 痛点
　　多个开发频繁更改的时候，一天下来，要升级很多版本，而且不同开发升级的版本号还容易冲突。

　　如果不升级版本号，对于common包的改动，虽然上传可以成功，但是构建之后，发现始终都是此版本中最开始构建的那一次包。

### 4. 措施
　　将依赖更改为`SNAPSHOT`的依赖，具体措施为增加如下的`updatePolicy`：
```groovy
<snapshots>
    <enabled>true</enabled>
    <updatePolicy>always</updatePolicy>
</snapshots>
```
　　改之后：
```groovy
<repositories>
    <repository>
        <id>ea-nexus</id>
        <url>https://xx.xx.xx/nexus/content/groups/public/</url>
        <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
        </snapshots>
    </repository>
</repositories>
```

## 二、snapshot与release区别

　　maven中的仓库分为两种，`snapshot`快照仓库和`release`发布仓库：
* `snapshot`快照仓库用于保存开发过程中的不稳定版本。snapshot一般是开发过程中的迭代版本，snapshot更新后，引用的项目可以不修改版本号自动下载构建。
* `release`正式仓库则是用来保存稳定的发行版本。release版本不允许修改，每次进行release版本修改，发布必须提升版本号。

　　定义一个组件/模块为快照版本，只需要在pom文件中在该模块的版本号后加上`-SNAPSHOT`即可(注意这里必须是大写)
```groovy
<artifactId>pmo-common</artifactId>
<packaging>jar</packaging>
<version>1.1.9-SNAPSHOT</version>
```

　　maven2会根据模块的版本号(pom文件中的version)中是否带有`-SNAPSHOT`来判断是快照版本还是正式版本。  
　　如果是快照版本，那么在`mvn deploy`时会自动发布到快照版本库中，而使用快照版本的模块，在不更改版本号的情况下，
直接编译打包时，maven会自动从镜像服务器上下载最新的快照版本。  
　　如果是正式发布版本，那么在`mvn deploy`时会自动发布到正式版本库中，而使用正式版本的模块，在不更改版本号的情况下，
编译打包时如果本地已经存在该版本的模块则**不会主动**去镜像服务器上下载。

```groovy
<distributionManagement>
    <repository>
        <id>nexus-releases</id>
        <name>Nexus Release Repository</name>
        <url>https://xx.xx.xx/nexus/content/repositories/releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <name>Nexus Snapshot Repository</name>
        <url>https://xx.xx.xx/nexus/content/repositories/snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```
　　在上面的配置中，我们分别配置了`release`版本和`snapshot`版本所对应的Repository，如果你项目的版本中包含了`SNAPSHOT`，此时将发布到
Nexus的`Snapshots Repository`，否则发布在`Releases Repository`。

　　所以，**我们在开发阶段，可以将公用库的版本设置为快照版本，而被依赖组件则引用快照版本进行开发**，在公用库的快照版本更新后，我们也不需要
修改pom文件提示版本号来下载新的版本，直接mvn执行相关编译、打包命令即可重新下载最新的快照库了，从而也方便了我们进行开发。

## 参考
* [理解Maven中的SNAPSHOT版本和正式版本](http://www.huangbowen.net/blog/2016/01/29/understand-official-version-and-snapshot-version-in-maven/)
* [官网Maven Setting](https://maven.apache.org/ref/3.6.0/maven-settings/settings.html)
