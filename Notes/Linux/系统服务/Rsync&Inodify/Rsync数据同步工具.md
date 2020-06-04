[TOC]

## 1.1 Rsync介绍

### 1.1.1 什么是Rsync？

> Rsync是一款开源的，快速的，多功能的，可实现全量及增量的本地或远程数据同步备份的优秀工具。Rsync软件适用于unix/linux/windows等多种操作系统平台。

### 1.1.2 Rsync简介

- Rsync英文全称Remote  synchronization，从软件的名称就可以看出来，Rsync具有可使本地和远程两台主机之间的数据快速复制同步镜像，远程备份的功能，这个功能类似ssh带的scp命令，但又优于scp命令的功能，scp每次都是全量拷贝，而rsync可以增量拷贝。当然，Rsync还可以在本地主机的不同分区或目录之间全量及增量的复制数据，这又类似cp命令，但同样也优于cp命令，cp每次都是全量拷贝，而rsync可以增量拷贝。

> 小提示：利用Rsync还可以实现删除文件何目录的功能，这又相当于rm命令！

- 一个rsync相当于scp，cp，rm，但是还优于他们每一个命令。
- 在同步备份数据时，默认情况下，Rsync通过其独特的“quick  check”算法，它仅同步大小或者最后修改时间发生变化的文件或目录，当然也可根据权限，属主等属性的变化同步，但需要指定相应的参数，甚至可以实现只同步一个文件里有变化的内容部分，所以，可以实现快速的同步备份数据。

> 提示：传统的cp，scp工具拷贝每次均为完整的拷贝，而rsync除了可以完整拷贝外，还具备增量拷贝的功能，因此，从同步数据的性能及效率上，Rsync工具更胜一筹。

- CentOS5，rsync2.x比对方法，把所有的文件比对一遍，然后进行同步。
- CentOS6，rsync3.x比对方法，一边比对差异，一边对差异的部分进行同步。

### 1.3 Rsync的特性

**Rsync的特性如下：**

- 支持拷贝特殊文件如链接文件，设备等
- 可以有排除（tar？find？）指定文件或目录同步的功能，相当于打包命令tar的排除功能
- 可以做到保持原文件或目录的权限，时间，软硬链接，属主，组等属性均不改变-p
- 可以实现增量同步，既只同步发生变化的数据，因此数据传输效率很高（tar-N）
- 可以使用rcp，rsh，ssh等方式来配合传输文件（rsync本身不对数据加密）
- 可以通过socket（进程方式）传输文件和数据（服务端和客户端）
- 支持匿名的或认证（无需系统用户）的进程模式传输，可实现方便安全的进行数据备份及镜像

### 1.1.4 Rsync的企业工作场景说明

#### 1.1.4.1 两台服务器之间数据同步（定时任务+备份数据）即crond+rsync

> **生产场景集群架构服务器备份方案项目**
>
> 借助crond+rsync把所有客户服务器数据同步到备份服务器

**简历项目经验：**
 全网服务器数据备份解决方案提出及负责实施200x.03 - 200x.09

1）针对公司重要数据备份混乱状况和领导提出备份全网数据的解决方案
 2）通过本地打包备份，然后rsync结合inotify应用把全网数据统一备份到一个固定存储服务器，然后在存储服务器上通过脚本检查并报警管理员备份结果
 3）定期将IDC机房的数据备份到公司的内部服务器，防止机房地震及火灾问题导致数据丢失。

#### 1.1.4.2 实时同步（解决存储服务器等的单点问题）

利用rsync结合inotify的功能做实时的数据同步，根据存储服务器上目录的变化，把变化的数据通过inotify或sersync结合rsync命令实时同步到备份服务器，还可以通过drbd方案以及双写的方案实现双机数据同步。

## 1.2 Rsync的工作方式

为了方便同学学习，我从实际的使用功能上进行了一下划分。一般来说，Rsync大致使用三种主要的传输数据的方式。分别为：

- 单个主机本地之间的数据传输（此时类似于cp命令的功能）
- 借助rcp，ssh等通道来传输数据（此时类似于scp命令的功能）
- 以守护进程（socket）的方式传输数据（这个是rsync自身的重要功能）

以上的几种rsync的工作方式，我们可以通过man rsync帮助或查看官方的手册获得：

```
NAME
       rsync -- a fast, versatile, remote (and local) file-copying tool

SYNOPSIS
       Local:  rsync [OPTION...] SRC... [DEST]

       Access via remote shell:
         Pull: rsync [OPTION...] [USER@]HOST:SRC... [DEST]
         Push: rsync [OPTION...] SRC... [USER@]HOST:DEST

       Access via rsync daemon:
         Pull: rsync [OPTION...] [USER@]HOST::SRC... [DEST]
               rsync [OPTION...] rsync://[USER@]HOST[:PORT]/SRC... [DEST]
         Push: rsync [OPTION...] SRC... [USER@]HOST::DEST
               rsync [OPTION...] SRC... rsync://[USER@]HOST[:PORT]/DEST

       Usages with just one SRC arg and no DEST arg will list the source files
       instead of copying.
```

### 1.2.1 本地数据传输模式（local-only mode）

Rsync本地传输模式的语法为：
 `rsync [OPTION...] SRC...[DEST]`
 语法说明：
 1）Rsync为同步的命令；
 2）[OPTION]为同步时的参数选项
 3）SRC为源，即待拷的分区，文件或目录等；
 4）[DEST]为目的分区，文件或目录等；

直接本地同步：相当于cp
 rsync /etc/hosts /tmp/

**示例1-1** 实例1:把系统的hosts文件同步到／opt目录

