---
layout: post
title: "Impact Mapping"
date: 2018-07-04 08:36:55
categories: method
tags: 
- agile
---

## 一、前言

　　 Impact Mapping（影响地图）多用于分析产品愿景，分解用户故事，分析内部策略，创建和排序需求清单。本文主要从以下几个方面讲述 Impact Mapping：

* 理解Impact Mapping
* 结构
* 如何使用
* 下一步
* 简单示例

<!-- more -->

## 二、背景
　　有幸参与到一个项目的初期阶段，初期是指项目基本还没形成，team 还未组建，只有一个非常 high level 的愿景。  
　　进行项目kick off的workshop之前，项目组的 BA/QA 在准备Init Client Meeting 的 Agenda 时候，跟我介绍了Impact Mapping 这个工具，它给了一个全新的视角和方式来进行需求（目标）的分析，是一种可协作、可视化、快速的思维分析模式。

## 三、理解Impact Mapping

### 1. 什么是Impact Mapping
<center>
　　Impact mapping is a strategic planning technique. It prevents organisations from getting lost while building products and delivering projects, by clearly communicating assumptions, helping teams align their activities with overall business objectives and make better roadmap decisions.
</center>

　　Impact Mapping(影响地图)，是一种战略规划技术，通过明确传达假设，以防止组织在构建产品、交付项目时迷失方向，从而帮助团队将其活动与整体业务目标保持一致，并制定更好的路线图决策。

![](/assets/img/impact-mapping-assumptions.png)

　　简言之，Impact Mapping是一种非常简单的可视化思维分析模式，用于规划从目标到可交付成果的路径。

### 2. Impact Mapping 的目标对象

* 制定决策的人，The people defining the direction
* 实施决策的人，The people consuming the direction

### 3. 什么时候使用 Impact Mapping？

* 项目开始之前：定义目标（Goal）和认知假设（Acknowledge
assumptions）；
* 项目过程之中：当scope变化时候评估决定，验证工作是否获得了期望的影响；

### 4. 为什么使用Impact Mapping？
* 战略规划（Strategic Planning）：通过可测量的指标，帮助团队专注于可交付成果。
* 质量保证（Define Quality）：通过优先级排列，减少当产品功能和业务目标之间的不一致时候，出现的分歧。
* 增强合作：通过创建蓝图，增加业务职能和开发职能之间的理解、沟通和协作。

## 四、结构

![](/assets/img/impact-mapping-structure.png)
　　Impact Mapping包含四个层次，WHY、WHO、HOW和WHAT，分别对应了Goal、Actors、Impacts、Deliverables。

## 五、如何使用

　　我们的目标是什么（WHY），为了达成目标需要哪些人（WHO）去怎样（HOW）影响，为此我们需要做什么（WHAT）。

### 1. WHY
　　第一个问题：Why are we doing this?

　　Impact Mapping应该是从上到下的进行。有一个非常普遍的现象：在构建解决方案之前，人们往往心里有解决方案的想法，但是并没有真正考虑过该问题或者不知道怎样才算问题解决了。为了确保我们不浪费时间和精力，我们可以从目标开始，也就是“我们为什么要做这个？”

　　“我们为什么做这个？”的答案，就是我们的目标。一方面，快速响应市场的变化；另外一方面，专注在我们真实的需求上。一旦目标得到了验证，我们就可以开始下面的第二步了。

　　Note：目标应该采用SMART 原则（Specific, Measurable, Achievable, Realistic, Timely）

### 2. WHO
　　第二个问题：Who will help us?

　　没有其他人的帮助，我们不可能完成目标，这些人我们称之为Actor。Actor，可以是顾客、用户、职员、其他组织机构、当局或者其他人。为了找到不同类型的关键人物，我们可以问自己：谁是用户？谁会受到影响？谁会阻挠目标？

　　重要的参与者是那些直接影响产品成功的人，这些人可以分为：主要（Primary）、次要（Secondary）、非现阶段（Off-stage ）的参与者。

* 主要参与者，实现目标的人。
* 次要参与者，服务提供者。
* 非现阶段参与者，有利益涉及但不直接的人。

### 3. HOW
　　第三个问题：How will they be impacted?

　　Actor将如何影响我们的目标？这里既包含促进目标实现的正面行为，也包含消除阻碍目标实现的负面行为。我们Actor的行为将如何改变？哪些行为最有可能帮助我们实现目标？这些问题的答案就是我们想创造出来的impact。

　　Note: 关注在Actor 对目标有帮助的那些impact上，而不是Actor能够做的所有事情。比如购买更多的产品，向其他人推荐该产品。Impact是Actor的活动，是业务活动而不是产品功能。理想情况下应展现Actor行为的变化，而不仅仅是行为本身。

### 4. WHAT
　　第四个问题：What will we do?

　　我们可以做什么来鼓励Actor帮我们实现Goal。这里关注的是可交付的产品或者是更加详细的解决办法。包括交付内容，软件功能以及组织的活动。

