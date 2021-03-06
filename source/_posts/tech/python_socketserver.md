---
title: Python模块解析 - SocketServer模块
date: 2016-04-02 19:03
tags: [Python,Web,TECH]
show_copyright : true
---
之前看python网络编程那一块的时候,有学习SocketServer的代码,上网有查过这个lib的用法,但是发觉不看source code的话很难真正厘清内部的运作.所以这篇博文主要从一个简单的例子从原码的角度做分析整个SocketServer类的解析.
注:本文基于python2.7,SocketServer的版本为0.4.不同的版本可能会导致差异


## 类的继承关系
我们首先先要厘清一下SocketServer里面基本的类的关系,我将大体的继承关系用下图表示：
![](/images/socketserver_1.png) ![](/images/socketserver_2.png)
可以看到,SocketServer主要包含4个基本类:`BaseServer`(socket server建立的基类,主要的方法:serve_forever),`BaseRequestHandler`(request的处理类,主要的方法:handle方法的重写),还有两个使用分叉和线程来处理异步通信的类:`ForkingMixIn`和`ThreadingMixIn`
其他的类都是在以上四个基类的基础上延伸出来的.我们用到的最多的是针对TCP流式套接字的类:`TCPServer`.后面的简单例子也是基于此

另外看source code的时候有一个关于多个类同时继承的地方需要多注意,也就是关于新式类和经典类的继承区别.详情请参考[Python新式类和经典类的继承区别](/2015/12/12/python_new_and_old_class/)

<!--more-->
## 举个栗子
先看一个可爱的小栗子:
``` python
from SocketServer import TCPServer,StreamRequestHandler

class Handler(StreamRequestHandler):
	def handle(self):
		addr = self.request.getpeername()
		print 'Got connect from ', addr
		self.wfile.write('Thanks for connecting')

server = TCPServer(('', 8000), Handler)
server.serve_forever()
```
从import的信息,我们下面用到的主要是`TCPServer`和`StreamRequestHandler`两个类,我们先跳过程式中新定义的`Handler`类,直接看程式的运行:

``` python
server = TCPServer(('', 8000), Handler)
server.serve_forever()
```
看`TCPServer`类的初始化函数：
``` python
    def __init__(self, server_address, RequestHandlerClass, bind_and_activate=True):
        """Constructor.  May be extended, do not override."""
        BaseServer.__init__(self, server_address, RequestHandlerClass)
        self.socket = socket.socket(self.address_family,
                                    self.socket_type)
        if bind_and_activate:
            self.server_bind()
            self.server_activate()
```
这里可以知道,因为是继承`BaseServer`,所以在`TCPServer`实例化之后,实例server就拥有`TCPServer`和`BaseServer`所有的方法和属性.后面就是socket的正常流程:创建(socket.socket())->绑定(socket.bind())->监听(socket.listen()),只不过这些方法在TCPServer中重新做了封装
最后调用`BaseServer`中的serve_forever()方法:
``` python
    def serve_forever(self, poll_interval=0.5):
        """Handle one request at a time until shutdown.\

        Polls for shutdown every poll_interval seconds. Ignores\
        self.timeout. If you need to do periodic tasks, do them in\
        another thread.\
        """
        self.__is_shut_down.clear()
        try:
            while not self.__shutdown_request:
                # XXX: Consider using another file descriptor or
                # connecting to the socket to wake this up instead of
                # polling. Polling reduces our responsiveness to a
                # shutdown request and wastes cpu at all other times.
                r, w, e = select.select([self], [], [], poll_interval)
                if self in r:
                    self._handle_request_noblock()
        finally:
            self.__shutdown_request = False
            self.__is_shut_down.set()
```
不断的处理request请求直到收到一个shutdown请求,主要的做的事情其实就是`_handle_request_noblock()`,在`_handle_request_noblock`中：
``` python
    def _handle_request_noblock(self):
        """Handle one request, without blocking.

        I assume that select.select has returned that the socket is
        readable before this function was called, so there should be
        no risk of blocking in get_request().
        """
        try:
            request, client_address = self.get_request()
        except socket.error:
            return
        if self.verify_request(request, client_address):
            try:
                self.process_request(request, client_address)
            except:
                self.handle_error(request, client_address)
                self.shutdown_request(request)
```
可以看到,先后做了`get_request`(TCPServer中的方法,其实就是socket.accept())->`process_request()`->`finish_request(request, client_address)`->
`shutdown_request(其实就是TCPServer中的socket.close)`

这里重点是`finish_request()`:
``` python
    def finish_request(self, request, client_address):
        """Finish one request by instantiating RequestHandlerClass."""
        self.RequestHandlerClass(request, client_address, self)
```
这里的`RequestHandlerClass`就是我们之前跳过的`Handler`类,也就是在一开始被传入`TCPServer`的参数.request处理的精华就在这里啦.
因为`Handler`是继承`StreamRequestHandler`,而`StreamRequestHandler`又是继承`BaseRequestHandler`,所以`Handler`的关键还是在`BaseRequestHandler`类中:
``` python
class BaseRequestHandler:

    def __init__(self, request, client_address, server):
        self.request = request
        self.client_address = client_address
        self.server = server
        self.setup()
        try:
            self.handle()
        finally:
            self.finish()

    def setup(self):
        pass

    def handle(self):
        pass

    def finish(self):
        pass
```
所以就三步:`setup()`->`handle()`->`finish()`,三个方法除了handle其他都是在`StreamRequestHandler`中有定义,而handle就是预留出来给用户自己定义要处理request的内容.
可以看我们定义的类`Handler`中的handle方法,其实就是简单的对setup()建立的connection做类文件的读写,其实这就是最简单的客户端和服务端的通信.finish的方法就是做一些关闭的收尾工作.


## 结语
至此,TCPServer的大体工作流程就结束了.`UDPServer`其实就是重写了一下部分的`TCPServer`的方法,大体上也一样.
所以`SocketServer`的用法还是主要对handle的方法重写,这个类算是很多服务器框架的基础了.

至于剩下的关于分叉和线程处理异步通信的部分就后面在写吧


That's all,Thank you
