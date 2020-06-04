[TOC]

# 系统安装

下载地址：https://www.ubuntu.com/download/alternative-downloads

1，选择系统语言：English

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/030fd0a8-689e-4599-8bb4-911d06274b26.png)

2，选择想要的操作

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/713ca1bf-4b22-43b0-b9c1-18002fbf62f0.png)

3，选择安装过程和系统的默认语言

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/03bf7a5e-80ee-4736-ba6c-0d663fe659ae.jpg)

4，选择区域

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/db037a88-ff8e-4d84-92a4-25d603780d26.jpg)

5，选择亚洲Asia

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/55bd54d4-947a-46ea-b750-14007326478f.jpg)

6，选择国家 China

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/b191c0f8-4f6e-420b-ab4d-d82b3c4d35d9.jpg)

7，选择字符集编码 United States

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/f93e843c-52c9-4ead-8ff0-fd3e36509744.jpg)

8，是否扫描和配置键盘 NO

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/2a03013a-8363-4f70-896f-d913c4f44345.jpg)

9，选择键盘类型 English(US)

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/2364ca1e-cd78-4a6c-aa56-abce18b585c0.jpg)

10，选择键盘布局 English(US)

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/402c098c-a496-46af-9308-e923571412cb.jpg)

11，设置主机名称

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/bc2cec6f-a7bf-4460-8948-00d4a41877b2.jpg)

12，设置一个普通用户

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/da072db7-a48c-4e1f-9374-0d901dca4ee4.jpg)

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/79833fd1-0651-44c4-ba3c-e8ab9e79af72.jpg)

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/3e54fb4e-5ffe-4c3a-a70d-53d828ee47ab.jpg)

13，是否加密home文件 NO

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/4d218f9e-2f45-4b99-85bb-3fc47c18b05e.jpg)

14，选择时区

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/af6ca910-4e1e-4ff3-a09c-4c77f5a510af.jpg)

15，选择硬盘分区方式

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/82070992-e0e6-4eab-8290-b72168195776.jpg)

16，选择硬盘

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/c4b5836f-6dc3-4e83-b53c-8e6056facdc6.jpg)

17，是否将分区变更写入硬盘

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/aa58f2da-5125-479f-8761-77911acbcade.jpg)

18，设置代理

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/9f4e83ee-8710-4a98-97f0-5f67d6f0af53.jpg)

19，设置系统升级方式

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/e965168a-d51d-48df-a100-6af56c73b90d.jpg)

20，选择要安装的软件 选择一个openssh server

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/72cfec30-1d8b-4659-ae97-56818e4a620f.jpg)

21，是否安装GRUB引导程序

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/561da58f-b081-4f3d-9d4a-1c48d56f5685.jpg)

21，完成安装

![img](ubuntu%20%E7%B3%BB%E7%BB%9F%E4%BC%98%E5%8C%96.assets/8328ddae-fd90-45d6-ad1b-da0e7993c183.jpg)



# root登录

默认root密码是随机的，即每次开机都有一个新的root密码。我们可以在终端输命令 sudo passwd，然后输入当前用户的密码，然后按enter键

终端会提示我们输入新的密码并确认，此时的密码就是root新密码。修改成功后，输入命令 su root，再输入新的密码就ok了

# 软件安装方法

## APT方式

普通安装：apt-get install softname1 softname2 …

修复安装：apt-get -f install softname1 softname2... (-f Atemp to correct broken dependencies)

重新安装：apt-get --reinstall install softname1 softname2...;

## Dpkg方式

普通安装：dpkg -i package_name.deb

源码安装（.tar、tar.gz、tar.bz2、tar.Z）

首先解压缩源码压缩包然后通过tar命令来完成

a．解xx.tar.gz：tar zxf xx.tar.gz 

b．解xx.tar.Z：tar zxf xx.tar.Z 

c．解xx.tgz：tar zxf xx.tgz 

d．解xx.bz2：bunzip2 xx.bz2 

e．解xx.tar：tar xf xx.tar

然后进入到解压出的目录中，建议先读一下README之类的说明文件，因为此时不同源代码包或者预编译包可能存在差异，然后建议使用ls -F --color或者ls -F命令（实际上我的只需要 l 命令即可）查看一下可执行文件，可执行文件会以*号的尾部标志。

