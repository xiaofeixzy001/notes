[TOC]

# 回顾

iptables

-P	设置默认策略

-F	清空规则链

-L	查看规则链

-A	在规则链的末尾加入新规则

-I num	在规则链的头部加入新规则

-D num	删除某一条规则

-s	匹配来源地址IP/MASK，加叹号“!”表示除这个IP外

-d	匹配目标地址

-i 网卡名称	匹配从这块网卡流入的数据

-o 网卡名称	匹配从这块网卡流出的数据

-p	匹配协议，如TCP、UDP、ICMP

--dport num	匹配目标端口号

--sport num	匹配来源端口号


# NAT功能

NAT(Network Adress Tanslate)

其可以分为SANT,DNAT,PNAT.

NAT的诞生是为了解决内网主机的安全,但后期又应用于IPV4地址不够用,发挥了很大的作用.

## SNAT

客户端访问公网时,才需做源地址转换

发生的位置为:POSTROUTING或者OUTPUT

## DNAT

本地网络中的某一主机上的某服务开放给公网的用户访问时

发生的位置为:PREROUTING,也可以完成端口映射

## PNAT

发布服务的端口重定向

例如80 -> 8080

NAT表的target

\- SNAT

--to-source [ipaddr[-ipaddr]][:port[-port]]

--random : 随机选一个源地址

示例:

```
iptables -t nat  POSTROUTING -s 172.16.0.0/24 -j SNAT --to-source 172.16.100.67
```



\- MASQUERADE

当源地址转换时,当地址为动态获取时,MASQUERADE可自行判断要转换的地址(其有额外开销,如果是静态地址,建议不要使用)

示例:

```
iptables -t nat -A POSTROUTING -s 172.16.0.0/24 -j MASQUERADE
```



\- DNAT

--to-destination [ipaddr[-ipaddr]][:port[-port]]

示例:

```
iptables -t nat -A PREROUTING -d 172.16.0.1 -p tcp --dport 80 -j DNAT --to-destination 192.168.0.1:80
```

iptables常用选项

iptables -L -n -v:查看规则

iptables -t nat -L -n:查看nat规则

iptables -F:清除所有规则

iptables -t nat -F PREROUTING:清除nat的PREROUTING规则

iptables -Z:清除所有的计数器

iptables -Z INPUT 1:清除指定规则的计数器

watch -n1 'iptables -L -n -v':动态监控iptables的数据信息

\# 抓取eth1接口tcp协议的80端口并且与172.16.36.61主机相关的报文

tcpdump -i eth1 tcp port 80 and host 172.16.36.61 

补充：利用iptables的recent模块来抵御DOS攻击: 22,建立一个列表,保存有所有访问过指定的服务的客户端IP

ssh: 远程连接

```
iptables -I INPUT -p tcp --dport 22 -m connlimit --connlimit-above 3 -j DROP
iptables -I INPUT -p tcp --dport 22 -m state --state NEW -m recent --set --name SSH
iptables -I INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 300 --hitcount 3 --name SSH -j LOG --log-prefix "SSH Attach: "
iptables -I INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 300 --hitcount 3 --name SSH -j DROP
```

1.利用connlimit模块将单IP的并发设置为3；会误杀使用NAT上网的用户,可以根据实际情况增大该值.

2.利用recent和state模块限制单IP在300s内只能与本机建立2个新连接.被限制五分钟后即可恢复访问.

下面对最后两句做一个说明：

1.第二句是记录访问tcp 22端口的新连接,记录名称为SSH

--set 记录数据包的来源IP, 如果IP已经存在将更新已经存在的条目

2.第三句是指SSH记录中的IP, 300s内发起超过3次连接则拒绝此IP的连接

--update:是指每次建立连接都更新列表

--seconds:必须与--rcheck或者--update同时使用

--hitcount:必须与--rcheck或者--update同时使用

3.iptables的记录:/proc/net/xt_recent/SSH

也可以使用下面的这句记录日志：

iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --name SSH --second 300 --hitcount 3 -j LOG --log-prefix "SSH Attack


layer 7 

让iptables可基于第7层应用层来做过滤,比如p2p,迅雷,QQ,等

l7filter

