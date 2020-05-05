[TOC]

# 编程流程

![img](socket%E7%BC%96%E7%A8%8Budp.assets/af0686c2-5c18-4e07-a707-dd71bc7cad89.png)

## 服务端

1 创建sokcet对象，socket.SOCK_DGRAM；

2 绑定IP和Port，bind()方法；

3 传输数据:

接收数据：socket.recvfrom(bufsize[, flags]), 获得一个二元组(string, address);

发送数据: scoket.sendto(string, address),发给某地址某信息.

4 释放资源。

 

```
server = socket.socket(type=socket.SCOK_DGRAM)
server.bind(('0.0.0.0', 9999,))  # 绑定地址和端口
data = server.recv(1024)  # 阻塞,等待连接
data = server.recvfrom(1024)  # 阻塞,等待数据(value, (ip, port,))
server.sendto(b'7', ('192.168.100.1', 10000,))
server.close()
```

## 客户端

1 创建socket对象，socket.SOCK_DGRAM

2 发送数据, socket.sendto(string, address)

3 接收数据, socket.recvfrom(bufsize[, flags]),获得一个二元组(string, address);

4 释放资源。

 

```
client = socket.socket(type=socket.SCOK_DGRAM)
raddr = ('192.168.100.1', 10000)
client.connect(raddr)
client.sendto(b'8', raddr)
client.send(b'9')
data = client.recvfrom(1024)  # 阻塞,等待数据(value, (ip, port,))
data = client.recv(1024)  # 阻塞,等待数据
client.close()
```

注意，UDP是无连接协议，所以可以只有任何一端，例如客户端数据发往服务端，服务端存在与否无所谓。

## UDP方法

bind: 绑定本地地址和端口，会立即占用

connect: 可立即占用本地地址和端口，填充远端地址和端口raddr

sendto: 可立即占用本地地址和端口，并把数据发往指定远端。只有有了本地绑定端口，sendto就可以向任何远端发送数据。

send: 需要和connect方法配合，可以使用已经从本地端口把数据发往raddr指定的远端。

recv: 要求一定要在占用了本地端口后，返回接收的数据。

recvfrom: 要求一定要占用了本地端口后，返回接收的数据和对端地址的二元组。


1 发消息，都是将数据发送到己端的发送缓冲中，收消息都是从己端的缓冲区中收。tcp：send发消息，recv收消息；udp：sendto发消息，recvfrom收消息；2 send与sendintotcp是基于数据流的，而udp是基于数据报的：send(bytes_data):发送数据流，数据流bytes_data若为空，自己这段的缓冲区也为空，操作系统不会控制tcp协议发空包sendinto(bytes_data,ip_port)：发送数据报，bytes_data为空，还有ip_port,所有即便是发送空的bytes_data,数据报其实也不是空的，自己这端的缓冲区收到内容，操作系统就会控制udp协议发包3 recv与recvfromtcp协议：- 如果收消息缓冲区里的数据为空，那么recv就会阻塞（阻塞很简单，就是一直在等着收）。- 只不过tcp协议的客户端send一个空数据就是真的空数据，客户端即使有无穷个send空，也跟没有一个样。- tcp基于连接通信基于链接，则需要listen（backlog），指定半连接池的大小；基于链接，必须先运行的服务端，然后客户端发起链接请求；对于mac系统：如果一端断开了链接，那另外一端的链接也跟着完蛋recv将不会阻塞，收到的是空(解决方法是：服务端在收消息后加上if判断，空消息就break掉通信循环)对于windows/linux系统：如果一端断开了链接，那另外一端的链接也跟着完蛋recv将不会阻塞，收到的是空(解决方法是：服务端通信循环内加异常处理，捕捉到异常后就break掉通讯循环)udp协议:- 如果如果收消息缓冲区里的数据为“空”，recvfrom也会阻塞- 只不过udp协议的客户端sendinto一个空数据并不是真的空数据（包含：空数据+地址信息，得到的报仍然不会为空），所以客户端只要有一个sendinto（不管是否发送空数据，都不是真的空数据），服务端就可以recvfrom到数据。- udp无链接无链接，因而无需listen（backlog），更加没有什么连接池之说了无链接，udp的sendinto不用管是否有一个正在运行的服务端，可以己端一个劲的发消息，只不过数据丢失recvfrom收的数据小于sendinto发送的数据时，在mac和linux系统上数据直接丢失，在windows系统上发送的比接收的大直接报错只有sendinto发送数据没有recvfrom收数据，数据丢失注意：1 你单独运行上面的udp的客户端，你发现并不会报错，相反tcp却会报错，因为udp协议只负责把包发出去，对方收不收，我根本不管，而tcp是基于链接的，必须有一个服务端先运行着，客户端去跟服务端建立链接然后依托于链接才能传递消息，任何一方试图把链接摧毁都会导致对方程序的崩溃。2 上面的udp程序，你注释任何一条客户端的sendinto，服务端都会卡住，为什么？因为服务端有几个recvfrom就要对应几个sendinto，哪怕是sendinto(b'')那也要有。3 udp不会粘包,但是会丢包.

