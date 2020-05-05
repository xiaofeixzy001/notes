[TOC]

## 1.1 Keepalived高可用软件

### 1.1.1 Keepalived介绍

> - Keepalived软件起初是专门为LVS负载均衡软件设计的，用来管理并监控LVS集群系统中各个服务节点的状态，后来又加入了可以实现高可用的VRRP功能。因此，Keepalived除了能够管理LVS软件外，还可以作为其他服务（例如：Nginx，Haproxy，MySQL等）的高可用解决方案软件。
> - Keepalived软件主要是通过VRRP协议实现高可用功能的。VRRP是Virtual Router Redundancy  Protocol（虚拟路由器冗余协议）的缩写，VRRP出现的目的就是为了解决静态路由单点故障问题的，他能够保证当个别节点宕机时，整个网络可以不间断地运行。所以，Keepalived一方面具有配置管理LVS的功能，同时还具有对LVS下面节点进行健康检查的功能，另一方面也可实现系统网络服务的高可用功能。
> - Keepalived软件的官方站点是http://www.keepalived.org

### 1.1.2 Keepalived服务的三个重要功能

（1）管理LVS负载均衡软件

> 早期的LVS软件，需要通过命令行或脚本实现管理，并且没有针对LVS节点的健康检查功能。为了解决LVS的这些使用不便问题，Keepalived诞生了，可以说，Keepalived软件起初是专为解决LVS的问题而诞生的。因此，Keepalived和LVS的感情很深，他们的关系如同夫妻一样，可以紧密地结合，愉快地工作。Keepalived可以通过读取自身的配置文件，实现通过更底层的接口直接管理LVS的配置以及控制服务的启动，停止功能，这使得LVS的应用更加简单方便了。

（2）实现对LVS集群节点健康检查功能（healthcheck）

> 前文已讲过，Keepalived可以通过在自身的Keepalived.conf文件里配置LVS的节点IP和相关参数实现对LVS的直接管理；除此之外，当LVS集群中的某一个甚至是几个节点服务器同时发生故障无法提供服务时，Keepalived服务会自动将失效的节点服务器从LVS的正常转发队列中清除出去，并将请求调度到别的正常节点服务器上，从而保证最终用户的访问不受影响；当故障的节点服务器被修复以后，Keepalived服务又会自动地把它们加入到正常转发队列中，对客户提供服务。

（3）作为系统网络服务的高可用功能（failover）

> - Keepalived可以实现任意两台主机之间，例如Master和Backup主机之间的故障转移和自动切换，这个主机可以是普通的不能停机的业务服务器，也可以是LVS负载均衡，Nginx反向代理这样的服务器。
> - Keepalived高可用功能实现的简单原理为，两台主机同时安装好Keepalived软件并启动服务，开始正常工作时，由角色为Master的主机获得所有资源并对用户提供服务，角色为Backup的主机作为Master主机的热备；当角色为Master的主机失效或出现故障时，角色为Backup的主机将自动接管Master主机的所有工作，包括接管VIP资源及相应资源服务；而当角色为Master的主机故障修复后，又会自动接管回它原来处理的工作，角色为Backup的主机则同时释放Master主机失效时它接管的工作，此时，两台主机将恢复到最初启动时各自的原始角色及工作状态。

**说明：** Keepalived的高可用功能是本章的重点，后面除了讲解Keepalived高可用的功能外，还会讲解Keepalived配合Nginx反向代理负载均衡的高可用的实战案例。

### 1.1.3 Keepalived高可用故障切换转移原理

> - Keepalived高可用服务之间的故障切换转移，是通过VRRP（Virtual Router Redundancy Protocol，虚拟路由器冗余协议）来实现的。
> - 在Keepalived服务正常工作时，主Master节点会不断地向备节点发送（多播的方式）心跳消息，用以告诉备Backup节点自己还活着，当主Master节点发生故障时，就无法发送心跳消息，备节点也就因此无法继续检测到来自主Master节点的心跳了，于是调用自身的接管程序，接管主Master节点的IP资源及服务。而当主Master节点恢复时，备Backup节点又会释放主节点故障时自身接管的IP资源及服务，恢复到原来的备用角色。
> - 那么，什么是VRRP呢？
>    VRRP，全称Virtual Router Redundancy Protocol，中文名为虚拟路由冗余协议，VRRP的出现就是为了解决静态路由的单点故障问题，VRRP是通过一种竞选机制来将路由的任务交给某台VRRP路由器的。
> - VRRP早期是用来解决交换机，路由器等设备单点故障的，下面是交换，路由的Master和Backup切换原理描述，同样适用于Keepalived的工作原理。
> - 在一组VRRP路由器集群中，有多台物理VRRP路由器，但是这多台物理的机器并不是同时工作的，而是由一台称为Master的机器负责路由工作，其他的机器都是Backup。Master角色并非一成不变的，VRRP会让每个VRRP路由参与竞选，最终获胜的就是Master。获胜的Master有一些特权，比如拥有虚拟路由器的IP地址等，拥有系统资源的Master负责转发发送给网关地址的包和响应ARP请求。
> - VRRP通过竞选机制来实现虚拟路由器的功能，所有的协议报文都是通过IP多播（Multicast）包（默认的多播地址224.0.0.18）形式发送的。虚拟路由器由VRID（范围0-225）和一组IP地址组成，对外表现为一个周知的MAC地址：00-00-5E-00-01-{VRID}。所以，在一个虚拟路由器中，不管谁是Master，对外都是相同的MAC和IP（称之为VIP）。客户端主机并不需要因Master的改变而修改自己的路由配置。对他们来说，这种切换是透明的。
> - 在一组虚拟路由器中，只有作为Master的VRRP路由器会一直发送VRRP广播包（VRRP Advertisement  messages），此时Backup不会抢占Master。当Master不可用时，Backup就收不到来自Master的广播包了，此时多台Backup中优先级最高的路由器会抢占为Master。这种抢占是非常快速的（可能只有1秒甚至更少），以保证服务的连续性。出于安全性考虑，VRRP数据包使用了加密协议进行了加密。