一般依次执行./configure && make && sudo make install

即可完成安装。

# 软件包的卸载方法

## APT方式

移除式卸载：apt-get remove softname1 softname2 …;（移除软件包，当包尾部有+时，意为安装）

清除式卸载 ：apt-get --purge remove softname1 softname2...;(同时清除配置)

清除式卸载：apt-get purge sofname1 softname2...;(同上，也清除配置文件)

## Dpkg方式

移除式卸载：dpkg -r pkg1 pkg2 ...;

清除式卸载：dpkg -P pkg1 pkg2...;

# 其他命令

apt-cache search # ------(package 搜索包)

apt-cache show #------(package 获取包的相关信息，如说明、大小、版本等)

apt-get install # ------(package 安装包)

apt-get install # -----(package --reinstall 重新安装包)

apt-get -f install # -----(强制安装, "-f = --fix-missing"当是修复安装吧...)

apt-get remove #-----(package 删除包)

apt-get remove --purge # ------(package 删除包，包括删除配置文件等)

apt-get autoremove --purge # ----(package 删除包及其依赖的软件包+配置文件等（只对6.10有效，强烈推荐）)

apt-get update #------更新源

apt-get upgrade #------更新已安装的包

apt-get dist-upgrade # ---------升级系统

apt-get dselect-upgrade #------使用 dselect 升级

apt-cache depends #-------(package 了解使用依赖)

apt-cache rdepends # ------(package 了解某个具体的依赖,当是查看该包被哪些包依赖吧...)

apt-get build-dep # ------(package 安装相关的编译环境)

apt-get source #------(package 下载该包的源代码)

apt-get clean && apt-get autoclean # --------清理下载文件的存档 && 只清理过时的包

apt-get check #-------检查是否有损坏的依赖

dpkg -S filename -----查找filename属于哪个软件包

apt-file search filename -----查找filename属于哪个软件包

apt-file list packagename -----列出软件包的内容

apt-file update --更新apt-file的数据库

dpkg --info "软件包名" --列出软件包解包后的包名称.

dpkg -l --列出当前系统中所有的包.可以和参数less一起使用在分屏查看. (类似于rpm -qa)

dpkg -l |grep -i "软件包名" --查看系统中与"软件包名"相关联的包.

dpkg -s 查询已安装的包的详细信息.

dpkg -L 查询系统中已安装的软件包所安装的位置. (类似于rpm -ql)

dpkg -S 查询系统中某个文件属于哪个软件包. (类似于rpm -qf)

dpkg -I 查询deb包的详细信息,在一个软件包下载到本地之后看看用不用安装(看一下呗).

dpkg -i 手动安装软件包(这个命令并不能解决软件包之前的依赖性问题),如果在安装某一个软件包的时候遇到了软件依赖的问题,可以用apt-get -f install在解决信赖性这个问题.

dpkg -r 卸载软件包.不是完全的卸载,它的配置文件还存在.

dpkg -P 全部卸载(但是还是不能解决软件包的依赖性的问题)

dpkg -reconfigure 重新配置

ubuntu 1804----> root密码：123456 

主要操作：

1.更改网卡名称为eth0：

```
root@ubuntu:vim /etc/default/grub
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
root@ubuntu:update-grub
root@ubuntu:reboot
```

 

2.更改系统ip地址:

```
root@ubuntu:/home/jack# vim /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: no
      addresses: [192.168.100.112/24]
      gateway4: 192.168.100.2
      nameservers:
              addresses: [192.168.100.2]
```

 

3.应用ip配置并重启测试：   

```
root@ubuntu:netplan apply 
```

 

4.更改主机名：

```
# cat /etc/hostname 
k8s-node1.example.com
```

 

5.安装常用命令

```
apt-get update
apt-get purge ufw lxd lxd-client lxcfs lxc-common #卸载不用的包
apt-get install iproute2 ntpdate tcpdump telnet traceroute nfs-kernel-server nfs-common lrzsz tree openssl libssl-dev libpcre3 libpcre3-dev zlib1g-dev ntpdate tcpdump telnet traceroute gcc openssh-server lrzsz tree openssl libssl-dev libpcre3 libpcre3-dev zlib1g-dev ntpdate tcpdump telnet traceroute iotop unzip zip
```



6.做快照