[TOC]

## 第1章 Squid介绍

### 1.1 缓存服务器介绍

> - 缓存服务器（英文意思cache  server）,即用来存储（介质为内存及硬盘）用户访问的网页，图片，文件等等信息的专用服务器。这种服务器不仅可以使用户可以最快的得到他们想要的信息，而且可以大大减少服务端网络传输的数据量。缓存服务器往往也是代理服务器。对于网站的用户来说，缓存服务器和代理是不可见的，即在用户看来所有的网站信息都来自其正在访问的网站，而实际上可能是缓存服务器在提供访问数据。
> - 目前国内互联网公司常用的缓存服务器有：squid，varnish（几乎绝迹），nginx，ats。
> - squid作为缓存和代理服务器的历史十分的悠久，本章我们主要讲述squid服务，尽管不少人谈及其他软件的缓存机制比squid如何优异，但我们还是要首先掌握这个老牌的开源cache软件，因为它经历的历程实在是太悠久了，等大家掌握了squid服务后，其他的软件就不在话下了。如果再有时间，可以测试下varnish，nginx，squid三者之间的性能对比，而不是轻易的听信于他人的说法，别人说好，只能代表他个人的观点，我们自己用事实证明，才是学习和工作的真谛。
> - 国内基本上90%以上的商业CDN公司，象国内的CDN基本都在用squid，象蓝汛，网宿，帝联，sina在用ats。
>    Squid官方：http://www.squid-cache.org/

### 1.2 Web缓存相关概念

#### 1.2.1 cache命中

> cache命中是在cache server  每次从它的缓存里满足客户端HTTP请求时发生。cache命中率，是所有客户端HTTP请求中命中的比例。Web缓存典型的cache命中率在30%到60%之间。另一个相似的度量单位叫做字节命中率，描绘了cache提供服务的数据容量（字节数），如何提升cache命中率？

1）apache nginx 可以expries，cache-control缓存头
 2）动静分离，静态化，对静态走CDN
 3）mysql cache让缓存靠前
 4）4XX，5XX之类错误页面，死链不缓存。

#### 1.2.2 cache丢失

> cache丢失在cache server不能从它的缓存里满足客户端HTTP请求时发生。cache丢失的原因有很多种。

1）当cache server第一次接收到对第一个新资源的请求时，就会产生一个cache丢失。**如何解决第一次命中？**

**预热或者预取。**
 a，内部先请求访问。可以通过脚本实现（这是个思路但不靠谱）。
 b，后端生成数据之后，统一推到前端cache server。即预取，预热。

2）存储空间满或者对象自身过期，cache server会清除这些缓存对象以释放空间给新对象。

a，加大内存或者磁盘。
 b，过期时间设置的长一些。
 c，参数设置，缓存的参数设置大一些。最大缓存对象2M（热点缓存）。
 d，分资源缓存，1M，10M，100M（分拆服务器，acl 正则匹配抛给不同的pools）

3）还有可能是客户访问的资源不可到达。原始服务器会指示cache server 怎样处理用户响应。例如，它会提示数据不能被缓存，或在有限的时间内才被重复使用等等

#### 1.2.3 cache确认

> - 对于缓存来讲，数据的一致性是一个特别头疼的问题，特别是memcached。
> - cache确认保证cache server不对访问的用户返回过期的数据。在重复使用缓存对象时，cache server需要经常从原始服务器确认它。假如服务器指示squid的拷贝仍然有效，数据就发送出去。否则，squid更新它的缓存拷贝，并且转发给客户。
> - 当用户更新了数据到数据库或者存储服务器的时候，可以从业务角度主动调用接口清除该对象缓存的指令。CDN 5-15分钟。
> - 图片放到CDN了需要更新吗？不需要更新。图片修改算更新，这样的业务就要推送。
> - 网站改版：再CDN上推送JS，css（改名）

from：http://home.arcor.de/mailerstar/jeff/squid/chap01.html#al

### 1.3 squid服务介绍

> - Squid是一个高性能的代理缓存服务器，Squid支持FTP，gopher和HTTP协议。和一般的代理缓存软件不同，Squid用一个单独的，非模块化的，I/O驱动的进程来处理所有的客户端请求。
> - Squid将数据元缓存在内存和硬盘中，同时也缓存DNS查询的结果。Squid支持SSL，支持访问控制。由于使用了ICP（轻量Internet缓存协议），Squid能够实现层叠的代理阵列，从而最大限度的节约带宽。
> - Squid Cache（简称Squid）是一个流行的代理服务武器和Web缓存服务器软件。Squid服务有相当多的用途：

1. 用于放置在Web服务器的前面，缓存网站Web服务器的相关数据，这样用户请求缓存服务器就可以直接返回数据给用户了，从而提升了用户的访问网站体验，从另一方面也减轻了Web服务器，数据库服务器，图片文件存储服务器等业务服务器的压力。这种应用被称之为反向代理服务。
2. 用于放置在企业内部关键出网位置或者某些共享网络的前端，缓存内部上网用户的数据，域名系统和其他网络搜索数据等，这样用户上网请求的数据，就可以由缓存服务器返回给内部用户，而不需要上网了，从而使得内部用户上网更快，更安全，也会大大节约公司的带宽。这种应用被称之为正向代理服务（普通代理或者透明代理）。
3. 通过放在网络的关键位置过滤网络流量和访问数据，提升整个网络安全。例如：可以监控及限制内部企业员工的上网行为，可以和iptables配合作为办公网的网关。
4. 用作局域网通过代理上网，只要是一台可以上网的机器就可以，位置随便，让所有的用户的浏览器设置这个服务器代理上网即可。

> Squid代理服务器主要用于类Unix系统中运行，其发展历史相当悠久，功能也相当完善。除了对HTTP支持的很好外，对于FTP与HTTPS的支持也相当好，在3.0测试版中也支持了IPv6，Squid的主页在http://www.squid-cache.org。目前业界主流CDN都是基于Squid进行二次开发作为cache缓存服务器的。

#### 1.3.1 传统代理服务原理

**传统的代理服务器就是前面我们所说的通过浏览器设置代理的方法：**