**注意：**如果同学们在面试时，要你解答Keepalived的工作原理，建议用自己的话回答如下内容，以下为对面试官的表述：

**Keepalived高可用之间是通过VRRP通信的，因此，我从VRRP开始给您讲起：**

1. VRRP也就是虚拟路由冗余协议，它的出现就是为了解决静态路由的单点故障。
2. VRRP是通过一种竞选协议机制来将路由任务交给某台VRRP路由器的。
3. VRRP用IP多播的方式（默认多播地址（224.0.0.18））实现高可用之间通信。
4. 工作时主节点发包，备节点接包，当备节点接收不到主节点发的数据包的时候，就启动接管程序接管主节点的资源。备节点可以有多个，通过优先级竞选，但一般Keepalived系统运维工作中都是一对。
5. VRRP使用了加密协议加密数据，但Keepalived官方目前还是推荐用明文的方式配置认证类型和密码。

**介绍完了VRRP，接下来我再介绍一下Keepalived服务的工作原理：**

- Keepalived高可用之间是通过VRRP进行通信的，VRRP是通过竞选机制来确定主备的，主的优先级高于备，因此，工作时主会优先获得所有的资源，备节点处于等待状态，当主挂了的时候，备节点就会接管主节点的资源，然后顶替主节点对外提供服务。
- 在Keepalived服务之间，只有作为主的服务器会一直发送VRRP广播包，告诉备它还活着，此时备不会抢占主，当主不可用时，即备监听不到主发送的广播包时，就会启动相关服务接管资源，保证业务的连续性。接管速度最快可以小于1秒。

## 1.2 Keepalived高可用服务搭建准备

### 1.2.1 安装Keepalived环境说明

> 这里建议大家使用《阅读材料18》介绍的Nginx负载均衡的系统环境来安装Keepalived服务，因为本章后面的实战案例是实现Nginx负载均衡的高可用案例。安装Keepalived的基础准备环境如下。

（1）硬件环境准备

准备4台物理服务器或4台VM虚拟机，两台用来做Keepalived服务，两台做测试的Web节点如下表所示：

| HOSTNAME | IP            | 说明                                    |
| -------- | ------------- | --------------------------------------- |
| lb01     | 192.168.0.221 | Keepalived主服务器（Nginx主负载均衡器） |
| lb02     | 192.168.0.222 | Keepalived备服务器（Nginx备负载均衡器） |
| web01    | 192.168.0.223 | web01服务器                             |
| web02    | 192.168.0.224 | web02服务器                             |

（2）CentOS系统及Nginx代理环境

**下面是CentOS系统及Nginx代理环境：**

```
[root@lb01 ~]# cat /etc/redhat-release 
CentOS release 6.5 (Final)
[root@lb01 ~]# uname -r
2.6.32-431.el6.x86_64
[root@lb01 ~]# uname -m
x86_64
[root@lb01 ~]# ls -l /usr/local/nginx/
total 36
drwx------. 2 nginx root 4096 Jul 27 09:00 client_body_temp
drwxr-xr-x. 2 root  root 4096 Jul 27 09:30 conf
drwx------. 2 nginx root 4096 Jul 27 09:00 fastcgi_temp
drwxr-xr-x. 2 root  root 4096 Jul 27 08:58 html
drwxr-xr-x. 2 root  root 4096 Jul 27 09:00 logs
drwx------. 2 nginx root 4096 Jul 27 09:00 proxy_temp
drwxr-xr-x. 2 root  root 4096 Jul 27 08:58 sbin
drwx------. 2 nginx root 4096 Jul 27 09:00 scgi_temp
drwx------. 2 nginx root 4096 Jul 27 09:00 uwsgi_temp
```

### 1.2.2 开始安装Keepalived软件

**说明:**下面有关Keepalived安装，启动服务的操作都是同时处理lb01，lb02两台机器。

> 可通过官方地址获取Keepalived源码软件包编译安装，也可以使用yum的安装方式直接安装，这里选择更为简便额后者--yum安装方式，下面以lb01为例，介绍整个安装步骤，如下：

```
[root@lb01 ~]# yum -y install keepalived
[root@lb01 ~]# rpm -qa keepalived
keepalived-1.2.7-3.el6.x86_64
```

**提示：**

1）上述安装过程需要在lb01和lb02两台服务器上同时安装。
 2）Keepalived版本为2.7版

### 1.2.3 启动Keepalived服务并检查

**启动及检查Keepalived服务的命令如下：**

```
[root@lb01 ~]# /etc/init.d/keepalived start
Starting keepalived:                                       [  OK  ]
[root@lb01 ~]# ps -ef | grep keep | grep -v grep
root       7111      1  0 11:18 ?        00:00:00 /usr/sbin/keepalived -D
root       7114   7111  0 11:18 ?        00:00:00 /usr/sbin/keepalived -D
root       7117   7111  0 11:18 ?        00:00:00 /usr/sbin/keepalived -D

#提示：启动后有3个Keepalived进程表示安装正确

[root@lb01 ~]# ip add | grep 192.168
    inet 192.168.0.221/24 brd 192.168.0.255 scope global eth0
    inet 192.168.200.16/32 scope global eth0
    inet 192.168.200.17/32 scope global eth0
    inet 192.168.200.18/32 scope global eth0
#提示：默认情况会启动三个VIP地址

[root@lb01 ~]# /etc/init.d/keepalived stop
Stopping keepalived:                                       [  OK  ]
#提示：测试完毕后关闭服务，上述测试需要同时在lb01和lb02两台服务器上进行
```

