[TOC]

PXEPXE(Pre-boot Execution Environment，预启动执行环境)，工作于Client/Server的网络模式，支持工作站通过网络从远端服务器下载映像，并由此支持通过网络启动操作系统，在启动过程中，终端要求服务器分配IP地址，再用TFTP（trivial file transfer protocol）或MTFTP(multicast trivial file transfer protocol)协议下载一个启动软件包到本机内存中执行，由这个启动软件包完成终端基本软件设置，从而引导预先安装在服务器中的终端操作系统。
在实际生产环境中，有时候我们会碰到为几十上百甚至上千台服务器安装Linux操作系统的需求，如果我们还是常规的去使用移动介质逐台安装，显然是一件低效又令人抓狂的事情，那要安装到何年何月啊？这对于我们追求高逼格形象的技术人员来讲当然是不可以接受的，为此，pxe模式批量部署系统应运而生。

原理我们知道，当我们使用其它引导介质（例如硬盘、软盘、U盘、CD或者DVD）安装操作系统时，是加载其首个扇区中MBR（主引导目录）中的引导程序并利用其查找各自介质中的必需数据来完成的。而pxe则是通过自带pxe bootrom的网卡使用TFTP（简单文件传输协议）和DHCP（动态主机配置协议）从网络服务器上查找并装载引导程序和必需的数据来完成系统的安装的。下面让我们通过实验来进一步理解其安装过程。
\1. PXE Client 从自己的PXE网卡启动，向本网络中的DHCP服务器索取IP；

\2. DHCP 服务器返回分配给客户机的IP 以及PXE文件的放置位置(该文件一般是放在一台TFTP服务器上) ；
\3. PXE Client 向本网络中的TFTP服务器索取pxelinux.0 文件；
\4. PXE Client 取得pxelinux.0 文件后之执行该文件；
\5. 根据pxelinux.0 的执行结果，通过TFTP服务器加载内核和文件系统 ；
\6. 进入安装画面, 此时可以通过选择HTTP、FTP、NFS 方式之一进行安装；

kickstart
Kickstart是一种无人值守的安装方式。它的工作原理是在安装过程中记录典型的需要人工干预填写的各种参数，并生成一个名为ks.cfg的文件。如果在安装过程中（不只局限于生成Kickstart安装文件的机器）出现要填写参数的情况，安装程序首先会去查找Kickstart生成的文件，如果找到合适的参数，就采用所找到的参数；如果没有找到合适的参数，便需要安装者手工干预了。所以，如果Kickstart文件涵盖了安装过程中可能出现的所有需要填写的参数，那么安装者完全可以只告诉安装程序从何处取ks.cfg文件，然后就去忙自己的事情。等安装完毕，安装程序会根据ks.cfg中的设置重启系统，并结束安装。

示例

实验环境：DHCP-server: 172.16.100.1tftp-server: 

1,安装配置DHCP

 

```
yum install dhcp -y
rpm -ql dhcp
cp /usr/share/doc/dhcp-4.1.1/dhcpd.conf.sample /etc/dhcp/dhcpd.conf
vim /etc/dhcp/dhcpd.conf
"""
option domain-name "pxe001.com";  #指定DHCP服务器的域名
option domain-name-server 172.16.100.1,8.8.8.8;  # 指定dns
default-lease-time 3600; #默认租约
max-lease-time 7200; #最大租约期限
subnet 172.16.100.0 netmask 255.255.0.0 {  # 开启地址池
    range 172.16.100.150 172.16.100.200;  # 地址池ip范围
    option routers 172.16.100.1;  # 路由网关,一般为本机
    filename "pxelinux.0";  # 引导程序文件
    next-server 172.16.100.1;  # pxe环境中的tftp地址
}
"""
chkconfig dhcpd on
chkconfig --list dhcpd
service dhcpd force-reload
service dhcpd start
ss -unl  # 67端口
cat /var/lib/dhcpd/dhcpd.leases  # 查看地址池分配记录
```

2，安装tftp-server

 

