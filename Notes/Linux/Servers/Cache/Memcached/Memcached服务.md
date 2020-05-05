[TOC]

## 一， Memcached介绍

### 1.1 Memcached与常见同类软件对比

**（1）Memcached是什么？**

> - Memcached是一个开源的，支持高性能，高并发的分布式内存缓存系统，由C语言编写，总共2000多行代码。从软件名称上看，前3个字符“Mem”就是内存的意思，而接下来的后面5个字符“cache”就是缓存的意思，最后一个字符d，是daemon的意思，代表是服务器端守护进程模式服务。
> - Memcached服务分为服务器端和客户端两部分，其中，服务器端软件的名字形如Memcached-1.4.24.tar.gz，客户端软件的名字形如Memcache-2.25.tar.gz
> - Memcached软件诞生于2003年，最初由LiveJournal的Brad  Fitzpatrick开发完成。Memcache是整个项目的名称，而Memcached是服务器端的主程序名，因其协议简单，应用部署方便，且支持高并发，因此被互联网企业广泛使用，直到现在仍然如此。其官方网站地址：http://memcached.org/.

**（2）Memcached的作用**

> - 传统场景中，多数Web应用都将数据保存到关系型数据库中（例如：MySQL），Web服务器从中读取数据并在浏览器中显示。但随着数据量的增大，访问的集中，关系型数据库的负担就会出现加重，响应缓慢，导致网站打开延迟等问题，影响用户体验。
> - 这时就需要Memcached软件出马了。使用Memcached的主要目的是，通过在自身内存中缓存关系型数据库的查询结果，减少数据库被访问的次数，以提高动态Web应用的速度，提高网站架构的并发能力和可扩展性。
> - Memcached服务的运行原理是通过在事先规划好的系统内存空间中临时缓存数据库中的各类数据，以达到减少前端业务服务对数据库的直接高并发访问，从而提升大规模网站集群中动态服务的并发访问能力。
>    -生产场景的Memcached服务一般被用来保存网站中经常被读取的对象或数据，就像我们的客户端浏览器也会把经常访问的网页缓存起来一样，通过内存缓存来存取对象或数据要比磁盘存取快很多，因为磁盘是机械的，因此，在当今的IT企业中，Memcached的应用范围很广泛。

### 1.2 互联网常见内存缓存服务软件