### 1.2.4 Keepalived配置文件说明

和其他使用yum安装的软件一样，Keepalived软件的配置文件默认路径及配置文件名为：

```
[root@lb02 ~]# ls -l /etc/keepalived/keepalived.conf 
-rw-r--r--. 1 root root 3562 Feb 21  2013 /etc/keepalived/keepalived.conf
```

前面已经说过，Keepalived软件有3个主要功能，而这里仅讲解其高可用部分的功能。

**这里的具备高可用功能的Keepalived.conf配置文件包含了两个重要区块，下面会分别说明**

（1）全局定义（Global Definitions）部分

这部分主要用来设置Keepalived的故障通知机制和Router ID标识。示例代码如下：

```
[root@lb02 ~]# head -13 /etc/keepalived/keepalived.conf | cat -n
     1  ! Configuration File for keepalived
     2  
     3  global_defs {
     4     notification_email {
     5       acassen@firewall.loc
     6       failover@firewall.loc
     7       sysadmin@firewall.loc
     8     }
     9     notification_email_from Alexandre.Cassen@firewall.loc
    10     smtp_server 192.168.200.1
    11     smtp_connect_timeout 30
    12     router_id LVS_DEVEL
    13  }
```

**基础参数说明：**

第1行是注释，！开头和#号开发一样，都是注释。
 第2行是空行。
 第3~8行是定义服务故障报警的Email地址。作用是当服务发生切换或RS节点等有故障时，发报警邮件。这几行是可选配置，notification_email指定在Keepalived发生事件时，需要发送的Email地址，可以有多个，每行一个。
 第9行是指定发送邮件的发送人，即发件人地址，也是可选的配置。
 第10行smtp_server指定发送邮件的smtp服务器，如果本机开启了sendmail或postfix，就可以使用上面默认配置实现邮件发送，也是可选配置。
 第11行smtp_connect_timeout是连接smtp的超时时间，也是可选配置。

> **注意：**
>  第4~11行所有和邮件报警相关的参数均可以不配，在实际工作中会将监控的任务交给更加擅长监控报警的Nagios或Zabbix软件。

第12行是Keepalived服务器的路由标识（router_id）.在一个局域网内，这个标识（router_id）应该是唯一的。
 大括号“{}”。用来分隔区块，要成对出现。如果漏写了半个大括号，Keepalived运行时，不会报错，但也不会得到预期的结果。另外，由于区块间存在多层嵌套关系，因此很容易遗漏区块结尾处的大括号，要特别注意。

（2）VRRP实例定义区块（VRRP instance（s））部分

```
[root@lb02 ~]# sed -n '15,30{=;p}' /etc/keepalived/keepalived.conf | xargs -L2
15 vrrp_instance VI_1 {
16 state MASTER
17 interface eth0
18 virtual_router_id 51
19 priority 100
20 advert_int 1
21 authentication {
22 auth_type PASS
23 auth_pass 1111
24 }
25 virtual_ipaddress {
26 192.168.200.16
27 192.168.200.17
28 192.168.200.18
29 }
30 }
```

**参数说明：**

第15行表示定义一个vrrp_instance实例，名字是VI_1,每个vrrp_instance实例可以认为是Keepalived服务的一个实例或者作为一个业务服务，在Keepalived服务配置中，这样的vrrp_instance实例可以有多个。注意，存在于主节点中的vrrp_instance实例在备节点中也要存在，这样才能实现故障切换接管。

第16行state  MASTER表示当前实例VI_1的角色状态，当前角色为MASTER，这个状态只能有MASTER和BACKUP两种状态，并且需要大写这些字符。其中MASTER为正式工作的状态，BACKUP为备用的状态。当MASTER所在的服务器故障或失效时，BACKUP所在的服务器会接管故障的MASTER继续提供服务。

第17行interface为网络通信接口。为对外提供服务的网络接口，如eth0,eth1。当前主流的服务器都有2~4个网络接口，在选择服务接口时，要搞清楚了。

第18行virtual_router_id为虚拟路由ID标识，这个标识最好是一个数字，并且要在一个keepalived.conf配置中是唯一的。但是MASTER和BACKUP配置中相同实例的virtual_router_id又必须是一致的，否则将出现脑裂问题。

第19行priority为优先级，其后面的数值也是一个数字，数字越大，表示实例优先级越高。在同一个vrrp_instance实例里，MASTER的优先级配置要高于BACKUP的。若MASTER的priority值为150，那么BACKUP的priority必须小于150，一般建议间隔50以上为佳，例如：设置BACKUP的priority为100或更小的数值。

第20行advert_int为同步通知间隔。MASTER与BACKUP之间通信检查的时间间隔，单位为秒，默认为1.

第21~24行authentication为权限认证配置。包含认证类型（auth_type）和认证密码（auth_pass）。认证类型有PASS（Simple Passwd（suggested）），AH（IPSEC（not  recommended））两种，官方推荐使用的类型为PASS。验证密码为明文方式，最好长度不要超过8个字符，建议用4位数字，同一vrrp实例的MASTER与BACKUP使用相同的密码才能正常通信。

