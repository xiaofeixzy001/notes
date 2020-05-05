[TOC]

## 一，LVS功能详解

### 1.1 LVS（Linux Virtual Server）介绍

> LVS是Linux Virtual Server 的简写（也叫做IPVS），意即Linux虚拟服务器，是一个虚拟的服务器集群系统，可以在UNIX/LINUX平台下实现负载均衡集群功能。

### 1.2 企业网站LVS集群架构图

![QQ截图20170807112955.png-667.2kB](http://static.zybuluo.com/chensiqi/g6ivlezgbq0nk2rzj4edfkyk/QQ%E6%88%AA%E5%9B%BE20170807112955.png)

### 1.3 IPVS软件工作层次图

![图片1.png-22.6kB](http://static.zybuluo.com/chensiqi/iw4jzrdryw6yygyj9c0p7il6/%E5%9B%BE%E7%89%871.png)

> 从上图我们看出，LVS负载均衡调度技术是在Linux内核中实现的，因此，被称之为Linux虚拟服务器（Linux Virtual  Server）。我们使用该软件配置LVS时候，不能直接配置内核中的ipbs，而需要使用ipvs管理工具ipvsadm进行管理，或者通过Keepalived软件直接管理ipvs。

### 1.4 LVS体系结构与工作原理简单描述

> - LVS集群负载均衡器接受服务的所有入站客户端计算机请求，并根据调度算法决定哪个集群节点应该处理回复请求。负载均衡器（简称LB）有时也被称为LVS Director（简称Director）。
> - LVS虚拟服务器的体系结构如下图所示，一组服务器通过高速的局域网或者地理分布的广域网相互连接，在他们的前端有一个负载调度器（Load  Balancer）。  负载调度器能无缝地将网络请求调度到真实服务器上，从而使得服务器集群的结构对客户是透明的，客户访问集群系统提供的网络服务就像访问一台高性能，高可用的服务器一样。客户程序不受服务器集群的影响不需要作任何修改。系统的伸缩性通过在服务集群中透明地加入和删除一个节点来达到，通过检测节点或服务进程故障和正确地重置系统达到高可用性。由于我们的负载调度技术是在Linux内核中实现的，我们称之为Linux虚拟服务器（Linux Virtual Server）。

### 1.5 LVS 基本工作过程图

**LVS基本工作过程如下图所示：**

![QQ截图20170807214557.png-289.1kB](http://static.zybuluo.com/chensiqi/264coxm9zs1mltzq6a2k7rff/QQ%E6%88%AA%E5%9B%BE20170807214557.png)

**为了方便大家探讨LVS技术，LVS社区提供了一个命名的约定，内容如下表：**

| 名称             | 缩写 | 说明                                                         |
| ---------------- | ---- | ------------------------------------------------------------ |
| 虚拟IP           | VIP  | VIP为Director用于向客户端计算机提供服务的IP地址。比如：www.yunjisuan.com域名就要解析到vip上提供服务 |
| 真实IP地址       | RIP  | 在集群下面节点上使用的IP地址，物理IP地址                     |
| Dirctor的IP地址  | DIP  | Director用于连接内外网络的IP地址，物理网卡上的IP地址。是负载均衡器上的IP |
| 客户端主机IP地址 | CIP  | 客户端用户计算机请求集群服务器的IP地址，该地址用作发送给集群的请求的源IP地址 |

> LVS集群内部的节点称为真实服务器（Real Server）,也叫做集群节点。请求集群服务的计算机称为客户端计算机。
>  与计算机通常在网上交换数据包的方式相同，客户端计算机，Director和真实服务器使用IP地址彼此进行通信。
>  不同架构角色命名情况如下图：

![QQ截图20170807215600.png-310.1kB](http://static.zybuluo.com/chensiqi/esgc9v0j1zqmvxfqqb0yjq5k/QQ%E6%88%AA%E5%9B%BE20170807215600.png)

### 1.6 LVS集群的3种常见工作模式介绍与原理讲解

**IP虚拟服务器软件IPVS**

> - 在调度器的实现技术中，IP负载均衡技术是效率最高的。在已有的IP负载均衡技术中有通过网络地址转换（Network Address  Translation）将一组服务器构成一个高性能的，高可用的虚拟服务器，我们称之为VS/NAT技术（Virtual Server via  Network Address  Translation），大多数商业化的IP负载均衡调度器产品都是使用NAT的方法，如Cisco的额LocalDirector，F5，Netscaler的Big/IP和Alteon的ACEDirector。
> - 在分析VS/NAT 的缺点和网络服务的非对称性的基础上，我们提出通过IP隧道实现虚拟服务器的方法VS/TUN（Virtual  Server via IP Tunneling）和通过直接路由实现虚拟服务器的方法VS/DR（Virtual Server via Direct Routing），他们可以极大地提高系统的伸缩性。所以，IPVS软件实现了这三种IP负载均衡技术。淘宝开源的模式FULLNAT.

**LVS的四种工作模式**

1. NAT（Network Address Translation）
2. TUN（Tunneling）
3. DR（Direct Routing）
4. FULLNAT（Full Network Address Translation）

#### 1.6.1 NAT模式-网络地址转换<==收费站模式（了解即可）

**Virtual Server via Network Address Translation（VS/NAT）**

调度时：目的IP改成RIP（DNAT）
 返回时：源IP改成VIP（SNAT）

![QQ截图20180203195841.png-39.9kB](http://static.zybuluo.com/chensiqi/tqcv6m1v8ohinap9afa5ik6r/QQ%E6%88%AA%E5%9B%BE20180203195841.png)

![QQ截图20180203195908.png-51.6kB](http://static.zybuluo.com/chensiqi/w3981159pej9pra1jxu6g38p/QQ%E6%88%AA%E5%9B%BE20180203195908.png)

**NAT模式特点小结：**

1. NAT技术将请求的报文（DNAT）和响应的报文（SNAT），通过调度器地址重写然后在转发发给内部的服务器，报文返回时在改写成原来的用户请求的地址。
2. 只需要在调度器LB上配置WAN公网IP即可，调度器也要有私有LAN IP和内部RS节点通信。
3. 每台内部RS节点的网关地址，必须要配成调度器LB的私有LAN内物理网卡地址（LDIP），这样才能确保数据报文返回时仍然经过调度器LB。
4. 由于请求与响应的数据报文都经过调度器LB，因此，网站访问量大时调度器LB有较大瓶颈，一般要求最多10-20台节点。
5. NAT模式支持对IP及端口的转换，即用户请求10.0.0.1：80，可以通过调度器转换到RS节点的10.0.0.2：8080（DR和TUN模式不具备的）
6. 所有NAT内部RS节点只需要配置私有LAN IP即可。
7. 由于数据包来回都需要经过调度器，因此，要开启内核转发net.ipv4.ip_forward=1,当然也包括iptables防火墙的forward功能（DR和TUN模式不需要）。

#### 1.6.2 TUN模式-（了解即可）

![QQ截图20180203200015.png-42.1kB](http://static.zybuluo.com/chensiqi/zdevm84h581lhitranvwfsyj/QQ%E6%88%AA%E5%9B%BE20180203200015.png)

**增加一个IP头部。通过IP隧道进行通信（可以跨网段找到RS节点）**

**TUN模式特点小结：**

1. 负载均衡器通过把请求的报文通过IP隧道的方式转发至真实服务器，而真实服务器将响应处理后直接返回给客户端用户。
2. 由于真实服务器将响应处理后的报文直接返回给客户端用户，因此，最好RS有一个外网IP地址，这样效率才会更高。理论上：只要能出网即可，无需外网IP地址。
3. 由于调度器LB只处理入站请求的报文。因此，此集群系统的吞吐量可以提高10倍以上，但隧道模式也会带来一定得系统开销。TUN模式适合LAN/WAN。
4. TUN模式的LAN环境转发不如DR模式效率高，而且还要考虑系统对IP隧道的支持问题。
5. 所有的RS服务器都要绑定VIP，抑制ARP，配置复杂。
6. LAN环境一般多采用DR模式，WAN环境可以用TUN模式，但是当前在WAN环境下，请求转发更多的被haproxy/nginx/DNS调度等代理取代。因此，TUN模式在国内公司实际应用的已经很少。跨机房应用要么拉光纤成局域网，要么DNS调度，底层数据还得同步。
7. 直接对外的访问业务，例如：Web服务做RS节点，最好用公网IP地址。不直接对外的业务，例如：MySQL，存储系统RS节点，最好用内部IP地址。

#### 1.6.3 DR模式-直接路由模式(重点)

**Virtual Server via Direct Routing（VS/DR）**

> VS/DR模式是通过**改写请求报文的目标MAC地址**，将请求发给真实服务器的，而真实服务器将响应后的处理结果直接返回给客户端用户。同VS/TUN技术一样，VS/DR技术可极大地提高集群系统的伸缩性。而且，这种DR模式没有IP隧道的开销，对集群中的真实服务器也没有必须支持IP隧道协议的要求，**但是要求调度器LB与正式服务器RS节点都有一块网卡连在同一物理网段上，即必须在同一个局域网环境。**

![QQ截图20180203200058.png-45.3kB](http://static.zybuluo.com/chensiqi/fzkwpis8szgq0axs8brhr952/QQ%E6%88%AA%E5%9B%BE20180203200058.png)

**只修改目标MAC地址，通过MAC找到RS节点（无法跨网段找到RS节点）**

**DR模式特点小结：**

1. 通过在调度器LB上修改数据包的目的MAC地址实现转发。（源IP地址仍然是CIP，目的IP地址仍然是VIP）
2. 请求的报文经过调度器，而RS响应处理后的报文无需经过调度器LB，因此，并发访问量大时使用效率很高（和NAT模式相比）
3. 因DR模式是通过MAC地址的改写机制实现的转发，因此，所有RS节点和调度器LB只能在一个局域网LAN中（缺点）
4. RS节点的默认网关不需要是调度器LB的DIP，而直接是IDC机房分配的上级路由器的IP（这是RS带有外网IP地址的情况），理论讲：只要RS可以出网即可，不是必须要配置外网IP
5. 由于DR模式的调度器仅进行了目的MAC地址的改写，因此，调度器LB无法改变请求的报文的目的端口（缺点）
6. 当前，调度器LB支持几乎所有的UNIX，LINUX系统，但目前不支持WINDOWS系统。真实服务器RS节点可以是WINDOWS系统。
7. 总的来说DR模式效率很高，但是配置也较麻烦，因此，访问量不是特别大的公司可以用haproxy/nginx取代之。这符合运维的原则：简单，易用，高效。日2000W PV或并发请求1万以下都可以考虑用haproxy/nginx（LVS NAT模式）
8. 直接对外的访问业务，例如：Web服务做RS节点，RS最好用公网IP地址。如果不直接对外的业务，例如：MySQl，存储系统RS节点，最好只用内部IP地址。

#### 1.6.4 FULLNAT模式-（了解即可）

![QQ截图20170809102737.png-158.7kB](http://static.zybuluo.com/chensiqi/76od970p40b562z8yvtebbfn/QQ%E6%88%AA%E5%9B%BE20170809102737.png)

![QQ截图20170809104552.png-185.4kB](http://static.zybuluo.com/chensiqi/lw104cymzrqdjl6gocldekge/QQ%E6%88%AA%E5%9B%BE20170809104552.png)

![QQ截图20170809104638.png-190.8kB](http://static.zybuluo.com/chensiqi/2gyv0bby1f15ocbggani2syq/QQ%E6%88%AA%E5%9B%BE20170809104638.png)

**淘宝的LVS应用模式**

**FULLANT特点：**
 1，源IP改成不同的VIP和目的IP改成RIP
 2，RS处理完毕返回时，返回给不同的LVS调度器
 3，所有LVS调度器之间通过session表进行Client Address的共享

### 1.7 LVS的调度算法

- LVS的调度算法决定了如何在集群节点之间分布工作负荷。
- 当Director调度器收到来自客户端计算机访问它的VIP上的集群服务的入站请求时，Director调度器必须决定哪个集群节点应该处理请求。Director调度器可用于做出该决定的调度方法分成两个基本类别：
   固定调度方法：rr,wrr,dh,sh
   动态调度算法：wlc,lc,lblc,lblcr,SED,NQ

**10种调度算法见如下表格（rr,wrr,wlc重点）：**

| 算法    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| **rr**  | 轮循调度，它将请求依次分配不同的RS节点，也就是在RS节点中均摊请求。这种算法简单，但是只适合于RS节点处理性能相差不大的情况 |
| **wrr** | 权重轮循，它将依据不同RS节点的权值分配任务。权值较高的RS将优先获得任务，并且分配到的连接数将比权值较低的RS节点更多。相同权值的RS得到相同数目的连接数 |
| dh      | 目的地址哈希调度，以目的地址为关键字查找一个静态hash表来获得需要的RS |
| sh      | 源地址哈希调度，以源地址为关键字查找一个静态hash表来获得需要的RS |
| **wlc** | 加权最小连接数调度，实际连接数除以权值，最小的RS作为分配的RS |
| lc      | 最小连接数调度，连接数最小的RS作为分配的RS                   |
| lblc    | 基于地址的最小连接数调度，将来自同一目的地址的请求分配给同一台RS节点 |
| lblcr   | 基于地址带重复最小连接数调度。（略）                         |
| SED     | 最短的期望的延迟（不成熟）                                   |
| NQ      | 最小队列调度（不成熟）                                       |

### 1.8 LVS的调度算法的生产环境选型

- [x] :一般的网络服务，如Http，Mail，MySQL等，常用的LVS调度算法为：
  - 基本轮叫调度rr算法
  - 加权最小连接调度wlc
  - 加权轮叫调度wrr算法
- [x] :基于局部性的最少链接LBLC和带复制的基于局部性最少链接LBLCR主要适用于Web Cache和Db Cache集群，但是我们很少这样用。（都是一致性哈希算法）
- [x] :源地址散列调度SH和目标地址散列调度DH可以结合使用在防火墙集群中，它们可以保证整个系统的唯一出入口。
- [x] :最短预期延时调度SED和不排队调度NQ主要是对处理时间相对比较长的网络服务。

> 实际使用中，这些算法的适用范围不限于这些。我们最好参考内核中的连接调度算法的实现原理，根据具体业务需求合理的选型。

### 1.9 LVS集群的特点

**LVS集群的特点可以归结如下：**

**（1）功能：**

> 实现三种IP负载均衡技术和10种连接调度算法的IPVS软件。在IPVS内部实现上，采用了高效的Hash函数和垃圾回收机制，能正确处理所调度报文相关的ICMP消息（有些商品化的系统反而不能）。虚拟服务的设置数目没有限制，每个虚拟服务都有自己的服务器集。它支持持久的虚拟服务（如HTTP Cookie 和HTTPS等需要该功能的支持），并提供详尽的统计数据，如连接的处理速率和报文的流量等。针对大规模拒绝服务（Deny of  service）攻击，实现了三种防卫策略：有基于内容请求分发的应用层交换软件KTCPVS，它也是在Linux内核中实现。有相关的集群管理软件对资源进行检测，能及时将故障屏蔽，实现系统的高可用性。主，从调度器能周期性地进行状态同步，从而实现更高的可用性。

**（2）适用性**

1)后端真实服务器可运行任何支持TCP/IP的操作系统，包括Linux，各种Unix（如FreeBSD，Sun Solaris，HP Unix等），Mac/OS和windows NT/2000等。

2）负载均衡调度器LB能够支持绝大多数的TCP和UDP协议：

| 协议 | 内容                                                         |
| ---- | ------------------------------------------------------------ |
| TCP  | HTTP，FTP，PROXY，SMTP，POP3，IMAP4，DNS，LDAP，HTTPS，SSMTP等 |
| UDP  | DNS，NTP，TCP，视频，音频流播放协议等                        |

**无需对客户机和服务作任何修改，可适用大多数Internet服务。**

3)调度器本身当前不支持windows系统。支持大多数的Linux和UINIX系统。

**（3）性能**

> LVS服务器集群系统具有良好的伸缩性，可支持几百万个并发连接。配置100M网卡，采用VS/TUN或VS/DR调度技术，集群系统的吞吐量可高达1Gbits/s；如配置千兆网卡，则系统的最大吞吐量可接近10Gbits/s

**（4）可靠性**

> LVS服务器集群软件已经在很多大型的，关键性的站点得到很好的应用，所以它的可靠性在真实应用得到很好的证实。

**（5）软件许可证**

> LVS集群软件是按GPL（GNU Public License）许可证发行的自由软件，这意味着你可以得到软件的源代码，有权对其进行修改，但必须保证你的修改也是以GPL方式发行。

### 1.10 LVS的官方中文阅读资料

| 标题                      | 地址                                           |
| ------------------------- | ---------------------------------------------- |
| LVS项目介绍               | http://www.linuxvirtualserver.org/zh/lvs1.html |
| LVS集群的体系结构         | http://www.linuxvirtualserver.org/zh/lvs2.html |
| LVS集群中的IP负载均衡技术 | http://www.linuxvirtualserver.org/zh/lvs3.html |
| LVS集群的负载调度         | http://www.linuxvirtualserver.org/zh/lvs4.html |

## 二，手动实现LVS的负载均衡功能（DR模式）

### 2.1 安装LVS软件

#### 2.1.1 LVS应用场景说明

1）数据库及memcache等对内业务的负载均衡环境

| 管理IP地址    | 角色                  | 备注                             |
| ------------- | --------------------- | -------------------------------- |
| 192.168.0.210 | LVS调度器（Director） | 对外提供服务的VIP为192.168.0.240 |
| 192.168.0.223 | RS1（真实服务器）     |                                  |
| 192.168.0.224 | RS2（真实服务器）     |                                  |

> **特别提示：上面的环境为内部环境的负载均衡模式，即LVS服务是对内部业务的，如数据库及memcache等的负载均衡**

2）web服务或web cache等负载均衡环境

| 外部IP地址      | 内部IP地址    | 角色                  | 备注                             |
| --------------- | ------------- | --------------------- | -------------------------------- |
| 192.168.200.210 | 192.168.0.210 | LVS调度器（Director） | 对外提供服务的VIP为192.168.0.240 |
| 192.168.200.223 | 192.168.0.223 | RS1（真实服务器）     |                                  |
| 192.168.200.224 | 192.168.0.224 | RS2（真实服务器）     |                                  |

> **提示：**
>  这个表格一般是提供Web或Web cache负载均衡的情况，此种情况特点为双网卡环境。这里把192.168.0.0/24假设为内网卡，192.168.200.0/24假设为外网卡。

### 2.1.2 实验一概述（同学们开始做）

![QQ截图20170810205508.png-22.4kB](http://static.zybuluo.com/chensiqi/8kv1tqz0gjvqb198umik4u96/QQ%E6%88%AA%E5%9B%BE20170810205508.png)

| 内部IP(eth0)  | 外部IP(eth1)    | 角色          | 备注                                    |
| ------------- | --------------- | ------------- | --------------------------------------- |
| 192.168.0.210 | 无              | LVS负载均衡器 | VIP：192.168.0.240网关为：192.168.0.100 |
| 192.168.0.223 | 无              | Web01节点     | 网关为：192.168.0.100                   |
| 192.168.0.224 | 无              | Web02节点     | 网关为：192.168.0.100                   |
| 192.168.0.220 | 无              | 内网客户端    | 网关为：192.168.0.100                   |
| 无            | 192.168.200.200 | 外网客户端    | 不配网关                                |
| 192.168.0.100 | 192.168.200.100 | 网关型防火墙  | 双网卡均无网关                          |

#### 2.1.3 两台Web配置简单的http服务

> 为了方便，我们可以用yum简单装一个apache提供httpd服务进行测试，过程略。

### 2.1.4 开始安装LVS

> 以下的安装都是在LVS LB 192.168.0.210上

1）下载相关软件包

```
wget http://www.linuxvirtualserver.org/software/kernel-2.6/ipvsadm-1.24.tar.gz  # <===适合5.x系统
wget http://www.linuxvirtualserver.org/software/kernel-2.6/ipvsadm-1.26.tar.gz  # <===适合6.x系统
```

2）安装准备命令

```
[root@lvs01 ~]# lsmod | grep ip_vs      #查看linux内核是否有ipvs服务
[root@lvs01 ~]# uname -r            #查看内核版本
2.6.32-431.el6.x86_64
[root@lvs01 ~]# cat /etc/redhat-release     #查看系统版本
CentOS release 6.5 (Final)
[root@lvs01 ~]# yum -y install kernel-devel     #光盘安装
[root@lvs01 ~]# ls -ld /usr/src/kernels/2.6.32-431.el6.x86_64/
drwxr-xr-x. 2 root root 4096 Aug  9 19:28    /usr/src/kernels/2.6.32-431.el6.x86_64/         #安装完就会出现此目录
[root@lvs01 ~]# ln -s /usr/src/kernels/2.6.32-431.el6.x86_64/ /usr/src/linux          #做一个软连接
[root@lvs01 ~]# ll -d /usr/src/linux/
drwxr-xr-x. 2 root root 4096 Aug  9 19:28 /usr/src/linux/
[root@lvs01 ~]# ll /usr/src/
total 8
drwxr-xr-x. 2 root root 4096 Sep 23  2011 debug
drwxr-xr-x. 3 root root 4096 Aug  9 19:28 kernels
lrwxrwxrwx. 1 root root   39 Aug  9 19:28 linux -> /usr/src/kernels/2.6.32-431.el6.x86_64/
[root@lvs01 ~]# 
```

> **特别注意：**
>  此ln命令的链接路径要和uname -r输出结果内核版本对应，工作中如果做安装虚拟化可能有多个内核路径
>  如果没有/usr/src/kernels/2.6.32-431.el6.x86_64/路径，很可能是因为缺少kernel-devel软件包。可通过yum进行安装
>  centos5.x版本不能用ipvs1.26

3）安装lvs命令：

```
[root@lvs01 ~]# yum -y install libnl* popt*     #需要通过公网源安装
[root@lvs01 ~]# cd /usr/src/ipvsadm-1.26/
[root@lvs01 ipvsadm-1.26]# make             #直接编译不需要./configure
[root@lvs01 ipvsadm-1.26]# make install
[root@lvs01 ~]# which ipvsadm
/sbin/ipvsadm
[root@lvs01 ~]# ipvsadm
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
[root@lvs01 ~]# lsmod | grep ip_vs          #执行完/sbin/ipvsadm就会有信息
ip_vs                 125220  0 
libcrc32c               1246  1 ip_vs
ipv6                  317340  270 ip_vs,ip6t_REJECT,nf_conntrack_ipv6,nf_defrag_ipv6

#==>出现这个内容就表示LVS已经安装好，并加载到了内核
```

> **LVS安装小结：**
>  1，CentOS5.X安装lvs，使用1.24版本。
>  2，CentOS6.X安装lvs，使用1.26版本。
>  3，安装lvs后，要执行ipvsadm把ip_vs模块加载到内核。

### 2.2 手动配置LVS负载均衡服务

#### 2.2.1 手工添加lvs转发

（1）配置LVS虚拟IP（VIP）

```
[root@lvs01 ~]# ifconfig eth0:0 192.168.0.240 broadcast 192.168.0.240 netmask 255.255.255.0 up
[root@lvs01 ~]# ifconfig eth0:0
eth0:0    Link encap:Ethernet  HWaddr 00:0C:29:D5:7F:9D  
          inet addr:192.168.0.240  Bcast:192.168.0.240  Mask:255.255.255.255
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
```

（2）手工执行配置添加LVS服务并增加两台RS

```
[root@lvs01 ~]# ipvsadm -C                  #清空ipvs历史设置
[root@lvs01 ~]# ipvsadm --set 30 5 60       #设置超时时间（tcp tcpfin udp）
[root@lvs01 ~]# ipvsadm -A -t 192.168.0.240:80 -s rr -p 20

#说明：
-A：添加一个虚拟路由主机（LB）
-t：指定虚拟路由主机的VIP地址和监听端口
-s：指定负载均衡算法
-p：指定会话保持时间

[root@lvs01 ~]# ipvsadm -a -t 192.168.0.240:80 -r 192.168.0.223:80 -g -w 1
[root@lvs01 ~]# ipvsadm -a -t 192.168.0.240:80 -r 192.168.0.224:80 -g -w 1

#说明：
-a：添加RS节点
-t：指定虚拟路由主机的VIP地址和监听端口
-r：指定RS节点的RIP地址和监听端口
-g：指定DR模式
-w：指定权值
```

（3）查看lvs配置结果

```
[root@lvs01 ~]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.240:80 rr persistent 20
  -> 192.168.0.223:80             Route   1      0          0         
  -> 192.168.0.224:80             Route   1      0          0         
```

（4）ipvs配置删除方法

```
[root@lvs01 ~]# #ipvsadm -D -t 192.168.0.240:80 -s rr       #删除虚拟路由主机
[root@lvs01 ~]# #ipvsadm -d -t 192.168.0.240:80 -r 192.168.0.223:80     #删除RS节点
```

> 此时，可以打开浏览器访问http://192.168.0.240体验结果，如果没意外，是无法访问的。（RS将包丢弃了）

![QQ截图20170810005652.png-434.1kB](http://static.zybuluo.com/chensiqi/4hna944v5ti02lptrocx64ep/QQ%E6%88%AA%E5%9B%BE20170810005652.png)

#### 2.2.2 手工在RS端绑定

```
#在Web01上操作
[root@web01 ~]# ifconfig lo:0 192.168.0.240/32 up       #掩码必须设置32
[root@web01 ~]# ifconfig lo:0
lo:0      Link encap:Local Loopback  
          inet addr:192.168.0.240  Mask:0.0.0.0
          UP LOOPBACK RUNNING  MTU:16436  Metric:1

#在Web02上操作
[root@web02 ~]# ifconfig lo:0 192.168.0.240/32 up       #掩码必须设置32
[root@web02 ~]# ifconfig lo:0
lo:0      Link encap:Local Loopback  
          inet addr:192.168.0.240  Mask:0.0.0.0
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
```

#### 2.2.3 浏览器测试LVS转发效果

![QQ截图20170810010649.png-24.7kB](http://static.zybuluo.com/chensiqi/nnc3l3ssvgryu63gghxtfvj9/QQ%E6%88%AA%E5%9B%BE20170810010649.png)

![QQ截图20170810010717.png-25.3kB](http://static.zybuluo.com/chensiqi/5733r48dwxyd1y0s14duzn9i/QQ%E6%88%AA%E5%9B%BE20170810010717.png)

> **注意：**
>  在测试时候你会发现刷新看的都是同一个RS节点
>  这是因为浏览器的缓存问题
>  等一段时间以后，刷新就会重新负载均衡到新RS节点了

#### 2.2.4 关于DR模式RS节点的ARP抑制的问题

![QQ截图20170810005652.png-434.1kB](http://static.zybuluo.com/chensiqi/4hna944v5ti02lptrocx64ep/QQ%E6%88%AA%E5%9B%BE20170810005652.png)

> - 因为在DR模式下，RS节点和LVS同处一个局域网网段内。
> - 当网关通过ARP广播试图获取VIP的MAC地址的时候
> - LVS和节点都会接收到ARP广播并且LVS和节点都绑定了192.168.0.240这个VIP，所以都会去响应网关的这个广播，导致冲突现象。
> - 因此，我们需要对RS节点做抑制ARP广播的措施。

```
[root@web01 ~]# echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
[root@web01 ~]# echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce 
[root@web01 ~]# echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore 
[root@web01 ~]# echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce 
```

#### 2.2.5 配置网关型防火墙

> 防火墙的双网卡都不要设置网关，因为自己的就网关

```
[root@localhost ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000                               #内网网卡
    link/ether 00:0c:29:ee:d3:15 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.100/24 brd 192.168.0.255 scope global eth0
    inet6 fe80::20c:29ff:feee:d315/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000                               #外网网卡
    link/ether 00:0c:29:ee:d3:1f brd ff:ff:ff:ff:ff:ff
    inet 192.168.200.100/24 brd 192.168.200.255 scope global eth1
    inet6 fe80::20c:29ff:feee:d31f/64 scope link 
       valid_lft forever preferred_lft forever
[root@localhost ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.0.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
192.168.200.0   0.0.0.0         255.255.255.0   U     0      0        0 eth1
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 eth1

#防火墙不需要配置网关，因此没有默认路由信息
[root@localhost ~]# head /etc/sysctl.conf           #开启网卡路由转发
# Kernel sysctl configuration file for Red Hat Linux
#
# For binary values, 0 is disabled, 1 is enabled.  See sysctl(8) and
# sysctl.conf(5) for more details.

# Controls IP packet forwarding
net.ipv4.ip_forward = 1                 #修改为1

# Controls source route verification
net.ipv4.conf.default.rp_filter = 1
[root@localhost ~]# sysctl -p               #让配置即刻生效
net.ipv4.ip_forward = 1
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
```

> **特别提示：**
>  Web01,Web02,LVS负载均衡器，以及内网客户端均将网关设置成网关型防火墙的eth0：192.168.0.100

#### 2.2.6 配置内网客户端

> 内网客户端用于模拟lvs应用于内网的负载均衡情况
>  比如lvs数据库读负载均衡，比如lvs memcached缓存组负载均衡
>  由于这类型的负载均衡请求都是由内网服务器发起，因此用内网客户端来模拟

```
#内网客户端访问测试

root@LanClient ~]# hostname -I
192.168.0.220               #内网客户端IP
[root@LanClient ~]# route -n        #默认路由为网关防火墙
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.0.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
0.0.0.0         192.168.0.100   0.0.0.0         UG    0      0        0 eth0

[root@LanClient ~]# curl 192.168.0.240
192.168.0.224 bbs
[root@LanClient ~]# curl 192.168.0.240
192.168.0.223 bbs
[root@LanClient ~]# curl 192.168.0.240
192.168.0.224 bbs
[root@LanClient ~]# curl 192.168.0.240
192.168.0.223 bbs

#从上面可以看出，内网客户端模拟访问lvs负载均衡器，成功！
```

#### 2.2.7 配置外网客户端

> 外网客户端模拟的是lvs转发外网用户访问需求给RS节点处理的情况
>  模拟外网客户端，要求客户端不能配置任何网关

![QQ截图20170810213311.png-9kB](http://static.zybuluo.com/chensiqi/3lko3cwkro4p8yuevs36kc64/QQ%E6%88%AA%E5%9B%BE20170810213311.png)

> 由于外网客户端要访问内网的LVS需要经过网关防火墙的跳转，因此需要在防火墙服务器上做iptables的DNAT和SNAT，配置如下：

```
[root@GATEWAY ~]# hostname -I
192.168.0.100(内网网卡) 192.168.200.100（外网网卡） 
[root@GATEWAY ~]# route -n      
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
192.168.0.0     0.0.0.0         255.255.255.0   U     0      0        0 eth0
192.168.200.0   0.0.0.0         255.255.255.0   U     0      0        0 eth1
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1003   0        0 eth1
[root@GATEWAY ~]# iptables -t nat -A PREROUTING -i eth1 -d 192.168.200.100 -p tcp --dport 80 -j DNAT --to-destination 192.168.0.240
[root@GATEWAY ~]# iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth1 -j SNAT --to-source 192.168.200.100
[root@GATEWAY ~]# iptables -t nat -L -nv
Chain PREROUTING (policy ACCEPT 25 packets, 5251 bytes)
 pkts bytes target     prot opt in     out     source               destination         
   32  1920 DNAT       tcp  --  eth1   *       0.0.0.0/0            192.168.200.100     tcp dpt:80 to:192.168.0.240 

Chain POSTROUTING (policy ACCEPT 53 packets, 3208 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 SNAT       all  --  *      eth1    192.168.0.0/24       0.0.0.0/0           to:192.168.200.100 

Chain OUTPUT (policy ACCEPT 22 packets, 1348 bytes)
 pkts bytes target     prot opt in     out     source               destination         
[root@GATEWAY ~]# 
```

**进行外网客户端访问LVS负载均衡器测试**

![QQ截图20170810214139.png-37.1kB](http://static.zybuluo.com/chensiqi/0uonb91gk3k42whsmc721r63/QQ%E6%88%AA%E5%9B%BE20170810214139.png)

> **特别提示：**
>  由于浏览器缓存及LVS默认会话保持等影响，个人简单的测试切换RS的几率要很多次并且间隔一定时间访问才行。尽可能关闭浏览器换不同的客户端IP来测试，效果更明显一些。用单机测试是有这种情况（负载均衡的算法倾向于一个客户端IP定向到一个后端服务器，以保持会话连贯性），如果用两三台机器去测试也许就不一样。
>  在测试访问的同时可以通过ipvsadm -Lnc来查看访问结果，如下所示：

```
[root@lvs01 network-scripts]# ipvsadm -lnc
IPVS connection entries
pro expire state       source             virtual            destination
TCP 01:41  FIN_WAIT    192.168.0.220:59805 192.168.0.240:80   192.168.0.224:80
TCP 01:40  FIN_WAIT    192.168.0.220:59803 192.168.0.240:80   192.168.0.224:80
TCP 01:50  FIN_WAIT    192.168.200.200:34926 192.168.0.240:80   192.168.0.223:80
TCP 01:41  FIN_WAIT    192.168.0.220:59804 192.168.0.240:80   192.168.0.223:80
TCP 01:50  FIN_WAIT    192.168.200.200:34925 192.168.0.240:80   192.168.0.224:80
TCP 01:41  FIN_WAIT    192.168.0.220:59806 192.168.0.240:80   192.168.0.223:80
TCP 01:40  FIN_WAIT    192.168.0.220:59802 192.168.0.240:80   192.168.0.223:80
TCP 01:51  FIN_WAIT    192.168.200.200:34927 192.168.0.240:80   192.168.0.224:80
```

### 2.3 arp抑制技术参数说明

- [x] : arp_ignore-INTRGER
- 定义对目标地址为本地IP的ARP询问不同的应答模式
  - 0(默认值)：回应任何网络接口上对任何本地IP地址的arp查询请求。
  - 1：只回答目标IP地址是来访网络接口本地地址的ARP查询请求
  - 2：只回答目标IP地址是来访网络接口本地地址的ARP查询请求，且来访IP必须在该网络接口的子网段内。
  - 3：不回应该网络界面的arp请求，而只对设置的唯一和连接地址做出回应。
  - 4-7：保留未使用
  - 8：不回应所有（本地地址）的arp查询。
- [x] :arp_announce-INTEGER
- 对网络接口上，本地IP地址的发出的，ARP回应，作出相应级别的限制：确定不同程度的限制，宣布对来自本地源IP地址发出Arp请求的接口。
  - 0（默认值）：在任意网络接口（eth0，eth1，lo）上的任何本地地址
  - 1：尽量避免不在该网络接口子网段的本地地址做出arp回应，当发起ARP请求的源IP地址是被设置应该经由路由达到此网络接口的时候很有用。此时会检查来访IP是否为所有接口上的子网段内IP之一。如果该来访IP不属于各个网络接口上的子网段内，那么将采用级别2的方式来进行处理。
  - 2：对查询目标使用最适当的本地地址，在此模式下将忽略这个IP数据包的源地址并尝试选择能与该地址通信的本地地址，首要是选择所有的网络接口的子网中外出访问子网中包含该目标IP地址的本地地址。如果没有合适的地址被发现，将选择当前的发送网络接口或其他的有可能接受到该ARP回应的网络接口来进行发送。限制了使用本地的vip地址作为优先的网络接口。

### 2.4 开发脚本配置LVS负载均衡器端

#### 2.4.1 LVS负载均衡器端自动配置脚本：

```
[root@lvs01 scripts]# cat ipvs_server.sh 
#!/bin/bash
# author:Mr.chen
#LVS scripts

. /etc/init.d/functions

VIP=192.168.0.240
SUBNET="eth0:`echo $VIP | awk -F "." '{print $4}'`"
PORT=80
RIP=(

192.168.0.223
192.168.0.224

)

function start(){
    
    if [ `ifconfig | grep $VIP | wc -l` -ne 0 ];then
        stop
    fi
    ifconfig $SUBNET $VIP broadcast $VIP netmask 255.255.255.0 up
    ipvsadm -C
    ipvsadm --set 30 5 60
    ipvsadm -A -t $VIP:$PORT -s rr -p 20
    for ((i=0;i<${#RIP[*]};i++))
    do
        ipvsadm -a -t $VIP:$PORT -r ${RIP[$i]} -g -w 1
    done

}

function stop(){

    ipvsadm -C
    if [ `ifconfig | grep $VIP | wc -l` -ne 0 ];then
        ifconfig $SUBNET down
    fi
    route del -host $VIP dev eth0 &>/dev/null

}

case "$1" in 
    start)
        start
        echo "ipvs is started"
    ;;
    stop)
        stop
        echo "ipvs is stopped"
    ;;
    restart)
        stop
        echo "ipvs is stopped"
        start
        echo "ipvs is started"
    ;;  
    *)
    echo "USAGE:$0 {start | stop | restart}"
esac
```

#### 2.4.2 RS节点Web服务器端自动配置脚本

```
[root@web01 scripts]# cat rs_server.sh 
#!/bin/bash
# author:Mr.chen
# RS_sever scripts


. /etc/rc.d/init.d/functions

VIP=192.168.0.240

case "$1" in 
    start)
        echo "start LVS of REALServer IP"
        interface="lo:`echo $VIP | awk -F "." '{print $4}'`"
        /sbin/ifconfig $interface $VIP broadcast $VIP netmask 255.255.255.255 up
        route add -host $VIP dev $interface
        echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
        echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
        echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
        echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce
    ;;
    stop)
        interface="lo:`echo $VIP | awk -F "." '{print $4}'`"
        /sbin/ifconfig $interface down
        echo "STOP LVS of REALServer IP"
        echo "0" > /proc/sys/net/ipv4/conf/lo/arp_ignore
        echo "0" > /proc/sys/net/ipv4/conf/lo/arp_announce
        echo "0" > /proc/sys/net/ipv4/conf/all/arp_ignore
        echo "0" > /proc/sys/net/ipv4/conf/all/arp_announce
    ;;
    *)
        echo "Usage: $0 {start|stop}"
        exit 1
esac
```

## 三，企业LVS负载均衡高可用最优方案（LVS+Keepalived）

### 3.1 实验二概述（同学们开始做）

![QQ截图20170811104939.png-22.8kB](http://static.zybuluo.com/chensiqi/0h2o37psvms71ym905snuvf5/QQ%E6%88%AA%E5%9B%BE20170811104939.png)

| 内部IP(eth0)  | 外部IP(eth1) | 角色                | 备注               |
| ------------- | ------------ | ------------------- | ------------------ |
| 192.168.0.210 | 无           | LVS负载均衡器（主） | VIP：192.168.0.240 |
| 192.168.0.211 | 无           | LVS负载均衡器（备） | VIP：192.168.0.240 |
| 192.168.0.223 | 无           | Web01节点           |                    |
| 192.168.0.224 | 无           | Web02节点           |                    |
| 192.168.0.220 | 无           | 内网客户端          |                    |

### 3.2 LVS负载均衡器主和备安装LVS软件

过程略

### 3.3 两台Web服务器安装Web服务

过程略

### 3.4 LVS负载均衡器主和备安装Keepalived软件

```
[root@lvs01 ~]# yum -y install keepalived   #光盘安装即可
```

### 3.5 仅实现LVS负载均衡器主和备的keepalived高可用功能

**LVS负载均衡器主的keepalived配置文件内容如下**

```
[root@lvs01 ~]# sed -n '1,30p' /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
    215379068@qq.com
   }
   notification_email_from yunjisuan
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS01
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 55
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
    192.168.0.240/24 dev eth0 label eth0:240
    }
}
```

**LVS负载均衡器主的keepalived配置文件内容如下**

```
[root@localhost ~]# sed -n '1,30p' /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
    215379068@qq.com
   }
   notification_email_from yunjisuan
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS02
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 55
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
    192.168.0.240/24 dev eth0 label eth0:240
    }
}
```

### 3.6 添加LVS的负载均衡规则

> 以下操作过程，在LVS主和备上完全一样

```
[root@localhost ~]# ipvsadm -C
[root@localhost ~]# ipvsadm -A -t 192.168.0.240:80 -s rr
[root@localhost ~]# ipvsadm -a -t 192.168.0.240:80 -r 192.168.0.223:80 -g -w 1
[root@localhost ~]# ipvsadm -a -t 192.168.0.240:80 -r 192.168.0.224:80 -g -w 1
[root@localhost ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.240:80 rr persistent 20
  -> 192.168.0.223:80             Route   1      0          0         
  -> 192.168.0.224:80             Route   1      0          0  
```

### 3.7 启动LVS主和备的keepalived服务

```
#在LVS主上
[root@lvs01 ~]# /etc/init.d/keepalived start
[root@lvs01 ~]# ifconfig 
eth0      Link encap:Ethernet  HWaddr 00:0C:29:D5:7F:9D  
          inet addr:192.168.0.210  Bcast:192.168.0.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fed5:7f9d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:23567 errors:0 dropped:0 overruns:0 frame:0
          TX packets:14635 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:2008524 (1.9 MiB)  TX bytes:1746298 (1.6 MiB)

eth0:240  Link encap:Ethernet  HWaddr 00:0C:29:D5:7F:9D  
          inet addr:192.168.0.240  Bcast:0.0.0.0  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:769 errors:0 dropped:0 overruns:0 frame:0
          TX packets:769 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:56636 (55.3 KiB)  TX bytes:56636 (55.3 KiB)

#在LVS副上
[root@localhost ~]# /etc/init.d/keepalived start
[root@localhost ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:E7:06:1D  
          inet addr:192.168.0.211  Bcast:192.168.0.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fee7:61d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:14109 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4902 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:12683754 (12.0 MiB)  TX bytes:553207 (540.2 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:155 errors:0 dropped:0 overruns:0 frame:0
          TX packets:155 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:11283 (11.0 KiB)  TX bytes:11283 (11.0 KiB)

#如果LVS副上没有VIP就对了。如果主副都有，那么请检查防火墙是否开启状态
```

### 3.8 内网客户端进行访问测试

```
#在内网客户端上进行访问测试
[root@LanClient ~]# curl 192.168.0.240
192.168.0.224 bbs
[root@LanClient ~]# curl 192.168.0.240
192.168.0.223 bbs
[root@LanClient ~]# curl 192.168.0.240
192.168.0.224 bbs
[root@LanClient ~]# curl 192.168.0.240
192.168.0.223 bbs
[root@LanClient ~]# curl 192.168.0.240
192.168.0.224 bbs
[root@LanClient ~]# curl 192.168.0.240
192.168.0.223 bbs
[root@LanClient ~]# curl 192.168.0.240
192.168.0.224 bbs


#在LVS主上进行访问连接查询
[root@lvs01 ~]# ipvsadm -Lnc
IPVS connection entries
pro expire state       source             virtual            destination
TCP 00:01  FIN_WAIT    192.168.0.220:59887 192.168.0.240:80   192.168.0.223:80
TCP 00:01  FIN_WAIT    192.168.0.220:59889 192.168.0.240:80   192.168.0.223:80
TCP 00:01  FIN_WAIT    192.168.0.220:59888 192.168.0.240:80   192.168.0.224:80
TCP 00:00  FIN_WAIT    192.168.0.220:59886 192.168.0.240:80   192.168.0.224:80

#在LVS主上停掉keepalived服务
[root@lvs01 ~]# /etc/init.d/keepalived stop
Stopping keepalived:                                       [  OK  ]
[root@lvs01 ~]# ifconfig | grep eth0:240

#在LVS副上查看VIP
[root@localhost ~]# ip a | grep eth0:240
    inet 192.168.0.240/24 scope global secondary eth0:240

#再次在内网客户端上进行访问测试
[root@LanClient ~]# curl 192.168.0.240
192.168.0.223 bbs
[root@LanClient ~]# curl 192.168.0.240
192.168.0.224 bbs
[root@LanClient ~]# curl 192.168.0.240
192.168.0.223 bbs
[root@LanClient ~]# curl 192.168.0.240
192.168.0.224 bbs
[root@LanClient ~]# curl 192.168.0.240
192.168.0.223 bbs

#在LVS副上进行访问连接查询
[root@localhost ~]# ipvsadm -Lnc
IPVS connection entries
pro expire state       source             virtual            destination
TCP 01:47  FIN_WAIT    192.168.0.220:59900 192.168.0.240:80   192.168.0.223:80
TCP 01:09  FIN_WAIT    192.168.0.220:59891 192.168.0.240:80   192.168.0.224:80
TCP 01:48  FIN_WAIT    192.168.0.220:59902 192.168.0.240:80   192.168.0.223:80
TCP 01:09  FIN_WAIT    192.168.0.220:59892 192.168.0.240:80   192.168.0.224:80
TCP 01:14  FIN_WAIT    192.168.0.220:59896 192.168.0.240:80   192.168.0.224:80
TCP 01:10  FIN_WAIT    192.168.0.220:59894 192.168.0.240:80   192.168.0.224:80

#开启LVS主上的keepalived服务
[root@lvs01 ~]# /etc/init.d/keepalived start
[root@lvs01 ~]# ip a | grep eth0:240
    inet 192.168.0.240/24 scope global secondary eth0:240

#查看LVS副上VIP资源是否释放
[root@localhost ~]# ip a | grep eth0:240
[root@localhost ~]# 
```

> 综上，至此基于LVS的keepalived高可用功能实验完毕

### 3.9 通过Keepalived对LVS进行管理的功能实现

```
[root@lvs01 ~]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
    215379068@qq.com
   }
   notification_email_from yunjisuan
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS01
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 55
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
    192.168.0.240/24 dev eth0 label eth0:240
    }
}

virtual_server 192.168.0.240 80 {       #虚拟主机VIP
    delay_loop 6            #
    lb_algo rr              #算法
    lb_kind DR              #模式
    nat_mask 255.255.255.0  #掩码
#    persistence_timeout 50 #会话保持
    protocol TCP            #协议

    real_server 192.168.0.223 80 {      #RS节点
        weight 1                #权重
        TCP_CHECK {             #节点健康检查
        connect_timeout 8       #延迟超时时间
        nb_get_retry 3          #重试次数
        delay_before_retry 3    #延迟重试次数
        connect_port 80         #利用80端口检查
    }
    }
    real_server 192.168.0.224 80 {      #RS节点
        weight 1
        TCP_CHECK {
        connect_timeout 8
        nb_get_retry 3
        delay_before_retry 3
        connect_port 80 
    }
    }
}
```

> 以上keepalived配置文件在LVS主和备上都进行修改。
>  然后在lvs服务器上通过ipvsadm -C清除之前设置的规则
>  重新启动keepalived服务进行测试，操作过程如下：

```
[root@lvs01 ~]# /etc/init.d/keepalived stop             #关闭主LVS的keepalived服务
Stopping keepalived:                                       [  OK  ]
[root@lvs01 ~]# ipvsadm -Ln                             #没有ipvs规则
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
[root@lvs01 ~]# ip a | grep 240                         #没有VIP
[root@lvs01 ~]# /etc/init.d/keepalived start            #启动keepalived服务
Starting keepalived:                                       [  OK  ]
[root@lvs01 ~]# ipvsadm -Ln                             #出现ipvs规则
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.240:80 rr
  -> 192.168.0.223:80             Route   1      0          0         
  -> 192.168.0.224:80             Route   1      0          0         
[root@lvs01 ~]# ip a | grep 240                         #出现VIP
    inet 192.168.0.240/24 scope global secondary eth0:240   
```

## 附录：LVS集群分发请求RS不均衡生产环境实战解决

> 生产环境中ipvsadm -L -n 发现两台RS的负载不均衡，一台有很多请求，一台没有。并且没有请求的那台RS经测试服务正常，lo：VIP也有。但是就是没有请求。

```
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.240:80 rr persistent 10
  -> 192.168.0.223:80             Route   1      0          0         
  -> 192.168.0.224:80             Route   1      8          12758         
```

**问题原因：**

> persistent 10的原因，persistent会话保持，当clientA访问网站的时候，LVS把请求分发给了52，那么以后clientA再点击的其他操作其他请求，也会发送给52这台机器。

**解决办法：**

> 到keepalived中注释掉persistent 10 然后/etc/init.d/keepalived reload,然后可以看到以后负载均衡两边都均衡了。

**其他导致负载不均衡的原因可能有：**

1. LVS自身的会话保持参数设置（-p 300,persistent 300）。优化：大公司尽量用cookies替代session
2. LVS调度算法设置，例如：rr，wrr，wlc，lc算法
3. 后端RS节点的会话保持参数，例如：apache的keepalive参数
4. 访问量较少的情况，不均衡的现象更加明显
5. 用户发送得请求时间长短，和请求资源多少大小。