\- 对内核中的netfilter打补丁layer7,重新编译内核

\- 对iptables打补丁,补上layer7模块,重新编译iptables

最高支持2.6.32内核版本

rhel source rpm

kernel-2.6.32-431.11.2.el6.src.rpm

netfilter-layer7-v2.23.tar.bz2

l7-protocols-2009-05-28.tar.gz

iptables-1.4.20.tar.bz2

方法步骤:

```
# 1,获取并编译内核
useradd mockbuild
rpm -ivh kernel-2.6.32-431.5.1.x86_64.el6.src.rpm
cd rpmbuild/SOURCES
tar linux-2.6.32-*.tar.gz -C /usr/src
cd /usr/src
ln -sv 

# 2,给内核打补丁
tar xf netfilter-layer7-v2.23.tar.bz2
cd /usr/src/linux
patch -p1 < /root/netfilter-layer7-v2.23/kernel-2.6.32-layer7-2.23.patch
cp /boot/config-*  .config
make menuconfig

"""
按如下步骤启用layer7模块 
Networking support → Networking Options →Network packet filtering framework → Core Netfilter Configuration
<M>  “layer7” match support
"""

# 3,编译并安装内核
make
make modules_install
 make install

# 4,重启系统，启用新内核

# 5,编译iptables
tar xf iptables-1.4.20.tar.gz
cp /root/netfilter-layer7-v2.23/iptables-1.4.3forward-for-kernel-2.6.20forward/* /root/iptables-1.4.20/extensions/
cp /etc/rc.d/init.d/iptales /root
cp /etc/sysconfig/iptables-config /root
rpm -e iptables iptables-ipv6 --nodeps
./configure  --prefix=/usr  --with-ksource=/usr/src/linux
make && make install

cp /root/iptables /etc/rc.d/init.d
cp /root/iptables-config /etc/sysconfig

# 6,为layer7模块提供其所识别的协议的特征码
tar zxvf l7-protocols-2009-05-28.tar.gz
cd l7-protocols-2009-05-28
make install
```



7,如何使用layer7模块

​		ACCT的功能已经可以在内核参数中按需启用或禁用,此参数需要装载 nf_conntrack 模块后方能生效

​		net.netfilter.nf_conntrack_acct = 1

​		l7-filter uses the standard iptables extension syntax 

```
iptables [specify table & chain] -m layer7 --l7proto [protocol name] -j [action] 
iptables -A FORWARD -m layer7 --l7proto qq -j REJECT
```



编译内核

```
make menuconfig
make -j #
make modules_install
make install
```

清理内核源码树：

提示:xt_layer7.ko依赖于nf_conntrack.ko模块

博客:iptables所有应用,包括layer7的实现

实例

1,允许192.168.1.0网段的访问本机192.168.1.1的ssh服务,其余全部拒绝

```
iptables -A INPUT -s 192.168.1.0/24 -d 192.168.1.1 -p tcp --dport 22 -j ACCEPT
# or
iptables -A OUTPUT -d 192.68.1.0/24 -s 192.168.1.1 -p tcp --sport 22 -j ACCEPT

iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
service iptables save
```



2,仅允许192.168.1.1主机ping别人,不允许别人ping该主机

```
iptables -A INPUT -s 0/0 -d 192.168.1.1 -p icmp --icmp-type 0 -j ACCEPT
# or
iptables -A OUTPUT -d 0/0 -s 192.168.1.1 -p icmp --icmp-type 8 -j ACCEPT
```



3,允许192.168.1.1主机ping别人,也允许别人ping此主机

```
iptables -A INPUT -s 0/0 -d 192.168.1.1 -p icmp --icmp-type 0 -j ACCEPT
iptables -A OUTPUT -d 0/0 -s 192.168.1.1 -p icmp --icmp-type 8 -j ACCEPT
# or
iptables -A INPUT -s 0/0 -d 192.168.1.1 -p icmp --icmp-type 8 -j ACCEPT
iptables -A OUTPUT -d 0/0 -s 192.168.1.1 -p icmp --icmp-type 0 -j ACCEPT
```



4,允许所有人访问本机ssh和web服务

