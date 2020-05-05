[TOC]

# SocketServer

socket编程过于底层，编程虽有套路，但想写出健壮的代码还是比较困难的，所有很多语言都对socket底层API进行封装，Python的封装就是socketserver模块。它是网络服务编程框架，便于企业级快速开发。

基于tcp的套接字，关键就是两个循环，一个链接循环，一个通信循环。

socketserver模块中分为两大类：server类（解决链接问题）和request类（解决通信问题）。

## 继承关系

SocketServer简化了网络服务器的编写。

它有4个同步类：TCPServer, UDPServer, UnixStreamServer, UnixDatagramServer。

2个Mixin类：ForkingMixln和ThreadingMixln类，用来支持异步。

继承关系图如下：

fork是创建多进程，thread是创建多线程。

![img](socketserver%E6%A8%A1%E5%9D%97.assets/96135aaf-1ceb-4ab6-8853-7d1610ea1a3b.jpg)

![img](socketserver%E6%A8%A1%E5%9D%97.assets/657b8c67-3803-4e9a-bb7c-a140cef21ff3.jpg)

 

```
class ForkingUDPServer(ForkingMixln, UDPServer): pass
class ForkingTCPServer(ForkingMixln, TCPServer): pass
class ThreadingUDPServer(ThreadingMixln, UDPServer): pass
class ThreadingTCPServer(ThreadingMixln, TCPServer): pass
```

### server类

单线程/单进程

![img](socketserver%E6%A8%A1%E5%9D%97.assets/32b88885-ed43-4d75-9687-4f2450e1c62a.jpg)

![img](socketserver%E6%A8%A1%E5%9D%97.assets/52589048-4136-49e9-b80d-15dd13845f81.jpg)

![img](socketserver%E6%A8%A1%E5%9D%97.assets/1b44f6c7-33d8-4807-a39b-6b6777672ac4.jpg)

### request类

![img](socketserver%E6%A8%A1%E5%9D%97.assets/fb849db9-6f89-4fc3-8762-91b4f04fd698.jpg)

![img](socketserver%E6%A8%A1%E5%9D%97.assets/87cbf8ab-8dcb-4dc5-9629-a7ef1594025e.jpg)

## 编程接口

socketserver.BaseServer(server_address, RequestHandlerClass)

需提供服务器绑定的地址信息，和用于处理请求的RequestHandlerClass类。

RequestHandlerClass类必须是BaseRequestHandler类的子类，在BaseServer中代码如下：

 

```
class BaseServer:
    def __init__(self, server_address, RequestHandlerClass):
        """Constructor.  May be extended, do not override."""
        self.server_address = server_address
        self.RequestHandlerClass = RequestHandlerClass
        self.__is_shut_down = threading.Event()
        self.__shutdown_request = False

    def finish_request(self, request, client_address):  # 处理请求的方法
        """Finish one request by instantiating RequestHandlerClass."""
        self.RequestHandlerClass(request, client_address, self)  # RequestHandlerClass构造
```

BaseRequestHandler类

它是和用户连接的用户请求处理类的基类，定义为BaseRequestHandler(request, client_address, server)

服务端Server实例接收用户请求后，最后会实例化这个类。

它被初始化时，送入3个构造参数: request, client_address, server自身。

以后就可以在BaseRequestHandler类的实例上使用一下属性：

self.request是和客户端的连接的socket对象；

self.client_address是客户端地址；

self.server是TCPServer实例本身。

这类类在初始化的时候，它会依次调用3个方法，子类可以覆盖这些方法。

 

```
# BaseRequestHandler子类覆盖的方法框架
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

    def setup(self):  # 每个连接初始化
        pass

    def handle(self):  # 每一次请求处理
        pass

    def finish(self):  # 每一个连接清理
        pass
```

以下述代码为例，分析socketserver源码：

 

