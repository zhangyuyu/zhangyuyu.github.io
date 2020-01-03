---
layout: post
title: "Mongo-记一次安装启动异常"
date: 2017-12-27 11:35:21
categories: db
tags:
- db
- mongo
---
## 一、背景
　　笔者最近换了电脑，从MacBook Pro 13"换成了MacBook Pro 15"，相关的工具软件全部得重新装。
再重新装Mongo的时候遇到了一些问题，特此记录。

## 二、安装异常

### 1. 错误描述
<!-- more -->
　　用`brew install mongodb`安装了最新的mongodb(3.6)之后，启动本地的java spring boot应用，报错如下：

```
com.mongodb.MongoCommandException: Command failed with error 9: 'The 'cursor' option is required, except for aggregate with the explain argument' on server localhost:27017. The full response is { "ok" : 0.0, "errmsg" : "The 'cursor' option is required, except for aggregate with the explain argument", "code" : 9, "codeName" : "FailedToParse" }

        at com.mongodb.CommandResult.getException(CommandResult.java:80)

        at com.mongodb.CommandResult.throwOnError(CommandResult.java:94)

        at org.springframework.data.mongodb.core.MongoTemplate.handleCommandError(MongoTemplate.java:2097)

        ... 119 common frames omitted
```

### 2. 错误原因

　　MongoDB 3.6的Document，对于aggregation cursor的描述如下:
>	
	Specify a document that contains options that control the creation of the cursor object.
>
	Changed in version 3.6: MongoDB 3.6 removes the use of aggregate command without the cursor
	option unless the command includes the explain option. Unless you include the explain option,
	you must specify the cursor option.
>
	To indicate a cursor with the default batch size, specify cursor: {}.
>
	To indicate a cursor with a non-default batch size, use cursor: { batchSize: <num> }.

　　MongoDB在3.6里面改变了aggregation指令的工作方式，现在aggregation需要cursor了。

### 3. 解决办法
　　对本地的mongodb降级。

#### 1）先用brew search查看支持的mongo版本

```
$ brew search mongo
==> Searching local taps...
mongodb@3.4 ✔                          mongo-cxx-driver                       mongodb                                mongodb@3.2                            percona-server-mongodb
mongo-c-driver                         mongo-orchestration                    mongodb@3.0                            mongoose
==> Searching taps on GitHub...
caskroom/cask/mongo-management-studio  homebrew/php/php53-mongo               homebrew/php/php54-mongo               homebrew/php/php55-mongo               homebrew/php/php56-mongo
==> Searching blacklisted, migrated and deleted formulae...
```

#### 2）再brew安装mongodb 3.4
`brew install mongodb@3.4`


## 三、启动异常

### 1. 错误描述
　　安装完成mongodb3.4之后，mongo服务启动异常。

```
$ mongo
MongoDB shell version v3.4.10
connecting to: mongodb://127.0.0.1:27017
2017-12-27T11:13:37.713+0800 W NETWORK  [thread1] Failed to connect to 127.0.0.1:27017, in(checking socket for error after poll), reason: Connection refused
2017-12-27T11:13:37.715+0800 E QUERY    [thread1] Error: couldn't connect to server 127.0.0.1:27017, connection attempt failed :
connect@src/mongo/shell/mongo.js:237:13
@(connect):1:6
exception: connect failed
```

### 2. 错误原因
　　mongodb的服务没有启动。

### 3. 解决办法

#### 探索过程：
##### 1）用brew services启动mongo不work

```
$ brew services start mongodb@3.4
==> Successfully started `mongodb@3.4` (label: homebrew.mxcl.mongodb@3.4)
$ brew services list
Name        Status  User    Plist
mongodb@3.4 started yuzhang /Users/yuzhang/Library/LaunchAgents/homebrew.mxcl.mongodb@3.4.plist
```