```
iptables -I INPUT -s 0/0 -d 192.168.1.1 -p tcp -m multiport --dports 22,23,80 -j ACCEPT
iptables -I OUTPUT -d 0/0 -s 192.168.1.1 -p tcp -m multiport --sports 22,23,80 -j ACCEPT
```



5,仅允许192.168.1.100-192.168.1.200范围内主机访问本机mysql服务

```
iptables -A INPUT -s 0/0 -d 192.168.1.1 -p tcp --dport 3306 -m iprange --src-range 192.168.1.100-192.168.1.200 -j ACCEPT
iptables -A OUTPUT -d 0/0 -s 192.168.1.1 -p tcp --sport 3306 -m iprange --dst-range 192.168.1.100-192.168.1.200 -j ACCEPT
```



6,允许所有人访问192.168.1.1的web服务,如果请求的内容包含admin敏感字符时拒绝掉.

```
iptables -I OUTPUT -s 192.168.1.1 -d 0/0 -p tcp --sport 80 -m string --algo bm --string "admin" -j REJECT
```



7,周一到周五,从8:30到18:30,禁止访问本机web服务

```
iptables -I INPUT -d 192.168.1.1 -p tcp --dport 80 -m time --timestart 08:30 --timestop 18:30 --weekdays 1,2,3,4,5 -j REJECT
```



8,仅允许5个客户端连接本机的ssh服务

```
iptables -A INPUT -s 0/0 -d 192.168.1.1 -p tcp --dport 22 --syn -m connlimit --connlimit-above 5 -j REJECT
```



9,客户端ping本机,限定每分钟接收ping包的个数为20个,最多一次接收5个

```
iptables -A INPUT -d 192.168.1.1 -p icmp --icmp-type 8 -m limit --limit 20/minute --limit-burst 5 -j ACCEPT
iptables -A INPUT -s 192.168.1.1 -p icmp --icmp-type 0 -j ACCEPT
```



10,访问本机ssh和we服务的连接,其连接状态只要是NEW,ESTABLISHD都放行,从本机出去的连接,状态为ESTABLISHED的放行,其他拒绝

```
iptables - I INPUT -d 192.168.1.1 -p tcp -m multiport --dports 22,80 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -I OUTPUT -m state --state ESTABLISHED -j ACCEPT
```



11,放行被动连接模式下的vsftp服务

```
# 装载模块
modporbe nf_conntrack_ftp
lsmod | less

# 放行连接状态为NEW,目标端口为21,22的请求报文
iptables -A INPUT -d 192.168.1.1 -p tcp -m multimport --dports 21,22 -m state --state NEW -j ACCEPT

# 放行连接状态为ESTABLISHED和RALATER的请求报文
iptables -A INPUT -d 192.168.1.1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# 放行连接状态为ESTABLISHED,RELATER的响应报文
iptables -A OUTPUT -s 192.168.1.1 -m state --state ESTABLISHED,RELATER -j ACCEPT

```



12,新建一个自定义链http_in,禁止192.168.1.100-192.168.1.200访问本机web服务,其他全部允许

```
iptables -t filter -N http_in
iptables -A http_in -d 192.168.100.1 -p tcp --dport 80 -m iprange --src-range 192.168.100.100-192.168.100.200 -j DROP
iptables -A http_in -d 192.168.100.1 -p tcp --dport 80 -m state --state NEW -j ACCEPT
iptables -A http_in -j RETURN # 如果自定义链中的规则无法匹配，则返回主链
iptables -A INPUT -d 192.168.1.1 -p tcp --dport 80 -j http_in
curl -I http://192.168.1.1
```



13,将172.16.100.67的80端口映射为8080

```
iptables -t nat -A PREROUTING -d 172.16.100.67 -p tcp --dport 80 -j REDIRECT --to-ports 8080
```



14,将内网的地址192.16.1.0/24使用转换为统一的地址172.16.100.67与外部地址通信

```
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j SNAT --to-source 172.16.100.6
```



15,将本地网络总的地址使用统一的地址向外发布服务,即外部访问内网服务器,把目标地址转换成内网的服务器地址,

外网地址任意

这里假设nat服务器外网接口地址172.16.100.1,web服务器192.168.1.1:8080

```
iptables -t nat -A PREROUTING -d 172.16.100.1 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.1:8080
```



