---
layout: post
title: "创建你自己的Web Server - part 2"
date: 2016-03-06 17:43:32
categories: java
tags: 
- java
- web server 
---
　　还记得吗，在第一部分我问了一个问题：如何在你刚写出来的web server上运行一个Django应用，Flask应用和Pyramid应用，同时不做出改动就能适应这些不同的Web框架呢？往下读就可以找到答案。  

　　在以前，你选择的Python web框架会限制你可选择的web server，反之亦然。如果框架和服务器被设计成协同工作的话，那就是极好的。
![](/assets/img/2016/WS_part2_before_wsgi.png){: .img-medium}

　　但是当你试图去连接没有被设计成协同工作的服务器和框架时， 你可能会遇到（可能你遇到过）下面的问题：
![](/assets/img/2016/WS_part2_after_wsgi.png){: .img-medium}

　　基本上，你不得不使用能协同工作的组件，而不是你想使用的组件。因此，你如何确保你的web server能够运行多种web框架，而不用改变web server和web框架的代码呢？问题的答案就是*Python Web Server GateWay Interface*(或简称WSHI，读作"wizgy")。
![](/assets/img/2016/WS_part2_wsgi_idea.png){: .img-medium}

　　WSGI允许开发者自由选择web server和web框架。现在你可以混合搭配不同的web server和web框架，并选择一个满足你需求的组合。比如，你可以用Gunicorn，Nginx/uWSGI或者Waitress运行Django，Flask或者Pyramid。真正的混合搭配，多亏了服务器和框架对WSGI的支持。  
![](/assets/img/2016/WS_part2_wsgi_interop.png){: .img-medium}

　　因此，WSGI是我在第一部分提出，并在本文开头重复的问题的答案。你的web server必须实现WSGI接口的服务器端部分，所有的Python web框架已经实现了WSGI接口的框架端部分。这样不用修改服务器的代码去适应指定的web框架，你就能使用你的web server。  
　　现在你已经知道web server和web框架都支持WSGI，它允许你选择适合的组合，同时也有利与服务器和框架的开发者专注于他们擅长的领域，不会因为越界而踩到对方的脚趾。其他语言也有类似的接口：比如Java有Servlet API，Ruby有Rack。

　　一切都很好，但是我猜你会说："把代码展示给我看！"好吧，一起看看这个简单的WSGI服务器的实现吧：
```python

# Tested with Python 2.7.9, Linux & Mac OS X
import socket
import StringIO
import sys


class WSGIServer(object):

    address_family = socket.AF_INET
    socket_type = socket.SOCK_STREAM
    request_queue_size = 1

    def __init__(self, server_address):
        # Create a listening socket
        self.listen_socket = listen_socket = socket.socket(
            self.address_family,
            self.socket_type
        )
        # Allow to reuse the same address
        listen_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        # Bind
        listen_socket.bind(server_address)
        # Activate
        listen_socket.listen(self.request_queue_size)
        # Get server host name and port
        host, port = self.listen_socket.getsockname()[:2]
        self.server_name = socket.getfqdn(host)
        self.server_port = port
        # Return headers set by Web framework/Web application
        self.headers_set = []

    def set_app(self, application):
        self.application = application

    def serve_forever(self):
        listen_socket = self.listen_socket
        while True:
            # New client connection
            self.client_connection, client_address = listen_socket.accept()
            # Handle one request and close the client connection. Then
            # loop over to wait for another client connection
            self.handle_one_request()

    def handle_one_request(self):
        self.request_data = request_data = self.client_connection.recv(1024)
        # Print formatted request data a la 'curl -v'
        print(''.join(
            '< {line}\n'.format(line=line)
            for line in request_data.splitlines()
        ))

        self.parse_request(request_data)

        # Construct environment dictionary using request data
        env = self.get_environ()

        # It's time to call our application callable and get
        # back a result that will become HTTP response body
        result = self.application(env, self.start_response)

        # Construct a response and send it back to the client
        self.finish_response(result)

    def parse_request(self, text):
        request_line = text.splitlines()[0]
        request_line = request_line.rstrip('\r\n')
        # Break down the request line into components
        (self.request_method,  # GET
         self.path,            # /hello
         self.request_version  # HTTP/1.1
         ) = request_line.split()

    def get_environ(self):
        env = {}
        # The following code snippet does not follow PEP8 conventions
        # but it's formatted the way it is for demonstration purposes
        # to emphasize the required variables and their values
        #
        # Required WSGI variables
        env['wsgi.version']      = (1, 0)
        env['wsgi.url_scheme']   = 'http'
        env['wsgi.input']        = StringIO.StringIO(self.request_data)
        env['wsgi.errors']       = sys.stderr
        env['wsgi.multithread']  = False
        env['wsgi.multiprocess'] = False
        env['wsgi.run_once']     = False
        # Required CGI variables
        env['REQUEST_METHOD']    = self.request_method    # GET
        env['PATH_INFO']         = self.path              # /hello
        env['SERVER_NAME']       = self.server_name       # localhost
        env['SERVER_PORT']       = str(self.server_port)  # 8888
        return env

    def start_response(self, status, response_headers, exc_info=None):
        # Add necessary server headers
        server_headers = [
            ('Date', 'Tue, 31 Mar 2015 12:54:48 GMT'),
            ('Server', 'WSGIServer 0.2'),
        ]
        self.headers_set = [status, response_headers + server_headers]
        # To adhere to WSGI specification the start_response must return
        # a 'write' callable. We simplicity's sake we'll ignore that detail
        # for now.
        # return self.finish_response

    def finish_response(self, result):
        try:
            status, response_headers = self.headers_set
            response = 'HTTP/1.1 {status}\r\n'.format(status=status)
            for header in response_headers:
                response += '{0}: {1}\r\n'.format(*header)
            response += '\r\n'
            for data in result:
                response += data
            # Print formatted response data a la 'curl -v'
            print(''.join(
                '> {line}\n'.format(line=line)
                for line in response.splitlines()
            ))
            self.client_connection.sendall(response)
        finally:
            self.client_connection.close()


SERVER_ADDRESS = (HOST, PORT) = '', 8888


def make_server(server_address, application):
    server = WSGIServer(server_address)
    server.set_app(application)
    return server


if __name__ == '__main__':
    if len(sys.argv) < 2:
        sys.exit('Provide a WSGI application object as module:callable')
    app_path = sys.argv[1]
    module, application = app_path.split(':')
    module = __import__(module)
    application = getattr(module, application)
    httpd = make_server(SERVER_ADDRESS, application)
    print('WSGIServer: Serving HTTP on port {port} ...\n'.format(port=PORT))
    httpd.serve_forever()
```

