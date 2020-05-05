[TOC]

VRRP协议

keepalived

VRRP协议

简述

虚拟路由冗余协议(virtual router redundancy protocol,简称VRRP)，是由IETF提出的解决局域网中配置静态网关出现单点失效现象的路由协议，1998年已推出正式的RFC2338协议标准，VRRP广泛应用在边缘网络中，它的设计目标是支持特定情况下IP数据流量失败转移不会引起混乱，允许主机使用单路由器，以及及时在实际第一跳路由器使用失败的情形下仍能够维护路由器间的连通性。

VRRP术语

虚拟路由器

由一个Master路由器和多个Backup路由器组成，主机将虚拟路由器当作默认网关

VRID

虚拟路由器的标识，有相同VRID的一组路由器构成一个虚拟路由器

Master路由器

虚拟路由器中承担报文转发任务的路由器，即主节点（仅能有一个）

Backup路由器

Master路由器出现故障时，能够代替Master路由器工作的路由器，即备用节点（可以有多个）

虚拟IP地址（VIP）

虚拟路由器的IP地址，已改为虚拟路由器可以拥有一个或多个IP地址

IP地址拥有者

接口IP地址与虚拟IP地址相同的路由器被称之为IP地址拥有者

虚拟MAC地址（VMAC)

一个虚拟路由器拥有一个虚拟MAC地址，虚拟路由器回应ARP请求使用的是虚拟MAC地址

优先级

VRRP根据优先级来确定虚拟路由器中每台路由器的地位,1-99，数字越高，优先级越高

非抢占方式

若Backup路由器工作在非抢占模式下，则只要Master路由器没有故障，Backup路由器即使随后被配置了更高的优先级也不会成为Master路由器

抢占方式

如果backup路由器工作在抢占方式下， 当它收到VRRP报文后，会将自己的优先级与通告报文中的优先级进行比较，如果自己的优先级比当前的Master优先级高，就会主动抢占成为Master路由器，否则，将保持Backup状态。

VRRP工作过程

1、虚拟路由器中的路由器根据优先级选举出Master，Master通过发送ARP报文，将自己的虚拟MAC地址发送给其它设备和主机

2、Master路由器周期性发送VRRP报文，以公布其配置信息（优先级等）和工作状态

3、如果Master路由器出现故障， 虚拟路由器的backup路由器根据优先级重新选举新的Master

4、虚拟路由器状态切换时，新的Master路由器只是简单地发送一个携带虚拟路由器的MAC地址和IP地下信息的ARP报文，这样就可以更新与它连接的主机或设备中的ARP相关信息，网络中的主机感知不到Master的切换

5、backup路由器优先级高于master路由器时，由backup路由的工作方式（抢占或非抢占方式）决定是否重新选举Master

VRRP优先级的取值范围为0-255（数值越大优先级越高），可配置的范围为1到254，优先级0为系统保留给路由器放弃master位置时使用，255则是系统保留给IP地址拥有者使用，当路由器为IP地址拥有者时，其优先级始终为255，当虚拟路由器拥有虚拟IP地址时，只要其工作正常，则为Master路由器

路由通告的工作原理

Master路由器周期发送VRRP报文，在虚拟路由器中公布其配置信息（优先级）和工作状态，backup路由器通过接收vrrp报文情况来判断master是否工作正常

master路由器主动放弃master地位时，发送优先级为0的VRRP报文，致使backup路由器切换为master路由器，这个切换时间为skew time, 计算方式为：(256-backup路由器的优先级/256,单位为秒)

当master路由器发送网络故障不能发送VRRP报文的时，backup路由器不能立即知道其工作状态，backup路由器等待一段时间后，如果还没有收到VRRP报文， 会认为master工作不正常， 而把自己升级为master路由器，周期发送VRRP报文，如果此时多个backup路由器竞争master路由的位置，将通过优先级选举master路由器，backup路由器默认等待的时间为master_down_interval,取值为：（3*VRRP报文的发送时间间隔+skew time,单位为秒）

在性能不稳定的网络中， backup路由器可能因为网络堵塞而在master_down_interval期间没有收到master路由的报文，而主动抢占master位置， 如果此时master报文又到达了， 就会出现虚拟路由器的成员频繁的进行master抢占现象，为了缓解这种情况发生，特制定了延迟等待定时器，它可以使得backup路由器在等待了master_down_interval后，再等待延迟等待时间，如果在此期间仍然没有收到VRRP报文，则此backup路由器才会切换为master路由器，对外发送VRRP报文