16,一台主机,adsl拨号上网,内网网段172.16.0.0/24,限制周一到周五早9点到晚6点禁止上网

```
iptables -t nat -A POSTROUTING -s 172.16.0.0/24 -j MASQUERADE 
iptables -t filter -A FORWARD -m time --timestart 09:00 --timestop 18:00 --weekdays 1,2,3,4,5 -j REJECT
```



INPUT和OUTPUT默认策略为DROP

17,限制本地主机的web服务器在周一不允许访问

新请求的速率不能超过100个每秒

web服务器包含了admin字符串的页面不允许访问

web服务器仅允许响应报文离开本机

 

```
iptables -P OUTPUT DORP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
iptables -A INPUT  -d 172.16.36.61 -p tcp --dport 80 -m time --weekdays 2,3,4,5,6,7 -m limit --limit 100/second  -j ACCEPT
iptables -A OUTPUT  -s 172.16.36.61 -p tcp --sport 80 -m string --algo bm --string "admin" -j DROP
#  (string的关键字过滤,一定要做在output链上, 在回应报文中才应该会有内容)
iptables -A OUTPUT -m state --state ESTABLISHED -j ACCEPT
```



18,在工作时间,即周一到周五的8:30-18:00,开放本机的ftp服务给172.16.0.0网络中的主机访问,数据下载请求的次数每分钟不得超过5个.

```
modprobe nf_conntrack_ftp (需要使用related的连接状态,需要挂载此模块)
iptables -P OUTPUT DORP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
iptables -A INPUT -s 172.16.0.0/16 -p tcp --dport 21 -m time --weekdays 1,2,3,4,5 --timestart 8:30 --timestop 18:00 -m limit --limit 5/minute -m state --state NEW -j ACCEPT
iptables -A INPUT -s 172.16.0.0/16 -p tcp -m state --state RELATED -j ACCEPT
iptables -A OUTPUT -m state --state ESTABLISHED -j ACCEPT
```



19,开放本机的ssh服务给172.16.x.1-172.16.x.100中的主机,x为你的学号,新请求建立的速率一分钟不得超过2个;仅允许响应报文通过其服务端口离开本机.

```
iptables -P OUTPUT DORP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
iptables -A INPUT -m state --state ESTABLISHED,NEW -j ACCEPT
iptables -A INPUT  -d 172.16.36.61 -p tcp --dport 22 -m iprange --src-range 172.16.36.1-172.16.36.100 -m connlimit --connlimit-above 2/minute -j DROP
iptables -A OUTPUT -m state --state ESTABLISHED -j ACCEPT
```



20,拒绝TCP标志位全部为1及全部为0的报文访问本机

 

```
iptables -P OUTPUT DORP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
```



21,允许本机ping别的主机,但不开放别的主机ping本机

 

```
iptables -A OUTPUT -p icmp --icmp-type 8 -j ACCEPT
iptables -A INPUT -p icmp --icmp-type 0 -j ACCEPT
```



说明:如果INPUT上放行了ESTABLISHED,就不需要再放行代码为0的

总结:

1,查看规则

iptables -L

2,清空规则

iptables -F

iptables -L

3,设置INPUT链默认策略为拒绝

iptables -P DROP

iptables -L

4,向INPUT链中添加允许ICMP进入规则

iptables -I INPUT -p icmp -j ACCEPT

iptables -L

5,删除INPUT链中允许ICMP进入规则,并将INPUT默认规则改为ACCEPT

iptables -L

iptables -D INPUT 1

iptables -P INPUT ACCEPT

iptables -L

6,将INPUT链规则设置为仅允许172.16.100.0网段的主机访问本机ssh端口,其他拒绝

iptables -I INPUT -s 172.16.100.0/24 -p tcp --dport 22 -j ACCEPT

iptables -A INPUT -p tcp --dport 22 -j REJECT

iptables -L

7,向INPUT链中添加:拒绝所有人访问本机的1234端口

iptables -I INPUT -p tcp --dport 1234 -j REJECT

iptables -I INPUT -p udp --dport 1234 -j REJECT

iptables -L

8,向INPUT链中添加:拒绝172.16.100.20主机访问本机80端口

