[TOC]

## 六，管理虚拟网络

- [x] Linux网桥基本概念
- [x] qemu-kvm支持的网络
- [x] 向虚拟机添加虚拟网络连接
- [x] 基于NAT的虚拟网络
- [x] 基于网桥的虚拟网络
- [x] 用户自定义的隔离的虚拟网络

### 6.1 Linux网桥与qemu-kvm支持的网络

**Linux网桥基本概念**

- [x] 数据链路的设备，基于MAC地址进行转发
- [x] Redhat/CentOS配置网桥常用方法
  - 命令行（推荐）
  - nmtui：NetworkManager的文本用户接口
  - nmcli：NetworkManager的命令行工具
     `# nmcli con add type bridge ifname br0`
     `# nmcli con show`
  - 图形界面管理工具

**qemu-kvm支持的网络**

- [x] 虚拟机的网络模式：
  - 基于NAT（NetworkAddressTranslation）的虚拟网络
  - 基于网桥（Bridge）的虚拟网络
  - 用户自定义的隔离的虚拟网络
  - 直接分配网络设备（包括VT-d和SR-IOV）
- [x] 虚拟机的网卡：
  - RTL8139，e1000，....
  - virtio
     `# /usr/libexec/qemu-kvm -net nic,mode1=?`

**演示：考察默认的虚拟网络的配置**

- [x] 查看宿主机的网络配置
- [x] 查看虚拟机的网络配置

![221.png-20.3kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/221.png)

![222.png-34.3kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/222.png)

![223.png-33.3kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/223.png)

```
#qemu-kvm的虚拟网络配置文件在哪？
[root@localhost ~]# ls /etc/libvirt/    #libvirt的所有配置文件目录
libvirt-admin.conf  lxc.conf  qemu.conf        virtlockd.conf
libvirt.conf        nwfilter  qemu-lockd.conf  virtlogd.conf
libvirtd.conf       qemu      storage       #storage目录，所有存储池的XML配置文件
[root@localhost ~]# ls /etc/libvirt/qemu    #qemu目录所有qemu有关的配置文件
autostart         centos6.5-2.xml  centos6.5.xml  erp.xml  LNMP.xml  oa.xml
Base_CentOS7.xml  centos6.5-3.xml  crm.xml        hr.xml   networks  vm2.xml
[root@localhost ~]# ls /etc/libvirt/qemu/networks/  #qemu里存储所有虚拟网络配置文件的目录networks
autostart  default.xml      #default.xml这个就是默认的虚拟网络的XML配置文件
[root@localhost ~]# cat /etc/libvirt/qemu/networks/default.xml  #查看default.xml内容
<!--
WARNING: THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
OVERWRITTEN AND LOST. Changes to this xml configuration should be made using:
  virsh net-edit default
or other application using the libvirt API.
-->

<network>
  <name>default</name>      #虚拟网络的名字
  <uuid>5687d2e1-c14d-42bb-abe2-fcb4bfac2a12</uuid> #UUID号
  <forward mode='nat'/>     #虚拟网络的模式NAT
  <bridge name='virbr0' stp='on' delay='0'/>    #虚拟网络的网桥名称
  <mac address='52:54:00:79:e3:41'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>      #网桥的IP和掩码
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>  #DHCP的分发范围
    </dhcp>
  </ip>
</network>
```

> 利用virsh 管理虚拟网络

