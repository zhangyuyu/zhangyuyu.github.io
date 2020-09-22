---
layout: post
title: "BUG根因分析"
date: 2020-09-21
categories: 测试
tags:
- 测试
comments: true
---

## 一、前言
从单个有价值的bug入手，追踪和分析bug产生的本质原因，在此基础上对产品各个角色、以及项目流程做改善和优化。

## 二、背景
　　在行业平台试点EP，已经三个迭代了，节奏慢慢脱离了"兵荒马乱"的时代，腾出手对这三个迭代进行了一次BUG分析，
尝试看看BUG的原因都是什么，怎么能让BUG少一点，代码更可靠一点，集成测试时间短一点，上线时间趋于确定一点，团队
更有信息一点。
　　许是没有理论基础，搜索关键词不对，在内网没有找到系统成型的分析方法，索性把自己脑袋的实践和想法，沉淀下来。
而后周会的时候有人提到了"ODC"，也对应查到了"T-RCA缺陷根因分析法"，因此本文将从已经存在的ODC理论、T-RCA出发
然后补充在行业平台实践方式，最后思考之后如何结合开展。

## 三、ODC正交缺陷分类法
[百度百科-ODC法](https://baike.baidu.com/item/ODC%E6%B3%95/23163538?fr=aladdin)
>正交缺陷分类法，Orthogonal Defect Classification（简称ODC）是一种缺陷分析方法，由IBM在1992年提出。
>它通过给每个缺陷添加一些额外的属性，利用对这些属性的归纳和分析，来反映出产品的设计、代码质量、测试水平等
>各方面的问题。从而得到一些解决办法来进行改进。
### 1. 几种常见的缺陷分类方法

#### 1）按照严重程度
　　一般而言，会分为：致命、严重、一般、建议、提示，按照这种方式，我们可以知道：
- 工作的优先级
- 项目的进展状态
- 产品的质量

#### 2）按照component/module分类
　　按照每个项目涉及到的模块进行分类，可以知道代码出现问题较多的部分。

### 2. ODC
　　作为一个开发人员我发现的问题如果按类型（type）分类可能是由如下几种可能：(括号中的英文为缩写图例)
- 1) 没有正确的初始化 （Init）
- 2) 代码没有正确的check-in （Chk）
- 3) 算法问题 （Alg）
- 4) 功能性的错误，可能是模块内的功能没有被正确实现，也可能是模块与模块之间相联系的部分没有被正确实现。（Fnct Cls）
- 5) 有可能是有关时间的错误 （Time）
- 6) 界面相关的错误 （Intf）
- 7) 代码之间相关联的错误，例如错误的继承关系 （Rel'n）


## 四、T-RCA
　　T-RCA缺陷根因分析法（T-RCA，Test Root Cause Analysis），主要目的是基于缺陷的过程改进
（开发的改进、测试的改进、组织的改进），通过问10个问题来对Bug进行根因分析，看似很简单，实则重在引导和交流。

### 1. 10w
![](/assets/img/2020/20200921-T-RCA.png)
从右边开始：
1. 1w：这个问题是什么？有什么影响？
2. 2w：为什么会出现这个问题？什么场景下会出现这个问题？	
3. 3w：这个问题是在哪个阶段发现的？	
4. 4w：缺陷是在哪个阶段引入的？	
5. 5w：为什么会在这个阶段引入问题？	
6. 6w：（hoW）如何避免引入这个问题？	
7. 7w：应该在哪个阶段发现这个问题？	
8. 8w：为什么没有在这个阶段发现这个问题？	
9. 9w：（hoW）如何才能在这个阶段发现这个问题？	
10. 10w：（hoW）如何改进基于风险测试的过程，提取预估到这样的产品风险？

### 2. 注意事项 
- 分析所有的缺陷，包括review类的缺陷，但可以不包括测试用例类的检视缺陷。
- 分析所有阶段发现的缺陷，包括review阶段、UT、IT、ST、AT等阶段。
- 根据缺陷级别的不同，缺陷根因分析的深度可以不同。
- 对于那些无法重现的问题，还可以分析为什么无法重现，在提高可测试性方面可以有哪些改进。

