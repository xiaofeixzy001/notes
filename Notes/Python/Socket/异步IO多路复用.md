[TOC]

# **同步异步**

函数或方法被调用的时候，调用者是否得到最终结果。

直接得到最终结果的是同步调用；没有得到的是异步调用。

例如我要打饭，你不打好我就不走开，一直等到你把饭给我，这是同步；

你不打好，我会干会别的事，但我会盯着，时不时过来问问打好了没，这是异步，异步不保证时长。

# 阻塞非阻塞

函数或方法调用的时候，是否立即返回；

立即就是非阻塞调用，不立即返回就是阻塞调用。

# 区别

同步异步，与阻塞非阻塞没关系。同步异步强调的是是否得到最终结果，而阻塞非阻塞强调的是时间，是否等待。

同步和异步区别在于：调用者是否得到了想要的最终结果。

阻塞和非阻塞区别在于：调用者在获取结果前是否还能干其他的事情。

# 联系

同步阻塞：我啥事都不干，就等你打饭给我。饭是结果，什么都不干一直等就是同步加阻塞。

同步非阻塞：我要打饭，在等饭的时候，我可以干其他事。饭是结果，但我不一直等。

异步阻塞：例如叫号，我要打饭，你说等着叫号，不会直接把饭给我，我啥事都不干，就等着叫号。

异步非阻塞：我要打饭，你让我等叫号，等待期间我可以干别的事，饭好了叫我。

# IO

## 两个阶段

数据准备阶段；

内核空间复制回用户进程缓冲区阶段。

发生IO时，内核从输入设备读写数据，进程从内核复制数据。

系统调用

## IO模型

### 同步IO模型

阻塞IO：进程等待，直到读写完成，全称等待。

非阻塞IO：进程不等待，时不时过来问一下好了没有，IO设备立即返回ERROR，进程不阻塞。

IO多路复用：同时监控多个IO，有一个准备好了，就不需要等了就开始处理，提高了同时处理IO的能力。

### 异步IO模型

进程发起异步IO请求，立即返回。内核完成IO的两个阶段，内核给进程发一个信号。

## IO多路复用实现机制

Win: IOCP，select

Linux：select(效率低),poll,epoll(最好),默认epoll

BSD、MAC: kqueue

select缺点：

1 每次调用select都要讲所有的fd（文件描述符）拷贝到内核空间导致效率下降；

2 遍历所有fd，是否有数据访问（最重要的问题）；

3 最大连接数（1024）。

Poll缺点：

1 最大连接数没有限制。

epoll：

通过三个函数实现：

1 第一个函数：创建epoll句柄：将所有的fd（文件描述符）拷贝到内核空间，但是只需拷贝一次；

2 回调函数：某一个函数或者某一个动作成功完成之后，会触发的函数；

为所有的fd绑定一个回调函数，一旦有数据访问，触发该回调函数，回调函数将fd放到列表中。

3 第三个函数 判断列表是否为空

最大连接数没有上限

## select库

实现了select、poll系统调用，这个基本上操作系统都支持，部分实现了epoll，是底层的IO多路复用模块。

开发中的选择：完全跨平台，使用select、poll，但性能较差；针对不同系统自行选择支持的技术，会提高IO处理性能。

3.4版本提供，高级IO服用库.

类层次结构:

BaseSelector

|-- SelectSelector  实现select

|-- PollSelector  实现poll

|-- EpollSelector  实现epoll

|-- DevpollSelector  实现devpoll

|-- KqueueSelector  实现kqueue

selectors.DefaultSelector返回当前平台最有效、性能最高的实现，但由于没有实现Windows下的IOCP，所以只能退化为select。

selectors模块最后有如下源码：

 

```
# Choose the best implementation, roughly:
#    epoll|kqueue|devpoll > poll > select.
# select() also can't accept a FD > FD_SETSIZE (usually around 1024)
if 'KqueueSelector' in globals():
    DefaultSelector = KqueueSelector
elif 'EpollSelector' in globals():
    DefaultSelector = EpollSelector
elif 'DevpollSelector' in globals():
    DefaultSelector = DevpollSelector
elif 'PollSelector' in globals():
    DefaultSelector = PollSelector
else:
    DefaultSelector = SelectSelector
```

一般选择默认即可,selectors选择器会自动根据当前系统选择最优方法。

## 示例:监控读操作

ChatServer群聊

 