```
yum install tftp tftp-server -y
rpm -ql tftp-server
vim /etc/xinetd.d/tftp
"""
service tftp
{
        socket_type             = dgram
        protocol                = udp
        wait                    = yes
        user                    = root
        server                  = /usr/sbin/in.tftpd
        server_args             = -s /var/lib/tftpboot
        disable                 = no  # 改yes为no
        per_source              = 11
        cps                     = 100 2
        flags                   = IPv4
}
"""
/etc/init.d/xinetd start
chkconfig xinetd on
ss -tunlp # 69端口
```

3,安装syslinux  

找到引导linux安装的文件: pxelinux.0 ,由syslinux提供

 

```
yum info syslinux
yum install syslinux -y
rpm -ql syslinux
cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/
```

4,提供引导及内核等文件

 

```
mount -r /dev/cdrom /media/cdorm
cd /media/cdrom/images/pxeboot/
cp vmlinuz initrd.img /var/lib/tftpboot/
cd /media/cdrom/isolinux/
cp boot.cat vesamenu.c32 splash.jpg /var/lib/tftpboot/
mkdir /var/lib/tftpboot/pxelinux.cfg 
cp isolinux.cfg /var/lib/tftpboot/pxelinux.cfg/default
vim /var/lib/tftpboot/pxelinux.cfg/default
"""
default vesamenu.c32   # 默认启动的是label为linux的内核
#prompt 1  # 显示“boot：”提示符，值为0时则不显示，直接启动default参数指定的内容
timeout 600  # 选择页面超时时间
display boot.msg  # 显示某个文件的内容，默认在/tftpboot目录下，也可以写绝对路径
menu background splash.jpg
menu title Welcome to CentOS 6.8!
menu color border 0 #ffffffff #00000000
menu color sel 7 #ffffffff #ff000000
menu color title 0 #ffffffff #00000000
menu color tabmsg 0 #ffffffff #00000000
menu color unsel 0 #ffffffff #00000000
menu color hotsel 0 #ff000000 #ffffffff
menu color hotkey 7 #ffffffff #ff000000
menu color scrollbar 0 #ffffffff #00000000
label linux   # 找到标签为linux的字段
  menu label ^Install or upgrade an existing system
  menu default
  kernel vmlinuz
  append initrd=initrd.img text ks=http://172.16.100.7/ks.cfg  # 添加ks文件路径
label vesa
  menu label Install system with ^basic video driver
  kernel vmlinuz
  append initrd=initrd.img nomodeset
label rescue
  menu label ^Rescue installed system
  kernel vmlinuz
  append initrd=initrd.img rescue
label local
  menu label Boot from ^local drive
  localboot 0xffff
label memtest86
  menu label ^Memory test
  kernel memtest
  append -
"""
chmod 755 /var/lib/tftpboot/pxelinux.cfg/default
```

4,安装httpd，提供安装源

 

```
yum install httpd -y
mkdir /var/www/html/c6
mount --bind /media/cdrom /var/www/html/c6  # 将光盘文件挂载至http网页根目录下
ls /var/www/html/c6
vim /etc/fstab
"""
/dev/cdrom /var/www/html/c6 iso9660 default,ro,loop 0 0
"""
service httpd start
# 浏览器测试: 172.16.100.1/c6
```

5,安装编辑 kickstart 文件

 

```
yum groupinstall Desktop
yum groupinstall "X Window System"
yum groupinstall chinese-support
yum install system-config-kickstart -y
system-config-kickstart &  # 这是图形配置ks.cfg文件.
cp /root/anaconda-ks.cfg /var/www/html/ks.cfg
vim /ks.cfg   # 这里可参考光盘文件内的/isolinux/isolinux.cfg
"""
 #platform=x86, AMD64
 #version=DEVEL
 # Firewall configuration
 firewall --disabled
 # Install OS instead of upgrade
 install
 # Use network installation
 url --url="http://172.16.100.7/c6"    # 指定安装介质位置
 # Root password（xiaofei.com）
 rootpw  --iscrypted  $6$.vc1q3MIhm7CdPfU$PDrzE3WQEURV7Dp2XxM3DtOhWZqmGcNMUpsVhnxDq6.yPLWaVcoRNU667cDdUWTxALh3zRyS Vsf8uN76lnHhd/    # 这里是root的密文密码
 # System authorization information
 auth  --useshadow  --passalgo=sha512
 # Use text mode install
 text
 firstboot --disable
 # System keyboard
 keyboard us
 # System language
 lang en_US.UTF-8
 # SELinux configuration
 selinux --disabled
 # Installation logging level
 logging --level=info
 # Reboot after installation
 reboot
 # System timezone
 timezone --utc Asia/Shanghai
 # System bootloader configuration
 bootloader --location=mbr
 # Clear the Master Boot Record
 zerombr
 # Partition clearing information
 clearpart --all --initlabel
 # Disk partitioning information
 part /boot --fstype="ext4" --size=1000
 part swap --fstype="swap" --size=4096
 part / --fstype="ext4" --grow --size=10000
 
 %packages
 @chinese-support
 %end
"""
chmod 755 /var/www/html/ks.cfg
```

