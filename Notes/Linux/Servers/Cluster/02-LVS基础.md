# LVS简介

Linux Virtual Server

这是由章文嵩博士发起的一个开源项目,现在LVS已经是Linux内核标准的一部分,使用LVS可达到的技术目标是: 通过LVS的负载均衡技术和Linux操作系统实现一个高性能高可用的Linux服务器群集,它具有良好的可靠性,可拓展性和可操作性. 从而以低廉的成本实现最优的性能. LVS从1998年开始,发展到现在已经是一个比较成熟的项目了.利用LVS技术可实现高性能, 高可压缩的网络服务. 例如WWW服务,FTP服务, MAIL服务等

官网:http://www.linuxvirtualserver.org/

# LVS的工作原理

LVS工作在四层,根据请求报文的目标IP和目标PORT将其转发至后端主机集群中的某台服务器(其是根据调度算法).

![img](02-LVS%E5%9F%BA%E7%A1%80.assets/a9791b81-9d80-41a1-8653-bb759d316b42.png)

1,当用户向负载均衡调度器（Director Server）发起请求，调度器将请求发往至内核空间 ;

2,PREROUTING链首先会接收到用户请求，判断目标IP确定是本机IP，将数据包发往INPUT链;

3,IPVS是工作在INPUT链上的，当用户请求到达INPUT时，IPVS会将用户请求和自己已定义好的集群服务进行比对，如果用户请求的就是定义的集群服务，那么此时IPVS会强行修改数据包里的目标IP地址及端口，并将新的数据包发往POSTROUTING链 ;

4,POSTROUTING链接收数据包后发现目标IP地址刚好是自己的后端服务器，那么此时通过算法选路，将数据包最终发送给后端的服务器;

# 主从工作原理

LVS在基本的生产环境中,都会同时运行在二台硬件相近的服务器上：LVS Router（主 LVS ），一个作为备份LVS(备份 LVS )

主 LVS 

服务器在网站的前端起二个作用：

1，均衡负载压力到真实服务器(如apache)上.

2，检查后面真实服务器提供的服务是否正常.

备份LVS

用来监控主LVS和备份主服务器，在当故障出现时主LVS 死机 fail 掉了以后，就会启动自己来接管主 LVS 的工作;

其中有一个叫 Pulse (心跳服务) 运行在主LVS和备份LVS上。在备份 LVS 上，每秒 pulse 发送一个心跳(heartbeat)到主LVS的外网接口检查主LVS的服务是否正常。当然在主LVS上，也有pulse服务,它主要是响应备份LVS的心跳;

# LVS进程调用

ipvsadm工具(RedHat开发)去配置和维护 IPVS 路由表，它会为每一个在真实服务器上的虚拟服务启动一个nanny进程。每一个nanny进程去检查真实服务器上的服务状态，如果有异常.就会将故障情况通知LVS进程。当故障时，LVS进程通知 ipvsadm 在 IPVS 路由表中将此节点删除。当然,它发现故障的机器恢复时也能自动的加入到服务中来;

如果备份LVS未收到来自于主LVS的响应，它将调用send_arp将虚拟IP地址再分配到备份LVS的公网接口上。并在公网接口和局域网接口上分别发送一个命令去关掉主LVS上的LVS进程。同时启动自己的LVS进程来调度客户端请求.

支持TCP,UDP,SCTP,AH,ESP,AH_ESP等协议的众多服务

# LVS工具

ipvs：工作在内核空间，netfilter上的input链上，是真正生效负责调度的代码，负责监听数据包来源的请求，到达端口时做一次重定向，仅提供负载均衡框架.

ipvsadm：工作在用户空间，负责为ipvs编写规则，定义谁是集群服务，后端有哪些集群server以及使用什么方法去调度.

# 名词术语

Real server：简称RS，后端集群的节点，即真正提供服务的服务器

Director Virtual server：简称VS，也可称为Director，负责接收客户端请求，并将请求按照某种算法分发到后台的rs

Virtual ip：简称VIP，LVS的前端ip，用来向外部客户端提供服务的ip地址

Director ip：简称DIP，LVS的后端ip，用来和RIP联系的ip

Real ip：简称RIP，集群节点所用的ip

Client ip：简称CIP，公网ip，即外部客户端的ip地址