```
[root@chen ~]# rsync /etc/hosts /opt
[root@chen ~]# cat /opt/hosts 
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.14.200 mirrors.aliyum.com
192.168.197.133 www.test.com
```

实例1-2 实例2:把opt目录拷贝到/mnt下

```
[root@chen ~]# rsync -avz /opt /mnt #相当于cp -ap /opt /mnt
sending incremental file list
opt/
opt/hosts
opt/rh/

sent 224 bytes  received 39 bytes  526.00 bytes/sec
total size is 221  speedup is 0.84
[root@chen ~]# ll /mnt
total 8
drwxr-xr-x. 3 root root 4096 Mar  5 19:54 opt
-rw-r--r--. 1 root root    5 Dec 25 11:19 test.txt
```

删除功能，相当于rm命令

```
[root@chen ~]# mkdir /old
[root@chen ~]# rsync -avz --delete /old/ /tmp/
sending incremental file list
./
deleting pear/temp/
deleting pear/
deleting old/
deleting .ICE-unix/
deleting user_passwd

sent 29 bytes  received 15 bytes  88.00 bytes/sec
total size is 0  speedup is 0.00
[root@chen ~]# ll /tmp/
total 0
```

### 1.2.2 rsync 命令常用参数选项说明：

-v，--verbose 详细模式输出，传输时的进度等信息
 -z，--compress 传输时进行压缩以提高传输效率，--compress-level=NUM可按级别压缩。
 -a，--archive 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rtopgD1（字母1）
 ==================================

| -r,--recursive    | 对子目录以递归模式，即目录下的所有目录都同样传输，注意是小写r |
| ----------------- | ------------------------------------------------------------ |
| -t，--times       | 保持文件时间信息                                             |
| -o，--owner       | 保持文件属主信息                                             |
| -p，--perms       | 保持文件权限                                                 |
| -g，--group       | 保持文件属组信息                                             |
| -P，--progress    | 显示同步的过程及传输时的进度等信息                           |
| -D，--devices     | 保持设备文件信息                                             |
| -l，--links       | 保留软链接                                                   |
| -e，--rsh=COMMAND | 使用的信道协议（remote shell），指定替代rsh的shell程序。例如：ssh --exclude=PATTERN 指定排除不需要传输的文件模式（和tar参数一样） |
| --bwlimit=RATE    | limit socket I/O bandwidth                                   |
| --delete          | 让源目录SRC和目标目录数据DST一致                             |

### 1.2.3 **案例**：某DBA做数据同步，带宽占满，导致用户无法访问网站。

```
rsync -avz dbfile 10.0.0.41:/backup #没有给带宽做限制
rsync -avz --bwlimit=100 dbfile 172.16.1.41:／backup   #限定了带宽
```

### 1.2.4 保持同步目录及文件属性

这里的-avzP 相当于-vzrtopgDIP（还多了DI功能），生产环境常用的参数选项为-avzP或-vzrtopgP如果是放入脚本中，也可以把-v何-P去掉。这里的--progress可以用-P代替。

> 特别说明：以上参数为企业生产环境常用参数，对于初学者来说掌握上面内容已足够。

生产参数：-avz或者用-vzrtopg

### 1.2.5 使用rsync在本地备份传输数据

实例1:测试本地rsync同步，rsync -avz /opt /tmp

```
[root@chen ~]# cd /opt #进入目录
[root@chen opt]# mkdir chensiqi  #创建目录
[root@chen opt]# touch chensiqi/test.txt #创建文件
[root@chen opt]# chmod -R 700 chensiqi #递归授权700
[root@chen opt]# ls -l #查看目录权限700
total 4
drwx------. 2 root root 4096 Mar  5 22:18 chensiqi
[root@chen opt]# ls -l chensiqi/  #查看文件权限700
total 0
-rwx------. 1 root root 0 Mar  5 22:18 test.txt
[root@chen opt]# rsync -avz /opt/ /tmp/  #通过rsync执行本地同步操作
sending incremental file list
./
chensiqi/
chensiqi/test.txt

sent 116 bytes  received 38 bytes  308.00 bytes/sec
total size is 0  speedup is 0.00
[root@chen opt]# tree /tmp  #目录文件完全同步过去了
/tmp
`-- chensiqi
    `-- test.txt

1 directory, 1 file
[root@chen opt]# ll /tmp/  #文件夹权限700，保持一致
total 4
drwx------. 2 root root 4096 Mar  5 22:18 chensiqi
[root@chen opt]# ll /tmp/chensiqi/ #文件权限700保持一致
total 0
-rwx------. 1 root root 0 Mar  5 22:18 test.txt
```

上例演示了将本地/opt目录下的文件（不包含opt本身）同步到/tmp下其中-avz就是保持目录或文件的相关属性的参数

> 特别提示：请注意以下两条命令的差别：
>  1)rsync -avz /opt/ /tmp/
>  2)rsync -avz /opt /tmp/
>  1)中/opt/的意思是，仅把/opt/目录里面的内容同步过来，opt目录本身并不同步；而后者2）中/opt表示把opt本身及其内部内容全都同步到/tmp下，仅一个/（斜线之差），意义大不相同，请同学们注意使用的差别。
>  2）在后边要讲的通过远程shell进行数据传输的内容也会有类似的问题，请牢记。

当本地的不同目录之间需要数据传输，特别是经常需要增量传输时，这个案例命令可以替代cp等命令，为你提升拷贝的效率。

实例2:将/etc下全部内容（包括/etc目录本身）备份到/tmp目录下