　　上面的代码确实比第一部分服务器的代码长，但是为了让你能够理解而不至于陷入细节的泥潭中，它已经足够短了（不到150行）。上面的服务器也能做更多——它能运行用你喜欢的框架（Pyramid, flask, Django或者其他Python WSGI框架）编写的基本web应用程序。
　　不信？动手试一下吧。保存上面的代码到`webserver2.py`或者直接从[GitHub](https://github.com/rspivak/lsbaws/blob/master/part2/webserver2.py)上下载。如果你试图运行时不带任何参数，它会提示并退出。
```
$ python webserver2.py
Provide a WSGI application object as module:callable
```

　　它真的想服务你的web应用程序，这就是有趣的开始。运行这个服务器你唯一要做的就是安装Python。但是要运行用Pyramid, flask或Django开发的应用，你需要先安装这些框架。让我们来安装这三个框架吧，我更喜欢使用virtualenv，只需要按照下面的步骤去创建和激活一个虚拟环境，就可以安装这三个框架了。
```
$ [sudo] pip install virtualenv
$ mkdir ~/envs
$ virtualenv ~/envs/lsbaws/
$ cd ~/envs/lsbaws/
$ ls
bin  include  lib
$ source bin/activate
(lsbaws) $ pip install pyramid
(lsbaws) $ pip install flask
(lsbaws) $ pip install django
```

### Pyramid框架
　　这时，你需创建一个Web应用程序。我们首先从Pyramid开始吧，保存下面的代码到`pyramidapp.py`并放到与`webserver2.py`的相同路径下，或者直接从[GitHub](https://github.com/rspivak/lsbaws/blob/master/part2/pyramidapp.py)上下载。
```python
from pyramid.config import Configurator
from pyramid.response import Response


def hello_world(request):
    return Response(
        'Hello world from Pyramid!\n',
        content_type='text/plain',
    )

config = Configurator()
config.add_route('hello', '/hello')
config.add_view(hello_world, route_name='hello')
app = config.make_wsgi_app()
```
　　现在，你可以准备用你自己的web server服务你的Pyramid应用了：
```
(lsbaws) $ python webserver2.py pyramidapp:app
WSGIServer: Serving HTTP on port 8888 …
```

　　刚才你告诉你的服务器去从python模块`pyramidapp`中加载一个可调用的app对象。现在你的服务已经准备好接收请求，转发请求至你的Pyramid应用程序。当前应用程序只处理一个路由：`\hello路由`。在浏览器地址栏输入`http://localhost:8888/hello`，回车观察结果：
![](/assets/img/2016/WS_part2_browser_pyramid.png){: .img-medium}

　　你也可以在命令行用`curl` 指令来测试服务器：
```
$ curl -v http://localhost:8888/hello
…
```
　　检查服务器和`curl`打印出的标准输出。

### Flask框架
　　现在轮到Flask了，让我们按照相同步骤来操作。
```python
from flask import Flask
from flask import Response
flask_app = Flask('flaskapp')


@flask_app.route('/hello')
def hello_world():
    return Response(
        'Hello world from Flask!\n',
        mimetype='text/plain'
    )

app = flask_app.wsgi_app
```
	
　　保存上面的代码到`flaskapp.py`，或者直接从[GitHub](https://github.com/rspivak/lsbaws/blob/master/part2/flaskapp.py)上下载，运行服务器：
```
(lsbaws) $ python webserver2.py flaskapp:app
WSGIServer: Serving HTTP on port 8888 …
```

　　在浏览器输入`http://localhost:8888/hello`，回车：
![](/assets/img/2016/WS_part2_browser_flask.png)

　　再次用`curl`命令，看一下服务器返回Flask应用程序生成的信息：
```
$ curl -v http://localhost:8888/hello
…
```
### Django框架
　　服务器也能处理Django应用吗？试一试！这一次涉及的内容有点复杂，我建议你克隆这个仓库，然后使用GitHUb仓库中的`djangoapp.py`文件。下面的源码主要是添加Django`Helloworld`工程（预先使用Django的`diango-admin.py`startproject命令）到当前Python路径，然后倒入项目中的WSGI应用。
```
import sys
sys.path.insert(0, './helloworld')
from helloworld import wsgi

app = wsgi.application
```
　　保存上面的代码到`djangoapp.py`，用你的web server运行Django应用程序：
```
(lsbaws) $ python webserver2.py djangoapp:app
WSGIServer: Serving HTTP on port 8888 …
```
　　输入下面的地址，回车：
![](/assets/img/2016/WS_part2_browser_django.png){: .img-medium}

　　正如你之前做过的那几次一样，你也可以在命令行中进行测试，确认Django应用处理了你这次的请求：
```
$ curl -v http://localhost:8888/hello
…
```
　　你试过了吗？你确认了服务器可以和上面三个框架一起工作吗？如果没有的话，一定要试一下。阅读很重要，但是这个系列是关于重新构建，也就意味着你必须自己动手尝试。去尝试一下吧，不用担心，我会等你的。我是认真的，你必须尝试，最好能够自己重新敲下这些代码，来确保它能达到预期的效果。
　　
### 构建自己微型的WSGI Web框架
　　好了，你已经体验了WSGI的威力了：它允许你混合搭配你的web server和web框架。WSGI提供了Python web server和Python web框架之间的最小接口。它非常简单，不管在服务器端还是在框架端实现它都是很容易的。下面的代码片段展示了服务器端和框架段的接口：
```python
def run_application(application):
    """Server code."""
    # This is where an application/framework stores
    # an HTTP status and HTTP response headers for the server
    # to transmit to the client
    headers_set = []
    # Environment dictionary with WSGI/CGI variables
    environ = {}

    def start_response(status, response_headers, exc_info=None):
        headers_set[:] = [status, response_headers]

    # Server invokes the ‘application' callable and gets back the
    # response body
    result = application(environ, start_response)
    # Server builds an HTTP response and transmits it to the client
    …

def app(environ, start_response):
    """A barebones WSGI app."""
    start_response('200 OK', [('Content-Type', 'text/plain')])
    return ['Hello world!']

run_application(app)
```

下面是它的工作原理：
1. 框架提供了`application`可调用对象（WSGI规范没有规定它的实现方式）。
2. 每当收到来自HTTP客户端的请求的时候，服务器端调用`application`可调用对象。它把一个包含`WSGI/CGI`变量的字典`environ`和一个`start_response`可调用对象作为参数传递给`application`可调用对象。
3. 框架/应用程序生成一个HTTP状态（status）和HTTP响应头（headers），并传递给`start_response`可调用对象，让服务器把它们存储起来。框架/应用程序也返回了一个响应正文(body)。
4. 服务器把状态、响应头以及响应正文合并为一个HTTP响应，然后把它传输给客户端（这个步骤不是规范的一部分，但它是流程的下一个逻辑步骤，为了清晰可见我把它加到这里）
下面是这个接口的可视化表现：
![](/assets/img/2016/WS_part2_wsgi_interface.png){: .img-medium}

　　到现在为止，你已经见到了Pyramid, Flask和Django Web应用程序，你也见到了实现WSGI规范的服务器端代码。你见到了不用任何框架实现的精简代码片段。
　　当你用上述其中之一的框架开发web应用程序时候，你是在一个高层面工作，并没有直接与WSGI打交道，但是我知道你一定对WSGI接口的框架端非常好奇，因为你在阅读这篇文章。那么，让我们来创建一个不使用Pyramid, Flask或者Django的微型WSGI Web应用/Web框架，并用你的服务器运行它：
```python
def app(environ, start_response):
    """A barebones WSGI application.

    This is a starting point for your own Web framework :)
    """
    status = '200 OK'
    response_headers = [('Content-Type', 'text/plain')]
    start_response(status, response_headers)
    return ['Hello world from a simple WSGI application!\n’]
```
　　同样的，把上面的代码保存为`wsgiapp.py`或者直接从[GitHub](https://github.com/rspivak/lsbaws/blob/master/part2/wsgiapp.py)上下载，然后用你的web server运行：
```
(lsbaws) $ python webserver2.py wsgiapp:app
WSGIServer: Serving HTTP on port 8888 …
```
　　输入下面的地址，回车，你会看到下面的结果：
![](/assets/img/2016/WS_part2_browser_simple_wsgi_app.png){: .img-medium}

### HTTP响应
　　在学习如何创建一个web server的同时，你刚刚写出了自己的微型WSGI Web框架。真是意外之喜。
　　现在让我们回到服务器给客户端传输什么。下面是当你用HTTP客户端调用你的Pyramid应用程序时，服务器生成的HTTP响应。
![](/assets/img/2016/WS_part2_http_response.png){: .img-medium}

　　上述响应和你在[第一部分](http://zhangyuyu.github.io/2016/03/05/WebServer1/)看到的有些类似，但是也有一些新的内容。比如，有四个之前没见过的HTTP headers: `Content-Type`，`Content-Length`，`Date`，`Server`。这些都是web server生成的响应头信息中应该含有的。虽然它们都不是被严格要求的，但是这些头信息的目的是传输有关HTTP请求/响应的附加信息。
　　现在你更多的了解了关于WSGI的接口，下面是同一个HTTP响应关于哪些部分生成它的详细信息：
![](/assets/img/2016/WS_part2_http_response_explanation.png){: .img-medium}

　　到现在我还没有说任何有关`environ`字典的信息，但是，它基本上就是一个Pyhon字典，这个字典必须包含某些有WSGI规定所规定的WSGI和CGI变量。解析完请求之后，服务器从HTTP请求中拿到了字典所需要的值。字典的内容如下：
![](/assets/img/2016/WS_part2_environ.png){: .img-medium}

　　Web 框架用这个字典的信息，来决定使用哪个view（基于指定路由和请求方法等）；决定从哪里读取请求的正文；以及把错误信息写在哪里，如果有的话。
　　
### 总结 
　　目前为止，现在你已经创建了你自己的WSGI Web server，并且你已经用不同的Web框架写了Web应用程序。这一路你也创建了精简的web应用/Web框架。这真是一件了不起的旅程。让我们重述为了服务一个针对WSGI应用的请求，你的WSGI Web server要做的事情：
1. 首先，服务器启动并且加载由Web框架/应用提供的`application`可调用对象。
2. 其次，服务器读取一个请求
3. 然后，服务器解析请求
4. 紧接着，服务器用请求数据构建一个`environ`字典
5. 然后，服务器把`environ`字典和一个`start_response`可调用对象作为参数传递给`application`可调用对象，并且获得相应正文。
6. 之后，服务器用通过调用`application`可调用对象获得body数据，以及通过`start_response`可调用对象设置的状态和响应头信息一起构造一个HTTP响应。
7. 最后，服务器把HTTP响应传输回客户端。
![](/assets/img/2016/WS_part2_server_summary.png){: .img-medium}

　　这就是全部了，你现在有了一个可以工作的WSGI服务器，它可以服务那些用WSGI兼容的Web框架（Django，Flask，Pyramid或者你自己写的WSGI框架）开发的Web应用程序。最好的是服务器能够在不改变服务器代码的情况下与多个Web框架使用。还不错！
在你离开之前，还有一个问题需要你思考，"如何让你的服务器一次处理多个请求？"
　　敬请关注，在第三部分我会展示给你一种实现方式。谢谢！
　　
>本文翻译自Ruslan's Blog [Let’s Build A Web Server. Part 2.](https://ruslanspivak.com/lsbaws-part2/)