# ipvsadm使用方法

-A: 在内核的虚拟服务器表中添加一条新的虚拟服务器记录,也就是增加一台新的虚拟服务器;

-E: 编辑虚拟服务器记录;

-D: 删除内核虚拟服务器表中的一条虚拟服务器记录;

-C: 清除内核虚拟服务器表中的所有记录;

-R: 恢复虚拟服务器规则;

-S: 保存虚拟服务器规则，输出为 -R 选项可读的格式;

-a: 在内核虚拟服务器表的一条记录里添加一条新的真实服务器记录,也就是在一个虚拟服务器中增加RS;

-e: 编辑一条虚拟服务器记录中的某条RS记录;

-d: 删除一条虚拟服务器记录中的某条RS记录;

-L|-l: 显示内核虚拟服务器表;

-Z: 虚拟服务表计数器清零(清空当前的连接数量等);

--set tcp tcpfin udp: 设置连接超时值;

--start-daemon: 启动同步守护进程.他后面可以是master 或 backup,用来说明LVS Router是 master 或是 backup.在这个功能上也可以采用keepalived的VRRP功能;

--stop-daemon: 停止同步守护进程;

-h: 显示帮助信息;

其他的选项: 

-t service-address: 说明虚拟服务器提供的是tcp 的服务[vip:port] or [real-server-ip:port];

-u service-address: 说明虚拟服务器提供的是udp 的服务[vip:port] or [real-server-ip:port];

-f fwmark: 说明是经过iptables 标记过的服务类型;

-s scheduler: 使用的调度算法，有这样几个选项rr|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq,默认的调度算法是： wlc;

-p [timeout]: 持久稳固的服务,这个选项的意思是来自同一个客户的多次请求，将被同一台真实的服务器处理;timeout 的默认值为300 秒;

-M: 指定客户地址的子网掩码;

-r server-address: 真实的服务器[Real-Server:port];

-g: 指定LVS 的工作模式为直接路由模式（也是LVS 默认的模式）;

-i: 指定LVS的工作模式为隧道模式;

-m: 指定LVS 的工作模式为NAT 模式;

-w weight: 真实服务器的权值;

-n: 输出IP 地址和端口的数字形式;

-c: 显示LVS目前的连接 如：ipvsadm -L -c;

-6: 如果fwmark用的是ipv6地址需要指定此选项;

--mcast-interface interface: 指定组播的同步接口;

--timeout: 显示tcp tcpfin udp 的timeout 值 如：ipvsadm -L --timeout;

--daemon: 显示同步守护进程状态;

--stats: 显示统计信息;

--rate: 显示速率信息;

--sort: 对虚拟服务器和真实服务器排序输出;

--exact: 显示精确值

# LVS的四种架构类型

## NAT

Network Address Translation（LVS-NAT)[常用]

![img](02-LVS%E5%9F%BA%E7%A1%80.assets/dcf71e84-9030-4234-8167-62d639841112.jpg)

![img](02-LVS%E5%9F%BA%E7%A1%80.assets/5d20cd11-87a6-4e8b-a0b0-48716a2a2ad8.png)

### 工作机制

目标地址转换，所有客户端的请求都被Director根据访问请求和算法定向转发到后台的real server上

客户端请求 -> 源IP:cip,目标ip:vip -> 源IP:cip,目标ip:dip -> 源IP:cip,目标ip:rip

服务器返回 -> 源ip:rip,目标ip:cip -> 源ip:dip,目标ip:cip -> 源ip:vip,目标ip:cip

(a)当用户请求到达Director Server，此时请求的数据报文会先到内核空间的PREROUTING链。 此时报文的源IP为CIP，目标IP为VIP 

(b)PREROUTING检查发现数据包的目标IP是本机，将数据包送至INPUT链

(c)IPVS比对数据包请求的服务是否为集群服务，若是，修改数据包的目标IP地址为后端服务器IP，然后将数据包发至POSTROUTING链。 此时报文的源IP为CIP，目标IP为RIP

(d)POSTROUTING链通过选路，将数据包发送给Real Server

(e)Real Server比对发现目标为自己的IP，开始构建响应报文发回给Director Server。 此时报文的源IP为RIP，目标IP为CIP 