```
[root@chen ~]# rsync -avz /etc /tmp/
sending incremental file list
etc/
etc/.pwd.lock
etc/DIR_COLORS
etc/DIR_COLORS.256color
etc/DIR_COLORS.lightbgcolor
etc/adjtime
etc/aliases
etc/aliases.db
etc/anacrontab
下面的输出内容省略....

[root@chen ~]# ll /tmp  #同步完成
total 4 
drwxr-xr-x. 79 root root 4096 Mar  5 19:25 etc
```

第一次运行命令会由于需要扫描并同步所有文件及目录，因此时间会长一些。如果再次备份就会进行快速对比，忽略通过的文件，速度更快，如下文：

```
[root@chen ~]# rsync -avz /etc /tmp/
sending incremental file list

sent 39813 bytes  received 196 bytes  80018.00 bytes/sec
total size is 27542875  speedup is 688.42
```

我们可以看到立刻就同步完成，要传输的数据也很少了。因为rsync会比对所有文件和目录，仅同步有变化（内容，修改时间等各种属性）的文件或目录。如果换做cp命令，那么还会重新执行完整的拷贝，浪费系统资源和时间。
 当然本地备份同步不仅仅备份目录，还可以同步单个文件，设备等，相信聪明的你都想到了，在此就不多费笔墨。

> 特别提示：
>  在传输数据时，rsync命令也需要有对同步的目录拥有权限如此才可以实现正常传输数据。

## 1.3 借助ssh通道在不同主机之间传输数据

示例1:**推送：将当前主机内容推送到远程主机**

```
rsync -avzP -e 'ssh -p 22'/etc/  root@192.168.197.129:/tmp/
[root@chensiqi ~]# rsync -avzP -e 'ssh -p 22' /etc/  root@192.168.197.129:/tmp/ #开始同步
忽略以上内容....
yum/version-groups.conf
         444 100%    1.14kB/s    0:00:00 (xfer#985, to-check=6/1558)
yum/pluginconf.d/
yum/pluginconf.d/fastestmirror.conf
         279 100%    0.72kB/s    0:00:00 (xfer#986, to-check=2/1558)
yum/pluginconf.d/security.conf
          17 100%    0.04kB/s    0:00:00 (xfer#987, to-check=1/1558)
yum/protected.d/
yum/vars/
yum/vars/infra
           6 100%    0.02kB/s    0:00:00 (xfer#988, to-check=0/1558)

sent 9847758 bytes  received 20677 bytes  1518220.77 bytes/sec
total size is 27542879  speedup is 2.79

#命令说明
-e 'ssh -p 22' 表示以ssh的方式通过22端口推送，如果不写默认22端口


[root@chensiqi ~]# ssh root@chensiqi2 "ls -l /tmp" #查看同步结果
root@chensiqi2's password: 
total 1668
drwxr-xr-x.  5 root root   4096 Dec 24 09:26 ConsoleKit
-rw-r--r--.  1 root root   4439 Apr 12  2016 DIR_COLORS
-rw-r--r--.  1 root root   5139 Apr 12  2016 DIR_COLORS.256color
-rw-r--r--.  1 root root   4113 Apr 12  2016 DIR_COLORS.lightbgcolor
drwxr-xr-x.  3 root root   4096 May 12  2016 NetworkManager
drwxr-xr-x.  4 root root   4096 Dec 24 09:26 X11
以下省略若干内容...

#命令说明：
ssh root@chensiqi2的意思是，以ssh的方式进行连接，通过root账户来登录主机名为chensiqi2的这台主机。
ssh root@chensiqi2 + 命令，可以将命令的结果反馈回来。
chensiqi2是当前主机下的一个hosts影射，/etc/hosts里面添加：IP地址  主机名  即作为映射对应。当输入主机名时，系统自动通过hosts解析出对应IP地址。例如ssh root@chensiqi2  <==> ssh root@192.168.197.129
```

示例2:**将远程主机内容拉取到当前主机**

```
rsync -avzP -e 'ssh -p 22' root@chensiqi2:/opt /tmp
```

关键语法说明：
 1）-avz相当于-vzrtopgDI，表示同步时文件和目录属性不变。
 2）-P显示同步的过程，可以用--progress替换。
 3）-e ‘ssh -p 22’表示通过ssh通道传输数据，可省略
 4）root@chensiqi2:/opt 远程主机系统用户，地址，路径
 5）/tmp本地的路径

实践演示：通过root用户从192.168.197.129的/opt目录（包含目录本身）把数据拉到本地的/tmp目录下

```
[root@chensiqi ~]# rsync -avzP -e 'ssh -p 22' root@192.168.197.129:/opt /tmp/
root@192.168.197.129's password: 
receiving incremental file list
opt/
opt/chensiqi
           0 100%    0.00kB/s    0:00:00 (xfer#1, to-check=1/3)
opt/rh/

sent 38 bytes  received 122 bytes  29.09 bytes/sec
total size is 0  speedup is 0.00
[root@chensiqi ~]# ll /tmp
total 4
drwxr-xr-x. 3 root root 4096 Mar  6  2017 opt
```

也可以去掉 -e ‘ssh -p 22’(默认22端口)

```
[root@chensiqi ~]# rsync -avzP root@192.168.197.129:/opt /tmp/
root@192.168.197.129's password: 
receiving incremental file list

sent 13 bytes  received 80 bytes  37.20 bytes/sec
total size is 0  speedup is 0.00
```

也可以通过映射好的主机名：(/etc/hosts)

