[TOC]

# 简介

cluster集群就是一组独立的计算机，通过网络连接组合成一个组合来共同完一个任务.

计算机集群简称集群是一种计算机系统，它通过一组松散集成的计算机软件和/或硬件连接起来高度紧密地协作完成计算工作。在某种意义上，他们可以被看作是一台计算机。集群系统中的单个计算机通常称为节点，通常通过局域网连接，但也有其它的可能连接方式。集群计算机通常用来改进单个计算机的计算速度和/或可靠性.

# 集群的特点

## 高性能performance

一些需要很强的运算处理能力比如天气预报，核试验等。这就不是几台计算机能够搞定的。这需要上千台一起来完成这个工作的。

## 价格有效性

通常一套系统集群架构，只需要几台或数十台服务器主机即可，与动则上百王的专用超级计算机具有更高的性价比。

## 可伸缩性

当服务器负载压力增长的时候，系统能够扩展来满足需求，且不降低服务质量。

## 高可用性

尽管部分硬件和软件发生故障，整个系统的服务必须是7*24小时运行的。

# 集群的优势

## 透明性

如果一部分服务器宕机了业务不受影响，一般耦合度没有那么高，依赖关系没有那么高。比如NFS服务器宕机了其他就挂载不了了，这样依赖性太强。

## 高性能

访问量增加，能够轻松扩展。

## 可管理性

整个系统可能在物理上很大，但很容易管理。

## 可编程性

在集群系统上，容易开发应用程序，门户网站会要求这个

# 集群和分布式区别

小饭店原来只有一个厨师，切菜洗菜备料炒菜全干。后来客人多了，厨房一个厨师忙不过来，又请了个厨师，两个厨师都能炒一样的菜，这两个厨师的关系是集群。为了让厨师专心炒菜，把菜做到极致，又请了个配菜师负责切菜，备菜，备料，厨师和配菜师的关系是分布式，一个配菜师也忙不过来了，又请了个配菜师，两个配菜师关系是集群.

# 集群的基本思想

由于现代化业务上线的需求, 单服务器已经不能满足业务的需要, 业务服务器需要承载更多的访问请求，单台服务器故障(SPOF,single point of failure)将导致所有服务不可用. 此情况下, 需要把各种相同的业务服务器连合起来为某个相同的业务提供服务.以达到高并发,快速响应的需求。集群技术和Linux操作系统实现了一个高性能和高可用的服务器, 具有很好的可伸缩性,很好的可靠性, 很好的可管理性.

# 系统的扩展方式

scale up向上扩展：提高单台服务器的性能

scale out向外扩展：多台服务器联合起来满足同一个需要

# 集群的类型

计算机集群架构按照功能和结构一般分成以下几类：

1）负载均衡集群（Load balancing clusters）简称LBC

2）高可用性集群（High-availability clusters）简称HAC

3）高性能计算集群（High-perfomance clusters）简称HPC

4）网格计算（Gridcomputing）

下面分别详细说明:

## LB(Load Balancing)负载均衡集群

### LB集群组成部分

\- 负载均衡器

\- 调度器

\- 分发器

\- 上游服务器(Upstream server)

\- 后端的"真服务器(realserver)"

### LB架构的基本表现方式

前端:负载均衡器,调度器

后端:上游服务器[upstream server],后端服务器(real server)

SPOF:Single point of failure(单点故障)

### LB集群的实现方式

硬件实现:F5(BIG-IP),Citrix NetScaler, A10, Array, Redware

软件实现[通过横向扩展提高系统性能]:

lvs, haproxy, nginx, ats(apache traffic server), perlbal

### LB集群基于工作的协议层划分

传输层:

  LVS

  HAPorxy(模拟传输层, Mode TCP)

应用层

  HAPorxy(Mode HTTP)

  Nginx

  ATS

  Perlbal

### 负载均衡集群的作用

