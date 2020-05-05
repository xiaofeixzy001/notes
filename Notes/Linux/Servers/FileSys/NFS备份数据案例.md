[TOC]

# [NFS项目实战一]备份全网服务器数据

## 背景

某公司里有一台NFS服务器，里面的数据很重要，但是如果硬盘坏了，数据就会丢失，现在领导要求你把数据在其他 机器上做一个周期性定时备份。要求如下:

每天晚上00点整在NFS服务器nfs01上打包备份网站程序目录等并通过rsync命令推送到备份服务器backup上备份保存(备份思路 可以是先在本地按IP地址+日期打包，然后再利用rsync推送到备份服务器上)。NFS存储服务器同Web服务器，实际工作中就是全部的服务器。

## 目标

- NFS服务器nfs01和备份服务器backup的备份目录必须都为/backup
- 要备份的系统配置文件包括但不限于：
  定时任务服务的配置文件（/var/spool/cron/root）
  开机自启动的配置文件（/etc/rc.local）
  日常脚本的目录（/server/scripts）
  防火墙iptables的配置文件（/etc/sysconfig/iptables）
  自己思考下还有什么需要备份呢
- Web服务器站点目录假定为（/var/html/www）
- Web服务器A访问日志路径假定为（/app/logs）
- Web服务器保留打包后的7天的备份数据即可（本地留存不能多于7天，因为太多硬盘会满）
- 备份服务器上，保留每周一的所有数据副本，其它要保留6个月的数据副本
- 备份服务器上要按照备份数据服务器的内网IP为目录保存备份，备份的文件按照时间名字保存。
- 需要确保备份的数据尽量完整正确，在备份服务器上对备份的数据进行检查，把备份的成功及失败结果信息发送给系统管理员邮箱中。

## 环境

主机网络参数设置：

| 主机名 | 网卡eth0    | 网卡eth1      | 用途                |
| ------ | ----------- | ------------- | ------------------- |
| backup | 10.0.0.41   | 172.16.1.41   | rsync服务端         |
| nfs01  | 10.0.0.31   | 172.16.1.31   | NFS存储服务器客户端 |
| web01  | 10.0.0.8/24 | 172.16.1.8/24 | nginx web服务器     |

```shell
[root@backup ~]# cat /etc/redhat-release 
CentOS release 6.9 (Final)

[root@backup ~]# uname -r
2.6.32-642.el6.x86_64
```



# 部署

## 备份服务器

配置文件：

```SHELL
#rsync_config____start
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
###################################
[backup]
# 使用目录
path = /backup/
# 有错误时忽略
ignore errors
# 可读可写（true或false）
read only = false
# 阻止远程列表（不让通过远程方式看服务端有啥）
list = false
# 允许IP
hosts allow = 172.16.1.0/24
# 禁止IP
hosts deny = 0.0.0.0/32
# 虚拟用户
auth users = rsync_backup
# 存放用户和密码的文件
secrets file = /etc/rsync.password

##rsync_config____end##
```

创建rsync账户及共享目录并修改属主

```shell
useradd -M -s /sbin/nologin rsync
mkdir /backup
chown -R rsync /backup

rsync --daemon
ss -antup | grep rsync
```

服务启动脚本/etc/init.d/rsyncd

```shell
#!/bin/bash
# author: XF
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

设置执行权限，开机执行

```shell
chmod +x /etc/init.d/rsyncd
/etc/init.d/rsyncd stop
/etc/init.d/rsyncd start
chkconfig rsyncd on
chkconfig --list | grep rsync
```

创建rsync虚拟账户和密码，并修改密码文件权限为600

```shell
echo "rsync_backup:123456" > /etc/rsync.password
cat /etc/rsync.password
chmod 600 /etc/rsync.password
ll /etc/rsync.password
# -rw-------. 1 root root 20 3月   7 20:54 /etc/rsync.password
```

服务器端检查脚本

```shell
#!/bin/bash
# 全网服务器备份解决方案_rsync服务器端检查脚本
# author:Mr.chen
# 2017-3-8

. /etc/init.d/functions
Path=/backup
fileName="md5sum.txt"
# 一共有几台客户端在推送数据
rsync_ClientNum=2

/etc/init.d/postfix status &>/dev/null || /etc/init.d/postfix start


if [ `find $Path/ -type f -name "md5sum*" | wc -l` -eq $rsync_ClientNum ];then
    for filepath in `find $Path/ -type f -name "md5sum*"`
    do
        /usr/bin/md5sum -c $filepath
        if [ $? -eq 0 ];then
            action "${filepath}备份正常！" /bin/true
            rm -rf $filepath
        else
            action "${filepath}备份异常！" /bin/false
            echo "${filepath}备份异常！" | mail -s "$(date +%F)备份检查告警" xxxxxxxx@qq.com
        fi
    done
