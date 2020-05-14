[TOC]

# 前言

## 防火墙(Firewall)

就是一个隔离工具,工作于主机或者网络的边缘,对于进出本主机或本网络的报文,根据事先定义的检查规则

做匹配检测,对于能够被规则匹配到的报文作出相应处理的组件,就称之为防火墙,即可称为主机防火墙和网络防火墙.

其主机防火墙工作于主机的边缘,网络防火墙工作于网络的边缘.

从防火墙实现的方法又可以划分为软件防火墙和硬件防火墙. 

其软件防火墙纯软件逻辑实现,而硬件防火墙能实现硬件层实现封包或者解包,但离不开软件的辅助.

主机防火墙:工作在主机边缘,在内核中tcp/ip协议栈上部署规则,对进出的数据报文进行检查;

网络防火墙:工作在一个网络边缘,控制该网络内部多个主机与外部进行数据检查;

目前市场上比较常见的三层,四层的防火墙,叫网络层的防火墙;七层的防火墙,其实就是代理层的网关,从设计上来讲,七层防火墙

更加安全,但是效率更低.所以市场的能用方案为两者结合,而又由于我们都需要从防火墙所控制的这个口来访问,所以防火墙的工

作效率就成了用户能够访问数据多少的一个最重要的控制,配置的不好甚至有可能成为流量的瓶颈.



## iptables的发展

iptables的前者叫ipfirewall(内核1.X时代),这是一个作者从freeBSD移值过来的,能够工作在内核当中,对数据包进行

检测的一款简易访问控制工具,但是ipfirewall工作极其有限(它需要将所有的规则都放进内核中,这样规则才能够运行起来,

而放进内核,这个做法一般是极其困难).当内核发展到2.X系列的时候,软件更名为ipchains,,它可以定义多条规则,将他

们串进来,共同发挥作用,被称之为iptables,可以将规则组成一个列表,实现绝对详细的访问控制功能.



他们都是工作在用户空间中,定义规则的工具本身并不算是防火墙,它们定义的规则,可以让在内核空间中的netfilter来读取,并实现让防火墙工作,而放进的地方必须要是特定的位置,必须是tcp/ip协议栈的地方,而这个tcp/ip协议栈必须经过的地方,可以实现读取规则的功能称之为netfilter(网络过滤器)

## iptables/netfilter

实际上iptables防火墙是包括netfilter和iptables两个部分

netfilter是内核中的防火墙框架

iptables是用户空间中的命令程序

所有这两者加起来才是一个完整的iptables.

作者一共在内核空间中选择了5个位置

\- 从一个网络接口进来,到另一个网络接口去的

\- 数据包从内核流入用户空间的

\- 数据包从用户空间流出的

\- 进入/离开本机的外网接口

\- 进入/离开本机的内网接口

iptables是工作在用户空间中的,它可以编写规则,其是一个rules until工具,通过ipfw(firewall framework)在内

核中生成系统规则(其是通过系统调用完成),再由ipchains工具将规则写入内核空间中,并写入netfilter.



## iptables的工作机制

从上面知道了作者选择的5个位置,来作为控制的地方,其实前三个位置已经基本上能够将路径彻底封锁了,但是为什么已经在进出的口设置了关卡后还要在内部卡呢?由于数据包尚未进行路由决策,还不知道数据要走向那里,所以在进出口是没有办法实现数据过滤的,所以要在内核空间里设置转发的关卡.进入用户空间的关卡,从用户空间出去的关卡,那么,既然他们没什么用,那我们为什么还要设置他们呢?因为我们在做NAT和DNAT的时候,目标地址转换必须在路由之前转换,所以我们在外网而后内网的接口进行设置关卡.

这五个位置也被称为五个钩子函数(hook functions),也叫五个内置规则链,简称五链.

### 五链

PREROUTING(路由前,在对数据包作路由选择前,应用此链中的规则)