iptables -I INPUT -p tcp -s 172.16.100.20 --dport 80 -j REJECT

iptables -L

9,向INPUT链中添加拒绝所有主机访问本机1000-1024端口

iptables -A INPUT -p tcp --dport 1000:1024 -j REJECT

iptables -A INPUT -p udp --dport 1000:1023 -j REJECT

iptables -L

C7系统,iptables的CLI版本

firewalld-cmd

--get-default-zone	查询默认的区域名称

--set-default-zone=<区域名称>	设置默认的区域，使其永久生效

--get-zones	显示可用的区域

--get-services	显示预先定义的服务

--get-active-zones	显示当前正在使用的区域与网卡名称

--remove-source=	将源自此IP或子网的流量导向指定的区域

--remove-source=	不再将源自此IP或子网的流量导向某个指定区域

--add-interface=<网卡名称>	将源自该网卡的所有流量都导向某个指定区域

--change-interface=<网卡名称>	将某个网卡与区域进行关联

--list-all	显示当前区域的网卡配置参数、资源、端口以及服务等信息

--list-all-zones	显示所有区域的网卡配置参数、资源、端口以及服务等信息

--add-service=<服务名>	设置默认区域允许该服务的流量

--add-port=<端口号/协议>	设置默认区域允许该端口的流量

--remove-service=<服务名>	设置默认区域不再允许该服务的流量

--remove-port=<端口号/协议>	设置默认区域不再允许该端口的流量

--reload	让“永久生效”的配置规则立即生效，并覆盖当前的配置规则

--panic-on	开启应急状况模式

--panic-off	关闭应急状况模式

默认Runtime模式,配置临时生效,重启失效

--Permanent模式,永久生效,需重启

firewall-cmd --reload 立即生效

实例1

1,允许192.168.1.0网段的访问本机192.168.1.1的ssh服务,其余全部拒绝

 

```
iptables -A INPUT -s 192.168.1.0/24 -d 192.168.1.1 -p tcp --dport 22 -j ACCEPT
# or
iptables -A OUTPUT -d 192.68.1.0/24 -s 192.168.1.1 -p tcp --sport 22 -j ACCEPT

iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
service iptables save
```





2,仅允许192.168.1.1主机ping别人,不允许别人ping该主机

 

```
iptables -A INPUT -s 0/0 -d 192.168.1.1 -p icmp --icmp-type 0 -j ACCEPT
# or
iptables -A OUTPUT -d 0/0 -s 192.168.1.1 -p icmp --icmp-type 8 -j ACCEPT
```





3,允许192.168.1.1主机ping别人,也允许别人ping此主机

 

```
iptables -A INPUT -s 0/0 -d 192.168.1.1 -p icmp --icmp-type 0 -j ACCEPT
iptables -A OUTPUT -d 0/0 -s 192.168.1.1 -p icmp --icmp-type 8 -j ACCEPT
# or
iptables -A INPUT -s 0/0 -d 192.168.1.1 -p icmp --icmp-type 8 -j ACCEPT
iptables -A OUTPUT -d 0/0 -s 192.168.1.1 -p icmp --icmp-type 0 -j ACCEPT
```





4,允许所有人访问本机ssh和web服务

 

```
iptables -I INPUT -s 0/0 -d 192.168.1.1 -p tcp -m multiport --dports 22,23,80 -j ACCEPT
iptables -I OUTPUT -d 0/0 -s 192.168.1.1 -p tcp -m multiport --sports 22,23,80 -j ACCEPT
```





5,仅允许192.168.1.100-192.168.1.200范围内主机访问本机mysql服务

 

```
iptables -A INPUT -s 0/0 -d 192.168.1.1 -p tcp --dport 3306 -m iprange --src-range 192.168.1.100-192.168.1.200 -j ACCEPT
iptables -A OUTPUT -d 0/0 -s 192.168.1.1 -p tcp --sport 3306 -m iprange --dst-range 192.168.1.100-192.168.1.200 -j ACCEPT
```





6,允许所有人访问192.168.1.1的web服务,如果请求的内容包含admin敏感字符时拒绝掉.

 

```
iptables -I OUTPUT -s 192.168.1.1 -d 0/0 -p tcp --sport 80 -m string --algo bm --string "admin" -j REJECT
```