```
[root@chensiqi ~]# tail -1 /etc/hosts
192.168.197.129 chensiqi2
[root@chensiqi ~]# rsync -avzP root@chensiqi2:/opt /tmp/
root@chensiqi2's password: 
Permission denied, please try again.
root@chensiqi2's password: 
receiving incremental file list
opt/
opt/chensiqi
           0 100%    0.00kB/s    0:00:00 (xfer#1, to-check=1/3)
opt/rh/

sent 38 bytes  received 122 bytes  21.33 bytes/sec
total size is 0  speedup is 0.00
```

## 1.4 以守护进程（socket）的方式传输数据（重点）

### 1.4.1 部署前的准备工作：

### 1.4.2 部署环境

考虑到很多同学没有实际的生产环境，本文使用VMWARE虚拟机环境下Linux主机来进行实验。
 和生产环境的真实服务器部署几乎没有任何区别。

**操作系统：**

```
[root@chensiqi ~]# cat /etc/redhat-release 
CentOS release 6.8 (Final)
```

**内核版本：**

```
[root@chensiqi ~]# uname -r
2.6.32-642.el6.x86_64
```

**主机网络参数设置：**
 |主机名|网卡eth0|用途|代号|
 |--|--|--|--|--|
 |chensiqi|192.168.197.133|rsync客户端|B-Server|
 |chensiqi2|192.168.197.129|rsync服务端|A-Server|

> **提示**：如无特殊说明。子网掩码均为255.255.255.0

### 1.4.3 具体要求

要求在A-Server上以rsync守护进程的方式部署rsync服务，使得所有rsync节点客户端主机，可以把本地数据通过rsync的方式备份到数据备份服务器A-Server上。本例的客户端仅以B-Server，C-Server为例。

