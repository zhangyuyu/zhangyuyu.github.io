---
layout: post
title: "Nexus - Sonatype Nexus搭建maven私服"
date: 2018-01-07 17:02:13
categories: devops
tags:
- devops
- nexus
---
## 一、前言
　　[上一篇](http://zhangyuyu.github.io/nexus-sonatype-nexus-primer/)里面介绍了Sonatype Nexus的基础知识，本篇文章将搭建一个maven私服，从里面获取应用程序需要的jar依赖。
<!-- more -->

## 二、搭建Nexus
　　笔者电脑是是Mac pro。

### 1. 安装Docker for Mac
　　参考https://store.docker.com/editions/community/docker-ce-desktop-mac

### 2. 安装Kitematic
　　直接点击Docker for mac，选择Kitematic即可下载安装。
![](/assets/img/nexus-install-kitematic.png){: .img-small}

### 3. 创建nexus容器
　　在Kitematic上搜索nexus，选择nexus3，点击create，即可下载镜像，创建容器。

#### 3.1 docker image
　　宿主机上docker image的存放地址：
```
$HOME/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux
```

#### 3.2 docker container
　　配置端口号如下：
![](/assets/img/nexus-container-configure.png){: .img-large}

#### 4. 访问nexus
　　访问 http://localhost:32768/ ，登录用户名`admin`，密码`admin123`。

#### 5. 配置proxy仓库
　　点击`Server Adminstration And configuration`，进入[配置页面](http://localhost:32768/#admin/repository/repositories)；选择repositories，并选择[maven-central](http://localhost:32768/#admin/repository/repositories:maven-central)。

默认配置如下：
![](/assets/img/nexus-repostories-maven-center.png)

　　如果没有改代理仓库，可自己创建一个maven central的代理仓库，并配置如上图。
r-workshop-4-docker-volume.md
## 三、搭建web应用

目录结构如下：
```
$ tree
.
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradle.properties
├── gradlew
├── gradlew.bat
└── src
    └── main
        ├── java
        │   └── com
        │       └── codetutr
        │           └── HelloWorldServlet.java
        └── webapp
            └── WEB-INF
                └── web.xml

```

### 1. gradle文件
build.gradle
```
apply plugin: 'war'
apply plugin: 'idea'
apply plugin: 'org.akhikhl.gretty'

buildscript {
  repositories {
    maven {
      url "${nexusUrl}/maven-central/"
    }
  }

  dependencies {
    classpath 'org.akhikhl.gretty:gretty:2.0.0'
  }
}

repositories {
  maven {
    url "${nexusUrl}/maven-central/"
  }
}

dependencies {
   providedCompile 'javax.servlet:servlet-api:2.5'
   runtime 'javax.servlet:jstl:1.1.2'
}

gretty {
  httpPort = 8090
  contextPath = '/simple-web'
}

task wrapper(type : Wrapper) {
  gradleVersion = '4.4.1'
}
```
注意：build.gralde文件中`${nexusUrl}`，后面跟上的是`maven-central/`，两者拼起来就是上述搭建nexus时候maven central的URL。

### 2. gralde配置文件

gradle.properties
```
nexusUrl=http://localhost:32768/repository/
nexusUsername=admin
nexusPassword=admin123
```

将nexusUrl设置为上述搭建Nexus时候，maven central的URL的前缀。

### 3. HelloWorldServlet文件
```
package com.codetutr;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class HelloWorldServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        resp.getOutputStream().write("Hello, World.".getBytes());
    }
}
```

### 4. web.xml文件
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

 <servlet>
   <display-name>HelloWorldServlet</display-name>
   <servlet-name>HelloWorldServlet</servlet-name>
   <servlet-class>com.codetutr.HelloWorldServlet</servlet-class>
 </servlet>

 <servlet-mapping>
   <servlet-name>HelloWorldServlet</servlet-name>
   <url-pattern>/</url-pattern>
 </servlet-mapping>

</web-app>
```

## 四、运行
　　.gradlew相关的文件是运行`gradle wrapper`创建的。

### 1. 从nexus上下载相应依赖jar文件
```
$ ./gradlew clean build
Download http://localhost:32768/repository/maven-central/org/akhikhl/gretty/gretty/1.4.1/gretty-1.4.1.pom
...
BUILD SUCCESSFUL in 37s
3 actionable tasks: 3 executed
```

### 2. 查看依赖jar文件

　　相关的依赖包的获取顺序为：
```
remote maven central -> nexus私服的nexus data -> 本地宿主机的~/.gradle
```

#### 2.1 nexus缓存jar文件

* 在`http://localhost:32768/#browse/browse:maven-central`可以看到：

![](/assets/img/nexus-jar-files.png){: .img-large}

* 关闭电脑的网络，手动点击右侧的path，可以下载jar文件

![](/assets/img/nexus-jar-path.png){: .img-medium}

* 手动删除宿主机上nexus-data/blob/default/content的内容，再次下载时候，会报错502。

#### 2.2 gradle缓存jar文件

* 查看本地.gradle缓存如下：
```
$ cd ~/.gradle/caches
$ ls -al
total 16
drwxr-xr-x  6 yuzhang  staff   204 Jan  7 17:25 .
drwxr-xr-x  8 yuzhang  staff   272 Jan  7 15:26 ..
-rw-r--r--@ 1 yuzhang  staff  6148 Jan  7 17:26 .DS_Store
drwxr-xr-x  6 yuzhang  staff   204 Jan  7 15:37 4.4.1
drwxr-xr-x  5 yuzhang  staff   170 Jan  7 17:25 modules-2
drwxr-xr-x  3 yuzhang  staff   102 Jan  7 17:25 transforms-1
```

* 断开网络，删除.gralde/cache里面的`modules-2`和`transforms-1`，再次运行`./gradlew clean build`，可以看到应用依然可以获取到jar文件，而且速度很快。

### 3. 运行web应用
```
$ ./gradlew appRun
17:18:49 INFO  Jetty 9.2.22.v20170606 started and listening on port 8090
17:18:49 INFO  simple-web runs at:
17:18:49 INFO    http://localhost:8090/simple-web

> Task :appRun
Press any key to stop the server.
<===========--> 87% EXECUTING [15s]
> :appRun
> IDLE
```

### 4. 浏览器查看
　　`http://localhost:8090/simple-web/` ,可以看到`Hello, World.`出现。

## 最后
　　本篇将nexus作为maven私服，使本地的gradle web应用直接从nexus获取相关依赖，从而避免网络断开造成应用不可访问。  
　　[下一篇](http://zhangyuyu.github.io/nexus-gradle-upload-package/)基于本篇工程的基础上，会用gradle生成程序
构建产物，打包上传到nexus。

　　Github代码地址：https://github.com/zhangyuyu/Simple-web

## References
* [Simple Gradle Web Application](http://codetutr.com/2013/03/23/simple-gradle-web-application/)