　　如果我们是开发技术产品或服务，那么“可交付成果”是系统和系统相关的功能。
　　如果我们是设计或者改进业务目标，那么“可交付成果”是一些小型实验，我们将尝试一些有助于我们解决问题的行为或方式。

## 六、下一步
### 1. 优先级
　　当我们画出了Impact Mapping 之后，发现最后一步的Deliverables可能有很多，这时候我们需要进行优先级的划分。  
　　优先级的划分，取决的是业务目标而不是产品功能，因此在Impact Mapping 上，我们可以找到该成果对应的Goal，从而列出优先级。

### 2. Story
　　优先级列出来之后，我们就可以对应建卡了。建卡的模板如下：
<p align="center"><font color=#0099ff size=3>As an <b>ACTOR</b> I want <b>ACTION</b>, so I get <b>VALUE.</b></font></p>

## 七、示例
　　以一个在线零售商店为例：

### 1. WHY
Goal: 增加在线商品的购买量。 
 
Measure:
* WHAT: 接下来的三个月内，在线销售量翻倍
* WHERE: 收入
* CURRENT: $250,000
* MIN: $375,000
* MAX: $500,000

### 2. WHO
    * Primary: 顾客
    * Secondary: 营销人员、广告商，保证我们有稳定的顾客人流
    * Off-stage: 一些分析人员，通过分析来确定下一阶段的目标

### 3. HOW
    * 顾客: 买更多的商品、买高质量的商品
    * 营销人员: 吸引更多的新顾客和老顾客

### 4. WHAT
    * 顾客购买了该产品也会购买其他产品
    * 根据顾客的偏好设置进行推荐
    * 排序高质量产品
    * 目标网站进行数字广告
    * 发送电子邮件通知新产品
 
　　汇总画处Impact Mapping 的图如下：   
　　![](/assets/img/impact-mapping-example.png)

### 5. STORY
　　接下来就可以按照模板写story 了，比如：

```
As a 顾客，
I want the 在线零售商店具有一个功能：顾客购买了该产品之后也会购买其他产品
so I can 购买更多的商品
```
或者

```
As a 营销人员，
I want 目标网站进行数字广告
so I can 吸引更多的新顾客
```

## 七、总结
　　Impact Mapping 尝试用WHY WHO HOW WHAT四个问题，回答某人（WHO）可以用某种不同的方式（HOW），来实现某个目标（WHY）的方法（WHAT）。

　　Impact Mapping 不应该专属于某个职能，也不应该是某一时刻的静态规划。开发过程中，团队持续交付功能，获得反馈及其它信息输入，深化对产品的认知。随着认知的深化，Impact Mapping 不断地被修正、拓展。这一过程需要各个职能的共同参与，Impact Mapping 是管理人员、业务人员、开发和测试人员共享的完整图景。

　　对于业务人员，他们不再是简单的把需求列表扔给开发团队，并等着最后的结果。通过Impact Mapping，业务人员和开发人员一同完成从目标到产品功能的映射，明确其中的假设，并在迭代交付中验证这些假设，当假设被证明或否定后，应该对 Impact Mapping 做出调整，如继续加强或停止在某个方向上的投入，或调整投入的方式。

　　对于开发人员，他们的目标不再限定于交付功能，而是拓展至交付业务目标。开发者除了知道交付什么功能，也了解为谁开发，为什么要开发。这样就可以更加主动和创新地思考，有依据的做出决策和调整。

　　对于测试人员，除了参与上面的规划和验证活动外，测试的责任不再局限于检查产品是否符预定的功能，而是验证产品是否产生了预期的影响。如果没有对用户产生期望的影响，即便完美符合功能定义，也不是高质量的产品。

## 最后
　　在绘制 Impact Mapping 的时候，一定要确保参会人员有决策者（Decision Maker）参与，包括高级技术人员和业务人员，否则会因为缺乏合适的参与人员，导致问题导论很久都没有决定。此外，设置合理的timebox，同时确保每一步（WHY、WHO、HOW、WHAT）都有输出对应的清单，这样才能高效的产出合适的Impact Mapping。
　　个人认为影响地图的思维方法和逻辑结构是普遍适用的，因此可以应用到很多领域，不仅是战略目标、营销战略、商业分析，也适用于自身的事情，比如旅行，健身，减肥，教育，学习计划等。因为，这个方法是以实用出发，以价值为导向，以目标为导向，结果为导向，从而保持简洁的。

## References
* [The Effect Backlog, by Martin Christensen](http://modernux.se/docs/impactmapping/)
* [Impact Mapping: Making a big impact with software products and projects](https://www.impactmapping.org/book.html)
* [Impact Mapping官网](https://www.impactmapping.org/index.html)
* [Impact Mapping 影响地图 读书与演练心得](https://devopshub.cn/2016/02/24/impact-mapping-practise/)
* [解析精益产品开发（二）—— 产品开发中的价值](http://www.infoq.com/cn/articles/value-in-product-development?utm_source=articles_about_influence_map&utm_medium=link&utm_campaign=influence_map)