VRRP实现的工作

路由选举

路由状态通知

为了提高安全性，VRRP还软件娃娃了认证功能

VRRP认证方式

无认证

简单字符认证，通常用于局域网

MD5认证，跨越互联网

VRRP高可用工作模型

1,主备备份

主备备份方式表示业务仅由Master路由器承担，当Master路由器出现故障时，才会由选举出来的Backup路由器接替它的工作

2,主主备份

在路由器的一个接口上可以创建多个虚拟路由器，使得该路由器可以在一个虚拟路由器中作为Master路由器，同时在其它的虚拟路由器中作为Bacup路由器，主主备份模式可以实现负载分担方式，是指多台路由器同时承担业务，因此负载分担方式需要两个或两个以上的虚拟路由器，每个虚拟路由器都包括一个Master路由器和若干个Backup路由器。各虚拟路由器的Master路由器可以不相同.

keepalived

一,keepalived功能

keepalived程序是vrrp协议在linux主机上以守护进程方式的实现，能够根据配置文件生成IPVS规则 ，并对各real server的健康做检测，以及Loadbalance主机和backup主机之间failover的实现，keepalived在Centos6.4+收录到了发行版光盘中。

二,keepalived核心组件

核心组件

Watchdog : 高可用监视器（监控服务本身，可实现重启的）

Checkers : 健康状态检测器，可实现如下协议

TCP

HTTP

SSL

MISC

SMTP : 支持发送邮件通知机制

System Call : 通过系统调用做出管理操作

VRRP stack : VRRP栈的实现，实现VRRP协议调用

NetLink Reflectior : VRRP借助于netlink监控网络，实现网络功能配置

Ipvs wrapper : ipvs控制

IO复用器

内存管理

控制面板（配件文件分析器，以实现应用配置文件）

三,Keepalive的工作原理

1、主节点主动向备用节点发送存活通知消息（只是3层判断）

2、发送存活通知消息机制:

广播（broadcast）

组播（multicast）

单播（unicast）

3、设定各服务器的优先级，优先级判断方法

手动设定

根据IP地址数值大小，大的优先级高

随机的挑选

4、需要监控服务器的存活状态，如果服务故障需要重启服务，如重启服务无效，就需要降低主节点的优先级

5、各节点需要安装keepalive服务，并且都加入到同一个集群中，并且每个节点都监听在某个套接字止，不断向外传递心跳信息

6、多个节点配置域共享密钥，防止有人恶意加入集群

7、集群自行决定来启动服务，不能够也不应该手动启动（建立策略来决定哪个节点启动服务）

8、将多个资源绑定在一起，一同调用或配置

9、模拟VRRP协议，实现地址飘移，keepalived仅能飘移IP地址

10、不能转移服务，内置了一个模块，能直接向内核的ipvs添加规则，创建一人LVS（keepalved天生高可用lvs）

11、内置的提供了一个接口，可以通过编写脚本，来检测服务的状态，根据返回的状态，如果发生了故障，就主动降低服务器的优先级（vrrp_script,track_script）

四,keepalived的详细配置

**1,单实例**

node1,172.16.1.31,主

node2,172.16.1.32,备

1.1,配置前提

```
cat /etc/redhat-release.2018-04-18 
"""
CentOS release 6.9 (Final)
"""

hostname
"""
node1(node2)
"""

# 2节点时间一致
ntpdate ntp1.aliyun.com

# iptables及selinux服务关闭
iptables -L
getenforce

# 可通过主机名互相通信,节点的名称设定与hosts文件中解析的主机名都要保持一致(AIS架构必须项)
cat /etc/hosts
"""
172.16.1.30 node1.com node1
172.16.1.31 node2.com node2
"""

# 各节点基于密钥认证的方式通过ssh互信通信,这里为了方便三个节点同一对密钥
# node1节点操作
ssh-genkey -t rsa
scp -P2201 /root/.ssh/{id_rsa,id_rsa.pub} node2:/root/.ssh/

# 所有节点操作
cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
chmod 600 .ssh/authorized_keys

#============================================
cat /etc/redhat-release.2018-04-18 
"""
CentOS release 6.9 (Final)
"""
hostname
"""
node1(node2)
"""
# 2节点时间一致
ntpdate ntp1.aliyun.com
# iptables及selinux服务关闭
iptables -L
getenforce
# 可通过主机名互相通信,节点的名称设定与hosts文件中解析的主机名都要保持一致(AIS架构必须项)
cat /etc/hosts
"""
172.16.1.30 node1.com node1
172.16.1.31 node2.com node2
"""
# 各节点基于密钥认证的方式通过ssh互信通信,这里为了方便三个节点同一对密钥
# node1节点操作
ssh-genkey -t rsa
scp -P2201 /root/.ssh/{id_rsa,id_rsa.pub} node2:/root/.ssh/
# 所有节点操作
cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
chmod 600 .ssh/authorized_keys
```

 