![QQ截图20170804211905.png-161.5kB](http://static.zybuluo.com/chensiqi/j11a6j2k6xzfjgnyr347glkh/QQ%E6%88%AA%E5%9B%BE20170804211905.png)

## 二，Memcached的用途与应用场景

### 2.1 Memcached常见用途工作流程

> Memcached是一种内存缓存软件，在工作中经常用来缓存数据库的查询数据，数据被缓存在事先与分配的Memcached管理的内存中，可以通过API或命令的方式存取内存中缓存的这些数据，Memcached服务内存中缓存的数据就像一张巨大的hash表，每条数据都是以key-value对的形式存在。

#### 2.1.1网站读取Memcached数据时工作流程

> 从逻辑上来说，当程序访问后端数据库获取数据时会优先访问Memcached缓存，如果缓存中有数据就直接返回给客户端用户，如果没有合适的数据（没有命中），再去后端的数据库读取数据，读取到需要的数据后，就会把数据返回给客户端，同时还会把读取到的数据缓存到Memcached内存中，这样客户端用户再次请求相同的数据时就会直接读取Memcached缓存的数据了，这就大大地减轻了后端数据库的压力，并提高了整个网站的响应速度，提升了用户体验。

**展示了Memcached缓存系统和后端数据库系统的协作流程**

![QQ截图20170804213054.png-77.7kB](http://static.zybuluo.com/chensiqi/j2mafhwd7l3orw41qncp2ngo/QQ%E6%88%AA%E5%9B%BE20170804213054.png)

如上图所示：使用Memcached缓存查询的数据来减少数据库压力的具体工作流程如下：

（1）Web程序首先检查客户端请求的数据是否在Memcached缓存中存在，如果存在，直接把请求的数据返回给客户端，此时不再请求后端数据库。

（2）如果请求的数据在Memcached缓存中不存在，则程序会去请求数据库服务，把从数据库中取到的数据返回给客户端，同时把新取到的数据缓存一份到Memcached缓存中。

#### 2.1.2 网站更新Memcached数据时的工作流程

**具体流程如下：**

（1）当程序更新或删除数据时，会首先处理后端数据库中的数据。

（2）在处理后端数据库中数据的同时，也会通知Memcached，告诉它对应的旧数据失效，从而保证Memcached中缓存的数据始终和数据库中一致，这个数据一致性非常重要，也是大型网站分布式缓存集群最头疼的问题所在。

（3）如果是在高并发读写场合，除了要程序通知Memcached过期的缓存失效外，还可能要通过相关机制，例如在数据库上部署相关程序（如在数据库中设置触发器使用UDFs），实现当数据库有更新时就把数据更新到Memcached服务中，这样一来，客户端在访问新数据时，因预先把更新过的数据库数据复制到Memcached中缓存起来了，所以可以减少第一次查询数据库带来的访问压力，提升Memcached中缓存的命中率，甚至新浪门户还会把持久化存储Redis做成MySQL数据库的从库，实现真正的主从复制。

**下图为Memcached网站作为缓存应用更新数据的流程**

![QQ截图20170804233122.png-66.7kB](http://static.zybuluo.com/chensiqi/ulp9etczxsunjilwhbc7y4uf/QQ%E6%88%AA%E5%9B%BE20170804233122.png)

**下图为Memcached服务作为缓存应用通过相关软件更新数据的流程**

![QQ截图20170804233252.png-62.5kB](http://static.zybuluo.com/chensiqi/um4nsru1t315dtbwe8p9bnhf/QQ%E6%88%AA%E5%9B%BE20170804233252.png)

> 在生产工作中，网站Web服务器作为缓存应用更新数据的方案更为常用，即由网站程序负责更新Memcached缓存。

### 2.2 Memcached在企业中的应用场景

#### 2.2.1 作为数据库的查询数据缓存

（1）完整数据缓存

> 例如：电商的商品分类功能不是经常变动的，因此可以事先放到Memcached里，然后再对外提供数据访问。这个过程被称之为“数据预热”。
>  此时只需读取缓存，无需读取数据库就能得到Memcached缓存里的所有商品分类数据了，所以数据库的访问压力就会大大降低。
>  为什么商品分类数据可以事先放在缓存里呢？
>  因为，商品分类几乎都是由内部人员管理的，如果需要更新数据，更新数据库后，就可以把数据同时更新到Memcached里。
>  如果把商品分类数据做成静态化文件，然后，通过在前端Web缓存或者使用CDN加速效果更好。

（2）热点数据缓存

热点数据缓存一般是用于由用户更新的商品，例如淘宝的卖家，在卖家新增商品后，网站程序就会把商品写入后端数据库，同时把这部分数据，放入Memcached内存中，下一次访问这个商品的请求就直接从Memcached内存中取走了。这种方法用来缓存网站热点的数据，即利用Memcached缓存经常被访问的数据。

> **提示：**
>  这个过程可以通过程序实现，也可以在数据库上安装相关软件进行设置，直接由数据库把内容更新到Memcached中，就相当于Memcached是MySQL的从库一样。

- 如果碰到电商双11，秒杀高并发的业务场景，必须要事先预热各种缓存，包括前端的Web缓存和后端的数据库缓存。
- 也就是先把数据放入内存预热，然后逐步动态更新。此时，会先读取缓存，如果缓存里没有对应的数据，再去读取数据库，然后把读到的数据放入缓存。如果数据库里的数据更新，需要同时触发缓存更新，防止给用户过期的数据，当然对于百万级别并发还有很多其他的工作要做。
- 绝大多数的网站动态数据都是保存在数据库当中的，每次频繁地存取数据库，会导致数据库性能急剧下降，无法同时服务更多的用过户（比如MySQL特别频繁的锁表就存在此问题），那么，就可以让Memcached来分担数据库的压力。增加Memcached服务的好处除了可以分担数据库的压力以外，还包括无须改动整个网站架构，只须简单地修改下程序逻辑，让程序先读取Memcached缓存查询数据即可，当然别忘了，更新数据时也要更新Memcached缓存。

#### 2.2.2 作为集群节点的session会话共享存储

> 即把客户端用户请求多个前端应用服务集群产生的session会话信息，统一存储到一个Memcached缓存中。由于session会话数据是存储在内存中的，所以速度很快。

**下图为Memcached服务在企业集群架构中的常见工作位置：**

![QQ截图20170805004501.png-236.6kB](http://static.zybuluo.com/chensiqi/mahua6h9apluvtsuve0wnq3x/QQ%E6%88%AA%E5%9B%BE20170805004501.png)

## 三，Memcached的特点与工作机制

### 3.1 Memcached的特点

**Memcached作为高并发，高性能的缓存服务，具有如下特点：**

- 协议简单。Memcached的协议实现很简单，采用的是基于文本行的协议，能通过telnet/nc等命令直接操作memcached服务存储数据。
- 支持epoll/kqueue异步I/O模型，使用libevent作为事件处理通知机制。
- 简单的说，libevent是一套利用c开发的程序库，它将BSD系统的kqueue，Linux系统的epoll等事件处理功能封装成一个接口，确保即使服务器端的连接数增加也能发挥很好的性能。Memcached就是利用这个libevent库进行异步事件处理的。
- 采用key/value键值数据类型。被缓存的数据以key/value键值形式存在，例如：

```
benet-->36,key=benet,value=36
yunjisuan-->28,key=yunjisuan,value=28
#通过benet key可以获取到36值，同理通过yunjisuan key可以获取28值
```

- 全内存缓存，效率高。Memcached管理内存的方式非常高效，即全部的数据都存放于Memcached服务事先分配好的内存中，无持久化存储的设计，和系统的物理内存一样，当重启系统或Memcached服务时，Memcached内存中的数据就会丢失。
- 如果希望重启后，数据依然能保留，那么就可以采用redis这样的持久性内存缓存系统。
- 当内存中缓存的数据容量达到服务启动时设定的内存值时，就会自动使用LRU算法（最近最少被使用的）删除过期的缓存数据。也可以在存放数据时对存储的数据设置过期时间，这样过期后数据就自动被清除，Memcached服务本身不会监控数据过期，而是在访问的时候查看key的时间戳判断是否过期。
- 可支持分布式集群
   Memcached没有像MySQL那样的主从复制方式，分布式Memcached集群的不同服务器之间是互不通信的，每一个节点都独立存取数据，并且数据内容也不一样。通过对Web应用端的程序设计或者通过支持hash算法的负载均衡软件，可以让Memcached支持大规模海量分布式缓存集群应用。

**下面是利用Web端程序实现Memcached分布式的简单代码：**

```
"memcached_servers" ==>array(
'10.4.4.4:11211',
'10.4.4.5:11211',
'10.4.4.6:11211',
```

**下面使用Tengine反向代理负载均衡的一致性哈希算法实现分布式Memcached的配置。**

```
http {

upstream test {

consistent_hash $request_uri;

server 127.0.0.1:11211 id=1001 weight=3;

server 127.0.0.1:11212 id=1002 weight=10;

server 127.0.0.1:11213 id=1003 weight=20;

}

}
```

> **提示：**
>  Tengine是淘宝网开源的Nginx的分支，上述代码来自：
>  http://tengine.taobao.org/document_cn/http_upstream_consistent_hash_cn.html

### 3.2 Memcached工作原理与机制

#### 3.2.1 Memcached工作原理

> Memcached是一套类似C/S模式架构的软件，在服务器端启动Memcached服务守护进程，可以指定监听本地的IP地址，端口号，并发访问连接数，以及分配了多少内存来处理客户端请求。

#### 3.2.2 Socket事件处理机制

> Memcached软件是由C语言来实现的，全部代码仅有2000多行，采用的是异步epoll/kqueue非阻塞I/O网络模型，其实现方式是基于异步的libevent事件单进程，单线程模式。使用libevent作为事件通知机制，应用程序端通过指定服务器的IP地址及端口，就可以连接Memcached服务进行通信。

#### 3.2.3 数据存储机制

> - 需要被缓存的数据以key/value键值对的形式保存在服务器端预分配的内存区中，每个被缓存的数据都有唯一的标识key，操作Memcached中的数据就是通过这个唯一标识的key进行的。缓存到Memcached中的数据仅放置在Memcached服务预分配的内存中，而非存储在Memcached服务器所在的磁盘上，因此存取速度非常快。
> - 由于Memcached服务自身没有对缓存的数据进行持久化存储的涉及，因此，在服务器端的Memcached服务进程重启之后，存储在内存中的这些数据就会丢失。且当内存中缓存的数据容量达到启动时设定的内存值时，也会自动使用LRU算法删除过期的数据。
> - 开发Memcached的初衷仅是通过内存缓存提升访问效率，并没有过多考虑数据的永久存储问题。因此，如果使用Memcached作为缓存数据服务，要考虑数据丢失后带来的问题，例如：是否可以重新生成数据，还有，在高并发场合下缓存宕机或重启会不会导致大量请求直接到数据库，导致数据库无法承受，最终导致网站架构雪崩等。

#### 3.2.4 内存管理机制

**Memcached采用了如下机制：**

- 采用slab内存分配机制
- 采用LRU对象清除机制
- 采用hash机制快速检索item

#### 3.2.5 多线程处理机制

多线程处理时采用的是pthread（POSIX）线程模式。

若要激活多线程，可在编译时指定：./configure --enable-threads

锁机制不够完善

负载过重时，可以开启多线程（-t 线程数为CPU核数）

### 3.3 Memcached预热理念及集群节点的正确重启方法

#### 3.3.1 Memcached预热理念

> - 当需要大面积重启Memcached时，首先要在前端控制网站入口的访问流量，然后，重启Memcached集群并进行数据预热，所有数据都预热完毕之后，再逐步放开前端网站入口的流量。
> - 为了满足Memcached服务数据可以持久化存储的需求，在较早时期，新浪网基于Memcached服务开发了一款NoSQL软件，名字为MemcacheDB，实现了在缓存的基础上增加了持久存储的特性，不过目前逐步被更优秀的Redis软件取代了。

#### 3.3.2 如何正确开启网站集群服务器

> 如果由于机房断电或者搬迁服务器集群到新机房，那么启动集群服务器时，一定要从网站集群的后端依次往前端开启，特别是开启Memcached缓存服务器时要提前预热。

## 四，Memcached内存管理

### 4.1 Memcached内存管理机制深入剖析

**（1）Malloc内存管理机制**

**在讲解Memcached内存管理机制前，先来了解malloc**

> - malloc的全称是memory allocation，中文名称动态内存分配，当无法知道内存具体位置的时候，想要绑定真正的内存空间，就需要用到动态分配内存。
> - 早期的Memcached内存管理是通过malloc分配的内存实现的1，使用完后通过free来回收内存。这种方式容易产生内存碎片并降低操作系统对内存的管理效率。因此，也会加重操作系统内存管理器的负担，最坏的情况下，会导致操作系统比Memcached进程本身还慢，为了解决上述问题，Slab Allocator内存分配机制就诞生了。

**（2）Slab内存管理机制**

**现在的Memcached是利用Slab Allocation机制来分配和管理内存的，过程如下：**

> 1）提前将大内存分配大小为1MB的若干个slab，然后针对每个slab再进行小对象填充，这个小对象称为chunk，避免大量重复的初始化和清理，减轻了内存管理器的负担。
>  Slab  Allocation内存分配的原理是按照预先规定的大小，将分配给Memcached服务的内存预先分割成特定长度的内存块（chunk），再把尺寸相同的内存块（chunk）分成组（chunks slab class），这些内存块不会释放，可以重复利用，如下图所示。

![QQ截图20170805233556.png-77.7kB](http://static.zybuluo.com/chensiqi/ny14ytypb0pmjirmp2mait6m/QQ%E6%88%AA%E5%9B%BE20170805233556.png)

> 2）新增数据对象存储时。因Memcached服务器中保存着slab内空闲chunk的列表，他会根据该列表选择chunk，然后将数据缓存于其中。当有数据存入时，Memcached根据接收到的数据大小，选择最适合数据大小的slab分配一个能存下这个数据的最小内存块（chunk）。例如：有100字节的一个数据，就会被分配存入下面112字节的一个内存块中，这样会有12字节被浪费，这部分空间就不能被使用了，这也是Slab Allocator机制的一个缺点。

![QQ截图20170805234534.png-35.7kB](http://static.zybuluo.com/chensiqi/bc3by5ny040inqm0no2p3g8h/QQ%E6%88%AA%E5%9B%BE20170805234534.png)

> Slab Allocator还可重复使用已分配的内存，即分配到的内存不释放，而是重复利用。

**（3）Slab Allocation的主要术语**

![QQ截图20170805234754.png-73.1kB](http://static.zybuluo.com/chensiqi/4fdydagst6ytwadlf4tkwbpt/QQ%E6%88%AA%E5%9B%BE20170805234754.png)

**（4）Slab 内存管理机制特点**

- 提前分配大内存Slab 1MB，再进行小对象填充chunk。
- 避免大量重复的初始化和清理，减轻内存管理器负担。
- 避免频繁malloc/free内存分配导致的碎片

**下面对Mc的内存管理机制进行一个小结**

- Mc的早期内存管理机制为malloc（动态内存分配）
- malloc（动态内存分配）产生内存碎片，导致操作系统性能急剧下降。
- Slab内存分配机制可以解决内存碎片的问题
- Memcached服务的内存预先分割成特定长度的内存块，称为chunk，用于缓存数据的内存空间或内存块，相当于磁盘的block，只不过磁盘的每一个block都是相等的，而chunk只有在同一个Slab Class内才是相等的。
- Slab Class指特定大小（1MB）的包含多个chunk的集合或组，一个Memcached包含多个Slab Class，每个Slab Class包含多个相同大小的chunk。
- Slab机制也有缺点，例如，Chunk的空间会有浪费等。

### 4.2 Memcached Slab Allocator内存管理机制的缺点

**（1）chunk存储item浪费空间**

> Slab  Allocator解决了当初的内存碎片问题，但新的机制也给Memcached带来了新的问题。这个问题就是，由于分配的是特定长度的内存，因此无法有效利用分配的内存。例如，将100字节的数据缓存到128字节的chunk中，剩余的28字节就浪费了，如下图所示：

![QQ截图20170806001806.png-23.1kB](http://static.zybuluo.com/chensiqi/y3ple6vbxae9ewao5b8lxe5u/QQ%E6%88%AA%E5%9B%BE20170806001806.png)

> 避免浪费内存的办法是，预先计算出应用存入的数据大小，或把同一业务类型的数据存入一个Memcached服务器中，确保存入的数据大小相对均匀，这样就可以减少内存的浪费。
>  还有一种办法是，在启动时指定“-f”参数，能在某种程度上控制内存组之间的大小差异。在应用中使用Memcached时，通常可以不重新设置这个参数，即使用默认值1.25进行部署即可。如果想优化Memcached对内存的使用，可以考虑重新计算数据的预期平均长度，调整这个参数来获得合适的设置值，命令如下：

```
-f <factor>chunk size growth factor (default:1.25)!
```

**（2）Slab尾部剩余空间**

> - 假设在classid=40中，两个chunk占用了1009384byte，那么就有1048576-1009384=39192byte会被浪费掉。解决办法：规划slab大小=chunk大小*n整数倍。

### 4.3 使用Growth Factor对Slab Allocator内存管理机制调优

> 在启动Memcached时指定Growth Factor因子（通过 -f  选项），就可以在某种程度上控制每组Slab之间的差异。默认值1.25。但是，在该选项出现之前，这个因子曾经被固定为2，称为2“powers of 2”策略。让我们用以前的设置，以verbose模式启动Memcached试试看：

```
#memcached -f 2 w
```

**下面是启动后的verbose输出：**

```
slab class 1:chunk size 128 perslab 8192
slab class 2:chunk size 256 perslab 4096
slab class 3:chunk size 512 perslab 2048
slab class 4:chunk size 1024 perslab 1024
slab class 5:chunk size 2048 perslab 512
slab class 6:chunk size 4096 perslab 256
slab class 7:chunk size 8192 perslab 128
slab class 8:chunk size 16384 perslab 64
slab class 9:chunk size 32768 perslab 32
slab class 10:chunk size 65536 perslab 16
slab class 11:chunk size 131072 perslab 8
slab class 12:chunk size 262144 perslab 4
slab class 13:chunk size 524288 perslab 2
```

> 可见，从128字节的组开始，组的大小依次增大为原来的2倍。这样设置的问题是，Slab之间的差别比较大，有些情况下就相当浪费内存。因此，为尽量减少内存浪费，两年前追加了growth factor这个选项。

**来看看现在的默认设置（f=1.25）时的输出：**

```
slab class 1:chunk size 88 perslab 11915 <---88*11915=1048520
slab class 2:chunk size 112 perslab 9362
slab class 3:chunk size 144 perslab 7281
slab class 4:chunk size 184 perslab 5698
slab class 5:chunk size 232 perslab 4519
slab class 6:chunk size 296 perslab 3542
slab class 7:chunk size 376 perslab 2788
slab class 8:chunk size 472 perslab 2221
slab class 9:chunk size 592 perslab 1771
slab class 10:chunk size 744 perslab 1409 <---744*1409=1048520
```

> - 此时每个Slab的大小是一样的，即1048520，1MB。组间的差距比因子为2时小得多，可见，这个值越小，Slab中chunk  size的差距就越小，内存浪费也就越小。可见，默认值1.25更适合缓存几百字节的对象。从上面的输出结果来看，可能会觉得有些计算误差，这些误差是为了保持字节数的对齐而故意设置的。
> - 当使用Memcached或是直接使用默认值进行部署时，最好是重新计算一下数据的预期平均长度，调整growth factor，以获得最恰当的设置。内存是珍贵的资源，浪费就太可惜了。

### 4.4 Memcached的检测过期与删除机制

**（1）Memcached懒惰检测对象过期机制**

> - 首先要知道，Memcached不会主动检测item对象是否过期，而是在进行get操作时检查item对象是否过期以及是否应该删除！
> - 因为不会主动检测item对象是否过期，自然也就不会释放已分配给对象的内存空间了，除非为添加的数据设定过期时间或内存缓存满了，在数据过期后，客户端不能通过key取出它的值，其存储空间将被重新利用。
> - Memcached使用的这种策略为懒惰检测对象过期策略，即自己不监控存入的key/value对是否过期，而是在获取key值时查看记录的时间戳（sed key flag exptime bytes），从而检查key/value对空间是否过期。这种策略不会在过期检测上浪费CPU资源。

**（2）Memcached懒惰删除对象机制**

> - 当删除item对象时，一般不会释放内存空间，而是做删除标记，将指针放入slot回收插槽，下次分配的时候直接使用。
> - Memcached在分配空间时，会优先使用已经过期的key/value对空间；若分配的内存空间占满，Memcached就会使用LRU算法来分配空间，删除最近最少使用的key/value对，从而将其空间分配给新的key/value对。在某些情况下（完整缓存），如果不想使用LRU算法，那么可以通过“-M”参数来启动Memcached，这样，Memcached在内存耗尽时，会返回一个报错信息，如下：

```
-M rerurn error on memory exhausted(rather than removing items)
```

**下面针对Memcached删除机制进行一个小结**

- 不主动检测item对象是否过期，而是在get时才会检查item对象是否过期以及是否应该删除。
- 当删除item对象时，一般不释放内存空间，而是做删除标记，将指针放入slot回收插槽，下次分配的时候直接使用。
- 当内存空间满的时候，将会根据LRU算法把最近最少使用的item对象删除。
- 数据存入可以设定过期时间，但是数据过期后不会被立即删除，而是在get时检查item对象是否过期以及是否应该删除。
- 如果不希望系统使用LRU算法清除数据，可以用使用-M参数。

## 五，Memcached服务安装

> Memcached的安装比较简单，支持Memcached的平台常见的有Linux，FreeBSD，Solaris，Windows。这里以Centos6.5为例进行讲解。

### 5.1 安装libevent及连接Memcached工具nc

**系统安装环境如下：**

```
[root@cache01 ~]# cat /etc/redhat-release 
CentOS release 6.5 (Final)
[root@cache01 ~]# uname -r
2.6.32-431.el6.x86_64
[root@cache01 ~]# uname -m
x86_64
```

**安装Memcached前需要先安装libevent，有关libevent的内容在前文已经介绍，此处用yum命令安装libevent。操作命令如下：**

```
[root@cache01 ~]# yum -y install libevent libevent-devel nc      #自带光盘里没有，需要公网yum源
[root@cache01 packages]# rpm -qa libevent libevent-devel nc
libevent-1.4.13-4.el6.x86_64
libevent-devel-1.4.13-4.el6.x86_64
nc-1.84-24.el6.x86_64
```

### 5.2 安装Memcached

**操作命令如下：**

```
[root@cache01 ~]# yum -y install memcached       #此处可通过光盘安装
```

> **提示：**
>  形如“memcache-2.2.7.tgz”文件名的软件为客户端源代码软件，而形如“memcached-1.4.24.tar.gz”的文件为服务器端的源代码软件。

## 六，Memcached服务的基本管理

### 6.1 启动Memcached

**启动Memcached的命令如下：**

```
[root@cache01 ~]# which memcached       #查看Memcached命令路径
/usr/bin/memcached
[root@cache01 ~]# memcached -m 16m -p 11211 -d -u root -c 8192 
#启动第一个Memcached实例
[root@cache01 ~]# netstat -antup | grep 11211       #查看启动情况
tcp        0      0 0.0.0.0:11211               0.0.0.0:*                   LISTEN      1303/memcached      
tcp        0      0 :::11211                    :::*                        LISTEN      1303/memcached      
udp        0      0 0.0.0.0:11211               0.0.0.0:*                               1303/memcached      
udp        0      0 :::11211                    :::*                                    1303/memcached   
[root@cache01 ~]# ps -ef | grep memcached | grep -v grep
#查看Memcached进程
root       1303      1  0 08:49 ?        00:00:00 memcached -m 16m -p 11211 -d -u root -c 8192

#启动第二个Memcached实例
[root@cache01 ~]# memcached -m 16m -p 11212 -d -u root -c 8192
[root@cache01 ~]# ps -ef | grep memcached | grep -v grep
root       1303      1  0 08:49 ?        00:00:00 memcached -m 16m -p 11211 -d -u root -c 8192
root       1317      1  0 08:55 ?        00:00:00 memcached -m 16m -p 11212 -d -u root -c 8192

#可把上述两个实例的启动命令放入/etc/rc.local，以便下次开机可以自启动
[root@cache01 ~]# tail -2 /etc/rc.local
memcached -m 16m -p 11211 -d -u root -c 8192
memcached -m 16m -p 11212 -d -u root -c 8192
```

### 6.2 Memcached启动命令相关参数说明

**进程与连接设置：**
 |命令参数|说明|
 |--|--|
 |-d|以守护进程（daemon）方式运行服务|
 |-u|指定运行Memcached的用户，如果当前用户为root，需要使用此参数指定用户|
 |-l|指定Memcached进程监听的服务器IP地址，可以不设置此参数|
 |-p（小写）|指定Memcached服务监听TCP端口号。默认为11211|
 |-P（大写）|设置保存Memcached的pid文件（$$）,保存PID到指定文件|

**内存相关设置：**

| 命令参数 | 说明                                                |
| -------- | --------------------------------------------------- |
| -m       | 指定Memcached服务可以缓存数据的最大内存，默认为64MB |
| -M       | Memcached服务内存不够时禁止LRU，如果内存满了会报错  |
| -n       | 为key+value——flags分配的最小内存空间，默认为48字节  |
| -f       | chunk size增长因子，默认为1.25                      |
| -L       | 启用大内存页，可以降低内存浪费，改进性能            |

**并发连接设置：**

| 并发连接设置 | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| -c           | 最大的并发连接数，默认是1024                                 |
| -t           | 线程数，默认4.由于Memcached采用的是NIO，所以太多线程作用不大 |
| -R           | 每个event最大请求数，默认是20                                |
| -C           | 禁用CAS（可以禁止版本计数，减少开销）                        |

**测试参数：**

| -v   | 打印较少的errors/warnings                        |
| ---- | ------------------------------------------------ |
| -vv  | 打印非常多调试信息和错误输出到控制台             |
| -vvv | 打印极多的调试信息和错误输出，也打印内部状态转变 |

**其他选项可通过“memcached -h”命令来显示。**

### 6.3 向Memcached中写入数据并检查

#### 6.3.1 Memcached中的数据形式及与MySQL相关语句对比

> 向Memcached中添加数据时，注意添加的数据一般为键值对的形式，例如：key1-->values1,key2-->values2

**这里把Memcached添加，查询，删除等的命令和MySQL数据库做一个基本类比，见下表：**

| MySQL数据库管理   | Memcached管理         |
| ----------------- | --------------------- |
| MySQL的insert语句 | Memcached的set命令    |
| MySQL的select语句 | Memcached的get命令    |
| MySQL的delete语句 | Memcached的delete命令 |

#### 6.3.2 向Memcached中写入数据实践

**（1）通过printf配合nc向Memcached中写入数据，命令如下：**

```
[root@cache01 ~]# printf "set key1 0 0 5\r\nbenet\r\n" | nc 127.0.0.1 11211
STORED          #出现STORED表示成功添加key1及对应的数据

#如果set命令的字节是6，那么后面就要6个字符（字节）。否则插入数据就会不成功。示例如下：
[root@cache01 ~]# printf "set key1 0 0 4\r\nbenet\r\n" | nc 127.0.0.1 11211
CLIENT_ERROR bad data chunk
ERROR

#通过printf配置nc从Memcached中读取数据，命令如下：
[root@cache01 ~]# printf "get key1\r\n" | nc 127.0.0.1 11211
VALUE key1 0 5
benet               #这就是读取到的key1对应额值

#通过printf配合nc从Memcached中删除数据，命令如下：
[root@cache01 ~]# printf "delete key1\r\n" | nc 127.0.0.1 11211
DELETED
[root@cache01 ~]# printf "get key1\r\n" | nc 127.0.0.1 11211
END
```

> **提示：**
>  推荐使用上述方法测试操作Memcached

**（2）通过telnet命令写入数据时，具体步骤如下：**

```
1)安装telnet工具
yum -y install telnet

2)通过telnet向Memcached中写入数据
[root@cache01 ~]# which telnet
/usr/bin/telnet
[root@cache01 ~]# telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
set user01 0 0 7            #写入数据
Welcome     
STORED
get user01                  #浏览数据
VALUE user01 0 7
Welcome
END
delete user01               #删除数据
DELETED
get user01                  #再浏览数据，数据被删除       
END
quit                        #退出
Connection closed by foreign host.
```

### 6.4 操作Memcached相关命令的语法

**以下为操作Memcached的相关命令基本语法：**

```
     set      key1    0       0       6      \r\n     benet     \r\n
<command name><key><flags><exptime><bytes><datablock><string><datablock>
STORED
<status>
```

**下表为操作Memcached相关命令的详细说明：**

![QQ截图20170806150843.png-141.4kB](http://static.zybuluo.com/chensiqi/1c6pvxfhaei72nyyp9j2ei6d/QQ%E6%88%AA%E5%9B%BE20170806150843.png)

### 6.5 关闭Memcached

**单实例关闭Memcached的方法如下：**

```
[root@cache01 ~]# ps -ef | grep memcached | grep -v grep
root       1473      1  0 11:11 ?        00:00:00 memcached -m 16m -p 11211 -d -u root -c 8192
[root@cache01 ~]# killall memcached或pkill memcached
[root@cache01 ~]# netstat -antup | grep 11211
```

**若启动了多个实例Memcached，使用killall或pkill方式就会同时关闭这些实例！因此最好在启动时增加-P参数指定固定的pid文件，这样便于管理不同的实例。示例如下：**

```
[root@cache01 ~]# memcached -m 16m -p 11211 -d -u root -c 8192 -P /var/run/11211.pid
[root@cache01 ~]# memcached -m 16m -p 11212 -d -u root -c 8192 -P /var/run/11212.pid
[root@cache01 ~]# ps -ef | grep memcached | grep -v grep
root       1486      1  0 11:14 ?        00:00:00 memcached -m 16m -p 11211 -d -u root -c 8192 -P /var/run/11211.pid
root       1493      1  0 11:14 ?        00:00:00 memcached -m 16m -p 11212 -d -u root -c 8192 -P /var/run/11212.pid

#此时，即可通过kill命令关闭Memcached
[root@cache01 ~]# kill `cat /var/run/11211.pid`
[root@cache01 ~]# netstat -antup | grep 11211
[root@cache01 ~]# netstat -antup | grep 11212
tcp        0      0 0.0.0.0:11212               0.0.0.0:*                   LISTEN      1493/memcached      
tcp        0      0 :::11212                    :::*                        LISTEN      1493/memcached      
udp        0      0 0.0.0.0:11212               0.0.0.0:*                               1493/memcached      
udp        0      0 :::11212                    :::*                                    1493/memcached      
```

### 6.6 企业工作场景中如何配置Memcached

> - 在企业实际工作中，一般是开发人员提出需求，说要部署一个Memcached数据缓存。运维人员在接到这个不确定的需求后，需要和开发人员深入沟通，进而确定要将内存指定为多大，或者和开发人员商量如何根据具体业务来指定内存缓存的大小。此外，还要确定业务的重要性，进而决定是否采取负载均衡，分布式缓存集群等架构，最后确定要使用多大的并发连接数等。
> - 对于运维人员，部署Memcached一般就是安装Memcached服务器端，把服务启动起来，做好监控，配好开机自启动，基本就OK了，客户端的PHP程序环境一般在安装LNMP环境时都会提前安装Memcached客户端插件，Java程序环境下，开发人员会用第三方的JAR包直接连接Memcached服务。

## 七，安装Memcached客户端

### 7.1 LNMP PHP环境准备

> 详细搭建过程略，请同学们参阅之前的LNMP章节

### 7.2 Memcached缓存PHP扩展插件安装

> 前面已经提过，Memcached分为服务器端软件和客户端插件两部分，这里是Memcached客户端PHP的扩展插件（memcache-2.2.7.tgz）在PHP环境中的安装，用于访问Memcached服务器端数据。
>  PHP的Memcached扩展插件下载地址为：http://pecl.php.net/package/memcache

```
[root@LNMP ~]# wget -q http://pecl.php.net/get/memcache-2.2.7.tgz
[root@LNMP ~]# ls
anaconda-ks.cfg  install.log  install.log.syslog  memcache-2.2.7.tgz
[root@LNMP ~]# tar xf memcache-2.2.7.tgz -C /usr/src/
[root@LNMP ~]# cd /usr/src/memcache-2.2.7/
[root@LNMP memcache-2.2.7]# /usr/local/php/bin/phpize 
Configuring for:
PHP Api Version:         20090626
Zend Module Api No:      20090626
Zend Extension Api No:   220090626
[root@LNMP memcache-2.2.7]# ./configure --enable-memcache --with-php-config=/usr/local/php/bin/php-config
[root@LNMP memcache-2.2.7]# make && make install
[root@LNMP ~]# ls -l /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/
total 244
-rwxr-xr-x. 1 root root 246576 Aug  6 12:53 memcache.so
#最后生成了memcache.so模块就表示memcache扩展插件成功安装
```

### 7.3 配置Memcache客户端，使其生效

**修改PHP的配置文件php.ini，加入Memcache客户端的配置，命令如下：**

```
[root@LNMP ~]# cd /usr/local/php/lib/
[root@LNMP lib]# vim php.ini

#添加如下两行内容到php.ini文件结尾
[root@LNMP lib]# tail -2 php.ini 
extension_dir = "/usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/"
extension=memcache.so
```

### 7.4 重启php-fpm服务使PHP的配置修改生效

**1）检查php-fpm语法：**

```
[root@LNMP lib]# /usr/local/php/sbin/php-fpm -t
[06-Aug-2017 13:03:04] NOTICE: configuration file /usr/local/php5.3.28/etc/php-fpm.conf test is successful
```

**2）重启fpm，命令如下：**

```
[root@LNMP lib]# pkill php-fpm
[root@LNMP lib]# netstat -antup | grep 9000
[root@LNMP lib]# /usr/local/php/sbin/php-fpm
[root@LNMP lib]# netstat -antup | grep 9000
tcp        0      0 127.0.0.1:9000              0.0.0.0:*                   LISTEN      46848/php-fpm 
```

**3）打开浏览器访问phpinfo页面，若出现如下图所示内容，则表示Memcache客户端安装成功**

![QQ截图20170806171013.png-62kB](http://static.zybuluo.com/chensiqi/8ph8zp4asx99putvc6z3fa1f/QQ%E6%88%AA%E5%9B%BE20170806171013.png)

### 7.5 编写测试Memcached服务的PHP脚本

**下面为简单的PHP程序连接Memcached测试脚本**

```
[root@LNMP bbs]# cat op_mem.php 
<?php                               #PHP开始标识

$memcache = new Memcache;           #创建一个Memcache对象

$memcache->connect('192.168.0.240','11211') or die ("Could not connect Mc server");                               #连接Memcached服务器

$memcache->set('key','yunjisuan book');         #设置一个变量到内存中

$get=$memcache->get('key');                 #从内存中取出key

echo $get;                              #输出key值到屏幕

?>                                      #PHP结束标识
[root@LNMP bbs]# ls
index.html  op_mem.php  test_info.php  test_mysql.php
```

**Linux本地测试：**

```
[root@LNMP bbs]# /usr/local/php/bin/php op_mem.php          #调用php解析器
yunjisuan book                          #对应的key的值
```

**windows访问测试：**

![QQ截图20170806172527.png-26.9kB](http://static.zybuluo.com/chensiqi/43u1vm0bsljkauxivayspbv6/QQ%E6%88%AA%E5%9B%BE20170806172527.png)

> 出现上述测试结果，就表示LNMP环境连接Memcached服务成功。

## 八，Memcached应用管理

### 8.1 通过命令管理Memcached

> 运维人员一般可通过在命令行执行“telnet ip port”的方式登陆到Memcached，然后执行一些管理的命令，除了telnet外，nc命令也是一个不错的管理Memcached服务的命令！
>  有关“telnet ip port”命令前文已经介绍过了，下面将重点讲解通过nc管理及监控Memcached的一些常见操作。

**以下通过脚本模拟用过户插入及删除数据来监控Memcached服务是否正常的示例**

#### 8.1.1 检查Memcached服务是否异常的监控脚本

**脚本内容如下：**

```
[root@cache01 scripts]# pwd
/server/scripts
[root@cache01 scripts]# cat mon_mc.sh 
#!/bin/bash

export MemcachedIp=$1
export MemcachedPort=$2
export NcCmd="nc $MemcachedIp $MemcachedPort"
export MD5="3fe396c01f03425cb5e2da8186eb090d"

USAGE(){
echo "$0 MemcachedIp MemcachedPort"
exit 3
}

[ $# -ne 2 ] && USAGE
printf "set $MD5 0 0 9\r\nyunjisuan\r\n" | $NcCmd >/dev/null 2>&1 
if [ $? -eq 0 ];then
    if [ `printf "get $MD5\r\n"|$NcCmd|grep yunjisuan|wc -l` -eq 1 ];then
        echo "Memcached status is ok"
        printf "delete $MD5\r\n"|$NcCmd >/dev/null 2>&1
        exit 0
    else
        echo "Memcached status is error"
        exit 2
    fi
else
    echo "Could not connect Mc server"
    exit 2
fi
```

**Memcached服务正常的情况下，测试检验脚本**

```
[root@cache01 scripts]# sh mon_mc.sh 127.0.0.1 11211
Memcached status is ok
```

**关闭Memcached服务，再测试脚本**

```
[root@cache01 scripts]# kill `cat /var/run/11211.pid`
[root@cache01 scripts]# netstat -antup | grep 11211
[root@cache01 scripts]# sh mon_mc.sh 127.0.0.1 11211
Could not connect Mc server
```

**最后开启Memcached服务，测试检验脚本**

```
[root@cache01 scripts]# memcached -m 16m -p 11211 -d -u root -c 8192 -P /var/run/11211.pid
[root@cache01 scripts]# netstat -antup | grep 11211
tcp        0      0 0.0.0.0:11211               0.0.0.0:*                   LISTEN      2053/memcached      
tcp        0      0 :::11211                    :::*                        LISTEN      2053/memcached      
udp        0      0 0.0.0.0:11211               0.0.0.0:*                               2053/memcached      
udp        0      0 :::11211                    :::*                                    2053/memcached      
[root@cache01 scripts]# sh mon_mc.sh 127.0.0.1 11211
Memcached status is ok
```

> 通过上面的测试，我们发现使用监控脚本监控Memcached服务是否异常的运行是正确的。

#### 8.1.2 通过nc命令查看Memcached服务的运行状态信息

**查看Memcached服务运行状态信息的nc命令如下：**

```
[root@cache01 scripts]# printf "stats\r\n"|nc 127.0.0.1 11211
STAT pid 2053
STAT uptime 225
STAT time 1502044003
STAT version 1.4.4
STAT pointer_size 64
STAT rusage_user 0.002999
STAT rusage_system 0.005999
STAT curr_connections 10
STAT total_connections 14
STAT connection_structures 11
STAT cmd_get 1
STAT cmd_set 1
STAT cmd_flush 0
STAT get_hits 1
STAT get_misses 0
STAT delete_misses 0
STAT delete_hits 1
STAT incr_misses 0
STAT incr_hits 0
STAT decr_misses 0
STAT decr_hits 0
STAT cas_misses 0
STAT cas_hits 0
STAT cas_badval 0
STAT auth_cmds 0
STAT auth_errors 0
STAT bytes_read 141
STAT bytes_written 77
STAT limit_maxbytes 16777216
STAT accepting_conns 1
STAT listen_disabled_num 0
STAT threads 4
STAT conn_yields 0
STAT bytes 0
STAT curr_items 0
STAT total_items 1
STAT evictions 0
END
```

> 除了上述输出的全部状态以外，Memcached还支持通过以下命令的输入来查看Memcached的部分运行状态信息。目前管理Memcached的命令见下表：

| Memcached状态命令 | 说明                                      |
| ----------------- | ----------------------------------------- |
| stats             | 统计Memcached的各种信息                   |
| stats settings    | 查看一些memcached的设置信息，例如：线程数 |
| stats slabs       | 查看slabs相关情况，例如：chunksize长度    |
| stats items       | 查看items相关情况                         |
| stats sizes       | 查看items个数和大小                       |
| stats reset       | 清理统计数据                              |

> 例如，要查看Memcached的统计信息，可先执行“telnet ip 监听端口”命令，登陆成功之后执行stats命令，具体过程如下：

```
[root@LNMP ~]# which telnet
/usr/bin/telnet
[root@LNMP ~]# telnet 192.168.0.240 11211
Trying 192.168.0.240...
Connected to 192.168.0.240.
Escape character is '^]'.
stats
STAT pid 2053               #启动的进程id
STAT uptime 682             #到目前1位置启动了多少秒
STAT time 1502044460
STAT version 1.4.4          #Memcached的版本信息
STAT pointer_size 64
STAT rusage_user 0.007998
STAT rusage_system 0.014997
STAT curr_connections 10        #当前的并发连接数
STAT total_connections 15       #总的连接数
STAT connection_structures 11
STAT cmd_get 1                  #执行的get命令的次数
STAT cmd_set 1                  #执行的set命令的次数
STAT cmd_flush 0                #执行flush命令的次数
STAT get_hits 1                 #get的命中数
STAT get_misses 0               #get的非命中数
STAT delete_misses 0
STAT delete_hits 1
STAT incr_misses 0
STAT incr_hits 0
STAT decr_misses 0
STAT decr_hits 0
STAT cas_misses 0
STAT cas_hits 0
STAT cas_badval 0
STAT auth_cmds 0
STAT auth_errors 0
STAT bytes_read 148
STAT bytes_written 848
STAT limit_maxbytes 16777216            #允许使用的最大内存容量
STAT accepting_conns 1
STAT listen_disabled_num 0
STAT threads 4
STAT conn_yields 0
STAT bytes 0
STAT curr_items 0
STAT total_items 1
STAT evictions 0
END
```

**使用printf及nc命令获取状态信息更佳，因为无需交互，可以使用以下脚本批量操作**

```
[root@LNMP ~]# printf "stats\r\n"|nc 192.168.0.240 11211
STAT pid 2053
STAT uptime 1270
STAT time 1502045048
STAT version 1.4.4
STAT pointer_size 64
STAT rusage_user 0.011998
STAT rusage_system 0.029995
STAT curr_connections 10
STAT total_connections 24
STAT connection_structures 11
#以下省略若干...
```

### 8.2 Memcached状态信息详细说明

**Memcached状态信息的详细说明，同学们可以通过该表来了解相关信息：**

![QQ截图20170806184641.png-59.9kB](http://static.zybuluo.com/chensiqi/19qkbvwrjcemzc4rr0t38h7r/QQ%E6%88%AA%E5%9B%BE20170806184641.png)

![QQ截图20170806184659.png-224.5kB](http://static.zybuluo.com/chensiqi/u7trp7ayiuiebz96k74756k8/QQ%E6%88%AA%E5%9B%BE20170806184659.png)

![QQ截图20170806184709.png-114.7kB](http://static.zybuluo.com/chensiqi/o844k0zq9bueaigwls38nxla/QQ%E6%88%AA%E5%9B%BE20170806184709.png)

### 8.3 通过memadmin php 工具展示Memcached状态信息

> 运维人员除了通过命令行管理Memcached以外，还可以使用很多的第三方开源管理工具，例如：通过memadmin工具管理及监控Memcached状态，软件的名称为“memadmin-1.0.12.tar.gz”,这个软件的部署十分简单，功能非常强大，但是依赖于PHP环境，推荐同学们初学Memcached时使用。

#### 8.3.1 部署memadmin php工具

> 因为这个软件是基于PHP程序的，因此，需要有PHP的环境才行，本章已经准备好了LNMP的环境，因此，可以简单地将上述“memadmin-1.0.12.tar.gz”解压到虚拟主机站点目录下，这里还是以bbs虚拟主机为例进行讲解。

**部署的关键命令如下：**

```
[root@LNMP ~]# ls -l memadmin-1.0.12.tar.gz 
-rw-r--r--. 1 root root 196734 Aug  6 14:55 memadmin-1.0.12.tar.gz
[root@LNMP ~]# tar xf memadmin-1.0.12.tar.gz 
[root@LNMP ~]# mv memadmin /usr/local/nginx/html/bbs/
```

#### 8.3.2 采用IP或者解析好的域名进行访问

**LNMP服务器的nginx.conf配置文件内容如下：**

```
[root@LNMP nginx]# cat conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
            root   html/www;
            index  index.html index.htm;
        }
    }
    server {
        listen       80;
        server_name  bbs.yunjisuan.com;
        location / {
            root   html/bbs;
            index  index.html index.htm;
        }
    location ~ .*\.(php|php5)$ {
        root   html/bbs;    
        fastcgi_pass    127.0.0.1:9000;
            fastcgi_index   index.php;
        include         fastcgi.conf;
    }
    }
}
```

**打开浏览器输入域名:http://bbs.yunjisuan.com/memadmin/index.php**

**程序的管理页面如下图所示：**

![QQ截图20170806191043.png-40.2kB](http://static.zybuluo.com/chensiqi/fa537cnnonomstekfq2rerv8/QQ%E6%88%AA%E5%9B%BE20170806191043.png)

> **提示：**
>  用户名和密码都是admin

**登陆之后的页面如下图所示：**

![QQ截图20170806191340.png-19.3kB](http://static.zybuluo.com/chensiqi/ptw8tzbym9yogyydj8xof5v5/QQ%E6%88%AA%E5%9B%BE20170806191340.png)

**然后点击管理，进入下图：**

![QQ截图20170806191746.png-39.4kB](http://static.zybuluo.com/chensiqi/cs4ngplw582beubcxd35cgoj/QQ%E6%88%AA%E5%9B%BE20170806191746.png)

> memadmin对单个或少量的Memcached服务器数量的监控，维护管理还是很直观容易的。
>  剩下的功能很容易懂，同学们自己玩。

## 九，Memcached服务应用的优化

### 9.1 Memcached服务应用优化案例

> 下面给大家介绍一个Memcached服务应用优化的企业案例。用户访问网站打开页面很慢，经运维人员排查后，发现MySQL数据库负载很高，load值大概为20~30，如下：

```
[root@LNMP ~]# uptime

10:41:01 up 15:02, 4 users, load average: 20, 15, 10

#登陆数据库后，使用“show full processlist；”查看，或者在Linux命令行使用下面的命令查看：
mysql -uroot -p123123 -e "show full processlist"|grep -vi sleep
```

> 发现数据库中像“LIKE‘%杜冷丁%’”这样的SQL语句特别多，导致数据库负载很高，我们都知道像LIKE‘%杜冷丁%’这样的SQL语句对于MySQl数据库来说，利用索引没有太大的优化余地。
>  打开该网站首页查看，发现该首页有一个搜索框，根据前面查看的SQL语句形式，可以推测上述“LIKE%杜冷丁%”应该是搜索框的语句带来的结果。事后从这个公司的数据库维护人员处得到了证实，请问如果是你，你该如何优化解决当前的问题呢？

![QQ截图20170806211125.png-21.8kB](http://static.zybuluo.com/chensiqi/85qk8lq6eui9atzspwulroly/QQ%E6%88%AA%E5%9B%BE20170806211125.png)

**大概的优化方案思路如下：**

1. 看是否可以从业务上整改，例如，只有在用户登陆后才可以进行搜索，通过这种改进来减少搜索的次数，达到减轻数据库服务压力的目的。
2. 如果有大量频繁的搜索SQL语句，很有可能是有网络爬虫在爬我们的网站，可以通过分析Web日志或者网络连接状态，封掉这些非正常的搜索请求。
3. 为主库配置多个从库，然后实现数据的读写分离，让“LIKE‘%杜冷丁%’”这样的查询去多个从库查，从而减轻主库的读写压力。
4. 在数据库前端加Memcached缓存服务器，这个效果在所有的方法里是最好的。
5. 在数据库里使用“LIKE‘%杜冷丁%’”实现搜索，并非明智的选择，可以通过Sphinx等搜索服务实现用户搜索。
6. 还可以利用C，Ruby等开发语言开发程序，部署计算服务器实现每日读数据库计算全量搜索索引，然后保存在提供搜索的Web服务器上，除了设定每日计算全量搜索索引外，可以每分钟读单独的1~2个从库做增量计算索引。这是大公司针对站内搜索采取的基本解决方案之一。

> - 其中1，2，3是短期的方案，简单容易实施。4，5，6是长期的方案目标。
> - 这个案例的最终问题是，网站前端有爬虫爬站，通过分析Web日志获取到爬站的IP，临时封掉后，MySQL数据库负载立刻下降了很多，网站服务恢复正常。

**下图是针对方案6给出的大型网站搜索集群架构逻辑案例图**

![QQ截图20170806212530.png-172.8kB](http://static.zybuluo.com/chensiqi/hoi1fyva2yx9reb71uhexg2i/QQ%E6%88%AA%E5%9B%BE20170806212530.png)

### 9.2 Memcached服务优化策略

> Memcached服务的额安装配置简单，几乎不需要优化就可以跑的很好，在特殊的大并发高访问量的场合才需要进行一些优化。

#### 9.2.1 提高Memcached访问命中率是优化的最关键指标。

> 例如：每次新增数据到数据库的同时，就将数据写入或者复制一份到Memcached里，然后从业务逻辑上让程序优先读缓存，没有数据再查数据库。

#### 9.2.2 提高内存利用率，减少内存浪费

- 减少chunk内存空间浪费的调优方法为，根据业务数据的大小，利用-n参数设定chunk的初始值，及通过-f参数factor增长因子设置chunk的大小尽可能接近业务数据的大小。
- 减少slab的浪费，设定slab的大小为chunk的整数倍。
- 采用一致性哈希分布式缓存集群架构

> 当网站后端的数据库数据量很大时，单台Memcached就无法存放绝大部分的数据库内数据，导致Memcached服务的命中率很低，此时，可以采用一致性哈希分布式缓存集群架构，提升网站的命中率，一致性哈希可以由程序实现或者支持一致性哈希算法的负载均衡器实现。

### 9.3 Memcached服务在大型站点中的架构优化

#### 9.3.1 大型网站的架构设计原则

> 当访问量增大时，在整个网站集群架构中最先出现瓶颈的几乎都是后端的数据库或用于存储的服务器，在企业生产工作中应尽量把用户的访问请求往整个网站架构的最前面推，即当用户请求数据时，越是在靠近用户端返回数据，效果就越好。

#### 9.3.2 大型网站的数据库架构常见设计

> 为了缓解应用对数据库的高并发访问压力，大型网站都会在数据库层配置数据读写分离，并提供读的数据库做负载均衡，逻辑图如下图所示：其中是以Amoeba软件作为读写分离说明的。

![QQ截图20170806214351.png-45.4kB](http://static.zybuluo.com/chensiqi/lka7oyexj0erc7l1v6xo1om4/QQ%E6%88%AA%E5%9B%BE20170806214351.png)

> 由于访问数据库读写的是磁盘，所以，对于高并发的业务场景，上述读写分离的架构还是远远解决不了问题的，根据离访问用户越近效率越高以及用内存代替磁盘的架构思想，可以在数据库架构的前端部署Memcached服务作为缓存区，最大限度地把对数据库查询信息保存在Memcached服务的内存中，这样前端的应用服务就能够迅速地从Memcached中读取到原本在数据库中才能读取到的数据，从而加快了网站的访问速度。

#### 9.3.3 分布式Memcached缓存服务架构

> 由于单台Memcached服务器的内存容量是有限的，并且单台也存在单点故障，因此，大型网站往往会将多个memcached服务器组合成集群提供服务，那么怎么组合效率才更高呢？

**（1）缓存服务器使用常规负载均衡模式的问题**

> 在常规负载均衡模式中，集群节点上的程序和数据都是一样的，例如Web集群。而缓存集群设计下的所有节点缓存的数据就不能都一样，因为这样一来，访问数据命中率会非常低下，低下的原因主要有两个：

- 当客户端访问缓存A无数据时，就会去查后端数据库，然后把查到的数据放入缓存A中一份，当下次客户端访问相同的数据时，可能被分配到的是缓存B，结果B中还是无数据，这样客户端又会去查后端数据库，这样就会给数据库造成了访问压力，同时造成缓存服务器的命中率很低。
- 在集群运行一段时间后，所有缓存的数据可能交叉并极其接近，如果数据库数据量远大于单台Memcached服务器的内存总量，Memcached的命中率也会非常低下。理想的情况是所有缓存数据之和接近数据库总的数据容量，这样缓存的效率才会高。
- 解决上述问题的方案最常见的就是使用哈希算法或一致性哈希算法将需要同样数据的请求始终调度到同一台缓存服务器，提升访问命中率。其次，所有缓存服务器缓存的数据都是不同的，所有服务器缓存的数据之和接近数据库总的数据容量，使得缓存集群无需换入换出，达到更高的命中率。

**下图是缓存服务器使用常规负载均衡模式的问题图解**

![QQ截图20170806220532.png-98.7kB](http://static.zybuluo.com/chensiqi/cyxyrqn1iux2vjengkt50zzo/QQ%E6%88%AA%E5%9B%BE20170806220532.png)

![QQ截图20170806220545.png-103.4kB](http://static.zybuluo.com/chensiqi/4qlt5uj9d8o0g6ol3oay5eh4/QQ%E6%88%AA%E5%9B%BE20170806220545.png)

**（2）分布式缓存集群的优劣势**

> - Memcached支持分布式集群，其中的方法之一就是在客户端应用程序上进行改造。例如：可以根据key适当进行有规律的封装。比如以用户为主的网站来说，每个用户都有UserID，那么可以按照固定的ID来进行提取和存取，假设以1开头的用户保存在第一台Memcached服务器上，以2开头的用过户的数据保存在第二台Memcached服务器上，那么存取数据时都会按照UserID来进行相应的转换和存取。
> - 但是这样也有缺点，就是需要对UserID进行判断，如果业务不一致，或者是其他类型的应用，可能不是那么合适，此时可以根据自己的实际业务来进行考虑，或者去想更合适的方法
> - 此外，还可以在应用服务器上通过程序用哈希算法或一致性哈希算法调度Memcached服务器，所有Memcached服务器的地址池可以简单地配在每个程序的配置文件里。并且可在负载均衡器上使用哈希算法或一致性哈希算法，不过此时需要负载均衡器的支持。

#### 9.3.4 分布式缓存集群设计思想

1）每一台Memcached服务器的内容都是不一样的。这些Memcached服务器缓存的内容加起来接近整个数据库的数据容量。
 2）通过在客户端程序或者Memcached的负载均衡器上用hash算法，让同一数据内容都分配到一个Memcached服务器。
 3）普通的hash算法对于节点宕机会带来大量的缓存数据流动（失效），可能会引起雪崩效应。
 4）一致性哈希算法（还可以带虚拟节点）可以让缓存节点宕机对节点的数据流动（失效）降到最低。

#### 9.3.5 分布式Memcached缓存集群的调度算法

**（1）取模计算hash**

优点：简单，分散性优秀
 缺点：添加/删除服务器时，缓存重组代价巨大，影响命中率

例：将26个字母缓存到了3个节点，此时新增加一个节点，访问命中率将下降23%如下表：

![QQ截图20170806223831.png-52.5kB](http://static.zybuluo.com/chensiqi/xek4hyazjq0b0ot66nrpdxkm/QQ%E6%88%AA%E5%9B%BE20170806223831.png)

**（2）Consistent hash（一致性哈希算法）**

> - 一致性哈希算法（consistent hash）是一种特殊的hash算法，简单地说，在移除以及添加一个cache节点时，它能够尽可能小地改变已存在key的映射关系，让缓存服务器缓存的内容受到的影响最小。
> - consistent  hash算法原理如下图所示，首先现象一个0~2^32-1次方的数值空间，将这个空间想象成一个首（0）尾（2^32-1）相接的圆环，然后算出不同Memcached服务器节点的哈希值（0~（2^32-1）之间），将这些值分散到上述的圆环上，接着用同样的方法算出存储不同数据的键的哈希值并映射到相同的圆环上，最后从数据映射到的位置开始顺时针查找，将键对应的数据保存到查找到的第一个服务器上，如下图所示：

![QQ截图20170806224650.png-103.3kB](http://static.zybuluo.com/chensiqi/sy9v2t9po6s2v97kjub7kkg1/QQ%E6%88%AA%E5%9B%BE20170806224650.png)

**假如我们要添加node5服务器，如下图所示：**

![QQ截图20170806224825.png-116.8kB](http://static.zybuluo.com/chensiqi/jo9vwumwzjs73g8vd1fr49df/QQ%E6%88%AA%E5%9B%BE20170806224825.png)

> - 可以看到添加node5服务器后，缓存的数据只影响node2到node5之间的数据范围，即node4的一部分数据，缓存到了node5，其他的缓存服务器没有受到影响，当移除服务器时，原理和添加服务器时一样，不再赘述。
> - 这就是一致性哈希算法的作用，可以最大限度地减少键的重新分布。该算法的实现方法还可以采用更好的虚拟节点的策略思路，一致性哈希算法可以通过在前端使用程序实现或者通过负载均衡器实现。

## 十，Memcached在集群中session共享案例

### 10.1 Memcached在集群中的session共享存储实战

**以下是PHP Web环境集群的session共享存储设置。**

```
#默认php.ini中session的类型和配置路径为：
[root@LNMP ~]# awk '/session.save_handler/{print NR,$0}/session.save_path/{print NR,$0}' /usr/local/php/lib/php.ini 
1461 session.save_handler = files               #修改本行数据
1469 ;     session.save_path = "N;/path"
1485 ;     session.save_path = "N;MODE;/path"
1490 ;session.save_path = "/tmp"                #修改本行数据
1566 ;       (see session.save_path above), then garbage collection does *not*

#修改成如下配置：
[root@LNMP ~]# sed -n '1461p;1490p' /usr/local/php/lib/php.ini
session.save_handler = memcache
session.save_path = "tcp://192.168.0.240:11211"

#重启php-fpm服务
[root@LNMP ~]# pkill php-fpm
[root@LNMP ~]# netstat -antup | grep 9000
[root@LNMP ~]# /usr/local/php/sbin/php-fpm 
[root@LNMP ~]# netstat -antup | grep 9000
tcp        0      0 127.0.0.1:9000              0.0.0.0:*                   LISTEN      1393/php-fpm 
```

> **说明：**
>  192.168.0.240：11211为Memcached数据库缓存的IP及端口
>  上述配置适合LNMP/LAMP环境
>  Memcached服务器也可以是多台，并且可通过hash算法调度
>  配置完毕通过phpinfo页面查看效果如下图所示：

![QQ截图20170807092852.png-79.3kB](http://static.zybuluo.com/chensiqi/hk5p1735cvp4ge9wrj11r9mg/QQ%E6%88%AA%E5%9B%BE20170807092852.png)

### 10.2 Memcached在集群中的session共享存储的优缺点

**优点：**

1）读写速度上会比普通files速度快很多
 2）可以解决多个服务器共用session的难题

**缺点：**

1）session数据都保存在memory中，持久化方面有所欠缺，但对session数据来说不是问题。
 2）一般是单台，如果部署多台，多台之间无法数据同步。通过hash算法分配依然有session丢失的问题。

**对于上面的缺点，解决思路如下：**

1）可以用其他的持久化系统存储sessions，例如：redis，ttserver来替代Memcached
 2）高性能高并发场景，cookies效率比session要好很多，因此，大网站都会用cookies解决会话共享问题
 3）有的已经就业了的同学通过牺牲LB的负载均衡的策略实现，例如：lvs-p,nginx ip_hash等，但这些不是好的方法。

## 十一，本章重点回顾

1. Memcached在企业中的应用场景案例
2. Memcached企业中常见用途读写工作流程
3. Memcached特性与工作原理机制
4. Memcached内存管理核心机制
5. Memcached检测过期与删除工作原理机制
6. Memcached服务器端安装与应用实践
7. Memcached客户端插件安装及配置
8. Memcached的运行状态信息监控
9. Memcached服务应用优化策略及案例
10. Memcached在集群后端作为session共享案例