7,周一到周五,从8:30到18:30,禁止访问本机web服务

 

```
iptables -I INPUT -d 192.168.1.1 -p tcp --dport 80 -m time --timestart 08:30 --timestop 18:30 --weekdays 1,2,3,4,5 -j REJECt
```





8,仅允许5个客户端连接本机的ssh服务

 

```
iptables -A INPUT -s 0/0 -d 192.168.1.1 -p tcp --dport 22 --syn -m connlimit --connlimit-above 5 -j REJECT
```





9,客户端ping本机,限定每分钟接收ping包的个数为20个,最多一次接收5个

 

```
iptables -A INPUT -d 192.168.1.1 -p icmp --icmp-type 8 -m limit --limit 20/minute --limit-burst 5 -j ACCEPT
iptables -A INPUT -s 192.168.1.1 -p icmp --icmp-type 0 -j ACCEPT
```





10,访问本机ssh和web服务的连接,其连接状态只要是NEW,ESTABLISHD都放行,从本机出去的连接,状态为ESTABLISHED的放行,其他拒绝

 

```
iptables - I INPUT -d 192.168.1.1 -p tcp -m multiport --dports 22,80 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -I OUTPUT -m state --state ESTABLISHED -j ACCEPT
```





11,放行被动连接模式下的vsftp服务

 

```
# 装载模块
modporbe nf_conntrack_ftp
lsmod | less

# 放行连接状态为NEW,目标端口为21,22的请求报文
iptables -A INPUT -d 192.168.1.1 -p tcp -m multimport --dports 21,22 -m state --state NEW -j ACCEPT

# 放行连接状态为ESTABLISHED和RALATER的请求报文
iptables -A INPUT -d 192.168.1.1 -m state --state RELATED,ESTABLISHED -j ACCEPT

# 放行连接状态为ESTABLISHED,RELATER的响应报文
iptables -A OUTPUT -s 192.168.1.1 -m state --state ESTABLISHED,RELATER -j ACCEPT

```





12,新建一个自定义链http_in,禁止192.168.1.100-192.168.1.200访问本机web服务,其他全部允许

 

```
iptables -t filter -N http_in
iptables -A http_in -d 192.168.100.1 -p tcp --dport 80 -m iprange --src-range 192.168.100.100-192.168.100.200 -j DROP
iptables -A http_in -d 192.168.100.1 -p tcp --dport 80 -m state --state NEW -j ACCEPT
iptables -A http_in -j RETURN # 如果自定义链中的规则无法匹配，则返回主链
iptables -A INPUT -d 192.168.1.1 -p tcp --dport 80 -j http_in
curl -I http://192.168.1.1
```





13,将172.16.100.67的80端口映射为8080

 

```
iptables -t nat -A PREROUTING -d 172.16.100.67 -p tcp --dport 80 -j REDIRECT --to-ports 8080
```





14,将内网的地址192.168.1.0/24使用转换为统一的地址172.16.100.67与外部地址通信

 

```
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -j SNAT --to-source 172.16.100.6
```





15,将本地网络总的地址使用统一的地址向外发布服务,即外部访问内网服务器,把目标地址转换成内网的服务器地址,

外网地址任意

这里假设nat服务器外网接口地址172.16.100.1,web服务器192.168.1.1:8080

 

```
iptables -t nat -A PREROUTING -d 172.16.100.1 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.1:8080
```





16,一台主机,adsl拨号上网,内网网段172.16.0.0/24,限制周一到周五早9点到晚6点禁止上网

 

```
iptables -t nat -A POSTROUTING -s 172.16.0.0/24 -j MASQUERADE 
iptables -t filter -A FORWARD -m time --timestart 09:00 --timestop 18:00 --weekdays 1,2,3,4,5 -j REJECT
```





INPUT和OUTPUT默认策略为DROP

17,限制本地主机的web服务器在周一不允许访问

新请求的速率不能超过100个每秒

web服务器包含了admin字符串的页面不允许访问

web服务器仅允许响应报文离开本机

 