提供一种廉价、有效、透明的方法，来扩展网络设备和服务器的负载带宽、增加吞吐量，加强网络数据处理能力、提高网络的灵活性和可用性。

1）把单台计算机无法承受的大规模的并发访问或数据流量分担到多台节点设备上分别处理，减少用户等待响应的时间，提升用户

体验。

2）单个重负载的运算分担到多台节点设备上做并行处理，每个节点设备处理结束后，将结果汇总，返回给用户，系统处理能力得到大幅度提高。

3）7*24小时的服务保证，任意一个或多个设备节点设备宕机，不能影响到业务。在负载均衡集群中，所有计算机节点都应该提供相同的服务，集群负载均衡获取所有对该服务的如站请求。

## HA(High avaliability) 高可用集群

### HA的组成部分

   活动服务器(active)

   备用服务器(passive)

   可用性(Availability)=平均无故障时间/平均无故障时间+平均修复时间

   可用性衡量术语: 99%, 99.9%, 99.99%, 99.999%（二九，三九，四九..）

### HA集群的实现

提供冗余主机，提升系统可用性

软件方式的实现方式：

KeepAlived: 通过实现vrrp协议,来实现地址漂移;

AIS

hearbeat

cman + rgmanager(红帽5的实现方式组件,RHCS)

corosync + pacemaker

缓存都是KV格式的数据,即key=value的格式,其查找数据的时间是o(1),即查询时间是恒定的

### HA架构的基本表现方式

前端:Active,活动服务器

后端:passive,备用服务器

avalilability = 平均无故障时间/(平均无故障时间+平均修复时间)

### HA的框架

Messaging Layer:信息层,基础事务层

[用于实现传递心跳信息,集群事务信息的传递]

CRM:Cluster Resource Manager集群资源管理器层

[承上启下,将无HA功能的资源放置于Messaging Layer上,使资源实现高可用]

主要功能是可以让用户自定义哪些资源以什么样的形式如何分组,来实现高可用 

可实现HA的程序有[类型]:

heartbeatV1(自带资源管理器haresources)；配置接口是一个配置文件haresources

heartbeatV2(自带资源管理器crm)；在各节点运行一个crmd服务进程，监听在指定的端口，配置接口是一个命令行客户端程序：crmsh；或者GUI客户端程序hb-gui，其配置文件为：cib(cluster information base)，xml格式

heartbeatV3(分裂为三个软件：heartbeat + pacemaker + cluster-glue)；

pacemaker：守护进程crmd，配置文件cib.xml，配置接口：CLI：crm，pcs(红帽提供的)，GUI(程序名hb_gui)：hawk，LCMC，pacemaker-mgmt

rgmanager(resource group manager)：cman的资源管理器，守护进程(/etc/cluster/cluster.xml),配置接口：Conga(基于web界面的GUI，控制端luci，被控端ricci，基于https协议连接)

cman + rgmanager：可实现故障转移域Failover Domain，节点优先级功能node priority

守护进程：/etc/cluster/culster.xml

配置接口：Conga(基于web页面的GUI):lucl(控制端) + ricci(被控端)和 clustat,cman_tool

LRM：Local Resource Manager本地资源管理器(负责执行本地与资源action相关的事务)

RA：Resource Agent资源代理

检测某个服务是否运行，控制服务运行或停止，一般常见的都是各种脚本：

heartbeat legacy: heartbeat风格，通常是/etc/ha.d/haresources.d/目录下的脚本；

