---
layout: post
title: "创建你自己的Web Server - part 1"
date: 2016-03-05 14:58:00
categories: java
tags: 
- java
- web server 
---
>　　一天，女士外出散步，经过施工现场看到三个工人正在干活。  
　　女士上前询问第一个工人："你在干什么？"，被问题惹恼的第一个工人咆哮道："难道你没看见我正在铺砖吗？"。  
　　女士不满意刚才的答案，于是问第二个人他正在干什么。"我正在建造一堵墙。"第二个工人说着，便把注意力转向第一个工人说道："喂，你刚刚超过了墙的末尾， 你需要把最后一块砖拿走。"  
　　女士对第二个人的回答仍然不满意，于是她问第三个人正在干什么。第三个人看着天空回答她："我正在建造世界上最大的教堂"。  
　　正当第三个人站在那里仰望天空的时候，前两个人开始争论那块错误的砖块。于是第三个工人转向前两个人："你们俩不用担心那块砖，这只是内部墙，它会被砖块砌满，没人见过那块砖。只需要把它挪到其它层就可以了。"</br>  
　　这个故事的寓意是：当你知道了整个系统并且理解不同的部分是如何组合在一起（砖，墙，教堂），你就可以更快的识别并修复问题（错误的砖块）。  

　　这与从头开始创建自己的web server有什么关系呢？
　　我相信要想成为一名更优秀的程序员，你必须更好的了解底层的软件系统，也就是日常使用的基础并且包括编程语言、编译器与中断、数据库与操作系统，web服务器与web框架。为了更好更深入的理解这些系统，你必须从头一点一点的开始重新构建它们。

>孔夫子云：
"吾听吾忘"
![](/assets/img/2016/WS_confucius_hear.png){: .img-medium}
"吾见吾记"
![](/assets/img/2016/WS_confucius_see.png){: .img-medium}
"吾做吾悟"
![](/assets/img/2016/WS_confucius_do.png){: .img-medium}
　　我希望在这一点上你们可以确信，从头开始构建不同的软件系统来学习它们如何运行是一个不错的方法。
　　在接下来的三部分的系列中，我会展示如何构建你自己基本的web服务器。让我们开始吧！