INPUT(数据包流入,当收到访问防火墙本机地址的数据包(入站)时,应用此链中的规则)

FORWARD(转发,当收到需要通过防火墙转发给其他地址的数据包(转发)是应用此链中的规则)

OUTPUT(数据包流出,当防火墙本机向外发送数据包(出站)时,应用此链中的规则)

POSTROUTING(路由后,当对数据包做出路由选择后,应用此链中的规则)

### 四表

raw:关闭NAT的连接追踪机制,防止在高并发的访问下服务器的内存溢出,它由PREROUTTING.OUTPUT实现

mangle:可以对匹配到的报文的数据进行拆解,修改后重新封装,它由五个链来实现

nat:网络地址转换,修改源目标或目标ip端口,nat表可以由PREROUTING,OUTPUT,POSTROUTING实现

filter:实现包过滤,是否允许通过防火墙,可以由INPUT,FORWARD,OUTPUT实现

四表的优先级:raw -> mangel -> nat -> filter

注:C7上可以在INPUT上实现nat功能



报文流向经过的链

跟本机内部进程通信：

\- 进入:PREROUTING--->INPUT

\- 流出:OUTPUT--->POSTROUTING



由本机转发：

\- 请求:PREROUTING--->FORWARD--->POSTROUTING

\- 响应:PREROUTING--->FORWARD--->POSTROUTING



路由功能发生的时刻:

当报文进入本机后,判断目标主机是谁?当报文离开本机前,判断经由那个接口送往下一站?



## iptables的链

内置链: 每个对应一个勾子函数(hook function)

自定义链: 用于对内置链的扩展和补充,可实现更灵活的规则管理机制,被内置链关联,必须由内置链调用才会生效.

iptables添加规则时思路:

1,要实现哪种功能,raw mangle filter,判断添加到哪张表上;

2,报文流经过的路径: 判断添加到哪个链上;

3,链上的规则次序,即是检查的次序,因此隐含一定的应用法则:

\- 同类规则(访问同一应用),匹配范围小的放在上面

\- 不同类规则(访问不同应用),匹配到报文频率较大的放在上面

\- 将那此可由一条规则描述的多个规则合并进来

\- 设置默认策略

总结:先确定功能(表),再确定报文的流向,然后确定要实现的目标,最后确定匹配的条件.

iptables命令

语法格式

 

```
iptables [-t tables] COMMAND chain [-m matchname [per-match-options]] -j target [per-target-options]
```

 

```
iptables [-t tables] COMMAND chain [-m matchname [per-match-options]] -j target [per-target-options]
```

COMMAND说明:

管理类的命令操作

链管理

(PREROUTING, INPUT, FORWARD, OUTPUT, POSTROUTING)

-N: 新增加一个自定义的链(自定义被内建链上的规则调用才生效,-j 自定义链名称)

-X: 删除自定义的空链(引用计数为0的空链)

-P: 设置链的默认策略

-E: 重命名自定义的未被引用(引用计数为0)的链=

-F: 清除

规则管理

-A: 追加,默认添加到最后

-I: 插入,默认为第一个

-D: 删除指定的规则

-R: 替换修改,将指定的链规则替换为新的规则         

-Z: 将packets和bytes计算器给重置为0

查看命令

-L: 列出规则列表

-n: 以数字形式显示地址和端口

-v: 显示详细的信息

-x: 显示计数器的精确值而非单位换算后的结果

--line-numbers: 显示链上的规则编号

实例

 

```
iptables -I 2 INPUT -s 192.168.1.1 -d 192.168.1.2 -j DROP # 将此规则插入在input链的第2条
iptables -vnL # 显示详细信息
iptables -vnL --line-number # 显示链上的规则编号
iptables -F # 清空所有链上的规则
iptables -F INPUT # 清空指定INPUT链上的所有规则
iptables -D INPUT 1 # 删除INPUT链上的第一条规则
iptables -R INPUT 2 -s 192.168.22.2 -d 192.168.1.1 -j ACCEPT # 将INPUT链的第2条规则替换
iptables -P INPUT DROP # 设置INPUT链的默认规则为拒绝
iptables -P OUTPUT DROP # 设置OUTPUT链的默认规则为拒绝
iptables -P FORWARD DROP # 设置FORWARD链的默认规则为拒绝
```

 

