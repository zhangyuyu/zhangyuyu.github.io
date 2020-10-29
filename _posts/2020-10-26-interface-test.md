---
layout: post
title: "接口测试指引"
date: 2020-10-15
categories: 测试
tags:
- 测试
comments: true
---
*  目录
{:toc}

# 前言
　　测试金字塔里，单元测试的上层便是接口测试了。接口测试用多种方式，在尝试给团队进行接口测试的标准化的时候，
接触了平台、框架等方式，由此整理一篇指引。

# 一、接口

## 1. 什么是接口

　　接口又叫API（application programing interface）应用程序编程接口，我们经常会说web接口，app接口。
是连接客户端和服务端的桥梁，同时规定了客户端和服务端之间数据交换格式，其中包括：协议，地址，参数和响应等。


## 2. 接口的类型

　　接口一般分为两种：

- 程序内部的接口
- 系统对外的接口


　　接口的分类：

- HTTP，基于HTTP协议（在TCP协议上进行封装），主要用于内部的服务调用，性能消耗低，传输效率高，服务治理方便。
- RPC，可以基于TCP协议，也可以基于HTTP协议，主要用于对外的异构环境、浏览器接口调用、APP接口调用、第三方接口调用等。



## 3. 接口生命周期

　　在研发过程中，不是只有开发阶段才涉及到接口：
![](/assets/img/2020/20201026_接口测试的生命周期.png)


# 二、接口测试

## 1. 什么是接口测试？

　　接口测试主要用于外部系统与系统之间以及内部各个子系统之间的交互点，定义特定的交互点，然后通过这些交互点来，
通过一些特殊的规则也就是协议，来进行数据之间的交互。

### 1.1 目的

　　测试接口的正确性和稳定性。

### 1.2 原理

　　模拟客户端向服务器发送请求报文，服务器接收请求报文后对相应的报文做处理并向客户端返回应答，客户端接收应答的过程。

### 1.3 重点

　　检查数据的交换，传递和控制管理过程，以及系统间的相互逻辑依赖关系，还包括处理的次数。


## 2. 为什么做接口测试？

- 尽早进行系统集成测试，暴露Bug
- 解决系统测试复杂度
- 屏蔽UI层的不稳定性
- 检查系统安全性，稳定性
- 接口经过测试稳定了，前端页面随便改，减少Bug的产生


## 3. 接口测试点是什么？

　　对于接口本身，需要验证：

- 接口是否可用
- 其请求和响应的情况
- 其业务逻辑是否正确
- 产生的效应是否符合系统设计行为

　　除了测试接口本身以外，还需要测试：

- 测试文档，验证接口说明文档与接口实际是否一致，必填项，其他字段，请求头，请求方式，返回数据格式等。
- 接口性能，比如搜索接口，要几十秒才能返回结果。



## 4. 接口测试的质量评估标准

- 业务功能覆盖是否完整  
- 业务规则覆盖是否完整    
- 参数验证是否达到要求（边界、业务规则）   
- 接口异常场景覆盖是否完整   
- 接口覆盖率是否达到要求   
- 代码覆盖率是否达到要求    
- 性能指标是否满足要求    
- 安全指标是否满足要求


# 三、接口测试平台工具

​　　目前，统观全局，既有HTTP接口，也有TAF的接口。

​　　在协议级的接口测试中，考虑到搭建和迁移成本，更快的引入接口测试，主要推荐的是用星海平台，
让开发和测试同学都投入编写接口测试，以屏蔽差异、避免重复造轮子、统一用法。

## 1. 星海平台