### 首先，什么是web server呢？
![](/assets/img/2016/WS_HTTP_request_response.png){: .img-medium}
　　简而言之，web server是一个坐落在物理服务器上的网络服务器（啊，在服务器上的服务器），并且等待客户端发送请求。当它收到请求时，它就生成一个响应，并将其发送回客户端。客户端和服务器之间的通信使用HTTP协议。一个客户端可以是你的浏览器或者任何其他支持HTTP的软件。  
　　那么web server非常简单的实现会是什么样子？下面是我的理解。示例是用Python，即使你不会Python（它是一种非常简单的语言，试一试！），你应该能够通过代码和下面的解释理解内容。  
```python
import socket

HOST, PORT = '', 8888

listen_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
listen_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
listen_socket.bind((HOST, PORT))
listen_socket.listen(1)
print 'Serving HTTP on port %s ...' % PORT
while True:
    client_connection, client_address = listen_socket.accept()
    request = client_connection.recv(1024)
    print request

    http_response = """\
HTTP/1.1 200 OK

Hello, World!
"""
    client_connection.sendall(http_response)
    client_connection.close()
```
　　保存上面的代码到`webserver.1py`文件或者在[GitHub](https://github.com/rspivak/lsbaws/blob/master/part1/webserver1.py)上直接下载，然后用下面的命令运行：
```
$ python webserver1.py
Serving HTTP on port 8888 …
```
　　然后在浏览器地址栏输入`http://localhost:8888/hello`，然后回车，见证奇迹的时刻到来了。你应该可以看到`Hello World!`显示在浏览器中，如下所示：
![](/assets/img/2016/WS_browser_hello_world.png)

　　试着做一下吧，你在测试时候我会等你的。做完了吗？好极了。现在我们一起讨论它是如何生效的吧。首先让我们从你输入的网址开始，它叫做[URL](http://baike.baidu.com/link?url=cigkFmQSCQHgSk9SfSVZH7palffbNp6nHiV0WjRy4LkAFH02SIDw-htgvq-wBlMDAORPltXx1i-xcqNpLeGxla),下面是它的基本结构：
![](/assets/img/2016/WS_URL_Web_address.png){: .img-medium}

　　这就是你如何告诉浏览器，它需要找到并且连接的web server地址，同时获取服务器页面的方式。在你的浏览器发送一个HTTP请求，它首先需要和Web server建立一个TCP连接。然后它基于TCP连接给服务器发送一个HTTP请求等待服务器发送一个HTTP的响应回来。当收到响应之后，它会显示响应信息`Hello World!`。
　　
### TCP连接
　　让我们更详细的探讨如何在发送HTTP请求和响应之前，建立客户端和服务器TCP连接。要做到这一点，它们都使用所谓的sockets。如果不直接使用浏览器，可以通过手动在命令行使用telnet来模拟浏览器。在运行web server的同一台计算机上，通过命令行激活一个telnet会话指定连接主机为localhost，连接端口为8888，然后回车：
```
$ telnet localhost 8888
Trying 127.0.0.1 …
Connected to localhost.
```
　　这样，你就和服务器建立了一个TCP连接，这个服务器运行在本机，并且准备发送和接收HTTP消息。通过下面的图片，你可以看到一个服务器要经过的能够接收TCP连接的标准步骤。
![](/assets/img/2016/WS_socket.png){: .img-medium}

　　在同一个telnet会话里，输入命令`GET /Hello HTTP/1.1`然后回车：
```
$ telnet localhost 8888
Trying 127.0.0.1 …
Connected to localhost.
GET /hello HTTP/1.1

HTTP/1.1 200 OK
Hello, World!
```

### HTTP请求
　　刚才，你手动的模拟了浏览器，发送了一个HTTP请求并且收到了HTTP响应，下面是一个HTTP请求的基本结构：
![](/assets/img/2016/WS_HTTP_request_anatomy.png){: .img-medium}

　　Http请求包含：请求行，以表明HTTP方法（GET，因为我们请求服务器返回给我们一些信息）；路径/hello，以表明我们想要的服务器页面；以及协议版本。  
　　为了简单起见，我们的web server在这一点上完全无视上述请求行，你一样可以输入其他任何东西代替`GET /Hello HTTP/1.1`,也会得到"Hello, World!"的响应。一旦你输入了请求行并按下回车，客户端会发送请求给服务器，服务器解析请求行，并打印，然后返回适当的HTTP响应。
　　
### HTTP响应
　　下面是服务器返回给客户端的HTTP响应（以telnet为例）：
![](/assets/img/2016/WS_HTTP_response_anatomy.png){: .img-medium}

　　让我们深入剖析它，响应包含：状态行`HTTP/1.1 200 OK`，紧接着是需要的空行，然后就是HTTP的响应正文。响应状态行`HTTP/1.1 200 OK`包含HTTP的版本，HTTP的状态码（status code），以及对状态码的简短描述`OK`(reason-phrase)。当浏览器得到响应，会把响应正文展示出来，这就是你为什么在浏览器看到`Hello World!`的原因。

### 总结　　
　　上述是web server如何运行的基本模型。总而言之，web server创建一个监听socket并且开始循环的接收新的连接。客户端初始化一个TCP连接，当连接成功建立之后，客户端发送HTTP请求给服务器，服务器返回用于展示给用户的HTTP响应。要建立一个TCP响应，客户端和服务器都要使用sockets。  
　　现在你有了一个非常基本的web server了， 你可以用浏览器和一些其他的HTTP客户端测试它。正如你所看到和你所希望的一样，你也可以通过telnet手动输入HTTP请求成为人工HTTP客户端。  
　　下面提一个问题：你如何用你刚写出来的web server去运行一个Django应用，Flask应用和Pyramid应用，同时不对这个服务器做出单一改变来容纳所有这些不同的Web框架呢？在接下来的第二部分我会展示如何实现，敬请关注。

>本文翻译自Ruslan's Blog [Let’s Build A Web Server. Part 1.](https://ruslanspivak.com/lsbaws-part1/)
