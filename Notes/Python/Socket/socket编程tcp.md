[TOC]

# Socket介绍

## 套接字

Python中提供socket.py标准库，非常底层的接口库。

Socket是一种通用的网络编程接口，和网络层次没有一一对应的关系。

Python提供了两个基本的socket模块。

第一个是Socket，它提供了标准的BSD Sockets API。

第二个是SocketServer，它提供了服务器中心类，可以简化网络服务器的开发。

下面讲的是Socket模块功能。

套接字格式：

socket(family,type[,protocal])：使用给定的地址族、套接字类型、协议编号(默认为0)来创建套接字。

## 协议族family

AF表示Address Family，用于socket()第一个参数。

AF_UNIX :只能够用于单一的Unix系统进程间通信，Unix Domain Socket，win没有。

AF_INET :IPv4，服务器之间网络通信。

AF_INET6 :IPv6。

## socket类型type

SOCK_STREAM: 面向连接的流套接字。默认值,TCP协议。

SOCK_DGRAM: 无连接的数据报文套接字，UDP协议。

SOCK_SEQPACKET: 可靠的连续数据报文服务。

SOCK_RAW: 原始套接字，普通的套接字无法处理ICMP、IGMP等网络报文，而SOCK_RAW可以；其次，SOCK_RAW也可以处理特殊的IPv4报文；此外，利用原始套接字，可以通过IP_HDRINCL套接字选项由用户构造IP头。

## TCP编程

Socket编程，需要两端，一般来说需要一个服务端(Server)，一个客户端(Client)。

服务端步骤：

1 创建Socket对象

2 绑定IP地址，二元组("IP", PORT)，使用bind()方法

3 开始监听，将在指定ip的端口上监听，使用listen()方法

4 获取用于传送数据的Socket对象,使用accept()方法，意思是阻塞等待至客户端建立连接，返回一个新的Socket对象和客户端地址的元组。

客户端步骤：

1 创建Socket对象,默认使用TCP协议

2 使用connect连接服务端，二元组("IP", PORT)形式,connect()方法

3 传输数据，使用send、recv方法发送、接收数据

4 关闭连接，释放资源

![img](socket%E7%BC%96%E7%A8%8Btcp.assets/e34f1b9a-6dfd-4968-a2d0-b8ab81b35665.png)

 

```
import socket
import logging
import threading

FORMAT = "%(asctime)s %(threadName)s %(thread)d %(message)s"
logging.basicConfig(format=FORMAT, level=logging.INFO)

# 服务端
s = socket.socket()
s.bind(('127.0.0.1', 9999,))
s.listen()
s1, info = s.accept()

while True:
    try:
        msg = s1.recv(1024)
        print(msg, info)
        if not msg: continue
        s1.send(msg.upper())
    except Exception:
        break
s.close()

# 客户端
client = socket.socket()
ipaddr = ('127.0.0.1', 9999)
client.connect(ipaddr)  # 连接服务器
client.send("b'abcd\n'")  # 发送数据
data = client.recv(1024)  # 接收服务端返回数据
print(data)
client.close()
```

新的Socket对象有两个方法用于和客户端通信：

接收数据：recv(bufsize, [flags]) 使用缓冲区接收数据

发送数据: send(bytes)

## **Socket对象方法**

注意:

1）TCP发送数据时，已建立好TCP连接，所以不需要指定地址。UDP是面向无连接的，每次发送要指定是发给谁。

2）服务端与客户端不能直接发送列表，元组，字典。需要字符串化repr(data)。

### 服务端

s.bind(): 绑定(主机,端口号)到套接字

s.listen(): 开始TCP监听，backlog指定在拒绝连接之前，操作系统可以挂起的最大连接数量。该值至少为1，大部分应用程序设为5就可以了。

s.accept(): 被动接受TCP客户的连接,(阻塞式)等待连接的到来，接受TCP连接并返回(conn,address),其中conn是新的套接字对象，可以用来接收和发送数据。address是连接客户端的地址。

### 客户端

s.connect(address): 主动初始化TCP服务器连接，连接到address处的套接字。一般address的格式为元组(hostname,port),如果连接出错，返回socket.error错误。

s.connect_ex(adddress): connect()函数的扩展版本,出错时返回出错码,而不是抛出异常，功能与connect(address)相同，但是成功返回0，失败返回errno的值。

### 公共函数

s.recv(bufsize[, flag])：接受TCP套接字的数据，默认是阻塞方式。数据以字符串形式返回，bufsize指定要接收的最大数据量。flag提供有关消息的其他信息，通常可以忽略，(send在待发送数据量大于己端缓存区剩余空间时,数据丢失,不会发完)。

s.send(bytes[, flag])：发送TCP数据,返回值是要发送的字节数量，该数量可能小于bytes的字节大小。

s.sendall(bytes[, flag])：TCP发送全部数据，但在返回之前会尝试发送所有数据。成功返回None，失败则抛出异常。本质就是循环调用send，在待发送数据量大于己端缓存区剩余空间时,数据不丢失,循环调用send直到发完。

s.sendto(string[, flag], address)：发送UDP数据。将数据发送到套接字，address是形式为(ipaddr，port)的元组，指定远程地址。返回值是发送的字节数。

s.recvfrom(bufsize[, flag])：接受UDP套接字的数据。与recv()类似，但返回值是(bytes,address)二元组。其中data是包含接收数据的字符串，address是发送数据的套接字地址。

s.recv_into(buffer[, nbytes[, flags]]): 获取到nbytes的数据后，存储到buffer中。如果nbytes没有指定或为0，将buffer大小的数据存储buffer中，返回接收的字节数。

