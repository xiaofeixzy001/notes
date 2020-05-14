[TOC]

# Linux系统网络概念

对于linux而言，ip地址属于内核，而非网卡

网络属性：IP/NETMASK路由：主机路由，网络路由，默认网关DNS：主，从主机名

配置IP工具：ifconfig(net-tools软件包，7版本以上系统需要额外安装),ip(iproute2软件包)网络配置文件：/etc/sysconfig/network

网口配置文件：/etc/sysconfig/network-scripts/ifcfg-ethX

修改主机名：/etc/sysconfig/network,/etc/resolv.conf

还有一种方式，图形界面的:setup

配置linux网络属性有临时生效和永久生效两种方式

# 基本网络配置

## 修改主机名

临时修改, 退出当前shell重新登录生效,重启计算机失效.

 

```
hostname node1
```



永久生效, 重新登录或重启计算机生效.

 

```
hostname node1

vim /etc/sysconfig/network
> NETWORKING=yes
> HOSTNAME=node1  # 修改此处

vim /etc/hosts
> 127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 node1
```



## 修改IP地址

临时配置,重启网络服务或重启计算机后失效

 

```
ifconfig eth0 192.168.1.10/24
ifconfig eth0 100.0.0.1 netmask 255.255.255.0 
# 网卡eth0配置临时ip100.0.0.1，掩码为255.255.255.0
#ip addr add 100.0.0.1/24 dev eth0    \\此命令可以设置多个临时ip给网卡eth0
#ip addr del 100.0.0.1 dev eth0
#route add default gw 100.0.0.254
#ip route add default via 100.0.0.254

# 给网络接口配置多个IP地址
ifconfig eth0:0 192.168.100.7/24 up
ifconfig eth0:0 down
```



永久配置生效,需重启网络服务或计算机

 

```
vim /etc/sysconfig/network-scripts/ifcfg-eth0
"""
DEVICE=eth0  # 网卡名称,第二块网卡一般为eth1,以此类推...
HWADDR=00:0C:29:77:73:B7  # mac地址
TYPE=Ethernet
UUID=3d15fc86-03ba-4d8e-82b8-d0ae451abf90
ONBOOT=yes  # 是否开机激活网卡
NM_CONTROLLED=yes
BOOTPROTO=none  # 地址获取IP方式,bootp|dhcp|static|none
IPADDR=172.16.100.7
NETMASK=255.255.255.0
GATEWAY=172.16.100.254
DNS1=219.141.136.10
DN2=8.8.8.8
ARPCHECK=no
USERCTL=no  # 是否允许普通用户控制此网卡设备
IPV6INIT=no  # 是否启用ipv6
PEERDNS=yes  # 是否在BOOTPROTO为dhcp时接受由DHCP服务器指定的DNS
"""

# 为eth0：0配置多个ip地址，永久生效
cp ifcfg-eth0 ifcfg-eth0:0
vim ifcfg-eth0:0
"""
DEVICE=eth0:0
ONBOOT=yes
IPADDR=192.168.100.7
NETMASK=255.255.255.0
GATEWAY=192.168.100.254
DNS1=219.141.136.10
"""
```



保存退出,重启网络服务或网卡

 

```
service network start|stop|restart
# 或
ifup eth0
ifdown eth0
# 或
ifconfig eth0 up|down
# 或
ip link set eth0 up|down
```



## 修改DNS

linux系统和windows系统都有自己的一个本地dns解析文件,也就是hosts文件,当访问一个域名时,最先检查本地hosts文件解析记录,如果没有匹配,则会向本地网络配置的DNS地址查找...

修改本地hosts文件

 

```
vim /etc/hosts
"""
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain node1
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.100.7    node1.com      node1
172.16.100.8    node2.com      node2
172.16.100.9    node3.com      node3
172.16.100.10   node4.com      node4
"""
```



修改本地网卡的DNS解析记录,同步网卡配置文件内信息

 

```
vim /etc/resolv.conf
"""
nameserver 202.106.46.151
nameserver 8.8.8.8
"""
```



## 配置路由功能

linux系统自带路由功能

临时配置路由条目

 

