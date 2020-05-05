[TOC]

# 简介

- 2049/TCP,UDP

- 基于IP的认证 

- RPC（Remote Procedure Call protocol）：远程过程调用协议，由本地发起函数调用请求，通过远程主机上的函数处理完成。

- RPC服务：C5:portmap,C6:rpcbind，监听端口为111，可用命令rpcinfo -p查看

- NIS（Network Information System）网络信息系统，帐号信息统一集中管理，集中于某服务器进行用户认证因是明文传送数据，通常用于局域网内。



# NFS服务需要的进程

nfsd进程：用于执行客户端函数调用请求

idmapd进程：用于将客户端的用户统一映射为本地的nfsnobody用户

mountd进程：用于验证远程客户端是否有权限访问本地nfs服务

# NFS的安装

NFS的安装是非常简单的，只需要两个软件包即可，而且在通常情况下，是作为系统的默认包安装的,这里以c6为例：

- nfs-utils：NFS主程序包,包括rpc.nfsd,rpc.mountd,daemons

- rpcbind：RPC主程序

```
# yum install nfs-utils rpmbind -y
# rpm -qa | grep nfs
# rpm -qa | grep rpmbind
```

守护进程：

nfsd：它是基本的NFS守护进程，主要功能是管理客户端是否能够登录服务器；

mountd：它是RPC安装守护进程，主要功能是管理NFS的文件系统。当客户端顺利通过nfsd登录NFS服务器后，在使用NFS服务所提供的文件前，还必须通过文件使用权限的验证。它会读取NFS的配置文件/etc/exports来对比客户端权限。

portmap：主要功能是进行端口映射工作。当客户端尝试连接并使用RPC服务器提供的服务（如NFS服务）时，portmap会将所管理的与服务对应的端口提供给客户端，从而使客户可以通过该端口向服务器请求服务。

# NFS配置

NFS服务器的配置相对比较简单，只需要在相应的配置文件中进行设置，然后启动NFS服务器即可。

NFS的常用目录：

/etc/exports              NFS服务的主要配置文件

/usr/sbin/exportfs          NFS服务的管理命令

/usr/sbin/showmount        客户端的查看命令

/var/lib/nfs/etab            记录NFS分享出来的目录的完整权限设定值

/var/lib/nfs/xtab            记录曾经登录过的客户端信息

NFS服务的配置文件为 /etc/exports，这个文件是NFS的主要配置文件，不过系统并没有默认值，所以这个文件不一定会存在，可能要使用vim手动建立，然后在文件里面写入配置内容。

## /etc/exports格式

<输出目录> [客户端1 选项（访问权限,用户映射,其他）] [客户端2 选项（访问权限,用户映射,其他）]

### 输出目录

输出目录是指NFS系统中需要共享给客户机使用的目录；

### 客户端

客户端是指网络中可以访问这个NFS输出目录的计算机

客户端常用的指定方式

指定ip地址的主机：192.168.0.200

指定子网中的所有主机：192.168.0.0/24 192.168.0.0/255.255.255.0

指定域名的主机：david.bsmart.cn

指定域中的所有主机：*.bsmart.cn

所有主机：*

### 选项

选项用来设置输出目录的访问权限、用户映射等。

NFS主要有3类选项：

访问权限选项：

sync，wdelay，hide 等等，no_root_squash 是让root保持权限，root_squash 是把root映射成nobody，no_all_squash 不让所有用户保持在挂载目录中的权限。所以，root建立的文件所有者是nfsnobody。

设置输出目录只读：ro

设置输出目录读写：rw

### 用户映射选项

all_squash：将远程访问的所有普通用户及所属组都映射为匿名用户或用户组（nfsnobody）；

no_all_squash：与all_squash取反（默认设置）；

root_squash：将root用户及所属组都映射为匿名用户或用户组（默认设置）；

no_root_squash：与rootsquash取反；

anonuid=xxx：将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户（UID=xxx）；

anongid=xxx：将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户（GID=xxx）；

### 其它选项

secure：限制客户端只能从小于1024的tcp/ip端口连接nfs服务器（默认设置）；

insecure：允许客户端从大于1024的tcp/ip端口连接服务器；

sync：将数据同步写入内存缓冲区与磁盘中，效率低，但可以保证数据的一致性；

async：将数据先保存在内存缓冲区中，必要时才写入磁盘；

wdelay：检查是否有相关的写操作，如果有则将这些写操作一起执行，这样可以提高效率（默认设置）；

no_wdelay：若有写操作则立即执行，应与sync配合使用；

subtree：若输出目录是一个子目录，则nfs服务器将检查其父目录的权限(默认设置)；

no_subtree：即使输出目录是一个子目录，nfs服务器也不检查其父目录的权限，这样可以提高效率；



# 查看NFS服务器端共享的文件系统

showmount -e HOST_IP

/etc/exports：

文件系统   客户端1(选项)   客户端2(选项)

客户端：IP,FQDN或DOMAIN、NETWORK

exportfs：维护exports文件导出的文件系统表的专用工具

  -ar：重新导出所有的文件系统

  -au：关闭导出的所有文件系统

  -u FS：关闭指定的导出的某一个文件系统FS

选项：

secure： 这个选项是缺省选项，它使用了 1024 以下的 TCP/IP 端口实现 NFS 的连接。指定 insecure 可以禁用这个选项。

rw： 这个选项允许 NFS 客户机进行读/写访问。缺省选项是只读的。

async： 异步访问。默认启用这个选项可以改进性能，但是如果没有完全关闭 NFS 守护进程就重新启动了 NFS 服务器，这也可          能会造成数据丢失。

no_wdelay： 这个选项关闭写延时。如果设置了 async，那么 NFS 就会忽略这个选项。