1.2,各节点安装keepalived

```shell
 `yum install keepalived -y``rpm -ql keepalived``"""``/etc/keepalived ``/etc/keepalived/keepalived.conf  # 主配置文件``/etc/rc.d/init.d/keepalived  # 服务启动脚本``/etc/sysconfig/keepalived  # 服务启动时的配置``/usr/libexec/keepalived``/usr/sbin/keepalived  # 主程序``/usr/bin/genhash     # 生成hash指纹的工具:genhash -s 192.168.100.7 -p 80 -u index.html``...``"""` `yum install keepalived -y``rpm -ql keepalived``"""``/etc/keepalived ``/etc/keepalived/keepalived.conf  # 主配置文件``/etc/rc.d/init.d/keepalived  # 服务启动脚本``/etc/sysconfig/keepalived  # 服务启动时的配置``/usr/libexec/keepalived``/usr/sbin/keepalived  # 主程序``/usr/bin/genhash   # 生成hash指纹的工具:genhash -s 192.168.100.7 -p 80 -u index.html``...``"""`
```



1.3,修改配置文件,实现vip漂移

```
# node1操作
cp keepalived.conf{,.bak}
man keepalived.conf  # 可查看配置文件帮助信息
vim keepalived.conf
"""
# vritual_server暂时用不到,先注释掉, :.,$s/^/#/g
! Configuration File for keepalived

# 全局配置
global_defs {
    # 邮件通知
    notification_email {
        # 收件人
        root@localhost
    }
    notification_email_from kadmin@localhost  # 发件人
    smtp_server 127.0.0.1  # 发送服务器
    smtp_connect_timeout 30  # 超时时长
    router_id node1.com  # 一般是主机名
    vrrp_mcast_group4 224.18.0.100  # 组播地址
}

# 配置虚拟路由实例
vrrp_instance VI_1 {  # 示例名,可有多个,不要重名
    state MASTER  # 初始状态
    interface eth0  # 定义vip配置在哪个网卡上
    virtual_router_id 51  # 虚拟路由组id,唯一,默认51
    priority 100  # 优先级,数字越大,优先级越高
    advert_int 1  # 每隔几秒发送一次心跳信息,默认1s
    authentication {
        auth_type PASS  # 心跳信息认证方式,简单字符认证和MD5认证
        auth_pass 64e06f86  # openssl rand -hex 4
    }
    virtual_ipaddress {  # 定义虚拟ip地址
        172.16.1.30
    }
}
"""

scp -P2201 /etc/keepalived/keepalived.conf node2:/etc/keepalived/

# node2操作
vim /etc/keepalived/keepalived.conf
"""
! Configuration File for keepalived

global_defs {
   notification_email {
        root@localhost
   }
   notification_email_from kaadmin@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id node2.com
   vrrp_mcast_group4 224.18.0.10
}

vrrp_instance VI_1 {
    state BACKUP  # 这里改成BACKUP
    interface eth0
    virtual_router_id 51
    priority 99  # 优先级改低一些
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 64e06f86
    }
    virtual_ipaddress {
        172.16.1.30
    }
}
"""

# 所有节点操作
service keepalived start
ps aux | grep keepalived
```

 