[星海](http://xinghai.wsd.com/)平台，支持接口测试和压力测试，是一个面向后台研发和测试团队的测试平台。

​　　优势在于，平台帮我们考虑了不用关注的细节，比如接口协议、运行机器、网络打通、日志收集、报告输出、任务管理等等。
对于测试人员来说，初步上手更快。

​　　劣势在于，添加用例比较麻烦。对于一个熟练的测试开发来说，写代码的速度是高于在web页面点点点的，
而且平台功能没有代码灵活。

​　　星海支持的协议类型：

- 传输层协议：TCP、UDP
- 应用层协议： HTTP、TRPC、TAF、WebSocket、SSO、OIDB、WUP、WNS…
- 接口描述语言：Json、Jce、PB
- 星海可以通过action自定义封装的方式，支持各种私有协议类型

## 2. 自定义接口测试框架

​　　很多项目采用Python，结合第三方HTTP库的Requests快速封装一套请求的脚本，形成一个自动化测试框架。

​　　优势在于，方便形成框架体系，比较方便优化改进，比较灵活。对于测试开发人员来说，开发工作量要小很多。

​　　劣势在于，不同的团队或者项目如果采用不一样的方式，重复造轮子现象比较严重，团队之间不能快速迁移，不便于横向对比。
此外，框架的运行受限于开发环境，不利于快速横向扩展。

## 3. 第三方工具

​　　开发同学在开发完接口，进行自测时，往往会采用一些快速便捷的方法，比如cURL 命令行，Jmeter，
Postman等第三方工具，简单明了，便于完成工作量较小的测试工作。

- cURL
- Jmeter
- Postman
- Postwoman
  
## 4. 源代码层面接口测试

​　　区别于上面3种的协议级别接口测试，在源代码层面，也是可以做接口测试的。源代码层面的接口测试，
是开发人员，使用代码自身的框架，直接模拟调用接口进行测试，这部分的测试，是自动化并且小而快的。

​　　优势在于开发直接编写，在早期就能预防Bug，同时跟随着源代码的测试更容易上手、维护和定位问题，能够增强代码的质量意识。

​　　难点在于门槛较高，一般由开发人员编写，依赖性比较强。而往往开发人员的项目时间比较紧，验证场景非常局限，
不能充分验证到异常场景。


# 四、如何做接口测试？

## 1. 接口测试流程

- 【用例管理】编写测试用例
- 【用例集管理】使用测试套件组合归类
- 【任务管理】配置执行计划
- 【报告列表】执行与报告

![](/assets/img/2020/20201026_接口测试平台.png)


## 2. 用例编写步骤

### 2.1 基本步骤

- 拿到接口的URL地址
- 查看接口是用什么方式发送
- 添加请求头，请求体
- 发送查看返回结果，校验返回结果是否正确

### 2.2 【示例】获取问答对标签列表

入口方法：

```python
def execute(context):
    host = context.get_global('''host''')
    CanonicalURI = "/industry-cgi"
    url = "https://{host}{CanonicalURI}".format(host=host, CanonicalURI=CanonicalURI)
    # 请求方法，POST、GET等
    method = '''POST'''

    get_qa_pair_tag_list(CanonicalURI, context, host, method, url)
```

测试用例方法：

```python
def get_qa_pair_tag_list(CanonicalURI, context, host, method, url):
    # 请求参数，JSON格式字符串
    param = ''''''
    # 请求体，字符串文本
    body = {
        "product_id": "2a_1585278364587_13",
        "feature_id": "2_WenDaGuanLi",
    }

    body = json.dumps(body)
    # service
    service_curr = "industry"
    request_curr = "yxw_request"
    action_curr = "GetQaPairTagList"
    version_curr = "v1"
    header = build_header(host, CanonicalURI, method, param, body,
                          service_curr, request_curr, action_curr, version_curr)
    context.ret = action.sys_httpRequest(context, url, method, param, header, body)
    print(context.ret.text)
```

构建header方法：

```python
def build_header(host, CanonicalURI, method, param, body, service_curr, request_curr, action_curr, version_curr):
    # 获得Unix时间戳
    data = time.time()
    credentialDate = time.gmtime(data)
    # 请求头，JSON格式字符串
    header = {'Content-Type': 'application/json; charset=utf-8'}
    header_str = ""
    SignedHeaders = []
    keys = header.keys()
    keys.sort()
    for key in keys:
        SignedHeaders.append(key.lower())
        header_str += key.lower() + ":" + header[key].lower() + "\n"
    SignedHeaders = ";".join(SignedHeaders)
    HashedRequestPayload = hashlib.sha256(body.encode('utf-8')).hexdigest().lower()
    # ***** 获取Signature签名 *****
    # 拼接规范请求串CanonicalRequest
    canonicalRequest = "{HTTPRequestMethod}\n{CanonicalURI}\n{CanonicalQueryString}\n{CanonicalHeaders}\n{SignedHeaders}\n{HashedRequestPayload}".format(
        HTTPRequestMethod=method, CanonicalURI=CanonicalURI, CanonicalQueryString=param, CanonicalHeaders=header_str,
        SignedHeaders=SignedHeaders, HashedRequestPayload=HashedRequestPayload)
    print(canonicalRequest)
    canonicalRequest_hash = hashlib.sha256(canonicalRequest.encode('utf-8')).hexdigest().lower()
    # 拼接待签名字符串
    CredentialScope = "{0}/{1}/{2}".format(time.strftime("%Y-%m-%d", credentialDate), service_curr, request_curr)
    StringToSign = "{Algorithm}\n{RequestTimestamp}\n{CredentialScope}\n{HashedCanonicalRequest}".format(
        Algorithm="HMAC-SHA256", RequestTimestamp=str(int(data)), CredentialScope=CredentialScope,
        HashedCanonicalRequest=canonicalRequest_hash)
    print(StringToSign)
    SecretDate = hmac.new(("YXW" + SecretKey).encode('utf-8'), (time.strftime("%Y-%m-%d", credentialDate)),
                          hashlib.sha256).digest()
    SecretService = hmac.new(SecretDate, service_curr.encode('utf-8'), hashlib.sha256).digest()
    SecretSigning = hmac.new(SecretService, request_curr.encode('utf-8'), hashlib.sha256).digest()
    Signature = hmac.new(SecretSigning, StringToSign.encode('utf-8'), digestmod=hashlib.sha256).hexdigest()
    Authorization = 'HMAC-SHA256 Credential={SecretId}/{CredentialScope}, SignedHeaders={SignedHeaders}, Signature={Signature}'.format(
        SecretId=secretId, CredentialScope=CredentialScope, SignedHeaders=SignedHeaders, Signature=Signature)
    print(Authorization)
    # header增加信息
    header['Authorization'] = Authorization
    header['YXW-Version'] = version_curr
    header['YXW-Timestamp'] = str(int(data))
    header['YXW-Action'] = action_curr
    header['YXW-SecretId'] = secretId
    header['Host'] = host
    return header
```

具体编写步骤，可以参考星海官方文档快速上手部分

## 3. 用例设计

通常情况下主要利用下面两种方式，对接口数据进行构造：

- 边界值测试
- 组合条件测试
  

请求场景：

- 覆盖所有必填参数
- 组合所有可选参数，比如某两参数，只要有一个存在即可
- 参数可选值枚举，比如type可选为day和hour
- 参数名称是否对应，比如username和name不对应
- 参数有、无、NULL、空
- 参数个数、类型
- 参数数值范围
- 参数字符长短
- 传递参数是否安全，比如明文的password
- 参数包含特殊字符或者其他校验规则
- 请求方法
- header里面的content type、token等

验证响应：

- 响应中参数是否返回
- 响应中返回的参数，是否与预期一致（包括名称、值、个数、类型、取值范围）
- 响应的状态码，是否与预期一致
- 响应的错误码，是否与业务定义一致
- 如果有多个断言点，确定断言顺序、依赖关系，比如如果某个字段出现，再去验证其值是否正确
- 如果响应没有内容，需要进一步验证效果

# 五、具体星海案例

## 1. 概览

以行业平台为案例，用户可以创建机器人，然后给自己的机器人，添加对应的问答对，支持添加一个标准问题和一系列的相似问题。

机器人的增删改查，也是需要测试的，有单独的测试用例覆盖到，下面主要以测试问答对的增删改查接口作为示例：


## 2. 问答对的接口测试

### 2.1 增

2.1.1 入口执行方法
包含了准备工作、测试逻辑、清理工作，为了与add_product和delete_product的方法区分开，测试核心部分以`test__`开头。

```python
def execute(context):
    # set up 添加一个产品
    product_id = add_product(context)
    
    # 测试核心
    test__add_qa_pairs(context, product_id)
    
    # clean up 删除产品
    delete_product(context, product_id)
   
```

2.1.2 测试逻辑方法

分为如下几个部分：

- 明确请求方法URL
- 构造请求参数、请求体
- 构造请求头
- 发送请求
- 验证结果
  其中请求体部分引入了random_str，防止重复添加。请求头`build_header`单独抽取成了一个方法。验证结果部分需要对响应中的每一个字段进行校验，而不仅仅是状态码。

![](/assets/img/2020/20201026_问答对_增_逻辑方法.png)

2.1.3 打印输出
调试完成之后，去掉不必要的打印，从剩下的信息中，可以看出测试的整体逻辑和响应输出。
![](/assets/img/2020/20201026_问答对_增_打印输出.png)

### 2.2 查

2.2.1 入口执行方法
准备工作add_product之后，添加了问答对，然后测试查询的逻辑，最后删除此次的测试数据。此时add_qa_pair去掉了前缀`test__`，而由测试核心`get_qa_pair`加上。

```python
def execute(context):
    # set up 添加一个产品、添加问答对
    product_id = add_product(context)
    qa_id = add_qa_pairs(context, product_id)
    
    # 测试核心
    test__get_qa_pairs(context, product_id, qa_id)
    
    # clean up 删除产品
    delete_product(context, product_id)
```

2.2.2 测试逻辑方法

验证结果里面对拿到的id进行了确认，确认是开始添加的qa_id。

![](/assets/img/2020/20201026_问答对_查.png)


### 2.3 改

2.3.1 入口执行方法
准备工作add_product之后，添加了问答对，获取修改之前的问答对。然后测试修改的逻辑，最后删除此次的测试数据。此时测试核心`modify_qa_pair`加上前缀`test__`。

```python
def execute(context):
    # set up 添加一个产品、添加问答对
    product_id = add_product(context)
    qa_id = add_qa_pairs(context, product_id)
    qa = get_qa_pairs(context, product_id, qa_id)
    
    # 测试核心
    test__modify_qa_pairs(context, product_id, qa_id, qa['group_id'])
    
    # clean up 删除产品
    delete_product(context, product_id)
```

2.3.2 测试逻辑方法

验证除了验证`modify`接口本身以外，还进一步获取修改之后的问答对，进行验证。

![](/assets/img/2020/20201026_问答对_改.png)

### 2.4 删

2.3.1 入口执行方法
准备工作add_product之后，添加了多个问答对。然后测试只删除前两个问答对的逻辑，最后删除此次的测试数据。

```python
def execute(context):
    # set up 添加一个产品、添加问答对
    product_id = add_product(context)
    qa_id_1 = add_qa_pairs(context, product_id)
    qa_id_2 = add_qa_pairs(context, product_id)
    qa_id_3 = add_qa_pairs(context, product_id)
    
    # 测试核心
    qa_ids = [qa_id_1, qa_id_2, qa_id_3]
    test__delete_qa_pairs_batch(context, product_id, qa_ids)
    
    # clean up 删除产品
    delete_product(context, product_id)
```

2.3.2 测试逻辑方法

验证除了验证`delete`接口本身以外，还进一步获取删除之后的剩余问答对，进行验证。

![](/assets/img/2020/20201026_问答对_删.png)

## 3. Bad Case

在review很多接口用例过程中，发现有几种常见的问题：

### 3.1 断言不够

下图中，只验证了响应是不是200。有不少业务代码，其实不管正常还是异常，都会返回200，其实具体的信息都在code和data里面。

验证code也是不够的，还需要验证具体的业务含义的字段，可以的话，最好验证实际响应里面所有的字段，包括返回字段之间的逻辑。

![](/assets/img/2020/20201026_bad_case_断言不够.png)

### 3.2 方法未抽取，层次不清晰

下图中，四个代码块，都在一个长方法里，不易于阅读，而且有两块代码块是重复的，不利于后面用例的维护和交接。

![](/assets/img/2020/20201026_bad_case_方法未抽取.png)

### 3.3 没有set up和clean up

下图中的代码出现在delete方法中，数据是写死的，也就意味着，下一次执行用例的时候一定会失败，需要手动在页面创建一个问答对，获取id，才能进行删除。
或者还有add类的方法，增加完成之后，没有清理现场，导致下一次创建会失败。

因此，建议每一个测试用例的数据，最好是先提前准备相应的字段，然后进行测试工作，最后及时清理现场，不依赖环境的数据，也不对环境造成影响。

![](/assets/img/2020/20201026_bad_case_无setup_cleanup.png)

# 六、思考

## 1. 如何提高接口测试的ROI

　　测试过程中，总是容易听到下面的声音：

![](/assets/img/2020/20201026_接口测试叠加效应.png)

　　要想提高接口测试的ROI，可以从两个方面入手：减少投入成本和增加使用率。

　　减少投入成本：

- 减少用例编写的成本，比如一个接口需要写一整天，比开发接口时间更长
- 减少用例优化的成本，比如用例描述不清，很难分层优化
- 减少用例维护的成本，比如一个接口变动，所有的测试用例都需要更改
- 减少工具开发的成本，比如公共测试服务没法用，一直重复造轮子

​　　增加使用率：

- 手工能用
- 自动化能用
- 人人能用
- 当工具用
- 各个阶段用

## 2. 如何提高接口测试的健壮性？

​　　执行接口测试的过程中，难免会报失败，执行失败的原因有很多，可以分为如下：

![](/assets/img/2020/20201026_接口测试失败原因.png)

​	被测系统出错，是我们需要验证到的；测试工具出错和不可抗力因素，是测试人员短期无法快速解决的；
那么，我们能做的就是在测试数据错误上进行避免：

- 尽量不依赖于现有的测试数据，执行前生成一些数据，执行之后删除一些数据
- 尽量支持通用逻辑部分抽取出来，便于统一修改
- 尝试支持参数使用另外一条测试用例的返回结果
- 尝试支持一些请求参数实时生成、随机生成


# 参考

- [【解决方案】1万+接口测试与管理的进阶之路](https://juejin.im/post/6857028286826315790)