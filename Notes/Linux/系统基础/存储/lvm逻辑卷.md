[TOC]

目录:

LVM简介

LVM术语

LVM安装使用

LVM的快照功能

LVM简介

LVM是 Logical Volume Manager(逻辑卷管理)的简写，它由Heinz Mauelshagen在Linux 2.4内核上实现。LVM将一个或多个硬盘的分区在逻辑上集合，

相当于一个大硬盘来使用，当硬盘的空间不够使用的时候，可以继续将其它的硬盘的分区加入其中，这样可以实现磁盘空间的动态管理，相对于普通的磁盘分区有

很大的灵活性。

与传统的磁盘与分区相比，LVM为计算机提供了更高层次的磁盘存储。它使系统管理员可以更方便的为应用与用户分配存储空间。在LVM管理下的存储卷可以按需

要随时改变大小与移除(可能需对文件系统工具进行升级)。LVM也允许按用户组对存储卷进行管理，允许管理员用更直观的名称(如"sales'、'development')

代替物理磁盘名(如'sda'、'sdb')来标识存储卷。

如图所示LVM模型：

[![clip_image002[7\]](file:///D:/My Knowledge/temp/12208284-e924-4522-a3f3-21706ca1da6a/128/index_files/0.09915795619599521.png)](http://images.cnblogs.com/cnblogs_com/gaojun/201208/201208221004444929.jpg)

由四个磁盘分区可以组成一个很大的空间，然后在这些空间上划分一些逻辑分区，当一个逻辑分区的空间不够用的时候，可以从剩余空间上划分一些空间给空间不

够用的分区使用。

基本术语

前面谈到，LVM是在磁盘分区和文件系统之间添加的一个逻辑层，来为文件系统屏蔽下层磁盘分区布局，提供一个抽象的盘卷，在盘卷上建立文件系统。首先我们

讨论以下几个LVM术语：

物理存储介质（The physical media）：这里指系统的存储设备：硬盘，如：/dev/hda1、/dev/sda等等，是存储系统最低层的存储单元。

物理卷（physical volume）：物理卷就是指硬盘分区或从逻辑上与磁盘分区具有同样功能的设备(如RAID)，是LVM的基本存储逻辑块，但和基本的物理存储

介质（如分区、磁盘等）比较，却包含有与LVM相关的管理参数。

卷组（Volume Group）：LVM卷组类似于非LVM系统中的物理硬盘，其由物理卷组成。可以在卷组上创建一个或多个“LVM分区”（逻辑卷），LVM卷组由一个或

多个物理卷组成。

逻辑卷（logical volume）：LVM的逻辑卷类似于非LVM系统中的硬盘分区，在逻辑卷之上可以建立文件系统(比如/home或者/usr等)。

PE（physical extent）：每一个物理卷被划分为称为PE(Physical Extents)的基本单元，具有唯一编号的PE是可以被LVM寻址的最小单元。PE的大小是

可配置的，默认为4MB。

LE（logical extent）：逻辑卷也被划分为被称为LE(Logical Extents) 的可被寻址的基本单位。在同一个卷组中，LE的大小和PE是相同的，并且一一对应。

简单来说就是：

PV: 是物理的磁盘分区

VG: 卷组,可以理解为一个仓库或者是几个大的硬盘

LV：也就是从VG中划分的逻辑分区

如下图所示PV、VG、LV三者关系：

[![clip_image004[7\]](file:///D:/My Knowledge/temp/12208284-e924-4522-a3f3-21706ca1da6a/128/index_files/0.7704408911522478.png)](http://images.cnblogs.com/cnblogs_com/gaojun/201208/201208221004465079.jpg)

pv管理工具

pvs

pvdisplay

pvcreate /dev/DEVICE

vg管理工具

vgs

vgdisplay

vgcreate [-s #[mgtMGT]] VolumeGroupName PhysicalDevicePath [PhysicalDevicePath ..]

vgextend VolumeGroupName PhysicalDevicePath [PhysicalDevicePath ..]

vgreduce VolumeGroupName PhysicalDevicePath [PhysicalDevicePath ..]

vgremove

注意:缩减vg时,先做pvremove

lv管理工具

lvs

lvdisplay

lvcreate -L #[mgtMGT] -n NAME VolumeGroupName

lvremove

扩展逻辑卷

lvextend -L [+]#[mgtMGT] /dev/VG_NAME/LV_NAME

LVM安装创建和管理

 

```
cat /etc/redhat-release.2018-04-18
"""
CentOS release 6.9 (Final)
"""

uname -r
"""
2.6.32-696.el6.x86_64
"""

rpm -qa | grep lvm*
yum install lvm*
```



要创建一个LVM系统,一般需要经过以下步骤:

1,先创建物理卷;

2,创建PV;

3,创建VG,将pv加入到VG;

4,在VG上创建LV,设置自动挂载;

添加一块硬盘,这里大小为10G,然后将该硬盘进行分区,分区类型为8e(LVM格式);

 

```
# 创建物理卷lvm格式:8e
fdisk -l
fdisk /dev/sdb
"""
n # 创建新分区
p # 创建主分区
1 # 分区号为1
默认 # 分区大小默认全部
t # 设置分区类型
1 # 选择目标分区
8e # 设置为lvm分区类型
w # 保存
"""

fdisk -l
"""
Device Boot      Start       End     Blocks    Id   System
/dev/sdb1          1        1305    10482381   8e  Linux LVM
"""

partx -a /dev/sdb

# 创建PV
pvcreate /dev/sdb1  # 将sdb1创建成pv物理卷
pvs
pvdisplay

# 创建VG，把pv加入到VG,-s指定vg大小,默认全部大小
vgcreate mydata /dev/sdb1  # 创建vg,名字为mydata,pv为/dev/sdb1

vgs

vgdisplay mydata

# 在VG中创建逻辑卷lv
lvcreate -L 9.99G -nlvdata vgdata

lvs

lvdisplay

ls /dev/mapper/
control  vgdata-lvdata

# 格式化lv,设置文件系统
mkfs.ext4 /dev/vgdata/lvdata
mke2fs -t ext4 -b 1024 -L mylvm /dev/vgdata/lvdata

# 挂载lvdate到/date/mydate目录下
mkdir /data/lvmdata
mount /dev/vgdata/lvdata /data/lvmdata

mount 

ls -l /data/lvmdata

# 设置自动挂载
vim /etc/fstab
"""
/dev/vgdata/lvdata     /date/mydata     ext4     default    0  0
"""

umount /data/mydata

. /etc/fstab

mount -a
```



场景1:如何扩容

1,卷组vgdate有剩余空间

 

```
lvextend -L +1G /dev/vgdata/lvdata # +1g是指增加1G,如果不在+号,表示设置为1G,修改时物理边界

lvs  # 扩展的是LV空间,而文件系统所在的空间还没有增加

resize2fs /dev/vgdata/lvdata   # 同步文件系统,修改逻辑边界
```



2,vg卷组没有空间,硬盘还有空间

 

```
fdisk  /dev/sdc
"""
n
p        #选择主分区，注意，主分区最多四个P-P-P-P，超过4个需要先创建逻辑分区，再创建扩展分区P-P-P-E）
3        
t      8e   
w          
"""

partprobe  

mkfs –t ext4 /dev/sdb3

partx /dev/sdb3

vgextend vgdata /dev/sdb3

pvs
```



3,vg卷组没空间了,所在硬盘也没有空间了,需要添加一块新的硬盘,这里新增硬盘大小依旧为5G.

 

```
fdisk /dev/sdc
"""
n # 创建新分区
p # 创建主分区
1 # 分区号为1
默认 # 分区大小默认全部
t # 设置分区类型
1 # 选择目标分区
8e # 设置为lvm分区类型
w # 保存
"""

fdisk -l  # 先对新硬盘进行分区,大小为2G,剩余空间留作以后扩容

pvcreate /dev/sdc1  # 初始化新硬盘的分区为物理卷pv

pvdisplay

vgextend vgdata /dev/sdc1   # 把新硬盘的分区加入到原硬盘的卷组vgdate中

pvdisplay   

umount /data/mydata

mount /dev/vgdata/lvdata /data/mydata

df -h  # 此时的逻辑空间还未扩容

lvresize -L +2G /dev/vgdata/lvdata   # 扩容

resize2fs /dev/vgdata/lvdata     # 重设lv分区大小

df -hl

```



场景2:如何缩减逻辑卷

释放逻辑卷空间给其他卷使用,通常步骤如下:

1,先卸载挂载的逻辑分区

umount /dev/VG_NAME/LV_NAME

2,使用e2fsck -f检查该逻辑卷上文件系统的强制检测与修复

e2fsck -f /dev/VG_NAME/LV_NAME

3,使用resize2fs将文件系统减少到合适的大小

resize2fs /dev/VG_NAME/LV_NAME #[mgtMGT]

4,再使用lvreduce命令将逻辑卷减少到合适的大小

lvreduce -L [-]#[mgtMGT] /dev/VG_NAME/LV_NAME

5,重新挂载

注意:文件系统大小和逻辑卷大小一定要保持一致才行.如果逻辑卷大于文件系统,由于部分区域未格式化成文件系统会造成空间的浪费.如果逻辑卷小于文件系

统,那数据就出问题了.

 

```
umount /date/mydate # 卸载挂载
e2fsck -f /dev/vgdate/lvdate # 检测
resize2fs /dev/vgdate/lvdate 1G  # 缩减逻辑边界
lvreduce -L 1G /dev/vgdate/lvdate # 缩减物理边界
mount /dev/vgdate/lvdate /date/mydate
dh -lh
```



场景3:如果硬盘或分区挂了,如何转移数据到同卷组内的其他空间去?

步骤:

1,通过pvmove命令转移空间数据

2,通过vgreduce命令将即将坏的磁盘或者分区从卷组vgdata里面移除除去。

3,通过pvremove命令将即将坏的磁盘或者分区从系统中删除掉。

4,手工拆除硬盘或者通过一些工具修复分区。

 

```
pvmove /dev/sdb1 /dev/sdc1 # 把sdb1的数据转移到sdc1上
pvs
vgreduce vgdate /dev/sdb1 # 移除卷组内挂掉的分区sdb1
pvremove /dev/sdb1 # 将挂掉的硬盘或分区从系统中删除掉
pvs
```



场景4:删除整个逻辑卷

操作步骤:

1,umount 卸载挂载

2,/etc/fstab 修改

3,先移除逻辑卷lv,在移除卷组vg,然后移除pv,还原成普通分区,最后记得修改分区的类型为linux

 

```
lvremove /dev/VolGroup00/LogVol01
vgremove /dev/VolGroup00
pvremove /dev/sdb1
```



LVM的快照功能snapshot

LVM中snapshot通过“写时复制”(copy on write) 来实现，即当一个snapshot创建的时候，仅拷贝原始卷里数据的元数据(meta-data)；创建的时候，并

不会有数据的物理拷贝，因此snapshot的创建几乎是实时的，当原始卷上有写操作执行时，snapshot跟踪原始卷块的改变，这个时候原始卷上将要改变的数据

在改变之前被拷贝到snapshot预留的空间里.

注意：采取CoW实现方式时，snapshot的大小并不需要和原始卷一样大，其大小仅仅只需要考虑两个方面：

从shapshot创建到释放这段时间内，估计块的改变

量有多大;数据更新的频率。一旦 snapshot的空间记录满了原始卷块变换的信息，那么这个snapshot立刻被释放，从而无法使用，从而导致这个snapshot无

效。所以，非常重要的一点，一定要在snapshot的生命周期里，做完你需要做得事情.

快照文件需要与lv在同一vg中,所以确保要做快照的lv所在的vg中有足够的空间,如果没有则需要做vg扩容

lvcreate -L #[mgtMGT] -p r -n snapshot_lv_name src_lv_name

参数说明:

-s:创建快照

-p:设置快照权限

-n:快照名字

-L:快照空间大小

 

```
# 创建快照
lvcreate -s -L 512M -n mylv-snap -p r /dev/vgdata/lvdata

# 挂载快照
mount /dev/vgdata/mylv-snap /mnt
ls /mnt

# 修改原lv内文件做测试
echo "hello,world" >> /dev/vgdata/lvdata/issue
cat /mnt/issue
cp -a /mnt/{fstab,issue} /tmp/
umount /mnt  # 若果提示busy,解决办法:fuser -km /datatmp
lvremove /dev/vgdata/mylv-snap
```



总结：

![img](lvm%E9%80%BB%E8%BE%91%E5%8D%B7.assets/2eb91f88-875a-4329-b580-335486022bdb.png)

普通分区创建为pv，使用pvcreate

创建vg，使用vgcreate

创建lv，使用lvcreate

格式化lv文件系统，mkfs.ext4

挂载lv,mount

lv扩容，lvextend,resize2fs

vg扩容，pvcreate,vgextend 

扫描，pvscan

移除pv，pvremove

移除vg，vgremove

移除lv，lvremove

 