## 群聊示例

客户端发消息，服务器端将收到的消息发送给所有已知的客户端；

利用心跳机制实现服务端可以知晓客户端退出；

服务端:

 

```
import socket
ip_port=('127.0.0.1',9000)
BUFSIZE=1024
udp_server_client=socket.socket(socket.AF_INET,socket.SOCK_DGRAM)

udp_server_client.bind(ip_port)

while True:
    msg,addr=udp_server_client.recvfrom(BUFSIZE)
    print(msg,addr)

    udp_server_client.sendto(msg.upper(),addr)
```

多线udp服务端:

 

```
import socket
import threading
import logging
import datetime

FORMAT = "%(asctime)s %(threadName)s %(message)s"
logging.basicConfig(format=FORMAT, level=logging.INFO)

class ChatUdpServer:
    def __init__(self, ip='127.0.0.1', port=9999, interval=10):
        self.sock = socket.socket(type=socket.SOCK_DGRAM)
        self.addr = (ip, port,)
        self.clients = {}  # 记录客户端
        self.event = threading.Event()
        self.interval = interval

    def start(self):
        self.sock.bind(self.addr)
        threading.Thread(target=self.recv, name='recv').start()

    def recv(self):
        while not self.event.is_set():
            localset = set()  # 暂存超时的客户端用于pop

            data, raddr = self.sock.recvfrom(1024)  # 等待接收客户端数据,二进制
            current = datetime.datetime.now().timestamp()  # float

            if data.strip() == b'^hb^':
                self.clients[raddr] = current
                continue

            elif data.strip() == b'quit':
                # self.clients.pop(raddr, None)
                if raddr in self.clients:
                    self.clients.pop(raddr)
                continue

            self.clients[raddr] = current  # 只要有心跳数据或其他有效数据,刷新最后一次时间

            logging.info(data)
            logging.info(raddr)

            msg = "Ack {}. from {}:{}".format(data.upper(), *raddr).encode()
            for c, stamp in self.clients.items():
                if current - stamp > self.interval:
                    localset.add(c)
                else:
                    self.sock.sendto(msg, c)

            for c in localset:
                self.clients.pop(c, None)

    def stop(self):
        for c in self.clients:
            self.sock.sendto(b'Bye!', c)
        self.sock.close()
        self.event.set()

def main():
    cs = ChatUdpServer()
    cs.start()

    while True:
        cmd = input('>> ').strip()
        if cmd == 'quit':
            cs.stop()
            threading.Event().wait(3)
            break
        logging.info(threading.enumerate())
        logging.info(cs.clients)

if __name__ == '__main__':
    main()
```

客户端:

 

```
class ChatUdpClient:
    def __init__(self, rip='127.0.0.1', rport=9999, interval=10):
        self.raddr = (rip, rport,)
        self.sock = socket.socket(type=socket.SOCK_DGRAM)
        self.event = threading.Event()
        self.interval = interval

    def start(self):
        self.sock.connect(self.raddr)
        threading.Thread(target=self.hb, name='heartbeat').start()
        threading.Thread(target=self.recv, name='recv').start()

    def hb(self):
        while not self.event.wait(self.interval):  # 每interval检查一下event是否被set,没有则返回False
            self.send('^hb^')

    def recv(self):
        while not self.event.is_set():
            data, addr = self.sock.recvfrom(1024)
            logging.info(data)

    def send(self, msg:str):
        self.sock.sendto(msg.encode(), self.raddr)

    def stop(self):
        self.event.set()
        self.sock.close()

if __name__ == '__main__':
    cc1 = ChatUdpClient()
    cc1.start()
    while True:
        cmd = input(">> ").strip()
        if cmd == 'quit':
            cc1.send("quit")
            cc1.stop()
            break
        cc1.send(cmd)
```

## 模拟QQ聊天