第25 ~ 29  行virtual_ipaddress为虚拟IP地址。可以配置多个IP地址，每个地址占一行，配置时最好明确指定子网掩码以及虚拟IP绑定的网络接口。否则，子网掩码默认是32位，绑定的接口和前面的interface参数配置的一致。注意，这里的虚拟IP就是在工作中需要和域名绑定的IP，即和配置的高可用服务监听的IP要保持一致！

## 1.3 Keepalived高可用服务单实例实战

### 1.3.1 配置Keepalived实现单实例单IP自动漂移接管

> - 事实上，网络服务的高可用功能基本原理都很简单，就是把手动的操作自动化运行而已。当没有配置高可用服务时，如果服务器宕机了怎么解决呢？无非就是找一个新服务器，配好域名解析的那个原IP，然后搭好相应的网络服务罢了，只不过手工去实现这个过程会比较漫长，相比而言，自动化切换效率更高，效果更好，而且还可以有更多的功能，例如：发送ARP广播，触发执行相关脚本动作等。
> - 实际上也可以将高可用对的两台机器应用服务同时开启，但是只让有VIP一端的服务器提供服务，若主的服务器宕机，VIP会自动漂移到备用服务器上，此时用户的请求直接发送到备用服务器上，而无需临时启动对应服务（事先开启应用服务）。下面就来学习VIP自动漂移的实战案例。

（1）实战配置Keepalived主服务器lb01 MASTER

**首先，配置lb01 MASTER的keepalived.conf配置文件，操作步骤如下：**

```
[root@lb01 ~]# cd /etc/keepalived/
[root@lb01 keepalived]# vim keepalived.conf

#删掉已有的所有默认配置，加入经过修改好的如下配置：
[root@lb01 keepalived]# cat keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
    215379068@qq.com        #邮箱随便写
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1        #邮件服务器IP
   smtp_connect_timeout 30
   router_id lb01               #id为lb1，不能和其他Keepalived节点相同（全局唯一）
}

vrrp_instance VI_1 {                #实例名字为VI_1,相同实例的备节点名字要和这个相同
    state MASTER                    #状态为MASTER，备节点状态需要为BACKUP
    interface eth1                  #通信（心跳）接口为eth1，此参数备节点设置和主节点相同
    virtual_router_id 55            #实例ID为55，要和备节点相同
    priority 150                    #优先级为150，备节点的优先级必须比此数字低
    advert_int 1                    #通信检查间隔时间1秒
    authentication {
        auth_type PASS              #PASS认证类型，此参数备节点设置和主节点相同
        auth_pass 1111              #密码1111，此参数备节点设置和主节点相同
    }
    virtual_ipaddress {
    192.168.0.240/24 dev eth0 label eth0:1 
    #虚拟IP，即VIP为192.168.0.240,子网掩码为24位，绑定接口为eth0，别名为eth0：1，此参数备节点设置和主节点相同
    }
}
```

**配置完毕后，启动Keepalived服务，如下：**

```
[root@lb01 keepalived]# /etc/init.d/keepalived start
```

**然后检查配置结果，查看是否有虚拟IP 192.168.0.240**

```
[root@lb01 keepalived]# ip a | grep 192.168.0.240
    inet 192.168.0.240/24 scope global secondary eth0:1
```

**出现上述带有vip:192.168.0.240行的结果表示lb01的Keepalived服务单实例配置成功**

（2）实战配置Keepalived备服务器lb02 BACKUP

**首先，配置lb02 BACKUP的keepalived.conf配置文件，操作步骤如下：**

```
[root@lb02 ~]# cd /etc/keepalived/
[root@lb02 keepalived]# vim keepalived.conf 

#删掉已有的默认配置，加入经过修改的如下配置（注意和lb01的不同）：
[root@lb02 keepalived]# cat keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
    215379068@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id lb02               #此参数和lb01 MASTER不同
}

vrrp_instance VI_1 {            #和lb01 MASTER相同
    state BACKUP                #此参数和lb01 MASTER不同
    interface eth1              #和lb01 MASTER相同
    virtual_router_id 55        #和lb01 MASTER相同
    priority 100                #此参数和lb01 MASTER不同
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
    192.168.0.240/24 dev eth0 label eth0:1
    }
}
```

**配置完成后，启动Keepalived服务，如下：**

```
[root@lb02 keepalived]# /etc/init.d/keepalived start
Starting keepalived:                                       [  OK  ]

#然后检查配置结果，查看是否有虚拟IP 192.168.0.240
[root@lb02 keepalived]# ip a | grep 192.168.0.240
[root@lb02 keepalived]# 

#这里没有返回任何结果就对了，因为lb02为BACKUP，当主节点活着的时候，它不会接管VIP 192.168.0.240
```

> - 出现上述无任何结果的现象，表示lb02的Keepalived服务单实例配置成功。如果同学们的配置过滤后有192.168.0.240的IP，则表示Keepalived工作不正常，同一个IP地址同一时刻应该只能出现一台服务器。
> - 如果查看BACKUP备节点VIP有如下信息，说明高可用裂脑了，裂脑是两台服务器争抢同一资源导致的，例如：两边都配置了同一个VIP地址。
> - 出现上述两台服务器争抢同一IP资源问题，一般要先考虑排查两个地方：
>    1）主备两台服务器对应的Keepalived.conf配置文件是否有错误？例如，是否同一实例的virtual_router_id配置不一致。

（3）进行高可用主备服务器切换实验