else
    echo “Rsync客户端推送不完整！”
    echo "Rsync推送不完整" | mail -s "$(date +%F)备份推送告警" xxxxxxxxx@qq.com
fi

# 找出超过180天的不是周1的备份文件并删除
find $Path/ ! -name "*_2.tar.gz" -mtime +180 -type f | xargs rm -rf
```

设置定时任务

```shell
crontab -e
00 6 * * * /bin/sh /server/scripts/rsync_Server.sh >/dev/null 2>&1
```

备份服务器配置完毕。

# NFS服务器

```shell
# 只需要创建密码文件（只包含密码即可），并赋予密码文件600权限
echo "123456" > /etc/rsync.password
chmod 600 /etc/rsync.password

# 创建共享目录backup
mkdir /backup

# 在客户端进行推送测试
cd /backup
touch {1..5}
ls
rsync -avzP /backup/ rsync_backup@192.168.197.132::backup --password-file=/etc/rsync.password
```

打包脚本

```shell
#!/bin/bash
# 全网服务器备份解决方案_rsync客户端打包脚本
# 2017-3-7

Path=/backup
backup_Server=172.16.1.41
local_IP=`/sbin/ifconfig eth1|awk -F"[ :]+" 'NR==2{print $4}'`
Dir=${local_IP}_$(date +%F_%w)


mkdir -p $Path/$Dir
[ -f /var/spool/cron/root ] && cp -rp /var/spool/cron/root $Path/$Dir/
[ -f /etc/rc.d/rc.local ] && cp -rp /etc/rc.d/rc.local $Path/$Dir/
[ -d /server/scripts ] && cp -rp /server/scripts $Path/$Dir/
[ -d /var/html/www ] && cp -rp /var/html/www $Path/$Dir/
[ -d /app/logs ] && cp -rp /app/logs $Path/$Dir/
[ -f /etc/sysconfig/iptables ] && cp -rp /etc/sysconfig/iptables $Path/$Dir/
cd $Path

tar -zcf $Path/${Dir}.tar.gz $Dir

rm -rf $Path/$Dir
# 创建md5sum验证信息
/usr/bin/md5sum $Path/${Dir}.tar.gz > $Path/md5sum_$(local_IP).txt

# 推送打包的文件到备份服务器
rsync -az $Path/ rsync_backup@${backupServer}::backup --password-file=/etc/rsync.password
# 找出超过7天的备份并删除
find $Path/ -name "${local_IP}*" -type f -mtime +7 | xargs rm -rf
```

加入定时任务

```shell
00 0 * * * /bin/sh /server/scripts/backup.sh >/dev/null 2>&1
```

NFS服务器Rsync客户端至此配置完毕。



# [NFS项目实战二]NFS共享数据的时时同步推送备份

## 企业案例

公司有两台web服务器一直在对外提供服务，但随着业务的发展用户越来越多，网站的功能也越来越强大，各种图片，视频等占用硬盘空间越来越大。于是，领导将web服务器的数据直接存储到NFS服务器上作为存储使用；并且为了防止NFS服务器发生单点故障，领导希望将web服务器存储的内容实时同步到Rsync备份服务器上。现在由你来计划完成领导的需求。

**具体要求如下：**

- [x] NFS服务器的要求如下：
  - 服务器的共享目录名为/data目录；
  - 权限要求只能内网网段访问且可读可写，时时同步；
  - 为了方便管理人员管理，需要指定NFS虚拟账户为chensiqi，uid=12306，gid=12306
  - 所有访问者的身份都压缩为最低身份
  - 将/data目录里的内容同步时时推送到备份服务器的/data目录里（inotify+rsync）
- [x] web服务器将NFS共享目录统一挂载到/var/html/www目录下

## 环境准备

**系统版本**

```
[root@nfs01 ~]# cat /etc/redhat-release 
CentOS release 6.8 (Final)
```

**内核参数**

```
[root@nfs01 ~]# uname -r
2.6.32-642.el6.x86_64
```

**主机网络参数设置**

| 主机名 | 外网网卡     | 内网网卡       | 用途                |
| ------ | ------------ | -------------- | ------------------- |
| web02  | 10.0.0.7/24  | 172.16.1.7/24  | B1-apache web服务器 |
| web01  | 10.0.0.8/24  | 172.16.1.8/24  | B2-nginx web服务器  |
| nfs01  | 10.0.0.31/24 | 172.16.1.31/24 | C1-NFS存储服务器    |
| backup | 10.0.0.41/24 | 172.16.1.41/24 | C2-rsync备份服务器  |

## 一，开始部署NFS服务器端nfs共享

### 第一步：NFS软件包安装

```
yum -y install nfs-utils rpcbind
```

### 第二步：创建uid=12306，gid=12306的用户chensiqi

```
[root@nfs01 ~]# useradd -u 12306 -s /sbin/nologin -M chensiqi
[root@nfs01 ~]# id chensiqi
uid=12306(chensiqi) gid=12306(chensiqi) 组=12306(chensiqi)
```

### 第三步：修改/etc/exports配置文件

```
[root@nfs01 ~]# echo "/data 172.16.1.0/24(rw,sync,all_squash,anonuid=12306,anongid=12306)" >> /etc/exports 
[root@nfs01 ~]# cat /etc/exports
/data 172.16.1.0/24(rw,rsync,all_squash,anonuid=12306,anongid=12306)
```

### 第四步：启动NFS相关服务

先启动rpcbind服务；再启动nfs服务

```
[root@nfs01 ~]# /etc/init.d/rpcbind start
正在启动 rpcbind：                                         [确定]
[root@nfs01 ~]# /etc/init.d/nfs start
启动 NFS 服务：                                            [确定]
关掉 NFS 配额：                                            [确定]
启动 NFS mountd：                                          [确定]
启动 NFS 守护进程：                                        [确定]
Starting RPC idmapd:                                       [  OK  ]
[root@nfs01 ~]# 
```

### 第五步：设置共享目录/data的属主和属组为指定用户

```
[root@nfs01 ~]# chown -R chensiqi.chensiqi /data
[root@nfs01 ~]# ll -d /data
drwxr-xr-x. 2 chensiqi chensiqi 4096 3月  14 00:14 /data
```

### 第六步：进行本地挂载测试

```
[root@nfs01 ~]# showmount -e  
Export list for nfs01:
/data 172.16.1.0/24
[root@nfs01 ~]# hostname -I
10.0.0.31 172.16.1.31 
[root@nfs01 ~]# mount 172.16.1.31:/data /mnt
[root@nfs01 ~]# ll -d /mnt
drwxr-xr-x. 2 chensiqi chensiqi 4096 3月  14 00:14 /mnt
[root@nfs01 ~]# df
Filesystem           1K-blocks    Used Available Use% Mounted on
/dev/mapper/VolGroup-lv_root
                      18003272 4154188  12927896  25% /
