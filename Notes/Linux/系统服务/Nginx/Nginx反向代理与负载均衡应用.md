[TOC]

## 1.1 集群简介

> - 简单地说，集群就是指一组（若干个）相互独立的计算机，利用高速通信网络组成的一个较大的计算机服务系统，每个集群节点（即集群中的每台计算机）都是运行各自服务的独立服务器。这些服务器之间可以彼此通信，协同向用户提供应用程序，系统资源和数据，并以单一系统的模式加以管理。当用户客户机请求集群系统时，集群给用户的感觉就是一个单一独立的服务器，而实际上用户请求的是一组集群服务器。
> - 打开谷歌，百度的页面，看起来好简单，也许你觉得用几分钟就可以制作出相似的网页，而实际上，这个页面的背后是由成千上万台服务器集群协同工作的结果。而这么多的服务器维护和管理，以及相互协调工作也许就是同学们未来的工作职责了。
> - 若要用一句话描述集群，即一堆服务器合作做同一件事，这些机器可能需要整个技术团队架构，设计和统一协调管理，这些机器可以分布在一个机房，也可以分布在全国全球各个地区的多个机房。

## 1.2 为什么要使用集群

（1）高性能

> 一些国家重要的计算密集型应用（如天气预报，核试验模拟等），需要计算机有很强的运算处理能力。以全世界现有的技术，即使是大型机，其计算能力也是有限的，很难单独完成此任务。因为计算时间可能会相当长，也许几天，甚至几年或更久。因此，对于这类复杂的计算业务，便使用了计算机集群技术，集中几十上百台，甚至成千上万台计算机进行计算。