(f)Director Server在响应客户端前，此时会将源IP地址修改为自己的VIP地址，然后响应给客户端。此时报文的源IP为VIP，目标IP为CIP

### 特性

所有RS的网关必须指向DIP，用来响应客户端的请求；

RIP和DIP一般是内网私有地址，用于和集群节点RS通信；

Director会响应所有的客户端请求，负载较大；

Director支持端口重映射，即前端使用标准端口，后端可使用非标准端口；

VS必须是linux，RS可以是任何可支持服务的OS;

缺陷: 请求和响应报文都要经过Director，高负载场景中，Director容易成为性能瓶颈；

## DR

Director Routing(LVS-DR)[常用]

大规模场景中,lvs默认使用的架构

![img](02-LVS%E5%9F%BA%E7%A1%80.assets/9de5fe19-5eb7-4232-84f1-6284ad1e0430.jpg)

![img](02-LVS%E5%9F%BA%E7%A1%80.assets/83463909-91a9-4f72-90b9-850ba09d1f21.png)

### 工作机制

客户端请求VIP,路由器收到该请求后,进行arp广播,找到VIP对应的主机Director,Directory将接收到的请求报文进行处理,源IP和目标IP不变,将源MAC设定自己的MAC,目标MAC设定为调度算法选择的目标RS主机的MAC,RS收到该请求进行回应,此时源IP和目标IP都没有变,改变的只是目标mac为路由器的内网端口mac.

根据以上工作流程,需要解决的问题:

1,集群后端的rs服务器必须有2个地址: rip和vip,且rip和dip互通,所有vip相同;

2.router必须知道内网只有一个vip,且该vip是directory,也就是说rs的vip需要禁止对外广播自己的vip地址;

解决方案

第一种:在前端路由器上,做静态地址和mac地址绑定,即绑定Director的VIP地址和其mac,以便强制使报文发给Director的VIP；

第二种:如果前端路由器没有操作权限，可以在RS上，利用arptables防火墙规则，限制RS接收前端路由器的广播报文；

第三种:修改RS上的内核参数（linux），将RS上的VIP配置在自己的lo接口的别名上，并限制其响应发送ARP广播请求；

### 特性

RS可以使用私有地址,亦可以使用公网地址(这时可以远程连接RIP对其访问)；

RS和Director必须在同一个物理网络中；

所有的请求报文需经由Director来调度，但响应报文决不能经过Director；

不支持端口映射；

RS的网关不允许指向DIP；

RS可以是大多数常见的OS；

缺陷:RS和DS必须在同一物理网络中

## FULLNAT

FULL Network Address Translation（LVS-FULLNAT）

### 工作机制

directory通过同时修改请求报文的目标地址和原地址进行转发.

Director收到CIP的请求报文后, 将源IP修改为DIP，目标地址修改为RIP，然后转发给RS, 当RS主机收到报文后回应，源IP为RIP，目标IP为Director的DIP，Director收到报文后, 将源IP修改为自己的VIP，目标IP修改为CIP。

### 特性

VIP是公网地址; RIP和DIP是私网地址,二者可以不在同一网络中,但需要通过路由互相通信；

RS收到的请求报文的源IP为DIP,因此其响应报文将发送给DIP；

请求报文和响应报文都必须由Director调度；

支持端口映射机制；

RS可以使用任意OS；

## TUN

IP Tunneling(LVS-TUN)

### 工作机制

![img](02-LVS%E5%9F%BA%E7%A1%80.assets/9b877a4a-d58e-43ea-a8ab-27847fe23c55.png)

隧道，与LVS-DR的结构差不多，但Director和RS可在不同的网路中，可实现异地容灾功能;

不修改请求报文的ip首部,而是通过在原有的ip首部之外,再封装一个ip首部.

### 传输过程

CIP发送报文到Director的VIP，基于隧道来传输，Director收到报文后，保持报文的源IP和目标IP不变，在其外层再封装一个源IP和目标IP，这个额外封装的源IP为Director的DIP，目标IP为RIP，然后将此报文发往另一个网络中的RIP，确保每个RS都有自己的VIP，RS接收到Director的报文后，回应时，源IP为自己的VIP，目标IP直接为CIP,不在经过Directory

### 特性

RIP,VIP,DIP全都是公网地址；

RS的网关不允许指向DIP；

