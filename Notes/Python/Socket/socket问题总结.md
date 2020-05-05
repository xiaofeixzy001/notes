[TOC]

# 占用地址问题

服务端报错：

![img](socket%E9%97%AE%E9%A2%98%E6%80%BB%E7%BB%93.assets/7f19886c-d67c-45f6-9fb4-b83451b1d8c1.png)

这个是由于你的服务端仍然存在四次挥手的time_wait状态在占用地址

如果不懂，请深入研究：

1.tcp三次握手，四次挥手；

2.syn洪水攻击；

3.服务器高并发情况下会有大量的time_wait状态的优化方法；

解决的办法：在服务端加一条socket配置，重用ip和端口

 

```
phone=socket(AF_INET,SOCK_STREAM)
phone.setsockopt(SOL_SOCKET,SO_REUSEADDR,1) #就是它，在bind前加
phone.bind(('127.0.0.1',8080))
```

如果发现系统存在大量TIME_WAIT状态的链接，通过调整linux内核参数可解决：

 

```
# vi /etc/sysctl.conf
'''
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_fin_timeout = 30
'''
/sbin/sysctl -p
```

net.ipv4.tcp_syncookies = 1 表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；

net.ipv4.tcp_tw_reuse = 1 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；

net.ipv4.tcp_tw_recycle = 1 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。

net.ipv4.tcp_fin_timeout 修改系統默认的 TIMEOUT 时间

# 粘包问题

只有TCP有粘包现象，UDP永远不会粘包.

首先需要掌握一个socket收发消息的原理：

![img](socket%E9%97%AE%E9%A2%98%E6%80%BB%E7%BB%93.assets/270189cc-9648-46e9-b85a-be11f6f9644d.png)

发送端可以是一K一K地发送数据，而接收端的应用程序可以两K两K地提走数据，当然也有可能一次提走3K或6K数据，或者一次只提走几个字节

的数据，也就是说，应用程序所看到的数据是一个整体，或说是一个流（stream），一条消息有多少字节对应用程序是不可见的，因此TCP协

议是面向流的协议，这也是容易出现粘包问题的原因。而UDP是面向消息的协议，每个UDP段都是一条消息，应用程序必须以消息为单位提取数

据，不能一次提取任意字节的数据，这一点和TCP是很不同的。怎样定义消息呢？可以认为对方一次性write/send的数据为一个消息，需要明

白的是当对方send一条信息的时候，无论底层怎样分段分片，TCP协议层会把构成整条消息的数据段排序完成后才呈现在内核缓冲区。

例如基于tcp的套接字客户端往服务端上传文件，发送时文件内容是按照一段一段的字节流发送的，在接收方看了，根本不知道该文件的字节流

从何处开始，在何处结束

所谓粘包问题主要还是因为接收方不知道消息之间的界限，不知道一次性提取多少字节的数据所造成的。

此外，发送方引起的粘包是由TCP协议本身造成的，TCP为提高传输效率，发送方往往要收集到足够多的数据后才发送一个TCP段。若连续几次

需要send的数据都很少，通常TCP会根据优化算法把这些数据合成一个TCP段后一次发送出去，这样接收方就收到了粘包数据。

1，TCP（transport control protocol，传输控制协议）是面向连接的，面向流的，提供高可靠性服务。收发两端（客户端和服务器端）

都要有一一成对的socket，因此，发送端为了将多个发往接收端的包，更有效的发到对方，使用了优化方法（Nagle算法），将多次间隔较小

且数据量小的数据，合并成一个大的数据块，然后进行封包。这样，接收端，就难于分辨出来了，必须提供科学的拆包机制。 即面向流的通信

是无消息保护边界的。

2，UDP（user datagram protocol，用户数据报协议）是无连接的，面向消息的，提供高效率服务。不会使用块的合并优化算法，, 由于