```
root@lb01 keepalived]# ip a | grep 192.168.0.240        #虚拟VIP在lb01服务器上
    inet 192.168.0.240/24 scope global secondary eth0:1
[root@lb01 keepalived]# /etc/init.d/keepalived stop     #停掉服务
Stopping keepalived:                                       [  OK  ]
[root@lb01 keepalived]# /etc/init.d/keepalived stop
[root@lb01 keepalived]# ip a | grep 192.168.0.240       #虚拟VIP消失了
[root@lb01 keepalived]# 

#再检查lb02服务器
[root@lb02 keepalived]# ip a | grep 192.168.0.240           #虚拟VIP出现在了lb02上
    inet 192.168.0.240/24 scope global secondary eth0:1
[root@lb02 keepalived]#
```

> - 我们可以发现，备节点lb02已经接管绑定了192.168.0.240这个VIP，这期间备节点还会发送ARP广播，让所有的客户端更新本地的ARP表，以便客户端访问新接管VIP服务的节点。
> - 此时如果再启动主服务器的Keepalived服务，主服务器就会接管回VIP 192.168.0.240，启动后可以观察下主备的IP漂移情况，备服务器是否释放了IP？主服务器是否又接管了IP？

**主节点启动Keepalived服务后，发现很快就又接管了VIP 192.168.0.240，操作及检查步骤如下：**

```
[root@lb01 keepalived]# ip a | grep 192.168.0.240
[root@lb01 keepalived]# /etc/init.d/keepalived start
Starting keepalived:                                       [  OK  ]
[root@lb01 keepalived]# ip a | grep 192.168.0.240
    inet 192.168.0.240/24 scope global secondary eth0:1

#与此同时，备节点上的VIP 192.168.0.240则被释放了，如下：
[root@lb02 keepalived]# ip a | grep 192.168.0.240
[root@lb02 keepalived]#

#这样就实现了单实例Keepalived服务IP自动漂移接管了，VIP飘逸到了新机器新服务上，用户的访问请求自然就会找新机器新服务了。
```

> **说明：**
>  这里仅实现了VIP的自动漂移切换，因此，仅适合两台服务器提供的服务均保持开启的应用场景，这也是工作中常用的高可用解决方案。

### 1.3.2 单实例主备模式Keepalived配置文件对比

![QQ截图20170731001623.png-40.4kB](http://static.zybuluo.com/chensiqi/mz2s1jc2it2re5cvrjb6zh54/QQ%E6%88%AA%E5%9B%BE20170731001623.png)

> 上述Keepalived单实例MASTER和BACKUP节点的配置差别项，只有3项是不同的。

| Keepalived配置参数   | MASTER节点特殊参数 | BACKUP节点特殊参数 |
| -------------------- | ------------------ | ------------------ |
| router_id(唯一标识)  | router_id lb01     | router_id lb02     |
| state（角色状态）    | state MASTER       | state BACKUP       |
| priority(竞选优先级) | priority 150       | priority 100       |

## 1.4 Keepalived高可用服务器的“裂脑”问题

### 1.4.1 什么是裂脑

> 由于某些原因，导致两台高可用服务器对在指定时间内，无法检测到对方的心跳消息，各自取得资源及服务的所有权，而此时的两台高可用服务器对都还活着并在正常运行，这样就会导致同一个IP或服务在两端同时存在而发生冲突，最严重的是两台主机占用同一个VIP地址，当用户写入数据时可能会分别写入到两端，这可能会导致服务器两端的数据不一致或造成数据丢失，这种情况就被称为裂脑。

### 1.4.2 导致裂脑发生的原因

**一般来说，裂脑的发生，有以下几种原因：**

- 高可用服务器对之间心跳线链路发生故障，导致无法正常通信。
- 心跳线坏了（包括断了,老化）
- 网卡及相关驱动坏了，IP配置及冲突问题（网卡直连）。
- 心跳线间连接的设备故障（网卡及交换机）
- 仲裁的机器出问题（采用仲裁的方案）
- 高可用服务器上开启了iptables防火墙阻挡了心跳消息传输
- 高可用服务器上心跳网卡地址等信息配置不正确，导致发送心跳失败。
- 其他服务配置不当等原因，如心跳方式不同，心跳广播冲突，软件BUG等

> **提示：**
>  Keepalived配置里同一VRRP实例如果virtual_router_id两端参数配置不一致，也会导致裂脑问题发生。

### 1.4.3 解决裂脑的常见方案

**在实际生产环境中，我们可以从以下几个方面来防止裂脑问题的发生：**

- 同时使用串行电缆和以太网电缆连接，同时用两条心跳线路，这样一条线路坏了，另一个还是好的，依然能传送心跳消息。
- 当检测到裂脑时强行关闭一个心跳节点（这个功能需特殊设备支持，如Stonith，fence）。相当于备节点接收不到心跳消息，通过单独的线路发送关机命令关闭主节点的电源。
- 做好对裂脑的监控报警（如邮件及手机短信等或值班），在问题发生时人为第一时间介入仲裁，降低损失。例如，百度的监控报警短信就有上行和下行的区别。报警信息发送到管理员手机上，管理员可以通过手机回复对应数字或简单的字符串操作返回给服务器，让服务器根据指令自动处理相应故障，这样解决故障的时间更短。
- 当然，在实施高可用方案时，要根据业务实际需求确定是否能容忍这样的损失。对于一般的网站常规业务，这个损失是可容忍的。

### 1.4.4 解决Keepalived裂脑的常见方案

> 作为互联网应用服务器的高可用，特别是前端Web负载均衡器的高可用，裂脑的问题对普通业务的影响是可以忍受的，如果是数据库或者存储的业务，一般出现裂脑问题就非常严重了。因此，可以通过增加冗余心跳线路来避免裂脑问题的发生，同时加强对系统的监控，以便裂脑发生时人为快速介入解决问题。

- 如果开启防火墙，一定要让心跳消息通过，一般通过允许IP段的形式解决。
- 可以拉一条以太网网线或者串口线作为主被节点心跳线路的冗余。
- 开发检测程序通过监控软件（例如Nagios）检测裂脑。

**下面是生产场景检测裂脑故障的一些思路：**

1)简单判断的思想：只要备节点出现VIP就报警，这个报警有两种情况，一是主机宕机了备机接管了；二是主机没宕，裂脑了。不管属于哪个情况，都进行报警，然后由人工查看判断及解决。