```
import socketserver
import threading
import logging

FORMAT = "%(asctime)s %(threadName)s %(thread)d %(message)s"
logging.basicConfig(format=FORMAT, level=logging.INFO)

class MyHandler(socketserver.BaseRequestHandler):
    def handle(self):
        # super().handle()  # 无需调用,因为父类的handle什么都没做
        print("-" * 30)
        print(self.request)  # 服务端收到客户端连接请求的socket对象
        print(self.client_address)  # 客户端地址信息
        print(self.server)  # 服务端信息

        print("-" * 30)
        print(threading.enumerate())
        print(threading.current_thread())

        for i in range(3):
            data = self.request.recv(1024)
            logging.info(data)

addr = ('127.0.0.1', 9999)
server = socketserver.ThreadingTCPServer(addr, MyHandler)  # 异步
# server = socketserver.TCPServer(addr, MyHandler)  # 同步
server.serve_forever()  # 永久
```

通过2个客户端连接并发送测试数据，查看效果。

handle方法相当于socket的recv方法。

每一个不同的连接请求过来后，生成这个连接的socket对象，也就是self.request，客户端地址是self.client_address。

创建服务端需要几个步骤：

1 从BaseRequestHandler类派生出子类，并覆盖其handler()方法来创建请求处理程序类，此方法将处理传入请求；

2 实例化一个服务器类，传参服务器的地址和请求处理类；

3 调用server_close()关闭套接字。

# 示例

## EchoServer

顾名思义，来什么消息回显什么消息。

 

```
import socketserver
import threading
import logging
import sys

FORMAT = "%(asctime)s %(threadName)s %(thread)d %(message)s"
logging.basicConfig(format=FORMAT, level=logging.INFO)

class EchoHandler(socketserver.BaseRequestHandler):
    def setup(self):
        super().setup()  # 因为没有自己的__init__
        self.event = threading.Event()  # 初始工作

    def finish(self):
        super().finish()
        self.event.set()

    def handle(self):
        super().handle()

        while not self.event.is_set():
            data = self.request.recv(1024)
            msg = '{} {}'.format(self.client_address, data).encode()
            self.request.send(msg)
        print('~~end~~')

addr = ('127.0.0.1', 9999)
server = socketserver.ThreadingTCPServer(addr, EchoHandler)
server_thread = threading.Thread(target=server.serve_forever, name='EchoServer', daemon=True)
server_thread.start()

try:
    while True:
        cmd = input('>> ').strip()
        if cmd == 'quit':
            # server.server_close()
            server.shutdown()
            break
        logging.info(threading.enumerate())
except Exception as e:
    logging.info(e)
except KeyboardInterrupt:
    sys.exit(0)
finally:
    logging.info('Exit')
    sys.exit(0)
```

## ChatServer群聊

 

```
import socketserver
import threading
import logging
import sys

FORMAT = "%(asctime)s %(threadName)s %(thread)d %(message)s"
logging.basicConfig(format=FORMAT, level=logging.INFO)

class ChatHandler(socketserver.BaseRequestHandler):

    clients = {}  # 记录多个实例

    def setup(self):
        super().setup()  # 因为没有自己的__init__
        self.event = threading.Event()  # 初始工作
        self.clients[self.client_address] = self.request

    def finish(self):
        super().finish()
        # self.clients.pop(self.client_address)  # finally,一定会执行
        self.event.set()

    def handle(self):
        super().handle()
        # event = threading.Event()  # 局部变量
        while not self.event.is_set():
            data = self.request.recv(1024)
            if data == b'' or data == b'quit':  # 客户端主动断开连接,默认会发送一个空bytes过来
                self.clients.pop(self.client_address)
                break
            print(data, '++++++++++++', self.client_address)
            msg = '{} {}'.format(self.client_address, data).encode()
            # 实现一对多
            for c in self.clients.values():
                c.send(msg)

        print('~~end~~')


addr = ('127.0.0.1', 9999)
server = socketserver.ThreadingTCPServer(addr, ChatHandler)

if __name__ == '__main__':
    server_thread = threading.Thread(target=server.serve_forever, name='ChatHandler', daemon=True)
    server_thread.start()

    try:
        while True:
            cmd = input('>> ').strip()
            if cmd == 'quit':
                # server.server_close()
                server.shutdown()
                break
            logging.info(threading.enumerate())
    except Exception as e:
        logging.info(e)
    except KeyboardInterrupt:
        sys.exit(0)
    finally:
        logging.info('Exit')
        sys.exit(0)
```