tmpfs                   502068       0    502068   0% /dev/shm
/dev/sda1               487652   34856    427196   8% /boot
172.16.1.31:/data     18003328 4154240  12928000  25% /mnt
```

### 第七步：设置rpcbind和nfs服务开机启动

```
[root@nfs01 ~]# tail -3 /etc/rc.local
#start up nfs service by chensiqi at 20170315
/etc/init.d/rpcbind start
/etc/init.d/nfs start
[root@nfs01 ~]# 
```

## 二，开始部署web端NFS客户端共享挂载

**配置web01服务器：**

### 第一步：nfs客户端需要安装nfs-utils软件包

```
yum -y install nfs-utils
```

### 第二步：挂载共享目录

```
[root@web01 ~]# showmount -e nfs01
Export list for nfs01:
/data 172.16.1.0/24
[root@web01 ~]# mkdir -p /var/html/www
[root@web01 ~]# showmount -e nfs01
Export list for nfs01:
/data 172.16.1.0/24
[root@web01 ~]# mount 172.16.1.31:/data /var/html/www
[root@web01 ~]# df
Filesystem           1K-blocks    Used Available Use% Mounted on
/dev/mapper/VolGroup-lv_root
                      18003272 4815804  12266280  29% /
tmpfs                   502068       0    502068   0% /dev/shm
/dev/sda1               487652   34856    427196   8% /boot
172.16.1.31:/data     18003328 4154240  12928000  25% /mnt
172.16.1.31:/data     18003328 4154240  12928000  25% /var/html/www
```

### 第三步：测试写入数据

```
[root@web01 ~]# cd /var/html/www
[root@web01 www]# ll
总用量 4
-rw-r--r--. 1 chensiqi chensiqi 0 3月  15 19:27 csfdsf
-rw-rw-r--. 1 chensiqi chensiqi 0 3月  14 00:14 test2
-rw-rw-r--. 1 chensiqi chensiqi 4 3月  14 00:14 test.txt
[root@web01 www]# touch 11111
[root@web01 www]# ll
总用量 4
-rw-r--r--. 1 chensiqi chensiqi 0 3月  15 19:34 11111
-rw-r--r--. 1 chensiqi chensiqi 0 3月  15 19:27 csfdsf
-rw-rw-r--. 1 chensiqi chensiqi 0 3月  14 00:14 test2
-rw-rw-r--. 1 chensiqi chensiqi 4 3月  14 00:14 test.txt
```

### 第四步：配置开机自动挂载

```
[root@web01 www]# tail -1 /etc/rc.local 
mount -t nfs -o nodev,noexec,nosuid,rw  172.16.1.31:/data /var/html/www
```

**配置web02服务器：**

配置方式同web01服务器

## 三，配置Rsync备份服务器

> **注意**：由于在项目实战一全网备份里里已经配置过了，所以此处只需要修改一下配置文件

### 第一步：在配置文件/etc/rsyncd.conf里添加nfsbackup新模块

在配置文件里添加如下内容

```
[nfsbackup]
# 使用目录
path = /data/
# 有错误时忽略
ignore errors
# 可读可写（true或false）
read only = false
# 阻止远程列表（不让通过远程方式看服务端有啥）
list = false
# 允许IP
hosts allow = 172.16.1.0/24
# 禁止IP
hosts deny = 0.0.0.0/32
# 虚拟用户
auth users = rsync_backup
# 存放用户和密码的文件
secrets file = /etc/rsync.password
```

### 第二步：启动rsync服务

**方法一：如果没有编写rsync启动脚本**

```
[root@backup ~]# rsync --daemon
[root@backup ~]# ss -antup | grep rsync
tcp    LISTEN     0      5                     :::873                  :::*      users:(("rsync",7098,5))
tcp    LISTEN     0      5                      *:873                   *:*      users:(("rsync",7098,4))
```

**方法二：如果已经编写了启动脚本**

```
[root@backup ~]# /etc/init.d/rsyncd start
Starting Rsync:                                            [确定]
[root@backup ~]# ss -antup | grep rsync
tcp    LISTEN     0      5                     :::873                  :::*      users:(("rsync",7098,5))
tcp    LISTEN     0      5                      *:873                   *:*      users:(("rsync",7098,4))
```

### 第三步：rsync服务加入开机启动

```
[root@backup ~]# echo ". /etc/init.d/rsyncd start" >> /etc/rc.local
[root@backup ~]# tail -1 /etc/rc.local 
. /etc/init.d/rsyncd start
```

## 四，在NFS服务端配置inotify事件监控工具

### 第一步：安装inotify事件监控工具

此工具需要安装epel源
 `wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo`

```
[root@nfs01 ~]# yum -y install inotify-tools
```

### 第二步：进行rsync + inotify时时推送测试

**开两个shell窗口**

```
在第一个窗口输入如下内容：
[root@nfs01 ~]# inotifywait -mrq --format '%w%f' -e delete,close_write,create /data
输入后，shell处于阻塞状态（时时监控）

