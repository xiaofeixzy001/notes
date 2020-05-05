[TOC]

# inotify简介

> - Inotify是一种强大的，细粒度的，异步的文件系统事件监控机制（软件），linux内核从2.6.13起，加入了Inotify支持，通过Inotify可以监控文件系统中添加，删除，修改，移动等各种事件，利用这个内核接口，第三方软件就可以监控文件系统下文件的各种变化情况，而inotify-tools正是实施这样监控的软件。还有国人周洋在金山公司开发的sersync。
> - Inotify实际是一种事件驱动机制，它为应用程序监控文件系统事件提供了实时响应事件的机制，而无须通过诸如cron等的轮询机制来获取事件。cron等机制不仅无法做到实时性，而且消耗大量系统资源。相比之下，inotify基于事件驱动，可以做到对事件处理的实时响应，也没有轮询造成的系统资源消耗，是非常自然的事件通知接口，也与自然世界的事件机制相符合。
> - inotify 的实现有几款软件
>    1）inotify-tools，
>    2）sersync（金山周洋）
>    3）lsyncd

**特别说明：**

下面的inotify配置是建立在rsync服务基础上的配置过程。

![屏幕快照 2017-03-11 下午8.33.20.png-898.3kB](http://static.zybuluo.com/chensiqi/t7fs6i6obz9w2gil66u8bgfv/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-11%20%E4%B8%8B%E5%8D%888.33.20.png)

# inotify安装

默认yum源：
base + extras + updates

扩展的yum源：
epel

- 网易163源
- 阿里云epel源

在安装inotify-tools前请先确认你的linux内核是否达到了2.6.13，并且在编译时开启CONFIG_INOTIFY选项，也可以通过以下命令检测。

## 查看当前系统是否支持inotify

```shell
[root@backup ~]# uname -r
2.6.32-642.el6.x86_64
[root@backup ~]# ls -l /proc/sys/fs/inotify
"""
总用量 0
-rw-r--r-- 1 root root 0 3月  11 05:01 max_queued_events
-rw-r--r-- 1 root root 0 3月  11 05:01 max_user_instances
-rw-r--r-- 1 root root 0 3月  11 05:01 max_user_watches
# 显示这三个文件证明支持
"""
```

**关键参数说明：**

在/proc/sys/fs/inotify目录下有三个文件，对inotify机制有一定的限制
max_user_watches:设置inotifywait或inotifywatch命令可以监视的文件数量（单进程）
max_user_instances:设置每个用户可以运行的inotifywait或inotifywatch命令的进程数。
max_queued_events：设置inotify实例事件（event）队列可容纳的事件数量。



Yum安装inotify-tools

```shell
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
# yum -y install inotify-tools
# rpm -qa inotify-tools
```

> 一共安装了2个工具，即inotifywait和inotifywatch
>  inotifywait：在被监控的文件或目录上等待特定文件系统事件（open，close，delete等）发生，执行后处于阻塞状态，适合shell脚本中使用。
>  inotifywatch：收集被监视的文件系统使用度统计数据，指文件系统事件发生的次数统计。

## inotifywait命令常用参数详解

```shell
[root@backup ~]# inotifywait --help
inotifywait 3.14
Wait for a particular event on a file or set of files.
Usage: inotifywait [ options ] file1 [ file2 ] [ file3 ] [ ... ]
Options:
    -h|--help       Show this help text.
    @<file>         Exclude the specified file在/proc/sys/fs/inotify目录下有三个文件，对inotify机制有一定的限制
max_user_watches:设置inotifywait或inotifywatch命令可以监视的文件数量（单进程）
max_user_instances:设置每个用户可以运行的inotifywait或inotifywatch命令的进程数。
max_queued_events：设置inotify实例事件（event）队列可容纳的事件数量。 from being watched.
    --exclude <pattern>
                    Exclude all events on files matching the
                    extended regular expression <pattern>.
    --excludei <pattern>
                    Like --exclude but case insensitive.
    -m|--monitor    Keep listening for events forever.  Without
                    this option, inotifywait will exit after one
                    event is received.
    -d|--daemon     Same as --monitor, except run in the background
                    logging events to a file specified by --outfile.
                    Implies --syslog.
    -r|--recursive  Watch directories recursively.
    --fromfile <file>
                    Read files to watch from <file> or `-' for stdin.
    -o|--outfile <file>
                    Print events to <file> rather than stdout.
    -s|--syslog     Send errors to syslog rather than stderr.
    -q|--quiet      Print less (only print events).
    -qq             Print nothing (not even events).
    --format <fmt>  Print using a specified printf-like format
                    string; read the man page for more details.
    --timefmt <fmt> strftime-compatible format string for use with
                    %T in --format string.
    -c|--csv        Print events in CSV format.
    -t|--timeout <seconds>
                    When listening for a single event, time out after
                    waiting for an event for <seconds> seconds.
                    If <seconds> is 0, inotifywait will never time out.
    -e|--event <event1> [ -e|--event <event2> ... ]
        Listen for specific event(s).  If omitted, all events are 
        listened for.

Exit status:
    0  -  An event you asked to watch for was received.
    1  -  An event you did not ask to watch for was received
          (usually delete_self or unmount), or some error occurred.
    2  -  The --timeout option was given and no events occurred
          in the specified interval of time.

Events:
    access      file or directory contents were read
    modify      file or directory contents were written
    attrib      file or directory attributes changed
    close_write file or directory closed, after being opened in
                writeable mode
    close_nowrite   file or directory closed, after being opened in
                read-only mode
    close       file or directory closed, regardless of read/write mode
    open        file or directory opened
    moved_to    file or directory moved to watched directory
    moved_from  file or directory moved from watched directory
    move        file or directory moved to or from watched directory
    **create**      file or directory created within watched directory
    **delete**      file or directory deleted within watched directory
    delete_self file or directory was deleted
    unmount     file system containing file or directory unmounted
```

**下面用列表详细解释一下各个参数的含义**

| inotifywait参数 | 含义说明                                           |
| --------------- | -------------------------------------------------- |
| -r --recursive  | 递归查询目录                                       |
| -q --quiet      | 打印很少的信息，仅仅打印监控事件的信息             |
| -m，--monitor   | 始终保持事件监听状态                               |
| --exclude       | 排除文件或目录时，不区分大小写。                   |
| --timefmt       | 指定时间输出的格式                                 |
| --format        | 打印使用指定的输出类似格式字符串                   |
| -e，--event     | 通过此参数可以指定需要监控的事件，如下一个列表所示 |

**-e ：--event的各种事件含义**

| Events   | 含义                                                     |
| -------- | -------------------------------------------------------- |
| access   | 文件或目录被读取                                         |
| modify   | 文件或目录内容被修改                                     |
| attrib   | 文件或目录属性被改变                                     |
| close    | 文件或目录封闭，无论读/写模式                            |
| open     | 文件或目录被打开                                         |
| moved_to | 文件或目录被移动至另外一个目录                           |
| move     | 文件或目录被移动到另一个目录或从另一个目录移动至当前目录 |
| create   | 文件或目录被创建在当前目录                               |
| delete   | 文件或目录被删除                                         |
| umount   | 文件系统被卸载                                           |

## 人工测试监控事件

开启两个窗口

### 测试create

```shell
在第一个窗口输入如下内容：
[root@backup ~]# ls /backup
[root@backup ~]# inotifywait -mrq --timefmt '%y %m %d %H %M' --format '%T %w%f' -e create /backup

在第二个窗口：输入如下内容
[root@backup ~]# cd /backup
[root@backup backup]# touch chensiqi

此时回到第一个窗口出现如下内容：
17 03 11 07 19 /backup/chensiqi

#命令说明
inotifywait：ionotify的命令工具
-mrq：-q只输入简短信息 -r，递归监控整个目录包括子目录 -m进行不间断持续监听
--timefmt 指定输出的时间格式 
--format：指定输出信息的格式
-e create：制定监控的时间类型，监控创建create事件。
```

### 测试delte

```shell
第一个窗口输入如下信息：
[root@backup ~]# inotifywait -mrq --timefmt '%y %m %d %H %M' --format '%T %w%f' -e delete /backup

第二个窗口输入如下信息：
[root@backup backup]# rm -rf chensiqi

此时第一个窗口会出现如下信息：
17 03 11 07 29 /backup/chensiqi

#命令说明：
-e delete：指定监听的事件类型。监听删除delete事件
```

### 测试close_write

```shell
第一个窗口输入如下信息：
inotifywait -mrq --timefmt '%y %m %d %H %M' --format '%T %w%f' -e close_write /backup
第二个窗口输入如下信息：
[root@backup backup]# touch close_write.log
[root@backup backup]# echo 111 >> close_write.log 
[root@backup backup]# rm -f close_write.log 
此时第一个窗口会出现如下信息：
17 03 11 07 38 /backup/close_write.log
17 03 11 07 39 /backup/close_write.log

#命令说明：
-e close_write:指定监听类型。监听文件写模式的关闭。
```

### 测试move_to

```shell
第一个窗口输入如下信息：
[root@backup ~]# inotifywait -mrq --timefmt '%y %m %d %H %M' --format '%T %w%f' -e moved_to /backup  
第二个窗口输入如下信息：

此时第一个窗口会出现如下信息：
[root@backup backup]# touch chensiqi
[root@backup backup]# mv chensiqi chen
[root@backup backup]# mkdir ddddd
[root@backup backup]# mv chen ddddd/
```

### 编写inotify实时监控脚本

```shell
#!/bin/bash

backup_Server=172.16.1.41


/usr/bin/inotifywait -mrq --format '%w%f' -e create,close_write,delete /data | while read line
do
    cd /data
    rsync -az ./ --delete rsync_backup@$backup_Server::nfsbackup --password-file=/etc/rsync.password
done

```

**提示：**

- 上边那个脚本效率很低，效率低的原因在于只要目录出现变化就都会导致我整个目录下所有东西都被推送一遍。因此，我们可以做如下改动提高效率

```shell
#!/bin/bash

Path=/data
backup_Server=172.16.1.41


/usr/bin/inotifywait -mrq --format '%w%f' -e create,close_write,delete /data  | while read line  
do
    if [ -f $line ];then
        rsync -az $line --delete rsync_backup@$backup_Server::nfsbackup --password-file=/etc/rsync.password       
    else
        cd $Path &&\
        rsync -az ./ --delete rsync_backup@$backup_Server::nfsbackup --password-file=/etc/rsync.password
    fi

done

```

**脚本可以加入开机启动：**
 `echo "/bin/sh /server/scripts/inotify.sh &" >> /etc/rc.local`
 提示：
 一个& 代表从后台开始运行该条命令。

### 关键参数调整

> 在/proc/sys/fs/inotify目录下有三个文件，对inotify机制有一定的限制
>  max_user_watches:设置inotifywait或inotifywatch命令可以监视的文件数量（单进程）
>  max_user_instances:设置每个用户可以运行的inotifywait或inotifywatch命令的进程数
>  max_queued_events:设置inotify实例事件（event）队列可容纳的事件数量。

**实战调整：**

```shell
[root@nfs01 data]# cat /proc/sys/fs/inotify/max_
max_queued_events   max_user_instances  max_user_watches
[root@nfs01 data]# cat /proc/sys/fs/inotify/max_user_watches 
8192
[root@nfs01 data]# echo "50000000" > /proc/sys/fs/inotify/max_user_watches
[root@nfs01 data]# cat /proc/sys/fs/inotify/max_user_watches 
50000000
[root@nfs01 data]# cat /proc/sys/fs/inotify/max_queued_events 
16384
[root@nfs01 data]# echo "326790" > /proc/sys/fs/inotify/max_queued_events
[root@nfs01 data]# cat /proc/sys/fs/inotify/max_queued_events 
326790
[root@nfs01 data]# sysctl -p
```

# Rsync+inotify实时数据同步并发简单测试

10K-100K

每秒100个并发

```shell
[root@nfs01 data]# paste inotify_100_server.log
inotify_100_backup_server.log > inotify_100.txt
[root@nfs01 data]# cat inotify_100.txt
23:05       34227   23:05   34227
23:05       34387   23:05   34387
23:05       35027   23:05   35027
23:05       35587   23:05   35587
23:05       36473   23:05   36473
23:05       36707   23:05   36707
23:05       37587   23:05   37587 
以下省略...

```

**Inotify实时并发：**

结论：经过测试，每秒200文件并发，数据同步几乎无延迟（小于1秒）

## inotify 优点：

1）监控文件系统事件变化，通过同步工具实现实时数据同步。

## inotify 缺点

1）并发如果大于200个文件（10-100k），同步就会有延迟
 2）我们前面写的脚本，每次都是全部推送一次，但确实是增量的。也可以只同步变化的文件，不变化的不理。
 3）监控到事件后，调用rsync同步是单进程的，而sersync为多进程同步。既然有了inotify-tools，为什么还要开发sersync？

## serysync功能多：（inotify+rsync命令）

1）支持通过配置文件管理
 2）真正的守护进程socket
 3）可以对失败文件定时重传（定时任务功能）
 4）第三方的HTTP接口（例如：更新cdn缓存）
 5）默认多进程rsync同步

## 高并发数据实时同步方案小结

1）inotify（sersync）+ rsync，是文件级别的。
 2）drbd文件系统级别，文件系统级别，基于block块同步，缺点：备节点数据不可用
 3）第三方软件的同步功能：mysql同步（主从复制），oracle，mongodb
 4）程序双写，直接写两台服务器。
 5）利用产品业务逻辑解决（读写分离，备份读不到，读主）

![屏幕快照 2017-03-13 上午11.35.54.png-266.6kB](http://static.zybuluo.com/chensiqi/hj7bzc9b1xxuc7ytp3wdka7d/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-13%20%E4%B8%8A%E5%8D%8811.35.54.png)
 说明：
 用户上传的图片或者附件单独存在NFS主服务器上；
 用户读取数据从两台NFS备份服务器上读取；
 NFS主和两台NFS备份通过inotify+rsync方式进行实时同步。

6）NFS集群（1，4，5方案整合）（双写主存储，备存储用inotify（sersync）+rsync

![屏幕快照 2017-03-13 下午12.38.28.png-861.7kB](http://static.zybuluo.com/chensiqi/b1ancrsfwv965bi8mkzf3zvf/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-03-13%20%E4%B8%8B%E5%8D%8812.38.28.png)



# 生产环境

实时监控目标目录的修改时间属性，发生变化就执行字符编码转码操作。

脚本如下：

```shell
#!/bin/bash
#
PATH="/data/shared"
/usr/bin/inotifywait -mrq --format '%w%f' -e modify $PATH | while read line
do
    if [ -f $line ];then
        /usr/bin/convmv -f gb2312 -t UTF-8 --notest -r $PATH >> $PATH/modify.log
    fi
done
```