```
iptables -I 2 INPUT -s 192.168.1.1 -d 192.168.1.2 -j DROP # 将此规则插入在input链的第2条
iptables -vnL # 显示详细信息
iptables -vnL --line-number # 显示链上的规则编号
iptables -F # 清空所有链上的规则
iptables -F INPUT # 清空指定INPUT链上的所有规则
iptables -D INPUT 1 # 删除INPUT链上的第一条规则
iptables -R INPUT 2 -s 192.168.22.2 -d 192.168.1.1 -j ACCEPT # 将INPUT链的第2条规则替换
iptables -P INPUT DROP # 设置INPUT链的默认规则为拒绝
iptables -P OUTPUT DROP # 设置OUTPUT链的默认规则为拒绝
iptables -P FORWARD DROP # 设置FORWARD链的默认规则为拒绝
```

chain说明:

基本匹配条件

无需加载任何模块,由iptables/netfilter自行提供

 

```
# 检查报文中源IP地址, 是否符合此处指定的地址或范围
[!]-s, --source address[/mask][,...]

# 检查报文中的目标IP地址,是否符合此处指定的地址或范围
[!]-d, --destination address[/mask][,...]

# 指定在传输层应用的协议,其可以有tcp,udp,udplite,icmp,icmpv6, esp,ah,sctp,mh 或者all
[!]-p, --protocol protocol

# 只能应用于数据报文流入的接口,INPUT,FORWARD and PREROUTING chains
[!]-i, --in-interface name

# 只能应用于数据报文流出的接口,OUTPUT,FORWARD POSTROUTING
[!]-o, --out-interface name
```

 

```
# 检查报文中源IP地址, 是否符合此处指定的地址或范围
[!]-s, --source address[/mask][,...]
# 检查报文中的目标IP地址,是否符合此处指定的地址或范围
[!]-d, --destination address[/mask][,...]
# 指定在传输层应用的协议,其可以有tcp,udp,udplite,icmp,icmpv6, esp,ah,sctp,mh 或者all
[!]-p, --protocol protocol
# 只能应用于数据报文流入的接口,INPUT,FORWARD and PREROUTING chains
[!]-i, --in-interface name
# 只能应用于数据报文流出的接口,OUTPUT,FORWARD POSTROUTING
[!]-o, --out-interface name
```

隐匿扩展

不需要手动加载扩展模块,因为它们是对协议的扩展,所以但凡使用-p指明了协议,就表示已经指明了要扩展的模块

\- tcp

 

```
# 匹配报文的源端口,可以是端口范围
[!]--source-port, --sport port[:port]

# 匹配报文的目标端口,可以是端口范围
[!]--destination-port, --dport port[:port]

# mask:必须要检查的标识位,必须以,号分隔; comp:必须为1的标识位,必须以逗号','分隔
[!]--tcp-flags mask comp

# 例如:表示要检查的标识位为syn,ack,fin,rst,但syn必须为1,余下的必须为0,其是匹配每一次握手
--tcp-flags syn,ack,fin,rst syn
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
iptables -A INPUT -p tcp !--syn -m state --state NEW -j DROP # 不是TCP第一次握手,但状态又是new的报文

# 匹配第一次握手,相当于 —tcp-flags syn,ack,fin,rst syn
--syn
```

 