2）比较严谨的判断：备节点出现对应VIP，并且主节点及对应服务（如果能远程连接主节点看是否有VIP就更好了）还活着，就说明发生裂脑了。

**具体检测系统裂脑的脚本见本节附录**

## 1.5 Keepalived双实例双主模式配置

### 1.5.1 Keepalived双实例双主模式配置实战

> 前面给出的是Keepalived单实例主备模式的高可用演示，Keepalived还支持多实例多业务双向主备模式，即A业务在lb01上是主模式，在lb02上是备模式，而B业务在lb01上是备模式，在lb02上是主模式，下面就以双实例为例讲解不同业务实现双主的配置。

**下图为Keepalived双实例双主模式IP及VIP规划表**

| HOSTNAME | IP            | 说明                                                  |
| -------- | ------------- | ----------------------------------------------------- |
| lb01     | 192.168.0.221 | VIP:192.168.0.240(用于绑定A服务www.yunjisuan.com域名) |
| lb02     | 192.168.0.222 | VIP:192.168.0.250(用于绑定B服务bbs.yunjisuan.com域名) |

**首先，配置lb01 192.168.0.221的Keepalived.conf，在单实例的基础上增加一个vrrp_instance VI_2实例，步骤及内容如下：**

```
[root@lb01 keepalived]# cat keepalived.conf
! Configuration File for keepalived

global_defs {
   notification_email {
    215379068@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id lb01
}

vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 55
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
    192.168.0.240/24 dev eth0 label eth0:1
    }
}

vrrp_instance VI_2 {
    state BACKUP
    interface eth1
    virtual_router_id 56
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
    192.168.0.250/24 dev eth0 label eth0:2
    }
}

#提示：
以vrrp_instance VI_1在lb01 192.168.0.221服务器上的角色为主，vrrp_instance VI_2在lb01 192.168.0.221服务器上的角色为备。
```

**然后配置lb02 192.168.0.222的Keepalived.conf，在单实例的基础上增加vrrp_instance VI_2实例,步骤及内容如下：**

```
root@lb02 keepalived]# cat keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
    215379068@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id lb02
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth1
    virtual_router_id 55
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
    192.168.0.240/24 dev eth0 label eth0:1
    }
}

vrrp_instance VI_2 {
    state MASTER
    interface eth1
    virtual_router_id 56
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
    192.168.0.250/24 dev eth0 label eth0:2
    }
}

#提示：
以vrrp_instance VI_1在lb02 192.168.0.222服务器上的角色为备，vrrp_instance VI_2在lb02 192.168.0.222服务器上的角色为主。
```

**接着，在lb01，lb02上分别重启Keepalived服务，观察初始VIP设置情况。**

**首先要关闭lb01和lb02的Keepalived服务，然后再进行如下操作：**

```
#在lb01上进行如下操作：
[root@lb01 keepalived]# /etc/init.d/keepalived start
Starting keepalived:                                       [  OK  ]
[root@lb01 keepalived]# ip a | egrep "192.168.0.240|192.168.0.250"
    inet 192.168.0.240/24 scope global secondary eth0:1         #由于lb02还没开服务
    inet 192.168.0.250/24 scope global secondary eth0:2         #主备VIP都显示在lb01上

#在lb02上进行如下操作：
[root@lb02 keepalived]# /etc/init.d/keepalived start
Starting keepalived:                                       [  OK  ]
[root@lb02 keepalived]# ip a | egrep "192.168.0.240|192.168.0.250"
    inet 192.168.0.250/24 scope global secondary eth0:2     #lb01开启的情况下，lb02开启服务后，只显示了vrrp_instance VI_2实例lb02作为主模式的VIP 192.168.0.250
    
#再次在lb01上进行如下操作：
[root@lb01 ~]# ip a | egrep "240|250"
    inet 192.168.0.240/24 scope global secondary eth0:1

#lb01上只有192.168.0.240了。
```

> **特别提示：**
>  如果测试结果不符，请查看是否没有关闭iptables
>  到此为止，我们发现lb01，lb02主备节点已经实现了初始配置的VIP服务状态，当任意一端宕机，VIP可以实现互相切换接管。在实际工作中，可以把www.yunjisuan.com解析到192.168.0.240提供服务，把bbs.yunjisuan.com解析到192.168.0.250提供服务，当然了，lb01，lb02也要配置相应服务，例如：Nginx反向代理服务等。

### 1.5.2 双实例双主模式的配置文件对比

**Keepalived双实例双主模式在企业工作场景也是比较常用的，下图为大家展示了这种情况下主备配置文件的差别。**