```
import selectors
import socket
import threading
import logging
import datetime

FORMAT = "%(asctime)s %(threadName)s %(thread)d %(message)s"
logging.basicConfig(format=FORMAT, level=logging.INFO)

class ChatServer:
    def __init__(self, ip="0.0.0.0", port=9999):
        self.addr = (ip, port)
        self.sock = socket.socket()
        self.selector = selectors.DefaultSelector()  # 创建selector
        self.event = threading.Event()

    def start(self):  # 启动监听
        self.sock.bind(self.addr)
        self.sock.listen()
        self.sock.setblocking(False)  # 非阻塞

        # 注册,有客户端连接(读)就交给accept函数处理
        # key: fileobj, fd, events, data, 至少应该注册一个之后才能select。
        key = self.selector.register(self.sock, selectors.EVENT_READ, self.accept)
        threading.Thread(target=self._select, name='selector').start()

    def _select(self):
        while not self.event.is_set():
            # 开始监视,等到有文件对象监控事件产生,返回(key,mask)元组
            events = self.selector.select()  # 阻塞
            for k, mask in events:
                callback = k.data  # 回调函数
                callback(k.fileobj, mask)

    def accept(self, sock, mask):  # 多人连接
        newscok, raddr = sock.accept()
        newscok.setblocking(False)  # 非阻塞
        logging.info("Client {} connected.".format(raddr))
        # 注册,监视每一个连接客户端sock对象
        self.selector.register(newscok, selectors.EVENT_READ, self.recv)
        # self.selector.register(newscok, selectors.EVENT_READ | selectors.EVENT_WRITE, self.recv)

    def recv(self, sock, mask):  # 接收客户端数据
        data = sock.recv(1024)  # 阻塞到数据到来
        if not data or data == b'quit':  # 客户端主动断开,注销并关闭客户端socket
            self.selector.unregister(sock)
            sock.close()
            return

        msg = "{:%Y/%m/%d %H:%M:%S} {}:{}\n{}\n".format(datetime.datetime.now(),
                                                        *sock.getpeername(),
                                                        data.decode())
        logging.info(msg)
        msg = msg.encode()

        # 群聊
        for k in self.selector.get_map().values():
            # print(k)
            if k.data == self.recv:

                k.fileobj.send(data)

            """
            print(k)
            (..., data=<bound method ChatServer.accept of <__main__.ChatServer object at 0x0000017DF1502438>>))
            (..., data=<bound method ChatServer.recv of <__main__.ChatServer object at 0x0000017DF1502438>>))
            (..., data=<bound method ChatServer.recv of <__main__.ChatServer object at 0x0000017DF1502438>>))
            """

    def stop(self):
        self.event.set()
        fobjs = []
        for fd, k in self.selector.get_map().items():  # get_map()获取所有sock连接对象
            fobjs.append(k.fileobj)

        for fobj in fobjs:
            self.selector.unregister(fobj)
            fobj.close()

        self.selector.close()

def main():
    cs = ChatServer()
    cs.start()

    while True:
        cmd = input('>> ').strip()
        if cmd == 'quit':
            # 清理工作
            cs.stop()
            threading.Event().wait(3)
            break
        logging.info(threading.enumerate())

if __name__ == '__main__':
    main()
```

模拟100个连接

 

```
import socket
import time

"""
Create a TCP/IP socket
使用列表生成式，生成多个请求。
Winuds使用select支持并发并不多 这里测试100个并发
Linux默认使用epoll可支持上万并发可修改10000
"""

# 并发连接数
n = 100

# 发送信息
msg = [b'This is the message', b'It will be sent', b'in parts']

# 传入链接参数
server_address = ('127.0.0.1', 9999)

# 创建n个socket对象
socks = [socket.socket(socket.AF_INET, socket.SOCK_STREAM) for _ in range(n)]

# 发起n个连接
for s in socks:
    s.connect(server_address)

# 循环发送数据
while True:
    for m in msg:
        for s in socks:
            s.send(m)
            print('{} sending {}'.format(s.getsockname(), m))
            time.sleep(1)
```

以上服务端的代码实现了基本功能，但异常处理没有加。经测试，当客户端强制中断连接,服务端会抛异常。

## 示例:监控写操作

send是写操作，也可以让select监听。

self.selector.register(conn, selectors.EVENT_READ | selectors.EVENT_WRITE, self.recv)

注册语句，要监听selectors.EVENT_READ | selectors.EVENT_WRITE读与写事件。

回调的时候，需要mask来判断究竟是触发了读还是写操作，所以需要修改方法声明，增加mask。

判断是读是写：

 