在另一个窗口的/data目录进行创建，修改，删除测试:
此时我们可以发现当前处于阻塞状态的shell窗口会记录所有目录发生改变的情况

命令说明：
inotifywait：监控命令
-m：持续不断的进行监控（处于阻塞状态）
-r：递归监控，监控目录及目录的所有子目录
-q：只输出简单的监控信息
--format：指定监控数据输出的格式
-e：指定监控的事件类型
delete：删除事件
close_write:文件写入的关闭事件（其实就是监控修改文件）
create：创建事件
```

### 第三步：编写inotify + inotify 时时同步推送脚本

```
#!/bin/bash

Path=/data
backup_Server=172.16.1.41


/usr/bin/inotifywait -mrq --format '%w%f' -e create,close_write,delete /data | while read line
do
        if [ -f $line ];then
                rsync -az $line --delete rsync_backup@$backup_Server::nfsbackup --password-file=/etc/rsync.password
        else
                cd $Path &&\
                rsync -az ./ --delete rsync_backup@$backup_Server::nfsbackup --password-file=/etc/rsync.password
        fi
done
```

### 第四步：脚本加入开机（后台）启动

```
[root@nfs01 ~]# echo "sh /server/scripts/inotify.sh &" >> /etc/rc.local
```

### 第五步：进行同步测试

**NFS存储服务器：进行如下操作**

```
[root@nfs01 ~]# cd /data
[root@nfs01 data]# ll
总用量 4
-rw-r--r--. 1 root root 4 3月  15 21:02 aaa
[root@nfs01 data]# touch chensiqi  #创建
[root@nfs01 data]# ll
总用量 4
-rw-r--r--. 1 root root 4 3月  15 21:02 aaa
-rw-r--r--. 1 root root 0 3月  15 21:16 chensiqi
[root@nfs01 data]# echo 1111 >> chensiqi #修改
[root@nfs01 data]# ll
总用量 8
-rw-r--r--. 1 root root 4 3月  15 21:02 aaa
-rw-r--r--. 1 root root 5 3月  15 21:17 chensiqi
[root@nfs01 data]# rm -rf aaa  #删除
```

**rsync备份服务器：查看目录同步效果**

```
[root@backup ~]# cd /data
[root@backup data]# ll
总用量 4
-rw-r--r--. 1 rsync rsync 5 3月  15 2017 chensiqi
[root@backup data]# cat chensiqi
1111
[root@backup data]# 
```