s.recvfrom_into(buffer[, nbytes[, flags]]): 获取数据，返回一个二元组(bytes, address)到buffer中。

s.setblocking(flag): 如果flag为0，则将套接字设为非阻塞模式，否则将套接字设为阻塞模式(默认值)。非阻塞模式下，如果调用recv()没有发现任何数据，或send()调用无法立即发送数据，那么将引起socket.error异常。

s.sendfile(file, offset=0, count=None): 发送一个文件直到EOF，使用高性能的os.sendfile机制，返回发送的字节数。如果win下不支持sendfile，或者不是普通文件，使用send()发送文件。offset告诉起始位置。3.5版本开始。

s.makefile(mode='r', buffering=None, *, encoding=None, errors=None, newline=None): 创建一个与该套接字相关连的文件对象，将recv方法看做读方法，将send方法看做写方法。

s.close(): 关闭套接字。

s.getpeername(): 返回连接套接字的远程地址。返回值通常是元组（ipaddr,port）。

s.getsockname(): 返回套接字自己的地址。通常是一个元组(ipaddr,port)。

s.setsockopt(level, optname,value): 设置指定套接字选项的值。比如缓冲区大小。太多了，去看文档。不同系统不同版本都不尽相同。

s.getsockopt(level, optname[, buflen]): 返回指定套接字选项的值。

s.settimeout(timeout): 设置套接字操作的超时期，timeout是一个浮点数，单位是秒。值为None表示没有超时期。一般，超时期应该在刚创建套接字时设置，因为它们可能用于连接的操作（如connect()）。

s.gettimeout(): 返回当前超时期的值，单位是秒，如果没有设置超时期，则返回None。

s.fileno(): 返回套接字的文件描述符。

### 锁方法

s.setblocking(): 设置套接字的阻塞与非阻塞模式

s.settimeout(): 设置阻塞套接字操作的超时时间

s.gettimeout(): 得到阻塞套接字操作的超时时间

### 文件方法

s.fileno(): 套接字的文件描述符

s.makefile(): 创建一个与该套接字相关的文件

## 示例：群聊

需求分析：

C/S程序

服务端功能：绑定地址和端口并监听

建立连接，可以和多个客户端建立连接

接收不同用户信息

分发，将接收的信息转发到已连接的所有客户端

停止服务

记录连接的客户端

 

```
# 服务端
import socket
import threading
import logging
import datetime

FORMAT = "%(asctime)s %(threadName)s %(message)s"
logging.basicConfig(format=FORMAT, level=logging.INFO)

class ChatServer:
    def __init__(self, ip='127.0.0.1', port=9999):
        """
        启动服务
        """
        self.sock = socket.socket()
        self.addr = (ip, port,)
        self.clients = {}  # 客户端字典,用于消息分发
        self.event = threading.Event()

    def start(self):
        """
        开始监听
        """
        self.sock.bind(self.addr)
        self.sock.listen()

        # accept会阻塞主线程,所以开启一个新线程
        threading.Thread(target=self.accept).start()

    def accept(self):
        """
        多人连接
        :return:
        """
        while not self.event.is_set():
            sock, client = self.sock.accept()  # 阻塞,等待客户端连接
            logging.info(sock)
            logging.info(client)
            self.clients[client] = sock  # 加入到客户端列表
            # recv是阻塞的,所以再开启一个新线程
            threading.Thread(target=self.recv, args=(sock, client,)).start()

    def recv(self, sock:socket.socket, client):
        """
        接收客户端数据
        :return:
        """
        while not self.event.is_set():
            try:
                data = sock.recv(1024)  # 等待接收客户端数据,二进制
                logging.info(data)
            except Exception as e:
                logging.error(e)
                data = b'quit'

            # 客户端退出命令
            if data == b'quit':
                self.clients.pop(sock.getpeername())
                sock.close()
                break

            msg = "ack{}. {} {}".format(sock.getpeername(),
                                        datetime.datetime.now().strftime("%Y/%m/%d-%H:%M:%S"),
                                        data.decode()).encode()
            logging.info(msg)
            for s in self.clients.values():
                """
                将消息分发所有已连接客户端
                """
                s.send(msg)

    def stop(self):
        """
        停止监听
        :return:
        """
        for s in self.clients.values():
            s.close()
        self.sock.close()
        self.event.set()

cs = ChatServer()
cs.start()

while True:
    cmd = input('>> ').strip()
    if cmd == 'quit':
        cs.stop()
        threading.Event().wait(3)
        break
    logging.info(threading.enumerate())

# 客户端
class ChatClient:
    def __init__(self, rip='127.0.0.1', rp=9999):
        self.raddr = (rip, rp,)
        self.sock = socket.socket()
        self.event = threading.Event()

    def start(self):
        self.sock.connect(self.raddr)
        self.send("I'm ready.")
        threading.Thread(target=self.recv, name='recv').start()

    def recv(self):
        while not self.event.is_set():
            try:
                data = self.sock.recv(1024)
            except Exception as e:
                logging.error(e)
                break
            # msg = "ack{} {} {}".format(self.raddr, )
            logging.info(data)

    def send(self, msg: str):
        self.sock.send(msg.encode())

    def stop(self):
        self.sock.close()
        self.event.wait(3)
        self.event.set()
        logging.info("Client stopped.")

def main():
    cc = ChatClient()
    cc.start()
    while True:
        cmd = input(">> ").strip()
        if cmd == 'quit':
            cc.stop()
            break
        cc.send(cmd)
    print(threading.enumerate())

if __name__ == '__main__':
    main()
```

仅做演示用，真正生产环境中不会用这种方式的。