![屏幕快照 2017-03-06 下午1.24.38.png-171.6kB](http://static.zybuluo.com/chensiqi/4gweznps8fzctiy64ljor2d8/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-06%20%E4%B8%8B%E5%8D%881.24.38.png)

## 1.5 开始部署rsync服务--Rsync服务器端A-Server操作过程：

### 1.5.1 配置rsyncd.conf

首先确认软件是否安装：

```
[root@chensiqi2 ~]# rpm -qa rsync
rsync-3.0.6-12.el6.x86_64
```

然后创建rsyncd.conf文件,并添加如下内容（文件默认不存在）

```
[root@chensiqi2 backup]# cat /etc/rsyncd.conf
#rsync_config_____start
#created by chensiqi 13:40 2017-3-6
##blog:http://www.cnblogs.com/chensiqiqi/
##rsyncd.conf start##

# 用户
uid = rsync
# 组
gid = rsync
# 程序安全设置
use chroot = no
# 客户端连接数
max connections = 200
# 超时时间
timeout = 300
# 进程号文件位置
pid file = /var/run/rsyncd.pid
# 进程锁
lock file = /var/run/rsync.lock
# 日志文件位置
log file = /var/log/rsyncd.log
##########################################
[backup]
# 使用目录
path = /backup/
# 有错误时忽略
ignore errors
# 可读可写（true或false）
read only = false
# 阻止远程列表（不让通过远程方式看服务端有啥）
list=false
# 允许IP
hosts allow = 192.168.197.0/24
# 禁止IP
hosts deny = 0.0.0.0/32
# 虚拟用户
auth users = rsync_backup
# 存放用户和密码的文件
secrets file = /etc/rsync.password

##rsync_config______end##
```

### 1.5.2 创建共享目录及添加rsync程序用户

```
[root@chensiqi2 ~]# useradd -M -s /sbin/nologin rsync  #创建rsync用户
[root@chensiqi2 ~]# cat /etc/passwd | grep rsync  
rsync:x:500:500::/home/rsync:/sbin/nologin
[root@chensiqi2 ~]# cat /etc/group | grep rsync
rsync:x:500:
[root@chensiqi2 ~]# mkdir /backup #创建共享目录
```

### 1.5.3 启动服务：rsync --daemon

```
[root@chensiqi2 ~]# rsync --daemon
[root@chensiqi2 ~]# netstat -antup | grep rsync
tcp        0      0 0.0.0.0:873                 0.0.0.0:*                   LISTEN      5163/rsync          
tcp        0      0 :::873                      :::*                        LISTEN      5163/rsync  
```

### 1.5.4 将A-Server上的/backup文件夹更改属主rsync

```
[root@chensiqi2 ~]# chown -R rsync /backup
[root@chensiqi2 ~]# ls -ld /backup
drwxr-xr-x. 2 rsync root 4096 3月   6 22:19 /backup
```

### 1.5.5 创建rsync虚拟账户名和密码

```
[root@chensiqi2 ~]# echo "rsync_backup:123456" >/etc/rsync.password
[root@chensiqi2 ~]# cat /etc/rsync.password
rsync_backup:123456
```

### 1.5.6 将账户密码文件的权限设置为600（必须否则失败）

```
[root@chensiqi2 ~]# chmod 600 /etc/rsync.password 
[root@chensiqi2 ~]# ll /etc/rsync.password 
-rw-------. 1 root root 20 3月   6 22:27 /etc/rsync.password
```

### 1.5.7 加入开机启动

```
[root@chensiqi2 ~]# echo "rsync --daemon" >> /etc/rc.local
[root@chensiqi2 ~]# tail -1 /etc/rc.local 
rsync --daemon
```

**注意：**
 当然还可以用chkconfig rsync on命令，但是必须要编写适合chkconfig操作的脚本才行。

**如何重启rsync服务？**
 pkill rsync #关闭rsync服务
 rsync --daemon #启动rsync服务

**至此rsync服务器端A-server配置完毕**

## 1.6 开始部署rsync服务--Rsync客户端B-Server

### 1.6.1 只需要创建密码文件

```
[root@chensiqi ~]# rpm -qa rsync
rsync-3.0.6-12.el6.x86_64
[root@chensiqi ~]# echo "123456" > /etc/rsync.password
```

### 1.6.2 将密码文件的权限设置为600（必须否则失败）

```
[root@chensiqi ~]# chmod 600 /etc/rsync.password 
[root@chensiqi ~]# ls -ld /etc/rsync.password 
-rw-------. 1 root root 7 Mar  6 01:42 /etc/rsync.password
```

**至此rsync客户端B-Server配置完毕。**

### 1.6.5 Rsync同步测试

#### 1.6.5.1 推送测试1：将客户端指定目录内容推送到服务器端rsync指定目录下。

**测试命令：**

```
rsync -avz /backup/ rsync_backup@192.168.197.129::backup --password-file=/etc/rsync.password

命令说明：
-avz：保持稳健各项属性不变，-v显示同步信息 -P显示具体同步过程
/backup/：要推送的内容所在目录
rsync_backup:服务器端rsync服务的同步的用户名（非Linux用户）
192.168.197.129:rsync服务器IP地址
backup：rsync服务器配置文件里的模块名
--password-file=/etc/rsync.password：免密码的操作，指定密码文件位置，如果不写，则会要求用户交互式输入密码。（如果想挂定时任务，必须得非交互式）
```

**演示：**

```
[root@chensiqi backup]# ls
opt.tar.gz
[root@chensiqi backup]# rsync -avzP /backup/ rsync_backup@192.168.197.129::backup --password-file=/etc/rsync.password #同步测试 

sending incremental file list
./
opt.tar.gz
         166 100%    0.00kB/s    0:00:00 (xfer#1, to-check=0/2)

sent 258 bytes  received 30 bytes  576.00 bytes/sec
total size is 166  speedup is 0.58

[root@chensiqi backup]# ssh root@chensiqi2 "ls -l /backup" #查看同步结果
root@chensiqi2's password: 
total 4
-rw-r--r--. 1 rsync rsync 166 Mar  6 21:02 opt.tar.gz
```

#### 1.6.5.2 推送测试2：将客户端任意目录推送到rsync服务器端指定目录下

**测试命令：**

```
rsync -avzP /tmp/ rsync_backup@192.168.197.129::backup --password-file=/etc/rsync.password
```

**演示过程：**

```
[root@chensiqi backup]# rsync -avzP /tmp/ rsync_backup@192.168.197.129::backup --password-file=/etc/rsync.password 
sending incremental file list
./
opt.tar.gz2017-03-06
         162 100%    0.00kB/s    0:00:00 (xfer#1, to-check=5/8)
backup/
opt/
opt/chensiqi
           0 100%    0.00kB/s    0:00:00 (xfer#2, to-check=1/8)
opt/rh/

sent 441 bytes  received 62 bytes  1006.00 bytes/sec
total size is 162  speedup is 0.32
[root@chensiqi backup]# ssh root@chensiqi2 "ls /backup"   #看一眼结果
root@chensiqi2's password: 
backup
opt
opt.tar.gz2017-03-06
```

#### 1.6.5.3 拉取测试1：将rsync服务器端指定目录全部内容同步到客户端

**测试命令：**

```
rsync -avzP rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password

命令说明：
和推送相比，只是两个目录换了个位置。
```

**演示过程：**

```
[root@chensiqi backup]# ls
[root@chensiqi backup]# rsync -avzP rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password 
receiving incremental file list
./
a
           0 100%    0.00kB/s    0:00:00 (xfer#1, to-check=1/3)
opt.tar.gz
         166 100%  162.11kB/s    0:00:00 (xfer#2, to-check=0/3)

sent 105 bytes  received 389 bytes  988.00 bytes/sec
total size is 166  speedup is 0.34
[root@chensiqi backup]# ls
a  opt.tar.gz
```

#### 1.6.5.4 拉取测试2：将rsync服务器端指定目录下的指定内容同步到客户端

**测试命令：**

```
rsync -avzP rsync_backup@192.168.197.129::backup/opt.tar.gz /backup/ --password-file=/etc/rsync.password
```

**演示过程：**

```
[root@chensiqi backup]# ls
[root@chensiqi backup]# rsync -avzP rsync_backup@192.168.197.129::backup/opt.tar.gz /backup/ --password-file=/etc/rsync.password 
receiving incremental file list
opt.tar.gz
         166 100%  162.11kB/s    0:00:00 (xfer#1, to-check=0/1)

sent 83 bytes  received 328 bytes  822.00 bytes/sec
total size is 166  speedup is 0.40
[root@chensiqi backup]# ls
opt.tar.gz
```

#### 1.6.5.5 拉取测试3: 将rsync服务器端指定目录下的全部内容排除某目录或文件后，同步到客户端

**环境准备**
 我们在rsync服务器端指定目录下创建如下文件结构

```
[root@chensiqi2 backup]# ls
a  b  c  chen  d  e
[root@chensiqi2 backup]# ls chen
1  2  3  4  5

说明：
a，b，c，d，e为文件，chen是目录。目录下有1，2，3，4，5文件
```

**方法一：通过命令行实现排除**

**测试命令：**

```
rsync -avz --exclude=a --exclude=chen/3 --exclude=chen/4 rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password 

命令说明：
--exlude=文件名 ：排除的文件
```

**演示过程：**

```
[root@chensiqi backup]# rsync -avz --exclude=a --exclude=chen/3 --exclude=chen/4 rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password 
receiving incremental file list
./
.pwd.lock
b
c
d
e
.ICE-unix/
chen/
chen/1
chen/2
chen/5

sent 258 bytes  received 558 bytes  1632.00 bytes/sec
total size is 0  speedup is 0.00

[root@chensiqi backup]# ls
b  c  chen  d  e
[root@chensiqi backup]# ls chen
1  2  5
```

**方法二：通过列表文件实现排除**

**创建排除列表文件**

```
[root@chensiqi backup]# cat /root/exclude.txt 
1
3
5
b
e
```

**测试命令：**

```
rsync -avz --exclude-from=/root/exclude.txt rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password

命令说明：
--exclude-from=文件的绝对路径 ：引用一个排除列表，列表里只需要输入排除的文件名即可
```

**演示过程：**

```
[root@chensiqi backup]# cat /root/exclude.txt 
1
3
5
b
e
[root@chensiqi backup]# rsync -avz --exclude-from=/root/exclude.txt rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password 
receiving incremental file list
./
a
c
d
chen/
chen/2
chen/4

sent 202 bytes  received 434 bytes  1272.00 bytes/sec
total size is 0  speedup is 0.00
[root@chensiqi backup]# ls
a  c  chen  d
[root@chensiqi backup]# ls chen
2  4
```

#### 1.6.5.6 rsync同步拉取测试：让rsync客户端指定目录内容始终和rsync服务器共享目录内容保持一致

1）和rsync服务器目录内容始终保持一致

> 始终保持一致的意思是说，当Rsync服务器共享目录增加文件，那么客户端指定目录也增加，服务器端共享目录删除文件，那么客户端指定目录也删除文件

**测试命令：**

```
rsync -avz --delete rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password 

命令说明：
--delete ：表示同步增，删,改（文件内容出现变化，也会同步的）
```

**演示过程：**

```
[root@chensiqi backup]# rsync -avz --delete rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password     #进行第一次同步
receiving incremental file list
./
a
b
c
d
e
chen/
chen/1
chen/2
chen/3
chen/4
chen/5

sent 262 bytes  received 663 bytes  1850.00 bytes/sec
total size is 8  speedup is 0.01
[root@chensiqi backup]# ls   #查看同步后的文件
a  b  c  chen  d  e
[root@chensiqi backup]# ssh root@chensiqi2 "rm -rf /backup/a"  #远程删除Rsync服务器共享目录下的文件a
root@chensiqi2's password: 
[root@chensiqi backup]# rsync -avz --delete rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password      #进行第二次同步
receiving incremental file list
deleting a   #可以看到同步过程中进行了一次delete同步
./

sent 69 bytes  received 278 bytes  694.00 bytes/sec
total size is 0  speedup is 0.00
[root@chensiqi backup]# ls  #查看同步结果，文件a消失了。
b  c  chen  d  e
[root@chensiqi backup]# ssh root@chensiqi2 "echo 1111 >/backup/chensiqi" #远程在rsync服务器端共享目录下创建一个有内容的文件chensiqi
root@chensiqi2's password: 
[root@chensiqi backup]# rsync -avz --delete rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password   #进行第三次同步
receiving incremental file list
./
chensiqi    #新增了chensiqi文件

sent 88 bytes  received 337 bytes  850.00 bytes/sec
total size is 5  speedup is 0.01
[root@chensiqi backup]# cat chensiqi    #查看同步后的文件内容
1111

[root@chensiqi backup]# ssh root@chensiqi2 "echo 222 >>/backup/chensiqi" #远程对rsync服务器端共享目录下的chensiqi文件增加一行内容。
root@chensiqi2's password: 
[root@chensiqi backup]# rsync -avz --delete rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password      #进行第四次同步
receiving incremental file list
chensiqi  #修改后的文件也被同步了

sent 91 bytes  received 338 bytes  858.00 bytes/sec
total size is 9  speedup is 0.02
[root@chensiqi backup]# cat chensiqi  #查看同步后文件内容
1111
222
```

2)排除某文件后，再和服务器进行同步