手动指定ks文件:

在linux安装初始界面，按esc键进入boot提示符模式

boot：linux ip=172.10.10.100 netmask=255.255.0.0 ks=http://172.16.100.7/ks.cfg

至此,pxe完成.

附: 图形配置ks.cfg文件

1,点击 File –> Open File –> root目录 –> anaconda-ks.cfg (该ks文件由服务器端系统安装完后生成） –> 点击Open 载入ks文件:

![img](PXE%E6%89%B9%E9%87%8F%E5%AE%89%E8%A3%85.assets/7561600d-322b-4a78-961b-c5dd4fba4750.jpg)

2, 基本配置[Basic Configuration]

![img](PXE%E6%89%B9%E9%87%8F%E5%AE%89%E8%A3%85.assets/f9874a40-3bf2-44a8-a1ef-7ccfadaa1b5c.png)

3, 安装方式[Installation Method]

![img](PXE%E6%89%B9%E9%87%8F%E5%AE%89%E8%A3%85.assets/962b1c5d-8393-434f-9126-67a168bab2ef.jpg)

4, 默认选项[Boot Loader Options]

默认即可

5, 配置分区信息[Partition Information]

![img](PXE%E6%89%B9%E9%87%8F%E5%AE%89%E8%A3%85.assets/8cc05b1f-8e71-485a-a290-0cdf00c22e56.png)

6, 网络配置[Network Configuration]

![img](PXE%E6%89%B9%E9%87%8F%E5%AE%89%E8%A3%85.assets/8241f0c7-32df-4055-bf6f-7e640fa297bd.png)

7, 验证[Authentication]

默认即可

8, 防火墙配置[Firewall Configuration]

![img](PXE%E6%89%B9%E9%87%8F%E5%AE%89%E8%A3%85.assets/2cddf324-312e-4f84-96dd-a14af5917bde.jpg)

9, 显示配置[Display Configuration]

默认即可

10, 预安装包[Package Selection]

默认即可,参照本机的.ks文件

11, Pre-Installation Script和Post-Installation Script均默认设置

12, 点击 File –> Save –> 修改文件名为 centos-6.5-ks.cfg 保存至 /var/www/html/centos.ks (本人自定义目录）下

13, 编辑/var/www/html/centos-6.5-ks.cfg，指定repo源到我们的http服务器对应repo源路径

![img](PXE%E6%89%B9%E9%87%8F%E5%AE%89%E8%A3%85.assets/d1fb5037-1553-473d-800c-62257088e7cf.png)

14, 提供PXE工作环境必须、内核以及其它所需

此处省略

总结:

yum install syslinux tftp-server -y

cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/

cp /media/cdrom/images/pxelinux/{vmlinuz,initrd.img} /var/lib/tftpboot/

cp /media/cdrom/isolinux/{boot.cfg,vesamenu.c32,splash.png} /var/lib/tftpboot/

mkdir /var/lib/tftpboot/pxelinux.cfg/

cp /media/cdrom/isolinux/isolinux.cfg /var/lib/tftpboot/pxelinux.cfg/default

cp /usr/share/syslinux/{chain.c32,mboot.c32,menu.c32,memdisk} /var/lib/tftpboot/

dhcp配置

filename: "pxelinux.0";

next-server 172.16.100.7;