![QQ截图20170801202912.png-51.7kB](http://static.zybuluo.com/chensiqi/lf1uy5yxzwdgckkilpkqtmia/QQ%E6%88%AA%E5%9B%BE20170801202912.png)

可以看出主备节点在增加实例方面就两项区别：

- state（状态）
- priority(精选优先级)
   其中优先级决定VIP在哪个机器上初始运行

## 1.6 Nginx负载均衡配合Keepalived服务案例实战

### 1.6.1 在lb01和lb02上配置Nginx负载均衡

结合上节介绍的Nginx负载均衡的环境，调整好主负载均衡器lb01，备用负载均衡器lb02服务器上Nginx负载均衡环境，两台服务器的安装基础环境一模一样。

![QQ截图20170801204820.png-166.3kB](http://static.zybuluo.com/chensiqi/by7zom6uga30toaioowq80fz/QQ%E6%88%AA%E5%9B%BE20170801204820.png)

**这里使用的Nginx负载均衡的配置如下：**

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

        }
    
    }
}

#提示：此配置仅代理了www.yunjisuan.com域名
```

### 1.6.2 在lb01和lb02上配置Keepalived服务

> Keepalived的配置与本节之前的实验配置一致。

### 1.6.3 用户访问准备

准备工作如下：

（1）在客户端hosts文件里把www.yunjisuan.com域名解析到VIP 192.168.0.240上，正式场景需通过DNS解析。

（2）两台服务器配好Nginx负载均衡服务，并且确保后面代理的Web节点可以测试访问。

```
#Web01上测试如下：
[root@web01 ~]# curl www.yunjisuan.com
192.168.0.223 www
[root@web01 ~]# curl www.yunjisuan.com
192.168.0.223 www
[root@web01 ~]# curl www.yunjisuan.com
192.168.0.223 www

#Web02上测试如下：
[root@web02 ~]# curl www.yunjisuan.com
192.168.0.224 www
[root@web02 ~]# curl www.yunjisuan.com
192.168.0.224 www
[root@web02 ~]# curl www.yunjisuan.com
192.168.0.224 www
```

（3）下面模拟实际的访问过程

**通过客户端浏览器输入www.yunjisuan.com测试访问，正常应该显示下图。**

![QQ截图20170801215600.png-34.6kB](http://static.zybuluo.com/chensiqi/jnippprh1fona87l69jqggt9/QQ%E6%88%AA%E5%9B%BE20170801215600.png)

**此时停止lb01服务器或停掉Keepalived服务，观察业务是否正常：**

```
[root@lb01 ~]# /etc/init.d/keepalived stop
Stopping keepalived:                                       [  OK  ]
[root@lb01 ~]# ip a | grep 192.168.0.240
```

**观察lb02备节点是否接管了VIP 192.168.0.240**

```
[root@lb02 ~]# ip a | grep 192.168.0.240
   inet 192.168.0.240/24 scope global secondary eth0:1
```

**再次在客户端浏览器输入www.yunjisuan.com测试访问，正常应该出现和切换lb02前相同的访问结果**

![QQ截图20170801215600.png-40kB](http://static.zybuluo.com/chensiqi/uf4l40rim4xrjz6eofrxyeke/QQ%E6%88%AA%E5%9B%BE20170801215600.png)

（4）开启lb01的Keepalived服务

```
 [root@lb01 ~]# /etc/init.d/keepalived start
Starting keepalived:                                       [  OK  ]
[root@lb01 ~]# ip a | grep 192.168.0.240
inet 192.168.0.240/24 scope global secondary eth0:1

#可以看到，VIP很快就接管回来了，此时浏览器访问结果依然正常
```

## 1.7 解决高可用服务只针对物理服务器的问题

> 默认情况下Keepalived软件仅仅在对方机器宕机或Keepalived停掉的时候才会接管业务。但在实际工作中，有业务服务停止而Keepalived服务还在工作的情况，这就会导致用户访问的VIP无法找到对应的服务，那么，如何解决业务服务宕机可以将IP漂移到备节点使之接管提供服务呢？

第一个方法：可以写守护进程脚本来处理。当Nginx业务有问题时，就停掉本地的Keepalived服务，实现IP漂移到对端继续提供服务。实际工作中部署及开发的示例脚本如下：

```
[root@lb01 scripts]# cat check_nginx.sh
#!/bin/sh

while true
do
    if [ `netstat -antup | grep nginx | wc -l` -ne 1 ];then
        /etc/init.d/keepalived stop
    fi
    sleep 5
done

#此脚本的基本思想是若没有80端口存在，就停掉Keepalived服务实现释放本地的VIP。在后台执行上述脚本并检查：
[root@lb01 scripts]# sh check_nginx.sh &
[1] 1521
[root@lb01 scripts]# ps -ef | grep check | grep -v grep
root       1521   1195  0 10:49 pts/0    00:00:00 sh check_nginx.sh

#确认Nginx以及Keepalived服务是正常的
[root@lb01 scripts]# netstat -antup | grep nginx
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      1492/nginx          
[root@lb01 scripts]# /etc/init.d/keepalived status
keepalived (pid  1512) is running...

#然后模拟Nginx服务挂掉，看IP是否发生切换。
[root@lb01 scripts]# /usr/local/nginx/sbin/nginx -s stop
[root@lb01 scripts]# Stopping keepalived:                  [  OK  ]
[root@lb01 scripts]# /etc/init.d/keepalived status
keepalived is stopped
[root@lb01 scripts]# netstat -antup | grep nginx

#此时，备节点已接管：
[root@lb02 ~]# ip a | grep 192.168.0.240
    inet 192.168.0.240/24 scope global secondary eth0:1
```

第二个方法：可以使用Keepalived的配置文件参数触发写好的监测服务脚本。首先要开发检测服务脚本，注意这个脚本与上一个脚本的不同。

```
[root@lb01 scripts]# cat chk_nginx_proxy.sh 
#!/bin/bash

if [ `netstat -antup | grep nginx | wc -l` -ne 1 ];then
    /etc/init.d/keepalived stop
