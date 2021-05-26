---
layout: post
title: "TarsGo Step By Step"
date: 2020-11-13
categories: tech
tags:
- golang
comments: true
---

*  目录
{:toc}

# 一、前言

　　[腾讯 Tars](https://github.com/TarsCloud/Tars) 是腾讯内部使用的 TAF（Tencent Application 
Framework）的对外开源版，去掉了许多冗杂多余的部分。该框架集开发、运维、微服务、RPC 等为一体。对程序员而言，
这就是一个能够快速搭建整个微服务体系的开发框架。这个框架支持基于 [C++](https://github.com/TarsCloud/TarsCpp)、
[Node.js](https://github.com/tars-node/Tars.js/)、[PHP](https://github.com/TarsPHP/TarsPHP/)、
[Java](https://github.com/TarsCloud/TarsJava) 等语言开发，最新版本已经支持后台开发语言
新贵——[Go](https://github.com/TarsCloud/TarsGo)。

　　本文旨在还原TarsGo Step by Step的过程。


# 二、背景

　　目前接触的很多应用都是GoLang开发的，虽然熟悉了GoLang的语法，但是与我而言，基于TAF协议开发的服务，
其整个的开发部署流程是一个黑盒，不知道开发完了，怎么调用，怎么访问，怎么在TAF上操作，
也不知道Goloang的开发模式是什么样子的。断断续续有几次按照Step by Step走，
始终没有形成一个整个的流程，此次终于走通了，特此记录一下。



# 三、Before Start

##  3.1 满脑子的疑问

1. TAF是什么?
2. TarsGo是什么?
3. Trpc是什么?
4. Jce是什么?
5. Jce Taf关系是什么?
6. 服务怎么部署？
7. 部署了怎么测试？
8. 一定要在开发机上开发，然后upload2test之后部署么？本地可否进行开发？
9. 本地可否运行，本地运行之后怎么测试？



## 3.2 整体步骤

　　大致步骤可以归纳如下：


1. 开发机申请
2. 开发机创建服务
3. 开发机关联本机，在Goland里面配置Deployment
4. Taf开发环境创建服务
5. 开发机里，进行make upload2test
6. 观察Taf开发环境是否部署成功
7. Taf开发环境接口测试创建用例，进行测试
8. 通过网络权限申请，或者繁琐的步骤，进行client测试
   

　　本文先按照这个流程重新梳理一遍，然后再附上在本地搭建、本地进行代码编写，本地运行，本地测试的方式。



# 四、开发机环境搭建

## 4.1 申请开发机

1. 登陆内部[Devcloud]，点击首页【申请云服务器】，申请开发机。

2. 连接登陆开发机。

   ![](/assets/img/2020/20201113_连接开发机.png){: .img-medium}

## 4.2 开发机环境

1. 登陆到开发机上，建立工作目录

   ```shell
   mkdir /root/go
   mkdir src bin
   cd src
   ```

   

2. 安装golang语言环境

   - 最新安装包下载 [goland下载](https://golang.org/dl/)，解压缩到/usr/local/。

     ```shell
     tar -C /usr/local -xzf go.XXX.tar.gz
     ```

   - 配置环境变量

     ```shell
     vim ~/.bashrc
     ```

   - 配置内容并退出

     ```shell
     export PATH=$PATH:/usr/local/go/bin
     export GOROOT=/usr/local/goexport GOPATH=/root/go
     ```

   - 导入系统环境

     ```shell
     source ~/.bashrc
     ```

   - 查看版本

     ```
     go version
     ```

   注：因为GOPATH目录配置的是/root/go，因此后续goland源码编译可以放置此目录进行编译。

   

3. 安装依赖包tarsgo

   - 腾讯已经将taggo开源为tarsgo了（下面是内部链接）

     ```shell
     cd /root/go/src
     git clone http://xxx.xxx.oa.com/tarsgo/tars.git
     ```

   - 安装 tars2go	

     ```shell
     cd /root/go/src/tars/tools/tars2gogo install
     ```

     

## 4.3 创建服务
## 4.3 创建服务

1. 首先想好你要创建的APP、SERVER 、SERVANT，利用`create_tars_server.sh`脚本创建服务。

   ```shell
   cd /root/go/src
   tars/tools/create_tars_server.sh EPTest EPTestServer EPTest
   ```

   **注意**：第三个参数Obj名称后**不要**加"Obj"。如：直接写EPTest就行，不要写EPTestObj。因为程序会自动在
   Servant名后加上Obj，如果自己写了Obj，就会变成EPTestObjObj。

- console里面打印如下：

  ```shell
  [root@VM_38_187_centos ~/go/src]# pwd
  /root/go/src
  [root@VM_38_187_centos ~/go/src]# tars/tools/create_tars_server.sh EPTest EPTestServer EPTest
  [create server: EPTest.EPTestServer ...]
  [mkdir: /root/go/src/EPTest/EPTestServer/]
  >>>Now doing:./start.bat >>>>
  >>>Now doing:./start.sh >>>>
  >>>Now doing:./Servant.jce >>>>
  >>>Now doing:./main.go >>>>
  >>>Now doing:./config.conf >>>>
  >>>Now doing:./Servant_imp.go >>>>
  >>>Now doing:./makefile >>>>
  >>>Now doing:./make_upload.bat >>>>
  >>>Now doing:client/client.go >>>>
  >>>Now doing:vendor/vendor.json >>>>
  >>>Now doing:debugtool/dumpstack.go >>>>
  >>> Great！Done! You can jump in /root/go/src/EPTest/EPTestServer
  >>> Tips: After editing the Tars file, execute the following cmd to automatically generate golang files.
  >>>       /root/go/bin/tars2go *.tars
  ```

2. 进入目录查看

   ```shell
   [root@VM_38_187_centos ~/go/src]# cd EPTest/EPTestServer/
   [root@VM_38_187_centos ~/go/src/EPTest/EPTestServer]# tree .
   .
   |-- EPTest.jce
   |-- client
   |   `-- client.go
   |-- config.conf
   |-- debugtool
   |   `-- dumpstack.go
   |-- eptest_imp.go
   |-- main.go
   |-- make_upload.bat
   |-- makefile
   |-- start.bat
   |-- start.sh
   `-- vendor
       `-- vendor.json
   
   3 directories, 11 files
   ```

- vendor是当前项目除 GOPATH 和 GOROOT 之外的依赖包的解决方案，在go 1.6 版本之后，vendor已经不需要再通过
配置环境变量了。具体查找依赖包的顺序为：`当前目录下vendor -> 上级目录直至 src目录下的vendor -> GOPATH下
查找 -> GOROOT目录下查找`。

- gomod模式，本次暂不涉及，注意在命令行`export GO111MODULE=off`避免干扰。


## 4.4 开发机和本地关联

1. 本机Goland新建项目
   转到本机的GoLand，新建一个同名项目，Go版本采用与开发容器一致的版本。
   ![](/assets/img/2020/20201113_新建Goland项目.png){: .img-medium}

   

2. Tools->Deployment->Configuration

   - 选择Configuration
     ![](/assets/img/2020/20201113_关联_选择Configuration.png){: .img-large}
   - 选择SFTP
     ![](/assets/img/2020/20201113_关联_选择SFTP.png){: .img-large}
   - 创建ssh configuration
     ![](/assets/img/2020/20201113_关联_创建ssh.png){: .img-large}
   - 填写Root Path和Web Server URL
     ![](/assets/img/2020/20201113_关联_填写Path.png){: .img-large}
   - 填写Mapping
     ![](/assets/img/2020/20201113_关联_填写Mapping.png){: .img-large}
   - 同步
     ![](/assets/img/2020/20201113_关联_同步.png){: .img-large}

   - 完成之后，即可在Goland里面看到工程结构



## 4.5 修改服务

1. Tools->Start SSH session
   ![](/assets/img/2020/20201113_修改_Start_SSH_Session.png){: .img-small}

   可以直接在Goland理进入到开发机里

2. 翻译jce文件，生成EPTest.tars.go文件

   ```shell
   [root@VM_38_187_centos ~]# cd go/src/EPTest/EPTestServer/
   [root@VM_38_187_centos ~/go/src/EPTest/EPTestServer]# ~/go/bin/tars2go EPTest.jce 
   EPTest.jce [EPTest.jce]
   [root@VM_38_187_centos ~/go/src/EPTest/EPTestServer]# tree .
   .
   |-- EPTest
   |   `-- EPTest.tars.go
   |-- EPTest.jce
   |-- client
   |   `-- client.go
   |-- config.conf
   |-- debugtool
   |   `-- dumpstack.go
   |-- eptest_imp.go
   |-- main.go
   |-- make_upload.bat
   |-- makefile
   |-- start.bat
   |-- start.sh
   `-- vendor
       `-- vendor.json
   ```

3. make之后，在vendor里面生成了EPTest.tars.go

   ```bash
   [root@VM_38_187_centos ~/go/src/EPTest/EPTestServer]# make
   cat: /tmp/container-info.env: No such file or directory
   /root/go/bin/tars2go  EPTest.jce ...
   /root/go/bin/tars2go  -outdir=vendor EPTest.jce
   EPTest.jce [EPTest.jce]
   /usr/local/go/bin/go build  -o EPTestServer
   [root@VM_38_187_centos ~/go/src/EPTest/EPTestServer]# tree .
   .
   ├── client
   │   └── client.go
   ├── config.conf
   ├── debugtool
   │   └── dumpstack.go
   ├── EPTest
   │   └── EPTest.tars.go
   ├── eptest_imp.go
   ├── EPTest.jce
   ├── EPTestServer
   ├── main.go
   ├── makefile
   ├── make_upload.bat
   ├── start.bat
   ├── start.sh
   └── vendor
       ├── EPTest
       │   └── EPTest.tars.go
       └── vendor.json
   
   5 directories, 14 files
   ```

4. 检查Obj是否写了两遍

   ```go
   // Register Servant
   app.AddServantWithContext(imp, cfg.App+"."+cfg.Server+".EPTestObj")
   ```

5. 修改代码逻辑

　　在eptest_imp.go里面：

   ```go
   func (imp *EPTestImp) Add(ctx context.Context, a int32, b int32, c *int32) (int32, error) {
   	//Doing something in your function
   	*c = a + b
   	return 0, nil
   }
   func (imp *EPTestImp) Sub(ctx context.Context, a int32, b int32, c *int32) (int32, error) {
   	//Doing something in your function
   	*c = a - b
   	return 0, nil
   }
   ```



# 五、代码仓库

1. 远程工蜂建仓，创建同名项目

2. 添加成员`svn_taf`，Reporter或者其以上的等级

3. 关联本地仓库

   ```shell
   [root@VM_38_187_centos ~/go/src/EPTest/EPTestServer]# git init
   [root@VM_38_187_centos ~/go/src/EPTest/EPTestServer]# git remote add origin http://xxx.xxx.oa.com/yukkizhang/EPTestServer.git
   [root@VM_38_187_centos ~/go/src/EPTest/EPTestServer]# git commit -m "[yukki] Init commit"
   [root@VM_38_187_centos ~/go/src/EPTest/EPTestServer]# git push -u origin master
   ```

   

# 六、TAF 开发环境

## 5.1 创建服务

1. TAF_开发环境->流程工具->TAF服务上线->Docker服务上线

   ![](/assets/img/2020/20201113_TAF_创建服务.png){: .img-large}



2. TAF业务信息
   ![](/assets/img/2020/20201113_TAF_业务信息.png)

3. Docker运行环境
   ![](/assets/img/2020/20201113_TAF_Docker运行环境.png)

4. 点击“提交”，提交后，自己可以进行审核，点击“通过”后，完成上线。

5. 查看服务详情



## 5.2 部署服务

1. 回到开发机窗口上传

   ```shell
   // 上线TAF开发环境
   make upload2test
   ```

　　注：如果上传失败，请检查代码的APP、SERVER 、SERVANT是否和TAF配置的一致。

2. 上传成功之后，可以看到ip地址和状态
   ![](/assets/img/2020/20201113_TAF_上传成功.png)



## 5.3 验证服务

### 1. TAF接口测试

- 添加测试接口
  ![](/assets/img/2020/20201113_TAF_添加接口测试1.png)

  ![](/assets/img/2020/20201113_TAF_添加接口测试2.png)

- 添加测试用例
  ![](/assets/img/2020/20201113_TAF_添加测试用例.png)

- 点击测试![](/assets/img/2020/20201113_TAF_点击测试.png)

- 测试结果
  ![](/assets/img/2020/20201113_TAF_测试结果.png){: .img-medium}



### 2. Client测试

- 查看服务端口号，可以选择一个节点，选择，查看Servant。
  ![](/assets/img/2020/20201113_TAF_查看Servant.png)



- 手动修改client代码，粘贴上面的ip和端口信息

  ```go
  	obj := fmt.Sprintf("EPTest.EPTestServer.EPTestObj@tcp -h 9.21.131.5 -p 10343 -t 60000")
  ```

- 开发机上

  ```
  [root@VM_38_187_centos ~/go/src/EPTest/EPTestServer]# cd client/
  [root@VM_38_187_centos ~/go/src/EPTest/EPTestServer/client]# go build
  [root@VM_38_187_centos ~/go/src/EPTest/EPTestServer/client]# ls
  client  client.go
  ```

- 开发机和部署服务的容器，网络不通，因此没办法直接运行`./client`，可以通过下面两种办法尝试：

  - 需要申请网络
  - 或者将./client文件上传到服务器的容器里，可以用`sz`和`rz`命令，然后再服务器上运行`./client`



# 六、本机开发运行

　　自此，前面第四、五部分是将3.2中的整体步骤进行了展开，讲述的是在开发机上完成创建服务、部署服务的过程。实际上，
也可以直接在本地安装环境，进行代码的编写。



## 6.1 本机环境搭建

　　笔者用的是Mac，暂时没切换到gomod：

- 安装golang环境

- 设置GOPATH GOROOT等

  

## 6.2 创建服务

　　只要6.1环境搭建好了，此步骤大致与在开发机里，完成的步骤一致。


## 6.3 运行

1. 直接点击main.go里面的运行小图标

   ```shell
   /usr/local/go/bin/go build -o /private/var/folders/t7/zh_h0h0d5vn0c2xsr_1tfqnh0000gn/T/___go_build_EPTestServer EPTestServer #gosetup
   can't load package: package EPTestServer is not in GOROOT (/usr/local/go/src/EPTestServer)
   ```

2. 命令行运行

   - 运行go build，报错

     ```shell
     $ go build              
     go: cannot find main module, but found vendor/vendor.json in /Users/yukki/Downloads/Go/src/EPTestServer
             to create a module there, run:
             go mod init
     ```

     

   - 上述错误信息提示跟go mod有关，我们先关掉go mod

     ```shell
     $ export GO111MODULE=off
     $ go build
     ```

   - 然后可以运行`EPTestServer`，报错

     ```shell
     $ ./EPTestServer 
     panic: runtime error: invalid memory address or nil pointer dereference
     [signal SIGSEGV: segmentation violation code=0x1 addr=0x18 pc=0x13dbb4e]
     
     goroutine 1 [running]:
     main.main()
             /Users/yukki/Downloads/Go/src/EPTestServer/main.go:26 +0x4e
     (base) 
     ```

     注：因为TarsGo底层是依赖一个基础的配置文件的。如果是运行在Tars平台上面的时候平台会根据服务配置和实际情况自行
     生成这个配置文件并加入到启动参数中，如果是本地的话就要自行填写。

   - 带上配置文件

     ```
     ./EPTestServer --config="./config.conf"
     ```

     

## 6.3 本机验证

　　上述服务是直接用`app.AddServantWithContext(imp, cfg.App+"."+cfg.Server+".EPTestObj")`注册的，
所以我们可以在client里面进行调用测试。

```go

func main() {
	comm := tars.NewCommunicator()
	obj := fmt.Sprintf("EPTest.EPTestServer.EPTestObj@tcp -h 127.0.0.1 -p 10015 -t 60000")
	app := new(EPTest.EPTest)
	comm.StringToProxy(obj, app)
	var out, i int32
	i = 123
	ret, err := app.Add(i, i*2, &out)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(ret, out)
}
```

　　类似的，再开启一个console：

```
$ cd client 
$ export GO111MODULE=off
$ go build 
$ ./client --config="../config.conf"
0 369
```



## 6.4 修改为HTTP服务

1. 修改main.go，将原来的EPTestImp改为Http路由


   ```go
   func main() {
   	// Get server config
   	cfg := tars.GetServerConfig()
   
   	// New servant imp，这里改为一个TarsHttpMux
   	app := &tars.TarsHttpMux{}
   	app.HandleFunc("/", HttpRootHandler)
   
   
   	// Register Servant，这里将vantWithContex改为AddHttpServant
   	tars.AddHttpServant(app, cfg.App+"."+cfg.Server+".EPTestObj")
   	
   	// Run application
   	tars.Run()
   }
   
   // 增加了一个HttpRootHandler路由处理
   func HttpRootHandler(w http.ResponseWriter, r *http.Request) {
   	time_fmt := "2006-01-02 15:04:05"
   	local_time := time.Now().Local()
   	time_str := local_time.Format(time_fmt)
   	ret_str := fmt.Sprintf("{\"msg\":\"Hello, Tars-Go!\", \"time\":\"%s\"}", time_str)
   
   	w.Header().Set("Content-Type", "application/json;charset=utf-8")
   	w.Write([]byte(ret_str))
   	return
   }
   ```



2. 重新go build，并运行

   

3. 浏览器访问config里面端口
   ![](/assets/img/2020/20201113_TAF_HTTP验证结果.png){: .img-medium}

## 6.5 Trpc服务

　　除了上述两种方法以外，还可以通过腾讯提供的tRPC-Go框架，以`trpc.NewServer()`提供RPC方法调用，搭建方法略去。

　　启动服务的时候，用`trpc-cli`发送请求：

```
trpc-cli -target=ip://127.0.0.1:8000 -func /trpc.yukki.trpc_helloworld.Hello/SayHelloTest -body '{"msg": "hello, yukki"}'
```



# 七、回到开始的疑问

## 1. TAF是什么?

　　TAF（Total Application Framework），在外部叫做Tars, 是腾讯内部的一套分布式微服务框架及其相关配套设施的总称，
各大核心业务都在使用，颇受欢迎，基于该框架部署运行的服务节点规模达到上万个。详细资料可以参考 
[GitHub介绍](https://github.com/TarsCloud/Tars/blob/master/README.zh.md)

> 目前支持C++、Java、PHP、Nodejs、Go语言。该框架为用户提供了涉及到开发、运维、以及测试的一整套
>解决方案，帮助一个产品或者服务快速开发、部署、测试、上线。 它集可扩展协议编解码、高性能RPC通信框架、
>名字路由与发现、发布监控、日志统计、配置管理等于一体，通过它可以快速用微服务的方式构建自己的稳定可靠的
>分布式应用，并实现完整有效的服务治理。



基础概念：

- APP：APP（应用）的概念经常出现在各类Tars有关的文档和配置平台中，其实就是一个划分。一般一个业务或
一个部门的服务会被分配到一个APP名，按需选择和填写即可。

- Server：服务就是在Tars运维平台上用来查找服务的名称，同一份代码也可以部署成多个服务。同一个服务的
同一个Set拥有相同的配置

- Obj：一个服务可以对外提供多个接口（接口是方法的集合），一个接口对应一个OBJ。`APP.Server.OBJ`
三个组成Tars RPC时的被调名称。

- Communicator：通讯器，用于生成远程被调服务的本地代理对象。通讯器本身持有RPC相关的配置参数，有调整时
一般是修改通讯器配置.

- JCE：Tars RPC使用的IDL文件格式。部分语境下也可能表示JCE的序列化实现。

- Imp:：通过JCE知识定义了接口，具体的实现在大部分编程语言中都会通过一个类对象来实现，这个对象常被称作Imp对象.

  

## 2. TarsGo是什么？

这里所说的 `TarsGo` ，狭义来说指的是使用GoLang实现支持Tars的一套代码库。


这个库目前主要提供以下功能：

- Tars RPC所使用的IDL格式是JCE（类似于GRPC中的Protobuf）, 扩展名是`.tars`或`.jce`(只是扩展名
不同，实质无差别)。TarsGo提供了将JCE转换成GoLang代码的工具, 并提供JCE了序列化和反序列化的能力库
- 实现了GoLang版本的Tars RPC, 支持调用其他的Tars服务(可以是CPP或者Java等其他语言实现的), 和
被作为Tars服务被其他服务所调用。

- 实现了对接Tars平台管理功能的相关能力，如：服务心跳上报、自定义特性上报、自定义命令处理、服务间调用的基础监控上报。

- 提供了符合Tars格式的基础日志支持, 如：固定大小滚动日志、按天日志, 也只Tars平台的四个日志级别。

- 提供了服务间染色的能力，也支持基础的染色日志收集(但没有实现对接染色管理平台, 有需要可以使用 qbmetis)。

- 提供了基于CGO的Tars远程日志支持 (如果希望使用纯GoLang实现可以使用 tarsark)。

- 提供了获取和解析Tars配置文件的能力 (解析实现与CPP版本有所差异，有需要的时候可以尝试 tarsconfig)

- 提供了构建和脚手架的脚本


**说明** : 可以将TarsGo看作是GoLang版的Tars SDK，以代码实现的形式为应用服务提供Tars支持，需要运行
在Tars架构内和使用Tars有关特性的服务需要引入的一个第三方库。而像公司内部taf.wsd.com 这类平台化的管理
页面则是语言无关的（只要是符合Tars协议规范的程序都可以使用），是配套的运维开发平台。



## 3. 其他问题

　　关于部署、开发、验证的问题，在本文的第三、四、五找中都可以找到答案。



## 4. 下一步

　　本文记录了按照vendor依赖包管理的模式，进行了HelloWorld版本的搭建部署，那么下一步需要熟悉下如何按照现有的
项目的模式，写RPC服务，写HTTP服务，以及TAF结构框架的理解，数据流的走向。



# 八、参考

1. [腾讯 Tars-Go 后台开发框架](https://cloud.tencent.com/developer/inventory/2481)
2. [TAF 必修课（一）：整体架构理解](https://cloud.tencent.com/developer/article/1005833)