所有的请求报文需经由Director来调度，但响应报文决不能经过Director；

不支持端口映射；

RS的OS必须支持隧道功能；

# LVS Scheduler(调度器)

支持8种调度算法

## 1,固定调度算法

仅根据调度算法本身进行调度，不考虑实时的连接数予以分配,起点公平 .

- Round-robin（RR）轮询：将外部请求按顺序轮流分配到集群中的真实服务器RS上,它均等地对待每一台服务器,而不管服务器上实际的连接数和系统负载;
- Weighted round-robin（WRR）加权轮询：给每台Real Server分配一个权重/位列，权重越大，分到的请求数越多，能者多劳； 
- Destination hashing （DH）目标地址哈希：来自CIP的请求，只要目标地址相同，将始终通过同一个网关进行转发，可用于正向代理等，不常用； 
- Source hashing（SH）源地址哈希：表示来源于同一个CIP的请求将始终被定向到同一个RS服务器，可实现session保持；

## 2,动态调度算法

通过检查RS服务器上当前连接的活动状态来重新决定下一步调度方式该如何实现,结果公平 .

- Least Connection （LC）最少连接：调度器通过”最少连接”调度算法,动态地将网络请求调度到已建立的链接数最少的服务器上,如果集群系统的真实服务器具有相近的系统性能，采用”最小连接”调度算法可以较好地均衡负载,overhead值小的被选中;

  算法：overhead=Active * 256 + Inactive [负载 = 活动连接数 * 256 + 非活动连接数]

- Weighted Least-Connection（WLC）加权最少连接：在集群系统中的服务器性能差异较大的情况下，调度器采用”加权最少链接”调度算法优化负载均衡性能，具有较高权值的服务器将承受较大比例的活动连接负载.调度器可以自动问询真实服务器的负载情况，并动态地调整其权值;

  算法：overhead=（Active * 256 + Inactive）/ weight [负载连接数=（活动连接数*256+非活动连接数）÷权重]

  一种比较理想的算法,overhead值小的被选中;

- Shortest Expected Delay (SED)  最短期望延迟：不再考虑非活动连接数；

  算法：Overhear = (Active + 1) / Weight

- Never Queue (NQ) 永不排队算法：对SED的改进，当新请求过来的时候不仅要取决于SED算法所得到的值，还要取决于Real Server上是否有活动连接,如果active为0,则无需计算,直接分过去;
- Locality-Based Least-Connection  (LBLC) 基于本地状态的最少连接：动态DH，在DH算法的基础上还要考虑服务器上的活动连接数。“基于局部性的最少链接” 调度算法是针对目标IP地址的负载均衡，目前主要用于Cache集群系统。该算法根据请求的目标IP地址找出该目标IP地址最近使用的服务器，若该服务器是可用的且没有超载，将请求发送到该服务器；若服务器不存在，或者该服务器超载且有服务器处于一半的工作负载，则用”最少链接”的原则选出一个可用的服务器，将请求发送到该服务器.
- Locality-Based Least-Connection  with  Replication  Scheduling  (LBLCR) 带复制的LBLC算法：LBLC算法的改进，可以让后端的缓存服务器之间互相通信,该调度算法也是针对目标IP地址的负载均衡，目前主要用于Cache集群系统。它与LBLC算法的不同之处是它要维护从一个目标IP地址到一组服务器的映射，而LBLC算法维护从一个目标IP地址到一台服务器的映射。该算法根据请求的目标IP地址找出该目标IP地址对应的服务器组，按”最小连接”原则从服务器组中选出一台服务器，若服务器没有超载，将请求发送到该服务器，若服务器超载；则按”最小连接”原则从这个集群中选出一台服务器，将该服务器加入到服务器组中，将请求发送到该服务器。同时，当该服务器组有一段时间没有被修改，将最忙的服务器从服务器组中删除，以降低复制的程度

# Session持久机制

1，session绑定：始终将来自同一个源IP的请求定向至同一个RS;

source ip hash

cookie hash

缺点：没有容错能力，有损均衡效果.

2，session复制：在RS间同步session，每个RS拥有集群中的所有session;

缺点：不适用于大规模集群.

3，session服务器：利用单独部署的服务器来统一管理集群中的session;

缺点：session服务器为单点故障所在，一旦损坏，所有信息全部丢失.