```
def handler(self, sock, mask):  # 多人连接
    if mask & selectors.EVENT_READ:
        pass

    # 注意:这里是某一个socket的写操作.
    if mask & selectors.EVENT_WRITE:
        # 写缓冲区准备好了,可以写入数据了.
        pass
```

handler方法里面处理读、写，mask有可能是0b01,0b10,0b11,对应EVENT_READ是1，EVENT_WRITE是2，同时都有为3，0表示没有任何活动。

为了解决读取到客户端发来的数据后,如何写出去的问题,这里采用queue队列。

为每一个与客户端连接的socket对象增加对应的队列；

与每一个客户端连接的socket对象，自己维护一个队列，某一个客户端收到信息后，会遍历发给所有客户端的队列。这里完成一对多，即一份数据放到了所有队列中；

与每一个客户端连接的socket对象，发现自己队列有数据，就发送给客户端。

 

```
#!/usr/bin/env python
# -*- coding:utf-8 -*-
# __Author__: xiaofei

import selectors
import socket
import threading
import logging
import datetime
from queue import Queue

FORMAT = "%(asctime)s %(threadName)s %(thread)d %(message)s"
logging.basicConfig(format=FORMAT, level=logging.INFO)

class ChatServer:
    def __init__(self, ip="127.0.0.1", port=9999):
        self.addr = (ip, port)
        self.sock = socket.socket()
        self.selector = selectors.DefaultSelector()  # 创建selector
        self.event = threading.Event()
        self.clients = {}  #

    def start(self):  # 启动监听
        self.sock.bind(self.addr)
        self.sock.listen()
        self.sock.setblocking(False)  # 非阻塞

        # 注册,有客户端连接(读)就交给accept函数处理
        # key: fileobj, fd, events, data, 至少应该注册一个之后才能select。
        key = self.selector.register(self.sock, selectors.EVENT_READ, self.accept)
        threading.Thread(target=self._select, name='selector', daemon=True).start()

    def _select(self):
        while not self.event.is_set():
            # 开始监视,等到有文件对象监控事件产生,返回(key,mask)元组
            events = self.selector.select()  # 阻塞
            for k, mask in events:
                if callable(k.data):
                    callback = k.data  # 回调函数
                    callback(k.fileobj, mask)
                else:
                    callback = k.data[0]
                    callback(k, mask)

    def accept(self, sock, mask):  # 多人连接
        conn, raddr = sock.accept()
        conn.setblocking(False)
        self.clients[raddr] = (self.handle, Queue())

        # 注册,监视每一个与客户端连接的socket对象
        self.selector.register(conn, selectors.EVENT_READ | selectors.EVENT_WRITE, self.clients[raddr])

    def handle(self, key: selectors.SelectorKey, mask):  # 接收客户端数据
        if mask & selectors.EVENT_READ:
            sock = key.fileobj
            raddr = sock.getpeername()
            data = sock.recv(1024)

            if not data or data == b'quit':
                self.selector.unregister(sock)
                sock.close()
                self.clients.pop(raddr)
                return
            msg = "{:%Y/%m/%d %H:%M:%S} {}:{}\n{}\n".format(datetime.datetime.now(),
                                                            *sock.getpeername(),
                                                            data.decode())
            logging.info(msg)
            msg = msg.encode()

            for k in self.selector.get_map().values():
                logging.info(k)
                if isinstance(k.data, tuple):
                    k.data[1].put(data)

        if mask & selectors.EVENT_WRITE:
            # 因为写一直就绪,mask为2,所以一直可以写,从而导致select()不断循环,如同不阻塞一般
            if not key.data[1].empty():
                key.fileobj.send(key.data[1].get())

    def stop(self):
        self.event.set()
        fobjs = []
        for fd, k in self.selector.get_map().items():  # get_map()获取所有sock连接对象
            fobjs.append(k.fileobj)

        for fobj in fobjs:
            self.selector.unregister(fobj)
            fobj.close()

        self.selector.close()

def main():
    cs = ChatServer()
    cs.start()

    while True:
        cmd = input('>> ').strip()
        if cmd == 'quit':
            # 清理工作
            cs.stop()
            threading.Event().wait(3)
            break

        logging.info(threading.enumerate())
        print("-" * 30)
        logging.info("{} {}".format(len(cs.clients), cs.clients))
        logging.info(list(map(lambda x: (x.fileobj.fileno(), x.data), cs.selector.get_map().values())))
        print("-" * 30)

if __name__ == '__main__':
    main()
```