### 3. 小结
　　T-RCA，是针对某个具体的问题，穿透表面现象，进行刨根问底的分析，这种分析方法很有价值，但是并不是每个BUG都要进行
如此重量的寻因。所以此方法，可以用来分析线上故障或者严重致命等级的缺陷。

## 四、行业平台的实践
前置条件：BUG单中填写相应的缺陷类型、缺陷原因、缺陷重现程度、严重级别、解决方法、版本、迭代等。

### 1. 缺陷类型
序号	缺陷类型名称	描述
1	功能缺陷	功能失效或者功能没有按照用户需求实现。
2	性能缺陷	性能导致用户体验差或不满足系统并发容量规格。
3	可靠性缺陷	包括故障出现无法检测，故障导致业务中断，故障出现后无法恢复等。
4	用户体验缺陷	包括可用性，业务响应时间，易用性，易学性，友好性。
5	安全缺陷	不满足公司的IT安全规范或不满足用户特定的安全需求和系统本身的安全设计。
6	兼容性缺陷	在用户使用需求的范围内出现的不兼容问题，包括浏览器，分辨率，文档，硬件兼容等。
7	可服务性缺陷	包括可安装性和可维护性。如按照指导书无法完成安装或升级，安装或升级失败无法回退，升级导致数据丢失，日常维护无健康检查手段，出现故障无日志，工具等定位手段。


### 2. 缺陷原因
按照缺陷引入主要原因的维度划分,缺陷包括以下类型：

序号	缺陷原因	描述
1	需求缺陷	由于需求的问题引起的缺陷，如需求遗漏，需求描述不清晰。
2	架构缺陷	由于构架的问题引起的缺陷，如架构不合理，架构性能不满足。
3	设计缺陷	由于设计的问题引起的缺陷，如UI设计、流程设计不符合需求，接口设计错误，设计描述不清晰导致开发错误。
4	编码缺陷	由于开发编码的问题引起的缺陷，如开发没有按照设计实现，编码没有按照编码规范导致低级问题，单测试和开发自验遗漏，合入新代码造成已测试过的旧功能出现问题。
5	测试缺陷	由于测试的问题引起的缺陷，如集成测试和系统测试的测试场景遗漏，测试因子遗漏等导致的测试用例遗漏，或测试因子取值错误导致的测试用例错误，还有测试执行遗漏或没有按照用例执行。
6	集成缺陷	由于集成发布的问题引起的缺陷，如发布打包错误。


### 3. 缺陷重现程度
按照缺陷重现的概率，划分为以下4个级别：

序号	缺陷重现程度	描述
1	必然重现	在满足指定条件下，缺陷必然出现。
2	有条件概率重现	在满足指定条件下，缺陷概率性出现。
3	无条件概率重现	没有特定的条件，缺陷概率性出现。
4	无法重现	只出现过一次，缺陷暂时无法在任何条件下重现。


### 4. 缺陷严重级别

### 5. 缺陷解决方法

序号	解决方法	描述
1	空	新创建的缺陷，还没有处理，还没有定解决方法，此时为空。
2	无需解决	确认是缺陷，但经过PM评估觉得没必要解决，并且以后也不会解决，此时选择无需解决。
3	延期解决	确认是缺陷，但经过PM评估在后续版本再解决，当前版本不解决，此时选择延期解决。
4	无法重现	从问题现象看是缺陷，但目前无法重现，此时选择无法重现。
5	外部原因	此问题不是本系统的问题，是由其他系统引起的问题，此时选择外部原因。
6	重复	此问题是重复问题，以前已经提过缺陷单，此时选择重复。
7	设计如此	确认为非缺陷，就是这么设计的，且已与客户沟通达成一致，此时选择设计如此。
8	问题描述不准确	此问题描述不准确，无法判断是否缺陷，此时选择问题描述不准确。
9	挂起	确认是缺陷且需要解决，但目前暂时没有解决办法，此时选择挂起。
10	需求变更	此问题是由于需求变更引起，此时选择需求变更。
11	已修改	此问题已修改解决，此时选择已修改。
12	已转需求	此问题已转为需求，此时选择已转需求。

## References