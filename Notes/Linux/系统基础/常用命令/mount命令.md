[TOC]

# mount

synopsis - mount [-lhV]

mount -a [-fFnrsvw] [-t vfstype] [-O optlist]

mount [-fnrsvw] [-o option[,option]...]  device|dir

mount [-fnrsvw] [-t vfstype] [-o options] device dir



## device

指明要挂载的设备

1,设备文件,如/dev/sdb1

2,卷标,-L 'LABEL':挂载时以卷标的方式指明设备,如 mount -L LABEL 'MYDATA'

3,UUID,-U 'UUID':挂载时以UUID的方式挂载,如 mount -U UUID '180b5975-7190-472e-b2df-42ae32cf4a81'

4,伪文件系统名称,如proc,sysfs,devtmpfs,configfs

dir:挂载点

注意: 挂载点需要事先存在,建议使用空目录,对于需要长时间挂载的分区,尽量不要挂载到media(便携式移动设备)和mnt(临时挂载)目录

## options

-r: readonly,只读挂载

-w: read and write,读写挂载

-t vsftype: 指定要挂载的设备上的文件系统类型,可省略

-n: 不更新记录至/etc/mtab文件(默认的挂载都会在挂载时记录至/etc/mtab文件中)

-a: 自动挂载/etc/fstab文件中的所有设备

-L 'LABEL': 挂载时以卷标的方式指明设备,如 mount -L LABEL 'MYDATA'

-U 'UUID': 挂载时以UUID的方式挂载,如 mount -U UUID '180b5975-7190-472e-b2df-42ae32cf4a81'

-B,--bind: 将目录绑定至另一个目录

-o OPTIONS: 挂载指明参数，即挂载的选项

## 挂载文件系统的选项

async/sync: 异步或同步操作,异步的方式是先将数据写入至内存,再保存至硬盘,同步是直接写入硬盘, 异步的性能会提升很多

atime/noatime: 立即更新访问时间戳/关闭

diratime/nodiratime: 开启/关闭目录的访问时间戳

remount: 重新挂载，在需要指明其他选项，即不影响其他用户访问的情况下使用

acl: 启用此文件系统上的acl功能

mount -o acl DEVICE DIR

tune2fs -o acl DEVICE :调整文件系统的默认属性为支持挂载时开启acl功能

ro : 只读

rw : 读写

dev/nodev: 是否支持在此文件系统上使用设备文件

exec/noexec: 是否支持将文件系统上的应用程序运行为进程

auto/noauto: 是否支持自动挂载

user/nouser: 是否允许普通用户挂载此文件系统

suid/nosuid: 是否允许SUID或SGID的程序生效

relatime: 是否参考atime mtime的时候来修改inode的访问时间

loop: 镜像文件

netdev: 在启动如果连接不上网络映射的盘,就停止挂载其网络系统,适用于ext2文件系统

注意:

上述选项可多个同时使用,彼此使用逗号分隔

默认加载选项:defaults

rw, suid, dev,exec, auto, nouser, async, relatime

# umount

取消挂载

## 格式

umount DEVICE

umount MOUNT_POINT

注意:如果挂载设备或挂载点有程序正在访问,则无法卸载

查看正在访问指定文件系统的进程

fuser -v MOUNT_POINT

踢出访问用户

fuser -km MOUNT_POINT

-k: kill

-m: 指定要结束访问进程的目录或设备



例:

```shell
# 挂载光驱到挂载点
mount /dev/cdrom /media/cdrom

# 切换进挂载目录,模拟使用
cd /media/cdrom

# 卸载挂载
umount /media/cdrom
"""
umount: /media/cdrom: device is busy.
        (In some cases useful info about processes that use
         the device is found by lsof(8) or fuser(1))
"""

# 查看挂载点使用的用户或程序
fuser -v /media/cdrom/
"""
                     USER        PID ACCESS COMMAND
/media/cdrom/:       root       1329 ..c.. bash
"""

# 踢出用户或程序,慎用
fuser -km /media/cdrom/
umount /media/cdrom
```

 

# mount挂载技巧

```shell
# 查看当前系统上所有挂载的设备
mount
cat /etc/mtab
cat /proc/mounts

# 可以实现将目录挂载到目录上，作为其临时访问的入口 
mount --bind 源目录 目录目录

# 挂载光盘
mount -r /dev/cdrom /media/cdrom

# 挂载本地回环设备
mount -o loop /FILE /DIR
umount - umount file system
synopsis: umount DEVICE|DIR

# 强制umount
# 使用lsof命令查看目录或设备被谁使用
lsof DIR|DEVICE

# 使用fuser -v 参数查看目录或设备被使用情况
fuser -v DIR|DEVICE

# 结束访问目录或设备的进程
fuser -km DIR|DIVECE
```





# end