**测试命令：**

```
rsync -avz --delete --exclude=c rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password 

命令说明：
--exclude=c：同步时不考虑文件名为c的文件
```

**演示过程：**

```
[root@chensiqi backup]# ls   #查看目录下内容
b  c  chen  chensiqi  d  e
[root@chensiqi backup]# rsync -avz --delete --exclude=c rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password   #第一次同步
receiving incremental file list

sent 73 bytes  received 283 bytes  237.33 bytes/sec
total size is 9  speedup is 0.03
[root@chensiqi backup]# ssh root@chensiqi2 "rm -rf /backup/c"  #远程删除服务器端c文件
root@chensiqi2's password: 
[root@chensiqi backup]# rsync -avz --delete --exclude=c rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password   #第二次同步
receiving incremental file list
./          #没有同步到任何东西

sent 76 bytes  received 286 bytes  241.33 bytes/sec
total size is 9  speedup is 0.02
[root@chensiqi backup]# ssh root@chensiqi2 "touch /backup/c"   #远程创建c文件
root@chensiqi2's password: 
[root@chensiqi backup]# rsync -avz --delete --exclude=c rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password     #第三次同步
receiving incremental file list
./     #还是没有同步到任何东西

sent 76 bytes  received 286 bytes  241.33 bytes/sec
total size is 9  speedup is 0.02
[root@chensiqi backup]# ssh root@chensiqi2 "echo 111 >>/backup/c"  #远程修改c文件
root@chensiqi2's password: 
[root@chensiqi backup]# rsync -avz --delete --exclude=c rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password   #第四次同步
receiving incremental file list
                                             #仍旧没有同步到任何东西
sent 73 bytes  received 283 bytes  712.00 bytes/sec
total size is 9  speedup is 0.03
```

#### 1.6.5.7 rsync同步推送测试：让Rsync服务器端共享目录始终和rsync客户端指定目录内容一致。