```
# 匹配报文的源端口,可以是端口范围
[!]--source-port, --sport port[:port]
# 匹配报文的目标端口,可以是端口范围
[!]--destination-port, --dport port[:port]
# mask:必须要检查的标识位,必须以,号分隔; comp:必须为1的标识位,必须以逗号','分隔
[!]--tcp-flags mask comp
# 例如:表示要检查的标识位为syn,ack,fin,rst,但syn必须为1,余下的必须为0,其是匹配每一次握手
--tcp-flags syn,ack,fin,rst syn
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
iptables -A INPUT -p tcp !--syn -m state --state NEW -j DROP # 不是TCP第一次握手,但状态又是new的报文
# 匹配第一次握手,相当于 —tcp-flags syn,ack,fin,rst syn
--syn
```

\- udp

 

```
# 匹配报文的源端口,可以是端口范围
[!] --source-port --sport port[:port]

# 匹配报文的目标端口,可以是端口范围
[!] --destination-port --dport port[:port]
```

 

```
# 匹配报文的源端口,可以是端口范围
[!] --source-port --sport port[:port]
# 匹配报文的目标端口,可以是端口范围
[!] --destination-port --dport port[:port]
```

\- icmp

 

```
[!] --icmp {type[/code] | typename}
echo-request === 8
echo-replay === 0
"""
ping别人: 出去==8 , 进来==0
别人ping我: 出去==0 , 进来==8
"""
```

 

```
[!] --icmp {type[/code] | typename}
echo-request === 8
echo-replay === 0
"""
ping别人: 出去==8 , 进来==0
别人ping我: 出去==0 , 进来==8
"""
```

![img](iptables.assets/98dbd7d7-ceaf-4470-a3fb-8f4658b48076.jpg)

显示扩展

必须使用-m选项手动加载模块,其扩展模块路径为:/lib64/xtables,其中大写的为目标扩展,小写的为规则扩展

获取扩展选项帮助

Centos 6 : man iptables

Centos 7 : man iptables-extensions

multiport扩展

以离散方式定义多端口匹配,但最多指定15个端口

 

```
# 指定多个源端口
[!] --source-ports, --sports port[,port | ,port:port]

# 指定多个目标端口
[!] --destination-ports, --dports port[,port | ,port:port]

# 指定多个目标及源端口
--ports port[,port | ,port:port]

# 实例
iptables -A INPUT -s 172.16.0.0/16 -d 172.16.100.67 -p tcp -m multiport --dports 22,80 -j ACCEPT
```

 

```
# 指定多个源端口
[!] --source-ports, --sports port[,port | ,port:port]
# 指定多个目标端口
[!] --destination-ports, --dports port[,port | ,port:port]
# 指定多个目标及源端口
--ports port[,port | ,port:port]
# 实例
iptables -A INPUT -s 172.16.0.0/16 -d 172.16.100.67 -p tcp -m multiport --dports 22,80 -j ACCEPT
```

iprange扩展

指明连续的(一般不能整个网络)ip地址范围

 

```
# 源IP地址
[!] --src-range from[-to]

# 目标IP地址
[!] --dst-range from[-to]

# 实例
iptables -A INPUT -d 172.16.100.67 -p tcp --dport 80 -m iprange --src-range 172.16.1.5-172.16.1.10 -j DROP

iptables -A INPUT -s 172.16.36.71 -p tcp -m multiport --dport 53,80 -m iprange --dst-range 172.16.1.70-172.16.1.75 -j DROP
```

 

```
# 源IP地址
[!] --src-range from[-to]
# 目标IP地址
[!] --dst-range from[-to]
# 实例
iptables -A INPUT -d 172.16.100.67 -p tcp --dport 80 -m iprange --src-range 172.16.1.5-172.16.1.10 -j DROP
iptables -A INPUT -s 172.16.36.71 -p tcp -m multiport --dport 53,80 -m iprange --dst-range 172.16.1.70-172.16.1.75 -j DROP
```

string 扩展

对报文中的应用层数据,做字符串模式匹配检测

--alog {bm | kmp} : 字符串匹配检测算法,bm和kmp是2中字符匹配检查的高效算法

