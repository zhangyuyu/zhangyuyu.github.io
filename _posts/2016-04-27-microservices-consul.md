---
layout: post
title: "Consul架构"
date: 2016-04-27 19:10:52
categories: devops
tags: 
- devops
- microservices
- consul
---

### 术语
- [Agent](#agent)
- [Client](#client)
- [Server](#server)
- [DataCenter](#datacenter)
- [Gossip](#gossip)
- [Consensus](#consensus)
- [RPC](#rpc)

#### <span id="agent"></span>Agent
　　`Consul agent`有两种运行模式：[Server](#server) 和 [Client](#client)。这里的Server和Client只是Consul集群层面的区分，与搭建在Cluster之上的应用服务无关。  
　　agent负责节点以及节点上服务的健康检查，健康检查是每个服务发现框架中重要组成部分，因为只有健康的服务才应该被clients发现，不健康的主机会被Consul服务注销.Agent之间是通过指定的端口以及TCP和UDP协议进行通信的。

#### <span id="client"></span>Client
　　Client节点是相对无状态的，Client的唯一活动就是转发请求给`Server agent`节点，以保持低延迟和少资源消耗。

#### <span id="server"></span>Server
　　以Server模式运行的Consul agent节点用于维护Consul集群的状态，官方建议每个Consul Cluster至少有3个或以上的运行在Server模式的Agent,Client节点则不限。  
　　每个[DataCenter](#datacenter)的Consul Cluster都会在`server agent`节点中选出一个Leader节点，这个选举过程通过Consul实现的[Raft Protocol](https://raft.github.io/)保证，多个`server agent`节点上的Consul数据信息是强一致的。

#### <span id="datacenter"></span>DataCenter
　　数据中心似乎是显而易见的，但也有微妙的细节，如EC2多个可用区。 我们定义了一个数据中心是一个联网环境是私有的，低延迟和高带宽。 这不包括通信，将穿越公共互联网。

#### <span id="gossip"></span>Gossip
　　Gossip协议是电脑之间的通信协议，受启发于现实社会的流言蜚语。现代分布式系统通常用Gossip协议来解决一些用其他方法难以解决的问题，可能是因为当前网络有一个不便的问题——过于庞大，或许是因为Gossip协议有时候是最为行之有效的方法。  
　　"传染病协议"(Epidemic protocol)有时候也是Gossip协议的同义词，因为gossip协议传播信息的方式，有时候很类似于生物体内的病毒传播。  
　　简言之，Gossip就是p2p协议。它主要要做的事情是，去中心化。
这个协议就是模拟人类中传播谣言的行为而来。首先要传播谣言就要有种子节点。种子节点每秒都会随机向其他节点发送自己所拥有的节点列表，以及需要传播的消息。任何新加入的节点，就在这种传播方式下很快地被全网所知道。  
　　LAN gossip pool包含了同一局域网内所有节点，包括`server agent`与`client agent`。这基本上是位于同一个数据中心DC。  
　　WAN gossip pool一般仅包含`server agent`，将跨越多个DC数据中心，通过互联网或广域网进行通信。

#### <span id="consensus"></span>Consensus
　　一致性协议使用的是[Raft Protocol](https://raft.github.io/)

#### <span id="rpc"></span>RPC
　　RPC(Remote Procedure Call)远程过程调用,这是一个请求/响应机制，允许一个客户端，向一个服务器发送请求。
`Leader server agent`负责所有的RPC请求，查询并相应。所以其他服务器收到`client agent`的RPC请求时，会转发到`Leader server agent`。  
　　同事整理的RPC文档→[Remote Procedure Call](http://koly.me/2016/04/22/RPC-and-Apache-Thrift/)

　　下图为Consul的架构图：
![](/assets/img/consul-arch.jpg)

### 使用Consul发现服务的三个组件
1. Consul 存储服务信息  
　　tool to store information about services

2. Registrator 注册Docker服务  
　　tool to register Docker servicecs

3. Consul-template 查询注册的服务并应用配置  
　　tool to query registered services and apply configuration

　　下图为Consul和ECS之间一起运作的架构：
![](/assets/img/consul-EC2.png)

### 参考
* Consul官网对于Consul架构的解释：
[CONSUL ARCHITECTURE](https://www.consul.io/docs/internals/architecture.html)

* Amazon官网上的博客：
    * [Service Discovery via Consul with Amazon ECS](https://aws.amazon.com/blogs/compute/service-discovery-via-consul-with-amazon-ecs/)  
    * 翻译的中文版地址: [在ECS上使用Consul实现服务发现](http://yaowenjie.github.io/cloud/service-discovery-via-consul-with-amazon-ecs)