nohide： 如果将一个目录挂载到另外一个目录之上，那么原来的目录通常就被隐藏起来或看起来像空的一样。要禁用这种行为，          需启用 hide 选项。

no_subtree_check： 这个选项关闭子树检查，子树检查会执行一些不想忽略的安全性检查。缺省选项是启用子树检查。

no_auth_nlm： 这个选项也可以作为 insecure_locks 指定，它告诉 NFS 守护进程不要对加锁请求进行认证。如果关心安全性问                题，就要避免使用这个选项。缺省选项是 auth_nlm 或 secure_locks。

mp (mountpoint=path)： 通过显式地声明这个选项，NFS 要求挂载所导出的目录。

fsid=num： 这个选项通常都在 NFS 故障恢复的情况中使用。如果希望实现 NFS 的故障恢复，请参考 NFS 文档。



# 用户映射

通过 NFS 中的用户映射，可以将伪或实际用户和组的标识赋给一个正在对 NFS 卷进行操作的用户。这个 NFS 用户具有映射所允许的用户和组的许可权限。对 NFS 卷使用一个通用的用户/组可以提供一定的安全性和灵活性，而不会带来很多管理负荷。

在使用 NFS 挂载的文件系统上的文件时，用户的访问通常都会受到限制，这就是说用户都是以匿名用户的身份来对文件进行访问的，这些用户缺省情况下对这些文件只有只读权限。这种行为对于 root 用户来说尤其重要。然而，实际上的确存在这种情况：希望用户以 root 用户或所定义的其他用户的身份访问远程文件系统上的文件。NFS 允许指定访问远程文件的用户——通过用户标识号（UID）和组标识号（GID），可以禁用正常的 squash 行为。

用户映射的选项包括：

- root_squash： 这个选项不允许 root 用户访问挂载上来的 NFS 卷。

- no_root_squash： 这个选项允许 root 用户访问挂载上来的 NFS 卷。

- all_squash： 这个选项对于公共访问的 NFS 卷来说非常有用，它会限制所有的 UID 和 GID，只使用匿名用户。缺省设置是 no_all_squash。

- anonuid 和 anongid： 这两个选项将匿名 UID 和 GID 修改成特定用户和组帐号。



客户端挂载时可以使用的特殊选项

Client

- Mounting remote directories

- Before mounting remote directories 2 daemons should be be started first:

- rpcbind

- rpc.statd

rsize 的值是从服务器读取的字节数。wsize 是写入到服务器的字节数。默认都是1024， 如果使用比较高的值，如8192,可以提高传输速度。 

# 相关的命令

## exportfs

如果我们在启动了NFS之后又修改了/etc/exports，是不是还要重新启动nfs呢？这个时候我们就可以用exportfs 命令来使改动立刻生效，该命令格式如下：

　　# exportfs [-aruv]

　　-a 全部挂载或卸载 /etc/exports中的内容 

　　-r 重新读取/etc/exports 中的信息 ，并同步更新/etc/exports、/var/lib/nfs/xtab

　　-u 卸载单一目录（和-a一起使用为卸载所有/etc/exports文件中的目录）

　　-v 在export的时候，将详细的信息输出到屏幕上。

具体例子： 

　　# exportfs -au 卸载所有共享目录

　　# exportfs -rv 重新共享所有目录并输出详细信息

## nfsstat

查看NFS的运行状态，对于调整NFS的运行有很大帮助。

## rpcinfo

查看rpc执行信息，可以用于检测rpc运行情况的工具，利用rpcinfo -p 可以查看出RPC开启的端口所提供的程序有哪些。

## showmount

　　-a 显示已经于客户端连接上的目录信息

　　-e IP或者hostname 显示此IP地址分享出来的目录

## netstat

可以查看出nfs服务开启的端口，其中nfs 开启的是2049，portmap 开启的是111，其余则是rpc开启的。

最后注意两点，虽然通过权限设置可以让普通用户访问，但是挂载的时候默认情况下只有root可以去挂载，普通用户可以执行sudo。

# 实例

NFS服务器：192.168.100.9

共享目录为/www/htdocs

要求：在客户端上挂载NFS服务器上的/www/htdocs到本地的/web/html目录下

1，在服务器上共享出/www/htdocs

```
# vim /etc/exports
/www/htdocs 192.168.100.9/24(rw,sync,no_root_squash)
# service portmap restart
# service nfs restart
# exportfs
# showmount -e   \\查看自己的共享状态(DNS可解析自己)
# showmount -a   \\可查看当前客户端连接的信息
```

2,在客户端上挂载nfs

```
# mount -t nfs 192.168.100.9:/www/htdocs /web/html
```

3，设置开机自动挂载

fstab格式：

<server>:</remote/export> </local/directory> nfs < options> 0 0

```
开机自动挂载nfs: 
# vim /etc/fstab
192.168.100.9:/www/htdocs           /web/html           nfs         defaults      0 0
```

4，卸载已挂载的NFS共享目录

```
# umount /web/html
```



# 总结

1、客户端表示方式

2、导出选项：

​	rw, async, sync, root_squash, no_root_squash, all_squash, anonuid, anongid

3、exportfs和showmount



# 有可能出现的错误：

客户端挂载nfs提示：

\# mount -t nfs 192.168.100.9:/www/htdocs/ /web/html/

mount: wrong fs type, bad option, bad superblock on 192.168.100.9:/www/htdocs/,

​    missing codepage or helper program, or other error

​    (for several filesystems (e.g. nfs, cifs) you might

​    need a /sbin/mount.<type> helper program)

​    In some cases useful info is found in syslog - try

​    dmesg | tail  or so

解决办法：

查看/sbin/mount*

没有mounts-nfs

需要安装nfs-utils

