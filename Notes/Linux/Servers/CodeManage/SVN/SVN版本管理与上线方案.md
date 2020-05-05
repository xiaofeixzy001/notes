[TOC]

## 第1章 SVN服务实战应用指南

### 1.1 SVN介绍

#### 1.1.1 什么是SVN（Subversion）？

> - Svn（subversion）是近年来崛起的非常优秀的版本管理工具，与CVS管理工具一样，SVN是一个跨平台的开源的版本控制系统。Svn版本管理工具管理着随时间改变的各种数据。这些数据放置在一个中央资料档案库（repository）中，这个档案库很像一个普通的文件服务器或者FTP服务器，但是，与其他服务器不同的是，SVN会备份并记录每个文件每一次的修改更新变动。这样我们就可以把任意一个时间点的档案恢复到想要的某一个旧的版本，当然也可以直接浏览指定文件的更新历史记录。
> - 为什么会有svn这样一个项目？
> - 官方解释：为了接管CVS的用户基础，确切的说，我们写了一个新的版本控制系统，它和CVS很相似，但是它修正了以前CVS所没有解决的许多问题。问题见SVN官方首页。
> - SVN是一个非常通用的软件系统，它常被用来管理程序源码，但是它也可以管理任何类型的文件，如文本，视频，图片等等。

**SVN相关站点：**

> Subversion官网：
>  http://subversion.tigris.org/
>  http://subversion.apache.org/
>  svn客户端：http://toroisesvn.net/
>  svn中文网站：http://www.iusesvn.com/
>  中文常见问题解答FAQ：http://subversion.apache.org/faq.zh.html
>  官方手册：http://svnbook.red-bean.com/ 中英都有

### 1.2 svn与git的区别

#### 1.2.1 svn集中式版本控制系统

> svn版本控制系统是集中式的数据管理，存在一个中央版本库，所有开发人员本地开发所使用的代码都是来自于这个版本库，提交代码也都必须提交到这个中央版本库。

**svn版本控制系统工作流程如下：**

1. 在中央库上创建或从主干复制一个分支
2. 从中央库check out 下这个分支的代码
3. 增加自己的代码文件，修改现存的代码或删除代码文件
4. commit代码，假设有人在刚刚的分支上提交了代码，你就会被提示代码过期，你得先up你的代码后再提交。up代码的时候如果出现冲突，需要解决好冲突后再进行提交。

**缺点：**

> 当无法连接到中央版本库的环境下，你无法提交代码，将代码加入版本控制；
>  你无法查看代码的历史版本以及版本的变化过程。提交到版本控制系统中的代码我们都默认通过自测可运行的，如果某个模块的代码比较复杂，不能短时间内实现为可测试的功能，那么你需要等很长的时间才能提交自己的代码，由于代码库集中管理，因此，需要对中央版本库的存储做备份。这点分布式的版本控制系统要好一些。Svn的备份要备份所有代码数据以及所有更改的版本记录。

#### 1.2.2 git分布式的版本控制

> - git是由Linus开发的，所以很自然的git和Linux文件系统结合的比较紧密，以至于在windows上你必须使用cygwin才能使其完美的工作。
> - 那git凭啥叫做分布式的版本控制系统呢？还是从其工作模式讲起把。
> - git中没有了中央版本库的说法了，但是为了开发小组的代码共享，我们通常还是会搭建一个远程的git仓库。
> - 但是和svn不同的是，开发者本地也包含了一个完整的git仓库，从某种程度上说本地的仓库和远程的仓库在身份上是等价的，没有主从之分。
> - 如果你的项目是闭源项目，或者你习惯于以往的集中式的管理模式的话，那么在git下你也可以像svn那样的工作，只是流程中可能会增加一些步骤。

1. 你本地创建一个git库，并将其add到远程git库中。
2. 你在本地添加或者删除文件，然后commit，当然commit操作都是提交到本地的git库中了。（嗯，其实是提交到git目录下的objects目录中去了）
3. 将本地git库的分支push到远程git库的分支，如果这个时候远程git库中已经有别人push过，那么远程git库将不允许你push，这时候你需要先pull，然后如果有冲突，处理好冲突，commit到本地git库后，再push到远程git库。

> 从上面的描述我们可以看到，我们每个开发人员的本地都会有一个git库，我们可以随时进行commit而不需要联网，可以随时查看历史版本，当某一个功能点开发完了之后我们可以将commit后的内容push到远程git库了，如果远程git库的版本在你上次clone或者pull之后变化了，那么你需要进行pull并处理冲突，提交之后，再push到远程git库。

### 1.3 企业应用场景

> svn仍是当前企业的主流。git正在发展，也许未来也会成为主流，但现在还不是。如果同学们有精力，能同时掌握更好。

### 1.4 运维人员掌握版本管理

**对于版本管理系统，运维人员需要掌握的技术点：**

1. 安装，部署，维护，排障。
2. 简单使用，很多公司都是由开发来管理，包括建立新仓库和添加删除账号
3. 对于版本控制系统，运维人员相当于开发商，开发人员是业主，运维搭建的系统为开发人员服务的。

### 1.5 SVN服务运行模式与访问方式

#### 1.5.1 SVN服务端运行方式

**svn服务常见的运行访问方式有3种：**

（1）独立服务器访问

访问地址如：svn：//svn.yunjisuan.org/sadoc;

（2）借助apache等http服务

访问地址如：http://svn.yunjisuan.com/sadoc;

a,单独安装apache+svn（不要用）
 b,CSVN（apache+svn）是一个单独的整合的软件，带web界面管理的SVN软件

（3）本地直接访问（例如：file://application/svndata/sadoc）