```
route -n  # 查看路由信息
"""
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
172.16.100.0    0.0.0.0         255.255.255.0   U     0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
0.0.0.0         172.16.100.254  0.0.0.0         UG    0      0        0 eth0
"""
```



添加路由

 

```
route add default gw 172.16.100.254  # 添加默认路由,默认访问任何网络下一跳都为172.16.100.254
route add -host 192.168.1.254/24 gw 172.16.100.254  # 添加一条主机路由,可访问主机192.168.1.254, 下一跳地址为172.16.100.254
route add -net 192.168.1.0/24 gw 192.168.100.254  # 添加一条网络路由,可访问192.168.1.0网络,下一跳地址为192.168.100.254
```



删除路由

 

```
route del default gw 172.16.100.254
```



注意:在指定下一跳地址时，本机的网关必须和下一跳地址在同一网段.

配置永久生效路由

方法一:

 

```
vim /etc/sysconfig/network-scripts/route-eht0
192.168.1.0/24 via 192.168.100.254
```



方法二：

  第一组：

  ADDRESS0= 目标地址

  NETMASK0= 目标掩码

  GATEWAY0= 下一跳地址

  第二组：

  ADDRESS1= 目标地址

  NETMASK1= 目标掩码

  GATEWAY1= 下一跳地址

注意：两种方法不能混用

# 开启路由转发功能

适用环境：一台linux服务器，两块网卡，分别连接不同网段

eth0:192.168.100.254

eth1:172.16.100.7

从而让eth0和eth1可以互通

临时生效:

 

```
echo "1" > /proc/sys/net/ipv4/ip_forward
# 默认ip_forward为0，表示关闭，修改为1表示打开，重启失效
```



永久生效：

 

```
vim /etc/sysctl.conf
"""
net.ipv4.ip_forward = 1  # 默认0,修改为1表示开启包转发
"""
```



然后添加静态路由：

 

```
route add -net 192.168.100.0 netmask 255.255.255.0 dev eth0
```



# 网络测试工具

网络检查:

ping [icmp协议]

-c: 发包次数

-s: 包大小

-i: 发包间隔

ping -c3 -s640 -i3 www.baidu.com

traceroute 路由跟踪

telnet 

nmap

# 抓包工具

tcpdump

tcpdump -n icmp -i eth0

# 域名解析

dig

nslookup

host

tcpdump

# 多网卡绑定

多网卡的7种bond模式原理

Linux网卡绑定mode共有七种(0~6) bond0、bond1、bond2、bond3、bond4、bond5、bond6

常用的有三种

mode=0：平衡负载模式，有自动备援，但需要”Switch”支援及设定。

mode=1：自动备援模式，其中一条线若断线，其他线路将会自动备援。

mode=6：平衡负载模式，有自动备援，不必”Switch”支援及设定。

需要说明的是如果想做成mode 0的负载均衡,仅仅设置这里options bond0 miimon=100 mode=0是不够的,与网卡相连的交换机必须做特殊配置（这两个端口应该采取聚合方式），因为做bonding的这两块网卡是使用同一个MAC地址.从原理分析一下（bond运行在mode 0下）：

mode 0下bond所绑定的网卡的IP都被修改成相同的mac地址，如果这些网卡都被接在同一个交换机，那么交换机的arp表里这个mac地址对应的端口就有多 个，那么交换机接受到发往这个mac地址的包应该往哪个端口转发呢？正常情况下mac地址是全球唯一的，一个mac地址对应多个端口肯定使交换机迷惑了。所以 mode0下的bond如果连接到交换机，交换机这几个端口应该采取聚合方式（cisco称为 ethernetchannel，foundry称为portgroup），因为交换机做了聚合后，聚合下的几个端口也被捆绑成一个mac地址.我们的解 决办法是，两个网卡接入不同的交换机即可。

mode6模式下无需配置交换机，因为做bonding的这两块网卡是使用不同的MAC地址。

七种bond模式说明

第一种模式：mod=0 ，即：(balance-rr) Round-robin policy（平衡抡循环策略）