UDP支持的是一对多的模式，所以接收端的skbuff(套接字缓冲区）采用了链式结构来记录每一个到达的UDP包，在每个UDP包中就有了消息头

（消息来源地址，端口等信息），这样，对于接收端来说，就容易进行区分处理了。 即面向消息的通信是有消息保护边界的。

3，tcp是基于数据流的，于是收发的消息不能为空，这就需要在客户端和服务端都添加空消息的处理机制，防止程序卡住，而udp是基于数据

报的，即便是你输入的是空内容（直接回车），那也不是空消息，udp协议会帮你封装上消息头，实验略

udp的recvfrom是阻塞的，一个recvfrom(x)必须对一个一个sendinto(y),收完了x个字节的数据就算完成,若是y>x数据就丢失，这意味

着udp根本不会粘包，但是会丢数据，不可靠

tcp的协议数据不会丢，没有收完包，下次接收，会继续上次继续接收，己端总是在收到ack时才会清除缓冲区内容。数据是可靠的，但是会粘

包

两种情况下会发生粘包。

1，发送端需要等缓冲区满才发送出去，造成粘包（发送数据时间间隔很短，数据了很小，会合到一起，产生粘包）

2，接收方不及时接收缓冲区的包，造成多个包接收（客户端发送了一段数据，服务端只收了一小部分，服务端下次再收的时候还是从缓冲区拿

上次遗留的数据，产生粘包） 

拆包的发生情况

当发送端缓冲区的长度大于网卡的MTU时，tcp会将这次发送的数据拆成几个数据包发送出去。

粘包的解决办法

问题的根源在于，接收端不知道发送端将要传送的字节流的长度，所以解决粘包的方法就是围绕:

如何让发送端在发送数据前，把自己将要发送的字节流总大小让接收端知晓

然后接收端来一个死循环接收完所有数据

服务端：

第一阶段：制作报头

定义字典，存储数据的长度

转换为固定字典格式的字符串，即报头长度

back_msg=res.stdout.read() 这是服务端将要返回给客户端的数据

head_dic={'head_size': len(back_msg)}  定义一个字典，用于记录将要返回的数据的长度

head_json=json.dumps(head_dic)  将字典序列化

head_bytes=head_json.encode('utf-8')  通过utf-8转换为bytes格式

第二阶段：发送报头的长度

先把报头长度发送回客户端

conn.send(struct.pack('i',len(head_bytes)))  先发送head_bytes的长度给客户端

第三阶段：发报头

conn.send(head_bytes)  在发送head_bytes数据

第四阶段：发真实数据

conn.sendall(back_msg)  最后发送数据给客户端

客户端：

第一阶段：收取报头的长度

head=client.recv(4)

head_size=struct.unpack('i', head)[0]

第二阶段:根据报头的长度，来收取报头

head_bytes=client.recv(head_size)

head_json=head_bytes.decode('utf-8')  解码

head_dic=json.loads(head_json)  反序列化

data_size=head_dic['data_size']

第三阶段：根据获取到的数据的长度来循环收取真实数据

recv_size=0  #定义接收的数据长度

recv_bytes=b''

while recv_size < data_size:  #判断客户端要接收的数据长度是否小于数据真实的长度 

  res=client.recv(1024)

  recv_bytes = recv_bytes + res  #字符串拼接

  recv_size= recv_size + len(res)

print(recv_bytes.decode('gbk'))

**struct模块**

该模块可以把一个类型，如数字，转成固定长度的bytes

\>>> struct.pack('i',1111111111111)

struct.error: 'i' format requires -2147483648 <= number <= 2147483647 #这个是范围

![img](socket%E9%97%AE%E9%A2%98%E6%80%BB%E7%BB%93.assets/495b3cbf-295f-41b4-a9ad-63f5ffca8cea.jpg)

关于struct的详细用法

 

```
import json,struct
# 假设通过客户端上传1T:1073741824000的文件a.txt

# 为避免粘包,必须自定制报头
header={'file_size':1073741824000,'file_name':'/a/b/c/d/e/a.txt','md5':'8f6fbf8347faa4924a76856701edb0f3'} #1T数据,文件路径和md5值

# 为了该报头能传送,需要序列化并且转为bytes
head_bytes=bytes(json.dumps(header),encoding='utf-8') #序列化并转成bytes,用于传输

# 为了让客户端知道报头的长度,用struck将报头长度这个数字转成固定长度:4个字节
head_len_bytes=struct.pack('i',len(head_bytes)) #这4个字节里只包含了一个数字,该数字是报头的长度

# 客户端开始发送
conn.send(head_len_bytes) #先发报头的长度,4个bytes
conn.send(head_bytes) #再发报头的字节格式
conn.sendall(文件内容) #然后发真实内容的字节格式

# 服务端开始接收
head_len_bytes=s.recv(4) #先收报头4个bytes,得到报头长度的字节格式
x=struct.unpack('i',head_len_bytes)[0] #提取报头的长度

head_bytes=s.recv(x) #按照报头长度x,收取报头的bytes格式
header=json.loads(json.dumps(header)) #提取报头

# 最后根据报头的内容提取真实的数据,比如
real_data_len=s.recv(header['file_size'])
s.recv(real_data_len)
```

\-----

 

```
#_*_coding:utf-8_*_
#http://www.cnblogs.com/coser/archive/2011/12/17/2291160.html
__author__ = 'Linhaifeng'
import struct
import binascii
import ctypes

values1 = (1, 'abc'.encode('utf-8'), 2.7)
values2 = ('defg'.encode('utf-8'),101)
s1 = struct.Struct('I3sf')
s2 = struct.Struct('4sI')

print(s1.size,s2.size)
prebuffer=ctypes.create_string_buffer(s1.size+s2.size)
print('Before : ',binascii.hexlify(prebuffer))
# t=binascii.hexlify('asdfaf'.encode('utf-8'))
# print(t)


s1.pack_into(prebuffer,0,*values1)
s2.pack_into(prebuffer,s1.size,*values2)

print('After pack',binascii.hexlify(prebuffer))
print(s1.unpack_from(prebuffer,0))
print(s2.unpack_from(prebuffer,s1.size))

s3=struct.Struct('ii')
s3.pack_into(prebuffer,0,123,123)
print('After pack',binascii.hexlify(prebuffer))
print(s3.unpack_from(prebuffer,0))
```

我们可以把报头做成字典，字典里包含将要发送的真实数据的详细信息，然后json序列化，然后用struck将序列化后的数据长度打包成4个字

节（4个自己足够用了）

发送时：

先发报头长度

再编码报头内容然后发送

最后发真实内容

接收时：

先手报头长度，用struct取出来

根据取出的长度收取报头内容，然后解码，反序列化

从反序列化的结果中取出待取数据的详细信息，然后去取真实的数据内容

解决粘包问题示例：

服务端：

 

```
import socket
import subprocess
import struct
import json
phone=socket.socket(socket.AF_INET,socket.SOCK_STREAM) # 买手机
phone.setsockopt(socket.SOL_SOCKET,socket.SO_REUSEADDR,1)
phone.bind(('127.0.0.1',8080)) # 插电话卡
phone.listen(5) # 开机，backlog
while True:
    print('starting....')
    conn,addr=phone.accept()
    print('cliet addr',addr)
    while True:
        try:
            cmd=conn.recv(1024)
            if not cmd:break
            res=subprocess.Popen(cmd.decode('utf-8'),shell=True,
                             stdout=subprocess.PIPE,
                             stderr=subprocess.PIPE)
            err=res.stderr.read()
            if err:
                cmd_res=err
            else:
                cmd_res=res.stdout.read()

            # conn.send(struct.pack('i',len(cmd_res))) # 先报报头
            head_dic={'filename':None,'hash':None,'total_size':len(cmd_res)}
            head_json=json.dumps(head_dic)
            head_bytes=head_json.encode('utf-8')

            # 先发送报头的长度
            conn.send(struct.pack('i',len(head_bytes)))

            # 再发送报头的bytes
            conn.send(head_bytes)

            # 最后发送真实的数据
            conn.send(cmd_res)

        except Exception:
            break
    conn.close()
phone.close()
```

客户端：

 

```
#!/usr/bin/python
# -*- coding:utf-8 -*-

import socket
import struct
import json
phone=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
phone.connect(('127.0.0.1',8080)) # 拨通电话

while True: # 通信循环
    cmd=input('>>: ').strip()
    if not cmd:continue # 防止客户端发空
    phone.send(cmd.encode('utf-8')) # 发消息

    # 先收报头的长度
    head_struct=phone.recv(4)
    head_len=struct.unpack('i',head_struct)[0]

    # 再收报头的bytes
    head_bytes=phone.recv(head_len)
    head_json=head_bytes.decode('utf-8')
    head_dic=json.loads(head_json)

    # 最后根据报头里的详细信息取真实的数据
    print(head_dic)
    total_size=head_dic['total_size']
    recv_size=0
    data=b''
    while recv_size < total_size: # 10240 +1
        recv_data=phone.recv(1024)
        data+=recv_data
        recv_size+=len(recv_data)
    print(data.decode('gbk'))
phone.close()
```