![QQ截图20170916222044.png-29.7kB](http://static.zybuluo.com/chensiqi/vc0pqtsrkvasph3wsiikgsjq/QQ%E6%88%AA%E5%9B%BE20170916222044.png)

**windows如何设置代理？**

![QQ截图20170916223042.png-143.8kB](http://static.zybuluo.com/chensiqi/xhvz7sm36iaqf2zt70ip8b2i/QQ%E6%88%AA%E5%9B%BE20170916223042.png)

#### 1.3.2 透明代理服务原理

> - 所谓透明代理，是相对于代理服务器而言，客户端不需要做任何和代理服务器相关的设置和操作，对用户而言，感觉不到代理服务器的存在，所以称之为透明代理。即把代理服务器部署在核心的上网出口，当用户上网浏览页面时，会交给代理服务器向外请求，如果结合iptables可以实现代理+网关+内容过滤+流量安全控制等完整的上网解决方案。

**透明代理流程说明：**

> 用户A发送一个访问请求到防火墙，由防火墙将该用户的访问请求转发到SQUID，SQUID在先检查自身缓存中有无该用户请求的访问内容，如果没有，则请求远端目的服务器，获取该用户的访问内容，在返回给用户的同时，在自身缓存保留一份记录以备下次调用；当用户B发送一个和用户A相同的访问请求时，由防火墙将转发该用户请求到SQUID，SQUID检查自身缓存发现有同样内容后，直接将该内容返回给用户。

![QQ截图20170916225025.png-13.2kB](http://static.zybuluo.com/chensiqi/tbuy9lzs6w4riknxvhqw7j3m/QQ%E6%88%AA%E5%9B%BE20170916225025.png)

#### 1.3.3 反向代理服务原理

> - 普通代理方式是代理内部网络用户访问internet上服务器的连接请求，客户端必须指定代理服务器，并将本来要直接发送到internet上服务器的连接请求发送给代理服务器处理。反向代理（Reverse  Proxy）方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从内部服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

**反向代理流程说明：**

> - SQUID作为反向代理服务器，通常工作在一个服务器集群的前端，在用户端看来，SQUID服务器就是他说要访问的服务器，而实际意义上SQUID只是接受用户的请求，同时将用过户请求转发给内网真正的WEB服务器，如果SQUID本身有用户要访问的内容，则SQUID直接将数据返回给用户，起到了缓存数据的作用，减少了后端服务的压力。

![QQ截图20170916230501.png-31.3kB](http://static.zybuluo.com/chensiqi/d0qeberzvu003suwtpgfsmtv/QQ%E6%88%AA%E5%9B%BE20170916230501.png)

#### 1.3.4 三种代理服务器对比

![QQ截图20170916231608.png-22.5kB](http://static.zybuluo.com/chensiqi/mt9w6ylhr9llmnc2x5qu4dll/QQ%E6%88%AA%E5%9B%BE20170916231608.png)

**问：网站什么时候就需要用squid（CDN）了？**

> 静态抗不住了，想节省带宽，节省成本，想提高访问速度

a，节省带宽及服务器成本。
 b，提升用户体验。
 c，源站抗不住了。

#### 1.3.5 淘宝最新CDN架构图

淘宝：

![QQ截图20170916234751.png-65.9kB](http://static.zybuluo.com/chensiqi/tii6yk6bqknw8ej7cyr8g373/QQ%E6%88%AA%E5%9B%BE20170916234751.png)

#### 1.3.6 如何选择squid服务的版本

> - 目前主流使用的Squid缓存服务，大公司，2.7是最多的，基本上90%以上的商业CDN公司，例如国内的CDN，蓝汛，网宿，帝联都在用squid2.7，squid3.0使用C++重写后，性能上和Squid 2.6和2.7还是有些距离的。使用的人并不是很多，性能稳定性等还有必要在等等看。

## 第2章 安装squid硬件和系统要求

### 2.1 操作系统环境

> Squid可以运行在几乎所有的常见Unix及Linux系统上，也可以在Microsoft  Windows上运行。尽管squid的Windows支持在不断改进，但在Unix及Linux系统上运行Squid依然是更简单，安全，更有效率，本章我们就使用Centos6.4 x86_64来运行Squid。

### 2.2 服务器硬件环境

1）第一重要资源：内存

> squid对硬件的要求最主要的是内存资源。内存短缺会严重影响性能。因为所有的对象都会尽可能的被缓存到内存中，这样才能更快的提升用户的响应及返回数据。

2）第二重要资源：磁盘

> 磁盘空间也是另一个squid能够高效运行的重要因素。更多的磁盘空间意味着更多的缓存目标和更高的命中率。快速的磁盘介质也是必要的。例如：用ssd，sas替代sata磁盘，除了使用过raid外，可以指定多个磁盘路径缓存。

3）其他：磁盘与内存的关联

> 因为squid对每个缓存响应使用少数内存，因此在磁盘空间和内存要求之间有一定联系。基本规则是，每G磁盘空间需要32M内存。这样，512M内存的系统，能支持16G的磁盘缓存。你的情况当然会不同。内存需求依赖于如下事实：缓存目标大小，CPU体系（32位或64位），同时在线的用户数量，和你使用的特殊功能。

### 2.3 虚拟服务器硬件环境

内存：512M
 硬盘：8-10G
 VM：1-2个，其中一个部署缓存服务器，一个部署web服务器做测试用。
 系统：Centos6.5 x86_64

### 2.4 虚拟服务器实施部署前主机规划列表

| 名称         | 接口 | IP            | 用途     |
| ------------ | ---- | ------------- | -------- |
| Squid server | eth0 | 192.168.0.190 | Squid    |
| Web server   | eth0 | 192.168.0.220 | nginxWeb |

## 第3章 squid编译与安装

### 3.1 squid下载与解压

1）下载squid软件

```
[root@localhost ~]# wget http://www1.it.squid-cache.org/Versions/v3/3.0/squid-3.0.STABLE20.tar.gz
[root@localhost ~]# ls -l squid-3.0.STABLE20.tar.gz 
-rw-r--r--. 1 root root 2452224 Oct 29  2009 squid-3.0.STABLE20.tar.gz
```

2)解开源代码包

```
[root@localhost ~]# tar xf squid-3.0.STABLE20.tar.gz -C /usr/src/
[root@localhost ~]# cd /usr/src/squid-3.0.STABLE20/
[root@localhost squid-3.0.STABLE20]# pwd
/usr/src/squid-3.0.STABLE20
```

### 3.2 squid编译前关键内核参数调整

#### 3.2.1 调整文件描述符

> Squid在高负载下，需要大量的内核资源。特别的，你需要给你的系统配置比正常情况更高的文件描述符和缓存，最好在开始编译squid之前来增加文件描述符的大小（在系统安装时我们已经讲解过）。squid和内核通过数据结构来交换信息，数据结构的大小不能超过设置的文件描述符的限制。squid在运行时检查这些设置，并且使用最安全的（最小的）值。

**文件描述符：**

> - 文件描述符是一个简单的整数，用以标明每一个被进程所打开的文件和socket。第一个打开的文件是0，第二个是1，依此类推。Unix操作系统通常给每个进程能打开的文件数量强加一个限制。更甚的是，unix通常有一个系统级的限制（1024）.因为squid的工作方式，文件描述符的限制可能会极大的影响性能。当squid用完所有的文件描述符后，它不能接收用户新的连接。也就是说，用完文件描述符导致拒绝服务。直到一部分当前请求完成，相应的文件和socket被关闭，squid不能接收新请求。当squid发现文件描述符短缺时，它会发布警告。
> - 在运行./configure之前，检查你的系统的文件描述符限制是否合适，能给你避免一些麻烦。大多数情况下，1024个文件描述符足够了。非常忙的cache可能需要4096或更多。在配置文件描述符限制时，我推荐设置系统级限制的数量为每个进程限制的2倍。

```
[root@localhost squid-3.0.STABLE20]# cd ~
[root@localhost ~]# ulimit -n
1024

#1024这是linux系统默认情况的值
```

**设置打开的最大文件描述符的数目**

```
[root@localhost squid-3.0.STABLE20]# cd ~
[root@localhost ~]# ulimit -n
1024
[root@localhost ~]# ulimit -n 20480 #记得在squid运行前设置好该参数
[root@localhost ~]# ulimit -n
20480
[root@localhost ~]# echo "ulimit -n 20480" >> /etc/rc.local
[root@localhost ~]# tail -1 /etc/rc.local
ulimit -n 20480
```

#### 3.2.2 调整临时端口范围

> - 临时端口是TCP/IP栈分配给出去连接的本地端口。换句话说，当squid发起一条连接到另一台服务器，内核给本地socket分配一个端口号。这些本地端口号有特定的范围限制。
> - 例如，Centos默认是32768-61000.
> - 临时端口号的短缺对非常慢的代理服务器（例如每秒数百个连接）来说，会较大的影响性能。这是因为一些TCP连接在他们被关闭时进入TIME_WAIT状态。当连接进入TIME_WAIT状态时，临时端口号不能被重用。

**调整临时端口范围方法：**

```
[root@localhost ~]# cat /proc/sys/net/ipv4/ip_local_port_range 
32768   61000
[root@localhost ~]# echo "net.ipv4.ip_local_port_range = 4000 65000" >> /etc/sysctl.conf 
[root@localhost ~]# sysctl -p
net.ipv4.ip_forward = 0
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
kernel.sysrq = 0
kernel.core_uses_pid = 1
net.ipv4.tcp_syncookies = 1
error: "net.bridge.bridge-nf-call-ip6tables" is an unknown key
error: "net.bridge.bridge-nf-call-iptables" is an unknown key
error: "net.bridge.bridge-nf-call-arptables" is an unknown key
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmmax = 68719476736
kernel.shmall = 4294967296
net.ipv4.ip_local_port_range = 4000 65000
[root@localhost ~]# cat /proc/sys/net/ipv4/ip_local_port_range 
4000    65000
```

### 3.3 squid编译前查看帮助

官方：http://www.squid-cache.org/Versions/v3/3.0/cfgman/

1）进入解压后的squid目录

```
[root@localhost squid-3.0.STABLE20]# more INSTALL   #查看编译帮助
To build and install the Squid Cache, type:

    % ./configure --prefix=/usr/local/squid     #安装过程总共3步
        % make all
        % make install

To run a Cache, you will need to:

    1. customize the squid.conf configuration file:
          % vi /usr/local/squid/etc/squid.conf      #配置文件

    2. Initalise the cache:
          % /usr/local/squid/sbin/squid -z          #初始化缓存
          
    3. start the cache:
          % /usr/local/squid/sbin/squid             #启动squid

If you want to use the WWW interface to the Cache Manager, copy
the cachemgr.cgi program into your httpd server's cgi-bin
directory.
```

### 3.4 squid编译安装

```
#先安装依赖
yum -y install openssl-devel

##配置编译环境
[root@localhost squid-3.0.STABLE20]# ./configure --prefix=/usr/local/squid3 --enable-async-io=100 --with-pthreads --enable-storeio="aufs,diskd,ufs" --enable-removal-policies="heap,lru" --enable-icmp --enable-delay-pools --enable-useragent-log --enable-referer-log --enable-kill-parent-hack --enable-cachemgr-hostname=localhost --enable-arp-acl --enable-default-err-language=English --enable-err-languages="Simplify_Chinese English" --disable-poll --disable-wccp --disable-wccpv2 --disable-ident-lookups --disable-internal-dns --enable-basic-auth-helpers="NCSA" --enable-stacktrace --with-large-files --disable-mempools --with-filedescriptors=64000 --enable-ssl --enable-x-acceletator-vary --disable-snmp --with-aio --enable-linux-netfilter --enable-linux-tproxy
#编译
[root@localhost squid-3.0.STABLE20]# make
#安装
[root@localhost squid-3.0.STABLE20]# make install
#把安装目录做成软连接
[root@localhost ~]# ln -s /usr/local/squid3/ /usr/local/squid
[root@localhost ~]# ll -d /usr/local/squid
lrwxrwxrwx. 1 root root 18 Sep 17 20:02 /usr/local/squid -> /usr/local/squid3/
```

### 3.5 squid目录文件结构介绍

> 在安装完后，将在squid的安装目录里（/usr/local/squid/）会看到下列目录和文件：

```
[root@localhost squid]# tree -L 2 /usr/local/squid
/usr/local/squid
├── bin
│   ├── RunAccel
│   ├── RunCache
│   └── squidclient
├── etc
│   ├── cachemgr.conf
│   ├── cachemgr.conf.default
│   ├── mime.conf
│   ├── mime.conf.default
│   ├── squid.conf
│   └── squid.conf.default
├── libexec
│   ├── cachemgr.cgi
│   ├── diskd
│   ├── dnsserver
│   ├── ncsa_auth
│   ├── pinger
│   └── unlinkd
├── sbin
│   └── squid
├── share
│   ├── errors
│   ├── icons
│   ├── man
│   └── mib.txt
└── var
    └── logs

10 directories, 17 files
```

**为了让同学们理解的更清楚明白，我们把这些内容列成了如下表格：**

| 文件名/目录名            | 功能描述                                                     |
| ------------------------ | ------------------------------------------------------------ |
| sbin                     | squid主从程序的目录，正常只能被root启动                      |
| sbin/squid               | Squid的主程序                                                |
| bin                      | bin目录包含对所有用户可用的程序                              |
| bin/RunCache             | RunCache是一个shell脚本，你能用它来启动squid。假如squid死掉，该脚本自动重启它，除非它检测到经常的重启 |
| bin/RunAccel             | RunAccel与RunCache几乎一致，唯一不同是它增加了一个命令行参数，告诉squid在哪里侦听HTTP请求 |
| bin/squidclient          | squidclient是个简单的HTTP客户端程序，你能用它来测试squid。它也有一些特殊功能，用以对运行的squid进程发起管理请求。 |
| libexec                  | libexec目录包含了辅助程序。有一些命令你不能正常的启动。然而，这些程序通常被其他程序启动 |
| libexec/unlinkd          | unlinkd是一个辅助程序，它从cache目录里删除文件               |
| libexec/cachemgr.cgi     | cachemgr.cgi是Squid管理功能的CGI接口。为了使用它，你需要拷贝该程序到你的WEB服务器的cgi-bin目录 |
| libexec/diskd(optional)  | 假如你指定了--enable-storeio=diskd,你才能看到它              |
| libexec/pinger(optional) | 假如你指定了--enable-icmp，你才能看到它                      |
| etc                      | etc目录包含squid的配置文件                                   |
| etc/squid.conf           | 这是squid的主配置文件                                        |
| var                      | var目录包含了不是很重要的和经常变化的文件。这些文件不必正常的备份他们 |
| var/logs                 | var/logs目录是squid不同日志文件的默认位置。当你第一次安装squid时，它是空的。一旦squid开始运行，你能在这里看到名字为access.log,cache.log和store.log这样的文件 |
| var/cache                | 假如你不在squid.conf文件里指定，这是默认的缓存目录（cache_dir） |

> 参考：http://home.arcor.de/mailerstar/jeff/squid/chap03.html

## 第4章 squid配置介绍

### 4.1 squid.conf语法

> - Squid的配置文件相对规范。它与其他许多unix程序相似。每行以配置指令开始，后面跟着数字值或关键字。在读取配置文件时，squid忽略空行和注释掉的行（以#开始）。
> - 默认的squid.conf内容有相当多的内容，如下：

```
[root@localhost ~]# cd /usr/local/squid/etc/
[root@localhost etc]# wc -l squid.conf
4863 squid.conf

#去掉以#开头的注释和空行（以#开头！）
[root@localhost etc]# egrep -v "^#|^$" squid.conf.default > squid.conf
[root@localhost etc]# cat squid.conf | wc -l
37
[root@localhost etc]# cat squid.conf
acl manager proto cache_object
acl localhost src 127.0.0.1/32
acl to_localhost dst 127.0.0.0/8 0.0.0.0/32
acl localnet src 10.0.0.0/8 # RFC1918 possible internal network
acl localnet src 172.16.0.0/12  # RFC1918 possible internal network
acl localnet src 192.168.0.0/16 # RFC1918 possible internal network
acl SSL_ports port 443
acl Safe_ports port 80      # http
acl Safe_ports port 21      # ftp
acl Safe_ports port 443     # https
acl Safe_ports port 70      # gopher
acl Safe_ports port 210     # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280     # http-mgmt
acl Safe_ports port 488     # gss-http
acl Safe_ports port 591     # filemaker
acl Safe_ports port 777     # multiling http
acl CONNECT method CONNECT
http_access allow manager localhost
http_access deny manager
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localnet
http_access deny all
icp_access allow localnet
icp_access deny all
htcp_access allow localnet
htcp_access deny all
http_port 3128
hierarchy_stoplist cgi-bin ?
access_log /usr/local/squid3/var/logs/access.log squid
refresh_pattern ^ftp:       1440    20% 10080
refresh_pattern ^gopher:    1440    0%  1440
refresh_pattern (cgi-bin|\?)    0   0%  0
refresh_pattern .       0   20% 4320
icp_port 3130
coredump_dir /usr/local/squid3/var/cache
```

### 4.2 squid服务的用户

> - 几乎所有的unix进程和文件拥有文件的组和属主的属性，你必须创建一个用户和组给squid服务，该用户和组的组合，必须对大部分squid相关的文件和目录有读和写的权限，所以需要创建“squid”的用户和组，这避免了某人利用squid来读取系统中的其他文件。
> - 运行squid必须以root身份运行，设置配置文件squid.conf中。cache_effective_user为squid来运行，这个用户和组的名称理论上可以任意起。

1）创建squid用户和组，禁止其登陆

```
[root@localhost etc]# useradd -s /sbin/nologin -M squid
```

2）编辑配置文件squid.conf

```
#设置启动账户为squid
[root@localhost etc]# echo "cache_effective_user squid" >> /usr/local/squid/etc/squid.conf
[root@localhost etc]# tail -1 /usr/local/squid/etc/squid.conf
cache_effective_user squid

#设置启动账户组为squid
[root@localhost etc]# echo "cache_effective_group squid" >> /usr/local/squid/etc/squid.conf
[root@localhost etc]# tail -1 /usr/local/squid/etc/squid.conf
cache_effective_group squid

#开启store日志
[root@localhost etc]# echo "cache_store_log /usr/local/squid/var/logs/store.log" >> /usr/local/squid/etc/squid.conf
[root@localhost etc]# tail -1 /usr/local/squid/etc/squid.conf
cache_store_log /usr/local/squid/var/logs/store.log     #缓存对象日志

#开启cache日志
[root@localhost etc]# echo "cache_log /usr/local/squid/var/logs/cache.log" >> /usr/local/squid/etc/squid.conf
[root@localhost etc]# tail -1 /usr/local/squid/etc/squid.conf
cache_log /usr/local/squid/var/logs/cache.log

#开启磁盘缓存cache_dir
[root@localhost etc]# echo "cache_dir ufs /usr/local/squid/var/cache 100 16 256" >> /usr/local/squid/etc/squid.conf
[root@localhost etc]# tail -1 /usr/local/squid/etc/squid.conf
cache_dir ufs /usr/local/squid/var/cache 100 16 256
```

### 4.3 squid端口号

> - http_port指令告诉squid在哪个端口侦听HTTP请求。默认端口是3128：
> - 假如你计划将squid作为web服务加速器运行，可以将该端口设置为80.
>    可以使用附加的http_port行，来指示squid侦听在多个端口上，例如，来自某个部门的浏览器发送请求到3128，然而另一个部门使用80端口。简单的将两个端口号列举出来：

| http_port | 3128 |
| --------- | ---- |
| http_port | 8080 |

> 也可以使用http_port指令来使squid侦听在指定的接口地址上，当squid作为防火墙运行时，它有两个网络接口：一个内部的和一个外部的，为了避免来自外部的http请求，使squid仅仅侦听在内部接口上，简单的将IP地址放在端口号的前面：

```
#squid 仅监听内网地址，拒绝外部Http请求访问
http_port 192.168.1.1:3128
```

### 4.4 squid日志文件

> - squid默认的日志目录是squid安装位置下的logs目录。例如，假如你在./configure中没有使用--prefix=选项，那么默认的日志文件路径是/usr/local/squid/var/logs，必须确认日志文件所存放的磁盘位置空间足够。在squid写日志时如果接受到错误，它会退出和重启。该行为的主要理由应引起你的注意，squid想确认你不会丢失任何重要的日志信息，特别是你的系统被滥用或者被攻击时。

squid有三个主要的日志文件：cache.log,access.log,store.log。

#### 4.4.1 cache.log日志文件

> cache.log包含多种消息，例如Squid的配置信息，性能警告，以及严重错误。如下是cache.log的输出样本：主要的错误和异常条件最可能报告在cache.log里。

```

```

> 刚开始运行squid时，需要密切关注该文件。假如squid拒绝运行，原因也许会出现在cache.log文件的结尾处。在正常条件下，该文件不会变得很大。假如你以-s选项来运行squid，重要的cache.log信息也可被送到你的rsyslog进程。通过使用cache_log指令，你以修改配置文件squid.conf来改变该日志文件的路径。

#### 4.4.2 转发cache.log消息到系统日志

> 为了让squid发送cache.log消息的拷贝到系统日志，请使用-s命令行选项。仅仅在debug级别0和1的消息会被转发。

#### 4.4.3 access.log日志文件

> - Squid把关于HTTP事务的关键信息存放在access.log里。该文件是基于行的，也就是说每行对应一个客户端请求。squid记录客户端IP（或主机名），请求URL，响应size等其他信息。
> - Squid在access.log里记录所有HTTP访问，除了那些在还没有发送数据前就断开的连接。Squid也记录所有的ICP（非HTCP）事务，除非你使用log_icp_queries指令关闭了这个功能。
> - 默认的access.log格式包含了10个域。如下是日志样本，长行分割并且缩进排版。
> - access.log文件记录了对squid发起的每个客户请求。每行平均约150个字节，也就是说，在接受一百万条客户请求后，它的体积约是150M。可以使用cache_access_log指令改变该日志文件的路径：
> - 如果不想squid记录客户端请求日志，修改日志文件的路径为/dev/null即可。

#### 4.4.4 store.log日志文件

> - store.log记录Squid关于存储或删除cache目标的决定。对每个存在cache里的目标每个不可cache的目标，以及每个被轮换策略删除的目标，Squid都会创建相应的日志条目。该日志文件内容既包含了内存cache，又包含了磁盘cache。
> - store.log文件对大多数cache管理员来说并非很有用，但是我们可以通过这个日志来解析客户端访问的数据是否被缓存，它包含了进入和离开缓存的每个目标的记录。使用cache_store_log指令来改变它的位置：
> - 通过指定路径为none，你能轻易的完全禁止store.log日志：
> - Squid的日志文件增加没有限制...为了保证日志文件大小合理，应创建计划任务来规律的重命名和打包日志。squid有内建的日志回滚功能，也可以避免单个日志过于庞大。

### 4.5 squid的访问控制

> ACL元素是Squid的访问控制基础。这里会告诉你如何指定包括IP地址，端口号，主机名，和URL匹配等变量。每个ACL元素有个名字，在编写访问控制规则时需要引用他们。
>
> - 基本的ACL元素语法如下：

```
acl name type value1 value2 ...

#例如：
acl Workstations src 10.0.0.0/16     #表示源地址匹配10.0.0.0/16网段
```

**在多数情况下，你能对一个ACL元素列举多个值。你也可以有多个ACL行。例如，下列两行配置是等价的：**

```
acl Http_ports port 80 8000 8080
#提示：三个端口是或的关系，or
```

**上面一行与下面三行等价**

```
acl Http_ports port 80
acl Http_ports port 8000
acl Http_ports port 8080
```

#### 4.5.1 IP地址的acl定义

**使用对象：src，dst,myip**

> squid在ACL里指定IP地址时，拥有强有力的语法。你能以子网，地址范围形式编写地址。squid支持标准IP地址写法（由“.”连接的4个小于256的数字）。另外，假如你忽略掩码，squid会自动计算相应的掩码。例如，下组是相等的：

```
acl Bar src 172.16.66.0/255.255.255.0
acl Bar src 172.16.66.0/24
acl Squid dst www.squid-cache.org
```

> 将ACl主机名转换到IP地址的过程会延缓squid的启动。除非绝对必要，请避免使用主机名。

#### 4.5.2 域名的acl定义

**使用对象：srcdomain,dstdomain和cache_host_domain指令**

域名简单的就是DNS名字或区域。例如，下面是有效的域名：

```
www.squid-cache.org
squid-cache.org
org
```

> - 域名ACL有点深奥，因为相对于匹配域名和子域有点微妙的差别。当ACL域名以“.”开头，squid将它作为通配符，它匹配在该域的任何主机名，甚至域名自身。相反的，如果ACL域名不以“.”开头，squid使用精确的字符串比较，主机名同样必须被严格检查。
> - 域名匹配可能让人迷惑，所以继续往下看，以便你可以真正理解它。如下是两个稍微不同的ACL。

```
acl A dstdomain foo.com
acl B dstdomain .foo.com
```

> - 用户对http://www.foo.com/的请求匹配ACL B，但不匹配A。ACL A要求严格的字符串匹配，然而ACL B 里领头的点就像通配符。
> - 另外，用户对http://foo.com/的请求同时匹配A和B。尽管在URL主机名里的foo.com前面没有字符，但ACL B里领头的点仍然导致一个匹配。

#### 4.5.3 正则表达式的acl定义

**使用对象：srcdom_regex,dstdom_regex,url_regex,urlpath_regex,browser,referer_regex,ident_regex,proxy_auth_regex,req_mime_type,rep_mime_type**

> 大量的ACL使用正则表达式来匹配字符串。对squid来说，最常使用的正则表达式功能用以匹配字符串的开头或结尾。例如，^字符是特殊元字符，它匹配行或字符串的开头：

- [x] :^http://
  - 该正则表达式匹配任意以http://开头的URL。$也是特殊的元字符，因为它匹配行或字符串的结尾
- [x] :.jpg$
  - 实际上，该示例也有些错误，因为.字符也是特殊元字符。它是匹配任意单个字符的通配符。我们实际想要的应该是，见下行：
- [x] :\.jpg$
  - 反斜杠对这个“.”进行转义。该正则表达式匹配以.jpg结尾的任意字符串。假如你不使用^或$字符，正则表达式的行为就象标准子串搜索。他们匹配在字符串里任何位置出现的单词或词组。
  - 对所有的squid正则表达式类，你可以使用大小写敏感的选项。匹配是默认大小写敏感的。为了大小写不敏感，在ACL类型后面使用-i选项。例如：
     `acl Foo url_regex -i ^http://www`

#### 4.5.4 TCP端口号的acl定义

**使用对象：port，myport**

> 该类型是相对的。值是个别的端口号或端口范围。回想一下TCP端口号是16位值，这样它的值必须大于0或小于65536。如下是一些示例：

```
acl Foo port 123
acl Bar port 1-1024
acl Foo port 123 80 443
```

参考：http://home.arcor.de/jeffpang/squid/chap06.html

#### 4.5.5 method的acl定义

> method ACL 指HTTP请求方法。GET是典型的最常用方法，接下来是POST，PUT，和其他。下例说明如何使用method ACL：

```
acl Uploads method PUT POST
```

> 注意：CONNECT方法非常特殊。它是用于通过HTTP代理来封装某种请求的方法。在处理CONNECT方法和远程服务器的端口号时应特别谨慎。就像前面章节讲过的一样，你不希望squid连接到某些远程服务。你该限制CONNECT方法仅仅能连接到HTTPS/SSL或NNTPS端口（443或563）.默认的squid.conf这样做：

```
acl CONNECT method CONNECT
acl SSL_ports 443 563
http_access allow CONNECT SSL_ports #限制CONNECT方法仅仅能连接到HTTPS/SSL
http_access deny CONNECT
```

> PURGE  是另一个特殊的请求方法。它是Squid的专有方法，没有在任何RFC里定义。它让管理员能强制删除缓存对象。既然该方法有些危险，squid默认拒绝PURGE请求，除非你定义了ACL引用该方法。否则，任何能访问cache者也许能够删除任意缓存对象。在这里，我建议仅仅允许来自localhost的PURGE。

```
acl Purge method PURGE
acl localhost src 127.0.0.1/32
http_access allow Purge Localhost
http_access deny Purge
```

#### 4.5.6 proto的acl定义

> 该类型指URI访问（或传输）协议。如下是有效值：http,https(same as  HTTP/TLS),ftp,gopher,urn,whois和cache_object。也就是说，这些是被squid支持的URL机制名字。例如，假如你想拒绝所有的FTP请求，你可使用下列指令：

```
acl FTP proto FTP
http_access deny FTP
```

> cache_object机制是squid的特性。它用于访问squid的缓存管理接口，不幸的是，它并非好名字，可能会被改变。默认的squid.conf文件有许多行限制缓存管理访问：

```
acl Manager proto cache_object
acl Localhost src 127.0.0.1
http_access allow Manager Localhost
http_access deny Manager
```

#### 4.5.7 url_regex的acl定义

> url_regex ACL用于匹配请求URL的任何部分，包括传输协议和原始服务器主机名。例如，如下ACL匹配从FTP服务器的MP3文件请求：

```
acl FTPMP3 url_regex -i ^ftp://.*\.mp3$
acl sex url_regex -i ^http://.*sex.*
```

#### 4.5.8 urlpath_regex的acl定义

> urlpath_regex与url_regex非常相似，不过传输协议和主机名不包含在匹配条件里。这让某些类型的检测非常容易。例如，假设你必须拒绝URL里的"sex"，但仍允许在主机名里含有"sex"的请求，那么这样做：

```
acl Sex urlpath_regex sex
http_access deny Sex
```

另一个例子，假如你想特殊处理cgi-bin请求，你能这样捕获它们：

```
acl CGI1 urlpath_regex ^/cgi-bin
```

当然，CGI程序并非总在/cgi-bin/目录下，这样你应该编写其他的ACL来捕获它们。

#### 4.5.9 更多acl定义见squid配置文件

（1）限制同一IP客户端的最大连接数

```
acl OverConnLimit maxconn 16        #定义连接数16
http_access deny OverConnLimit      #拒绝达到16个的
```

（2）防止天涯盗链，转嫁给百度

```
acl tianya referer_regex -i tianya  #referer含有tianya
http_access deny tianya         #拒绝
deny_info http://www.baidu.com/logs.gif tianya  #拒绝信息回百度
```

（3）防止被人利用为HTTP代理，设置允许访问的IP地址

```
acl myip dst 192.168.1.1
http_access deny !myip
```

（4）防止百度机器人爬死服务器

```
acl AntiBaidu req_header User-Agent Baiduspider
http_access deny AntiBaidu
```

（5）允许本地管理

```
acl Manager proto cache_object
acl Localhost src 127.0.0.1 192.168.1.1
http_access allow Manger Localhost
```

> **提示：**
>  更多acl定义及用法请见acl配置文件401行到603行
>  `sed -n '401,603p' /usr/local/squid/etc/squid.conf.default`

### 4.6 Squid如何匹配访问控制元素

> 理解squid如何搜索ACL元素去匹配是很重要的。当ACL元素有多个值时，任何单个值能导致匹配。也就是说，squid在检查ACL元素值时使用OR逻辑。当squid找到第一个值匹配时，它停止搜索。这意味着把最可能匹配的值放在列表开头处，能减少延时。

**重点强调：**

（1）squid在搜索ACL元素时使用的"或"逻辑。在acl里的任何单值都可以导致匹配。
 （2）而应用访问规则恰好相反。对http_access和其他规则设置，squid使用"与"逻辑。

> squid默认的配置文件拒绝每一个客户请求。在任何人能使用代理之前，你必须在squid.conf文件里加入附加的访问控制规则。最简单的方法就是定义一个针对客户IP地址的ACL和一个访问规则，告诉squid允许来自这些地址的HTTP请求。squid有许多不同的ACL类型。src类型匹配客户IP地址，squid会针对客户HTTP请求检查http_access规则。这样，你就需要增加两行：

```
acl MyNetwork src 192.168.0.0/16
http_access allow MyNetwork
```

> 这两行需要放在正确的位置。**http_access的顺序非常重要，但是acl行的顺序不必介意。**squid默认的配置文件包含了一些重要的访问控制，最好不要改变或删除它们，除非你完全理解他们的意义。在你第一次编辑squid.conf文件时，请看如下注释：

**#INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS**

> 在该注释之后，以及"http_access deny all"之前插入你自己的规则，即MyNetwork的定义，如下是：

**一个典型的ACL设置，请大家用心理解。**

```
#定义squid acl访问控制规则
acl Safe_ports port 80
acl SSL_ports port 443
acl lannet src 10.0.0.0/24
acl localhost src 127.0.0.1/255.255.255.255
acl webip dst 10.0.0.8
acl webdomain dstdomain .yunjisuan.com
acl manager proto cache_object
acl CONNECT method CONNECT

#应用squid acl访问控制规则
http_access allow manager localhost
http_access deny manager
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
acl MyNetwork src all
http_access allow MyNetwork
http_access allow lannet
http_access deny all

#关于acl all src 0.0.0.0/0.0.0.0,在squid 3里，已经默认定义了all范围，所以不需要像squid 2.x那样手动定义all了
```

### 4.7 squid可见主机名

> - 如果不设置可见主机名，squid可能会报错无法运行。
> - 设置主机名有如下好处：
> - 主机名出现在squid的错误消息里，这帮助用户验证潜在问题的源头。
> - 主机名出现在squid转发的cache单元的HTTP Via头里。当请求到达原始主机时，Via头包含了在传输过程中涉及的代理列表。squid也使用Via头来检测转发环路。
> - 通过修改squid配置文件squid.conf中visible_hostname字段，可修改可见主机名：

```
[root@localhost etc]# sed -n '2965,2987p' /usr/local/squid/etc/squid.conf.default 
#
#Default:
# httpd_suppress_version_string off

#  TAG: visible_hostname
#   If you want to present a special hostname in error messages, etc,
#   define this.  Otherwise, the return value of gethostname()
#   will be used. If you have multiple caches in a cluster and
#   get errors about IP-forwarding you must set them to have individual
#   names with this setting.
#
#Default:
# none

#  TAG: unique_hostname
#   If you want to have multiple machines with the same
#   'visible_hostname' you must give each machine a different
#   'unique_hostname' so forwarding loops can be detected.
#
#Default:
# none
visible_hostname www.yunjisuan.com

#如果不指定可见主机名，那么系统会返回squid服务的主机名(也可能导致服务无法启动)
```

**编辑squid.conf配置文件,添加可见主机名**

```
[root@localhost etc]# echo "visible_hostname www.yunjisuan.com" >> /usr/local/squid/etc/squid.conf
[root@localhost etc]# tail -1 /usr/local/squid/etc/squid.conf
visible_hostname www.yunjisuan.com
```

### 4.8 squid管理联系信息

> 设置cache_mgr指令作为对用户的帮助，它是一个email地址，假如故障发生，用户能写信给管理员来通知管理员，cache_mgr地址默认出现在squid的错误消息里，修改配置文件squid.conf中cache_mgr字段。

```
cache_mgr  215379068@qq.com
```

**编辑squid.conf配置文件，添加邮件联系人信息**

```
[root@localhost etc]# echo "cache_mgr 215379068@qq.com" /usr/local/squid/etc/squid.conf
[root@localhost etc]# tail -1 /usr/local/squid/etc/squid.conf
cache_mgr 215379068@qq.com
```

### 4.9 squid最终的配置文件

> 根据以上的设置之后，squid.conf的内容如下：

```
[root@localhost etc]# cat /usr/local/squid/etc/squid.conf
acl manager proto cache_object
acl localhost src 127.0.0.1/32
acl to_localhost dst 127.0.0.0/8 0.0.0.0/32
acl localnet src 10.0.0.0/8 # RFC1918 possible internal network
acl localnet src 172.16.0.0/12  # RFC1918 possible internal network
acl localnet src 192.168.0.0/16 # RFC1918 possible internal network
acl SSL_ports port 443
acl Safe_ports port 80      # http
acl Safe_ports port 21      # ftp
acl Safe_ports port 443     # https
acl Safe_ports port 70      # gopher
acl Safe_ports port 210     # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280     # http-mgmt
acl Safe_ports port 488     # gss-http
acl Safe_ports port 591     # filemaker
acl Safe_ports port 777     # multiling http
acl CONNECT method CONNECT
http_access allow manager localhost
http_access deny manager
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access allow localnet
http_access deny all
icp_access allow localnet
icp_access deny all
htcp_access allow localnet
htcp_access deny all
http_port 3128
hierarchy_stoplist cgi-bin ?
access_log /usr/local/squid3/var/logs/access.log squid
refresh_pattern ^ftp:       1440    20% 10080
refresh_pattern ^gopher:    1440    0%  1440
refresh_pattern (cgi-bin|\?)    0   0%  0
refresh_pattern .       0   20% 4320
icp_port 3130
coredump_dir /usr/local/squid3/var/cache

#以下是添加的修改内容
cache_effective_user squid      #程序运行账户
cache_effective_group squid     #程序运行账户组
cache_store_log /usr/local/squid/var/logs/store.log     #store日志
cache_log /usr/local/squid/var/logs/cache.log           #cache日志
cache_dir ufs /usr/local/squid/var/cache 100 16 256     #cache缓存
visible_hostname www.yunjisuan.com                      #可见主机名
cache_mgr 215379068@qq.com                              #邮件联系人
```

## 第5章 运行squid并实现squid的普通代理模式

### 5.1 Squid主程序命令行选项

> 在运行squid前，需要了解squid主程序命令行选项。执行如下命令可以获得系统帮助：

```
[root@localhost etc]# /usr/local/squid/sbin/squid -h
Usage: squid [-cdhvzCDFNRVYX] [-s | -l facility] [-f config-file] [-[au] port] [-k signal]
       -a port   Specify HTTP port number (default: 3128).
       -d level  Write debugging to stderr also.
       -f file   Use given config-file instead of   #指定配置文件启动；重要
                 /usr/local/squid3/etc/squid.conf
       -h        Print help message.
       -k reconfigure|rotate|shutdown|interrupt|kill|debug|check|parse  #控制服务运行状态；重要
                 Parse configuration file, then send signal to 
                 running copy (except -k parse) and exit.
       -s | -l facility
                 Enable logging to syslog.
       -u port   Specify ICP port number (default: 3130), disable with 0.
       -v        Print version.
       -z        Create swap directories    #初始化缓存；重要
       -C        Do not catch fatal signals.
       -D        Disable initial DNS tests. #禁止DNS解析；重要
       -F        Dont serve any requests until store is rebuilt.
       -N        No daemon mode.        #不启用后台模式
       -R        Do not set REUSEADDR on port.
       -S        Double-check swap during rebuild.
       -X        Force full debugging.      #强制debug模式
       -Y        Only return UDP_HIT or UDP_MISS_NOFETCH during fast reload.
```

### 5.2 检查配置文件语法

```
/usr/local/squid/sbin/squid -k parse   检查语法的命令
[root@localhost etc]# /usr/local/squid/sbin/squid -k parse      #检查语法
2017/09/18 05:17:06| Processing Configuration File: /usr/local/squid3/etc/squid.conf (depth 0)
2017/09/18 05:17:06| Initializing https proxy context
WARNING: Cannot write log file: /usr/local/squid/var/logs/cache.log
/usr/local/squid/var/logs/cache.log: Permission denied  #出现错误，权限拒绝
         messages will be sent to 'stderr'.
[root@localhost etc]# ll -d /usr/local/squid/var/logs   #查看目录权限
drwxr-xr-x. 2 root root 4096 Sep 17 19:59 /usr/local/squid/var/logs #没有授权程序用户访问
[root@localhost etc]# chown -R squid /usr/local/squid/var/logs  #递归授权属主为squid
[root@localhost etc]# /usr/local/squid/sbin/squid -k parse  #再次检查语法
2017/09/18 05:19:36| Processing Configuration File: /usr/local/squid3/etc/squid.conf (depth 0)
2017/09/18 05:19:36| Initializing https proxy context
```

### 5.3 初始化cache目录

> 在运行squid之前，或者增加了新的cache_dir,你必须初始化cache，命令为：squid -z

1）设置环境变量，或者做命令的软连接

```
[root@localhost squid]# ln -s /usr/local/squid/sbin/* /usr/local/sbin/
[root@localhost squid]# ln -s /usr/local/squid/bin/* /usr/local/bin/
[root@localhost squid]# which squid
/usr/local/sbin/squid
```

2）初始化cache

```
[root@localhost squid]# squid -z    #初始化缓存命令
2017/09/18 05:26:07| Creating Swap Directories  
FATAL: Failed to make swap directory /usr/local/squid/var/cache: (13) Permission denied #报错，权限拒绝
[root@localhost squid]# ll -d /usr/local/squid/var/ #原来是目录对于程序用户没有权限
drwxr-xr-x. 3 root root 4096 Sep 17 19:59 /usr/local/squid/var/
[root@localhost squid]# chown -R squid /usr/local/squid/var/    #授权程序用户squid
[root@localhost squid]# squid -z        #再次初始化cache
2017/09/18 05:27:00| Creating Swap Directories
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/00
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/01
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/02
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/03
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/04
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/05
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/06
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/07
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/08
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/09
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/0A
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/0B
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/0C
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/0D
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/0E
2017/09/18 05:27:00| Making directories in /usr/local/squid/var/cache/0F
```

### 5.4 光盘安装一些必须的工具

```
[root@localhost squid]# yum -y install tree telnet dos2unix
```

**查看缓存目录**

```
[root@localhost squid]# tree /usr/local/squid/var/cache/
```

![QQ截图20170921004735.png-16.5kB](http://static.zybuluo.com/chensiqi/rkmzrj4zpcebiivx77ykib3d/QQ%E6%88%AA%E5%9B%BE20170921004735.png)

### 5.5 在终端窗口里测试启动squid

> 初始化cache目录后，就可以在终端窗口里运行squid，将日志记录到标准日子里，就可以轻易的定位任何错误或问题，并且确认squid是否成功启动。

```
[root@localhost squid]# squid -N -d1        #以debug调试模式在前台启动squid
2017/09/18 05:37:01| Starting Squid Cache version 3.0.STABLE20 for x86_64-unknown-linux-gnu...
2017/09/18 05:37:01| Process ID 59661
2017/09/18 05:37:01| With 4096 file descriptors available
2017/09/18 05:37:01| Performing DNS Tests...
2017/09/18 05:37:02| Successful DNS name lookup tests...
2017/09/18 05:37:02| helperOpenServers: Starting 5/5 'dnsserver' processes
2017/09/18 05:37:02| User-Agent logging is disabled.
2017/09/18 05:37:02| Referer logging is disabled.
2017/09/18 05:37:02| Unlinkd pipe opened on FD 14
2017/09/18 05:37:02| Swap maxSize 102400 + 8192 KB, estimated 8507 objects
2017/09/18 05:37:02| Target number of buckets: 425
2017/09/18 05:37:02| Using 8192 Store buckets
2017/09/18 05:37:02| Max Mem  size: 8192 KB
2017/09/18 05:37:02| Max Swap size: 102400 KB
2017/09/18 05:37:02| Rebuilding storage in /usr/local/squid/var/cache (DIRTY)
2017/09/18 05:37:02| Using Least Load store dir selection
2017/09/18 05:37:02| Set Current Directory to /usr/local/squid3/var/cache
2017/09/18 05:37:02| Loaded Icons.
2017/09/18 05:37:02| Accepting  HTTP connections at 0.0.0.0, port 3128, FD 15.
2017/09/18 05:37:02| Accepting ICP messages at 0.0.0.0, port 3130, FD 16.
2017/09/18 05:37:02| HTCP Disabled.
2017/09/18 05:37:02| Pinger socket opened on FD 18
2017/09/18 05:37:02| Ready to serve requests.           #出现这个表示启动成功！
2017/09/18 05:37:03| Done scanning /usr/local/squid/var/cache swaplog (0 entries)
2017/09/18 05:37:03| Finished rebuilding storage from disk.
2017/09/18 05:37:03|         0 Entries scanned
2017/09/18 05:37:03|         0 Invalid entries.
2017/09/18 05:37:03|         0 With invalid flags.
2017/09/18 05:37:03|         0 Objects loaded.
2017/09/18 05:37:03|         0 Objects expired.
2017/09/18 05:37:03|         0 Objects cancelled.
2017/09/18 05:37:03|         0 Duplicate URLs purged.
2017/09/18 05:37:03|         0 Swapfile clashes avoided.
2017/09/18 05:37:03|   Took 0.91 seconds (  0.00 objects/sec).
2017/09/18 05:37:03| Beginning Validation Procedure
2017/09/18 05:37:03|   Completed Validation Procedure
2017/09/18 05:37:03|   Validated 25 Entries
2017/09/18 05:37:03|   store_swap_size = 0
2017/09/18 05:37:03| storeLateRelease: released 0 objects
```

> **注意：**
>  此时命令行无法再继续输入命令了，如果要查看窗口可以单开一个窗口进行查看

![QQ截图20170921005543.png-16.6kB](http://static.zybuluo.com/chensiqi/82uffhtsbvz9l1ptgwnb2nw3/QQ%E6%88%AA%E5%9B%BE20170921005543.png)

> 一旦你见到"Ready to requests"消息，就可用一些HTTP请求来测试squid；你的浏览器使用squid作为代理，然后打开某个web页面。假如squid工作正常，正常载入就像没用过squid一样。

### 5.6 进行squid代理测试

（1）设置squid服务器为浏览器进行代理

![QQ截图20170921010226.png-58.7kB](http://static.zybuluo.com/chensiqi/ki1beiqwxa04zw9z2qw4ltw3/QQ%E6%88%AA%E5%9B%BE20170921010226.png)

（2）重启浏览器，登陆一个网页，比如www.baidu.com

![QQ截图20170921010622.png-23.9kB](http://static.zybuluo.com/chensiqi/hc7kyg6hxxtexrov6ou5s8xu/QQ%E6%88%AA%E5%9B%BE20170921010622.png)

```
[root@localhost ~]# tail /usr/local/squid/var/logs/access.log
1505728306.391     18 192.168.0.110 TCP_MISS/200 353 GET http://suggestion.baidu.com/su? - DIRECT/123.125.114.101 text/javascript
1505728307.520  61084 192.168.0.110 TCP_MISS/200 5569 CONNECT cm.g.doubleclick.net:443 - DIRECT/203.208.51.57 -
1505728307.543     22 192.168.0.110 TCP_MISS/000 0 GET http://suggestion.baidu.com/su? - DIRECT/123.125.114.101 -
1505728309.726     27 192.168.0.110 TCP_MISS/200 422 GET http://suggestion.baidu.com/su? - DIRECT/123.125.114.101 text/javascript
1505728309.871     25 192.168.0.110 TCP_MISS/200 422 GET http://suggestion.baidu.com/su? - DIRECT/123.125.114.101 text/javascript
1505728310.162     33 192.168.0.110 TCP_MISS/200 423 GET http://suggestion.baidu.com/su? - DIRECT/123.125.114.101 text/javascript
1505728310.240     23 192.168.0.110 TCP_MISS/200 417 GET http://suggestion.baidu.com/su? - DIRECT/123.125.114.101 text/javascript
1505728310.384     35 192.168.0.110 TCP_MISS/200 384 GET http://suggestion.baidu.com/su? - DIRECT/123.125.114.101 text/javascript
1505728310.440     19 192.168.0.110 TCP_MISS/200 356 GET http://suggestion.baidu.com/su? - DIRECT/123.125.114.101 text/javascript
1505728310.692     17 192.168.0.110 TCP_MISS/302 600 GET http://www.baidu.com/ - DIRECT/61.135.169.125 text/html         #记录在案
```

> 到这里位置，我们就是想了squid的普通（传统）代理模式
>  默认情况下，squid是普通代理模式

### 5.7 将squid作为服务进程运行

> 正常情况下你想将squid以后台进程运行（不出现在终端窗口里）。最容易的方法是简单执行如下命令：

```
[root@localhost squid]# /usr/local/squid/sbin/squid -D
[root@localhost squid]# netstat -antup | grep 3128
tcp        0      0 0.0.0.0:3128                0.0.0.0:*                   LISTEN      59774/(squid)       
tcp        0      0 192.168.0.190:3128          192.168.0.110:57625         TIME_WAIT   -  

-D:跳过DNS初始化检测
```

> **特别说明：**
>  -s  选项导致squid将重要的状态和警告信息写到syslogd，同样的消息被写进cache.log文件，忽略-s选项也是安全的。注意日志文件cache.log，无论squid以什么方式运行，cache.log总会记录squid的日志信息，有时候squid服务意外终止，通过这个文件也能查看到很重要的信息。

### 5.8 开机自启动squid

> 通常squid在每次服务器重启后需要自动启动，有如下两种方法可以实现squid的自动启动：

#### 5.8.1 将启动命令追加入/etc/rc.local

> 最容易的方法是修改/etc/rc.local脚本，在每次系统启动时以root运行。使用该脚本来启动squid非常容易，增加如下行到/etc/rc.lcoal里。

```
[root@localhost squid]# echo "#startup squid by Mr.chen at 2017/9/21" >> /etc/rc.local
[root@localhost squid]# echo "/usr/local/squid/sbin/squid -D" >> /etc/rc.local
[root@localhost squid]# tail -2 /etc/rc.local
#startup squid by Mr.chen at 2017/9/21
/usr/local/squid/sbin/squid -D
```

> - 当然你的安装位置可能不同，还有你可能要使用其他命令行选项。不要在这里使用-N选项（会打印很多调试日志，这在生产环境中检查日志是非常痛苦的）。
> - 如果没有使用cache_effective_user指令设置squid用户，你可以尝试使用su来让squid以非root用户运行：
> - /usr/bin/su -nobody -c '/usr/local/squid/sbin/squid -s' 这样也是可以的。
>    但是设置cache_effective_user为root用户运行是绝对不允许的。

#### 5.8.2 使用init.d和rc.d机制启动

（1）编写squid启动脚本

```
#!/bin/sh
# chkconfig:345 88 14
# description:squid Daemon
case "$1" in 
start)
    /usr/local/squid/sbin/squid -D
    ;;
stop)
    /usr/local/squid/sbin/squid -k shutdown
    ;;
restart)
    /usr/local/squid/sbin/squid -k reconfigure
    ;;
parse)
    /usr/local/squid/sbin/squid -k parse
    ;;
check)
    /usr/local/squid/sbin/squid -k check
    ;;
*)
    echo "Usage:$0  start|stop|restart|check|parse"
    ;;
esac
```

（2）添加squid开机自动启动服务

**拷贝squid启动脚本到/etcc/rc.d/init.d目录下**

```
chmod +x /etc/init.d/squid
chkconfig --add squid
```

**查看squid服务是否已经成功加上**

```
chkconfig --list squid
```

### 5.9 启动squid服务

> 启动squid服务方法有两种，一种是直接运行squid程序，另外一种是通过服务启动squid

**直接运行squid程序**

```
/usr/local/squid/sbin/squid -D
```

**通过服务启动squid**

```
/usr/local/squid/sbin/squid -k shutdown`
 `/etc/init.d/squid start
```

**启动完成后，记得检查squid进程启动情况**

```
ps -ef | grep squid | grep -v grep
```

### 5.10 停止squid

> 停止squid服务方法有两种，一种是直接运行squid程序，另外一种是通过服务停止squid

**直接运行squid程序**

```
/usr/local/squid/sbin/squid -k shutdown
```

**通过服务停止squid**

```
/etc/init.d/squid stop
```

### 5.11 squid日志轮询

> 加入squid的访问日志每天有上G，那么我们需要每天对squid的日志进行回滚，回滚的方法是如下：

```
/usr/lcoal/squid/sbin/squid -k rotate
```

**一旦执行squid日志回滚，这个命令会把access.log，store.log，cache.log都回滚**

```
[root@localhost squid]# /usr/local/squid/sbin/squid -k rotate
#每执行一次回滚，就会产生一个新的回滚日志，但是最多的回滚数为squid.conf中的logfile_rotate参数，默认是10个，即扩展名0-9，并且包含一个最新的日志文件（不带下标）

[root@localhost squid]# tree /usr/local/squid/var/logs/
/usr/local/squid/var/logs/
├── access.log
├── access.log.0
├── cache.log
├── cache.log.0
├── squid.pid
├── store.log
└── store.log.0

0 directories, 7 files
```

> - 日志回滚主要避免单个日志文件过大导致squid崩溃的问题，有些较老的系统版本文件大小有2GB限制，所以需要定期回滚一次，并且还可以节省磁盘空间。
> - 除非你在squid.conf里禁止，squid会写大量的日志文件。你必须周期性的滚动日志文件，以阻止他们变得太大。squid将大量的重要信息写入日志，假如写不进去了，squid会发生错误并退出。为了合理控制磁盘空间消耗，在cron里使用如下命令：

**squid -k rotate**

例如：如下任务接口在每天的早上0点滚动日志：

```
0 0 * * * /bin/sh /server/scripts/rotate_squid.sh > /dev/null 2>&1
#!/bin/bash
#cat /server/scripts/rotate_squid.sh
cd /usr/local/squid/var/logs/
[ -f access.log ] && mv access.log  access_$(date +%F).log
[ -f store.log ] && mv store.log  store_$(date +%F).log
[ -f cache.log ] && mv cache.log  cache_$(date +%F).log
/usr/local/squid/sbin/squid -k rotate
```

> - 该命令做两件事。首先，它关闭当前打开的日志文件。然后，通过在文件名后加数字扩展名，它重命名cache.log,store.log和access.log。例如，cache.log变成cache.log.0,cache.log.0变成cache.log.1，如此继续，滚动到logfile_rotate选项指定的值。
> - squid仅仅保存每个日志文件的最后logfile_rotate版本。更老的版本在重命名过程中被删除。假如你想保存更多的拷贝，你需要增加logfile_rotate限制,或者编写脚本用于将日志文件移动到其他位置上。

### 5.12 实战测试squid服务acl控制

```
acl sex_host url_regex -i ^http://.*yunjisuan.*
acl sex_path urlpath_regex2561410
http_access deny sex_path
http_access deny sex
```

> 提示：放置位置要注意
>  同学们开始自己实验

### 5.13 利用Web界面来管理squid

> squid有一个cachemgr.cgi的程序，可以Web来显示内容，这个对于调整squid的参数很是方便。可以平时我们安装完squid后其实就有这个程序了。我们只要在Apache中配置以下即可。

（1）安装apache服务

```
yum -y install httpd
```

（2）查找cachemgr.cgi的存放位置

```
[root@localhost ~]# find /usr/local/squid/ -name "cachemgr.cgi"
/usr/local/squid/libexec/cachemgr.cgi
```

（3）配置Apache中的squid。加入cachemgr.cgi来显示。

```
#在http配置文件里加入如下6行。
[root@localhost ~]# tail -6 /etc/httpd/conf/httpd.conf
ScriptAlias "/squid" "/usr/local/squid/libexec/cachemgr.cgi"
<Location "/squid">
    Order   deny,allow
    Deny    from all
    Allow   from all
</location>
```

（4）为了避免和后边的squid反向代理冲突，http修改端口为8080

```
[root@localhost ~]# sed -n '136p' /etc/httpd/conf/httpd.conf
Listen 80
[root@localhost ~]# sed -i '136 s#80#8080#g' /etc/httpd/conf/httpd.conf
[root@localhost ~]# sed -n '136p' /etc/httpd/conf/httpd.conf
Listen 8080
```

（5）启动apache服务

```
[root@localhost ~]# /etc/init.d/httpd start
Starting httpd: httpd: Could not reliably determine the servers fully qualified domain name, using ::1 for ServerName                    #忽略提示即可
                                                           [  OK  ]
[root@localhost ~]# netstat -antup | grep 8080
tcp        0      0 :::8080                     :::*                        LISTEN      1395/httpd    
```

（6）浏览器访问http://IP地址（squid服务器）:端口号/squid/

![QQ截图20170922211234.png-44.8kB](http://static.zybuluo.com/chensiqi/e3jxkocebsr0t68tr7h1cww7/QQ%E6%88%AA%E5%9B%BE20170922211234.png)

（7）登陆squid管理Web界面

![QQ截图20170922211800.png-46.9kB](http://static.zybuluo.com/chensiqi/va85qurhpa0mlhbukhznjydq/QQ%E6%88%AA%E5%9B%BE20170922211800.png)

![QQ截图20170922211917.png-94.9kB](http://static.zybuluo.com/chensiqi/7mixz877znawdzsh4c7yknj8/QQ%E6%88%AA%E5%9B%BE20170922211917.png)

## 附录一：squid编译参数详细解释

```
#指定安装路径
./configure --prefix=/usr/local/squid3 \
#
--enable-debug-cbdata \
#使用100个线程进行同步IO
--enable-async-io=100 \
#使用POSIX（可移植性操作系统接口）线程
--with-pthreads \
#Squid支持大量的不同存储模块。该选项指定squid编译时使用哪个模块
--enable-storeio="aufs,diskd,ufs" \
#指定排除元素，排除元素是squid需要腾出空间给新的cache目标时，用以排除旧目标的机制。squid在2.5支持3个排除元素：最少近期使用（LRU），贪婪对偶大小（GDS），最少经常使用（LFU）。
--enable-removal-policies="heap,lru" \
#启用ICMP，为了激活netdb，必须使用--enable-icmp选项来配置squid。也必须以超级用户权限来安装pinger程序。
--enable-icmp \
#启用延迟池。延时池是squid用于传输形状或带宽限制的技术。该池由大量的客户端IP地址组成。当来自这些客户端的请求处于cache丢失状态，他们的响应可能被人工延迟。
--enable-delay-pools \
#该选项激活来自客户请求的HTTP用户代理日志
--enable-useragent-log \
#该选项激活来自客户请求的HTTP referfer日志
--enable-referer-log \
#遇到骇客时才有用，自动反hackers
--enable-kill-parent-hack \
#squid在一些操作系统中支持ARP，或者以太地址访问控制列表。该代码使用非标准的函数接口，来执行ARP访问控制列表，所以它默认被禁止。假如，你在linux或solaris上使用squid，你可能用的上这个功能。
--enable-arp-acl \
#该选项设置error_directory指令的默认值
--enable-default-err-language=Simplify_Chinese \
#该选项指定复制到安装目录（$prefix/share/errors）的语言。假如你不使用该选项，所有可用语言被安装。
--enable-err-languages="Simplify_Chinese English" \
#强制使用“poll（）”函数扫描文件描述符
--disable-poll \
#禁用WCCP协议
--disable-wccp \
#禁用WCCP协议V2
--disable-wccpv2 \
#禁用ident协议
--disable-ident-lookups \
#禁用内部DNS
--disable-internal-dns \
#设置基础帮助名单
--enable-basic-auth-helpers="NCSA" \
#启用崩溃追踪，squid崩溃后会自动记录cache.log
--enable-stacktrace \
#启用大文件服务
--with-large-files \
#禁用mempools
--disable-mempools \
#默认的文件描述符是65535
--with-filedescriptors=65535 \
#支持SSL
--enable-ssl \
#该高级功能可能在squid被配置成加速器时使用。它建议squid在响应请求时，从后台原始服务器中寻找X-Accelerator-Vary头。
--enable-x-acceletator-vary
#Enable Transparent Proxy support for Linux (Netfilter)
--enable-linux-netfilter
#Enable real Transparent Proxy support for Netfilter
--enable-linux-tproxy
```

## 附录二：生产环境squid3.0的编译参数展示

**squid3.0实际编译的参数：**

```shell
./configure --prefix=/usr/local/squid3 \
--enable-async-io=100 \
--with-pthreads \
--enable-storeio="aufs,diskd,ufs" \
--enable-removal-policies="heap,lru" \
--enable-icmp \
--enable-delay-pools \
--enable-useragent-log \
--enable-referer-log \
--enable-kill-parent-hack \
--enable-cachemgr-hostname=loccalhost \
--enable-arp-acl \
--enable-default-err-language=English \
--enable-err-languages="Simplify_Chinese English" \
--disable-poll \
--disable-wccp \
--disable-wccpv2 \
--disable-ident-lookups \
--disable-internal-dns \
--enable-basic-auth-helpers="NCSA" \
--enable-stacktrace \
--with-large-files \
--disable-mempools \
--with-filedescriptors=64000 \
--enable-ssl \
--enable-x-acceletator-vary \
--disable-snmp \
--with-aio \
--enable-linux-netfilter \
--enable-linux-tproxy
```

