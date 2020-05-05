[TOC]

## 一，虚拟机的安装

略

## 二，Linux系统下的网络配置（Linux虚拟机的网络设定为桥接模式）

> 桥接模式：虚拟机同主机一样，在网络中相当于一个真实存在的装有Linux系统的电脑。（我们先用这个模式）
>  NAT模式：在主机中虚拟一个局域网络，局域网络中的系统想要连接Internet，只能通过主机进行跳转连接（依靠主机），在网络中不占据真实的IP地址（目前不用，以后讲）
>  仅主机模式：只能和主机进行同网段网络连接，不能上Internet。（以后再说）

（1）ifconfig 查看网卡信息

![QQ截图20170417093856.png-14.7kB](http://static.zybuluo.com/chensiqi/hrmenpecedfhli2a7t5jdt6e/QQ%E6%88%AA%E5%9B%BE20170417093856.png)

（2）ifconfig eth0 192.168.0.222/24 临时配置网卡IP

![QQ截图20170417094309.png-15.4kB](http://static.zybuluo.com/chensiqi/sgpvl4mzd8p4rkc399mazrgf/QQ%E6%88%AA%E5%9B%BE20170417094309.png)

## 三，用Xshell连接虚拟机

> linux虚拟机有了IP和掩码才能进行同网段的数据传输（要和主机同网段）

- [X] :Xshell软件如下图

![QQ截图20170417100342.png-9.9kB](http://static.zybuluo.com/chensiqi/w7uazormthfcrr9z5avlfujv/QQ%E6%88%AA%E5%9B%BE20170417100342.png)

> 学校是Xshell4，回家下个Xshell5即可，都一样

- [X] :双击打开后如下图

![QQ截图20170417100802.png-28.9kB](http://static.zybuluo.com/chensiqi/671swn8sr5rmq0wn70f4fm11/QQ%E6%88%AA%E5%9B%BE20170417100802.png)

比如我们在windows的CMD中输入ipconfig。如下图：

![QQ截图20170417100957.png-15.4kB](http://static.zybuluo.com/chensiqi/c86tk0of4ezy0v84sg5bynv3/QQ%E6%88%AA%E5%9B%BE20170417100957.png)

我们在Xshell的本地shell里输入ipconfig。如下图：

![QQ截图20170417101139.png-24.8kB](http://static.zybuluo.com/chensiqi/d2pucvyutsend9ezdcl8kyzs/QQ%E6%88%AA%E5%9B%BE20170417101139.png)

- [X] :尝试连接linux虚拟机

（1）点击Xshell软件左上角的文件---->新建

![QQ截图20170417101348.png-21.9kB](http://static.zybuluo.com/chensiqi/shuce3fekdlywwnjrd4a1dt4/QQ%E6%88%AA%E5%9B%BE20170417101348.png)

（2）配置远程连接的信息

![QQ截图20170417101843.png-28.3kB](http://static.zybuluo.com/chensiqi/itu5wigwzroniev26zoj6ehk/QQ%E6%88%AA%E5%9B%BE20170417101843.png)

![QQ截图20170417102048.png-24.6kB](http://static.zybuluo.com/chensiqi/2w09rmxuzefoyfc9qiesl2x7/QQ%E6%88%AA%E5%9B%BE20170417102048.png)

（3）开始连接远程主机

![QQ截图20170417102231.png-16.1kB](http://static.zybuluo.com/chensiqi/g3p4fam3hgbh8fk5iuc35qel/QQ%E6%88%AA%E5%9B%BE20170417102231.png)

> **注意**：
>  只要弹出下图的界面就说明连接成功了，如果连接的速度异常缓慢，那么说明在你当前的网络范围内，有人和你的Linux虚拟机的IP地址是一样的，产生冲突所致，尝试将Linux虚拟机的IP地址换一个。

![QQ截图20170417102253.png-12.8kB](http://static.zybuluo.com/chensiqi/6qo2283p7tr10i71a25zcucr/QQ%E6%88%AA%E5%9B%BE20170417102253.png)

![QQ截图20170417102336.png-10.5kB](http://static.zybuluo.com/chensiqi/ce1asf81ht56oifr5qbsx23j/QQ%E6%88%AA%E5%9B%BE20170417102336.png)

![QQ截图20170417102420.png-23.7kB](http://static.zybuluo.com/chensiqi/z6ubp7ax2ghyg65kswepu5ab/QQ%E6%88%AA%E5%9B%BE20170417102420.png)

![QQ截图20170417102452.png-5.5kB](http://static.zybuluo.com/chensiqi/3o6x46bni3aqeet3x3c8cszr/QQ%E6%88%AA%E5%9B%BE20170417102452.png)

（4）测试xshell(在xshell里输入ifconfig)

```
[root@localhost ~]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:AB:4B:25  
          inet addr:192.168.0.222  Bcast:192.168.0.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:feab:4b25/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:895 errors:0 dropped:0 overruns:0 frame:0
          TX packets:41 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:75551 (73.7 KiB)  TX bytes:5091 (4.9 KiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:32 errors:0 dropped:0 overruns:0 frame:0
          TX packets:32 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:2372 (2.3 KiB)  TX bytes:2372 (2.3 KiB)
```

## 四，Linux的网络管理

（1）setup 管理网络设置

```
[root@localhost ~]# setup
```

（2）Xshell和linux桌面下都是中文界面，但是Linux命令模式下的setup是英文界面

**Linux虚拟机的命令行模式setup界面**

![QQ截图20170417110653.png-9.5kB](http://static.zybuluo.com/chensiqi/wspv6kq0kpub6lj8kt7uu4d8/QQ%E6%88%AA%E5%9B%BE20170417110653.png)

**Xshell界面下的setup界面**

![QQ截图20170417110740.png-9.9kB](http://static.zybuluo.com/chensiqi/6r41zu95wkhqdz62dp0blfpc/QQ%E6%88%AA%E5%9B%BE20170417110740.png)

（3）进入Network configuration界面（网络配置）

**Linux虚拟机的命令行模式setup界面**

![QQ截图20170417111419.png-4.5kB](http://static.zybuluo.com/chensiqi/cwcfml6ls91dxt0vc60k6cug/QQ%E6%88%AA%E5%9B%BE20170417111419.png)

**Xshell界面下的setup界面**

![QQ截图20170417111517.png-5.5kB](http://static.zybuluo.com/chensiqi/1qqqh4d1x64dq0urarwwh9b3/QQ%E6%88%AA%E5%9B%BE20170417111517.png)

（4）进入设备配置Device configuration

**Linux虚拟机的命令行模式setup界面**

![QQ截图20170417111722.png-6.3kB](http://static.zybuluo.com/chensiqi/e4zavkkrkc9sykbhj8p9tp1a/QQ%E6%88%AA%E5%9B%BE20170417111722.png)

**Xshell界面下的setup界面**

![QQ截图20170417111729.png-7kB](http://static.zybuluo.com/chensiqi/rs8piodvv3tos28lx2km6au1/QQ%E6%88%AA%E5%9B%BE20170417111729.png)

（5）选择一个网卡配置文件然后进入

**Linux虚拟机的命令行模式setup界面**

![QQ截图20170417112818.png-16.2kB](http://static.zybuluo.com/chensiqi/5yqswy4kfp35ws3xlutb8i53/QQ%E6%88%AA%E5%9B%BE20170417112818.png)

**Xshell界面下的setup界面**

![QQ截图20170417112844.png-10.6kB](http://static.zybuluo.com/chensiqi/evwvo8bngkd7xucry8cgy3rn/QQ%E6%88%AA%E5%9B%BE20170417112844.png)

> **特别提示：（重点）**
>
> - DHCP：是网络中专门用于IP，掩码，网关信息自动批量分发的一种软件服务，当我们点选了使用DHCP以后，如果网络中存在DHCP服务，那么系统的IP，掩码，网关信息将会自动进行获取，此功能和windows的自动获取是相同的。
> - DNS解析服务：我们知道我们想上百度，那么输入www.baidu.com就可以了。但是大家回想一下我们学过的网络知识，我们要在网络中找到目标的位置，是要通过网关进行转发的，网关需要的是你所发数据包的“IP头部”里的“目的IP地址”，因此，我们其实只有正确的输入了www.baidu.com的确切IP地址以后网关才能帮我们找到它。那么为何我们输入www.baidu.com也能连接到百度呢？这就是在网络中有一个东西帮我们将www.baidu.com这个域名自动转换成了具体的IP地址。这个东西就是DNS解析服务器。因此，如果在系统中没有配置DNS解析服务的话，我们是不能直接通过域名（www.baidu.com）来访问网站的。
> - 假如我们没有启动DHCP服务，那么我们就需要手动设定linux系统网卡的（eth0）IP，掩码，网关等信息。假如我们只配置了IP，子网掩码。那么根据网络知识，数据只能在同网段内进行传输，是不能发到网关去的（同网段内数据传输原理）。如果我们配置了网关，数据就可以进行跨网段的数据传输了（跨网段数据传输原理）因此，如果你的网关可以沟通Internet的话，当你配置了网关，就可以上Internet了。
>
> **特别注意：（重点）**
>
> 我们手动设定的Linux虚拟机的IP，掩码一定要和我们主机（我们的家用电脑的系统）所处的网络环境在同一个网段内，不然是无法通信的。网关也必须是这个网段内的真实网关IP地址（在家里就是猫的IP地址）
>  关于DNS的设置：一般情况下，DNS可以直接设定成网关IP（也有DNS功能）。或者设定为网络中公认的DNS地址，比如：202.106.0.10或201.106.0.20 （联通DNS）或8.8.8.8 and 4.4.4.4 （有点慢不是很好用）

## 五，在setup中手动设定系统的IP地址，子网掩码，网关地址，DNS信息

> 在setup里手动设定的IP，子网掩码，网关，DNS等信息，一旦经过保存，永远不会消失

（1）在setup的设备管理里手动配置信息

**Linux虚拟机的命令行模式setup界面**

![QQ截图20170417115925.png-8.2kB](http://static.zybuluo.com/chensiqi/mwqdkshzqgpq0xmqr8tml5f5/QQ%E6%88%AA%E5%9B%BE20170417115925.png)

**Xshell界面下的setup界面**

![QQ截图20170417120405.png-12.7kB](http://static.zybuluo.com/chensiqi/tpl2lteuk5xecdcnz3353xj1/QQ%E6%88%AA%E5%9B%BE20170417120405.png)

> **特别提示**：
>  设定好网卡信息之后，一定要一路保存退出，一定要选保存。

（2）保存好以后，回到linux的命令行界面启动网卡配置文件

```
[root@localhost ~]# ifup eth0     #启动网卡配置文件的命令（ifdown eth0关闭）
活跃连接状态：激活的
活跃连接路径：/org/freedesktop/NetworkManager/ActiveConnection/3
[root@localhost ~]# ifconfig eth0    #只查看eth0的网卡信息
eth0      Link encap:Ethernet  HWaddr 00:0C:29:AB:4B:25  
          inet addr:192.168.0.222  Bcast:192.168.0.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:feab:4b25/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1854 errors:0 dropped:0 overruns:0 frame:0
          TX packets:436 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:152823 (149.2 KiB)  TX bytes:81587 (79.6 KiB)
```

（3）测试网络连通性（ping命令）

```
[root@localhost ~]# ping www.baidu.com      #通过ping命令连接百度
PING www.a.shifen.com (61.135.169.125) 56(84) bytes of data.
64 bytes from 61.135.169.125: icmp_seq=1 ttl=57 time=4.28 ms
64 bytes from 61.135.169.125: icmp_seq=2 ttl=57 time=4.14 ms
64 bytes from 61.135.169.125: icmp_seq=3 ttl=57 time=4.33 ms
64 bytes from 61.135.169.125: icmp_seq=4 ttl=57 time=4.34 ms
64 bytes from 61.135.169.125: icmp_seq=5 ttl=57 time=4.66 ms
64 bytes from 61.135.169.125: icmp_seq=6 ttl=57 time=8.28 ms

特别提示：
因为已经设置DNS，所以直接ping域名我们才能通，如果没有设定DNS，则系统不认识www.baidu.com是什么。
百度的IP是61.135.169.125，连接的延迟时间为time。上边的信息表示数据连接畅通
Crtl+C键就可以终止ping的连接测试
```