特点：传输数据包顺序是依次传输（即：第1个包走eth0，下一个包就走eth1….一直循环下去，直到最后一个传输完毕），此模式提供负载平衡和容错能力；但是我们知道如果一个连接或者会话的数据包从不同的接口发出的话，中途再经过不同的链路，在客户端很有可能会出现数据包无序到达的问题，而无序到达的数据包需要重新要求被发送，这样网络的吞吐量就会下降

第二种模式：mod=1，即： (active-backup) Active-backup policy（主-备份策略）

特点：只有一个设备处于活动状态，当一个宕掉另一个马上由备份转换为主设备。mac地址是外部可见得，从外面看来，bond的MAC地址是唯一的，以避免switch(交换机)发生混乱。此模式只提供了容错能力；由此可见此算法的优点是可以提供高网络连接的可用性，但是它的资源利用率较低，只有一个接口处于工作状态，在有 N 个网络接口的情况下，资源利用率为1/N

第三种模式：mod=2，即：(balance-xor) XOR policy（平衡策略）

特点：基于指定的传输HASH策略传输数据包。缺省的策略是：(源MAC地址 XOR 目标MAC地址) % slave数量。其他的传输策略可以通过xmit_hash_policy选项指定，此模式提供负载平衡和容错能力

第四种模式：mod=3，即：broadcast（广播策略）

特点：在每个slave接口上传输每个数据包，此模式提供了容错能力

第五种模式：mod=4，即：(802.3ad) IEEE 802.3ad Dynamic link aggregation（IEEE 802.3ad 动态链接聚合）

特点：创建一个聚合组，它们共享同样的速率和双工设定。根据802.3ad规范将多个slave工作在同一个激活的聚合体下。

外出流量的slave选举是基于传输hash策略，该策略可以通过xmit_hash_policy选项从缺省的XOR策略改变到其他策略。需要注意的 是，并不是所有的传输策略都是802.3ad适应的，尤其考虑到在802.3ad标准43.2.4章节提及的包乱序问题。不同的实现可能会有不同的适应 性。

必要条件：

条件1：ethtool支持获取每个slave的速率和双工设定

条件2：switch(交换机)支持IEEE 802.3ad Dynamic link aggregation

条件3：大多数switch(交换机)需要经过特定配置才能支持802.3ad模式

第六种模式：mod=5，即：(balance-tlb) Adaptive transmit load balancing（适配器传输负载均衡）

特点：不需要任何特别的switch(交换机)支持的通道bonding。在每个slave上根据当前的负载（根据速度计算）分配外出流量。如果正在接受数据的slave出故障了，另一个slave接管失败的slave的MAC地址。

该模式的必要条件：ethtool支持获取每个slave的速率

第七种模式：mod=6，即：(balance-alb) Adaptive load balancing（适配器适应性负载均衡）

特点：该模式包含了balance-tlb模式，同时加上针对IPV4流量的接收负载均衡(receive load balance, rlb)，而且不需要任何switch(交换机)的支持。接收负载均衡是通过ARP协商实现的。bonding驱动截获本机发送的ARP应答，并把源硬件地址改写为bond中某个slave的唯一硬件地址，从而使得不同的对端使用不同的硬件地址进行通信。

来自服务器端的接收流量也会被均衡。当本机发送ARP请求时，bonding驱动把对端的IP信息从ARP包中复制并保存下来。当ARP应答从对端到达 时，bonding驱动把它的硬件地址提取出来，并发起一个ARP应答给bond中的某个slave。使用ARP协商进行负载均衡的一个问题是：每次广播 ARP请求时都会使用bond的硬件地址，因此对端学习到这个硬件地址后，接收流量将会全部流向当前的slave。这个问题可以通过给所有的对端发送更新 （ARP应答）来解决，应答中包含他们独一无二的硬件地址，从而导致流量重新分布。当新的slave加入到bond中时，或者某个未激活的slave重新 激活时，接收流量也要重新分布。接收的负载被顺序地分布（round robin）在bond中最高速的slave上

