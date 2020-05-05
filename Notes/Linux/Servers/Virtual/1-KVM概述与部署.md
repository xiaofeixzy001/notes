[TOC]

## 一，KVM概述

### 1.1 虚拟化概述

> 在计算机技术中，虚拟化意味着创建设备或资源的虚拟版本，如服务器，存储设备，网络或者操作系统等等
>
> - [x] 虚拟化技术分类：
>   - 系统虚拟化（我们主要讨论的反向）
>   - 存储虚拟化（raid，lvm）
>   - 网络虚拟化（sdn）
>   - GPU虚拟化（比特币）
>   - 软件虚拟化
>   - 硬件支持虚拟化

#### 1.1.1 系统虚拟化

- 这种虚拟化通常表现为在单一系统上运行多个操作系统
- 这些虚拟操作系统同时运行，每个操作系统又是相互独立

![QQ截图20180329205226.png-74.4kB](http://static.zybuluo.com/chensiqi/h8t13ui6x2n2amn1tctno689/QQ%E6%88%AA%E5%9B%BE20180329205226.png)

#### 1.1.2 虚拟化的三种实现方式

（1）纯软件仿真

> - 通过模拟完整的硬件环境来虚拟化用户平台
> - 模拟X86，ARM，PowerPC等多种CPU
> - 效率比较低
> - QEMU，Bochs，PearPC

（2）虚拟化层翻译

> - 多数的虚拟化而采用虚拟机管理程序Hypervisor
> - Hypervisor是一个软件层或子系统
>   - 也称为VMM（Virtual Machine Monitor，虚拟机监控器）
> - 允许多种操作系统在相同的物理系统中运行
> - 控制硬件并向用户操作系统提供访问底层硬件的途径
> - 向来宾操作系统提供虚拟化的硬件

![QQ截图20180329222240.png-56.1kB](http://static.zybuluo.com/chensiqi/a9t6hzq1zc1e34o62ghp11pm/QQ%E6%88%AA%E5%9B%BE20180329222240.png)

![QQ截图20180329224952.png-323kB](http://static.zybuluo.com/chensiqi/wcgb4a5dd077mx5y5u5kh2x4/QQ%E6%88%AA%E5%9B%BE20180329224952.png)

**无硬件辅助的全虚拟化**

> - 基于二进制翻译的全虚拟化
> - Hypervisor运行在Ring 0
> - Guest OS运行在Ring 1
> - 机制：异常，捕获，翻译
> - 示例：
>   - VMware Workstation
>   - QWMU
>   - Virtual PC

![QQ截图20180329225949.png-90.3kB](http://static.zybuluo.com/chensiqi/z37cz8wy8504xyjo2v285xg0/QQ%E6%88%AA%E5%9B%BE20180329225949.png)

**硬件辅助的全虚拟化**

> - Intel VT 和 AMD-V创建一个新的Ring -1 单独给Hypervisor使用
> - Guest OS可以直接使用Ring 0 而无须修改
> - 示例：
>   - VMware ESXi
>   - Microsoft Hyper-V
>   - Xen3.0
>   - KVM

![QQ截图20180329225949.png-105.5kB](http://static.zybuluo.com/chensiqi/yq1qkirdrq5w6afm9ejvukjh/QQ%E6%88%AA%E5%9B%BE20180329225949.png)

![QQ截图20180330084828.png-169.7kB](http://static.zybuluo.com/chensiqi/uk2bdwbr2fkbale0dry55dv1/QQ%E6%88%AA%E5%9B%BE20180330084828.png)

（3）容器技术

### 1.2 KVM概述与相关参考资料

## 二，KVM安装

### 2.1 实现环境准备

CentOS7.3DVD镜像下载地址：http://man.linuxde.net/download/CentOS_7_3

#### 2.1.1 生产环境硬件配置

> - CPU必须支持虚拟化技术，在BIOS设置为启动
> - 目前，多数服务器基础桌面计算机均处理启动状态

![QQ截图20180330091735.png-236.3kB](http://static.zybuluo.com/chensiqi/h5rvun28hu30kgbiuiy37z69/QQ%E6%88%AA%E5%9B%BE20180330091735.png)

#### 2.1.2 实验准备

> 我们需要先用虚拟机，然后在虚拟机里再用虚拟化，也就是嵌套虚拟化
>
> - VMware 嵌套虚拟化
>   - 产品：Workstation，Player，ESXi
>   - 支持：ESXi，Hyper-V，KVM，Xen

![QQ截图20180330091735.png-204.6kB](http://static.zybuluo.com/chensiqi/vtjaq0u6bdhbrdtporcg8esz/QQ%E6%88%AA%E5%9B%BE20180330091735.png)

![QQ截图20180330092445.png-23.6kB](http://static.zybuluo.com/chensiqi/26lyjcjqtnxo50kjpzfrhmma/QQ%E6%88%AA%E5%9B%BE20180330092445.png)

### 2.2 KVM安装

> 装机时虚拟机需要安装如下软件

![QQ截图20180330094613.png-168.2kB](http://static.zybuluo.com/chensiqi/q0u0zl2xjsi5b58qe3j8iz06/QQ%E6%88%AA%E5%9B%BE20180330094613.png)

> 进入后，ifconfig我们发现

![QQ截图20180330094613.png-16.7kB](http://static.zybuluo.com/chensiqi/gw0sfhl456t1jzkatnfvto9e/QQ%E6%88%AA%E5%9B%BE20180330094613.png)

> 网卡并没有IP地址，我们可以通过如下操作，打开网卡配置文件的ONBOOT

```
[root@localhost network-scripts]# pwd
/etc/sysconfig/network-scripts
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
#NBOOT=yes          #打开这个
```

> 然后重启网络服务

![QQ截图20180330094613.png-18.2kB](http://static.zybuluo.com/chensiqi/hj4qyee7kequk2j3sdsijr3j/QQ%E6%88%AA%E5%9B%BE20180330094613.png)

#### 2.2.1 解决CentOS7.3的Xshell连接很慢的问题

```
#将ssh配置文件修改成如下所示
[root@localhost ~]# sed -n '93p;129p' /etc/ssh/sshd_config
GSSAPIAuthentication no
UseDNS no

#重启动服务
[root@localhost ~]# systemctl restart sshd
```

#### 2.2.2 解决Centos7.3重启卡在license information

> 如果出现license information（license not accepted），即说明需要同意许可信息，输入1-回车-2-回车-c-回车-c-回车。即可解决

#### 2.2.3 搭建本地yum仓库光盘源，安装软件包

```
#搭建本地光盘源yum仓库
[root@localhost ~]# cd /etc/yum.repos.d/
[root@localhost yum.repos.d]# ls
CentOS-Base.repo       CentOS-fasttrack.repo  CentOS-Vault.repo
CentOS-CR.repo         CentOS-Media.repo
CentOS-Debuginfo.repo  CentOS-Sources.repo
[root@localhost yum.repos.d]# mkdir bak
[root@localhost yum.repos.d]# mv C* bak/
[root@localhost yum.repos.d]# vim local.repo
[root@localhost yum.repos.d]# cat local.repo 
[local]
name=local
baseurl=file:///media/cdrom/
gpgcheck=0
enabled=1
[root@localhost yum.repos.d]# mount /dev/sr0 /media/cdrom/
mount: /dev/sr0 写保护，将以只读方式挂载
[root@localhost yum.repos.d]# yum -y clean all
已加载插件：fastestmirror, langpacks
正在清理软件源： local
Cleaning up everything
Cleaning up list of fastest mirrors
[root@localhost yum.repos.d]# yum makecache
已加载插件：fastestmirror, langpacks
local                                                | 3.6 kB     00:00     
(1/4): local/filelists_db                              | 3.0 MB   00:00     
(2/4): local/group_gz                                  | 155 kB   00:00     
(3/4): local/other_db                                  | 1.3 MB   00:00     
(4/4): local/primary_db                                | 3.0 MB   00:00     
Determining fastest mirrors
元数据缓存已建立
```

> 我们的装机方式已经安装了如下软件组

```
@base
@core
@virtualization-hypervisor              #虚拟化主机选项
@virtualization-platform                #虚拟化平台选项
@virtualization-tools                   #虚拟化主机选项
```

> 我们还需要增加如下软件包

```
@virtualization-client
@gnome-desktop
#yum安装包组
[root@localhost ~]# yum -y group install virtualization-client
[root@localhost ~]# yum -y group install gnome-desktop
```

#### 2.2.3 修改虚拟化引擎配置并检查CPU特性

![QQ截图20180330094613.png-18.7kB](http://static.zybuluo.com/chensiqi/gqnpg96fn7nesqvbtkibf0yb/QQ%E6%88%AA%E5%9B%BE20180330094613.png)

```
#检查CPU特性
[root@localhost ~]# grep vmx /proc/cpuinfo
flags       : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts nopl xtopology tsc_reliable nonstop_tsc aperfmperf eagerfpu pni pclmulqdq vmx ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch ida arat epb pln pts dtherm hwp hwp_noitfy hwp_act_window hwp_epp tpr_shadow vnmi ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 invpcid rtm rdseed adx smap xsaveopt

[root@localhost ~]# egrep '^flags.*(vmx|svm)' /proc/cpuinfo 
flags       : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts nopl xtopology tsc_reliable nonstop_tsc aperfmperf eagerfpu pni pclmulqdq vmx ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm 3dnowprefetch ida arat epb pln pts dtherm hwp hwp_noitfy hwp_act_window hwp_epp tpr_shadow vnmi ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 invpcid rtm rdseed adx smap xsaveopt
```

### 2.3 KVM远程管理

```
[root@localhost ~]# which virsh #查看虚拟机管理命令
/usr/bin/virsh
[root@localhost ~]# virsh list --all    #查看所有虚拟机
 Id    名称                         状态
----------------------------------------------------

[root@localhost ~]# startx    切换到图形界面模式
```

> 在图形界面下选择左上角Application

![QQ截图20180330094613.png-122.4kB](http://static.zybuluo.com/chensiqi/08lmjcxbg83fp6rjcucokx04/QQ%E6%88%AA%E5%9B%BE20180330094613.png)

![QQ截图20180330094613.png-138.4kB](http://static.zybuluo.com/chensiqi/8func3tnouiwzpvoaq1qw04x/QQ%E6%88%AA%E5%9B%BE20180330094613.png)

> 但是，我们在工作中通常需要远程来管理KVM的环境。而SSH方式是看不到桌面模式的。

#### 2.3.1 KVM的两种远程管理方式

（1）SSH图形化显示

> windows安装软件x-manager。然后xshell软件开启X11转发

![QQ截图20180410193227.png-22.9kB](http://static.zybuluo.com/chensiqi/b8zd3cndeaaaas27sr8g1gop/QQ%E6%88%AA%E5%9B%BE20180410193227.png)

> 然后我们连接上虚拟机以后，输入virt-manager出现下图

![QQ截图20180410193416.png-43kB](http://static.zybuluo.com/chensiqi/tgfbzoukllxu2qkks848na4e/QQ%E6%88%AA%E5%9B%BE20180410193416.png)

（2）VNC图形化显示

> VNC是一个优秀的远程管理软件，它有两部分组成VNCServer，VNCViewer。

```
#看一下系统里是否有必须的包
[root@localhost ~]# rpm -qa | grep vnc
tigervnc-license-1.3.1-9.el7.noarch     #必须的
gtk-vnc2-0.5.2-7.el7.x86_64
gvnc-0.5.2-7.el7.x86_64
tigervnc-server-minimal-1.3.1-9.el7.x86_64  #必须的
#安装vnc-server
[root@localhost ~]# yum -y install tigervnc-server  #安装服务端软件包
[root@localhost ~]# cat /etc/sysconfig/vncservers   #查看vnc配置文件
# THIS FILE HAS BEEN REPLACED BY /lib/systemd/system/vncserver@.service
[root@localhost ~]# ll /lib/systemd/system/vncserver@.service   #原来这才是配置文件
-rw-r--r--. 1 root root 1880 11月 16 2016 /lib/systemd/system/vncserver@.service

#然后我们需要创建vnc密码
[root@localhost ~]# vncpasswd
Password:
Verify:

#启动vnc-server
[root@localhost ~]# vncserver

New 'localhost.localdomain:1 (root)' desktop is localhost.localdomain:1

Creating default startup script /root/.vnc/xstartup
Starting applications specified in /root/.vnc/xstartup
Log file is /root/.vnc/localhost.localdomain:1.log

[root@localhost ~]# ps aux | grep vnc
root       6241  0.9  1.6 250192 31032 pts/0    Sl   10:07   0:00 /usr/bin/vnc :1 -desktop localhost.localdomain:1 (root) -auth /root/.Xauthority -geometry 1024x768 -rfbwait 30000 -rfbauth /root/.vnc/passwd -rfbport 5901 -fp catalogue:/etc/X11/fontpath.d -pn
root       6250  0.0  0.2  96716  4068 pts/0    S    10:07   0:00 /usr/bin/vncconfig -iconic
root       6991  0.0  0.0 112668   972 pts/0    S+   10:07   0:00 grep --color=auto vnc
```

**然后我们需要关闭Centos7.3的防火墙**

```
[root@localhost ~]# service firewalld stop  #不然vnc客户端连接不上
Redirecting to /bin/systemctl stop  firewalld.service
[root@localhost ~]# systemctl disable firewalld.service #永久关闭防火墙
```

> 接下来我们在windows主机上安装vnc客户端

https://www.realvnc.com/en/connect/download/viewer/ 可以下载vnc viewer

![QQ截图20180402103131.png-1.8kB](http://static.zybuluo.com/chensiqi/7iub2uh17q6eowk90co902cq/QQ%E6%88%AA%E5%9B%BE20180402103131.png)

![QQ截图20180402103827.png-25.1kB](http://static.zybuluo.com/chensiqi/75sgb5anjm95er5jkvr1421p/QQ%E6%88%AA%E5%9B%BE20180402103827.png)

![QQ截图20180402104005.png-15.9kB](http://static.zybuluo.com/chensiqi/00y3ffempsdden2104zfb14w/QQ%E6%88%AA%E5%9B%BE20180402104005.png)

![QQ截图20180402104026.png-9.6kB](http://static.zybuluo.com/chensiqi/mxqfeep20ii1a821dr219t7w/QQ%E6%88%AA%E5%9B%BE20180402104026.png)

![QQ截图20180402104113.png-537.3kB](http://static.zybuluo.com/chensiqi/lluh19r7pnrp99cdjdf0dd54/QQ%E6%88%AA%E5%9B%BE20180402104113.png)

## 三，创建虚拟机

### 3.1 使用virt-manager创建虚拟机

- [x] virt-manager 基本使用
- [x] 实验
  - 环境准备
  - 创建Windows虚拟机
  - 创建Linux虚拟机

![QQ截图20180402111815.png-42.8kB](http://static.zybuluo.com/chensiqi/4vjlv2edfieoqrolt0dd2ssw/QQ%E6%88%AA%E5%9B%BE20180402111815.png)

（1）我们需要添加一块80G的硬盘来存储操作系统的安装介质，ISO文件

![QQ截图20180403220908.png-28.3kB](http://static.zybuluo.com/chensiqi/pyh2kas2r1n14fdfpqz96qu0/QQ%E6%88%AA%E5%9B%BE20180403220908.png)

（2）利用fdisk分出一块40G的分区

```
[root@localhost ~]# ll /dev/sdb*
brw-rw----. 1 root disk 8, 16 Apr  4 08:48 /dev/sdb
brw-rw----. 1 root disk 8, 17 Apr  4 08:48 /dev/sdb1    #40G
```

（3）创建LVM逻辑卷

```
[root@localhost ~]# ll /dev/sdb*
brw-rw----. 1 root disk 8, 16 Apr  4 08:48 /dev/sdb
brw-rw----. 1 root disk 8, 17 Apr  4 08:48 /dev/sdb1
[root@localhost ~]# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
[root@localhost ~]# vgcreate vmvg /dev/sdb1
  Volume group "vmvg" successfully created
[root@localhost ~]# vgs
  VG   #PV #LV #SN Attr   VSize  VFree 
  cl     1   2   0 wz--n- 19.00g     0 
  vmvg   1   0   0 wz--n- 40.00g 40.00g
[root@localhost ~]# vgdisplay       #查看vg详细
  --- Volume group ---
  VG Name               cl
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               19.00 GiB
  PE Size               4.00 MiB
  Total PE              4863
  Alloc PE / Size       4863 / 19.00 GiB
  Free  PE / Size       0 / 0   
  VG UUID               6ROh78-78oc-QfPu-1YnI-pW76-TiFa-4kWjSQ
   
  --- Volume group ---
  VG Name               vmvg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               40.00 GiB
  PE Size               4.00 MiB
  Total PE              10239
  Alloc PE / Size       0 / 0   
  Free  PE / Size       10239 / 40.00 GiB       #vmvg可用的PE一共10239
  VG UUID               Hxeycr-8UEv-qiF2-JTZi-txEl-E5G7-Iho3x5
   
[root@localhost ~]# lvcreate -n lvvm1 -l 10239 vmvg     #将vmvg可以用PE全部分配给lvvm1
  Logical volume "lvvm1" created.
[root@localhost ~]# lvs
  LV    VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root  cl   -wi-ao---- 17.00g                                                    
  swap  cl   -wi-ao----  2.00g                                                    
  lvvm1 vmvg -wi-a----- 40.00g                                                    
[root@localhost ~]# mkfs.ext4 /dev/vmvg/lvvm1   格式化lvvm1
```

（4）挂载逻辑卷

```repl
[root@localhost ~]# mkdir /vm
[root@localhost ~]# mount /dev/vmvg/lvvm1 /vm/
[root@localhost ~]# df -hT
Filesystem             Type      Size  Used Avail Use% Mounted on
/dev/mapper/cl-root    xfs        17G  3.7G   14G  22% /
devtmpfs               devtmpfs  901M     0  901M   0% /dev
tmpfs                  tmpfs     912M     0  912M   0% /dev/shm
tmpfs                  tmpfs     912M  8.9M  903M   1% /run
tmpfs                  tmpfs     912M     0  912M   0% /sys/fs/cgroup
/dev/sda1              xfs      1014M  144M  871M  15% /boot
tmpfs                  tmpfs     183M     0  183M   0% /run/user/0
/dev/mapper/vmvg-lvvm1 ext4       40G   49M   38G   1% /vm
[root@localhost ~]# echo "mount /dev/vmvg/lvvm1 /vm/" >> /etc/rc.local
```

（5）创建iso镜像文件存放目录

```
[root@localhost ~]# mkdir /ios
root@localhost ~]# cd /iso/
[root@localhost iso]# ls
CentOS-6.5-x86_64-bin-DVD1.iso
```

**将光盘安装镜像文件上传到/ios目录下：**

![QQ截图20180404091939.png-20.8kB](http://static.zybuluo.com/chensiqi/bdoix4svtbgkyb7hm2duqler/QQ%E6%88%AA%E5%9B%BE20180404091939.png)

#### 3.1.1 virt-manager基本使用

- [x] 启动virt-manager
- [x] 虚拟机管理主窗口
- [x] 硬件细节窗口
  - 配置虚拟机启动选项
  - 附加USB设备给虚拟机
  - 准备工作
  - USB重定向
- [x] 虚拟机图形控制台
- [x] 添加远程连接
- [x] 显示虚拟机细节
- [x] 性能监视

（1）使用向导的默认配置来创建虚拟机

启动VNC远程管理程序连接Linux，打开Virtual Machine Manager

![1.png-343.1kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/1.png)

点击创建新的虚拟机

![2.png-17.4kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/2.png)

选择本地安装iso镜像

![3.png-40.8kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/3.png)

![4.png-28.2kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/4.png)

![5.png-39.3kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/5.png)

![6.png-51.8kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/6.png)

![7.png-28.7kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/7.png)

![1.png-42.5kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/1-1577242653855.png)

![2.png-23.2kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/2-1577242653745.png)

![1.png-24.4kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/1-1577242653755.png)

![1.png-30.1kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/1-1577242653825.png)

最后新建的虚拟机自动进入装机状态

> 同学们选择全英文，最小化装机即可。由于咱们是嵌套的虚拟化，装机图形界面可能稍微有点卡。不过没关系，等一下就好。

![1.png-15.9kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/1-1577242653835.png)

**特别提示：**

> 当我们以virt-manager进行手动管理创建虚拟机时，有可能在进入安装操作系统界面时大几率遭遇到键盘失灵的情况。如果同学们遇到这个问题，不要着急，这是因为字符集混乱识别的问题，我们需要调整一下虚拟机的设置后，即可恢复。
>  我们做如下调整即可。

![123.png-74.5kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/123.png)

![125.png-76.3kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/125.png)

![124.png-73kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/124.png)

> 然后我们正常开机就会进入装机界面，你会发现键盘的操作恢复了。。。

（2）以自定义规划方式创建虚拟机

> 我们发现按照向导的默认方式安装虚拟机，虚拟机的磁盘并没有放在我们规划好的目录里

![1.png-46.8kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/1-1577242654054.png)

> 因此在工作中，我们需要在安装过程中进行自定义存储池的操作，步骤如下

![1.png-33.4kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/1-1577242654136.png)

![2.png-25.5kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/2-1577242654254.png)

![2.png-34.7kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/2-1577242654154.png)

![3.png-21.7kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/3-1577242654255.png)

![4.png-19.1kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/4-1577242654234.png)

![5.png-67.9kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/5-1577242654305.png)

![6.png-17.2kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/6-1577242654285.png)

> 到此我们新的VM存储池就创建完了，但是在存储池里我们还需要创建一个Volume卷（磁盘）

![22.png-32.6kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/22.png)

![33.png-28.9kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/33.png)

![44.png-34.7kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/44.png)

![1.png-28.9kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/1-1577242654525.png)

![2.png-33.8kB](1-KVM%E6%A6%82%E8%BF%B0%E4%B8%8E%E9%83%A8%E7%BD%B2.assets/2-1577242654555.png)

> 然后就进入操作系统的装机界面了。装完机以后我们查看，磁盘卷位置已经修改。

![QQ截图20180404212916.png-41.5kB](http://static.zybuluo.com/chensiqi/zwmpooo0s0pjt7mvvs61xgkf/QQ%E6%88%AA%E5%9B%BE20180404212916.png)

### 3.2 使用virt-install创建虚拟机

```
#创建一块虚拟机的存储磁盘
[root@localhost ~]# qemu-img create -f qcow2 /vm/chensiqi.qcow2 10G #qcow2格式磁盘 /vm/chensiqi.qcow2磁盘位置 10G为磁盘大小
Formatting '/vm/chensiqi.qcow2', fmt=qcow2 size=10737418240 encryption=off cluster_size=65536 lazy_refcounts=off 
[root@localhost ~]# ll -h /vm/
总用量 1.1G
-rw-------. 1 qemu qemu 8.1G 4月   9 12:26 centos6.5-2.qcow2    #真占了8G
-rw-r--r--  1 qemu qemu 193K 4月   9 12:09 chensiqi.qcow2       #只是小文件
drwx------  2 root root  16K 4月   4 09:03 lost+found
```

> 我们通过qemu-img来创建的磁盘在最初只是个小文件，直到磁盘空间被占满。
>  然而我们通过virt-manager创建的磁盘却真的占了8G

```
#创建一个虚拟机
[root@localhost ~]# virt-install \  #创建虚拟机命令
> --name=vm1 \  #虚拟机的名字
> --disk path=/vm/chensiqi.qcow2 \  #虚拟磁盘路径
> --vcpus=1 --ram=1024 \        #分配的CPU和内存大小
> --cdrom=/iso/CentOS-6.5-x86_64-bin-DVD1.iso \ #系统安装盘位置
> --network network=default \   #网络模式，default为NAT
> --graphics vnc,listen=0.0.0.0 \   #虚拟机的显示类型为VNC
> --os-type=linux \
> --os-variant=rhel6
```

> 执行上边的命令就会弹出Virt Viewer的窗口，进入装机界面

![QQ截图20180410200507.png-456.4kB](http://static.zybuluo.com/chensiqi/yjkbzlw940q216ybckcqj1eh/QQ%E6%88%AA%E5%9B%BE20180410200507.png)

### 3.3 半虚拟化驱动virtio

#### 3.3.1 使用半虚拟化驱动virtio的目的

> 没有virtio的全虚拟化的设备访问路径如下图所示：

![QQ截图20180410200507.png-157.1kB](http://static.zybuluo.com/chensiqi/emv2yenzggicisdhdxwremwp/QQ%E6%88%AA%E5%9B%BE20180410200507.png)

> 拥有virtio的全虚拟化的设备访问路径如下图所示：

![QQ截图20180410200507.png-152.5kB](http://static.zybuluo.com/chensiqi/yszmjhmmmx59lazyv7l7o7rl/QQ%E6%88%AA%E5%9B%BE20180410200507.png)

#### 3.3.2 virtio的半虚拟化设备统一接口原则

![QQ截图20180410200507.png-151.1kB](http://static.zybuluo.com/chensiqi/velill9fmse0bqjmjpvlkvxj/QQ%E6%88%AA%E5%9B%BE20180410200507.png)

#### 3.3.3 Linux虚拟机直接选择virtio半虚拟化驱动设备

![QQ截图20180410210541.png-59.1kB](http://static.zybuluo.com/chensiqi/5r2mvi2x92nee76fyzpre5cq/QQ%E6%88%AA%E5%9B%BE20180410210541.png)

![QQ截图20180410210635.png-61.3kB](http://static.zybuluo.com/chensiqi/7v7yd8513dyf37t7kl2yj2a0/QQ%E6%88%AA%E5%9B%BE20180410210635.png)

### 3.4 QEMU Guest Agent

> - [x] 如果VM中安装了QEMU guest agent，Host就可以使用libivrt向VM发送命令，例如“冻结”，“释放”文件系统，虚拟CPU的热添加及移除等。
> - [x] RHEL/CetnOS7中有相应的安装包。qemu-guest-agent-xxx.rpm
> - [x] Windows需要手工安装

```
#这个管理包已经安装
[root@localhost ~]# rpm -qa | grep qemu-guest-agent
qemu-guest-agent-2.5.0-3.el7.x86_64
[root@localhost ~]# which virsh
/usr/bin/virsh
```

**通过libvirt来使用QEMU guest agent**

![QQ截图20180410220134.png-230.8kB](http://static.zybuluo.com/chensiqi/t94ovtg6yjg4wumwqgdor6fx/QQ%E6%88%AA%E5%9B%BE20180410220134.png)

## 四，管理虚拟机

### 4.1 libvirt架构概述

![QQ截图20180410222125.png-103kB](http://static.zybuluo.com/chensiqi/4mcvvi377ww2dla9jfqvo190/QQ%E6%88%AA%E5%9B%BE20180410222125.png)

> libvirtd是一个守护进程，virsh，virt-install等等都是依靠这个守护进程来间接访问qemu-kvm及配置文件。如果我们关闭这个进程，那么virsh，virsh-install，virt-manager就都不能访问了。（同学们可以试一下）

### 4.2 使用virt-manager管理虚拟机

- [x] virt-manager主要功能：
  - 定义和创建虚拟机
  - 硬件管理
  - 性能监视
  - 虚拟机的保存和恢复，暂停和继续，关闭和启动
  - 控制台
  - 在线和离线迁移
- [x] virt-manager
  - 方法1：Applications菜单>System Tools>Virtual Machine Manager (virt-manager)
  - 方法2 :在SSH会话中输入virt-manager

### 4.3 使用virsh来管理虚拟机

#### 4.3.1 virsh概述

- [x] virsh是使用libvirt management API构建的管理工具

- [x] virsh的名称的含义是virtualization shell。它有两种工作模式

  - 立即模式

  ```
      [root@localhost ~]# virsh list --all
   Id    名称                         状态
  ----------------------------------------------------
   3     centos6.5-2                    running
   14    vm2                            running
   -     centos6.5                      关闭
  ```

  - 交互模式

  ```
      [root@localhost ~]# virsh
  欢迎使用 virsh，虚拟化的交互式终端。
  
  输入：'help' 来获得命令的帮助信息
         'quit' 退出
  
  virsh # list --all
   Id    名称                         状态
  ----------------------------------------------------
   3     centos6.5-2                    running
   14    vm2                            running
   -     centos6.5                      关闭
  
  virsh # 
  ```

#### 4.3.2 关于virsh的命令帮助

> virsh所支持的命令有很多，建议同学们从virsh的帮助里查看

```
[root@localhost ~]# virsh --help

virsh [options]... [<command_string>]
virsh [options]... <command> [args...]

  options:
    -c | --connect=URI      hypervisor connection URI
    -d | --debug=NUM        debug level [0-4]
    -e | --escape <char>    set escape sequence for console
    -h | --help             this help
    -k | --keepalive-interval=NUM
                            keepalive interval in seconds, 0 for disable
    -K | --keepalive-count=NUM
                            number of possible missed keepalive messages
    -l | --log=FILE         output logging to file
    -q | --quiet            quiet mode
    -r | --readonly         connect readonly
    -t | --timing           print timing information
    -v                      short version
    -V                      long version
         --version[=TYPE]   version, TYPE is short or long (default short)
  commands (non interactive mode):

 **以下省略若干字**
```

#### 4.3.3 virsh常用命令

| 命令             | 概述                                                       |
| ---------------- | ---------------------------------------------------------- |
| attach-device    | 使用XML文件中的设备定义在虚拟机中添加设备                  |
| attach-disk      | 在虚拟机中附加新磁盘设备                                   |
| attach-interface | 在虚拟机中附加新网络接口                                   |
| create           | 从XML配置文件生成虚拟机并启动新虚拟机                      |
| define           | 为虚拟机输出XML配置文件                                    |
| destroy          | 强制虚拟机停止                                             |
| detach-device    | 从虚拟机中分离设备，使用同样的XML描述作为命令attach-device |
| detach-disk      | 从虚拟机中分离磁盘设备                                     |
| detach-interface | 从虚拟机中分离网络接口                                     |
| domblkstat       | 显示正在运行的虚拟机的块设备统计                           |
| domid            | 显示虚拟机ID                                               |
| domifstat        | 显示正在运行的虚拟机的网络接口统计                         |
| dominfo          | 显示虚拟机信息                                             |
| domname          | 显示虚拟机名称                                             |
| domstate         | 显示虚拟机状态                                             |
| domuuid          | 显示虚拟机UUID                                             |
| dumpxml          | 输出虚拟机XML配置文件                                      |
| help             | 打印基本帮助信息                                           |
| list             | 列出所有虚拟机                                             |
| migrate          | 将虚拟机迁移到另一台主机中                                 |
| nodeinfo         | 有关管理程序的输出信息                                     |
| quit             | 退出这个互动终端                                           |
| reboot           | 重新启动虚拟机                                             |
| restore          | 恢复以前保存在文件中的虚拟机                               |
| resume           | 恢复暂停的虚拟机                                           |
| save             | 将虚拟机当前状态保存到某个文件中                           |
| setmaxmem        | 为管理程序设定内存上限                                     |
| setmem           | 为虚拟机设定分配的内存                                     |
| setvcpus         | 修改为虚拟机分配的虚拟CPU数目                              |
| shutdown         | 关闭某个虚拟机                                             |
| start            | 启动未激活的虚拟机                                         |
| suspend          | 暂停虚拟机                                                 |
| undefine         | 删除与虚拟机关联的所有文件                                 |
| vcpuinfo         | 显示虚拟机的虚拟CPU信息                                    |
| vcpupin          | 控制虚拟机的虚拟CPU亲和性                                  |
| version          | 显示virsh版本                                              |

#### 4.3.4 实操演示virsh管理虚拟机

**（1）通过命令开启和关闭虚拟机**

```
#交互模式管理虚拟机
#启动虚拟机
[root@localhost ~]# virsh   #进入交互模式
欢迎使用 virsh，虚拟化的交互式终端。

输入：'help' 来获得命令的帮助信息
       'quit' 退出

virsh # list    #显示所有启动状态的虚拟机
 Id    名称                         状态
----------------------------------------------------
 3     centos6.5-2                    running
 14    vm2                            running

virsh # list --all  #显示所有虚拟机
 Id    名称                         状态
----------------------------------------------------
 3     centos6.5-2                    running
 14    vm2                            running
 -     centos6.5                      关闭

virsh # start centos6.5 #启动名称为centos6.5的虚拟机
域 centos6.5 已开始

virsh # list --all
 Id    名称                         状态
----------------------------------------------------
 3     centos6.5-2                    running
 14    vm2                            running
 15    centos6.5                      running   #已经启动了

#关闭虚拟机
virsh # shutdown 14 #shutdown优雅的关闭计算机，但有时我们这样关闭不了
域 14 被关闭

virsh # list --all
 Id    名称                         状态
----------------------------------------------------
 3     centos6.5-2                    running
 14    vm2                            running   #仍旧在运行
 -     centos6.5                      关闭

virsh # destroy 14      #destroy强制关闭虚拟机
域 14 被删除

virsh # list --all
 Id    名称                         状态
----------------------------------------------------
 3     centos6.5-2                    running
 -     centos6.5                      关闭
 -     vm2                            关闭      #被强制关闭了。
```

**（2）通过命令来设定虚拟机的主机开启自动引导启动**

> 关于主机开机引导时是否自动启动虚拟机，我们可以通过虚拟机的图形界面或者命令来设置，图形界面设置方式如下图所示：

![QQ截图20180418093116.png-54.3kB](http://static.zybuluo.com/chensiqi/60qlvap7zgn26uqazxncpihy/QQ%E6%88%AA%E5%9B%BE20180418093116.png)

```
[root@localhost ~]# virsh list --all
 Id    名称                         状态
----------------------------------------------------
 1     vm2                            running   #设置主机开机自动引导后，重启我们发现虚拟机自动开启
 -     centos6.5                      关闭
 -     centos6.5-2                    关闭
```

> 我们通过命令来控制虚拟机的开机自动引导

```
[root@localhost ~]# virsh   
欢迎使用 virsh，虚拟化的交互式终端。

输入：'help' 来获得命令的帮助信息
       'quit' 退出

virsh # help autostart  #查看autostart的帮助
  NAME
    autostart - 自动开始一个域

  SYNOPSIS
    autostart <domain> [--disable]

  DESCRIPTION
    设置一个域在启动时自动开始.

  OPTIONS
    [--domain] <string>  域名，id 或 uuid   #可以通过域名，id或uuid来控制
    --disable        禁止自动启动


virsh # list --all
 Id    名称                         状态
----------------------------------------------------
 1     vm2                            running
 -     centos6.5                      关闭
 -     centos6.5-2                    关闭

virsh # autostart centos6.5 #开启虚拟机的开机自引导
域 centos6.5标记为自动开始

virsh # autostart centos6.5 --disable   #关闭虚拟机开机自引导
域 centos6.5取消标记为自动开始

virsh # autostart centos6.5
域 centos6.5标记为自动开始
```

> 重启主机后，我们发现虚拟机已经可以自动启动

```
[root@localhost ~]# virsh list --all
 Id    名称                         状态
----------------------------------------------------
 1     vm2                            running
 2     centos6.5                      running
 -     centos6.5-2                    关闭
```

**（3）通过命令进行虚拟机的暂停和唤醒**

```
[root@localhost ~]# virsh
欢迎使用 virsh，虚拟化的交互式终端。

输入：'help' 来获得命令的帮助信息
       'quit' 退出

virsh # help suspend    #查看命令帮助
  NAME
    suspend - 挂起一个域

  SYNOPSIS
    suspend <domain>

  DESCRIPTION
    挂起一个运行的域。

  OPTIONS
    [--domain] <string>  域名，id 或 uuid


virsh # list --all
 Id    名称                         状态
----------------------------------------------------
 1     vm2                            running
 2     centos6.5                      running
 -     centos6.5-2                    关闭

virsh # suspend vm2 #暂停虚拟机
域 vm2 被挂起

virsh # list --all
 Id    名称                         状态
----------------------------------------------------
 1     vm2                            暂停      #成功
 2     centos6.5                      running
 -     centos6.5-2                    关闭

virsh # resume vm2  #唤醒虚拟机
域 vm2 被重新恢复

virsh # list --all
 Id    名称                         状态
----------------------------------------------------
 1     vm2                            running
 2     centos6.5                      running
 -     centos6.5-2                    关闭
```