```
#virsh里关于网络部分的命令
[root@localhost ~]# virsh help network
 Networking (help keyword 'network'):
    net-autostart                  自动开始网络
    net-create                     从一个 XML 文件创建一个网络
    net-define                     define an inactive persistent virtual network or modify an existing persistent one from an XML file
    net-destroy                    销毁（停止）网络
    net-dhcp-leases                print lease info for a given network
    net-dumpxml                    XML 中的网络信息
    net-edit                       为网络编辑 XML 配置
    net-event                      Network Events
    net-info                       网络信息
    net-list                       列出网络
    net-name                       把一个网络UUID 转换为网络名
    net-start                      开始一个(以前定义的)不活跃的网络
    net-undefine                   undefine a persistent network
    net-update                     更新现有网络配置的部分
    net-uuid                       把一个网络名转换为网络UUID

#查看所有虚拟网络信息
[root@localhost ~]# virsh net-list
 名称               状态     自动开始  持久
----------------------------------------------------------
 default              活动     是           是

#查看某虚拟网络详细信息
[root@localhost ~]# virsh net-info default
名称：       default
UUID:           5687d2e1-c14d-42bb-abe2-fcb4bfac2a12
活跃：       是
持久：       是
自动启动： 是
桥接：       virbr0

#查看某虚拟网络的XML配置文件信息
[root@localhost ~]# virsh net-dumpxml default
<network connections='1'>
  <name>default</name>
  <uuid>5687d2e1-c14d-42bb-abe2-fcb4bfac2a12</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:79:e3:41'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
```

> 什么叫做网桥？网桥到底是怎么回事？

```
[root@localhost ~]# ifconfig -a
ens32: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500     #宿主机的真实网卡接口
        inet 192.168.200.132  netmask 255.255.255.0  broadcast 192.168.200.255
        inet6 fe80::d302:4c4f:17a0:b161  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:96:3a:c5  txqueuelen 1000  (Ethernet)
        RX packets 793722  bytes 74452602 (71.0 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1308099  bytes 2734536899 (2.5 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536                    #宿主机的lo回环接口
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1  (Local Loopback)
        RX packets 739954  bytes 1460949048 (1.3 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 739954  bytes 1460949048 (1.3 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

virbr0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500            #虚拟网桥（虚拟交换机）virbr0
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:79:e3:41  txqueuelen 1000  (Ethernet)
        RX packets 2780  bytes 222708 (217.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3652  bytes 360625 (352.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

virbr0-nic: flags=4098<BROADCAST,MULTICAST>  mtu 1500                   #连接到网桥virbr0上的宿主机的虚拟网卡接口
        ether 52:54:00:79:e3:41  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vnet0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500             ##连接到virbr0上的虚拟机的网卡接口
        inet6 fe80::fc54:ff:fe0c:8bd2  prefixlen 64  scopeid 0x20<link>
        ether fe:54:00:0c:8b:d2  txqueuelen 1000  (Ethernet)
        RX packets 2780  bytes 261628 (255.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 30643  bytes 1764617 (1.6 MiB)
        TX errors 0  dropped 7222 overruns 0  carrier 0  collisions 0

[root@localhost ~]# brctl show
bridge name bridge id       STP enabled interfaces
virbr0      8000.52540079e341   yes     virbr0-nic      #连接到网桥virbr0上的宿主机的虚拟网卡接口
                                        vnet0           #连接到网桥virbr0上的虚拟机的虚拟网卡接口
```

**NAT模式的网桥连接概念图：**