## FTP程序

Server端

 

```
import socketserver
import struct
import json
import os
class FtpServer(socketserver.BaseRequestHandler):
    coding='utf-8'
    server_dir='file_upload'
    max_packet_size=1024
    BASE_DIR=os.path.dirname(os.path.abspath(__file__))
    def handle(self):
        print(self.request)
        while True:
            data=self.request.recv(4)
            data_len=struct.unpack('i',data)[0]
            head_json=self.request.recv(data_len).decode(self.coding)
            head_dic=json.loads(head_json)
            # print(head_dic)
            cmd=head_dic['cmd']
            if hasattr(self,cmd):
                func=getattr(self,cmd)
                func(head_dic)
    def put(self,args):
        file_path = os.path.normpath(os.path.join(
            self.BASE_DIR,
            self.server_dir,
            args['filename']
        ))

        filesize = args['filesize']
        recv_size = 0
        print('----->', file_path)
        with open(file_path, 'wb') as f:
            while recv_size < filesize:
                recv_data = self.request.recv(self.max_packet_size)
                f.write(recv_data)
                recv_size += len(recv_data)
                print('recvsize:%s filesize:%s' % (recv_size, filesize))

ftpserver=socketserver.ThreadingTCPServer(('127.0.0.1',8080),FtpServer)
ftpserver.serve_forever()
```

Client端

 

```
import socket
import struct
import json
import os

class MYTCPClient:
    address_family = socket.AF_INET
    socket_type = socket.SOCK_STREAM
    allow_reuse_address = False
    max_packet_size = 8192
    coding='utf-8'
    request_queue_size = 5

    def __init__(self, server_address, connect=True):
        self.server_address=server_address
        self.socket = socket.socket(self.address_family,
                                    self.socket_type)
        if connect:
            try:
                self.client_connect()
            except:
                self.client_close()
                raise

    def client_connect(self):
        self.socket.connect(self.server_address)

    def client_close(self):
        self.socket.close()

    def run(self):
        while True:
            inp=input(">>: ").strip()
            if not inp:continue
            l=inp.split()
            cmd=l[0]
            if hasattr(self,cmd):
                func=getattr(self,cmd)
                func(l)

    def put(self,args):
        cmd=args[0]
        filename=args[1]
        if not os.path.isfile(filename):
            print('file:%s is not exists' %filename)
            return
        else:
            filesize=os.path.getsize(filename)

        head_dic={'cmd':cmd,'filename':os.path.basename(filename),'filesize':filesize}
        print(head_dic)
        head_json=json.dumps(head_dic)
        head_json_bytes=bytes(head_json,encoding=self.coding)

        head_struct=struct.pack('i',len(head_json_bytes))
        self.socket.send(head_struct)
        self.socket.send(head_json_bytes)
        send_size=0
        with open(filename,'rb') as f:
            for line in f:
                self.socket.send(line)
                send_size+=len(line)
                print(send_size)
            else:
                print('upload successful')

client=MYTCPClient(('127.0.0.1',8080))
client.run()
```

# 总结

为每一个连接提供RequestHandlerClass类实例，依次调用setup、handle、finish方法，且使用了try..finally结构，保证finish方法一定能被调用。这些方法依次执行完成，如果想维持这个连接和客户端通信，就需要在handle函数中使用循环。

socketserver模块提供不同的类，但是编程接口是一样的，即使是多进程多线程的类也是一样，大大减少编程难度。