```
iptables -P OUTPUT DORP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
iptables -A INPUT  -d 172.16.36.61 -p tcp --dport 80 -m time --weekdays 2,3,4,5,6,7 -m limit --limit 100/second  -j ACCEPT
iptables -A OUTPUT  -s 172.16.36.61 -p tcp --sport 80 -m string --algo bm --string "admin" -j DROP
#  (string的关键字过滤,一定要做在output链上, 在回应报文中才应该会有内容)
iptables -A OUTPUT -m state --state ESTABLISHED -j ACCEPT
```





18,在工作时间,即周一到周五的8:30-18:00,开放本机的ftp服务给172.16.0.0网络中的主机访问,数据下载请求的次数每分钟不得超过5个.

 

```
modprobe nf_conntrack_ftp (需要使用related的连接状态,需要挂载此模块)
iptables -P OUTPUT DORP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
iptables -A INPUT -s 172.16.0.0/16 -p tcp --dport 21 -m time --weekdays 1,2,3,4,5 --timestart 8:30 --timestop 18:00 -m limit --limit 5/minute -m state --state NEW -j ACCEPT
iptables -A INPUT -s 172.16.0.0/16 -p tcp -m state --state RELATED -j ACCEPT
iptables -A OUTPUT -m state --state ESTABLISHED -j ACCEPT
```





19,开放本机的ssh服务给172.16.x.1-172.16.x.100中的主机,x为你的学号,新请求建立的速率一分钟不得超过2个;仅允许响应报文通过其服务端口离开本机.

 

```
iptables -P OUTPUT DORP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
iptables -A INPUT -m state --state ESTABLISHED,NEW -j ACCEPT
iptables -A INPUT  -d 172.16.36.61 -p tcp --dport 22 -m iprange --src-range 172.16.36.1-172.16.36.100 -m connlimit --connlimit-above 2/minute -j DROP
iptables -A OUTPUT -m state --state ESTABLISHED -j ACCEPT
```





20,拒绝TCP标志位全部为1及全部为0的报文访问本机

 

```
iptables -P OUTPUT DORP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
```





21,允许本机ping别的主机,但不开放别的主机ping本机

 

```
iptables -A OUTPUT -p icmp --icmp-type 8 -j ACCEPT
iptables -A INPUT -p icmp --icmp-type 0 -j ACCEPT
```





说明:如果INPUT上放行了ESTABLISHED,就不需要再放行代码为0的


实例2

1,查看firewalld服务当前所使用的区域

firewall-cmd --get-default-zone

2,查看eno16777728网卡在firewalld服务中的区域

firewall-cmd --get-zone-of-interface=eno16777728

3,修改eno16777728网卡默认区域为external

firewall-cmd --permanent --zone=external --change-interface=eno16777728

firewall-cmd --get-zone-of-interface=eno16777728 --permanent

4,设置默认区域为public

firewall-cmd --set-default-zone=public

5,启用/关闭firewalld防火墙服务的应急模式,阻断一切网络连接(慎用)

firewall-cmd -panic-on

firewall-cmd --panic-off

6,查询public区域是否允许请求SSH和https协议的流量

firewall-cmd --zone=public --query-service=ssh

firewall-cmd --zone=public --query-service=https

7,把firewall服务中请求https协议的流量设置为永久允许,并立即生效

firewall-cmd --zone=public --add-service=https

firewall-cmd --permanent --zone=public --add-service=https

firewall-cmd --reload

8,把firewall服务中请求http协议的流量设置为永久拒绝,并立即生效

firewall-cmd --permanent --zone=public --remove-service=http

firewall-cmd --reload

9,把在firewall服务中访问8080和8081端口的流量策略设置为允许,但仅限当前生效

firewall-cmd --zone=public --add-port=8080-8081/tcp

firewall-cmd --zone=public --list-ports

10,把原本访问本机888端口的流量转发到22端口

firewall-cmd --permanent --zone=public --add-forward-port=port=888:proto=tcp:toport=22:toaddr=172.16.100.10

firewall-cmd --reload

注:流量转发命令格式为firewall-cmd --permanent --zone=<区域> --add-forward-port=port=<源端口号>:proto=<协议>:toport=<目标端口号>:toaddr=<目标IP地址>

INPUT和OUTPUT默认策略为DROP;

限制本地主机的web服务器在周一不允许访问;