由于udp无连接,所以可以同时多个客户端去跟服务端通信

 

```
# 服务端
import socket

ip_port = ('127.0.0.1', 8081)
udp_server_sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp_server_sock.bind(ip_port)

while True:
    qq_msg, addr = udp_server_sock.recvfrom(1024)
    print('来自[%s:%s]的一条消息:\033[1;44m%s\033[0m' % (addr[0], addr[1], qq_msg.decode('utf-8')))
    back_msg = input('回复消息: ').strip()

    udp_server_sock.sendto(back_msg.encode('utf-8'), addr)
    
# 客户端1
import socket
BUFSIZE = 1024
udp_client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

qq_name_dic = {
    '狗哥': ('127.0.0.1', 8081),
    '猴哥': ('127.0.0.1', 8081),
    '武大郎': ('127.0.0.1', 8081),
    '一棵树': ('127.0.0.1', 8081),
}

while True:
    qq_name = input('请选择聊天对象: ').strip()
    while True:
        msg = input('请输入消息,回车发送: ').strip()
        if msg == 'quit': break
        if not msg or not qq_name or qq_name not in qq_name_dic: continue
        udp_client_socket.sendto(msg.encode('utf-8'), qq_name_dic[qq_name])

        back_msg, addr = udp_client_socket.recvfrom(BUFSIZE)
        print('来自[%s:%s]的一条消息:\033[1;44m%s\033[0m' % (addr[0], addr[1], back_msg.decode('utf-8')))

        # udp_client_socket.close()
        
# 客户端2
import socket
BUFSIZE = 1024
udp_client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

qq_name_dic = {
    '狗哥': ('127.0.0.1', 8081),
    '猴哥': ('127.0.0.1', 8081),
    '武大郎': ('127.0.0.1', 8081),
    '一棵树': ('127.0.0.1', 8081),
}

while True:
    qq_name = input('请选择聊天对象: ').strip()
    while True:
        msg = input('请输入消息,回车发送: ').strip()
        if msg == 'quit': break
        if not msg or not qq_name or qq_name not in qq_name_dic: continue
        udp_client_socket.sendto(msg.encode('utf-8'), qq_name_dic[qq_name])

        back_msg, addr = udp_client_socket.recvfrom(BUFSIZE)
        print('来自[%s:%s]的一条消息:\033[1;44m%s\033[0m' % (addr[0], addr[1], back_msg.decode('utf-8')))

        # udp_client_socket.close()
```

运行结果:

服务端:

![img](socket%E7%BC%96%E7%A8%8Budp.assets/1f8170ad-7301-42d3-a6d5-8544a6136768.png)

客户端:

![img](socket%E7%BC%96%E7%A8%8Budp.assets/5cae0a07-8404-4678-bbb3-59c6cc37ca0c.png)

## 模拟时间服务器

 

```
# ntp服务端
from socket import *
from time import strftime

ip_port=('127.0.0.1',9000)
bufsize=1024

tcp_server=socket(AF_INET,SOCK_DGRAM)
tcp_server.bind(ip_port)

while True:
    msg,addr=tcp_server.recvfrom(bufsize)
    print('===>',msg)
    
    if not msg:
        time_fmt='%Y-%m-%d %X'
    else:
        time_fmt=msg.decode('utf-8')
    back_msg=strftime(time_fmt)

    tcp_server.sendto(back_msg.encode('utf-8'),addr)

tcp_server.close()

# ntp客户端
from socket import *
ip_port=('127.0.0.1',9000)
bufsize=1024

tcp_client=socket(AF_INET,SOCK_DGRAM)

while True:
    msg=input('请输入时间格式(例%Y %m %d)>>: ').strip()
    tcp_client.sendto(msg.encode('utf-8'),ip_port)

    data=tcp_client.recv(bufsize)

    print(data.decode('utf-8'))

tcp_client.close()
```

# UDP协议应用

UDP是无连接协议，它基于以下假设：

网络足够好

消息不会丢包

包不会乱序

但是，即使在局域网，也不能保证不丢包，而且包的到达不一定有序。

应用场景：

视频、音频传输，一般来说，丢些包，问题不大，最多丢些图像、听不清话语，可以重新发来解决。

海量采集数据，例如传感器发来的数据，丢几十、几百条数据也没有关系。DNS协议，数据内容小，一个包就能查询到结果，不存在乱序，丢包，重新请求解析。

一般来说，UDP性能优于TCP，但是可靠性要求高的场合还是建议选择TCP协议。