fi

[root@lb01 scripts]# chmod +x chk_nginx_proxy.sh 
[root@lb01 scripts]# ls -l chk_nginx_proxy.sh 
-rwxr-xr-x. 1 root root 102 Jul 31 10:59 chk_nginx_proxy.sh
```

**此时，Keepalived服务的完整配置为：**

```
[root@lb01 scripts]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   notification_email {
    215379068@qq.com
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id lb01
}

vrrp_script chk_nginx_proxy {       #定义vrrp脚本，检测HTTP端口

    script "/server/scripts/chk_nginx_proxy.sh"     #执行脚本，当Nginx服务有问题，就停掉Keepalived服务
    interval 2          #间隔2秒
    weight 2

}

vrrp_instance VI_1 {
    state MASTER
    interface eth1
    virtual_router_id 55
    priority 150
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
    192.168.0.240/24 dev eth0 label eth0:1
    }
    
    track_script {
    
    chk_nginx_proxy         #触发检查
    
    }
}
```

**下面测试接管结果**

```
#先杀掉之前的后台进程脚本的运行，之后进行如下操作
[root@lb01 scripts]# /usr/local/nginx/sbin/nginx 
[root@lb01 scripts]# netstat -antup | grep nginx
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      3937/nginx          
[root@lb01 scripts]# /etc/init.d/keepalived start
Starting keepalived:                                       [  OK  ]
[root@lb01 scripts]# /etc/init.d/keepalived status
keepalived (pid  3949) is running...
[root@lb01 scripts]# ip a | grep 192.168.0.240
    inet 192.168.0.240/24 scope global secondary eth0:1
[root@lb01 scripts]# /usr/local/nginx/sbin/nginx -s stop
[root@lb01 scripts]# ip a | grep 192.168.0.240
[root@lb01 scripts]# /etc/init.d/keepalived status
keepalived is stopped

#当停掉Nginx的时候，Keepalived 2秒钟内会被自动停掉，VIP被释放，由对端接管，这样就实现了即使服务宕机也会进行IP漂移，业务切换。
```

## 1.8 解决多组Keepalived服务器在一个局域网的冲突问题

> 当在同一个局域网内部署了多组Keepalived服务器对，而又未使用专门的心跳线通信时，可能会发生高可用接管的严重故障问题。之前已经讲解过Keepalived高可用功能是通过VRRP协议实现的，VRRP协议默认通过IP多播的形式实现高可用对之间的通信，如果同一个局域网内存在多组Keepalived服务器对，就会造成IP多播地址冲突问题，导致接管错乱，不同组的Keepalived都会使用默认的224.0.0.18作为多播地址。此时的解决办法是，在同组的Keepalived服务器所有的配置文件里指定独一无二的多播地址，配置如下：

```
global_defs {

router_id LVS_19
vrrp_mcast_group4 224.0.0.19    #这个就是指定多播地址的配置

}

#提示：
1）不同实例的通信认证密码也最好不同，以确保接管正常。
2）另一款高可用软件Heartbeat，如果采用多播方式实现主备通信，同样会有多播地址冲突问题。
```

## 1.9 开发检测Keepalived裂脑的脚本

> 检测思路：在备节点上执行脚本，如果可以ping通主节点并且备节点有VIP就报警，让人员介入检查是否裂脑。

1）在lb02备节点开发脚本并执行

```
[root@lb02 scripts]# cat check_split_brain.sh 
#!/bin/bash

lb01_vip=192.168.0.240
lb01_ip=192.168.0.221

while true
do
    ping -c 2 -W 3 $lb01_ip &>/dev/null
    if [ $? -eq 0 -a `ip a | grep "$lb01_vip" | wc -l` -eq 1 ];then
        echo "ha is split brain.warning."
    else
        echo "ha is OK"
    fi
    sleep 5
done
[root@lb02 scripts]# sh check_split_brain.sh 
ha is OK
ha is OK
ha is OK

#正常情况下，主节点活着，VIP 192.168.0.221在主节点，因此不会报警，提示“ha is OK”
```

2）停止Keepalived服务看lb02脚本执行情况。

**lb01上：**

```
[root@lb01 scripts]# /etc/init.d/keepalived stop
Stopping keepalived:                                       [  OK  ]
[root@lb01 scripts]# ip a | grep 192.168.0.240
[root@lb01 scripts]# 

#在lb02上观察即可，此前脚本已经执行。
[root@lb02 scripts]# sh check_split_brain.sh 
ha is OK
ha is OK
ha is OK
ha is split brain.warning.
ha is split brain.warning.
ha is split brain.warning.
```

3）关掉lb01服务器，然后再观察lb02脚本的输出。

```
[root@lb02 scripts]# sh check_split_brain.sh 
ha is OK
ha is OK
ha is OK
ha is split brain.warning.
ha is split brain.warning.
ha is split brain.warning.
ha is OK
ha is OK
ha is OK

#裂脑报警恢复了。
```

4）可以将此脚本整合到Nagios或Zabbix监控服务里，进行监控报警。

## 1.10 本节重点回顾

1. Keepalived及VRRP协议原理
2. Keepalived配置文件重要参数
3. Keepalived单多实例VIP漂移的配置及实战
4. 裂脑的概念及如何规避解决裂脑问题
5. Keepalived单多实例结合Nginx代理配置实战
6. 高可用服务只针对物理服务器问题解决方案
7. 多组Keepalived服务器对通信冲突问题解决方案
8. 开发检测Keepalived裂脑的思路及脚本

Keepalived软件的官方文档地址：http://www.keepalived.org/documentation.html