当某个链路被重新接上，或者一个新的slave加入到bond中，接收流量在所有当前激活的slave中全部重新分配，通过使用指定的MAC地址给每个 client发起ARP应答。下面介绍的updelay参数必须被设置为某个大于等于switch(交换机)转发延时的值，从而保证发往对端的ARP应答 不会被switch(交换机)阻截。

必要条件：

条件1：ethtool支持获取每个slave的速率；

条件2：底层驱动支持设置某个设备的硬件地址，从而使得总是有个slave(curr_active_slave)使用bond的硬件地址，同时保证每个bond 中的slave都有一个唯一的硬件地址。如果curr_active_slave出故障，它的硬件地址将会被新选出来的 curr_active_slave接管

其实mod=6与mod=0的区别：mod=6，先把eth0流量占满，再占eth1，….ethX；而mod=0的话，会发现2个口的流量都很稳定，基本一样的带宽。而mod=6，会发现第一个口流量很高，第2个口只占了小部分流量

Linux网口绑定

网卡bond是通过多张网卡绑定为一个逻辑网卡，实现本地网卡的冗余，带宽扩容和负载均衡，在生产场景中是一种常用的技术。Kernels 2.4.12及以后的版本均供bonding模块，以前的版本可以通过patch实现。可以通过以下命令确定内核是否支持 bonding：

 

```
[root@study-linux ~]# cat /boot/config-2.6.32-642.el6.x86_64 | grep -i bonding
"""
CONFIG_BONDING=m
"""
```



通过网口绑定(bond)技术,可以很容易实现网口冗余，负载均衡，从而达到高可用高可靠的目的。前提约定：

2个物理网口分别是：eth0,eth1

绑定后的虚拟口是：bond0

服务器IP是：172.16.100.10

第一步，配置设定文件:

 

```
[root@study-linux ~]# cat /etc/redhat-release
[root@study-linux ~]# uname -r
[root@study-linux ~]# cd /etc/sysconfig/network-scripts/
[root@study-linux network-scripts]# cp ifcfg-eth0 ifcfg-bond0
[root@study-linux network-scripts]# cp ifcfg-eth0 ifcfg-eth0.bak
[root@study-linux network-scripts]# cp ifcfg-eth1 ifcfg-eth1.bak
[root@study-linux network-scripts]# vim ifcfg-bond0
"""
DEVICE=bond0
TYPE=Ethernet
BOOTPROTO=static
ONBOOT=yes
IPADDR=172.16.100.10
NETMASK=255.255.255.0
GATEWAY=172.16.100.1
DNS1=219.141.136.10
"""

[root@study-linux network-scripts]# vim ifcfg-eth0
"""
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
"""

[root@study-linux network-scripts]# vim ifcfg-eth1
"""
DEVICE=eth1
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
"""
```



第二步，修改/etc/modprobe.d/modprobe.conf,没有则创建一个

[root@study-linux ~]# vim /etc/modprobe.d/modprobe.conf

"""

alias bond0 bonding

options bond0 miimon=200 mode=0

"""

\# miimon:配置bond0的链路检查时间为200ms,模式为0

注意：

linux网卡bonging的备份模式实验在真实机器上做完全没问题（前提是linux内核支持），但是在vmware workstation虚拟中做就会出现如下图问题。  

![img](Linux%E7%BD%91%E7%BB%9C.assets/ab1daf40-d059-4edf-a6cb-8bedf10f341b.png)

配置完成后出现如上图问题，但是bond0能够正常启动也能够正常使用，只不过没有起到备份模式的效果。当使用ifdown eth0后，网络出现不通现象。

内核文档中有说明：bond0获取mac地址有两种方式,一种是从第一个活跃网卡中获取mac地址，然后其余的SLAVE网卡的mac地址都使用该mac地址；另一种是使用fail_over_mac参数，是bond0使用当前活跃网卡的mac地址，mac地址或者活跃网卡的转换而变。  

既然vmware workstation不支持第一种获取mac地址的方式，那么可以使用fail_over_mac=1参数，所以这里我们添加fail_over_mac=1参数

2.加载模块(重启系统后就不用手动再加载了)

[root@test ~]# modprobe bonding

3.确认模块是否加载成功：

[root@test ~]# lsmod | grep bonding

bonding 100065 0