![QQ截图20170726094121.png-298.5kB](http://static.zybuluo.com/chensiqi/c3u53lax6rsew0z4cq7b73q9/QQ%E6%88%AA%E5%9B%BE20170726094121.png)

假如你配一个LNMP环境，每次只需要服务10个并发请求，那么单台服务器一定会比多个服务器集群要快。只有当并发或总请求数量超过单台服务器的承受能力时，服务器集群才会体现出优势。

（2）价格有效性

> - 通常一套系统集群架构，只需要几台或数十台服务器主机即可。与动辄价值上百万元的专用超级计算机相比便宜了很多。在达到同样性能需求的条件下，采用计算机集群架构比采用同等运算能力的大型计算机具有更高的性价比。
> - 早期的淘宝，支付宝的数据库等核心系统就是使用上百万元的小型机服务器。后因使用维护成本太高以及扩展设备费用成几何级数翻倍，甚至成为扩展瓶颈，人员维护也十分困难，最终使用PC服务器集群替换之，比如，把数据库系统从小机结合Oracle数据库迁移到MySQL开源数据库结合PC服务器上来。不但成本下降了，扩展和维护也更容易了。

（3）可伸缩性

**当服务负载，压力增长时，针对集群系统进行较简单的扩展即可满足需求，且不会降低服务质量。**

> 通常情况下，硬件设备若想扩展性能，不得不增加新的CPU和存储器设备，如果加不上去了，就不得不够买更高性能的服务器，就拿我们现在的服务器来讲，可以增加的设备总是有限的。如果采用集群技术，则只需要将新的单个服务器加入现有集群架构中即可，从访问的客户角度来看，系统服务无论是连续性还是性能上都几乎没有变化，系统在不知不觉中完成了升级，加大了访问能力，轻松地实现了扩展。集群系统中的节点数目可以增长到几千乃至上万个，其伸缩性远超过单台超级计算机。

（4）高可用性

> - 单一的计算机系统总会面临设备损毁的问题，如CPU，内存，主板，电源，硬盘等，只要一个部件坏掉，这个计算机系统就可能会宕机，无法正常提供服务。在集群系统中，尽管部分硬件和软件也还是会发生故障，但整个系统的服务可以是7*24小时可用的。

- 集群架构技术可以使得系统在若干硬件设备故障发生时仍可以继续工作，这样就将系统的停机时间减少到了最小。集群系统在提高系统可靠性的同时，也大大减小了系统故障带来的业务损失，目前几乎100%的互联网网站都要求7*24小时提供服务。

（5）透明性

> 多个独立计算机组成的松耦合集群系统构成一个虚拟服务器。用户或客户端程序访问集群系统时，就像访问一台高性能，高可用的服务器一样，集群中一部分服务器的上线，下线不会中断整个系统服务，这对用户也是透明的。

（6）可管理性

> 整个系统可能在物理上很大，但其实容易管理，就像管理一个单一映像系统一样。在理想状况下，软硬件模块的插入能做到即插即用。

（7）可编程性

> 在集群系统上，容易开发及修改各类应用程序。

## 1.3 集群的常见分类

### 1.3.1 集群的常见分类

**计算机集群架构按功能和结构可以分成以下几类：**

- 负载均衡集群，简称LBC或者LB
- 高可用性集群，简称HAC
- 高性能计算集群，简称HPC
- 网格计算集群

**提示：**
 负载均衡集群和高可用性集群是互联网行业常用的集群架构模式，也是我们要学习的重点。

### 1.3.2 不同种类的集群介绍

**（1）负载均衡集群**

> -负载均衡集群为企业提供了更为实用，性价比更高的系统架构解决方案。负载均衡集群可以把很多客户集中的访问请求负载压力尽可能平均地分摊在计算机集群中处理。客户访问请求负载通常包括应用程序处理负载和网络流量负载。这样的系统非常适合使用同一组应用程序为大量用户提供服务的模式，每个节点都可以承担一定的访问请求负载压力，并且可以实现访问请求在各节点之间动态分配，以实现负载均衡。
>  -负载均衡集群运行时，一般是通过一个或多个前端负载均衡器将客户访问请求分发到后端的一组服务器上，从而达到整个系统的高性能和高可用性。一般高可用性集群和负载均衡集群会使用类似的技术，或同时具有高可用性与负载均衡的特点。

**负载均衡集群的作用为：**

- 分摊用户访问请求及数据流量（负载均衡）
- 保持业务连续性，即7*24小时服务（高可用性）。
- 应用于Web业务及数据库从库等服务器的业务

负载均衡集群典型的开源软件包括LVS，Nginx，Haproxy等。如下图所示：

![QQ截图20170726181402.png-193.2kB](http://static.zybuluo.com/chensiqi/fsxovr6y6zf2kd515es4yry0/QQ%E6%88%AA%E5%9B%BE20170726181402.png)

**提示：**
 不同的业务会有若干秒的切换时间，DB业务明显长于Web业务切换时间。

**（2）高可用性集群**

> 一般是指在集群中任意一个节点失效的情况下，该节点上的所有任务会自动转移到其他正常的节点上。此过程并不影响整个集群的运行。
>
> - 当集群中的一个节点系统发生故障时，运行着的集群服务会迅速作出反应，将该系统的服务分配到集群中其他正在工作的系统上运行。考虑到计算机硬件和软件的容错性，高可用性集群的主要目的是使集群的整体服务尽可能可用。如果高可用性集群中的主节点发生了故障，那么这段时间内将由备节点代替它。备节点通常是主节点的镜像。当它代替主节点时，它可以完全接管主节点（包括IP地址及其他资源）提供服务，因此，使集群系统环境对于用户来说是一致的，既不会影响用户的访问。
> - 高可用性集群使服务器系统的运行速度和响应速度会尽可能的快。他们经常利用在多台机器上运行的冗余节点和服务来相互跟踪。如果某个节点失败，它的替补者将在几秒钟或更短时间内接管它的职责。因此，对于用户而言，集群里的任意一台机器宕机，业务都不会受影响（理论情况下）。

**高可用性集群的作用为：**

- 当一台机器宕机时，另外一台机器接管宕机的机器的IP资源和服务资源，提供服务。
- 常用于不易实现负载均衡的应用，比如负载均衡器，主数据库，主存储对之间。

高可用性集群常用的开源软件包括Keepalived，Heartbeat等，其架构图如下图所示：

![QQ截图20170726203115.png-193.3kB](http://static.zybuluo.com/chensiqi/eekzz1r4ijrjo2kkg8minb86/QQ%E6%88%AA%E5%9B%BE20170726203115.png)

**（3）高性能计算集群**

> 高性能计算集群也称并行计算。通常，高性能计算集群涉及为集群开发的并行应用程序，以解决复杂的科学问题（天气预报，石油勘探，核反应模拟等）。高性能计算集群对外就好像一个超级计算机，这种超级计算机内部由数十至上万个独立服务器组成，并且在公共消息传递层上进行通信以运行并行应用程序。在生产环境中实际就是把任务切成蛋糕，然后下发到集群节点计算，计算后返回结果，然后继续领新任务计算，如此往复。

**（4）网格计算集群**

> 由于很少用到，在此略

**特别提示：**
 在互联网网站运维中，比较常用的就是负载均衡集群和高可用性集群

## 1.4 常用的集群软硬件介绍及选型

### 1.4.1 企业运维中常见的集群软硬件产品

互联网企业常用的开源集群软件有：Nginx，LVS，Haproxy，Keepalived，heartbeat。

互联网企业常用的商业集群硬件有：F5，Netscaler，Radware，A10等，工作模式相当于Haproxy的工作模式。

淘宝，赶集网，新浪等公司曾使用过Netscaler负载均衡产品。集群硬件Netscaler的产品图如下图所示：

![QQ截图20170726204518.png-45kB](http://static.zybuluo.com/chensiqi/e94uqbjrgjqzrhlo0aeb583k/QQ%E6%88%AA%E5%9B%BE20170726204518.png)

集群硬件F5产品如下图所示：

![QQ截图20170726204604.png-32.5kB](http://static.zybuluo.com/chensiqi/0bw0mlvnvkyru68d2zpggksw/QQ%E6%88%AA%E5%9B%BE20170726204604.png)

### 1.4.2 对于集群软硬件产品如何选型

下面是我对同学们的基本选择建议，更多的建议等大家学完负载均衡内容后再细分讲解。

- 当企业业务重要，技术力量又薄弱，并且希望出钱购买产品及获取更好的服务时，可以选择硬件负载均衡产品，如F5，Netscaler，Radware等，此类公司多为传统的大型非互联网企业，如银行，证券，金融业及宝马，奔驰公司等
- 对于门户网站来说，大多会并用软件及硬件产品来分担单一产品的风险，如淘宝，腾讯，新浪等。融资了的企业会购买硬件产品，如赶集等网站。
- 中小型互联网企业，由于起步阶段无利润可赚或者利润很低，会希望通过使用开源免费的方案来解决问题，因此会雇佣专门的运维人员进行维护。例如：51CTO等

> 相比较而言，商业的负载均衡产品成本高，性能好，更稳定，缺点是不能二次开发，开源的负载均衡软件对运维人员的能力要求较高，如果运维及开发能力强，那么开源的负载均衡软件是不错的选择，目前的互联网行业更倾向于使用开源的负载均衡软件。

### 1.4.3 如何选择开源集群软件产品

> - 中小企业互联网公司网站在并发访问和总访问量不是很大的情况下，建议首选Nginx负载均衡，理由是Nginx负载均衡配置简单，使用方便，安全稳定，社区活跃，使用的人逐渐增多，成为流行趋势，另外一个实现负载均衡的类似产品为Haproxy（支持L4和L7负载，同样优秀，但社区不如Nginx活跃）。
> - 如果要考虑Nginx负载均衡的高可用功能，建议首选Keepalived软件，理由是安装和配置简单，使用方便，安全稳定，与Keepalived服务类似的高可用软件还有Heartbeat（使用比较复杂，并不建议初学者使用）
> - 如果是大型企业互联网公司，负载均衡产品可以使用LVS+Keepalived在前端做四层转发（一般是主备或主主，如果需要扩展可以使用DNS或前端使用OSPF），后端使用Nginx或者Haproxy做7层转发（可以扩展到百台），再后面是应用服务器，如果是数据库与存储的负载均衡和高可用，建议选择LVS+Heartbeat，LVS支持TCP转发且DR模式效率很高，Heartbeat可以配合drbd，不但可以进行VIP的切换，还可以支持块设备级别的数据同步（drbd），以及资源服务的管理。

## 1.5 Nginx负载均衡集群介绍

### 1.5.1 搭建负载均衡服务的需求

**负载均衡集群提供了一种廉价，有效，透明的方法，来扩展网络设备和服务器的负载，带宽和吞吐量，同时加强了网络数据处理能力，提高了网络的灵活性和可用性。**

**搭建负载均衡服务的需求如下：**

（1）把单台计算机无法承受的大规模并发访问或数据流量分担到多台节点设备上，分别进行处理，减少用户等待响应的时间，提升用户体验。
 （2）单个重负载的运算分担到多台节点设备上做并行处理，每个节点设备处理结束后，将结果汇总，返回给用户，系统处理能力得到大幅度提高。
 （3）7*24小时的服务保证，任意一个或多个有限后面节点设备宕机，不能影响业务。

> 在负载均衡集群中，同组集群的所有计算机节点都应该提供相同的服务。集群负载均衡器会截获所有对该服务的入站请求。然后将这些请求尽可能地平均地分配在所有集群节点上。

### 1.5.2 Nginx负载均衡集群介绍

（1）反向代理与负载均衡概念简介

> - 严格地说，Nginx仅仅是作为Nginx Proxy反向代理使用的，因为这个反向代理功能表现的效果是负载均衡集群的效果，所以本文称之为Nginx负载均衡。那么，反向代理和负载均衡有什么区别呢？
> - 普通负载均衡软件，例如大名鼎鼎的LVS，其实功能只是对请求数据包的转发（也可能会改写数据包），传递，其中DR模式明显的特征是从负载均衡下面的节点服务器来看，接收到的请求还是来自访问负载均衡器的客户端的真实用户，而反向代理就不一样了，反向代理接收访问用户的请求后，会代理用户重新发起请求代理下的节点服务器，最后把数据返回给客户端用户，在节点服务器看来，访问的节点服务器的客户端用户就是反向代理服务器了，而非真实的网站访问用户。
> - 一句话，LVS等的负载均衡是转发用户请求的数据包，而Nginx反向代理是接收用户的请求然后重新发起请求去请求其后面的节点。

（2）实现Nginx负载均衡的组件说明

**实现Nginx负载均衡的组件主要有两个，如下表：**

![QQ截图20170726213351.png-51.8kB](http://static.zybuluo.com/chensiqi/a0gyrbvrs7n7hlp5mf7kqwtr/QQ%E6%88%AA%E5%9B%BE20170726213351.png)

## 1.6 快速实践Nginx负载均衡环境准备

> 本节先带同学们一起操作实战，让同学们对Nginx负载均衡有一个初步的概念，然后再继续深入讲解Nginx负载均衡的核心知识应用。

![QQ截图20170726214327.png-38.8kB](http://static.zybuluo.com/chensiqi/p95hv5fnmeh7gmr5ut8ugw5d/QQ%E6%88%AA%E5%9B%BE20170726214327.png)

**上图是快速实践Nginx负载均衡的逻辑架构图**

在上图中，所有用户的请求统一发送到Nginx负载均衡器，然后由负载均衡器根据调度算法来请求Web01和Web02

### 1.6.1 软硬件准备

（1）硬件准备

准备4台VM虚拟机（有物理服务器更佳），两台做负载均衡，两台做RS，如下表：

| HOSTNAME | IP            | 说明              |
| -------- | ------------- | ----------------- |
| lb01     | 192.168.0.221 | Nginx主负载均衡器 |
| lb02     | 192.168.0.222 | Nginx副负载均衡器 |
| web01    | 192.168.0.223 | Web01服务器       |
| web02    | 192.168.0.224 | Web02服务器       |

（2）软件准备

系统：CentOS6.5 x86_64
 软件：nginx-1.10.2.tar.gz

### 1.6.2 安装Nginx软件

> 下面将在以上4台服务器上安装Nginx,这里只给出安装的命令部分。

（1）安装依赖软件包命令集合。

```
[root@localhost ~]# yum -y install openssl openssl-devel pcre pcre-devel
[root@localhost ~]# rpm -qa openssl openssl-devel pcre pcre-devel
```

（2）安装Nginx软件包命令集合

```
[root@localhost ~]# tar xf nginx-1.10.2.tar.gz -C /usr/src/
[root@localhost ~]# cd /usr/src/nginx-1.10.2/
[root@localhost nginx-1.10.2]# useradd -M -s /sbin/nologin nginx
[root@localhost nginx-1.10.2]# ./configure --user=nginx --group=nginx --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module && make && make install
[root@localhost nginx]# ln -s /usr/local/nginx/sbin/* /usr/local/sbin/
```

### 1.6.3 配置用于测试的Web服务

> 本小节将在两台NginxWeb服务器的节点上操作：配置并查看Web服务器的配置结果。

```
[root@localhost nginx]# cat conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    log_format main '$remote_addr-$remote_user[$time_local]"$request"'
    '$status $body_bytes_sent "$http_referer"'
    '"$http_user_agent""$http_x_forwarded_for"';
    server {
        listen       80;
        server_name  bbs.yunjisuan.com;
        location / {
            root   html/bbs;
            index  index.html index.htm;
        }
    access_log logs/access_bbs.log main;
    }
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
            root   html/www;
            index  index.html index.htm;
        }
    access_log logs/access_www.log main;
    }
}


#提示：
这里故意将www虚拟主机放在下面，便于用后面的参数配置测试效果
```

**配置完成后检查语法，并启动Nginx服务**

```
[root@localhost nginx]# /usr/local/nginx/sbin/nginx 
[root@localhost nginx]# netstat -antup | grep nginx
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      4100/nginx    
```

**然后填充测试文件数据，如下：**

```
[root@localhost nginx]# mkdir /usr/local/nginx/html/{www,bbs}
[root@localhost nginx]# echo "`hostname -I `www" >> /usr/local/nginx/html/www/index.html
[root@localhost nginx]# cat /usr/local/nginx/html/www/index.html
192.168.0.223 www
[root@localhost nginx]# echo "`hostname -I `bbs" >> /usr/local/nginx/html/bbs/index.html
[root@localhost nginx]# cat /usr/local/nginx/html/bbs/index.html
192.168.0.223 bbs

#提示：
以上操作命令，在Web01上和Web02上是一样的
```

**配置解析Web01的IP和主机名后，用curl测试一下**

```
[root@localhost nginx]# tail -2 /etc/hosts
192.168.0.223 www.yunjisuan.com
192.168.0.223 bbs.yunjisuan.com
[root@localhost nginx]# curl www.yunjisuan.com
192.168.0.223 www
[root@localhost nginx]# curl bbs.yunjisuan.com
192.168.0.223 bbs
```

**配置解析Web02的IP和主机名后，用curl测试一下**

```
[root@localhost nginx]# vim /etc/hosts
[root@localhost nginx]# tail -2 /etc/hosts
192.168.0.224 www.yunjisuan.com
192.168.0.224 bbs.yunjisuan.com
[root@localhost nginx]# curl www.yunjisuan.com
192.168.0.224 www
[root@localhost nginx]# curl bbs.yunjisuan.com
192.168.0.224 bbs
```

**提示：**
 （1）不同Web测试节点，返回的结果是不同的，这是为了方便测试演示！
 （2）通过上面配置就实现了两台Web服务器基于域名的虚拟主机配置。

### 1.6.4 实现一个简单的负载均衡

> 本小节将在nginx lb01服务器节点操作（lb02和lb01相同，后文配置负载均衡器高可用时会用到lb02），其准备信息下表。

| HOSTNAME | IP            | 说明              |
| -------- | ------------- | ----------------- |
| lb01     | 192.168.0.221 | Nginx主负载均衡器 |

> 下面进行一个简单的Nginx负载均衡配置，代理www.yunjisuan.com服务，节点为Web01和Web02.nginx.conf配置文件内容如下：

```
[root@lb01 nginx]# cat conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream www_server_pools {         #这里定义Web服务器池，包含了223，224两个Web节点

    server 192.168.0.223:80 weight=1;

    server 192.168.0.224:80 weight=1;

    }
    server {            #这里定义代理的负载均衡域名虚拟主机
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
        proxy_pass http://www_server_pools;     #访问www.yunjisuan.com,请求发送给www_server_pools里面的节点
        }
    }
}
```

**现在检查语法并启动。命令如下：**

```
[root@lb01 nginx]# nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@lb01 nginx]# /usr/local/nginx/sbin/nginx
[root@lb01 nginx]# netstat -antup | grep nginx
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      4105/nginx 
```

**然后，检查负载均衡测试结果。Linux作为客户端的测试结果如下：**

```
[root@lb01 nginx]# hostname -I
192.168.0.221 
[root@lb01 nginx]# tail -1 /etc/hosts
192.168.0.221 www.yunjisuan.com         #这里是lb1负载均衡器IP
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.224 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.224 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.224 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.224 bbs
```

> 从上面的测试结果可以看出来。两个Web节点按照1：1的比例被访问。
>  下面宕掉任意一个Web节点，看看测试结果如何，测试如下：

```
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 bbs
```

> 可以看到，网站业务不受影响，访问请求都定位到了正常的节点上。
>  现在，宕掉所有Web节点，此时，访问测试结果如下：

```
root@lb01 nginx]# curl www.yunjisuan.com
<html>
<head><title>502 Bad Gateway</title></head>
<body bgcolor="white">
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.10.2</center>
</body>
</html>
[root@lb01 nginx]# curl www.yunjisuan.com
<html>
<head><title>502 Bad Gateway</title></head>
<body bgcolor="white">
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.10.2</center>
</body>
</html>
[root@lb01 nginx]# curl www.yunjisuan.com
<html>
<head><title>502 Bad Gateway</title></head>
<body bgcolor="white">
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.10.2</center>
</body>
</html>
```

> 可以看到，Nginx代理下面没有节点了，因此，Nginx向用户报告了502错误。如果同时开启所有的Web服务又会怎样？测试结果如下：

```
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.224 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.224 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.224 bbs
```

> 结果是Nginx又把请求一比一分配到了Nginx后面的节点上。
>  同学们感觉如何？Nginx的负载均衡很简单吧？嗯，就是这么容易。

## 1.7 Nginx负载均衡核心组件介绍

### 1.7.1 Nginx upstream模块

#### 1.7.1.1 upstream模块介绍

> - Nginx的负载均衡功能依赖于ngx_http_upsteam_module模块，所支持的代理方式包括proxy_pass,fastcgi_pass,memcached_pass等,新版Nginx软件支持的方式有所增加。本文主要讲解proxy_pass代理方式。
> - ngx_http_upstream_module模块允许Nginx定义一组或多组节点服务器组，使用时可以通过proxy_pass代理方式把网站的请求发送到事先定义好的对应Upstream组的名字上，具体写法为“proxy_pass http://  www_server_pools”,其中www_server_pools就是一个Upstream节点服务器组名字。ngx_http_upstream_module模块官方地址为：http://nginx.org/en/docs/http/ngx_http_upstream_module.html

#### 1.7.1.2 upstream模块语法

upstream模块的语法相当简单，这里直接上范例给同学们讲。

**范例1：**基本的upstream配置案例

```
upstream www_server_pools {

#       upstream是关键字必须有，后面的www_server_pools为一个Upstream集群组的名字，可以自己起名，调用时就用这个名字

server 192.168.0.223:80 weight=5;
server 192.168.0.224:80 weight=10;
server 192.168.0.225:80 weight=15;


#       server关键字是固定的，后面可以接域名（门户会用）或IP。如果不指定端口，默认是80端口。weight代表权重，数值越大被分配的请求越多，结尾有分号，别忘了。

}
```

**范例2：**较完整的upstream配置案例

```
upstream blog_server_pool {

server 192.168.0.223;   #这行标签和下行是等价的
server 192.168.0.224:80 weight=1 max_fails=1 fail_timeout=10s;       #这行标签和上一行是等价的，此行多余的部分就是默认配置，不写也可以。

server 192.168.0.225:80 weight=1 max_fails=2 fail_timeout=20s backup;

#   server最后面可以加很多参数，具体参数作用看下文的表格

}
```

**范例3：**使用域名及socket的upstream配置案例

```
upstream backend {

server backend1.example.com weight=5;
server backend2.example.com:8080;   #域名加端口。转发到后端的指定端口上
server unix:/tmp/backend3;  #指定socket文件

#提示：server后面如果接域名，需要内网有DNS服务器或者在负载均衡器的hosts文件做域名解析。

server 192.168.0.223;
server 192.168.0.224:8080;
server backup1.example.com:8080 backup;

#备份服务器，等上面指定的服务器都不可访问的时候会启动，backup的用法和Haproxy中用法一样

server backup2.example.com:8080 backup;

}
```

> 如果是两台Web服务器做高可用，常规方案就需要keepalived配合，那么这里使用Nginx的backup参数通过负载均衡功能就可以实现Web服务器集群了，对于企业应用来说，能做集群就不做高可用。

#### 1.7.1.3 upstream模块相关说明

**upstream模块的内容应放于nginx.conf配置的http{}标签内，其默认调度节点算法是wrr(weighted round-robin，即权重轮询)**。下图为upstream模块内部server标签部分参数说明

![QQ截图20170727194120.png-320.6kB](http://static.zybuluo.com/chensiqi/jmvgewo8hgt8etbqlcygktol/QQ%E6%88%AA%E5%9B%BE20170727194120.png)

**提示：**
 以上参数与专业的Haproxy参数很类似，但不如Haproxy的参数易懂。

来看个示例，如下：

```
upstream backend {

server backend1.example.com weight=5; #如果就是单个Server，没必要设置权重
server 127.0.0.1:8080 max_fail=5 fail_timeout=10s;
#当检测次数等于5的时候，5次连续检测失败后，间隔10s再重新检测。
server unix:/tmp/backend3;
server backup1.example.com:8080 backup; #热备机器设置

}
```

> 需要特别说明的是，如果是Nginx代理Cache服务，可能需要使用hash算法，此时若宕机，可通过设置down参数确保客户端用户按照当前的hash算法访问，这一点很重要。示例配置如下：

```
upstream backend {

ip_hash;
server backend1.example.com;
server backend2.example.com;
server backend3.example.com down;
server backend4.example.com;

}
```

**下面是Haproxy负载均衡器server标签的配置示例。**

```
#开启对后端服务器的健康检测，通过GET /test/index.php来判断后端服务器的健康情况
server php_server_1 192.168.0.223:80 cookie 1 check inter 2000 rise 3 fall 3 weight 2
server php_server_2 192.168.0.224:80 cookie 2 check inter 2000 rise 3 fall 3 weight 1
server php_server_bak 192.168.0.225:80 cookie 3 check inter 1500 rise 3 fall 3 backup
```

> **上述命令的说明如下：**
>
> - weight:调节服务器的请求分配权重。
> - check：开启对该服务器健康检查。
> - inter：设置连续两次的健康检查间隔时间，单位毫秒，默认值2000
> - rise：指定多少次连续成功的健康检查后，即可认定该服务器处于可用状态。
> - fall:指定多少次不成功的健康检查后，即认为服务器为宕机状态，默认值3.
> - maxconn：指定可被发送到该服务器的最大并发连接数。

#### 1.7.1.4 upstream模块调度算法

> 调度算法一般分为两类：
>  第一类为静态调度算法，即负载均衡器根据自身设定的规则进行分配，不需要考虑后端节点服务器的情况，例如：rr,wrr,ip_hash等都属于静态调度算法。
>  第二类为动态调度算法，即负载均衡器会根据后端节点的当前状态来决定是否分发请求，例如：连接数少的优先获得请求，响应时间短的优先获得请求。例如：least_conn,fair等都属于动态调度算法。

**下面介绍一下常见的调度算法。**

（1） rr轮询（默认调度算法，静态调度算法）

> 按客户端请求顺序把客户端的请求逐一分配到不同的后端节点服务器，这相当于LVS中的rr算法，如果后端节点服务器宕机（默认情况下Nginx只检测80端口），宕机的服务器会被自动从节点服务器池中剔除，以使客户端的用户访问不受影响。新的请求会分配给正常的服务器。

（2）wrr（权重轮询，静态调度算法）

> 在rr轮询算法的基础上加上权重，即为权重轮询算法，当使用该算法时，权重和用户访问成正比，权重值越大，被转发的请求也就越多。可以根据服务器的配置和性能指定权重值大小，有效解决新旧服务器性能不均带来的请求分配问题。

（3）ip_hash（静态调度算法）（会话保持）

> 每个请求按客户端IP的hash结果分配，当新的请求到达时，先将其客户端IP通过哈希算法哈希出一个值，在随后的客户端请求中，客户IP的哈希值只要相同，就会被分配至同一台服务器，该调度算法可以解决动态网页的session共享问题，但有时会导致请求分配不均，即无法保证1：1的负载均衡，因为在国内大多数公司都是NAT上网模式，多个客户端会对应一个外部IP，所以，这些客户端都会被分配到同一节点服务器，从而导致请求分配不均。LVS负载均衡的-p参数，Keepalived配置里的persistence_timeout 50参数都类似这个Nginx里的ip_hash参数，其功能都可以解决动态网页的session共享问题。

**我们来看一个示例，如下：**

```
upstream yunjisuan_lb{

ip_hash;
server 192.168.0.223:80;
server 192.168.0.224:8080;

}

upstream backend{

ip_hash;
server backend1.example.com;
server backend2.example.com;
server backend3.example.com down;
server backend4.example.com;

}
```

**注意：**
 当负载调度算法为ip_hash时，后端服务器在负载均衡调度中的状态不能有weight和backup，即使有也不会生效。

（4）fair（动态调度算法）

> 此算法会根据后端节点服务器的响应时间来分配请求，响应时间短的优先分配。这是更加智能的调度算法。此种算法可以根据页面大小和加载时间长短智能地进行负载均衡，也就是根据后端服务器的响应时间来分配请求，响应时间短的优先分配。Nginx本身不支持fair调度算法，如果需要使用这种调度算法，必须下载Nginx相关模块upstream_fair。

**示例如下：**

```
upstream yunjisuan_lb{

server 192.168.0.223;
server 192.168.0.224;
fair;

}
```

（5）least_conn

> least_conn算法会根据后端节点的连接数来决定分配情况，哪个机器连接数少就分发。
>  除了上面介绍的这些算法外，还有一些第三方调度算法，例如：url_hash,一致性hash算法等，介绍如下。

（6）url_hash算法(web缓存节点)

> - 与ip_hash类似，这里是根据访问URL的hash结果来分配请求的，让每个URL定向到同一个后端服务器，后端服务器为缓存服务器时效果显著。在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method使用的是hash算法。
>    url_hash按访问URL的hash结果来分配请求，使每个URL定向到同一个后端服务器，可以进一步提高后端缓存服务器的效率命令率。Nginx本身是不支持url_hash的，如果需要使用这种调度算法，必须安装Nginx的hash模块软件包。

**url_hash(web缓存节点)和ip_hash（会话保持）类似。示例配置如下：**

```
upstream yunjisuan_lb {

server squid1:3128;
server squid2:3128;
hash $request_uri;
hash_method crc32;

}
```

（7）一致性hash算法

> 一致性hash算法一般用于代理后端业务为缓存服务（如Squid，Memcached）的场景，通过将用户请求的URI或者指定字符串进行计算，然后调度到后端的服务器上，此后任何用户查找同一个URI或者指定字符串都会被调度到这一台服务器上，因此后端的每个节点缓存的内容都是不同的，一致性hash算法可以解决后端某个或几个节点宕机后，缓存的数据动荡最小，一致性hash算法知识比较复杂，详细内容可以参考百度上的相关资料，这里仅仅给出配置示例：

```
http {
upstream test {

consistent_hash $request_uri;
server 127.0.0.1:9001 id=1001 weight=3;
server 127.0.0.1:9002 id=1002 weight=10;
server 127.0.0.1:9003 id=1003 weight=20;

}

}
```

**虽然Nginx本身不支持一致性hash算法，但Nginx得分支Tengine支持。详细可参考http://tengine.taobao.org/document_cn/http_upstream_consistent_hash_cn.html**

### 1.7.2 http_proxy_module模块

#### 1.7.2.1 proxy_pass指令介绍

> proxy_pass指令属于ngx_http_proxy_module模块，此模块可以将请求转发到另一台服务器，在实际的反向代理工作中，会通过location功能匹配指定的URI，然后把接收到的符合匹配URI的请求通过proxy_pass抛给定义好的upstream节点池。该指令官方地址1见：http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass

**下面proxy_pass的使用案例：**

（1）将匹配URI为name的请求抛给http://127.0.0.1/remote/.

```
location /name/ {

proxy_pass http://127.0.0.1/remote/;

}
```

（2）将匹配URI为some/path的请求抛给http://127.0.0.1

```
location /some/path/ {

proxy_pass http://127.0.0.1;

}
```

（3）将匹配URI为name的请求应用指定的rewrite规则，然后抛给http://127.0.0.1

```
location /name/ {

rewrite /name/( [^/]+ )  /username=$1 break;
proxy_pass http://127.0.0.1;

}
```

#### 1.7.2.2 http proxy模块参数

> Nginx的代理功能是通过http proxy模块来实现的。默认在安装Nginx时已经安装了http proxy模块，因此可直接使用http proxy模块。下面详细解释模块1中每个选项代表的含义，见下表：

![QQ截图20170727222042.png-93.5kB](http://static.zybuluo.com/chensiqi/4oqknt6krw30dnopqfmhx6pd/QQ%E6%88%AA%E5%9B%BE20170727222042.png)

![QQ截图20170727222503.png-172.6kB](http://static.zybuluo.com/chensiqi/i9x8i3wv95xnmsgmli6ls1ep/QQ%E6%88%AA%E5%9B%BE20170727222503.png)

## 1.8 Nginx负载均衡配置实战(上接1.6节)

| 主机名 | IP地址        | 角色说明          |
| ------ | ------------- | ----------------- |
| lb01   | 192.168.0.221 | nginx主负载均衡   |
| lb02   | 192.168.0.222 | nginx从负载均衡   |
| Web01  | 192.168.0.223 | nginx web01服务器 |
| Web02  | 192.168.0.224 | nginx web02服务器 |

### 1.8.1 查看lb01的配置文件如下：

```
[root@lb01 nginx]# cat /usr/local/nginx/conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream www_server_pools {         #默认调度算法wrr，即权重轮询算法
                                        #虽然定义的www服务器池但是这个服务器池也可以作为BBS等业务的服务器池。因为节点服务器的虚拟主机都是根据访问的主机头字段区分的。

    server 192.168.0.223:80 weight=1;

    server 192.168.0.224:80 weight=1;

    }
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
        proxy_pass http://www_server_pools;     #通过proxy_pass功能把用过户的请求交给上面反向代理upstream定义的www_server_pools服务器池处理。
        }
    }
}
```

**现在配置hosts解析到代理服务器lb01上，重新加载服务，访问测试：**

```
[root@lb01 nginx]# tail -2 /etc/hosts
192.168.0.221 www.yunjisuan.com
192.168.0.221 bbs.yunjisuan.com
[root@lb01 nginx]# /usr/local/nginx/sbin/nginx -s reload
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 bbs
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.224 bbs
```

> 从测试结果可以看出，已经实现了反向代理，负载均衡功能，但是有一个特殊问题，出来的结果并不是带有www的字符串，而是bbs的字符串，根据访问结果，我们推测是访问了Web节点下bbs的虚拟主机，明明代理的是www虚拟主机，为什么结果是访问了后端的bbs虚拟主机了呢？问题又该如何解决？请同学们继续往下看。

### 1.8.2 反向代理多虚拟主机节点服务器企业案例

> 上一节代理的结果不对，究其原因是当用户访问域名时确实是携带了www.yunjisuan.com主机头请求Nginx反向代理服务器，但是反向代理向下面节点重新发起请求时，默认并没有在请求头里告诉节点服务器要找哪台虚拟主机，所以，Web节点服务器接收到请求后发现没有主机头信息，因此，就把节点服务器的第一个虚拟主机发给了反向代理了（节点上第一个虚拟主机放置的是故意这样放置的bbs）。解决这个问题的方法，就是当反向代理向后重新发起请求时，要携带主机头信息，以明确告诉节点服务器要找哪个虚拟主机。具体的配置很简单，就是在Nginx代理www服务虚拟主机配置里增加如下一行配置即可：

```
proxy_set_header host $host;
```

> 在代理向后端服务器发送的http请求头中加入host字段信息后，若后端服务器配置有多个虚拟主机，它就可以识别代理的是哪个虚拟主机。这是节点服务器多虚拟主机时的关键配置。整个Nginx代理配置为：

```
[root@lb01 nginx]# cat /usr/local/nginx/conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream www_server_pools {

    server 192.168.0.223:80 weight=1;

    server 192.168.0.224:80 weight=1;

    }
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
        proxy_pass http://www_server_pools;
        proxy_set_header host $host;    #在代理向后端服务器发送的http请求头中加入host字段信息，用于当后端服务器配置有多个虚拟主机时，可以识别代理的是哪个虚拟主机。这是节点服务器多虚拟主机时的关键配置。
        }
    }
}
```

**此时，再重新加载Nginx服务，并用curl测试检查，结果如下：**

```
[root@lb01 nginx]# /usr/local/nginx/sbin/nginx -s reload
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.223 www
[root@lb01 nginx]# curl www.yunjisuan.com
192.168.0.224 www
```

**可以看到这次访问的结果和访问的域名就完全对应上了，这样代理多虚拟主机的节点服务器就不会出问题了**

### 1.8.3 经过反向代理后的节点服务器记录用户IP企业案例

> 完成了反向代理WWW服务后，自然很开心，但是，不久后你用其他客户端作为客户端测试时，就会发现一个问题，节点服务器对应的WWW虚拟主机的访问日志的第一个字段记录的并不是客户端的IP，而是反向代理服务器的IP，最后一个字段也是“-”！

**例如：**使用任意windows客户端计算机，访问已经解析好代理IP的www.yunjisuan.com后，去节点服务器www服务日志查看，就会发现如下日志：

```
[root@web01 ~]# tail -2 /usr/local/nginx/logs/access_www.log
192.168.0.221--[27/Jul/2017:04:05:22 -0400]"GET / HTTP/1.0"200 18 "-""curl/7.19.7 (x86_64-redhat-linux-gnu) libcurl/7.19.7 NSS/3.14.0.0 zlib/1.2.3 libidn/1.18 libssh2/1.4.2""-"
192.168.0.221--[27/Jul/2017:04:33:06 -0400]"GET / HTTP/1.0"200 18 "-""curl/7.19.7 (x86_64-redhat-linux-gnu) libcurl/7.19.7 NSS/3.14.0.0 zlib/1.2.3 libidn/1.18 libssh2/1.4.2""-"
```

> Web01节点服务器对应的WWW虚拟主机的访问日志的第一个字段记录的并不是客户端的IP而是反向代理服务器本身的IP（192.168.0.221），最后一个字段也是一个“-”，那么如何解决这个问题？其实很简单，同样是增加如下一行参数：

```
proxy_set_header X-Forwarded-For $remote_addr;
#这是反向代理时，节点服务器获取用户真实IP的必要功能配置
```

> 在反向代理请求后端节点服务器的请求头中增加获取的客户端IP的字段信息，然后节点后端可以通过程序或者相关的配置接收X-Forwarded-For传过来的用户真实IP的信息。

**解决上述问题的整个Nginx代理配置为：**

```
[root@lb01 nginx]# cat /usr/local/nginx/conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream www_server_pools {

    server 192.168.0.223:80 weight=1;

    server 192.168.0.224:80 weight=1;

    }
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
        proxy_pass http://www_server_pools;
        proxy_set_header host $host;
        proxy_set_header X-Forwarded-For $remote_addr;
        #在代理向后端服务器发送的http请求头中加入X-Forwarded-For字段信息，用于后端服务器程序，日志等接收记录真实用户的IP，而不是代理服务器的IP
        }
    }
}
```

**重新加载Nginx反向代理服务：**

```
[root@lb01 nginx]# /usr/local/nginx/sbin/nginx -s reload
```

> 特别注意，虽然反向代理已经配好了，但是节点服务器需要的访问日志如果要记录用户的真实IP，还必须进行日志格式配置，这样才能把代理传过来的X-Forwarded-For头信息记录下来，具体配置为：

```
[root@web01 ~]# cat /usr/local/nginx/conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    log_format main '$remote_addr-$remote_user[$time_local]"$request"'
    '$status $body_bytes_sent "$http_referer"'
    '"$http_user_agent""$http_x_forwarded_for"';
    #就是这里的“$http_x_forwarded_for”参数，如果希望在第一行显示，可以替换掉第一行的$remote_addr变量。
    server {
        listen       80;
        server_name  bbs.yunjisuan.com;
        location / {
            root   html/bbs;
            index  index.html index.htm;
        }
    access_log logs/access_bbs.log main;
    }
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
            root   html/www;
            index  index.html index.htm;
        }
    access_log logs/access_www.log main;
    }
}

#注意：这里是客户端Web01的配置
```

> 完成Web01，Web02节点服务器的日志配置后，就可以检查了，注意，不要用curl从反向代理上检查，最好换一个客户端检查，这样才能看到效果。这里使用Windows客户端计算机（IP为192.168.0.110）访问已经解析好代理IP的www.yunjisuan.com，如下图所示：

![QQ截图20170728011520.png-42.1kB](http://static.zybuluo.com/chensiqi/gedwkedpajlvdewhx21gexg0/QQ%E6%88%AA%E5%9B%BE20170728011520.png)

**此时，再去节点服务器WWW服务的访问日志里查看，会发现日志的结尾已经变化了：**

```
[root@web01 ~]# tail -2 /usr/local/nginx/logs/access_www.log 
192.168.0.221--[27/Jul/2017:05:11:22 -0400]"GET / HTTP/1.0"200 18 "-""Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 10.0; WOW64; Trident/7.0; .NET4.0C; .NET4.0E; .NET CLR 2.0.50727; .NET CLR 3.0.30729; .NET CLR 3.5.30729)""192.168.0.110"
192.168.0.221--[27/Jul/2017:05:11:23 -0400]"GET / HTTP/1.0"200 18 "-""Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 10.0; WOW64; Trident/7.0; .NET4.0C; .NET4.0E; .NET CLR 2.0.50727; .NET CLR 3.0.30729; .NET CLR 3.5.30729)""192.168.0.110"
```

> 其中，日志里的192.168.0.221为反向代理的IP，对应Nginx日志格式里的$remote_addr变量，而日志结尾的192.168.0.110对应的时日志格式里的“$http_x_forwarded_for”变量，即接收了前面反向代理配置中“proxy_set_header X-Forwarded-For $remote_addr;”参数X-Forwarded-For的IP了。
>  关于X-Forwarded-For的详细说明，可见http://en.wikipedia.org/wiki/X-Forwwawrded-For。下图是反向代理相关重要基础参数的总结，供同学们参考。

![QQ截图20170728012312.png-148.3kB](http://static.zybuluo.com/chensiqi/oltxsuu7alrskbp8t0e7n0ky/QQ%E6%88%AA%E5%9B%BE20170728012312.png)

### 1.8.4 与反向代理配置相关的更多参数说明

> 除了具有多虚拟主机代理以及节点服务器记录真实用户IP的功能外，Nginx软件还提供了相当多的作为反向代理和后端节点服务器对话的相关控制参数，具体见前面在讲解proxy模块时提供的图表。

**相信同学们对这些参数有了一定了解了，由于参数众多，最好把这些参数放到一个配置文件里，然后用include方式包含到虚拟主机配置里，效果如下：**

```
[root@lb01 conf]# cat /usr/local/nginx/conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream www_server_pools {

    server 192.168.0.223:80 weight=1;

    server 192.168.0.224:80 weight=1;

    }
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
        proxy_pass http://www_server_pools;
        include proxy.conf;         #这就是包含的配置，具体配置内容见下文
        }
    }
}

[root@lb01 conf]# cat proxy.conf 
proxy_set_header host $host;
proxy_set_header x-forwarded-for $remote_addr;
proxy_connect_timeout 60;
proxy_send_timeout 60;
proxy_read_timeout 60;
proxy_buffer_size 4k;
proxy_buffers 4 32k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;
```

**更多Nginx反向代理参数说明**：
 http：//nginx.org/en/docs/http/ngx_http_proxy_module.html

### 1.8.3 根据URL中的目录地址实现代理转发

#### 1.8.3.1 根据URL中的目录地址实现代理转发的说明

> 为了让同学们能学以致用。还是通过真实的企业案例模拟来给大家讲解这部分的知识

**案例背景：**
 通过Nginx实现动静分离，即通过Nginx反向代理配置规则实现让动态资源和静态资源及其他业务分别由不同的服务器解析，以解决网站性能，安全，用户体验等重要问题。

下图为企业常见的动静分离集群架构图，此架构图适合网站前端只使用同一个域名提供服务的场景，例如，用户访问的域名是www.yunjisuan.com，然后，当用户请求www.yunjisuan.com/upload/xx地址时候，代理会分配请求到上传服务器池处理数据；当用户请求www.yunjisuan.com/static/xx地址的时候，代理会分配请求到静态服务器池请求数据；当用户请求www.yunjisuan.com/xx地址的时候，即不包含上述指定的目录地址路径时，代理会分配请求到默认的动态服务器池请求数据（注意：上面的xx表示任意路径）。

![QQ截图20170728093705.png-123kB](http://static.zybuluo.com/chensiqi/ni2yigygmhp0rp2u36aaqta9/QQ%E6%88%AA%E5%9B%BE20170728093705.png)

#### 1.8.3.2 准备：案例配置实战

**先进行企业案例需求梳理：**

- 当用户请求www.yunjisuan.com/upload/xx地址时，实现由upload上传服务器池处理请求。
- 当用户请求www.yunjisuan.com/static/xx地址时，实现由静态服务器池处理请求。
- 除此以外，对于其他访问请求，全都由默认的动态服务器池处理请求。

了解了需求后，就可以进行upstream模块服务器池的配置了。

```
#static_pools为静态服务器池，有一个服务器，地址为192.168.0.223，端口为80.

upstream static_pools {

server 192.168.0.223:80 weght=1;

}

#upload_pools为上传服务器池，有一个服务器地址为192.168.0.224，端口为80.

upstream upload_pools {

server 192.168.0.224:80 weight=1;

}

#default_pools为默认的服务器池，即动态服务器池，有一个服务器，地址为192.168.0.225，端口为80.

upstream default_pools {

server 192.168.0.225:80 weight=1;

}

#提示：需要增加一台测试Web节点Web03（ip:192.168.0.225）,配置与Web01，Web02一样。
```

> 下面利用location或if语句把不同的URI（路径）请求，分给不同的服务器池处理，具体配置如下。

**方案1**：以location方案实现

```
#将符合static的请求交给静态服务器池static_pools，配置如下：
location /static/ {

proxy_pass http://static_pools;
include proxy.conf;

}

#将符合upload的请求交给上传服务器池upload_pools,配置如下：
location /upload/ {

proxy_pass http://upload_pools;
include proxy.conf;

}

#不符合上述规则的请求，默认全部交给动态服务器池default_pools,配置如下：
location / {

proxy_pass http://default_pools;
include proxy.conf;

}
```

**方案2：**以if语句实现。

```
if ($request_uri ~* "^/static/(.*)$")
{
proxy_pass http://static_pools/$1;
}

if ($request_uri ~* "^/upload/(.*)$")
{
proxy_pass http://upload_pools/$1;
}

location / {

proxy_pass http://default_pools;
include proxy.conf;

}
```

**下面以方案1为例进行讲解，Nginx反向代理的实际配置如下：**

```
[root@lb01 ~]# cat /usr/local/nginx/conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream static_pools {

    server 192.168.0.223:80 weight=1;

    }
    upstream upload_pools {

    server 192.168.0.224:80 weight=1;

    }
    upstream default_pools {

    server 192.168.0.225:80 weight=1;

    }
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
        proxy_pass http://default_pools;
        include proxy.conf;
        }
    location /static/ {
        proxy_pass http://static_pools;
        include proxy.conf;
    }
    location /upload/ {
        proxy_pass http://upload_pools;
        include proxy.conf;
    }
    
    }
}
```

**重新加载配置生效，如下：**

```
[root@lb01 ~]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@lb01 ~]# /usr/local/nginx/sbin/nginx -s reload
```

> 暂时不要立刻测试成果，为了实现上述代理的测试，还需要在Web01和Web02上做节点的测试配置，才能更好地展示测试效果。
>  以Web01作为static静态服务，地址端口为：192.168.0.223：80，需要事先配置一个用于测试静态的地址页面，并测试访问，确定它会返回正确结果。操作步骤如下：

```
[root@web01 ~]# cd /usr/local/nginx/html/www/
[root@web01 www]# mkdir static
[root@web01 www]# echo "static_pools" >> static/index.html
[root@web01 www]# curl http://www.yunjisuan.com/static/index.html      #这里的www.yunjisuan.com是解析过的Web01的本地IP
static_pools

#提示：测试的静态地址为http://www.yunjisuan.com/static/index.html,注意，是带static路径的地址。
```

> 以Web02作为upload上传服务，地址端口为：192.168.0.224：80，需要事先配置一个用于测试上传服务的地址页面，并测试访问，确定它会返回正确结果。操作步骤如下：

```
[root@web02 ~]# cd /usr/local/nginx/html/www/
[root@web02 www]# mkdir upload
[root@web02 www]# echo "upload_pools" >> upload/index.html
[root@web02 www]# curl http://www.yunjisuan.com/upload/index.html      #这里的www.yunjisuan.com是解析过的Web02的本地IP
upload_pools

#提示：测试的上传地址为http://www.yunjisuan.com/upload/index.html,注意，是带upload路径的地址。
```

> 在Web03作为动态服务节点，地址端口为192.168.0.225：80，同样需要事先配置一个默认的地址页面，并测试访问，确定它会返回正确结果。操作步骤如下：

```
[root@web03 www]# cd /usr/local/nginx/html/www
[root@web03 www]# echo "default_pools" >> index.html
[root@web03 www]# curl http://www.yunjisuan.com
default_pools
```

> 以上准备了上台Web节点服务器，分别加入到了upstream定义的不同服务器池，代表三组不同的业务集群组，从本机通过hosts解析各自的域名，然后测试访问，其地址与实际访问的内容输出请对照下表：

| 节点  | IP及端口         | 测试地址                                   | 字符串为代表业务 |
| ----- | ---------------- | ------------------------------------------ | ---------------- |
| web01 | 192.168.0.223:80 | http://www.yunjisuan.com/static/index.html | static_pools     |
| web02 | 192.168.0.224:80 | http://www.yunjisuan.com/upload/index.html | upload_pools     |
| web03 | 192.168.0.225:80 | http://www.yunjisuan.com                   | default_pools    |

> 使用客户端计算机访问测试时，最好选用集群以外的机器，这里先在浏览器客户端的hosts文件里把www.yunjisuan.com解析到Nginx反向代理服务器的IP，然后访问上述URL，看代理是不是把请求正确地转发到了指定的服务器上。如果可以得到与上表对应的内容，表示配置的Nginx代理分发的完全正确，因为如果分发请求到错误的机器上就没有对应的URL页面内容，输出会是404错误。

![QQ截图20170728104914.png-65.9kB](http://static.zybuluo.com/chensiqi/b3n5l3kvmjs0md4pkdrhk8tg/QQ%E6%88%AA%E5%9B%BE20170728104914.png)

#### 1.8.3.3 根据URL目录地址转发的应用场景

> - 根据HTTP的URL进行转发的应用情况，被称为第7层（应用层）的负载均衡，而LVS的负载均衡一般用于TCP等的转发，因此被称为第4层（传输层）的负载均衡。
> - 在企业中，有时希望只用一个域名对外提供服务，不希望使用多个域名对应同一个产品业务，此时就需要在代理服务器上通过配置规则，使得匹配不同规则的请求会交给不同的服务器池处理。这类业务有：
> - 业务的域名没有拆封或者不希望拆分，但希望实现动静分离，多业务分离，这在前面已经讲解过案例了。
> - 不同的客户端设备（例如：手机和PC端）使用同一个域名访问同一个业务网站，就需要根据规则将不同设备的用户请求交给后端不同的服务器处理，以便得到最佳用户体验。这也是非常重要的，接下来，我就带同学们看看这类的相关案例。

### 1.8.4 根据客户端的设备（user_agent）转发实践需求

#### 1.8.4.1 根据客户端的设备（user_agent）转发实践需求

> 在企业中，为了让不同的客户端设备用户访问有更好的体验，需要在后端架设不同服务器来满足不同的客户端访问，例如：移动客户端访问网站，就需要部署单独的移动服务器及程序，体验才能更好，而且移动端还分苹果，安卓，Ipad等，在传统的情况下，一般用下面的办法解决这个问题。

（1）常规4层负载均衡解决方案架构

> 在常规4层负载均衡架构下，可以使用不同的域名来实现这个需求，例如，人为分配好让移动端用户访问wap.yunjisuan.com,PC客户端用户访问www.yunjisuan.com,通过不同域名来引导用户到指定的后端服务器，该解决方案的架构图如下：

![QQ截图20170728114132.png-137.4kB](http://static.zybuluo.com/chensiqi/drd1egvv9lt7d2tljetxo6aa/QQ%E6%88%AA%E5%9B%BE20170728114132.png)

> 此解决方案的最大问题就是不同客户端的用户要记住对应的域名！而绝大多数用户只会记住www.yunjisuan.com，不会记住wap.yunjisuan.com，这样一来就会导致用户体验不是很好。有没有办法让所有客户端用户只访问一个统一的www.yunjisuan.com这个地址，还能让不同客户端设备都能有更好的访问体验呢？当然有！那就是下面的第7层负载均衡解决方案。

（2）第7层负载均衡解决方案

> 在第7层负载均衡架构下，就可以不需要人为拆分域名了，对外只需要用一个域名，例如www.yunjisuan.com，通过获取用户请求中的设备信息（利用$http_user_agent获取），根据这些信息转给后端合适的服务器处理，这个方案最大好处就是不需要让用户记忆多个域名了，用户只需要记住主网站地址www.yunjisuan.com，剩下的由网站服务器处理，这样的思路大大地提升了用户访问体验，这是当前企业网站非常常用的解决方案。

**下面我们就来讲解此方案，下图描述了上述解决方案相应的架构逻辑图**

![QQ截图20170728115018.png-127.2kB](http://static.zybuluo.com/chensiqi/a0krm0c8rryhefi78rooye1f/QQ%E6%88%AA%E5%9B%BE20170728115018.png)

#### 1.8.4.2 根据客户端设备（user_agent）转发请求实践

> 这里还是使用static_pools,upload_pools作为本次实验的后端服务器池。下面先根据计算机客户端浏览器的不同设置对应的匹配规则。**(由于没有合适的实验验证环境，这里仅作需求实现的细节讲解)**

```
location / {

if ($http_user_agent ~* "MSIE")
#如果请求的浏览器为微软IE浏览器（MSIE），则让请求由static_pools池处理
{
proxy_pass http://static_pools;
}
if ($http_user_agent ~* "Chrome")
#如果请求的浏览器为谷歌浏览器（Chrome），则让请求由upload_pools池处理
{
proxy_pass http：//upload_pools;
}
proxy_pass http://default_pools;
#其他客户端，由default_pools处理
include proxy.conf;
}
```

**除了针对浏览器外，上述“$http_user_agent”变量也可针对移动端，比如安卓，苹果，Ipad设备进行匹配，去请求指定的服务器，具体细节配置如下：**

```
location / {
if ($http_user_agent ~* "android")
{
proxy_pass http://android_pools;    #这里是android服务器池
}
if ($http_user_agent ~* "iphone")
{
proxy_pass http://iphone_pools;    #这里是iphone服务器池
}
proxy_pass http://pc_pools;     #这里是默认的pc服务器池
include extra/proxy.conf;
}
```

> - 这部分的测试同学们可以回家通过局域网的Wifi功能来实现，用手机等连接到wifi，然后访问服务器的IP测试就可以了。测试时，请用节点的第一个虚拟主机请求测试，这样就不需要本地hosts域名解析了，因为手机端测试做hosts解析也不容易，当然有公网的域名和服务器测试最佳，这部分的配合和测试与浏览器设备实践几乎一样，因此，这里的测试就留给同学们了，看看能不能达到你想的测试效果？
> - 此外，查找移动设备的user_agent对应的具体名称时，还是先用对应的设备通过IP地址访问节点服务器，然后看访问日志，注意IP访问只找第一个虚拟主机的网站。

```
192.168.0.110--[28/Jul/2017:02:12:10 -0400]"GET / HTTP/1.1"200 18 "-""Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 10.0; WOW64; Trident/7.0; .NET4.0C; .NET4.0E; .NET CLR 2.0.50727; .NET CLR 3.0.30729; .NET CLR 3.5.30729)""-" 
#PCwindows访问日志

192.168.0.106--[28/Jul/2017:02:12:22 -0400]"GET / HTTP/1.1"200 18 "-""Mozilla/5.0 (iPhone; CPU iPhone OS 10_3_3 like Mac OS X) AppleWebKit/603.3.8 (KHTML, like Gecko) Version/10.0 Mobile/14G60 Safari/602.1""-"

#苹果iphone6手机设备访问的日志。
```

### 1.8.5 根据文件扩展名实现代理转发

> 除了根据URI路径及user_agent转发外，还可以实现根据文件扩展名进行转发**(这里仅以细节配置作为讲解内容，如需测试请同学们自行实验)**

#### 1.8.5.1 相关server配置

```
#先看看location方法的匹配规则，如下：

location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|css|js)$ {

proxy_pass http://static_pools;
include proxy.conf;

}

#下面是if语句方法的匹配规则：
if ($request_uri ~* ".*\.(php|php5)$")
{

proxy_pass http://php_server_pools;

}

if ($request_uri ~* ".*\.(jsp|jsp*|do|do*)$")
{

proxy_pass http://java_server_pools;

}
```

#### 1.8.5.2 根据扩展名转发的应用场景

> 可根据扩展名实现资源的动静分离访问，如图片，视频等请求静态服务器池，PHP，JSP等请求动态服务器池。

```
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|css|js)$ {

proxy_pass http://static_pools;
include proxy.conf;

}

location ~ .*\.(php|php3|php5)$ {

proxy_pass http://dynamic_pools;
include proxy.conf

}
```

> 在开发无法通过程序实现动静分离的时候，运维可以根据资源实体进行动静分离，而不依赖于开发，具体实现策略是先把后端的服务器分成不同的组。注意，每组服务器的程序都是相同的，因为开发没有把程序拆开，分组后，在前端代理服务器上通过讲解过的路径，扩展名进行规则匹配，从而实现请求的动静分离。

## 1.9 Nginx负载均衡检测节点状态

> 淘宝技术团队开发了一个Tengine（Nginx的分支）模块Nginx_upstream_check_module,用于提供主动式后端服务器健康检查。通过它可以检测后端realserver的健康状态，如果后端realserver不可用，则所有的请求就不会转发到该节点上。
>  Tengine原生支持这个模块，而Nginx则需要通过打补丁的方式将该模块添加到Nginx中。补丁下载地址：https://github.com/yaoweibin/nginx_upstream_check_module。下面介绍如何使用这个模块。

（1）安装nginx_upstream_check_module模块

```
#系统已经安装了nginx-1.10.2软件
[root@lb01 ~]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.10.2

#下载补丁包
[root@lb01 ~]# wget https://codeload.github.com/yaoweibin/nginx_upstream_check_module/zip/master
[root@lb01 ~]# unzip master
[root@lb01 ~]# ls
anaconda-ks.cfg  install.log  install.log.syslog  master  nginx-1.10.2.tar.gz  nginx_upstream_check_module-master
[root@lb01 nginx-1.10.2]# mv ~/nginx_upstream_check_module-master /usr/src/

#因为是对源程序打补丁，所以还需要Nginx源程序
[root@lb01 ~]# cd /usr/src/nginx-1.10.2/
[root@lb01 nginx-1.10.2]# patch -p0 < /usr/src/nginx_upstream_check_module-master/check_1.9.2+.patch
patching file src/http/modules/ngx_http_upstream_hash_module.c
patching file src/http/modules/ngx_http_upstream_ip_hash_module.c
patching file src/http/modules/ngx_http_upstream_least_conn_module.c
patching file src/http/ngx_http_upstream_round_robin.c
patching file src/http/ngx_http_upstream_round_robin.h

#备份源安装程序
[root@lb01 nginx-1.10.2]# cd /usr/local/
[root@lb01 local]# ls
bin  etc  games  include  lib  lib64  libexec  nginx  sbin  share  src
[root@lb01 local]# mv nginx{,.ori}
[root@lb01 local]# ls
bin  etc  games  include  lib  lib64  libexec  nginx.ori  sbin  share  src
[root@lb01 local]# cd /usr/src/nginx-1.10.2/

#重新进行编译，编译的参数要和以前一致，最后加上 --add-module=/usr/src/nginx_upstream_check_module-master/

[root@lb01 nginx-1.10.2]# ./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_stub_status_module --add-module=/usr/src/nginx_upstream_check_module-master/
[root@lb01 local]# /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.10.2
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-4) (GCC) 
built with OpenSSL 1.0.1e-fips 11 Feb 2013
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_stub_status_module --add-module=/usr/src/nginx_upstream_check_module-master/

#拷贝源配置文件到当前Nginx的安装目录下
[root@lb01 local]# pwd
/usr/local
[root@lb01 local]# cp nginx.ori/conf/nginx.conf nginx/conf/
cp: overwrite `nginx/conf/nginx.conf'? y
[root@lb01 local]# cp nginx.ori/conf/proxy.conf nginx/conf/
[root@lb01 local]# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

（2）配置Nginx健康检查，如下：

```
[root@lb01 local]# cat nginx/conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream static_pools {

    server 192.168.0.223:80 weight=1;
    server 192.168.0.224:80 weight=1;
    check interval=3000 rise=2 fall=5 timeout=1000 type=http;   #对static服务器池开启健康监测

    }
    upstream default_pools {

    server 192.168.0.225:80 weight=1;

    }
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
        proxy_pass http://default_pools;
        include proxy.conf;
        }
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|css|js)$ {
        proxy_pass http://static_pools;
        include proxy.conf;
    }
    location /status {
    
    check_status;           #启动健康检查模块
    access_log off;         #关闭此location的访问日志记录

    }
    
    }
}
```

**重启lb1的nginx服务**

```
[root@lb01 local]# /usr/local/nginx/sbin/nginx 
[root@lb01 local]# netstat -antup | grep nginx
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      7008/nginx 

#注意此处必须重启Nginx，不能重新加载
```

> check interval=3000 rise=2 fall=5 timeout=1000 type=http;
>  上面配置的意思时，对static_pools这个负载均衡条目中的所有节点，每隔3秒检测一次，请求2次正常则标记realserver状态为up，如果检测5次都失败，则标记realserver的状态为down,超时时间为1秒，检查的协议是HTTP。
>  详细用法见官网：http://tengine.taobao.org/document_cn/http_upstream_check_cn.html

**访问页面时，显示如下图所示：**

![QQ截图20170728203028.png-44.9kB](http://static.zybuluo.com/chensiqi/od7uqpalvjw0qk6qp28jltah/QQ%E6%88%AA%E5%9B%BE20170728203028.png)

**在lb1配置文件的upstream default_pools{}里也加入健康监测命令，如下：**

```
[root@lb01 local]# cat nginx/conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream static_pools {

    server 192.168.0.223:80 weight=1;
    server 192.168.0.224:80 weight=1;
    check interval=3000 rise=2 fall=5 timeout=1000 type=http;          #对static服务器池开启健康监测

    }
    upstream default_pools {

    server 192.168.0.225:80 weight=1;
    check interval=3000 rise=2 fall=5 timeout=1000 type=http;       #对default服务器池开启健康监测
    
    }
    server {
        listen       80;
        server_name  www.yunjisuan.com;
        location / {
        proxy_pass http://default_pools;
        include proxy.conf;
        }
    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|css|js)$ {
        proxy_pass http://static_pools;
        include proxy.conf;
    }
    location /status {
    
    check_status;           #启动健康检查模块
    access_log off;         #关闭此location的访问日志记录

    }
    
    }
}
```

**再次访问健康监测页面时，显示如下图所示：**

![QQ截图20170728204324.png-48kB](http://static.zybuluo.com/chensiqi/1srrdsdsyowv3b8h9tdyg0vm/QQ%E6%88%AA%E5%9B%BE20170728204324.png)

**关闭任意一个RS节点后（3个Web服务器任选一个关闭nginx服务）**

```
#关闭Web02的nginx服务
[root@web02 ~]# /usr/local/nginx/sbin/nginx -s stop
```

**再次访问健康监测页面时，显示如下图所示：**

![QQ截图20170728204638.png-44.7kB](http://static.zybuluo.com/chensiqi/jk3pdaulejegpyyspwm8fue1/QQ%E6%88%AA%E5%9B%BE20170728204638.png)

## 1.10 proxy_next_upstream 参数补充

> 当Nginx接收后端服务器返回proxy_next_upstream参数定义的状态码时，会将这个请求转发给正常工作的后端服务器，例如500，502，503，504，此参数可以提升用户的访问体验，具体配置如下：

```
server {

listen 80;
server_name www.yunjisuan.com;
location / {

proxy_pass http://static_pools;
proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
include proxy.conf;

}

}
```

## 1.11 本章重点回顾

1. 集群的概念，分类，软硬件知识。
2. upstream负载均衡模块知识
3. http_proxy代理模块知识及相关参数
4. 基于URI路径，user_agent,扩展名规则知识及实践案例。
5. 监测Nginx代理下面节点的健康状态。