[!] --string pattern : 要检测的字符串模式

[!] --hex-string pattern : 要检测的字符串模式,16进制格式

实例

 

```
iptables -A OUTPUT -s 172.16.36.61 -p tcp --sport 80 -m string --algo bm --string "gay" -j REJECT
```

 

```
iptables -A OUTPUT -s 172.16.36.61 -p tcp --sport 80 -m string --algo bm --string "gay" -j REJECT
```

time 扩展

根据报文到达时间与指定的时间范围进行匹配

--datestart YYYY[-MM-DD[Thh[:mm[:ss]]]]

--datestop YYYY[-MM-DD[Thh[:mm[:ss]]]

--timestart hh:mm[:ss]

--timestop hh:mm[:ss]

[!]--monthdays day,[day….]

[!]--weekdays day[,day…]

--kerneltz : 使用内核的时间,而非默认的UTC时区(Centos 7)

示例:

 

```
iptables -A INPUT -s 172.16.0.0/16 -d 172.168.100.67 -p tcp --dport 80 -m time --timestart 14:30 --time-stop 18:30 --weekdays sat,sun -j DROP
```

 

```
iptables -A INPUT -s 172.16.0.0/16 -d 172.168.100.67 -p tcp --dport 80 -m time --timestart 14:30 --time-stop 18:30 --weekdays sat,sun -j DROP
```

说明:每周的星期几可以使用数据表示方法: 1,2,3,4,5,6,7 可以使用离散取值方法

connlimit 扩展

对每客户端IP做并发连接数量匹配

--connlimit-upto n : 当现在的连接数量低于或等于这个数量(n),就匹配

--connlimit-above n : 当现有的连接数量大于这个数量, 就匹配

示例:

 

```
iptables -A INPUT -d 172.16.36.61 -p tcp --dport 22 -m connlimit --connlimit-above 2 -j REJECT
```

 

```
iptables -A INPUT -d 172.16.36.61 -p tcp --dport 22 -m connlimit --connlimit-above 2 -j REJECT
```

limit扩展

基于收发报文的速率做匹配

--limit rate [/second | /minute | /hour] : 平均速率

--limit-burst NUMBER : 峰值数量,默认5个

示例:

 

```
iptables -A INPUT -d 172.16.100.67 -p icmp --icmp-type 8 -m limit --limit 3/minute --limit-burst 5 -j ACCEPT
```

 

```
iptables -A INPUT -d 172.16.100.67 -p icmp --icmp-type 8 -m limit --limit 3/minute --limit-burst 5 -j ACCEPT
```

state扩展

根据连接追踪机制,查检连接的状态,跟TCP没有关系,是内核中netfilter实现, 能实现tcp,udp,icmp的连接追踪,内核会记录每一个连接(放置在内存中),谁,通过什么协议,访问什么服务,访问的时间,这种机制被称之为conntrack机制.也正是有了state扩展,iptables成为了有连接追踪的防火墙,安全性是更高.

连接追踪功能是由state扩展提供,库文件为ibxt_conntrack.so.

追踪连接功能在内核的内存空间中,把出去和进来的连接通过模板建立关联关系. 追踪本机的请求和响应之间的关系,状态如下几种:

\- NEW:新发起的请求

\- ESTABLISHED:new状态之后,连接追踪模板中为其建立的条目失效之前期间内所有的通信状态

\- RELATED:相关的连接,如FTP协议中的命令连接与数据连接之间的关系

\- INVALID:无效的连接,如tcp状态全为1或者全为0的连接

\- UNTRACKED:未进行追踪的连接

调整连接追踪功能所容纳的最大连接数量:

/proc/sys/net/nf_conntrack_max

已经追踪到的并记录下来的连接:

/porc/net/nf_conntrack

不同协议的连接追踪状态时长(可修改)

/proc/sys/net/netfilter

[!] --state STATE: 多个state可以使用,号分隔

示例:

 

```
iptables -A INPUT -d 172.168.100.67 -p tcp -m multiport --dport 22,80 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -A OUTPUT -s 172.16.100.67 -p tcp -m multiport --sport 22,80 -m state --state ESTABLISHED -j ACCEPT

iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
```

 

```
iptables -A INPUT -d 172.168.100.67 -p tcp -m multiport --dport 22,80 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -s 172.16.100.67 -p tcp -m multiport --sport 22,80 -m state --state ESTABLISHED -j ACCEPT
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
```

连接追踪超时问题

iptables的连接追踪表最大容量为/proc/sys/ipv4/ip_conntrack_max

链接达到各种状态的超时后,会从表中删除,当模板满载时,后续的链接可能会超时,可以有如下两种解决方法(但加大max值,也会加大内存的压力)

 

```
# 1,修改max的内核参数
vim /etc/sysctl.conf
"""
net.ipv4.nf_conntrack_max = 393216
net.ipv4.netfilter.nf_conntrack_max = 393216
"""

# 2,降低nf_conntrack timout时间
vim /etc/sysctl.conf
"""
net.ipv4.netfilter.nf_conntrack_tcp_timeout_established = 300
net.ipv4.netfilter.nf_conntrack_tcp_timeout_time_wait = 120
net.ipv4.netfilter.nf_conntrack_tcp_timeout_close_wait = 60
net.ipv4.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120
"""
```

 

```
# 1,修改max的内核参数
vim /etc/sysctl.conf
"""
net.ipv4.nf_conntrack_max = 393216
net.ipv4.netfilter.nf_conntrack_max = 393216
"""
# 2,降低nf_conntrack timout时间
vim /etc/sysctl.conf
"""
net.ipv4.netfilter.nf_conntrack_tcp_timeout_established = 300
net.ipv4.netfilter.nf_conntrack_tcp_timeout_time_wait = 120
net.ipv4.netfilter.nf_conntrack_tcp_timeout_close_wait = 60
net.ipv4.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120
"""
```

target说明:

target的分类:

\- ACCEPT:接受

\- DROP:丢弃

\- REJECT:拒绝

\- RETURN:返回调用链

\- REDIRECT:端口重定向

\- MASK : 做防火墙标记

\- DNAT : 目标地址转换

\- SNAT : 源地址转换

\- MASQUERADE : 地址伪装

\- LOG:记录日志

--log-level LEVEL : 日志的等级

--log-prefix FREFIX : 日志的提示语句的前缀

 

```
iptables -A INPUT -d 172.16.36.61 -p tcp --dport 21 -j LOG --log-prefix "netfilter log"
```

 

```
iptables -A INPUT -d 172.16.36.61 -p tcp --dport 21 -j LOG --log-prefix "netfilter log"
```

\- 用户自定义链

自定义链的使用方法

 

```
iptables -N in_icmp
iptables -A INPUT -j in_icmp
iptables -A in_icmp -j RETURN
```

 

```
iptables -N in_icmp
iptables -A INPUT -j in_icmp
iptables -A in_icmp -j RETURN
```

总结:

优化规则：

  尽量减少规则条目

  修改规则时，先添加后改删

  匹配几率越大，位置越靠前

iptables的配置及服务管理

配置管理

service iptables save:将规则保存至/etc/sysconfig/iptables文件中,默认保存路径

iptables-save > /path/to/some_rules_file(适用于Centos 7)

iptabes-restore < /etc/sysconfig/iptables(适用于Centos 7)

说明:用规则文件保存各规则,至/etc/rc.d/rc.local文件,开机可自动加载

服务管理

serivce iptables {start|stop|restart|status}

start:读取事先保存的规则,并应用于netfilter上

stop:清空netfilter的规则,以及还原默认策略

status:显示生效的规则

restart:先清空netfilter的规则,再读取规则

C7:

systemctl {start|stop|disable} firewalld.service