---
layout: post
title: "持续交付的思考"
date: 2019-03-28
excerpt: "持续集成 持续交付 持续部署"
categories:
- devops
tags:
- continuous delivery
- devops
comments: true
---

## 一、前言

　　敏捷宣言的第一原则："我们的首要任务是尽早持续交付有价值的软件并让客户满意"。软件发布，应该是一个快速且可重复的过程。什么是持续交付？
持续集成和持续部署又是什么？

* [说到持续持续交付，你想到了什么？](#anchor-0)
* [持续集成、持续交付、持续部署](#anchor-1)
* [从一个案例说起](#anchor-2)
* [回到开发流程](#anchor-3)
* [持续交付的原则](#anchor-4)
* [回顾](#anchor-5)
* [参考](#anchor-6)

<!-- more -->

## 二、背景
　　最近部门EA（Effective Application）有不少同事在看《持续交付——发布可靠软件的系统方法》一书。
　　这本书是前公司ThoughtWorks的同事编写的；笔者虽然没仔细按照书的内容，一个章节一个章节的去阅读，但是一直以来，都在实践着这一套方式方法。
因此，借着内部分享会议，根据自己的实践经历，进行一次技术经验分享。
　　本文是按照当时的PPT进行整理的。

## <span id="anchor-0">三、说到持续交付，你想到了什么？</span>
　　不妨先BrainStorming一下，看大家都能想到什么名词。

   ![](/assets/img/cd-branstorming.png){: .img-medium}

## <span id="anchor-1">四、持续集成、持续交付、持续部署</span>

### 1. 持续集成
　　在《持续集成》一书中，对持续集成的定义如下：持续集成是一种软件开发实践。在持续集成中，团队成员频繁集成他们的工作成果，
一般每人每天至少集成一次,也可以多次。每次集成会经过自动构建(包括自动测试)的检验，以尽快发现集成错误。
自从在团队中引入这样的实践之后，Martin Fowler发现这种方法可以显著减少集成引起的问题，并可以加快团队合作软件开发的速度。

   ![](/assets/img/continuous-integration.png){: .img-large}
　　持续集成强调开发人员提交了新代码之后，立刻进行构建、（单元）测试。根据测试结果，我们可以确定新代码和原有代码能否正确地集成在一起。


### 2. 持续交付
　　持续交付在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境的「类生产环境」（production-like environments）中。
比如，我们完成单元测试后，可以把代码部署到连接数据库的 Staging 环境中更多的测试。如果代码没有问题，可以继续手动部署到生产环境中。

   ![](/assets/img/continuous-delivery.png){: .img-large}

### 3. 持续部署
　　持续部署则是在持续交付的基础上，把部署到生产环境的过程自动化。

   ![](/assets/img/continuous-deployment.png){: .img-large}


## <span id="anchor-2">五、从一个案例说起</span>

### 1. 项目背景
　　该项目是笔者在德国的时候，作为Agile Coach时候经历的一个项目。
> 某保险公司，业务跨增，收购很多小保险公司，形成Portal的架构模式。
不同的子保险公司都有一个单独Portal，而且还有一些公共的Portal。

### 2. Portal架构
　　一般而言，Portal长什么样子呢？
   ![](/assets/img/cd-anchor2-portal1.png){: .img-large}

　　Portal的技术架构：
   ![](/assets/img/cd-anchor2-portal2.png){: .img-large}
　　详细了解Portal，可以参考笔者之前的[Portal 和 Portlet](https://zhangyuyu.github.io//portal-portlet/)

　　该项目的实际架构：
   ![](/assets/img/cd-anchor2-portal3.png){: .img-large}

### 3. Initial Workshop
　　为了挖掘客户的痛点，确定实际的需求，团队邀请相关的技术人员进行了一次Initial Workshop：
   ![](/assets/img/cd-anchor2-initial-workshop.png){: .img-large}

　　Initial Workshop中涉及到的`Elevator Pitch`和`Impact Mapping`可以参考笔者之前的文章：
* [Elevator Pitch](https://zhangyuyu.github.io//elevator-pitch/)
* [Impact Mapping](https://zhangyuyu.github.io//impact-mapping/)

### 4. 痛点和目标
　　通过大家激烈的讨论和showcase，了解到：

>* 项目有一个叫做integrator的角色
* 每次部署前，都由integrator手动的选择对应的包、环境等进行组合
* 部署的过程，涉及到300多个步骤，分类大概是100个类型
* 准备部署时，只是准备必要的包，大概需要一整周的时间
* 由于只是到最后才集成，导致整体的集成测试非常困难

　　因此，确定下来第一个阶段中
* 痛点：部署周期长，很难进行集成测试
* 目标：构建持续集成、持续交付、持续部署

### 5. 部署流水线
　　改进部署流程如下：
   ![](/assets/img/cd-anchor2-pipeline.png){: .img-large}

　　改进框架及工具如下：
   ![](/assets/img/cd-anchor2-tools.png){: .img-large}

　　可以看出，主要设计的关键点：
>1. 配置管理
2. 测试
3. 部署

#### 5.1 配置管理
　　配置管理是指一个过程，通过该过程，所有与项目相关的产物，以及它们之间的关系都被唯一定义、修改、存储和检索。主要包括：

   ![](/assets/img/cd-anchor2-config-management1.png)

　　改进计划中，采用docker，确保`工具集配置`的一致性：
   ![](/assets/img/cd-anchor2-config-management2.png){: .img-large}

　　改进计划中，采用openshift，确保`软件配置`的一致性：
   ![](/assets/img/cd-anchor2-config-management3.png){: .img-large}

#### 5.2 测试
　　测试，此处略（由另外一个团队实施自动化测试、TDD实践等。）

#### 5.3 部署
　　项目实际部署方式：
   ![](/assets/img/cd-anchor2-deployment.png)
　　左边是包，右边是环境变量，包和环境变量进行组合，手动拖拽，生成执行计划。这个过程，十分实施费力，同时手动过程容易出错。

　　因此，我们需要将上面的部署过程进行自动化。

　　自动化部署，大多数需要一系列步骤，比如配置应用程序、初始化数据、配置基础设施、操作系统和中间件，以及安装所需要的模拟外部系统等。
折旧需要我们至少关注：`构建工具`和`部署脚本`。

## <span id="anchor-3">六、回到开发流程</span>

### 1. 传统的开发模式
　　开发过程
   ![](/assets/img/cd-anchor3-traditional-develop-pattern1.png){: .img-large}

　　开发过程的循环方式：
   ![](/assets/img/cd-anchor3-traditional-develop-pattern2.png){: .img-large}

　　中间所涉及到的痛点：
   ![](/assets/img/cd-anchor3-traditional-develop-pattern3.png){: .img-large}

### 2. 持续集成的工作模式
　　开发过程：
   ![](/assets/img/cd-anchor3-develop-pattern1.png){: .img-large}

　　开发过程中的循环方式：
   ![](/assets/img/cd-anchor3-develop-pattern2.png){: .img-medium}

### 3. 持续集成的好处

>* 解放重复性劳动
* 更快修复问题
* 可视化
* 减少等待时间
* 更快交付成果

## <span id="anchor-4">七、持续交付的原则</span>

### 1. 原则一：为软件的发布创建一个可重复且可靠的过程
> * 将构建和部署流程自动化
* 将单元测试和代码分析自动化
* 将构建和部署流程自动化

　　部署流水线是一个很好的解决方案：

　　常见的Pipeline的Stage:
   ![](/assets/img/continuous-delivery-pipeline1.png){: .img-large}

　　Go cd工具的流水线示例：
   ![](/assets/img/continuous-delivery-pipeline2.png){: .img-large}

　　Jenkins工具的流水线示例：
   ![](/assets/img/continuous-delivery-pipeline3.png){: .img-large}

### 2. 原则二：将几乎所有事情自动化
> * 自动化构建（maven、gradle）
* 自动化测试（cucumber）
* 基础设施即代码（CloudFormation、Dockerfile）
* 自动化配置管理（Ansible、Terraform）

### 3. 原则三：对所有内容进行版本控制
   ![](/assets/img/continuous-delivery-7-steps-git-commit.png){: .img-large}

Note: 版本控制的时候，通常采用七步提交法
   ![](/assets/img/continuous-delivery-version-everything.png)

### 4. 原则四：提前并频繁的做让你感到痛苦的事情

>* 集成
* 测试
* 发布

### 5. 原则五：内建质量
　　越早发现缺陷，修复他们的成本越低。

　　测试不是一个阶段，当然也不应该开发结束之后开始。测试不纯粹或者主要是测试人员的领域。

### 6. 原则六：“Done”意味着“已发布”
　　团队应该统一定义`Done`的含义，开发完成和测试完成并不是真正意义的Done，只能算成是Completed。
   ![](/assets/img/continuous-delivery-done-wall.png){: .img-large}

　　最好的做法，是每次上线前，在pre production环境有一次showcase，确保理解一致，确保用户验收，然后上线。
   ![](/assets/img/continuous-delivery-showcase.png){: .img-large}

### 7. 原则七：交付过程是每个人的责任
   ![](/assets/img/continuous-delivery-team.png){: .img-large}


### 7. 原则八：持续改进
　　每个迭代完成之后，需要一个retrospective回顾会议对上次的迭代进行回顾。继续做的好的，停止做的不好的，对一些改进措施分配具体的Action。
   {% capture images %}
        /assets/img/continuous-delivery-retro1.png
        /assets/img/continuous-delivery-retro2.png
   {% endcapture %}
   {% include gallery images=images cols=2 %}

## <span id="anchor-5">八、回顾</span>

### 1. 持续集成包括哪些要素？
　　一个最小化的持续集成系统需要包含以下几个要素：

   ![](/assets/img/continuous-integration-factors.png)

### 2. 持续集成一般都包含哪些任务？
　　持续集成并不是说只要代码能编译通过就是集成成功，我们已经把每次集成都看做一次完整的测试。
　　任何迁入到代码库中的代码都应该是可以部署到产品环境的。
   ![](/assets/img/continuous-integration-tasks.png)

### 3. 持续集成的任务都应该遵循什么顺序？
　　Fail fast，即快速失败。

　　一般会把运行时间短的、价值大的任务放在前面，而运行时间长的任务放置到后面。

### 4. 持续交付平台
   ![](/assets/img/continuous-delivery-platform.png){: .img-large}

### 5. Pipeline阶段
   ![](/assets/img/continuous-delivery-pipeline-stages.png)

### 6. 说到持续交付，你想到了什么
　　回顾最开始的"[说到持续持续交付，你想到了什么？](#anchor-0)"，对应到下面的图中的关键字。
   ![](/assets/img/continuous-delivery-keyword.png)

## 参考
* [The Product Managers’ Guide to Continuous Delivery and DevOps](https://www.mindtheproduct.com/2016/02/what-the-hell-are-ci-cd-and-devops-a-cheatsheet-for-the-rest-of-us/)
* [持续集成是什么？](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)
* [对于持续集成实践的常见问题的解答](https://insights.thoughtworks.cn/faq-in-continuous-integration/)
* [持续集成理论和实践的新进展](http://insights.thoughtworkers.org/continuous-integration/)
* [为什么我们迫切需要持续集成](https://waylau.com/why-we-need-continuous-integration/)
* [CI/CD Course大纲](https://www.zybuluo.com/zhongjianxin/note/606360)