1）和rsync客户端目录内容始终保持一致

> 始终保持一致的意思是说，当Rsync客户端指定目录增加文件，那么服务器端共享目录也增加，客户端指定目录删除文件，那么服务器端共享目录也删除文件

**测试命令：**

```
rsync -avz --delete /backup/ rsync_backup@192.168.197.129::backup --password-file=/etc/rsync.password 

命令说明：
--delete ：表示同步增，删,改（文件内容出现变化，也会同步的）
与同步拉取相比：只是客户端目录放在了服务器端的前边。
```

**演示过程：**

```
[root@chensiqi backup]# ls
a  b  c  chen  d  e
[root@chensiqi backup]# rsync -avz --delete /backup/ rsync_backup@192.168.197.129::backup --password-file=/etc/rsync.password #第一次同步
sending incremental file list
./
a
b
c
d
e
chen/
chen/1
chen/2
chen/3
chen/4
chen/5

sent 594 bytes  received 206 bytes  533.33 bytes/sec
total size is 0  speedup is 0.00

[root@chensiqi backup]# rm a   #客户端删除文件a
rm: remove regular empty file `a'? y
[root@chensiqi backup]# rsync -avz --delete /backup/ rsync_backup@192.168.197.129::backup --password-file=/etc/rsync.password  #第二次同步
sending incremental file list
./
deleting a  #同步了删除文件a的过程

sent 225 bytes  received 13 bytes  476.00 bytes/sec
total size is 0  speedup is 0.00
[root@chensiqi backup]# touch a   #创建文件a
[root@chensiqi backup]# rsync -avz --delete /backup/ rsync_backup@192.168.197.129::backup --password-file=/etc/rsync.password #第三次同步
sending incremental file list
./
a  #推送了a文件到服务器端

sent 271 bytes  received 32 bytes  606.00 bytes/sec
total size is 0  speedup is 0.00

[root@chensiqi backup]# echo 1111 >>a  #修改文件a
[root@chensiqi backup]# cat a
1111
[root@chensiqi backup]# rsync -avz --delete /backup/ rsync_backup@192.168.197.129::backup --password-file=/etc/rsync.password #第四次同步
sending incremental file list
a      #同步了修改后的文件a到服务器端

sent 277 bytes  received 29 bytes  612.00 bytes/sec
total size is 5  speedup is 0.02