```
# node1操作
cp keepalived.conf{,.bak}
man keepalived.conf  # 可查看配置文件帮助信息
vim keepalived.conf
"""
# vritual_server暂时用不到,先注释掉, :.,$s/^/#/g
! Configuration File for keepalived
# 全局配置
global_defs {
    # 邮件通知
    notification_email {
        # 收件人
        root@localhost
    }
    notification_email_from kadmin@localhost  # 发件人
    smtp_server 127.0.0.1  # 发送服务器
    smtp_connect_timeout 30  # 超时时长
    router_id node1.com  # 一般是主机名
    vrrp_mcast_group4 224.18.0.100  # 组播地址
}
# 配置虚拟路由实例
vrrp_instance VI_1 {  # 示例名,可有多个,不要重名
    state MASTER  # 初始状态
    interface eth0  # 定义vip配置在哪个网卡上
    virtual_router_id 51  # 虚拟路由组id,唯一,默认51
    priority 100  # 优先级,数字越大,优先级越高
    advert_int 1  # 每隔几秒发送一次心跳信息,默认1s
    authentication {
        auth_type PASS  # 心跳信息认证方式,简单字符认证和MD5认证
        auth_pass 64e06f86  # openssl rand -hex 4
    }
    virtual_ipaddress {  # 定义虚拟ip地址
        172.16.1.30
    }
}
"""
scp -P2201 /etc/keepalived/keepalived.conf node2:/etc/keepalived/
# node2操作
vim /etc/keepalived/keepalived.conf
"""
! Configuration File for keepalived
global_defs {
   notification_email {
        root@localhost
   }
   notification_email_from kaadmin@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id node2.com
   vrrp_mcast_group4 224.18.0.10
}
vrrp_instance VI_1 {
    state BACKUP  # 这里改成BACKUP
    interface eth0
    virtual_router_id 51
    priority 99  # 优先级改低一些
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 64e06f86
    }
    virtual_ipaddress {
        172.16.1.30
    }
}
"""
# 所有节点操作
service keepalived start
ps aux | grep keepalived
```

测试:当node1主节点down掉后,VIP会自动切换至node2备节点,当node1上线后,VIP又切换回了node1,可知默认工作在抢占模式下.

扩展:有时候可能会需要手动切换vip,可使用vrrp_script实现依赖脚本监测文件的存在性,来降低节点的优先级,以此实现主备模式的切换.

 

```
# 在主节点node1上配置
! Configuration File for keepalived

global_defs {
    notification_email {
        root@localhost
    }
    notification_email_from kadmin@localhost
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id node1.com
    vrrp_mcast_group4 224.18.0.100
}

# 添加如下:
vrrp_script chk_down {  # chk_down:自定义名字
    script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"  # 判断文件是否存在,true返回1,false返回0
    interval 1  # 检测间隔
    weight -2  # true则权重减2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 64e06f86
    }
    virtual_ipaddress {
        172.16.1.30
    }
    # 添加如下:
    track_script {
        chk_down
    }
}
```

 

```
# 在主节点node1上配置
! Configuration File for keepalived
global_defs {
    notification_email {
        root@localhost
    }
    notification_email_from kadmin@localhost
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id node1.com
    vrrp_mcast_group4 224.18.0.100
}
# 添加如下:
vrrp_script chk_down {  # chk_down:自定义名字
    script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"  # 判断文件是否存在,true返回1,false返回0
    interval 1  # 检测间隔
    weight -2  # true则权重减2
}
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 64e06f86
    }
    virtual_ipaddress {
        172.16.1.30
    }
    # 添加如下:
    track_script {
        chk_down
    }
}
```

测试:在node1上/etc/keepalived/目录下touch一个down文件,查看vip位置

**2,双实例**

VI_1:

node1, 172.16.1.31,主

node2, 172.16.1.32,备

VIP: 172.16.1.100

VI_2:

node1, 172.16.1.31,备

node2, 172.16.1.32,主

VIP: 172.16.1.200

要求:为保证两个主机资源同时利用,可以配置两个实例,让两者互为主备,因此需要配置两个虚拟路由,两个VIP等.

 

```
# node1配置
"""
! Configuration File for keepalived
 
global_defs {
   notification_email {
        root@localhost
   }
   notification_email_from kaadmin@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id node1.com
   vrrp_mcast_group4 224.18.0.10
}

vrrp_script chk_down {  # chk_down:自定义名字
    script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"  # 判断文件是否存在,true返回1,false返回0
    interval 1  # 检测间隔
    weight -2  # true则权重减2
}

vrrp_instance VI_1 {  # 第一个vrrp主
    state MASTER
    interface eth0  # 如果是centos7,注意网卡名
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 7db4740d  # 注意不要与第二个vrrp的心跳验证一样
    }
    virtual_ipaddress {
        172.16.1.100/24 dev eth0 label eth0:0
    }
    track_script {
        chk_down
    }
}
vrrp_instance VI_2 {  # 第二个vrrp备
    state BACKUP
    interface eth0
    virtual_router_id 61
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 7db4740dqw
    }
    virtual_ipaddress {
        172.16.1.200/24 dev eth0 label eth0:1
    }
    track_script {
        chk_down
    }
}
"""

# node2配置
"""
! Configuration File for keepalived
 
global_defs {
   notification_email {
        root@localhost
   }
   notification_email_from kaadmin@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id node1.com
   vrrp_mcast_group4 224.18.0.10
}

vrrp_script chk_down {  # chk_down:自定义名字
    script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"  # 判断文件是否存在,true返回1,false返回0
    interval 1  # 检测间隔
    weight -2  # true则权重减2
}

vrrp_instance VI_1 {  # 第一个vrrp备
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 7db4740d
    }
    virtual_ipaddress {
        172.16.1.100/24 dev eth0 label eth0:0
    }
    track_script {
        chk_down
    }
}
vrrp_instance VI_2 {  # 第二个vrrp主
    state MASTER
    interface eth0
    virtual_router_id 61
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 7db4740dqw
    }
    virtual_ipaddress {
        172.16.1.200/24 dev eth0 label eth0:1
    }
    track_script {
        chk_down
    }
}
"""

# 测试
```

 