![QQ截图20170912203307.png-75.6kB](http://static.zybuluo.com/chensiqi/b9id9wzh7wi4gqni8ykdmxtk/QQ%E6%88%AA%E5%9B%BE20170912203307.png)

> 在这里，主要给同学们介绍第一种方式以及第二种方式中的CSVN web管理方式

#### 1.5.2 SVN客户端访问方式

> SVN客户端可以通过多种方式访问服务器端，例如：本地磁盘访问，或各种各样不同的网络协议访问，但一个版本库地址永远都是一个URL，URL反映了访问方法。

| 访问方式   | 说明                                               |
| ---------- | -------------------------------------------------- |
| file://    | 直接通过本地磁盘或者网络磁盘访问版本库             |
| http://    | 通过WebDAV协议访问支持Subversion的Apache服务器     |
| https://   | 与http://相似，但是用SSL加密访问                   |
| svn://     | 通过TCP/IP自定义协议访问svnserve服务器             |
| svn+ssh:// | 通过认证并加密的TCP/IP自定义协议访问svnserve服务器 |

### 1.6 SVN档案库数据格式

> svn存储版本数据有2种方式：BDB（一种事务安全型表类型）和FSFS（一种不需要数据库的存储系统）。因为BDB方式在服务器中断时，有可能锁住数据，所以还是FSFS方式更安全一点。

- BDB：

> 伯克利DB（Berkeley DB），版本库可以使用的一种经过充分测试的后台数据库实现，不能在通过网络共享的文件系统上使用，伯克利DB是Subversion 1.2版本以前的缺省版本库格式

- FSFS：

> 一个专用于Subversion版本库的文件系统后端，可以使用网络文件系统（例如 NFS 或 SMBFS）。是1.2版本及其后的缺省版本库格式。

### 1.7.1 SVN 集中式版本管理系统

> Svn是一种集中式文件版本管理系统。集中式管理的工作流程如下图：

![QQ截图20170912205406.png-31.5kB](http://static.zybuluo.com/chensiqi/e98wsq190pz0htyzvfjddwv8/QQ%E6%88%AA%E5%9B%BE20170912205406.png)

> 集中式代码管理的核心是SVN服务器，所有开发者在开始新一天的工作之前必须从服务器获取代码，然后进行开发，最后解决冲突，提交。所有的版本信息都放在SVN服务器上。因此如果脱离了服务器，开发者就无法进行提交代码工作。

### 1.7.2 开发者利用SVN版本管理系统工作过程

**下面举例说明：**

开始新一天的工作：

1. 首先从SVN服务器下载项目组最新代码。
2. 进入自己的分支，进行开发工作，每隔一小时向服务器上自己的分支提交一次代码（很多程序员都有这个习惯。因为有时候自己对代码改来改去，最后又想还原到新一个小时的版本，或者看看前一个小时自己修改了哪些代码，就需要这样做了）。
3. 下班时间快到了，把自己的分支合并到服务器主分支上，一天的工作完成，并反映给服务器。

**优点：**

1. 管理方便，逻辑清晰明确，符合一般人思维习惯。
2. 易于管理，集中式svn服务器更能保证数据安全性。
3. 代码一致性非常高。
4. 适合开发人数不多的项目开发。
5. 普及度高，大部分软件配置管理的大学教材都是使用svn和vss。

## 第2章 搭建SVN服务端

### 2.1 安装配置SVN服务

```
#检查环境
 [root@localhost ~]# cat /etc/redhat-release 
CentOS release 6.5 (Final)
[root@localhost ~]# uname -m
x86_64
[root@localhost ~]# uname -r
2.6.32-431.el6.x86_64

#光盘安装svn
[root@localhost ~]# yum -y install subversion
[root@localhost ~]# rpm -qa subversion
subversion-1.6.11-9.el6_4.x86_64

#建立svn版本库数据存储根目录（svndata）及用户，密码权限目录（svnpasswd）
mkdir -p /application/svndata   #数据存储根目录
mkdir -p /application/svnpasswd #用户，密码权限目录
```

### 2.2 建立项目版本库

> 创建一个新的Subversion项目yunjisuan，其实，类似yunjisuan这样的项目可以创建多个，每个项目对应不同的代码，这里只是以创建一个项目为例演示：

```
[root@localhost ~]# svnadmin create /application/svndata/yunjisuan
[root@localhost ~]# tree /application/svndata/yunjisuan/
/application/svndata/yunjisuan/
|-- README.txt
|-- conf
|   |-- authz
|   |-- passwd
|   `-- svnserve.conf
|-- db
|   |-- current
|   |-- format
|   |-- fs-type
|   |-- fsfs.conf
|   |-- min-unpacked-rev
|   |-- rep-cache.db
|   |-- revprops
|   |   `-- 0
|   |       `-- 0
|   |-- revs
|   |   `-- 0
|   |       `-- 0
|   |-- transactions
|   |-- txn-current
|   |-- txn-current-lock
|   |-- txn-protorevs
|   |-- uuid
|   `-- write-lock
|-- format
|-- hooks
|   |-- post-commit.tmpl
|   |-- post-lock.tmpl
|   |-- post-revprop-change.tmpl
|   |-- post-unlock.tmpl
|   |-- pre-commit.tmpl
|   |-- pre-lock.tmpl
|   |-- pre-revprop-change.tmpl
|   |-- pre-unlock.tmpl
|   `-- start-commit.tmpl
`-- locks
    |-- db-logs.lock
    `-- db.lock

10 directories, 28 files

#提示：查看svnadmin命令帮助的方法
```

### 2.3 编辑svn配置文件

```
[root@localhost ~]# cd /application/svndata/yunjisuan/conf/
[root@localhost conf]# ll
total 12
-rw-r--r--. 1 root root 1080 Sep 13 16:02 authz
-rw-r--r--. 1 root root  309 Sep 13 16:02 passwd
-rw-r--r--. 1 root root 2279 Sep 13 16:02 svnserve.conf
[root@localhost conf]# cp svnserve.conf{,.bak}
[root@localhost conf]# vim svnserve.conf
#修改配置文件的如下信息
[root@localhost ~]# cat -n /application/svndata/yunjisuan/conf/svnserve.conf.bak | sed -n '12p;13p;20p;27p'
    12  # anon-access = read
    13  # auth-access = write
    20  # password-db = passwd
    27  # authz-db = authz

#将配置文件代码修改为如下所示：
[root@localhost ~]# cat -n /application/svndata/yunjisuan/conf/svnserve.conf | sed -n '12p;13p;20p;27p'
    12  anon-access = none          #禁止匿名访问
    13  auth-access = write         #验证访问可写
    20  password-db = /application/svnpasswd/passwd #密码文件位置
    27  authz-db = /application/svnpasswd/authz     #验证文件位置
    
特别提示：
此配置文件里的每条配置代码必须顶格写，不能有空格。
```

### 2.4 将authz文件和passwd文件拷贝到/application/svnpasswd下

```
[root@localhost conf]# cp /application/svndata/yunjisuan/conf/authz /application/svnpasswd/
[root@localhost conf]# cp /application/svndata/yunjisuan/conf/passwd /application/svnpasswd/
[root@localhost conf]# ll /application/svnpasswd/
total 8
-rw-r--r--. 1 root root 1080 Sep 13 17:07 authz
-rw-r--r--. 1 root root  309 Sep 13 17:07 passwd
```

### 2.5 启动svn服务

```
[root@localhost conf]# svnserve --help          #svn启动命令帮助
svnserve: warning: cannot set LC_CTYPE locale
svnserve: warning: environment variable LANG is en
svnserve: warning: please check that your locale name is correct
usage: svnserve [-d | -i | -t | -X] [options]

Valid options:
  -d [--daemon]            : daemon mode        #守护进程启动（后台）
  -i [--inetd]             : inetd mode
  -t [--tunnel]            : tunnel mode
  -X [--listen-once]       : listen-once mode (useful for debugging)
  -r [--root] ARG          : root of directory to serve #指定根目录
  -R [--read-only]         : force read only, overriding repository config file
  --config-file ARG        : read configuration from file ARG
  --listen-port ARG        : listen port        #监听端口默认3690
                             [mode: daemon, listen-once]
  --listen-host ARG        : listen hostname or IP address  #监听IP
                             [mode: daemon, listen-once]
  -T [--threads]           : use threads instead of fork [mode: daemon]
  --foreground             : run in foreground (useful for debugging)
                             [mode: daemon]
  --log-file ARG           : svnserve log file
  --pid-file ARG           : write server process ID to file ARG
                             [mode: daemon, listen-once]
  --tunnel-user ARG        : tunnel username (default is current uids name)
                             [mode: tunnel]
  -h [--help]              : display this help
  --version                : show program version information

#启动svn服务
[root@localhost conf]# svnserve -d -r /application/svndata/
svnserve: warning: cannot set LC_CTYPE locale   #警告可以忽略
svnserve: warning: environment variable LANG is en #警告可以忽略
svnserve: warning: please check that your locale name is correct    ##警告可以忽略
[root@localhost conf]# netstat -antup | grep 3690
tcp        0      0 0.0.0.0:3690                0.0.0.0:*                   LISTEN      1256/svnserve    
```

### 2.6 解决svnserve启动时的警告问题

```
[root@localhost conf]# source /etc/sysconfig/i18n   #启用中文字符集
[root@localhost conf]# pkill svnserve
[root@localhost conf]# svnserve -d -r /application/svndata/
[root@localhost conf]# netstat -antup | grep 3690
tcp        0      0 0.0.0.0:3690                0.0.0.0:*                   LISTEN      1261/svnserve 
[root@localhost conf]# cat /etc/sysconfig/i18n 
LANG="en_US.UTF-8"
SYSFONT="latarcyrheb-sun16"
```

### 2.7 passwd文件及密码设置

```
#在/application/svnpasswd/passwd文件末尾追加如下内容：
[root@localhost conf]# tail -4 /application/svnpasswd/passwd 
yunjisuan = 123123      #设置账号密码
benet     = 123123      #设置账号密码
stu001    = 123         #设置账号密码
stu002    = 456         #设置账号密码
```

### 2.8 authz的授权

> **注意：**
>  1，权限配置文件中出现的用户名必须已在用户配置文件中定义
>  2，对权限配置文件的修改立即生效，不必重启svn

```
权限配置说明
#用户组格式：
【groups】
=，
其中，1个用户组可以包含1个或多个用过户，用户间以逗号分隔。
例如：harry\_and\_sally = harry,sally     #==>用户组 = 用户1，用户2

#版本库目录格式：
[<版本库>：/项目/目录]  #例如：[repository:/baz/fuz]
@<用户组名> = <权限>    #例如：@harry\_and\_sally = rw
<用户名> = <权限>       #例如：harry = rw
#其中，方框号内部分可以有多种写法：
[/],表示根目录及以下，根目录是svnserve启动时指定的，我们指定为/application/svndata，[/]就是表示对全部版本库设置权限。
[repos:/]，表示对版本库repos设置权限。
[repos:/yunjisuan]，表示对版本库repos中的yunjisuan项目设置权限。
[repos:/yunjisuan/benet]，表示对版本库repos中的yunjisuan项目的benet目录设置权限。
#权限主体可以是用户组，用户或*，用户组在前面加@，*表示全部用户。
#权限可以是w，r，wr和空，空表示没有任何权限。
#authz中每个参数都要顶格写，开头不能有空格。
#对于组，要以@开头，用户不需要@开头。
#编辑authz配置文件进行授权，在authz末尾加入以下几句代码
[root@localhost conf]# egrep -v "#|^$" /application/svnpasswd/authz
[aliases]
[groups]
sagroup = stu001,stu002         #新增本行，定义组名
[yunjisuan:/]                   #定义授权的范围
yunjisuan = rw                  #用户单独授权
benet = r                       #用户单独授权
@sagroup = r                    #组用户授权
```

### 2.9 重启动svnserve

```
[root@localhost conf]# ps -ef | grep svn | grep -v grep
root       1261      1  0 17:16 ?        00:00:00 svnserve -d -r /application/svndata/
[root@localhost conf]# kill 1261
[root@localhost conf]# ps -ef | grep svn | grep -v grep
[root@localhost conf]# svnserve -d -r /application/svndata/
```

## 第3章 搭建SVN客户端

### 3.1 使用svn客户端（windows版）

#### 3.1.1 软件版本选择

推荐：TortoiseSVN-1.9.7.27907-x64-svn-1.9.7

> 注意：32位系统要用32位软件版本

#### 3.1.2 svn客户端软件安装

> 一路yes即可

#### 3.1.3 svn客户端软件的使用

（1）先在本地创建一个目录，起名任意，比如data

![QQ截图20170914001000.png-15.4kB](http://static.zybuluo.com/chensiqi/rzvzn5sk1yjk0d3zyrnuffk3/QQ%E6%88%AA%E5%9B%BE20170914001000.png)

（2）鼠标右键点击data目录

选择右键菜单里的SVN Checkout，出现下图：

![QQ截图20170914001615.png-21.6kB](http://static.zybuluo.com/chensiqi/whgfzqjhzr3cjfxrkokuwblf/QQ%E6%88%AA%E5%9B%BE20170914001615.png)

> 特别提示：
>  如果连接不通，请检查Linux虚拟机的iptables是否关闭。

点击OK后，出现下图：

![QQ截图20170914001854.png-29.6kB](http://static.zybuluo.com/chensiqi/q3ilb6tx1ahucx1icjzhgdin/QQ%E6%88%AA%E5%9B%BE20170914001854.png)

再次点击OK以后，结束。此时目录里多了一个隐藏的目录，表示此目录已经和svn服务器连通

![QQ截图20170914002236.png-9.7kB](http://static.zybuluo.com/chensiqi/6kxxec64camqe322w6i0vxyq/QQ%E6%88%AA%E5%9B%BE20170914002236.png)

> **命令说明：**
>  （1）SVN Checkout:相当于下载，第一次连接svn服务器的时候需要和服务器的对应存储目录进行数据同步，如果服务器的对应目录里有数据文件，那么就会下载到你的本地对应目录里。
>  （2）SVN Update：更新数据，检查服务器端svn存储目录里是否和本地svn存储目录数据不一致，如果不一致，那么下载改变或新增的部分到本地svn目录里。（不会删除本地目录内容）
>  （3）SVN Commit：提交数据到svn服务器端存储目录。本地svn存储目录会和服务器端存储目录进行比对校验。会把本地改变的部分和新增的部分同步上传至服务器端。

#### 3.1.4 svn客户端使用测试

（1）向windows的svn存储目录data里放一个空文件

![QQ截图20170914004001.png-9.4kB](http://static.zybuluo.com/chensiqi/2v1gjm6xzyqqd87tb7zecx81/QQ%E6%88%AA%E5%9B%BE20170914004001.png)

（2）右键点击data目录，选择SVN Commit

![QQ截图20170914004135.png-33.4kB](http://static.zybuluo.com/chensiqi/7h539c17fk6ddo264xx7td4b/QQ%E6%88%AA%E5%9B%BE20170914004135.png)

![QQ截图20170914004208.png-28.9kB](http://static.zybuluo.com/chensiqi/9cov4zzno1gu3w8x2sn7se38/QQ%E6%88%AA%E5%9B%BE20170914004208.png)

（3）打开本地data目录里的文件，随便写点内容后，再次进行SVN commit

![QQ截图20170914004748.png-38.9kB](http://static.zybuluo.com/chensiqi/ajcrt3i2wypjqf7u3pcwf02j/QQ%E6%88%AA%E5%9B%BE20170914004748.png)

![QQ截图20170914004759.png-24.5kB](http://static.zybuluo.com/chensiqi/tw1i5fpvxcon6ok9z4ikjwfj/QQ%E6%88%AA%E5%9B%BE20170914004759.png)

（4）直接从本地查看服务器端的数据内容

右键点击本地svn存储目录data，选择TortoiseSVN ===>Repo-browser后出现下图：

![QQ截图20170914005217.png-47.3kB](http://static.zybuluo.com/chensiqi/rfznghia32ngxifdepil9hby/QQ%E6%88%AA%E5%9B%BE20170914005217.png)

双击文件可以直接远程打开文件，可以看到里面刚刚被修改后的内容已经更新至服务器端。

（5）删除本地svn存储目录data里的文件，后选择SVN Update

> 同学们会发现，刚刚删除的文件又重新下载回来了。

（6）继续删除本地svn存储目录data里的文件，后选择SVN Commit

![QQ截图20170914005649.png-43.7kB](http://static.zybuluo.com/chensiqi/1a9rygepm2dgytphtujxpvbo/QQ%E6%88%AA%E5%9B%BE20170914005649.png)

![QQ截图20170914005657.png-22.5kB](http://static.zybuluo.com/chensiqi/woxdrqvrnyf1c1t61d5em2ub/QQ%E6%88%AA%E5%9B%BE20170914005657.png)

（7）再次查看服务器端存储目录里，发现文件已经被删除了

![QQ截图20170914005806.png-34.7kB](http://static.zybuluo.com/chensiqi/fusfq75d3o9zpj2qmn1qcobg/QQ%E6%88%AA%E5%9B%BE20170914005806.png)

### 3.2 SVN的管理命令（Linux）

```
[root@localhost ~]# svn --help
usage: svn <subcommand> [options] [args]
Subversion command-line client, version 1.6.11.
Type 'svn help <subcommand>' for help on a specific subcommand.
Type 'svn --version' to see the program version and RA modules
  or 'svn --version --quiet' to see just the version number.

Most subcommands take file and/or directory arguments, recursing
on the directories.  If no arguments are supplied to such a
command, it recurses on the current directory (inclusive) by default.

Available subcommands:
   add
   blame (praise, annotate, ann)
   cat
   changelist (cl)
   checkout (co)        #下载数据
   cleanup
   commit (ci)          #提交数据
   copy (cp)
   delete (del, remove, rm)
   diff (di)
   export
   help (?, h)
   import
   info
   list (ls)            #显示服务器端内容
   lock
   log
   merge
   mergeinfo
   mkdir
   move (mv, rename, ren)
   propdel (pdel, pd)
   propedit (pedit, pe)
   propget (pget, pg)
   proplist (plist, pl)
   propset (pset, ps)
   resolve
   resolved
   revert
   status (stat, st)
   switch (sw)
   unlock
   update (up)                  #更新数据

Subversion is a tool for version control.
For additional information, see http://subversion.tigris.org/
```

#### 3.2.1 从SVN库提取数据

> 将文件checkout到本地目录
>  svn checkout（co） remotepath localpath

```
[root@localhost ~]# mkdir yunjisuan
[root@localhost ~]# cd yunjisuan/
[root@localhost yunjisuan]# pwd
/root/yunjisuan

#下载服务器端数据到Linux本地目录
[root@localhost yunjisuan]# svn co svn://192.168.0.220/yunjisuan/ /root/yunjisuan/ --username=benet --password=123123
Restored '/root/yunjisuan/ffff.txt'
Checked out revision 6.
[root@localhost yunjisuan]# ll
total 4
-rw-r--r--. 1 root root 30 Sep 13 19:45 ffff.txt
```

#### 3.2.2 查看SVN版本库中的数据

> svn list file:///application/svndata/yunjisuan

```
[root@localhost yunjisuan]# svn list file:///application/svndata/yunjisuan/
ffff.txt
```

#### 3.2.3 提交数据到SVN版本库

(1)一次失败的提交

```
[root@localhost yunjisuan]# pwd
/root/yunjisuan
[root@localhost yunjisuan]# mkdir {111..120}    #创建目录
[root@localhost yunjisuan]# ll
total 44
drwxr-xr-x. 2 root root 4096 Sep 13 19:56 111
drwxr-xr-x. 2 root root 4096 Sep 13 19:56 112
drwxr-xr-x. 2 root root 4096 Sep 13 19:56 113
drwxr-xr-x. 2 root root 4096 Sep 13 19:56 114
drwxr-xr-x. 2 root root 4096 Sep 13 19:56 115
drwxr-xr-x. 2 root root 4096 Sep 13 19:56 116
drwxr-xr-x. 2 root root 4096 Sep 13 19:56 117
drwxr-xr-x. 2 root root 4096 Sep 13 19:56 118
drwxr-xr-x. 2 root root 4096 Sep 13 19:56 119
drwxr-xr-x. 2 root root 4096 Sep 13 19:56 120
-rw-r--r--. 1 root root   30 Sep 13 19:45 ffff.txt
-rw-r--r--. 1 root root    0 Sep 13 19:48 xxxx
[root@localhost yunjisuan]# svn add *       #提交前需要先把要提交的内容做标记A
A         111
A         112
A         113
A         114
A         115
A         116
A         117
A         118
A         119
A         120
svn: warning: 'ffff.txt' is already under version control       #这个文件已经标记过了
A         xxxx
[root@localhost yunjisuan]# svn ci -m "message"         #提交时需要同时-m指定一段话作为备注
Authentication realm: <svn://192.168.0.220:3690> e498878c-b578-41a4-8361-6c1c5baa75f9
Password for 'benet':           #输入benet用户密码（之前checkout时的账号密码已经记录到了此目录）

-----------------------------------------------------------------------
ATTENTION!  Your password for authentication realm:

   <svn://192.168.0.220:3690> e498878c-b578-41a4-8361-6c1c5baa75f9

can only be stored to disk unencrypted!  You are advised to configure
your system so that Subversion can store passwords encrypted, if
possible.  See the documentation for details.

You can avoid future appearances of this warning by setting the value
of the 'store-plaintext-passwords' option to either 'yes' or 'no' in
'/root/.subversion/servers'.
-----------------------------------------------------------------------
Store password unencrypted (yes/no)? yes        #是否记录账户
svn: Commit failed (details follow):
svn: Authorization failed                       #提交失败，账户没有写权限，认证失败
```

（2）换账户重新Checkout

```
[root@localhost yunjisuan]# svn co svn://192.168.0.220/yunjisuan/ /root/yunjisuan/ --username=yunjisuan --password=123123      #换拥有写入权限的账户checkout

-----------------------------------------------------------------------
ATTENTION!  Your password for authentication realm:

   <svn://192.168.0.220:3690> e498878c-b578-41a4-8361-6c1c5baa75f9

can only be stored to disk unencrypted!  You are advised to configure
your system so that Subversion can store passwords encrypted, if
possible.  See the documentation for details.

You can avoid future appearances of this warning by setting the value
of the 'store-plaintext-passwords' option to either 'yes' or 'no' in
'/root/.subversion/servers'.
-----------------------------------------------------------------------
Store password unencrypted (yes/no)? yes            #是否作为目录的新账户和密码
Checked out revision 6.
[root@localhost yunjisuan]# ll
total 44
drwxr-xr-x. 3 root root 4096 Sep 13 19:57 111
drwxr-xr-x. 3 root root 4096 Sep 13 19:57 112
drwxr-xr-x. 3 root root 4096 Sep 13 19:57 113
drwxr-xr-x. 3 root root 4096 Sep 13 19:57 114
drwxr-xr-x. 3 root root 4096 Sep 13 19:57 115
drwxr-xr-x. 3 root root 4096 Sep 13 19:57 116
drwxr-xr-x. 3 root root 4096 Sep 13 19:57 117
drwxr-xr-x. 3 root root 4096 Sep 13 19:57 118
drwxr-xr-x. 3 root root 4096 Sep 13 19:57 119
drwxr-xr-x. 3 root root 4096 Sep 13 19:57 120
-rw-r--r--. 1 root root   30 Sep 13 19:45 ffff.txt
-rw-r--r--. 1 root root    0 Sep 13 19:48 xxxx


#重新提交
[root@localhost yunjisuan]# svn add *
svn: warning: '111' is already under version control        #文件或目录已经纳入了版本控制（也就是做过标记了）
svn: warning: '112' is already under version control
svn: warning: '113' is already under version control
svn: warning: '114' is already under version control
svn: warning: '115' is already under version control
svn: warning: '116' is already under version control
svn: warning: '117' is already under version control
svn: warning: '118' is already under version control
svn: warning: '119' is already under version control
svn: warning: '120' is already under version control
svn: warning: 'ffff.txt' is already under version control
svn: warning: 'xxxx' is already under version control
[root@localhost yunjisuan]# svn ci -m "message"         #重新提交
Adding         111
Adding         112
Adding         113
Adding         114
Adding         115
Adding         116
Adding         117
Adding         118
Adding         119
Adding         120
Adding         xxxx
Transmitting file data .
Committed revision 7.

#查看服务器端数据
[root@localhost yunjisuan]# svn list file:///application/svndata/yunjisuan/
111/
112/
113/
114/
115/
116/
117/
118/
119/
120/
ffff.txt
xxxx
```

## 第4章 SVN钩子脚本

### 4.1 钩子脚本简介

> - 钩子脚本的具体写法就是操作系统中shell脚本程序的写法，可根据自己的SVN所在的操作系统和shell程序进行相应的开发。
> - 钩子脚本就是被某些版本库事件触发的程序，例如：创建新版本或修改未被版本控制的属性。每个钩子都能掌管足够的信息来了解发生了什么事件，操作对象是什么以及触发事件用户的账号。
> - 根据钩子的输出或返回状态，钩子程序能够以某种方式控制该动作继续执行，停止或挂起。

**默认情况下，钩子的子目录中包含各种版本库钩子模板**

```
[root@localhost ~]# ls -l /application/svndata/yunjisuan/hooks/
total 36
-rw-r--r--. 1 root root 1977 Sep 13 16:02 post-commit.tmpl
-rw-r--r--. 1 root root 1638 Sep 13 16:02 post-lock.tmpl
-rw-r--r--. 1 root root 2289 Sep 13 16:02 post-revprop-change.tmpl
-rw-r--r--. 1 root root 1567 Sep 13 16:02 post-unlock.tmpl
-rw-r--r--. 1 root root 3426 Sep 13 16:02 pre-commit.tmpl
-rw-r--r--. 1 root root 2410 Sep 13 16:02 pre-lock.tmpl
-rw-r--r--. 1 root root 2786 Sep 13 16:02 pre-revprop-change.tmpl
-rw-r--r--. 1 root root 2100 Sep 13 16:02 pre-unlock.tmpl
-rw-r--r--. 1 root root 2780 Sep 13 16:02 start-commit.tmpl
```

> - 对每种Subversion版本库支持的钩子都有一个模板，通过查看这些脚本的内容，你能看到是什么事件触发了脚本及如何给传脚本传递数据。
> - 同时，这些模板也是如何使用这些脚本，结合Subversion支持的工具来完成有用任务的例子。
> - 要实际安装一个可用的钩子，你需要在repos/hooks目录下安装一些与钩子同名（如start-commit或者post-commit）的可执行程序或脚本，注意，去掉模板的扩展名。

**重要提示：**

> 由于安全原因，Subversion版本库在一个空环境中执行钩子脚本就是没有任何环境变量，甚至没有$PATH或%PATH%。由于这个原因，许多管理员会感到很困惑，他们的钩子脚本手工运行时正常，可在Subversion中却不能运行。要注意，必须在你的钩子中设置好环境变量或为你的程序指定好绝对路径。

### 4.2 SVN的hooks模板

#### 4.2.1 常用钩子脚本

| 钩子脚本     | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| post-commit  | 在提交完成成功创建版本之后执行该钩子，提交已经完成，不可更改，因此，本脚本的返回值被忽略。提交完成时触发事务 |
| pre-commit   | 提交完成前触发执行该脚本                                     |
| start-commit | 在客户端还没有向服务器提交数据之前，即还没有建立Subversion transaction之前，执行该脚本（提交前出发事务） |

#### 4.2.2 非常用钩子脚本

1. pre-revprop-change:在修改revision属性之前，执行该脚本
2. post-revprop-change:在修改revision属性之后，执行该脚本。因为修改稿已经完成，不可更改，因此本脚本的返回值被忽略（不过实际上的实现似乎是该脚本的正确执行与否影响属性修改）
3. pre-unlock:对文件进行解锁操作之前执行该脚本
4. post-unlock:对文件进行解锁操作之后执行该脚本
5. pre-lock:对文件进行加锁操作之前执行该脚本
6. post-lock：对文件进行加锁操作之后执行该脚本。

#### 4.2.3 利用钩子脚本触发同步数据的注意事项

（1）一定要定义变量，主要是用过的命令的路径。因为SVN的考虑的安全问题，没有调用系统变量，如果手动执行是没有问题，但SVN自动执行就会无法执行了。

（2）SVN的同步目录在 update之前一定要先checkout一份出来，还有这里一定要添加用户和密码。

（3）加上了对前一个命令的判断，如果update的时候出了问题，程序没有退出的话还会继续同步代码到Web服务器上，这样会造成代码有问题。

（4）建议最好记录日志，出错的时候可以很快的排错

（5）最后是数据同步，rsync的相关参数一定要清楚。

### 4.3 svn钩子生产应用场景举例

- pre-commit:

> 限制上传文件扩展名及大小，控制提交要输入的信息等。

- post-commit:

> SVN更新自动周知，MSN，邮件或短信周知。
>  SVN更新触发checkout程序，然后实时rsync推送到服务器等。

### 4.4 svn钩子生产应用实战

#### 4.4.1 rsync与svn钩子结合实现数据实时同步某企业小案例

（1）建立同步WEB目录

mkdir -p /data/www

（2）将SVN中内容checkout到WEB目录一份。

```
[root@localhost yunjisuan]# mkdir -p /data/www
[root@localhost yunjisuan]# svn checkout svn://192.168.0.220/yunjisuan /data/www --username=yunjisuan --password=123123
A    /data/www/xxxx
A    /data/www/111
A    /data/www/120
A    /data/www/112
A    /data/www/113
A    /data/www/114
A    /data/www/ffff.txt
A    /data/www/115
A    /data/www/116
A    /data/www/117
A    /data/www/118
A    /data/www/119
Checked out revision 7.
[root@localhost yunjisuan]# ll /data/www/
total 44
drwxr-xr-x. 3 root root 4096 Sep 13 21:32 111
drwxr-xr-x. 3 root root 4096 Sep 13 21:32 112
drwxr-xr-x. 3 root root 4096 Sep 13 21:32 113
drwxr-xr-x. 3 root root 4096 Sep 13 21:32 114
drwxr-xr-x. 3 root root 4096 Sep 13 21:32 115
drwxr-xr-x. 3 root root 4096 Sep 13 21:32 116
drwxr-xr-x. 3 root root 4096 Sep 13 21:32 117
drwxr-xr-x. 3 root root 4096 Sep 13 21:32 118
drwxr-xr-x. 3 root root 4096 Sep 13 21:32 119
drwxr-xr-x. 3 root root 4096 Sep 13 21:32 120
-rw-r--r--. 1 root root   30 Sep 13 21:32 ffff.txt
-rw-r--r--. 1 root root    0 Sep 13 21:32 xxxx
```

（3）制作钩子脚本，post-commit

```
root@localhost yunjisuan]# cd /application/svndata/yunjisuan/hooks/
[root@localhost hooks]# cp post-commit.tmpl post-commit #复制模板一份
[root@localhost hooks]# egrep -v "#|^$" post-commit     #模板原始内容
REPOS="$1"
REV="$2"
mailer.py commit "$REPOS" "$REV" /path/to/mailer.conf
[root@localhost hooks]# vim post-commit         #修改post-commit脚本
[root@localhost hooks]# egrep -v "#|^$" post-commit
REPOS="$1"                  #传参（未用上）
REV="$2"                    #传参（未用上）
SvnIP="192.168.0.220"       #svn服务端的IP地址
ProjectName="yunjisuan"     #svn服务端的项目库名称
UserName="yunjisuan"        #账户姓名
PassWord="123123"           #账户密码
LocalPath="/data/www"       #位于svn本地的共享目录
SVN=/usr/bin/svn            #svn命令的绝对路径
export LC_CTYPE="en_US.UTF-8"   #中文字符集支持
export LC_ALL=
if [ ! -d ${LocalPath} ];then   
    mkdir -p ${LocalPaht}
    $SVN checkout svn://${SvnIP}/${ProjectName} ${LocalPath} --username=${UserName} --password=${PassWord}       #新创建目录需要先经过checkout才能update
else
    $SVN update --username yunjisuan --password 123123 /data/www        #更新共享目录内容
fi
if [ $? -eq 0 ];then
    /usr/bin/rsync -az --delete /data/www /tmp/         #数据同步推送到本地/tmp目录下（生产环境可以直接同步推送到Web测试服务器）
fi
```

（4）进行钩子脚本同步测试

```
#删除之前的测试记录
[root@localhost hooks]# rm -rf /data/www/
[root@localhost hooks]# ll -d /data/www
ls: cannot access /data/www: No such file or directory
[root@localhost hooks]# rm -rf /tmp/*
[root@localhost hooks]# ll /tmp/
total 0
[root@localhost hooks]# chmod 700 post-commit   #给钩子脚本可执行权限
```

> **特别提示：**
>  当用户通过svn更新钩子post-commit所在的项目库时，在更新完毕之后会自动触发钩子脚本

**模拟更新项目库版本**

![QQ截图20170915014410.png-37.6kB](http://static.zybuluo.com/chensiqi/zoqu2jbrdlsve3vasj8hn4g6/QQ%E6%88%AA%E5%9B%BE20170915014410.png)

![QQ截图20170915014425.png-31.1kB](http://static.zybuluo.com/chensiqi/mgskjv721t2hmhalf8vr3jsl/QQ%E6%88%AA%E5%9B%BE20170915014425.png)

```
#查看svn服务器端钩子脚本执行情况
[root@localhost hooks]# ll /data/www/                   #svn服务器端本地共享目录
total 28
drwxr-xr-x. 3 root root 4096 Sep 13 23:07 111
drwxr-xr-x. 3 root root 4096 Sep 13 23:07 112
drwxr-xr-x. 3 root root 4096 Sep 13 23:07 113
drwxr-xr-x. 3 root root 4096 Sep 13 23:07 116
drwxr-xr-x. 3 root root 4096 Sep 13 23:07 117
drwxr-xr-x. 3 root root 4096 Sep 13 23:07 118
-rw-r--r--. 1 root root    0 Sep 13 23:07 test.txt
-rw-r--r--. 1 root root    9 Sep 13 23:07 xxx.txt
-rw-r--r--. 1 root root    0 Sep 13 23:07 xxxx
[root@localhost hooks]# ll /tmp/www/                    #推送后的数据目录
total 28
drwxr-xr-x. 3 root root 4096 Sep 13 23:07 111
drwxr-xr-x. 3 root root 4096 Sep 13 23:07 112
drwxr-xr-x. 3 root root 4096 Sep 13 23:07 113
drwxr-xr-x. 3 root root 4096 Sep 13 23:07 116
drwxr-xr-x. 3 root root 4096 Sep 13 23:07 117
drwxr-xr-x. 3 root root 4096 Sep 13 23:07 118
-rw-r--r--. 1 root root    0 Sep 13 23:07 test.txt
-rw-r--r--. 1 root root    9 Sep 13 23:07 xxx.txt
-rw-r--r--. 1 root root    0 Sep 13 23:07 xxxx
```

> 综上，post-commit钩子脚本测试成功。

#### 4.4.2 通过pre-commit的钩子脚本还可以对用户上传的内容进行大小和扩展名的限制

> 因为并不常用，这部分内容，我们略过

## 第5章 大中小型企业上线解决方案

### 5.1 SVN 上线解决方案说明

#### 5.1.1 小型公司代码上线案例（十几台服务器）

![QQ截图20170915023258.png-137.6kB](http://static.zybuluo.com/chensiqi/bguhkryi3rt812qccw10rkom/QQ%E6%88%AA%E5%9B%BE20170915023258.png)

> 开发每次修改完代码就直接提交，然后通过FTP直接更新到Web服务器网页目录；没有专门的测试人员，完全是由用户来进行测试体验。

**小型企业现状：**

> 小型公司一般只有几个开发人员，网站核心程序大多数都是PHP语言开发，为了方便，会直接通过FTP直接上传程序代码到线上服务器，随时随地上线更新。

**上述上线方案的特点和问题：**

1. 发布快，及时，随时随地就可以发布代码。
2. 开发人员发布的代码不经过测试人员的测试，且用户访问页面刷新后页面即改变，也可能刷新瞬间程序在更新，到时无法访问，对网站用户的体验比较差，如果开发写错了代码，造成的影响就更大了，这是拿用户作为测试的上线方案。
3. 据统计，网站中大概50%以上的故障是和开发程序代码有关的，（比如：开发写错了一个循环代码，导致了死循环，此时大量用户访问这个程序，就能把服务器资源耗尽，搞死服务器）
4. 在中小公司网站出了问题一般是运维人员的问题（例如网站宕机），但这种情况下，问题大多可能由开发人员或代码引起的，这里比较好的策略是开发项目负责制思想。

**小型企业上线架构方案建议：**

1. 开发人员需在个人电脑搭建LAMP环境测试开发好的网站代码，并且在办公室或IDC机房的测试环境测试通过，最好有专职测试人员。
2. 程序代码上线规定时间，例如，三天上线一次，如网站需经常更新可每天下午17点上线，这个看网站业务性质而定，原则就是影响用户体验最小。
3. 代码上线之前需备份，网站程序出了问题方便回退，另外，网站程序出了问题方便回退，另外，从上线技巧上讲，上传代码时尽可能先传到服务器网站临时目录，传完整后一步mv过去，或者通过ln做软连接。（线上更新代码思路）
4. 务必由运维人员管理上线，对于代码的功能性，开发人员更在意，而对于代码的性能和服务的稳定，运维更在意，因此，如果网站问题归运维管，就要让运维上线这样更规范科学。否则，开发随意更新，出了问题运维负责，这样就错了。

#### 5.1.2 中型企业上线解决方案

> 中型企业上线，一般是规范运维人员操作步骤，制定统一的上线操作脚本，备份文件名称，备份文件路径。使操作人性化，统一化，自动化。

**Web代码的上线流程演示图：**

![QQ截图20170915101748.png-28.3kB](http://static.zybuluo.com/chensiqi/vsnsrd1xh5hcbfsfpb8oyyaw/QQ%E6%88%AA%E5%9B%BE20170915101748.png)

#### 5.1.3 大型企业上线解决方案

> 大型企业上线一般制度和流程控制较多，比较严谨，下面是某大型企业上线解决方案架构：

![QQ截图20170915110524.png-74kB](http://static.zybuluo.com/chensiqi/mrw26qtcbbuyew9vblo860hv/QQ%E6%88%AA%E5%9B%BE20170915110524.png)

> **SVN里的内容：**
>  1，程序代码
>  2，服务的配置
>  3，项目文档，设计文档，运维部署优化文档

**门户大型网站架构环境代码上线具体方案：**

1. 本地开发人员从SVN中取代码。当天上线的提交到trunk，否则，长期项目单开分支开发，然后在合并主线（trunk）
2. 办公内网开发测试时，由开发人员或配置管理员通过部署平台jenkins实现统一部署，（即在部署平台上控制开发机器从SVN取代码，编译，打包，发布到开发机器，包名如idc_dep.war）
3. 开发人员通知或和测试人员一起测试程序，没有问题后，打上新的tag标记。
4. 配置管理员，根据上步的tag标记，checkout出上线代码，并配置好IDC测试环境的所有配置，执行编译，打包（mvn，ant）（php不需要），然后发布到IDC内的统一分发服务器，这里要注意，不同环境的配置文件是随代码同时发布的。
5. 配置管理员或SA上线人员，把分发的程序代码内容推送到相关测试服务器（包名如idc_test.war）,然后通知开发及测试人员进行测试。如果有问题向上回退，继续修改。
6. 如果测试没有问题，继续打好tag标记，此时，配置管理员，根据上步的tag标记，checkout出测试好的代码，并配置好IDC正式环境的所有配置，执行编译，打包（mvn，ant）（php不需要），然后发布到IDC内的统一分发服务器主机，准备批量发布。
7. 配置管理员或SA上线人员，把分发的内容推送到相关正式服务器（包名如idc_product.war），然后通知开发及测试人员进行测试。如果有问题直接发布回滚指令。

> IDC正式上线的过程对于JAVA程序，可以是AB分组上线的思路，即平滑下线一半的服务器，然后发布更新代码测试，无问题后，挂上服务器，同时在平滑下线另一半的服务器，然后发布更新代码测试（或者直接发布后就挂上线）

**PHP程序代码上线的具体方案：**

> 对于PHP上线方法：发布代码时（也需要测试流程）可以直接发布到正式线临时目录，然后mv或更改link的方式发布到正式线目录，不需要重启http服务。这是sina，ganji的上线方案。

**JAVA程序代码上线的具体方案：**

> 对于java上线方法：较大公司需要分组平滑上线，例如，首先从负载均衡器上摘掉一半的服务器，发布代码后，重启服务器测试，没问题后，挂上经过测试的这一半，再下另外一半。如果前端有DNS智能解析，上线还可以分地区上线若干服务器，逐渐普及到全国的服务器，这个被称为灰度发布。

#### 5.1.4 更多大型代码上线解决方案案例

（1）SINA网的代码发布流程逻辑图：

![QQ截图20170915221011.png-52.2kB](http://static.zybuluo.com/chensiqi/39p3qpwx8onzdnxzpnhluq1u/QQ%E6%88%AA%E5%9B%BE20170915221011.png)

（2）和讯案例

```
ABCD 12:33:24
我们这里代码发布都不太标准，全部都是开发自己搞

Mr.chen 12：35：14
目前是什么个方式呢
说下现状即可。

ABCD 12:36:04
就是很传统，开发有权限可以上机器，我们就把应用部署好，他们随便折腾。

ABCD 12:41:05
源代码是svn，静态内容都是同步分发
```

（3）小米案例

```
XYZ 13:36:49
代码上线都是开发上，我们运维这边没有流程...如果代码发布导致了问题，就是开发的问题。

XYZ 13:37:55
服务器上面有一个客户端，开发自己在页面上点发布，客户端就去拉代码了。
就是这么个额流程，就像你以前说的，项目责任制，谁的项目出问题了。找开发和运维

Mr.chen 13:49:08
不需要重启服务器么？还有直接拉到站点目录么？

XYZ 13:49:17
嗯，都是自动的
他们有个管理系统

Mr.chen 13:49:49
如何保证不影响用户呢？
还有怎么回滚的。

XYZ 13:50:12
还没有做到这点把
那个管理系统可以回滚的，好像
平时把客户的部署上去，再把机器加入到那个系统中
他们就可以发了。

XYZ 13:58:16
运维这边就管添加机器和安装客户端，也有发布权限，项目上线后很少发。一教就会没有在这块搞过太多，那个程序和版本管理结合的。实现原理应该就是客户端收到服务器发来的clone命令和路径，就去执行了。
```

**什么是配置管理员呢？**

> 就是在开发和运维中间起一个连接纽带的一个职位，这个职位一般在大公司里会设置，负责SVN的管理，上线管理，申请，协调等工作。

### 5.2 自动化部署和上线代码管理

> 对于门户网站或重视规范或开发能力较强的公司也许会结合系统服务和WEB界面管理来更科学更自动的进行上线代码管理，如开发一个自动化代码上线部署平台，其实就是一个web管理界面（界面底层调用相关脚本实现分发推送代码以及重启服务器），然后普通的初级上线人员就可以在平台里实现仅仅点鼠标，敲回车，就能实现平滑上线和平滑回滚代码了，当然，自动化和完善的程度也许没我们说的这么好，但是，思路是这样的。下面就是管理平台的一个图例：

![QQ截图20170915224132.png-90.3kB](http://static.zybuluo.com/chensiqi/6dmot0ywbqqto8ri4f8yrhar/QQ%E6%88%AA%E5%9B%BE20170915224132.png)

**开发自动化部署平台的思路很多，例如：我们可以通过nagios的被动模式实现上线管理平台原理思路：**

> 实际上就是生成配置在分发服务器上执行命令请求，应用服务器，然后脚本在应用服务器处理完毕后回传结果到web界面显示：

例如：check_nrpe -h 10.0.0.178 -c check_load

### 5.3 开发人员和运维人员业务变更管理平台

**业务变更管理平台优点：**

1. 变更管理制度流程有利于业务稳定。

2. 保留变更业务历史，便于核查发现的问题。

3. 故障跟踪平台，有利于跟踪问题的解决进度，而不是半途而废。

4. 相关常用软件（

   同学们自己有时间最好研究一下

   ）

   - JIRA 用于缺陷跟踪，客户服务，需求收集，流程审批，任务跟踪，项目跟踪和敏捷管理等工作领域。
   - Mantis是一款PHP开源Bug跟踪系统，比较适合中小型项目的管理及跟踪，具有多特性。包括：易于安装，易于操作，基于Web，支持任何可运行PHP的平台（Windows，Linux，Mac，Solaris，AS400/i5等），已经被翻译成68种语言，支持多个项目，为每一个项目设置不同的用户访问级别，跟踪缺陷变更历史，定制我的视图页面，提供全文搜索功能，内置报表生成功能（包括图形报表），通过Email报告缺陷，用户可以监视特殊的Bug，附件可以保存在web服务器上或数据库中（还可以备份到FTP服务器上），自定义缺陷处理工作流，支持输出格式包括csv，MicrosoftExcel，MicrosoftWord，集成源代码控制（SVN与CVS），集成wiki知识库与聊天工具（可选/可不选），支持多种数据库（MySQL，MSSQL，PostgreSQ，Oracle，DB2），提供WebService（SOAP）接口，提供Wap访问。