[root@chensiqi backup]# ssh root@chensiqi2 "cat /backup/a"  #远程查看一下服务器端共享目录下的文件a的内容
root@chensiqi2's password: 
1111
```

2)--exclude=文件名。排除某文件后，再和服务器进行同步

**和同步拉取排除完全一致，只是目录换个位置，在此不在多费篇幅，同学们自己测试。**

### 1.6.6 Rsync企业应用之风险提示

> **特别说明：**
>  执行--delete参数从rsync服务器端往rsync客户端拉取数据时，一定要小心，最好不用，它比从rsync客户端带--delete参数往rsync服务端推送危险的多。客户端带--delete参数往服务端推送仅删除服务端模块下的数据，而前者有能力删除rsync客户端本地的所有数据包括跟下的所有目录。

**rsync推送企业工作场景：**
 1）备份 --delete 风险
 本地有啥，远端就有啥，本地没有的远端有也要删除。服务器端的目录数据可能丢失。

**rsync拉取企业工作场景：**
 1）代码发布，下载。--delete风险
 远端有啥，本地（客户端）就有啥，远端没有的本地有也要删除。本地的目录数据可能丢失。

### 1.6.7 rsync无差异同步的生产场景应用

> 一般是有需要两台服务器之间，必须要求数据一致，且时时性又不是很高的情况下，如两台负载均衡下面web服务器之间的同步，或者高可用双机配置之间的同步等，rsync无差异同步非常的危险，而且，有很多的替代方案，因此，生产场景没有特殊的需求，应避免使用。切记，有很多朋友都已经有了血的教训。

## 1.7 Rsync 优缺点

### 1.7.1 rsync优点：

1，增量备份，支持socket（daemon），集中备份（支持推拉，都是以客户端为参照物）。
 2，远程SHELL通道模式还可以加密（SSH）传输，socket（daemon）需要加密传输，可以利用vpn服务或ipsec服务

### 1.7.2 rsync缺点：

1，大量小文件时候同步的时候，比对时间较长，有的时候，同步过程中，rsync进程可能会停止，僵死了。
 2，同步大文件，10G这样的大文件有时也会出问题，中断。未完整同步前，是隐藏文件，可以通过续传（--partial）等参数实现传输
 3，一次性远程拷贝可以用scp，大量小文件要打成一个包再拷贝。（重要）

## 1.8 排错必备思想

- 部署流程步骤熟练
- rsync原理理解
- 学会看日志，rsync命令行输出，日志文件/var/log/rsyncd.log

## 1.9 Rsync守护进程服务传输数据排错思路：

### 1.9.1 Rsync服务端排错思路

1. 查看rsync服务配置文件路径是否正确，正确的默认路径为：/etc/rsyncd.conf
2. 查看配置文件里host allow,host deny，允许的IP网段是否是允许客户端访问的ip网段
3. 查看配置文件中path参数里的路径是否存在，权限是否正确（正常应为配置文件中的UID参数对应的属主和组）
4. 查看rsync服务是否启动。查看命令为：ps -ef|grep rsync。端口是否存在netstat -antup |grep 873
5. `查看iptables防火墙和selinux是否开启允许rsync服务通过，也可以考虑关闭`。
6. 查看服务端rsync配置的密码文件是否为600的权限，密码文件格式是否正确，正确格式为：用户名：密码，文件路径和配置文件里的secrect files参数对应。
7. 如果是推送数据，要查看下，配置rsyncd.conf文件中用户是否对模块下目录有可读写的权限。

### 1.9.2 Rsync客户端拍错思路

1. 查看客户端rsync配置的密码文件是否600的权限，密码文件格式是否正确，注意：仅需要有密码，并且和服务器端的密码保持一致。
2. 用telnet连接rsync服务器ip地址873端口，查看服务是否启动（可测试服务端防火墙是否阻挡）telnet 192.168.197.129 873
3. 客户端执行命令时：`rsync -avzP rsync_backup@192.168.197.129::backup /backup/ --password-file=/etc/rsync.password`
4. 此命令的细节要记清楚，尤其192.168.197.129::backup 处的双冒号及其后的backup为模块名称

## 2【Rsync项目实战】备份全网服务器数据生产架构方案案例模型（必做）

**【企业案例】：**
 某公司里有一台Web服务器，里面的数据很重要，但是如果硬盘坏了，数据就会丢失，现在领导要求你把数据在其他机器上做一个周期性定时备份。要求如下：

> 每天晚上00电整在Web服务器A上打包备份网站程序目录并通过rsync命令推送到服务器B上备份保存（备份思路可以是先在本地按日期打包，然后再利用rsync推送到备份服务器上）。

**具体要求如下：**
 1）NFS服务器nfs01和备份服务器backup的备份目录必须都为/backup
 2）NFS服务器站点目录假定为（/var/www/html）
 3）NFS服务器本地仅保留7天内的备份。
 4）备份服务器上检查备份结果是否正常，并将每天的备份结果发给管理员信箱。
 5）备份服务器上每周六的数据都保留，其他备份仅保留180天备份。

## 附录1: rsyncd.conf配置文件常用参数说明：

| rsyncd.conf参数     | 参数说明                                                     |
| ------------------- | ------------------------------------------------------------ |
| uid=rsync           | #rsync使用的用户。                                           |
| gid=rsync           | #rsync使用的用户组（用户所在的组）                           |
| use chroot=no       | #如果为true，daemon会在客户端传输文件前“chroot to the path”。这是一种安全配置，因为我们大多数都在内网，所以不配也没关系 |
| max connections=200 | #设置最大连接数，默认0，意思无限制，负值为关闭这个模块       |
| timeout=400         | #默认为0，表示no timeout，建议300-600（5-10分钟）            |
| pid file            | #rsync daemon启动后将其进程pid写入此文件。如果这个文件存在，rsync不会覆盖该文件，而是会终止 |
| lock file           | #指定lock文件用来支持“max connections”参数，使得总连接数不会超过限制 |
| log file            | #不设或者设置错误，rsync会使用rsyslog输出相关日志信息        |
| ignore errors       | #忽略I/O错误                                                 |
| read only=false     | #指定客户端是否可以上传文件，默认对所有模块为true            |
| list=false          | #是否允许客户端可以查看可用模块列表，默认为可以              |
| hosts allow         | #指定可以联系的客户端主机名或和ip地址或地址段，默认情况没有此参数，即都可以连接 |
| hosts deny          | #指定不可以联系的客户端主机名或ip地址或地址段，默认情况没有此参数，即都可以连接 |
| auth users          | #指定以空格或逗号分隔的用户可以使用哪些模块，用户不需要在本地系统中存在。默认为所有用户无密码访问 |
| secrets file        | #指定用户名和密码存放的文件，格式；用户名；密码，密码不超过8位 |
| [backup]            | #这里就是模块名称，需用中括号扩起来，起名称没有特殊要求，但最好是有意义的名称，便于以后维护 |
| path                | #这个模块中，daemon使用的文件系统或目录，目录的权限要注意和配置文件中的权限一致，否则会遇到读写的问题 |

> **特别说明**：
>  1）模块中的参数项可以拿到全局配置中使用
>  2）以上配置文件中的参数，为生产中经常使用的参数，初学者掌握这些足够了。
>  3）以上配置文件中没有提到的参数请参考man rsyncd.conf查看

## 附录2: 开发rsync服务启动脚本

```
#!/bin/bash
#author:Mr.chen
# chkconfig:35 13 91
# description:This is Rsync service management shell script


# Source function library
. /etc/rc.d/init.d/functions


start(){
    rsync --daemon
    if [ $? -eq 0 -a `ps -ef|grep -v grep|grep rsync|wc -l` -gt 0 ];then
        action "Starting Rsync:" /bin/true
        sleep 1
    else
        action "Starting Rsync:" /bin/false
        sleep 1
    fi
}

stop(){
    pkill rsync;sleep 1;pkill rsync
    if [ `ps -ef|grep -v grep|grep "rsync --daemon"|wc -l` -lt 1 ];then
        action "Stopping Rsync: " /bin/true
        sleep 1
    else
        action "Stopping Rsync:" /bin/true
        sleep 1
    fi
}

case "$1" in
    start)
        start;
        ;;
    stop)
        stop;
        ;;
    restart|reload)
        stop;
        start;
        ;;
    *)
        echo $"Usage: $0 {start|stop|restart|reload}"
        ;;
esac
```

保存成rsyncd，放到/etc/init.d/rsyncd

```
[root@chensiqi2 ~]# cp rsync /etc/init.d/rsyncd
[root@chensiqi2 ~]# chmod +x /etc/init.d/rsyncd 
[root@chensiqi2 ~]# /etc/init.d/rsyncd stop
已终止
[root@chensiqi2 ~]# /etc/init.d/rsyncd start
Starting Rsync:                                            [确定]
```