第三步，重启一下网络，然后确认一下状况：

 

```
[root@test ~]# /etc/init.d/network restart
[root@test ~]# cat /proc/net/bonding/bond0
Ethernet Channel Bonding Driver: v3.5.0 (November 4, 2008)
Bonding Mode: fault-tolerance (active-backup)
Primary Slave: None
Currently Active Slave: eth0
……
[root@test ~]# ifconfig | grep HWaddr
bond0 Link encap:Ethernet HWaddr 00:16:36:1B:BB:74
eth0 Link encap:Ethernet HWaddr 00:16:36:1B:BB:74
eth1 Link encap:Ethernet HWaddr 00:16:36:1B:BB:74
```



从上面的确认信息中，我们可以看到3个重要信息：

1.现在的bonding模式是active-backup

2.现在Active状态的网口是eth0

3.bond0,eth1的物理地址和处于active状态下的eth0的物理地址相同，这样是为了避免上位交换机发生混乱。

任意拔掉一根网线，然后再访问你的服务器，看网络是否还是通的。

第四步，系统启动自动绑定、增加默认网关：

[root@test ~]# vi /etc/rc.d/rc.local

\#追加

ifenslave bond0 eth0 eth1

route add default gw 192.168.0.1

\#如可上网就不用增加路由，0.1地址按环境修改.

\------------------------------------------------------------------------

留心：前面只是2个网口绑定成一个bond0的情况，如果我们要设置多个bond口，比如物理网口eth0和eth1组成bond0，eth2和eth3组成bond1，

那么网口设置文件的设置方法和上面第1步讲的方法相同，只是/etc/modprobe.d/bonding.conf的设定就不能像下面这样简单的叠加了：

alias bond0 bonding

options bonding mode=1 miimon=200

alias bond1 bonding

options bonding mode=1 miimon=200

正确的设置方法有2种：

第一种,你可以看到，这种方式的话，多个bond口的模式就只能设成相同的了：

alias bond0 bonding

alias bond1 bonding

options bonding max_bonds=2 miimon=200 mode=1

第二种，这种方式，不同的bond口的mode可以设成不一样：

alias bond0 bonding

options bond0 miimon=100 mode=1

install bond1 /sbin/modprobe bonding -o bond1 miimon=200 mode=0

仔细看看上面这2种设置方法，现在如果是要设置3个，4个，甚至更多的bond口，你应该也会了吧！

后记：简单的介绍一下上面在加载bonding模块的时候，options里的一些参数的含义：

miimon 监视网络链接的频度，单位是毫秒，我们设置的是200毫秒。

max_bonds 配置的bond口个数

mode bond模式，主要有以下几种，在一般的实际应用中，0和1用的比较多，

如果你要深入了解这些模式各自的特点就需要靠读者你自己去查资料并做实践了。

# VMWare虚拟机问题

## 克隆后网卡问题

克隆系统后,网卡无法正常工作及解决办法:

 

```
[root@test ~]# service network start
"""
Bringing up loopback insterface: [ OK ]
Bringing up interface eth0: Device eth0 does not seem to be present,delaying initialization. [FAILED]
"""

[root@test ~]# ifconfig -a  # 查看所有网卡信息,发现少了eth0,多了一个eth1
[root@test ~]# ls /etc/sysconfig/network-scripts/  # 有eth0文件,没有eth1文件
[root@test ~]# ifconfig eth1  # 查看新网卡mac
[root@test ~]# nmcli con  # 查看UUID

# 解决办法1:
[root@test ~]# cp /etc/sysconfig/network-scripts/ifcfg-eth0  /etcsysconfig/network-scripts/ifcfg-eth1
[root@test ~]# vim /etc/udev/rules.d/70-persistent.rules
"""
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="00:0c:29:77:73:b7", ATTR{type}=="1", KERNEL=="eth*", NAME="eth1"  # 这里的eth0改为eth1
"""
[root@test ~]# service network start

# 解决办法2: 删除模板及网卡配置里mac所在行及uuid所在行
[root@test ~]# > /etc/udev/rules.d/70-persistent-net.rules
[root@test ~]# reboot
```



