---
layout: post
title: "Nexus - Sonatype Nexus入门"
date: 2018-01-06 18:08:15
categories: devops
tags:
- devops
- nexus
---
## 一、前言
　　最近在组建公司内部的Devops Community，按照一整套的项目故事线列出了Devops knowledge Library。
其中涉及到基础知识、云平台、持续集成、产出物管理、开发流程和工具、容器编排、配置管理、监控管理、日志管理、HA等。    
　　为了尽快的搭建内部人员的知识体系，按照上述流程，我们利用最常见的工具，构想了一个简单的流水线，每个人负责流水线的一部分，
然后顺序串到一起，进行输出。  
　　这里，我主要负责利用Nexus进行仓库的管理，本篇主要讲述nexus的一些基础，[下一篇](http://zhangyuyu.github.io/nexus-sonatype-nexus-maven-server/)将利用nexus作为maven的私服，利用gradle进行构建一个简单的web应用。

<!-- more -->

## 二、Nexus

　　Nexus是谷歌手机的一个牌子，我们要介绍的不是Nexus，而是Sonatype Nexus。  
　　Sonatype Nexus是Sonatype公司的一个产品，叫Nexus，它是Maven的私服。  
　　![](/assets/img/nexus-maven仓库.png)

　　事实上有三种专门的Maven仓库管理软件可以帮助我们创建私服：

* [Apache Archiva](http://archiva.apache.org/index.cgi)
* [ Artifactory](https://link.jianshu.com/?t=http://www.jfrog.com/home/v_artifactory_opensource_overview/)
* [Sonatype Nexus](http://www.sonatype.org/nexus/)。 

　　其中Archiva是开源的，Artifactory和Nexus的核心也是开源的。详细的对比可参考[Binary Repository Manager Feature Matrix](https://binary-repositories-comparison.github.io/)。Nexus是目前最常用的一个。

## 三、私服

　　私服是指私有服务器，是架设在局域网的一种特殊的远程仓库，目的是代理远程仓库及部署第三方构建。有了私服之后，当 Maven 需要下载构件时，直接请求私服，私服上存在则下载到本地仓库；否则，私服请求外部的远程仓库，将构件下载到私服，再提供给本地仓库下载。

![](/assets/img/nexus-私服.png)

## 四、Why Nexus?

### 1. 节省外网带宽
　　大量对于外部仓库的重复请求会消耗带宽，利用私服代理外部仓库，可以消除对外的重复构件下载，降低带宽的压力。

### 2. 加速Maven构建
　　不停地连接请求外部仓库十分的耗时，Maven在执行构建的时候不停地检查远程仓库的数据。利用私服，Maven只检查局域网的数据，提高构建的速度。

### 3. 部署第三方构件
　　当某个构件无法从任何一个外部远程仓库获得。建立私服之后，便可以将这些构件部署到私服，供内部的Maven项目使用。

### 4. 提高稳定性，增强控制
　　Maven构建高度依赖于远程仓库，因此，当网络不稳定的时候，Maven构建也会变得不稳定，甚至无法构建。私服缓存了大量构建，即使暂时没有网络，Maven也可以正常的运行。

### 5. 降低中央仓库的负荷
　　使用私服可以避免很多对中央仓库的重复下载，降低中央仓库的压力。

## 五、Docker搭建

　　安装过程，网上有很多资料，这里不再赘述。笔者是MAC Pro，安装了Docker for Mac，因此很容易Kitematic进行容器的安装。
![](/assets/img/nexus-kitematic-container.png)

### 1. 安装目录

下面是nexus的一些环境变量：
![](/assets/img/nexus-环境变量.png)

nexus-container中：
```
sh-4.2$ whoami
nexus
sh-4.2$ pwd
/opt/sonatype/nexus
sh-4.2$ ls -l
total 68
-rw-r--r--  1 root root 39222 Dec 27 16:00 LICENSE.txt
-rw-r--r--  1 root root   395 Dec 27 16:00 NOTICE.txt
drwxr-xr-x  3 root root  4096 Jan  2 21:41 bin
drwxr-xr-x  2 root root  4096 Jan  2 21:41 deploy
drwxr-xr-x  7 root root  4096 Jan  2 21:41 etc
drwxr-xr-x  4 root root  4096 Jan  2 21:41 lib
drwxr-xr-x  3 root root  4096 Jan  2 21:41 public
drwxr-xr-x 21 root root  4096 Jan  2 21:41 system
```
* LICENSE.txt和NOTICE.txt，包含有关许可证和版权声明的法律细节；
* bin，包含启动脚本以及启动相关的配置文件
* etc，包含配置文件
* lib，包含与Apache Karaf相关的二进制库
* public，包含应用相关的公共资源
* system，包含构成应用程序相关的所有组件和插件

### 2. 数据目录
nexus-container中：
```
sh-4.2$ cd ~
sh-4.2$ pwd
/opt/sonatype/nexus
sh-4.2$ ls ../
nexus  sonatype-work  start-nexus-repository-manager.sh
sh-4.2$ cd ../sonatype-work/nexus3/
sh-4.2$ ls
backup  blobs  cache  db  elasticsearch  etc  generated-bundles  health-check  instances  javaprefs  keystores  lock  log  orient  port  tmp
sh-4.2$
```

上述地址`/opt/sonatype/sonatype-work/nexus3`或者`/nexus-data`中，则是数据存放的地址了。

* blobs/，这是blob存储的默认地址，可在UI上的Server Adminstration And configuration进行配置。
* cache/，包含当前缓存的Karaf bundles信息。
* db/，包含OrientDB数据库，数据库存的是Nexus respository manager的元数据。
* elasticsearch，包含当前配置的Elasticsearch状态
* etc/，包含主要的运行时配置和自定义的Nexus respository manager配置。
* health-check/，包含来自 Repository Health Check功能的缓存 报告
* keystores/，包含用于鉴别Nexus respository manager的自动生成的密钥。
* log/，nexus.log文件包含Nexus respository manager运行实例的信息；该目录也包含一些日志文件的归档副本；日志滚动（Log rotation）每天进行。
* tmp/，用于临时存储。

## 六、功能介绍

### 1. Browse Server Content
![](/assets/img/nexus-功能-browse-server-content.png)

#### 1.1 Search
　　搜索功能，就是从私服上查找是否有哪些包:
* 在Search这级是支持模糊搜索的
* 如果进入具体的目录，不支持模糊搜索

#### 1.2 Browse
* Assets，能看到所有的资源，包含Jar，已经对Jar的一些描述信息。
* Components，只能看到Jar

### 2. Server Adminstration And configuration
　　看到这选项是要进行登录的，在右上角点击"Sign In"的登录按钮，输入`admin/admin123`,登录成功之后，即可看到此功能，如图所示：
![](/assets/img/nexus-功能-server-adminstration-configuration.png)

#### 2.1 Repository
* Blob Stores, 文件存储的地方，创建一个目录的话，对应文件系统的一个目录
* Repositories，仓库

##### 2.1.1 仓库类型
* hosted 
 
　　宿主仓库，用户可以把自己的一些构件，deploy到hosted中，也可以手工上传构件到hosted里。比如说oracle的驱动程序，ojdbc6.jar，在central repository是获取不到的，就需要手工上传到hosted里 
* proxy  

　　远程仓库的代理。比如说在nexus中配置了一个central repository的proxy，当用户向这个proxy请求一个artifact，这个proxy就会先在本地查找，如果找不到的话，就会从远程仓库下载，然后返回给用户，相当于起到一个中转的作用 
* group  

　　仓库组，在maven里没有这个概念，是nexus特有的。目的是将上述多个仓库聚合，对用户暴露统一的地址，这样用户就不需要在pom中配置多个地址，只要统一配置group的地址就可以了 
* virtual  

　　虚拟类型仓库，此类型主要是为了兼容maven的版本，maven版本经过大幅度提升，虚拟类型仓库主要是为了兼容maven1。

##### 2.1.2 仓库格式
[最新Nexus支持的仓库格式](https://help.sonatype.com/display/NXRM3/Supported+Formats)如下：

|Format  | 2.x| 3.x                    |
|--------|----|------------------------|
|Bower   | ❌ | ✅ 3.0+               |
|Docker  | ❌ | ✅ 3.0+               |
|git-lfs | ❌ | ✅ 3.3+ (只支持hosted) |
|Maven 1 | ✅ | ❌                    |
|Maven 2 | ✅ | ✅ 3.1+               |
|npm     | ✅ | ✅ 3.0+               |
|NuGet   | ✅ | ✅ 3.0+               |
|OBR     | ✅ | ❌                    |
|P2      | ✅ | ❌                    |
|PyPI    | ❌ | ✅ 3.0.2+             |
|RubyGems| ✅ | ✅ 3.0.2+             |
|Site/Raw| ✅ | ✅ 3.0+               |
|Yum     | ✅ | ❌ (在3.5+ 支持Proxy)  |

##### 2.1.3 仓库策略
* releases，发布版，稳定版的jar
* snapshots，快照版，一般是处于开发中的jar
* mixed，混合的

##### 2.1.4 预定义本地仓库
![](/assets/img/nexus-功能-repositories.png)

#### 2.2 Security
主要是用户、角色、权限的配置

#### 2.3 Support
包含日志及数据分析

#### 2.4 System
主要是邮件服务器，调度的设置地方

## 七、实现原理
　　Nexus Repository是以Java和JavaScript为主，实现的一个包含前端与后台的Web服务。 后台方面，它采用Jetty作为应用服务器、
Karaf作为OSGi容器、OrientDB作为数据库。 前端方面，它使用Swagger UI作为框架，是一个单页面Web App。

　　另外，它也通过Resteasy支持REST API，可以通过网络进行访问控制。并且，自行实现了一个插件系统，用插件的方式支持了更多复杂的功能。
比如，Maven、PyPI、Docker这些支持，都是由插件实现的。 如果希望支持其它方式的代理、缓存、发布，比如APT，也可以通过插件定制。

## 八、最后
　　本篇主要讲述了Nexus的概念及好处，介绍了nexus涉及到的安装目录以及数据目录，简单的列举了一下Nexus repository manager UI上的功能。  
　　[下一篇](http://zhangyuyu.github.io/nexus-sonatype-nexus-maven-server/) 将利用nexus作为maven的私服，
利用gradle进行构建一个简单的web应用。

## References
* [【项目管理】Sonatype Nexus,Maven私服](http://blog.csdn.net/gaoying_blogs/article/details/48917847)
* [使用仓库管理器——Sonatype Nexus的九大理由](http://juvenshun.iteye.com/blog/285059)
* [Nexus入门指南（图文）](http://juvenshun.iteye.com/blog/349534)
* [Tiny Nexus3.0.0+Maven的使用](http://www.tinygroup.org/docs/d0e8e7273742486ab59f161785e07a66)
* [Reference nexus-book](https://books.sonatype.com/nexus-book/3.5/reference/install.html#data-directory)