　　上述操作之后，似乎mongo已经启动了，但是问题依旧复现，可能是brew services不再有人维护的原因（[brew services is unsupported and will be removed soon](https://github.com/Homebrew/legacy-homebrew/issues/32006)）。

##### 2）用mongod启动mongo报错

```
$ mongod
2017-12-27T11:11:28.980+0800 I CONTROL  [initandlisten] MongoDB starting : pid=8655 port=27017 dbpath=/data/db 64-bit host=CNyuzhang-2.local
2017-12-27T11:11:28.981+0800 I CONTROL  [initandlisten] db version v3.4.10
2017-12-27T11:11:28.981+0800 I CONTROL  [initandlisten] git version: 078f28920cb24de0dd479b5ea6c66c644f6326e9
2017-12-27T11:11:28.981+0800 I CONTROL  [initandlisten] OpenSSL version: OpenSSL 1.0.2n  7 Dec 2017
2017-12-27T11:11:28.981+0800 I CONTROL  [initandlisten] allocator: system
2017-12-27T11:11:28.981+0800 I CONTROL  [initandlisten] modules: none
2017-12-27T11:11:28.981+0800 I CONTROL  [initandlisten] build environment:
2017-12-27T11:11:28.981+0800 I CONTROL  [initandlisten]     distarch: x86_64
2017-12-27T11:11:28.981+0800 I CONTROL  [initandlisten]     target_arch: x86_64
2017-12-27T11:11:28.981+0800 I CONTROL  [initandlisten] options: {}
2017-12-27T11:11:28.981+0800 I STORAGE  [initandlisten] exception in initAndListen: 29 Data directory /data/db not found., terminating
2017-12-27T11:11:28.981+0800 I NETWORK  [initandlisten] shutdown: going to close listening sockets...
2017-12-27T11:11:28.981+0800 I NETWORK  [initandlisten] shutdown: going to flush diaglog...
2017-12-27T11:11:28.981+0800 I CONTROL  [initandlisten] now exiting
2017-12-27T11:11:28.981+0800 I CONTROL  [initandlisten] shutting down with code:100
```

　　上述错误信息里，明确指出`/data/db not found`，因此是在启动的时候没有找到db的地址。为了确认是这个问题，尝试使用如下指令：

```
mongo -nodb
```
　　然后，发现可以直接进入mongo了。

##### 3）创建/data/db

```
$mkdir -p /data/db
```

　　再次启动mongodb:

```
2017-12-27T11:10:02.679+0800 I CONTROL  [initandlisten] MongoDB starting : pid=8510 port=27017 dbpath=/data/db 64-bit host=CNyuzhang-2.local
2017-12-27T11:10:02.680+0800 I CONTROL  [initandlisten] db version v3.4.10
2017-12-27T11:10:02.680+0800 I CONTROL  [initandlisten] git version: 078f28920cb24de0dd479b5ea6c66c644f6326e9
2017-12-27T11:10:02.680+0800 I CONTROL  [initandlisten] OpenSSL version: OpenSSL 1.0.2n  7 Dec 2017
2017-12-27T11:10:02.680+0800 I CONTROL  [initandlisten] allocator: system
2017-12-27T11:10:02.680+0800 I CONTROL  [initandlisten] modules: none
2017-12-27T11:10:02.680+0800 I CONTROL  [initandlisten] build environment:
2017-12-27T11:10:02.680+0800 I CONTROL  [initandlisten]     distarch: x86_64
2017-12-27T11:10:02.680+0800 I CONTROL  [initandlisten]     target_arch: x86_64
2017-12-27T11:10:02.680+0800 I CONTROL  [initandlisten] options: {}
2017-12-27T11:10:02.681+0800 I STORAGE  [initandlisten] exception in initAndListen: 20 Attempted to create a lock file on a read-only directory: /data/db, terminating
2017-12-27T11:10:02.681+0800 I NETWORK  [initandlisten] shutdown: going to close listening sockets...
2017-12-27T11:10:02.681+0800 I NETWORK  [initandlisten] shutdown: going to flush diaglog...
2017-12-27T11:10:02.681+0800 I CONTROL  [initandlisten] now exiting
2017-12-27T11:10:02.681+0800 I CONTROL  [initandlisten] shutting down with code:100
```

　　可以看出是对`/data/db`没有权限。

##### 4）用sudo执行mongod

```
$ sudo mongod
2017-12-27T11:10:08.167+0800 I CONTROL  [initandlisten] MongoDB starting : pid=8521 port=27017 dbpath=/data/db 64-bit host=CNyuzhang-2.local
2017-12-27T11:10:08.167+0800 I CONTROL  [initandlisten] db version v3.4.10
2017-12-27T11:10:08.167+0800 I CONTROL  [initandlisten] git version: 078f28920cb24de0dd479b5ea6c66c644f6326e9
2017-12-27T11:10:08.167+0800 I CONTROL  [initandlisten] OpenSSL version: OpenSSL 1.0.2n  7 Dec 2017
2017-12-27T11:10:08.167+0800 I CONTROL  [initandlisten] allocator: system
2017-12-27T11:10:08.167+0800 I CONTROL  [initandlisten] modules: none
2017-12-27T11:10:08.167+0800 I CONTROL  [initandlisten] build environment:
2017-12-27T11:10:08.167+0800 I CONTROL  [initandlisten]     distarch: x86_64
2017-12-27T11:10:08.167+0800 I CONTROL  [initandlisten]     target_arch: x86_64
2017-12-27T11:10:08.167+0800 I CONTROL  [initandlisten] options: {}
2017-12-27T11:10:08.170+0800 I -        [initandlisten] Detected data files in /data/db created by the 'wiredTiger' storage engine, so setting the active storage engine to 'wiredTiger'.
2017-12-27T11:10:08.170+0800 I STORAGE  [initandlisten] wiredtiger_open config: create,cache_size=7680M,session_max=20000,eviction=(threads_min=4,threads_max=4),config_base=false,statistics=(fast),log=(enabled=true,archive=true,path=journal,compressor=snappy),file_manager=(close_idle_time=100000),checkpoint=(wait=60,log_size=2GB),statistics_log=(wait=0),
2017-12-27T11:10:08.781+0800 I CONTROL  [initandlisten]
2017-12-27T11:10:08.781+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2017-12-27T11:10:08.781+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2017-12-27T11:10:08.781+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2017-12-27T11:10:08.781+0800 I CONTROL  [initandlisten]
2017-12-27T11:10:08.822+0800 I FTDC     [initandlisten] Initializing full-time diagnostic data capture with directory '/data/db/diagnostic.data'
2017-12-27T11:10:08.823+0800 I NETWORK  [thread1] waiting for connections on port 27017
2017-12-27T11:10:11.182+0800 I NETWORK  [thread1] connection accepted from 127.0.0.1:52959 #1 (1 connection now open)
2017-12-27T11:10:11.182+0800 I NETWORK  [thread1] connection accepted from 127.0.0.1:52960 #2 (2 connections now open)
2017-12-27T11:10:11.182+0800 I NETWORK  [thread1] connection accepted from 127.0.0.1:52961 #3 (3 connections now open)

```

#### 解决方法小结
1. 创建`/data/db`，因为mongodb默认的db地址是`/data/db`
2. 用sudo执行mongod

```
mkdir -p /data/db
sudo mongod
```

## 四、总结
1. 问题1是新版本3.6的mongodb做了一些改动造成的不兼容
2. 问题2是brew services不再维护造成的启动异常

## References
* [Stackoverflow - Mongo connection refused osx](https://stackoverflow.com/questions/29630399/mongo-connection-refused-osx)
* [Stackoverflow - Cannot connect to MongoDB errno:61](https://stackoverflow.com/questions/23439343/cannot-connect-to-mongodb-errno61)
* [Mac下使用brew安装mongodb](http://hcysun.me/2015/11/21/Mac下使用brew安装mongodb/)