总之，只要保证/etc/sysconfig/network-scripts/ifcfg-eth0 与/etc/udev/rules.d/70-persistent-net.rules的信息一致即可，即网卡地址与网卡编号一致，这样service network restart 就可以配置成功.

错误2: Error: Connection activation failed: Device not managed by NetworkManager解决办法：

 

```
# ——Remove Network Manager from startup Services.
chkconfig NetworkManager off

# —— Add Default Net Manager
chkconfig network on

# ——Stop NetworkManager first
service NetworkManager stop

# ——and then start Default Manager
service network restart
```



修改主机名和IP脚本

 

```
#!/bin/sh
if [ $# -ne 2 ];then
  echo "/bin/sh $0 hostname PartIP"
  exit 1
fi
sed -i "s#oldboy#$1#g" /etc/sysconfig/network
hostname $1
sed -i "s#100#$2#g" /etc/sysconfig/network-scripts/ifcfg-eth0
sed -i "s#100#$2#g" /etc/sysconfig/network-scripts/ifcfg-eth1
```



## VM虚拟机共享上网

1，虚拟机的网络模式选择VMnet8(NAT)

![img](Linux%E7%BD%91%E7%BB%9C.assets/95640e62-a548-49b8-8e60-50a035461ed5.png)

2，修改IP地址

如虚拟机的系统IP为 172.16.100.7,本地的VMnet8网卡IP为172.16.100.254/24,DNS和本地真实网卡的DNS一致:

![img](Linux%E7%BD%91%E7%BB%9C.assets/972371cb-42c3-4933-9757-486d174049fe.png)

3，打开虚拟机主程序, 打开 编辑 ---> 虚拟网络编辑器 --> 选择VMnet8

修改子网IP地址为172.16.100.0网段

![img](Linux%E7%BD%91%E7%BB%9C.assets/597cd651-15c7-4752-bf35-2ae45fcd91f8.png)

然后点击 NAT设置, 修改网关IP为172.16.100.1

![img](Linux%E7%BD%91%E7%BB%9C.assets/4826dfcd-062a-4f72-9278-3291a28904bc.png)

4, 最重要的一步,打开本地真实网卡的属性, 定位到'共享'标签, 勾选连接共享,共享网卡为VMnet8.

![img](Linux%E7%BD%91%E7%BB%9C.assets/dbc22915-d24a-4753-a36b-7f203c37157c.png)

5, 虚拟机系统的网关设置为VMnet8的ip地址: 172.16.100.254

最后重启网卡服务即可.

对于没有共享选项情况,选择nat模式

将虚机网卡设置为net模式

![img](Linux%E7%BD%91%E7%BB%9C.assets/f85e71fb-4eaa-4cb0-91ce-f37dbced4a9d.png)

![img](Linux%E7%BD%91%E7%BB%9C.assets/a2cf17de-025a-4e0a-b43c-a942442a05b1.png)

![img](Linux%E7%BD%91%E7%BB%9C.assets/17969664-4aa9-43f8-8c65-e25e4bc66019.png)

物理机中的VMware8虚拟网卡设置

![img](Linux%E7%BD%91%E7%BB%9C.assets/3dbbff2b-62ff-4d95-84be-9f6a0f9e033f.png)

网络相关命令总结:

ifconfig      #查看所有网络接口的属性 

route -n      #查看路由表 

netstat -tnlp    #查看所有监听端口 

netstat -antp    #查看所有已经建立的连接 

netstat -s     #查看网络统计信息进程 

netstat  -at   #列出所有tcp端口 

netstat  -au   #列出所有udp端口 

netstat -lt     #只列出所有监听tcp端口

ifup      启用网络设备

ifdown     关闭网络设备

lsof      显示指定端口由谁监听

sysctl     控制TCP/IP内核参数

adsl-setup   设置ADSL连接参数

adsl-status  显示ADSL连接状态

adsl-connect  启动ADSL连接

netstat    显示系统网络状态信息

route     查看路由表

ip       强大的网络管理工具

ping      测试连通性

traceroute   路径跟踪

ifcfg

CentOS7: nmcli, nmtui