web新请求的速率不能超过100/s;

web服务器包含了admin字符串的页面不允许访问;

web服务器仅允许响应报文离开本机;



```
iptables -P OUTPUT DORP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP

iptables -A INPUT -d 172.16.100.10 -p tcp --dport 80 -m time --weekdays 2,3,4,5,6,7 -m limit --limit 100/second -j ACCEPT

# string的关键字过滤,一定要做在output链上, 在回应报文中才应该会有内容
iptables -A OUTPUT -s 172.16.100.10 -p tcp --sport 80 -m string --algo bm --string "admin" -j DROP

# 仅允许响应报文离开本机
iptables -A OUTPUT -m state --state ESTABLISHED -j ACCEPT

```



在工作时间，即周一到周五的8:30-18:00,

开放本机的ftp服务给172.16.0.0网络中的主机访问;

数据下载请求的次数每分钟不得超过5个;

 

```
modprobe nf_conntrack_ftp # 需要使用related的连接状态,需要挂载此模块
iptables -P OUTPUT DORP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP

iptables -A INPUT -s 172.16.0.0/16 -p tcp --dport 21 -m time --weekdays 1,2,3,4,5 --timestart 8:30 --timestop 18:00 -m limit --limit 5/minute -m state --state NEW -j ACCEPT

iptables -A INPUT -s 172.16.0.0/16 -p tcp -m state --state RELATED -j ACCEPT

iptables -A OUTPUT -m state --state ESTABLISHED -j ACCEPT

```



开放本机的ssh服务给172.16.x.1-172.16.x.100中的主机，x为你的学号;

新请求建立的速率一分钟不得超过2个；

仅允许响应报文通过其服务端口离开本机；

```
iptables -P OUTPUT DORP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP

iptables -A INPUT -m state --state ESTABLISHED,NEW -j ACCEPT

iptables -A INPUT -d 172.16.36.61 -p tcp --dport 22 -m iprange --src-range 172.16.36.1-172.16.36.100 -m connlimit --connlimit-above 2/minute -j DROP

iptables -A OUTPUT -m state --state ESTABLISHED -j ACCEPT

```



拒绝TCP标志位全部为1及全部为0的报文访问本机；

 

```
iptables -P OUTPUT DORP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP

```



允许本机ping别的主机；但不开放别的主机ping本机；

 

```
iptables -A OUTPUT -p icmp --icmp-type 8 -j ACCEPT
iptables -A INPUT -p icmp --icmp-type 0 -j ACCEPT 
# 如果INPUT上放行了ESTABLISHED,就不需要再放行代码为0的

```



练习：判断下述规则的意义： 

 

```
iptables -N clean_in
iptables -A clean_in -d 255.255.255.255 -p icmp -j DROP
iptables -A clean_in -d 172.16.255.255 -p icmp -j DROP
iptables -A clean_in -p tcp ! --syn -m state --state NEW -j DROP
iptables -A clean_in -p tcp --tcp-flags ALL ALL -j DROP
iptables -A clean_in -p tcp --tcp-flags ALL NONE -j DROP
iptables -A clean_in -d 172.16.100.7 -j RETURN 
iptables -A INPUT -d 172.16.100.7 -j clean_in
iptables -A INPUT -i lo -j ACCEPT

iptables -A OUTPUT -o lo -j ACCEPT
# 如果在OUTPUT上允许了state为ESTABLISHED的连接, 就不需要再放行,直接可以通过 
# 等同
iptables -A INPUT -d 127.0.0.1 -j ACCEPT
iptables -A OUTPUT -s 127.0.0.1 -j ACCEPT

# 如果在OUTPUT上允许了state为ESTABLISHED的连接, 就不需要再放行,直接可以通过
iptables -A INPUT -i eth0 -m multiport -p tcp --dports 53,113,135,137,139,445 -j DROP
iptables -A INPUT -i eth0 -m multiport -p udp --dports 53,113,135,137,139,445 -j DROP
iptables -A INPUT -i eth0 -p udp --dport 1026 -j DROP
iptables -A INPUT -i eth0 -m multiport -p tcp --dports 1433,4899 -j DROP
iptables -A INPUT -p icmp -m limit --limit 10/second -j ACCEPT
```



 