```
# node1配置
"""
! Configuration File for keepalived
 
global_defs {
   notification_email {
        root@localhost
   }
   notification_email_from kaadmin@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id node1.com
   vrrp_mcast_group4 224.18.0.10
}
vrrp_script chk_down {  # chk_down:自定义名字
    script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"  # 判断文件是否存在,true返回1,false返回0
    interval 1  # 检测间隔
    weight -2  # true则权重减2
}
vrrp_instance VI_1 {  # 第一个vrrp主
    state MASTER
    interface eth0  # 如果是centos7,注意网卡名
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 7db4740d  # 注意不要与第二个vrrp的心跳验证一样
    }
    virtual_ipaddress {
        172.16.1.100/24 dev eth0 label eth0:0
    }
    track_script {
        chk_down
    }
}
vrrp_instance VI_2 {  # 第二个vrrp备
    state BACKUP
    interface eth0
    virtual_router_id 61
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 7db4740dqw
    }
    virtual_ipaddress {
        172.16.1.200/24 dev eth0 label eth0:1
    }
    track_script {
        chk_down
    }
}
"""
# node2配置
"""
! Configuration File for keepalived
 
global_defs {
   notification_email {
        root@localhost
   }
   notification_email_from kaadmin@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id node1.com
   vrrp_mcast_group4 224.18.0.10
}
vrrp_script chk_down {  # chk_down:自定义名字
    script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"  # 判断文件是否存在,true返回1,false返回0
    interval 1  # 检测间隔
    weight -2  # true则权重减2
}
vrrp_instance VI_1 {  # 第一个vrrp备
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 7db4740d
    }
    virtual_ipaddress {
        172.16.1.100/24 dev eth0 label eth0:0
    }
    track_script {
        chk_down
    }
}
vrrp_instance VI_2 {  # 第二个vrrp主
    state MASTER
    interface eth0
    virtual_router_id 61
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 7db4740dqw
    }
    virtual_ipaddress {
        172.16.1.200/24 dev eth0 label eth0:1
    }
    track_script {
        chk_down
    }
}
"""
# 测试
```

问题总结:

1,CentOS7上,每个vrrp_instance需要专用的组播地址

2,日志记录信息过于简单

附:

配置文件/etc/keepalived/keepalived.conf说明:

 

```
Global configuration : # 全局配置段
  global_defs {
      ...
  }
VRRP Configuration : # 配置VRRP实例
  vrrp_instance NAME {
      ...
  }
LVS Configuration : # IPVS的相关配置
  virtual_server IP PORT {
      ...
      real_server IP PORT {
          ...
      }
  }
```

 

```
Global configuration : # 全局配置段
  global_defs {
      ...
  }
VRRP Configuration : # 配置VRRP实例
  vrrp_instance NAME {
      ...
  }
LVS Configuration : # IPVS的相关配置
  virtual_server IP PORT {
      ...
      real_server IP PORT {
          ...
      }
  }
```

Global指令

notification_email { mail_address } : 邮件通知的对象，收件人邮箱

notification_email_from mail_name: 发件人邮箱

smtp_server : 邮件发送服务器IP地址

smtp_connect_timeout : 连接邮件服务器的超时时长

router-id HOSTNAME : 物理节点的标识符，建议使用主机名

vrrp_mcast_grou4 224.0.0.18 : vrrp的多播地址，IPV4，默认为224.0.0.18

vrrp_mcast_group6 ff02::12 : vrrp的多播地址， IPV6

vrrp_script NAME { } : 定义脚本，可以在vrrp_instance中使用track_script引用

