---
layout: post
title: "TW|我的第一个IoT项目"
date: 2018-06-25 09:12:00
categories: life
tags:
- team
- tw
- iot
---

无关技术，无关客户，无关生活，记录而已。  
诸多小点，诸多思绪，诸多体验，纯粹感受。  

<!-- more -->

### 前言
　　刚开始听到要去一个 IoT 相关项目的时候，内心是激动的，也许是最近 IoT 大火的缘故，也许是至少能够运用大学相关知识的兴奋，
也许是不在熟悉的领域体验不一样事物的新奇，也许可能只是终于上项目了...

### 一、简介

#### 关于客户
　　Client 是世界顶尖的工业车辆(主要是物料搬运设备Material handling equipment）、仓储技术（Warehousing）以及
物流技术（Logistics technology）的供应商之一。  

产业： 内部物流（Intralogistics），机械工程（Mechanical engineering）  
产品： 叉车（Forklift truck）

#### 关于项目
　　项目的前两周，我一直在一张 Story Card 上 pair，以至于当有人问我的项目具体是干什么的，我惊讶的发现，我竟然无法概括。  

　　而后才明确，我们项目的主要内容是构建一个IoT 系统和平台，其他组基于该平台开发相关应用，最终实现通过网络
（Mobile Data 或者 WLAN）来连接设备（Forklift）和应用的目的。  

#### 关于技术栈

* 语言
    - C++
    - Java
    - Scala
    - Ruby
    - Lua
* 框架
    - Dropwizard
* 持续集成工具
    - GO CD
    - Docker
    - Terraform
    - AWS
* 基础交付工具
    -  Jia 及其 Confluence
* 构建工具
    - CMake
    - Gradle
* 其他第三方工具
    - Consul
    - Vault
* 内部沟通工具
    - Riot
    - Outlook web 版
* 涉及知识
    - 嵌入式
    - CAN
    - MQTT协议
    - FreeRTOS
* 其他有用的链接
    - GOCD 用来找sad 或者happy gif的网站：https://giphy.com/
    - C++编程标准：https://wiki.sei.cmu.edu/confluence/display/cplusplus/2+Rules
    - C++参考文档：https://en.cppreference.com/w/cpp
    - ARM Developer: http://infocenter.arm.com/help/index.jsp
    - Freertos手册: https://www.freertos.org/uxTaskGetStackHighWaterMark.html
    - CANopen solution: http://www.canopensolutions.com/
    - CANopen byteme: http://www.byteme.org.uk/canopenparent/canopen/
    - Managing Team Secrets Effectively: https://holderbaum.io/articles/SharedSecrets/
    - Getting Started with GNU Privacy Guard: https://spin.atomicobject.com/2013/09/25/gpg-gnu-privacy-guard/

#### 关于组织结构
　　我们是在Digital Solutions (又叫 "Digital Factory")的部门工作。  
该 Account 下有好几个项目组，其中我们组最大，人员配备大致如下：

* 来源比例上，由客户内部的7个人和 TW 的7个人组成；
* 角色分布上，一个 BA；一个 Devops，是 Tech lead；剩下的全部是 Devopler；没有 QA；
* 性别比例上，BA 是女生，而且她还是DP（Delivery Priciple），是 CST 的成员；Dev 只有我一个女生。
* 国籍分布上，有两个出差的英国人，有一个希腊人，我中国的 LTA，其余人都是德国人。
* 技术分布上，客户内部的都只 foucs 在C++的代码上；TW 的基本上都是从头学起 C++，但是有几个学的很不错的，
有对 Scala 和 Ruby非常熟悉的，有对 Java 熟悉的人。

### 二、沟通
　　这里的很多人，平常都说德语，但是他们的英文也很好，随时切换成英文。大部分时候，只要有English speaking 的人在，
他们都会选择用英文交流。

### 三、文化

　　在德国的第一个项目，感受有很多，不知道是所有德国项目的典型特征，还是只是这一个客户现场的氛围，总之，多记录多发现，
以后回想起来才会知道当初经历的一切是必然还是偶然，

* 咖啡文化

　　客户现场办公室的喝的，就是咖啡、气泡水、各种果茶包。看到很多同事一天会喝4-5杯咖啡，早上来一杯，站会后一杯，中饭之后一杯，
下午1-2杯。希望找你catch up 的时候，会提议一起喝杯咖啡，马上要开会的时候，带杯咖啡，总之，很多杯咖啡。
当然也有人会倒一大瓶气泡水，然后拿个杯子，慢慢倒着喝。

* 吃饭文化

　　大家吃饭都不会走很远，基本在同一栋楼里，下去买了就拿上来，通常是面包、pasta、沙拉、pizza，很少有人带。    
　　拿到楼上之后，大家会一起坐着聊天(详见下面的聊天文化)，度过中午的时间，中午不会休息，吃饭就是休息了。  
　　起初，不太好意思只跟中国同事坐在一起，觉得会显得太小群化，但是后来发现他们自己也小群化，比如内部员工经常坐在一起，
其他一个国家的坐在一起（只是我们看不出来）。

* 聊天文化

　　一个字形容——"尬"。很多东西不太好聊，比如敏感的政治，敏感的私人生活，敏感的各种信息。
聊的最多的是天气、各种旅游、各种技术实践、各种工作相关、或者吃的东西或者 P3文化。尝试着问他们附近的好玩的地方，
发现大部分不是本地人，他们也不知道（尴尬）。

* 博士文化

　　发现那些是博士的人，往往邮件上或者称呼上都会加上Dr.，而且大家对博士会显得特别尊重。

* 开会文化

　　时间上：本来以为德国，特别注重准时，但是我却个人感觉，他们注重的是准时结束会议，而不是准时开始会议。
比如10：00的会议，其实都是10：05分大家陆陆续续走过去，然后开始setup机器，然后时间就过去了，可能事情没有讨论完，那就定下一次会议。
想起来之前的开会，一定会提前几分钟去把环境setup 好，准时开始会议，避免浪费大家的时间，也许这个有待改进。  
　　形式上：还是和之前遇到的差不多，比较随意，可以吃、喝东西，往往不会随意打断，一般先举手，如果多个人举手，则一个一个的说自己的观点。

* 工作文化 
　　态度上，非常认真，工作期间，拿手机的次数很少，基本不会看手机、玩手机。  
　　时间上，实事求是，午饭都不会花太久，break的时间也不会很久，一起工作起来有一种一整天全神贯注在工作的感受。每周填timecard 
的时候，工作了7.5小时就填7.5小时，工作10小时就填10小时。  
　　工作与生活上，以前听说他们下班了就下班了，不再管工作的事情。是的，他们工作生活分的很开，下班了确实不管工作的事情，
但是他们也会写代码，是真心的喜欢写代码，会研究项目上可以改进的地方（不是工作的Story 卡）。此外，工作上的同事，很少看到他们约起来，
成为很好的朋友。  

* Pair文化

　　之前的项目，虽然会Pair，但是不是所有的时候都Pair，更多的时候是遇到需要share knowledge 的时候或者需要帮助的时候再去Pair。
　　这个项目组，基本上一定会Pair，每天早上站会的时候都会问是否有人rotate pair，来确保知识的传递，此外由于没有QA，所以Pair 
也是保证代码质量的途径了。
有一个现象，内部员工不太喜欢Pair，每次有人问"要Pair 吗"的时候，他们大概都会说"这个太boring 了"或者"这个快做完了"之类的话，
或者他们会内部员工pair，Twer和Twer Pair。

* Cake文化

　　经常有人放蛋糕在厨房，往往是自己的生日，自己就要负责提供蛋糕，或者一些纪念日之类的。这些蛋糕很多时候都是自己烘焙的，带到公司。
后来发现他们也会买超市的速成的，然后重新加工一下。
![](/assets/img/2018/team-cake1.png){: .img-medium}
![](/assets/img/2018/team-cake2.png){: .img-medium}

* 喝酒文化

　　客户现场，并没有其他吃的喝的，一般都是私人的。但是，有活动的时候，各种鸡尾酒调起来；看世界杯的时候，各种啤酒喝起来；
难得team building的时候，约在酒吧喝起来。  
　　在TW内部，冰箱里放满了啤酒，平常随意喝都可以，去参加awayday 的时候，火车上买了一箱啤酒喝着。  
![](/assets/img/2018/team-drink.png){: .img-medium}
　　上图是 TW 的照片，但是客户现场组织活动的时候，也差不多如上所示的各种酒水。

### 四、实践

#### 1. On boarding

##### Team 内部
　　虽然在 Confluence 上，有一个 On boarding 的 Checklist，列举了新人需要的一些权限（e.g. PGP， Riot账号，GoCD账号，
客户邮箱账号，WIFI 账号 ），但是始终觉得不够。

　　笔者个人觉得，如果还包含以下条目，就能更快的帮助新人快速了解项目：

* 环境搭建，e.g.开发环境
* 关于team，e.g.Team 介绍与定义，Team 相关的Concept，Jira 链接、GoCD链接、代码库链接等
* Assert，项目Assert相关的架构图，能够帮助了解整个项目技术相关的东西
* 技术栈List，如同上文我列出的技术栈一样，包含语言、工具、理论等，这个可以参考TW 技术雷达。

##### TW总结的Account的On boarding
　　该On boarding Slide是我加入了大概一个多月之后，才创建的，主要是 High level 的帮你了解客户。里面包括：

* 客户的介绍
* 客户组织架构及 CST团队的介绍
* TW各个团队的介绍
* Timecard 的填写
* 交通及差旅
* TW内部群组，邮件组，Google drive组

#### 2. 活动

* Team Huddle

　　这个在之前的一篇文章[Team Huddle](http://zhangyuyu.github.io/2018/05/27/Team-Event/#more)有详细的介绍，
主要是对日常Team的相关Event进行总结，里面包含基本上所有相关的活动。

* Team Outing

　　就是我们常说的 Team building，不一样的是这样的Team building是包含客户的人和我们 TW 的人。一般情况下，
会采用[doodle](https://doodle.com/)这个工具进行投票选择时间。

　　我在 Team 的这段时间，有过一次 Outing，但是这次是去酒吧喝酒，作为唯一一个女生，大晚上跑到红灯区喝酒，我犹豫了，最后没去。
这边人更喜欢的酒吧，不是那种灯红酒绿的酒吧，而是带有古典气息或者文化气息的酒馆，纯粹喝酒聊天。

* Hacknight

　　带着好奇，和想融入客户的想法报名参加了这个所谓的Hacknight。我们主要是玩的fischertechnik，用他们给我解释的话说，
就是一种Technical 的乐高。
![](/assets/img/2018/team-hacknight1.png){: .img-medium}
![](/assets/img/2018/team-hacknight2.png){: .img-medium}

附上往期的视频，https://youtu.be/vRFGdXRym8w

* Barbeque

　　这边的烤肉闻名遐迩，有幸跟着整个Account和客户们一起参与了BBQ。主要食物是：香肠、面包、芝士、番茄酱、烤肉酱、各种酒。
大口吃肉才是这边的风格，他们并不烤素菜，素菜是用来拌沙拉的。
![](/assets/img/2018/team-bbq1.png){: .img-medium}
![](/assets/img/2018/team-bbq2.png){: .img-medium}

#### 3. 物理环境与硬件

* 办公区域

![](/assets/img/2018/team-office.png){: .img-medium}

　　左边的A区域是一个大厨房，右边的B区域是工作区域，中间的部分是一些会议室。
大厨房：大家中午吃饭一般都是带上来在大厨房坐着吃，此外，Weekly全体会议，各种其他活动（比如烧烤、世界杯、HackNight）。  
　　工作区域：各个项目组都在这边，划分了组的开放区域，每个组都有对应的墙可以setup 物理story 墙。  
　　中间区域：这里有一个小厨房，放了咖啡机，小冰箱。此外，很多小组的定期 review 也会在这个开放区域进行，方便大家随时加入。  

* 办公桌
　　最喜欢的莫过于这个办公桌了，可以升降，想站着办公的时候，就升起桌子，不用自己额外买小桌子或者把电脑垫的高高的。
![](/assets/img/2018/team-adjustable-table.png){: .img-medium}

* 编程设备（显示屏、鼠标、键盘）

　　对于一个以pair为主的 team 来说，鼠标和键盘显然不够，虽然retro 上提过好几次，但是我离开这个组的时候，
还是只有一个Apple的鼠标和键盘。
德语的键盘倒是有几个，但是我和几个其他国家的人不太适应。
显示屏，倒是每个桌子都有一个，只要pair，都会接显示屏。想起以前一个人两个屏幕的时候了，一个显示屏写代码， 
一个显示屏放terminal 的输出等。

* 调试模块

　　起初的时候，各个模块都散放着，大家各取所需，比如radio module，relay module，peakcan，uart，jlink，power 等，
但是有一段时间，每对pair 都需要硬件进行调试，这就需要所有的模块都能组成一整套环境。
于是，后来大家就把一整套环境所需要的模块组装起来放在一个盒子里，每次要用的时候都拿一整个盒子，不能随意拆卸，保证每套环境的模块完整性。

* Frank

　　前面的调试模块是我们开发的平台，但是并没有和真实的物理设备关联。为了模拟IoT 的测试，我们需要真实的传感器，传送信号。
于是，为了让开发们最快的测试自己的功能，模块连接了一些传感器，比如Shock Sensor、Crash Sensor、Card Reader、
Corporate Display 等等，组装好之后，我们给其取了一个名字Frank。
Frank是几个组share 的，只有一个，大家平常用的时候，都是事先准备好自己的调试模块，然后再去连接Frank。   

* Forklift

　　有了Frank 的测试，还是不是很够，比如 Forklift 是否能够正常驾驶，是否能够正常举起货物，不会突然停在路上。于是在客户的basement 
有一个专门用来测试的Forklift，偶尔会下楼测试一些功能。
只下楼过一次，进行测试，还是看着内部员工操作，没有自己试过。

* Factory

　　上线之前， 几个内部员工会带着最近做的story，去Forklift factory，那里有更多的车可供测试。（笔者从来没有去过）

#### 4. 文档

* 技术文档
    - 代码相关，很多是直接放在了各个代码库的README里面，有如何运行应用的各种说明；
    - 硬件相关，放在了Confluence 里面，包含了很多硬件配置文档，比如Prodrive文档；
    - 软件相关，也是放在了Conflence 里面，主要是第三方软件的说明文档；

* 帮助文档，比如《如何使用DGB 进行对hardfault 的调试》、《如何发送CAN Frame》等等；
* 硬件购买，比如ARM JTAG、UART、RALY 的相关型号及其链接
* 业务文档，比如Process（e.g.Definition of Done、Team Huddle、Sprint 等）

　　还有一些缺失的，或者是正在建立/改进的文档：

* GuideLine，比如C++ Style Guide
* ADR（Architecture Decision Records）
* Concepts，对 team 涉及到相关的概念的说明，比如Agent、Behaviour
* Retro记录
* Show & Tell 记录
* 一些过程文档，比如Release的checklist、创建一个新的service的checklist
* 一些Template的整理归纳，比如globus-service-template、aws-template 等等

### 五、落幕
　　当第一时间知道要离开的时候，内心是欣喜的，而后却非常沮丧。欣喜的是，终于可以离开自己并不是很适应的组，沮丧的是因为为什么离开。
对于离开的原因，纠结了很久；对于是否留下来在另外的小组，也纠结了很久。最终还是决定离开这个组，去体验其他不一样的风格。

　　周五是我最后在项目的最后一天，想象中，站会时候跟大家交代一下今天的任务并且及时handover 出去；大家中午一起吃个饭，
或者拍个照留念一下；下班会早点走，跟大家say goodbye。

　　现实是，周五有几个同事请假了，站会人很少，承诺自己尽量把手上的这张story 做完；吃饭和往常一样，大家各自小组的去吃饭；
下班的时候，一直留到最后，送走一个一个同事，跟他们一个个说goodbye，然后继续尝试努力把手头的工作做完。

　　听着他们每个人一样的台词"It's my pleasure to work with you"，看着金数据"Feedback for Yu"里的空空如也，心情复杂不已。
相遇便注定了离别，也许在他们的眼里，这个中国女孩不过是一瞬，可是我却莫名的有种不舍，叹一口气，也许是文化不同而已吧。

### 六、思考
　　本来这个部分应该包含一些反思和未来，本来应该是积极向上的总结与展望，但是想那么多，还是要做的不是么？于是，重新给自己建了
一个trello 墙，列一些具体的可做的，Record & Track。

### 最后
　　虽然我很想在一个项目上待的时间略长，等待稍有积淀，稍有输出，稍有总结的时候。但是作为一个 LTA，只有一年的时间，更多的时候，
我不想以偏概全，多多体验不一样的项目，不一样的客户，不一样的文化，才能站的更高，视野更广，才不足以局限于自己狭窄的看法。