LSB格式脚本: /etc/init.d/*

OCF格式脚本(Open Cluster Framework)：provider(提供者)

STONITH(shoot the other node in the head):直接枪击另一个节点的头部。即控制硬件(软件或肉件)实现资源隔离的脚本或程序

# 资源隔离机制

节点隔离级别: STONITHH,需要一些硬件设备来辅助,比如电源交换机之类的设备,服务器硬件管理模块;

资源隔离级别:fencing,仅拒绝访问某些特定的硬件资源;

资源隔离的原因是集群会**分裂**.当节点之间无法进行心跳信息传递后,自己会抢夺资源.

解决办法:法定票数(大于总票数的一半),如果法定票数一致,则需指定一个仲裁.

相关参数:

 

```
with quorum
without quorum
```

 

```
with quorum
without quorum
```

当without quorum时,如何采取对资源的管控策略:

stopped 停止

ignore 忽略[慎用]

freeze 冻结,完成当前的资源服务,不再接收新的请求

suicide 自杀

# 集群的解决方案

## CentOS或RHEL系统的高可用

CentOS 5：

  RHCS：cman + gmanager

  也可选用第三方方案：corosync + pacemaker，heartbeat(V1,V2)，keepalived

CentOS 6：

  RHCS：cman + gmanager

  corosync + rgmanager

  cman + pacemaker

  heartbeat v3 + pacemaker

  keepalived  (6.4+的系统自带)

## HA集群的工作模型

  A/P:两节点,active/passive,主备模型;

  A/A:两节点,active/active,双主模型;

  N-M:N>M, N个节点,M个服务;活动节点数为N,备用节点数为N-M

  N-N:N个节点,N个服务

## 资源运行时的倾向性

rgmanager：故障转移域(failover domain) + 节点优先级(node priority) 

pacemaker：

资源粘性：资源留在当前节点上的倾向性，是个分数，数值为(-oo，oo+)

资源约束：inf正无穷，-inf负无穷，n正数，-n负数

位置约束：资源对某节点上运行的倾向性(-oo，oo+)

排列约束：定义资源彼此间在一起的倾向性

顺序约束：同一节点上的资源服务启动或关闭的次序约束

例如：定义一个高可用的web服务，有三个资源vip，httpd，nfs，如何限制它们要运行于同一个节点，并且按照vip-->nfs-->httpd次序启动

定义排序约束为inf，顺序约束为inf或mandator(强制)

## 资源类型

primitive，native：主资源，仅能运行于某一个节点，而不是某些节点

group：组资源，将多个资源以组的形式运行于同一个节点，对这些资源进行统一管理，类似排列约束的功能

clone：克隆资源，一个资源可运行于多个节点，克隆时需要指明：克隆总数和每个节点允许运行的克隆数

master/slave：主从资源，特殊的克隆资源，仅克隆2份

# HP: 高性能计算集群(high performance)

将多个CPU通过总线连接起来,共同进行计算,以达到计算效率

组合多台主机解决一个问题， 每个主机只负责其中一部分运算

## HP集群的实现

hadoop

# DS: 分布式系统(distributed system)

## 分布式存储

   HDFS[hadoop]

   mogileFS

   ClusterFS

   Ceph(已经收录到linux内核)

## 分布式计算

   Hadoop的YARN框架

   batch : MapReduce(批处理计算)

   in-memory : spark

   stream : storm

# 负载均衡构建需要考虑的问题

## Session的保持

如下应用场景

1，Session sticky:通过负载均衡器追踪用户的请求, 始终将同一IP的会话发送至相同的real server.

缺点: 如果一直访问的real server当机,部分用户不能正常访问

2，Session Cluster: 将session做成一个集群,进行多服务器同步session

缺点: 会话大的情况下, 服务器会大量的同步session,将浪费大量的资源,适合会话比较小的规模

3，通过全局的负载调度器(GSLB), 将请求调度到两个不同的机房,机房之间进行session同步

4，Session server : 将存储session的时候,使用网络存储,多服务器调用

缺点: 网络存储session的服务,将会有网络瓶颈, 或者session的存储档机

## 文件存储

使用分布式存储系统

共享存储

   NAS : network attached storage(文件级别存储服务器)

   SAN : Storage Area Network(块级别存储)

   DS : Distributed Storage(分布式文件级别存储)

数据同步

   Rsync

## 业务系统构建思路

分层:

   接入层

   缓存层

   业务层

   数据层

分布式

   应用

   数据

   存储

   计算   