script COMMAND : script是固定字段，后面为脚本的内容，有空格需要使用引号包括起来

interval # : 间隔多长时间进行状态查看,以秒为单位

weight [+|-] # : 如果脚本的返回状态是失败的，将优先级减去相应的数值

nopreempt : 定义为非抢占模式，默认抢占模式

preempt_delay TIME : 定义为延迟抢占模式

VRRP_instance指令

state MASTER | BACKUP : 在当前VRRP实例中（虚拟路由器组）此节点的初始实例

Interface IFACE_NAME : vrrp用于绑定VIP的接口，各节点网卡接口名称需保持一致

virtual_route_id # : 虚拟路由器的ID（VRID），可用值为0-255,默认为51，唯一

priority # : 当前路由器节点的优先级，可用范围为0-255

advert_in # : 通告时间间隔，单位是秒种，默认是1秒

authentication { } : 定义认证的特殊引用段

auth type PASS : 指定集群密钥方式

auth_pass 1234 : 字符密钥有前8个有效(生成随机字串:openssl rand -hex 4)

virtual_ipaddress { } : 定义集群中主机的特殊引用

\/brd \Dev\scope \label \

notify_master <string> | <quoted-string> : 当前节点转为主节点触发的脚本

notify_backup <string> | <quoted-string> : 当前节点转为备用节点触发的脚本

notify_fault <string> | <quoted-string> : 当前节点出现故障时触发的脚本

notify <string> | <quoted-string> : 一般情况下， 只使用上面三个， 或者只使用这一个

track_script { VRRP_SCRIPT_NAME } : 使用此引用，可以调用vrrp_script定义的脚本并执行

track_script { IFACE_NAME } : 调用vrrp_script内置方法，可以判断主机的网络接口是否正常，如果不正常将自动降低其权重，转为backup模式

virtual_server指令

delay_loop <INT> : 延迟多长时间检测集群服务是否OK

lb_algo rr|wrr|lc|wlc|lblc|sh|dh : lvs的调度算法

lb_kind NAT|DR|TUN : lvs的类型，如需支持fullnat，需要打补丁

persistence_timeout <INT> : 持久时长，0表示不启用

nat_mask 255.255.255.0 : IP掩码地址，此处的nat没有写错

protocol TCP : lvs调度的协议，默认是udp,如果是udp可以不用添加此指令

virtual_host <string> : 对哪个虚拟主机做健康状态检测,可以不定义

quorum <INT> : 最少法定票数，判断realserver有几台才是OK的

quorum_up : 添加票数

quorum_down : 降低票数

sorry_server <IPADDR><PORT> : 定义sorry server

real_server IP PORT { } :定义一个real_server主机

weight <INT> : 权重

inhibit_no_failure : 如果检测失败，就把权重设置为0

notify_up <string> | <quoted-string> : realserver上线通知，依赖脚本完成

notify_down <string> | <quoted-string> : realserver下线通知，依赖脚本完成

HTTP_GET|SSL_GET|TCP_CHECK|SMTP_CHECK|MISC_CHECK : 对后端的realserver主机，使用相应的方法做健康状态检测

url { } : 对url做健康检测的特殊引用

path <string> : 检测的url路径

status_code <INT> : 依赖返回状态码进行检测

digest <string> : 依赖页面的hash值进行检测，基于genhash命令完成hash值计算

nb_get_retry <INT> : get请求的重试次数

delay_befor_retry <INT> : 两次重试之间的时间间隔，要延迟多长时间，再retry

connect_ip <IPADDR> : 默认会按real server的IP做健康状态检测，一般不需要再写，但有的时候可能有一个IP专门做健康状态检测的IP，故要手动添加上去

connect_port <PORT> : 连接的端口

bindto <IPADDR> : 如果keepalived主机有多个IP，定义用哪个IP完成健康状态检测

connect_timeout <INT> : 连接超时时长，默认为5秒，但这个时长比较长， 建议调整至3秒

warmup <INT> : 做健康状态检测的延迟， 即keepalived服务起来后，等待多长时间再开始健康状态检测，有时候后端的real server还未启动完成，故需要等待一段时间

TCP_CHECK { } : 传输层健康状态检测

connect_timeout <INT> : 连接超时时长

connect_ip : 检测的realserver的IP,一般不需要写，同url中的参数一样

connect_port : 检测的realserver的port,一般不需要写，同url中的参数一样

bindto <IP ADDR> : 同url中的参数一样

bind_port <port> : 使用哪个端口做健康状态检测