![QQ截图20180521231224.png-37.1kB](http://static.zybuluo.com/chensiqi/qo6shvhhxfr69o6o4dviaxbr/QQ%E6%88%AA%E5%9B%BE20180521231224.png)

**查看虚拟机的网络连接配置：**

```
[root@localhost ~]# virsh list
 Id    名称                         状态
----------------------------------------------------
 1     centos6.5                      running

[root@localhost ~]# virsh edit centos6.5    #相当于以vim在内存中即时打开虚拟机的XML配置文件
#############以上省略若干############
    <interface type='network'>      #虚拟机的网络接口类型
      <mac address='52:54:00:0c:8b:d2'/>        #虚拟机的网卡MAC地址
      <source network='default'/>               #虚拟机的网卡的源网络名称
      <model type='virtio'/>                    #虚拟机的网络接口模式virtio
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
    </interface>

#############以下省略若干############
```

> 在宿主机中测试网络联通性

![231.png-38.9kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/231.png)

> 在虚拟机中测试网络联通性

![232.png-23.1kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/232.png)

### 6.2 基于NAT的虚拟网络

![234.png-312.1kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/234.png)

```
[root@localhost ~]# virsh list
 Id    名称                         状态
----------------------------------------------------
 1     centos6.5                      running
 5     centos6.5-2                    running

[root@localhost ~]# brctl show
bridge name bridge id       STP enabled interfaces
virbr0      8000.52540079e341   yes     virbr0-nic
                                        vnet0
                                        vnet1
```

**通过图形界面向虚拟机中添加一块NAT网卡**

![241.png-31.9kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/241.png)

```
[root@localhost ~]# virsh list
 Id    名称                         状态
----------------------------------------------------
 1     centos6.5                      running
 5     centos6.5-2                    running
 
[root@localhost ~]# virsh domiflist centos6.5   #查看虚拟机网络接口类型
接口     类型     源        型号      MAC
-------------------------------------------------------
vnet0      network    default    virtio      52:54:00:0c:8b:d2
vnet2      network    default    virtio      52:54:00:ea:57:f7

[root@localhost ~]# virsh domiflist centos6.5-2 #查看虚拟机网络接口类型
接口     类型     源        型号      MAC
-------------------------------------------------------
vnet1      network    default    virtio      52:54:00:35:29:ea

[root@localhost ~]# virsh domifaddr centos6.5   #查看虚拟机网卡IP地址
 名称     MAC 地址           Protocol     Address
-------------------------------------------------------------------------------
 vnet0      52:54:00:0c:8b:d2    ipv4         192.168.122.123/24

[root@localhost ~]# virsh domifstat centos6.5 vnet0 #查看虚拟机指定网卡的状态
vnet0 rx_bytes 2556553
vnet0 rx_packets 42292
vnet0 rx_errs 0
vnet0 rx_drop 7222
vnet0 tx_bytes 625776
vnet0 tx_packets 6566
vnet0 tx_errs 0
vnet0 tx_drop 0
```

### 6.3 基于网桥的虚拟网络

![QQ截图20180525212201.png-339.5kB](http://static.zybuluo.com/chensiqi/mzv5x13r767bu9e88yfq3vu3/QQ%E6%88%AA%E5%9B%BE20180525212201.png)

> 由上图可知：
>
> - NAT模式：通过网桥virbr0将宿主机上的一块虚拟网卡virbr0-nic与虚拟机的虚拟网卡vnet0进行连接，而后虚拟机的数据包通过网桥virbr0发送到宿主机的虚拟网卡virbr0-net上，再进行宿主机网卡间的数据包转发实现的虚拟机通过宿主机来上网。
> - 桥接模式：通过网桥virbr0将虚拟机的虚拟网卡vnet1直接连接在宿主机的真实物理网卡上，然后通过宿主机的真实物理网卡来上网。
>    **因此，想要实现桥接的上网模式，我们首先需要学会如何来创建网桥virbr1**

```
#modprobe探测内核对于某个模块是否加载的命令
[root@localhost ~]# which modprobe
/usr/sbin/modprobe

#探测bridge模块是否安装，如果没有，那么--first-time第一时间加载这个模块
[root@localhost ~]# modprobe --first-time bridge    #探测网桥模块是否被内核加载
modprobe: ERROR: could not insert 'bridge': Module already in kernel    #kernel已经加载了

[root@localhost ~]# modinfo bridge
filename:       /lib/modules/3.10.0-514.el7.x86_64/kernel/net/bridge/bridge.ko
alias:          rtnl-link-bridge
version:        2.3
license:        GPL
rhelversion:    7.3
srcversion:     FF0448CD85C271287DE1963
depends:        stp,llc
intree:         Y
vermagic:       3.10.0-514.el7.x86_64 SMP mod_unload modversions 
signer:         CentOS Linux kernel signing key
sig_key:        D4:88:63:A7:C1:6F:CC:27:41:23:E6:29:8F:74:F0:57:AF:19:FC:54
sig_hashalgo:   sha256
```

#### 6.3.1 通过命令行进行网桥virbr1的创建

```
[root@localhost ~]# cd /etc/sysconfig/network-scripts/  #进入宿主机物理网卡配置文件目录
[root@localhost network-scripts]# cat ifcfg-ens32       #查看宿主机物理网卡配置文件信息
TYPE=Ethernet
BOOTPROTO=dhcp
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens32
UUID=17fb5987-5317-4bca-8514-9e1b73933184
DEVICE=ens32
ONBOOT=yes
[root@localhost network-scripts]# mkdir bak #创建备份目录
[root@localhost network-scripts]# cp ifcfg-ens32 bak/ifcfg-ens32.bak    #复制一份网卡配置文件备份

#复制一份物理网卡配置文件进行修改，作为网桥配置文件
[root@localhost network-scripts]# cp ifcfg-ens32 ifcfg-virbr1
[root@localhost network-scripts]# vim ifcfg-virbr1 
[root@localhost network-scripts]# cat ifcfg-virbr1 
DEVICE=virbr1
TYPE=Bridge
BOOTPROTO=static
IPADDR=192.168.200.200  #桥接的网桥IP肯定和宿主机要同一网段
NETMASK=255.255.255.0
GATEWAY=192.168.200.2
DNS1=192.168.200.2
ONBOOT=yes

#让物理网卡配置文件可以识别网桥virbr1
[root@localhost network-scripts]# cat ifcfg-ens32 
TYPE=Ethernet
BOOTPROTO=dhcp
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens32
UUID=17fb5987-5317-4bca-8514-9e1b73933184
DEVICE=ens32
ONBOOT=yes
BRIDGE=virbr1           #增加本条配置语句

#重启宿主机的网络模式
[root@localhost network-scripts]# service network restart
```

> 重启后，同学们会发现，很大概率我们的xshell已经掉线了。

![QQ截图20180525223603.png-49.7kB](http://static.zybuluo.com/chensiqi/e4b1ragn0q6d6cdylpu87ipp/QQ%E6%88%AA%E5%9B%BE20180525223603.png)

> xshell连不上了，怎么办呢？我们现在修改xshell的连接配置。我们去连接virbr1网桥的IP地址。

![QQ截图20180525224117.png-40.1kB](http://static.zybuluo.com/chensiqi/60ruz3bfye7czgyb9z0qoop6/QQ%E6%88%AA%E5%9B%BE20180525224117.png)

> 打开virt-manager图形模式，我们查看网络接口情况

![QQ截图20180525224935.png-26.3kB](http://static.zybuluo.com/chensiqi/70w3kjybt501kxij2muiam2q/QQ%E6%88%AA%E5%9B%BE20180525224935.png)

> 打开虚拟机此时我们发现之前NAT模式连接的虚拟机已经无法连接外网了

![QQ截图20180525230153.png-28.2kB](http://static.zybuluo.com/chensiqi/qyep69trdzkdyh06ot6hm8ju/QQ%E6%88%AA%E5%9B%BE20180525230153.png)

> 我们利用virt-manager图形界面将虚拟机的网卡连接模式切换到桥接模式桥接virbr1

![QQ截图20180525230300.png-54.6kB](http://static.zybuluo.com/chensiqi/1kvjr7xj6uzwo59s8tapgqy9/QQ%E6%88%AA%E5%9B%BE20180525230300.png)

> 进入虚拟机我们重启网卡的配置文件

![QQ截图20180525230508.png-36.3kB](http://static.zybuluo.com/chensiqi/iq0wu1e3pc4i55p70nqdiu98/QQ%E6%88%AA%E5%9B%BE20180525230508.png)

> 此时我们的外部机器也可以ping通我们的桥接的虚拟机了

![QQ截图20180525230930.png-23.1kB](http://static.zybuluo.com/chensiqi/bff4hd2wem818owgqzp2yt4i/QQ%E6%88%AA%E5%9B%BE20180525230930.png)

#### 6.3.2 通过图形界面进行网桥virbr1的创建

```
[root@localhost ~]# brctl show
bridge name bridge id       STP enabled interfaces
virbr0      8000.52540079e341   yes     virbr0-nic
                                        vnet1
                                        vnet2
virbr1      8000.000c29963ac5   no      ens32
                                        vnet0       #之前本来插在virbr0上的网卡（NAT模式），如今插到virbr1上（桥接模式）
```

> 现在我们还原一下virbr1的配置文件。（移走virbr0，还原ens32网卡配置文件等）
>  如果同学们还原后，发现virbr1还在，也没关系，那是还没清空的缓存导致。
>  只要图形界面下没有了即可。

![QQ截图20180525233152.png-20.4kB](http://static.zybuluo.com/chensiqi/81tbb0q91lcuck4mo5iy8bku/QQ%E6%88%AA%E5%9B%BE20180525233152.png)

**图形化创建virbr1**

![311.png-20.7kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/311.png)

![312.png-11.2kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/312.png)

![313.png-22.3kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/313.png)

![314.png-6.2kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/314.png)

![315.png-20.3kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/315.png)

```
#验证virbr1配置
[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens32 #我们发现网卡配置文件已经改变
DEVICE=ens32
ONBOOT=yes
BRIDGE="virbr1"
[root@localhost ~]# cat /etc/sysconfig/network-scripts/ifcfg-virbr1     #网桥文件被自动创建
DEVICE="virbr1"
ONBOOT="yes"
TYPE="Bridge"
BOOTPROTO="dhcp"
PEERDNS="yes"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
DHCPV6C="no"
STP="on"
DELAY="0.0"
[root@localhost ~]# brctl show
bridge name bridge id       STP enabled interfaces
virbr0      8000.52540079e341   yes     virbr0-nic
                            vnet0
                            vnet1
                            vnet2
virbr1      8000.000c29963ac5   no      ens32
[root@localhost ~]# service network restart #重启网络，激活virbr1网桥配置
Restarting network (via systemctl):                        [  确定  ]
[root@localhost ~]# brctl show
bridge name bridge id       STP enabled interfaces
virbr0      8000.52540079e341   yes     virbr0-nic
                            vnet0
                            vnet1
                            vnet2
virbr1      8000.000c29963ac5   yes     ens32   #已经激活
[root@localhost ~]# ifconfig ens32      #查看物理网卡IP已经消失
ens32: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::20c:29ff:fe96:3ac5  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:96:3a:c5  txqueuelen 1000  (Ethernet)
        RX packets 1146650  bytes 107636905 (102.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1877446  bytes 3852832284 (3.5 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@localhost ~]# ifconfig virbr1     #查看网桥IP，已经出现
virbr1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.200.132  netmask 255.255.255.0  broadcast 192.168.200.255
        inet6 fe80::20c:29ff:fe96:3ac5  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:96:3a:c5  txqueuelen 1000  (Ethernet)
        RX packets 164  bytes 11098 (10.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 101  bytes 16483 (16.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

> 在工作中，我们通过网桥连接的物理网卡通常都是单点的，因此，我们需要将网桥绑定到多块物理网卡上，这就需要我们学习物理网卡绑定的相关知识。

### 6.4 配置网卡绑定

![QQ截图20180525234951.png-383kB](http://static.zybuluo.com/chensiqi/n3i355x8sgz5nxn5ba628amd/QQ%E6%88%AA%E5%9B%BE20180525234951.png)

**实验：配置多网卡绑定的KVM桥接模式**

- [x] 绑定网卡
  - 启用Bonding
  - 配置物理网卡
  - 配置绑定接口
  - 重新启动服务
  - 测试
- [x] 配置网桥

![1.png-151.2kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/1.png)

![2.png-73.5kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/2.png)

**实操（1）：创建多网卡bond**

> 我们给KVM宿主机多天加一块网卡，并删除virbr1网桥还原ens32网卡的初始设置

```
#查看两块网卡的初始配置
[root@localhost ~]# ifconfig ens32
ens32: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.200.132  netmask 255.255.255.0  broadcast 192.168.200.255
        inet6 fe80::d302:4c4f:17a0:b161  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:96:3a:c5  txqueuelen 1000  (Ethernet)
        RX packets 24084  bytes 13474369 (12.8 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 323  bytes 28440 (27.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@localhost ~]# ifconfig ens36
ens36: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.200.136  netmask 255.255.255.0  broadcast 192.168.200.255
        inet6 fe80::7d41:4e00:f272:89d2  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:96:3a:cf  txqueuelen 1000  (Ethernet)
        RX packets 11580  bytes 1266636 (1.2 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 12852  bytes 22770539 (21.7 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

#查看网池基本情况
[root@localhost ~]# virsh net-list
 名称               状态     自动开始  持久
----------------------------------------------------------
 default              活动     是           是

#查看默认网池的xml配置信息
[root@localhost ~]# virsh net-dumpxml default
<network connections='2'>
  <name>default</name>
  <uuid>5687d2e1-c14d-42bb-abe2-fcb4bfac2a12</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:79:e3:41'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>

#查看KVM宿主机的物理网络接口信息
[root@localhost network-scripts]# virsh iface-list
 名称               状态     MAC 地址
---------------------------------------------------
 ens32                活动     00:0c:29:96:3a:c5
 lo                   活动     00:00:00:00:00:00
```

> 我们发现在查看网络接口时并没有新添加进来的ens36的网卡信息，这是因为还没有相应的网络接口配置文件，我们可以选择手动在/etc/sysconfig/network-scripts/目录下创建，也可以通过virt-manager创建。

**通过virt-manager创建网卡接口文件**

![441.png-20.3kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/441.png)

![442.png-11.5kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/442.png)

![443.png-20kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/443.png)

```
[root@localhost network-scripts]# ls ifcfg-ens32 ifcfg-ens36
ifcfg-ens32  ifcfg-ens36    #已经有了
[root@localhost network-scripts]# cat ifcfg-ens36   
DEVICE="ens36"
HWADDR="00:0c:29:96:3a:cf"
ONBOOT="yes"
BOOTPROTO="dhcp"
[root@localhost network-scripts]# virsh iface-list
 名称               状态     MAC 地址
---------------------------------------------------
 ens32                活动     00:0c:29:96:3a:c5
 ens36                活动     00:0c:29:96:3a:cf    #已经有了
 lo                   活动     00:00:00:00:00:00
```

**查看kernel是否支持网卡绑定**

```
#查看是否支持网卡绑定
[root@localhost network-scripts]# lsmod | grep bonding  

#激活内核网卡绑定模块
[root@localhost network-scripts]# modprobe --first-time bonding 
[root@localhost network-scripts]# lsmod | grep bonding
bonding               141566  0 

#备份网卡配置文件
[root@localhost network-scripts]# cp ifcfg-ens* bak/

#修改ens32和ens36网卡配置文件让band0绑定接口为主，他们为从
[root@localhost network-scripts]# vim ifcfg-ens32
[root@localhost network-scripts]# cat ifcfg-ens32
TYPE=Ethernet
BOOTPROTO=none
NAME=ens32
DEVICE=ens32
ONBOOT=yes
MASTER=bond0
SLAVE=yes
NM_CONTROLLED=no
USERCTL=NO
[root@localhost network-scripts]# vim ifcfg-ens36
[root@localhost network-scripts]# cp ifcfg-ens32 ifcfg-ens36
cp：是否覆盖"ifcfg-ens36"？ y
[root@localhost network-scripts]# vim ifcfg-ens36
[root@localhost network-scripts]# cat ifcfg-ens36
TYPE=Ethernet
BOOTPROTO=none
NAME=ens36
DEVICE=ens36
ONBOOT=yes
MASTER=bond0
SLAVE=yes
NM_CONTROLLED=no
USERCTL=NO

#创建bond0绑定接口配置文件
[root@localhost network-scripts]# vim ifcfg-bond0
[root@localhost network-scripts]# cat ifcfg-bond0 
DEVICE=bond0
ONBOOT=yes
NM_CONTROLLED=no
USERCTL=no
BONDING_OPTS="mode=1 miimon=100"   #mode=1是主备模式，两块从卡不同时生效
BOOTPROTO=static
IPADDR=192.168.200.132
NETMASK=255.255.255.0

#重新启动网络服务
[root@localhost ~]# service network restart

#查看bond0绑定接口
[root@localhost ~]# ifconfig bond0
bond0: flags=5187<UP,BROADCAST,RUNNING,MASTER,MULTICAST>  mtu 1500
        inet 192.168.200.132  netmask 255.255.255.0  broadcast 192.168.200.255
        inet6 fe80::20c:29ff:fe96:3ac5  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:96:3a:c5  txqueuelen 1000  (Ethernet)
        RX packets 186  bytes 16691 (16.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 122  bytes 22238 (21.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

#查看bond0详细信息
[root@localhost ~]# cat /proc/net/bonding/bond0 
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup)   #mode=1的模式==>主被动模式
Primary Slave: None
Currently Active Slave: ens32   #当前活动中的网卡ens32
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: ens32
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 00:0c:29:96:3a:c5
Slave queue ID: 0

Slave Interface: ens36
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 00:0c:29:96:3a:cf
Slave queue ID: 0
```

**测试绑定中的网卡：**

> 我们断开ens32的网卡

![445.png-31.6kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/445.png)

```
[root@localhost ~]# cat /proc/net/bonding/bond0 
Ethernet Channel Bonding Driver: v3.7.1 (April 27, 2011)

Bonding Mode: fault-tolerance (active-backup)
Primary Slave: None
Currently Active Slave: ens36   #ens36被启动了
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0

Slave Interface: ens32
MII Status: down            #ens32 down了
Speed: Unknown
Duplex: Unknown
Link Failure Count: 1
Permanent HW addr: 00:0c:29:96:3a:c5
Slave queue ID: 0

Slave Interface: ens36
MII Status: up
Speed: 1000 Mbps
Duplex: full
Link Failure Count: 1
Permanent HW addr: 00:0c:29:96:3a:cf
Slave queue ID: 0
```

> 我们用windows ping KVM宿主机，仍然能通

![446.png-10.5kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/446.png)

**实操（2）：搭建bond的KVM网桥**

![406.png-41.2kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/406.png)

```
#验证网桥virbr1状态
[root@localhost ~]# brctl show
bridge name bridge id       STP enabled interfaces
virbr0      8000.52540079e341   yes     virbr0-nic
                            vnet0
                            vnet1
virbr1      8000.000c29963ac5   no      bond0
[root@localhost ~]# ifconfig bond0
bond0: flags=5187<UP,BROADCAST,RUNNING,MASTER,MULTICAST>  mtu 1500
        inet6 fe80::20c:29ff:fe96:3ac5  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:96:3a:c5  txqueuelen 1000  (Ethernet)
        RX packets 3717  bytes 425857 (415.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 7004  bytes 12720203 (12.1 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@localhost ~]# ifconfig virbr1
virbr1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.200.132  netmask 255.255.255.0  broadcast 192.168.200.255
        inet6 fe80::20c:29ff:fe96:3ac5  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:96:3a:c5  txqueuelen 1000  (Ethernet)
        RX packets 3667  bytes 372147 (363.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2354  bytes 7601615 (7.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

> **特别提示**，如果利用图形化方式添加桥接网桥virbr1时遇到如下问题，可尝试在网卡配置文件中加入NM_CONTROLLED=no和USERCTL=NO解决（ens32，ens36，bond0），如果还未解决，可能是由于之前频繁做实验搭建过多次同名网桥，系统留有缓存，请尝试reboot重启操作系统。

![447.png-4.4kB](3-KVM%E7%AE%A1%E7%90%86%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C.assets/447.png)

