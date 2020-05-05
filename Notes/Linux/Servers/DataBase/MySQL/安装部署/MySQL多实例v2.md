[TOC]

## 一，MySQL多实例介绍

在之前LNMP的讲解中，已经针对MySQL数据库进行了介绍，并说明了为什么要选择MySQL数据库，以及MySQL数据库在Linux系统下的多种安装方式，同时讲解了MySQL的二进制方式单实例安装，基础优化等内容，本节将为同学们讲解更为实用的MySQL多实例安装，主从复制集群等重要应用实践。

### 1.1 什么是MySQL多实例

- 简单的说，MySQL多实例就是在一台服务器上同时开启多个不同的服务器端口（如：3306，3307），同时运行多个MySQL服务进程，这些服务进程通过不同的socket监听不同的服务器端口来提供服务。
- 这些MySQL多实例共用一套MySQL安装程序，使用不同的my.cnf（也可以相同）配置文件，启动程序（也可以相同）和数据文件。在提供服务时，多实例MySQL在逻辑上看起来是各自独立的，它们根据配置文件的对应设定值，获得服务器相应数量的硬件资源。
- 打个比方吧，MySQL多实例就相当于房子的多个卧室，每个实例可以看作一间卧室，整个服务器就是一套房子，服务器的硬件资源（CPU，Mem，Disk），软件资源（Centos操作系统）可以看作房子的卫生间，厨房，客厅，是房子的公用资源。
- 其实很多网络服务都是可以配置多实例的，例如Nginx，Apache，Haproxy，Redis，Memcache等。这在门户网站使用得很广泛。

### 1.2 MySQL多实例的作用与问题

**MySQL多实例的作用如下：**

（1）有效利用服务器资源

> 当单个服务器资源有剩余时，可以充分利用剩余的资源提供更多的服务，且可以实现资源的逻辑隔离。

（2）节约服务器资源

> 当公司资金紧张，但是数据库又需要各自尽量独立地提供服务，而且，需要主从复制等技术时，多实例就再好不过了。

![屏幕快照 2017-07-16 下午7.32.58.png-783.4kB](http://static.zybuluo.com/chensiqi/7oyn08it5n0b47gxqan07q2o/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-07-16%20%E4%B8%8B%E5%8D%887.32.58.png)

MySQL多实例有它的好处，但也有其弊端，比如，会存在资源互相抢占的问题。

当某个数据库实例并发很高或有SQL慢查询时，整个实例会消耗大量的系统CPU，磁盘I/O等资源，导致服务器上的其他数据库实例提供服务的质量一起下降。这就相当于大家住在一个房子的不同卧室一样，早晨起来上班，都要刷牙，洗脸等，这样卫生间就会长期占用，其他人要等待一样。不同实例获取的资源是相对独立的，无法像虚拟化一样完全隔离。

## 二， MySQL多实例的生产应用场景

### 2.1 资金紧张型公司的选择

> 若公司资金紧张，公司业务访问量不太大，但又希望不同业务的数据库服务各自尽量独立地提供服务而互相不受影响，同时，还需要主从复制等技术提供备份或读写分离服务，那么多实例就再好不过了。例如：可以通过3台服务器部署9～15个实例，交叉做主从复制，数据备份及读写分离，这样就可达到9～15台服务器每个只装一个数据库才有的效果。这里要强调的是，所谓的尽量独立是相对的。

### 2.2 并发访问不是特别大的业务

> 当公司业务访问量不太大的时候，服务器的资源基本上都浪费了，这时就很适合多实例的应用，如果对SQL语句的优化做得比较好，MySQL多实例会是一个很值得使用的技术，即使并发很大，合理分配好系统资源，搭配好服务，也不会有太大问题。

### 2.3 门户网站应用MySQL多实例场景

> 门户网站通常都会使用多实例，因为配置硬件好的服务器，可节省IDC机柜空间，同时，跑多实例也会减少硬件资源跑不满的浪费。比如，百度公司的很多数据库都是多实例，不过，一般是从库多实例，例如某部门中使用的IBM服务器为48核CPU，内存96GB，一台服务器跑3～4个实例；此外，新浪网使用的也是多实例，内存48GB左右。

**说明：**

据调查，新浪网的数据库单机1～4个数据库实例的居多，其中又数1～2个的最多，因为大业务占用的机器比较多。服务器是DELL  R510的居多，CPU是E5210，48GB内存，磁盘12×300G  SAS，做RAID10，此为门户网站的服务器配置参考，希望能给同学们的面试带来一些启迪。

另外，新浪网站安装数据库时，一般采用编译安装的方式，并且会在优化之后做成rpm包，以便统一使用。

## 三， MySQL多实例常见的配置方案

### 3.1 单一配置文件，单一启动程序的多实例部署方案

> 下面是MySQL官方文档提到的单一配置文件，单一启动程序多实例部署方案，但不推荐此方案，这里仅作为知识点提及，后文不再涉及此方案的说明。my.cnf配置文件示例（MySQL手册里提到的方法）如下：

```
[mysqld_multi]
mysqld      =   /usr/bin/mysqld_safe
mysqladmin  =   /usr/bin/mysqladmin
user        =   mysql
[mysqld1]
socket      =   /var/lib/mysql/mysql.sock
port        =   3306
pid-file    =   /var/lib/mysql/mysql.pid
datadir     =   /var/lib/mysql/
user        =   mysql
[mysqld2]
socket      =   /mnt/data/db1/mysql.sock
port        =   3302
pid-file    =   /mnt/data/db1/mysql.pid
datadir     =   /mnt/data/db1/
user        =   mysql
skip-name-resolv
server-id=10
default-storage-engine=innodb
innodb_buffer_pool_size=512M
innodb_additional_mem_pool=10M
default_character_set=utf8
character_set_server=utf8
#read-only
relay-log-space-limit=3G
expire_logs_day=20
```

**启动程序的命令如下：**

```
mysqld_multi --config-file=/data/mysql/my_multi.cnf start 1,2
```

> 该方案的缺点是耦合度太高，一个配置文件，不好管理。工作开发和运维的统一原则为降低耦合度。

### 3.2 多配置文件，多启动程序的部署方案

> 多配置文件，多启动程序部署方案，是本文主要讲解的方案，也是非常常用并极力推荐的多实例方案。下面来看配置示例。

```
[root@localhost /]# tree /data
/data
├── 3306
│   ├── data        #3306实例的数据目录
│   ├── my.cnf      #3306实例的配置文件
│   └── mysql       #3306实例的启动文件
└── 3307
    ├── data        #3307实例的数据目录
    ├── my.cnf      #3307实例的配置文件
    └── mysql       #3307实例的启动文件

4 directories, 4 files
```

> **提示：**
> 这里的配置文件my.cnf,启动程序mysql都是独立的文件，数据文件data目录也是独立的。
> 多实例MySQL数据库的安装和之前讲解的单实例没有任何区别，因此，同学们如果有前文单实例的安装环境，那么可以直接略过5.1节的内容。

## 四，安装并配置多实例MySQL数据库

### 4.1 安装MySQL多实例

1，安装MySQL需要的依赖包和编译软件

（1）安装MySQL需要的依赖包

安装MySQL之前，最好先安装MySQL需要的依赖包，不然后面会出现很多报错信息，到那时还得再回来安装MySQL的依赖包。安装命令如下：

```
[root@localhost ~]# yum -y install ncurses-devel libaio-devel
[root@localhost ~]# rpm -qa ncurses-devel libaio-devel
ncurses-devel-5.7-4.20090207.el6.x86_64
libaio-devel-0.3.107-10.el6.x86_64
```

（2）安装编译MySQL需要的软件

> 首先通过网络获得cmake软件，然后进行如下操作：

```
[root@localhost ~]# ls -lh cmake-2.8.6.tar.gz 
-rw-r--r-- 1 root root 5.4M 7月  19 20:43 cmake-2.8.6.tar.gz          #此软件需提前准备
[root@localhost ~]# tar xf cmake-2.8.6.tar.gz -C /usr/src/
[root@localhost ~]# cd /usr/src/cmake-2.8.6/
[root@localhost cmake-2.8.6]# ./configure
[root@localhost cmake-2.8.6]# gmake && gmake install
[root@localhost cmake-2.8.6]# which cmake
```

2，开始安装MySQL

> 为了让同学们学习更多的MySQL技术，接下来会以相对复杂的源代码安装来讲解MySQL多实例的安装。大型公司一般都会将MySQL软件定制成rpm包，然后放到yum仓库里，使用yum安装，中小企业里的二进制和编译安装的区别不大。

（1）建立MySQL用户账号

> 首先以root身份登录到Linux系统中，然后执行如下命令创建mysql用户账号：

```
[root@localhost ~]# useradd -s /sbin/nologin -M mysql
[root@localhost ~]# id mysql
uid=500(mysql) gid=500(mysql) 组=500(mysql)
```

（2）获取MySQL软件包

> MySQL软件包的下载地址为：https://dev.mysql.com/downloads/mysql/
> 提示：
> 本例以MySQL编译的方式来讲解，之前已经演示过二进制方式安装了。在生产场景中，二进制和源码包两种安装方法都是可以用的，其应用场景一般没什么差别。不同之处在于，二进制的安装包较大，名字和源码包也有些区别，二进制安装过程比源码更快。

MySQL源码包和二进制安装包的名称见下图

![QQ截图20170719211317.png-31.8kB](http://static.zybuluo.com/chensiqi/01ob7ctns4ae9xo6eif77huz/QQ%E6%88%AA%E5%9B%BE20170719211317.png)

（3）采用编译方式安装MySQL

配置及编译安装的步骤如下：

```
[root@localhost ~]# tar xf mysql-5.5.22.tar.gz -C /usr/src/
[root@localhost ~]# cd /usr/src/mysql-5.5.22/
[root@localhost mysql-5.5.22]# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql-5.5.22 \
> -DMYSQL_DATADIR=/usr/local/mysql-5.5.22/data \        #数据存放目录
> -DMYSQL_UNIX_ADDR=/usr/local/mysql-5.5.22/tmp/mysql.sock \    #MySQL进程间通信的套接字位置
> -DDEFAULT_CHARSET=utf8 \                  #默认字符集为utf8
> -DDEFAULT_COLLATION=utf8_general_ci \     #默认字符集排序规则
> -DEXTRA_CHARSETS=gbk,gb2312,utf8,ascii \  #额外的字符集支持
> -DENABLED_LOCAL_INFILE=ON \               #是否启用加载本地数据
> -DWITH_INNOBASE_STORAGE_ENGINE=1 \        #静态编译innodb存储引擎到数据库
> -DWITH_FEDERATED_STORAGE_ENGINE=1 \       #静态编译FEDERATED存储引擎到数据库
> -DWITH_BLACKHOLE_STORAGE_ENGINE=1 \       #静态编译blackhole存储引擎到数据库
> -DWITHOUT_EXAMPLE_STORAGE_ENGINE=1 \      #不编译EXAMPLE存储引擎到数据库
> -DWITHOUT_PARTITION_STORAGE_ENGINE=1 \    #不支持数据库分区
> -DWITH_FAST_MUTEXES=1 \
> -DWITH_ZLIB=bundled \                     #zlib压缩模式
> -DENABLED_LOCAL_INFILE=1 \                #是否启用本地的LOCAL_INFILE
> -DWITH_READLINE=1 \                       #使用捆绑的readline
> -DWITH_EMBEDDED_SERVER=1 \                #是否要建立嵌入式服务器
> -DWITH_DEBUG=0                            #禁用DEBUG（开启影响性能）
# 提示：编译时可配置的选项很多，具体可参考官方文档
[root@localhost mysql-5.5.22]# make && make install
```

**下面设置不带版本号的软链接/usr/local/mysql,操作步骤如下：**

```
[root@localhost mysql-5.5.22]# ln -s /usr/local/mysql-5.5.22 /usr/local/mysql
[root@localhost mysql-5.5.22]# ls /usr/local/mysql
bin      data  include         lib  mysql-test  scripts  sql-bench
COPYING  docs  INSTALL-BINARY  man  README      share    support-files
```

> 如果上述操作未出现错误，查看/usr/local/mysql目录下有内容，则MySQL5.5.22源代码包采用cmake方式的安装就算成功了。

### 4.2 创建MySQL多实例的数据文件目录

> 在企业中，通常以/data目录作为MySQL多实例总的根目录，然后规划不同的数字（即MySQL实例端口号）作为/data下面的二级目录，不同的二级目录对应的数字就作为MySQL实例的端口号，以区别不同的实例，数字对应的二级目录下包含MySQL的数据文件，配置文件及启动文件等。
> 下面以配置3306,3307两个实例为例进行讲解。创建MySQL多实例的目录如下：

```
[root@localhost ~]# mkdir -p /data/{3306,3307}/data
[root@localhost ~]# tree /data/
/data/
├── 3306            #3306实例目录
│   └── data        #3306实例的数据文件目录
├── 3307            #3307实例目录
    └── data        #3307实例的数据文件目录


4 directories, 0 files
```

> **提示：**
> （1）mkdir -p /data/{3306,3307}/data相当于mkdir -p /data/3306/data;mkdir -p /data/3307/data两条命令
> （2）如果是创建多个目录，可以增加如3308,3309这样的目录名，在生产环境中，一般为3~4个实例为佳。

### 4.3 创建MySQL多实例的配置文件

MySQL数据库默认为用户提供了多个配置文件模板，用户可以根据服务器硬件配置的大小来选择。

```
[root@localhost mysql]# ls -l support-files/my*.cnf
-rw-r--r-- 1 root root  4751 7月  19 21:33 support-files/my-huge.cnf
-rw-r--r-- 1 root root 19805 7月  19 21:33 support-files/my-innodb-heavy-4G.cnf
-rw-r--r-- 1 root root  4725 7月  19 21:33 support-files/my-large.cnf
-rw-r--r-- 1 root root  4736 7月  19 21:33 support-files/my-medium.cnf
-rw-r--r-- 1 root root  2900 7月  19 21:33 support-files/my-small.cnf
```

> **注意：**
> 这些配置文件里的注释非常详细，不过是英文的。。。

上面是单实例的默认配置文件模板，如果配置多实例，和单实例会有不同。为了让MySQL多实例之间彼此独立，要为每一个实例建立一个my.cnf配置文件和一个启动文件MySQL，让他们分别对应自己的数据文件目录data。
 首先，通过vim命令添加配置文件内容，命令如下：

```
vim /data/3306/my.cnf
vim /data/3307/my.cnf
```

不同的实例需要添加的my.cnf内容会有区别,其中的配置由官方的配置模板修改而来。当然，在实际工作中，我们是拿早已配置好的模板来进行修改的，可以通过rz等方式上传配置文件模板my.cnf文件到相关目录下。

**MySQL3306,3307实例配置文件如下**

```
##实例3306配置文件my.cnf
[root@localhost ~]# cat /data/3306/my.cnf
[client]
port        = 3306
socket      = /data/3306/mysql.sock
[mysqld]
user        = mysql
port        = 3306
socket      = /data/3306/mysql.sock
basedir     = /usr/local/mysql
datadir     = /data/3306/data
open_files_limit    = 1024
back_log = 600
max_connections = 800
max_connect_errors = 3000
table_open_cache = 614
external-locking = FALSE
max_allowed_packet = 8M
#binlog_cache_size = 1M
#max_heap_table_size = 64M
#read_buffer_size = 2M
#read_rnd_buffer_size = 16M
sort_buffer_size = 1M
join_buffer_size = 1M
thread_cache_size = 100
thread_concurrency = 2
query_cache_size = 2M
query_cache_limit = 1M
query_cache_min_res_unit = 2k
#ft_min_word_len = 4
#default-storage-engine = MYISAM
thread_stack = 192K
transaction_isolation = READ-COMMITTED
tmp_table_size = 2M
max_heap_table_size = 2M
#log-bin=mysql-bin
#binlog_format=mixed
#slow_query_log
long_query_time = 1
pid-file = /data/3306/mysql.pid
relay-log = /data/3306/relay-bin
relay-log-info-file = /data/3306/relay-log.info
binlog_cache_size = 1M
max_binlog_cache_size = 1M
max_binlog_size = 2M
key_buffer_size = 16M
read_buffer_size = 1M
read_rnd_buffer_size = 1M
bulk_insert_buffer_size = 1M
lower_case_table_names = 1
skip-name-resolve
slave-skip-errors = 1032,1062
replicate-ignore-db = mysql

server-id = 1

#key_buffer_size = 32M
#bulk_insert_buffer_size = 64M
#myisam_sort_buffer_size = 128M
#myisam_max_sort_file_size = 10G
#myisam_repair_threads = 1
#myisam_recover
innodb_additional_mem_pool_size = 4M
innodb_buffer_pool_size = 32M
innodb_data_file_path = ibdata1:128M:autoextend
innodb_file_io_threads = 4
#innodb_write_io_threads = 8
#innodb_read_io_threads = 8
innodb_thread_concurrency = 8
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 2M
innodb_log_file_size = 4M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout = 120
innodb_file_per_table = 0

[mysqldump]
quick
max_allowed_packet = 2M

[mysql]
no-auto-rehash

#[myisamchk]
#key_buffer_size = 512M
#sort_buffer_size = 512M
#read_buffer = 8M
#write_buffer = 8M
#[mysqlhotcopy]
#interactive-timeout

[mysqld_safe]
log-error = /data/3306/mysql_yunjisuan3306.err
pid-file = /data/3306/mysqld.pid
```

**提示：**实例3307的配置文件只需要将3306配置文件里的所有3306数字替换成3307(server-id换个数字)即可。

**最终完成后的多实例根/data目录结果如下：**

```
[root@localhost ~]# tree /data
/data
├── 3306
│   ├── data
│   └── my.cnf      #这个就是3306实例的配置文件
└── 3307
    ├── data
    └── my.cnf      #这个就是3307实例的配置文件

4 directories, 2 files
```

### 4.4 创建MySQL多实例的启动文件

MySQL多实例启动文件的创建和配置文件的创建几乎一样，也可以通过vim命令来添加，如下：

```
vim /data/3306/mysql
vim /data/3307/mysql
```

**需要添加的MySQL启动文件内容如下。（当然，在实际工作中我们是拿早已配置好的模板来进行修改的，可以通过rz等方式上传配置文件模板MySQL文件到相关目录下）**

```
[root@localhost ~]# cat /data/3306/mysql
#!/bin/bash
###############################################
#this scripts is created by Mr.chen at 2016-06-25

port=3306
mysql_user="root"
mysql_pwd=""            #这里需要修改为用户的实际密码

CmdPath="/usr/local/mysql/bin"
mysql_sock="/data/${port}/mysql.sock"

#startup function
function_start_mysql(){
    
    if [ ! -e "$mysql_sock" ];then
        printf "Starting MySQL....\n"
        /bin/sh ${CmdPath}/mysqld_safe --defaults-file=/data/${port}/my.cnf 2>&1 >/dev/null &
    else
        printf "MySQL is running...\n"
        exit
    fi
}

#stop function
function_stop_mysql(){

    if [ ! -e "$mysql_sock" ];then
        printf "MySQL is stopped...\n"
        exit
    else
        printf "Stoping MySQL...\n"
        ${CmdPath}/mysqladmin -u ${mysql_user} -p${mysql_pwd} -S /data/${port}/mysql.sock shutdown
    fi
}

#restart function
function_restart_mysql(){

    printf "Restarting MySQL...\n"
    function_stop_mysql
    sleep 2
    function_start_mysql
}

case $1 in 
start)
    function_start_mysql
;;
stop)
    function_stop_mysql
;;
restart)
    function_restart_mysql
;;
*)
    printf "Usage: /data/${port}/mysql{start|stop|restart}\n"
esac
```

**3307实例的启动文件只需修改3306启动文件的端口即可**

**最终完成后的多实例根/data目录结果如下：**

```
[root@localhost ~]# tree /data
/data
├── 3306
│   ├── data
│   ├── my.cnf          #3306实例的配置文件
│   └── mysql           #3306实例的启动文件
└── 3307
    ├── data
    ├── my.cnf          #3307实例的配置文件
    └── mysql           #3307实例的启动文件

4 directories, 4 files
```

> 需要特别说明一下，在多实例启动文件中，启动MySQL不同实例服务，所执行的命令实质是有区别的，例如，启动3306实例的命令如下：
> `mysqld_safe --defaults-file=/data/3306/my.cnf 2>&1 >/dev/null &`
> 启动3307实例的命令如下：
> `mysqld_safe --defaults-file=/data/3307/my.cnf 2>&1 >/dev/null &`
> 下面看看在多实例启动文件中，停止MySQL不同实例服务的实质命令。
> 停止3306实例的命令如下：
> `mysqladmin -uroot -pyunjisuan123 -S /data/3306/mysql.sock shutdown`
> 停止3307实例的命令如下：
> `mysqladmin -u root -pyunjisuan123 -S /data/3307/mysql.sock shutdown`

### 4.5 配置MySQL多实例的文件权限

1）通过下面的命令，授权mysql用户和组管理整个多实例的根目录/data

```
[root@localhost ~]# chown -R mysql.mysql /data
[root@localhost ~]# find /data -name "mysql" | xargs ls -l
-rw-r--r--. 1 mysql mysql 1039 Jul 20 19:33 /data/3306/mysql
-rw-r--r--. 1 mysql mysql 1039 Jul 20 19:34 /data/3307/mysql
```

2）通过下面的命令，授权MySQL多实例所有启动文件的mysql可执行，设置700权限最佳，注意不要用755权限，因为启动文件里有数据库管理员密码，会被读取到。

```
[root@localhost ~]# find /data -name "mysql" | xargs chmod 700
[root@localhost ~]# find /data -name "mysql" | xargs ls -l
-rwx------. 1 mysql mysql 1039 Jul 20 19:33 /data/3306/mysql
-rwx------. 1 mysql mysql 1039 Jul 20 19:34 /data/3307/mysql
```

### 4.6 MySQL相关命令加入全局路径的配置

（1）配置全局路径的意义

> 如果不为MySQL的命令配置全局路径，就无法直接在命令行输入mysql这样的命令，只能用全路径命令（/usr/local/mysql/bin/mysql）,这种带着路径输入命令的方式很麻烦。

（2）配置MySQL全局路径的方法

1）确认mysql命令所在路径，命令如下：

```
[root@localhost ~]# ls /usr/local/mysql/bin/mysql
/usr/local/mysql/bin/mysql
```

2）在PATH变量前面增加/usr/local/mysql/bin路径，并追加到/etc/profile文件中，命令如下：

```
[root@localhost ~]# echo 'export PATH=/usr/local/mysql/bin:$PATH' >> /etc/profile
#注意，echo后边是单引号，双引号的话变量内容会被解析掉。
[root@localhost ~]# tail -1 /etc/profile
export PATH=/usr/local/mysql/bin:$PATH
[root@localhost ~]# source /etc/profile
#执行source使上一行添加到/etc/profile中，内容直接生效
#以上命令的用途为定义mysql全局路径，实现在任意路径执行mysql命令
```

> **提示：**
> 更简单的设置方法为用下面命令做软链接：ln -s /usr/local/mysql/bin/* /usr/local/sbin/,把mysql命令说在路径链接到全局路径/usr/local/sbin/的下面。

### 4.7 初始化MySQL多实例的数据库文件

> 上述步骤全都配置完毕后，就可以初始化数据库文件了，这个步骤其实也可以在编译安装MySQL之后就操作，只不过放到这里更合理一些。

（1）初始化MySQL数据库
 初始化命令如下：

```
[root@localhost scripts]# ./mysql_install_db --basedir=/usr/local/mysql --datadir=/data/3306/data --user=mysql
[root@localhost scripts]# ./mysql_install_db --basedir=/usr/local/mysql --datadir=/data/3307/data --user=mysql
```

> **提示：**
> --basedir=/usr/local/mysql为MySQL的安装路径，--datadir为不同的实例数据目录

（2）初始化数据库的原理及结果说明

> 初始化数据库的实质就是创建基础的数据库系统的库文件，例如：生成MySQL库表等。
> 初始化数据库后查看对应实例的数据目录，可以看到多了如下文件：

```
[root@localhost scripts]# tree /data

#以下省略若干...
```

### 4.8 启动MySQL多实例的命令

```
[root@localhost scripts]# /data/3306/mysql start
Starting MySQL....
[root@localhost scripts]# /data/3307/mysql start
Starting MySQL....
root@localhost scripts]# netstat -antup | grep 330
tcp        0      0 0.0.0.0:3307                0.0.0.0:*                   LISTEN      24743/mysqld        
tcp        0      0 0.0.0.0:3306                0.0.0.0:*                   LISTEN      24020/mysqld   
```

**从输出中可以看到，3306和3307实例均已正常启动。**

## 五，配置及管理MySQL多实例数据库

### 5.1 配置MySQL多实例数据库开机自启动

服务的开机自启动很关键，MySQL多实例的启动也不例外，把MySQL多实例的启动命令加入/etc/rc.local,实现开机自启动，命令如下：

```
[root@localhost ~]# echo "#mysql multi instances" >> /etc/rc.local 
[root@localhost ~]# echo "/data/3306/mysql start" >> /etc/rc.local
[root@localhost ~]# echo "/data/3307/mysql start" >> /etc/rc.local 
[root@localhost ~]# tail -3 /etc/rc.local 
#mysql multi instances
/data/3306/mysql start
/data/3307/mysql start

#这里一定要确保MySQL脚本可执行~
```

### 5.2 登陆MySQL测试

```
[root@localhost ~]# mysql -S /data/3306/mysql.sock  #直接敲就进来了，而且身份还是root。但是多了-S /data/3306/mysql.sock,用于区别登陆不同的实例
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.5.22 Source distribution

Copyright (c) 2000, 2011, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;              #查看当前所有的数据库
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.00 sec)
mysql> select user();               #查看当前的登陆用户
+----------------+
| user()         |
+----------------+
| root@localhost |
+----------------+
1 row in set (0.00 sec)
```

### 5.3 MySQL多实例数据库的管理方法

- MySQL安装完成后，默认情况下，MySQl管理员的账号root是无密码的。登陆不同的实例需要指定不同实例的mysql.sock文件路径，这个mysql.sock是在my.cnf配置文件里指定的。
- 下面是无密码情况下登陆数据库的方法，关键点是-S参数及后面指定的/data/33306/mysql.sock,注意，不同实例的sock虽然名字相同，但是路径是不同的，因此是不同的文件。

```
mysql -S /data/3306/mysql.sock
mysql -S /sata/3307/mysql.sock
```

**下面是重启对应实例数据库的命令**

```
/data/3306/mysql stop
/data/3307/mysql start
```

### 5.4 MySQL安全配置

**MySQL管理员的账号root密码默认为空，极不安全，可以通过mysqladmin命令为MySQL不同实例的数据库设置独立的密码，命令如下：**

```
[root@localhost ~]# mysqladmin -u root -S /data/3306/mysql.sock password '123123'     #为mysql设置密码
[root@localhost ~]# mysql -uroot -p -S /data/3306/mysql.sock       #无法直接登陆了
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.5.22 Source distribution

Copyright (c) 2000, 2011, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

#提示：3307实例设置方法和3306实例相同，只是连接时的mysql，sock路径不同，这已经提醒多次，大家要注意
```

**带密码登陆不同实例数据库的方法：**

```
#登陆3306实例的命令如下：
mysql -uroot -p123123 -S /data/3306/mysql.sock
#登陆3307实例的命令如下：
mysql -uroot -p123123 -S /data/3307/mysql.sock
```

> **提示：**
> 基础弱的同学，在测试时尽量保证多实例的密码相同，可以减少麻烦，后面还原数据库时会覆盖密码！
> 若要重启多实例数据库，也需要进行相应的如下配置。再重启数据库前，需要调整不同实例启动文件里对应的数据库密码。

```
[root@localhost ~]# vim /data/3306/mysql
[root@localhost ~]# sed -n '7p' /data/3306/mysql  #这是之前mysql多实例启动脚本里mysql的登陆密码变量
mysql_pwd=""
[root@localhost ~]# sed -i '7 s#""#"123123"#' /data/3306/mysql
[root@localhost ~]# sed -n '7p' /data/3306/mysql
mysql_pwd="123123"          #修改成实际的登录密码
[root@localhost ~]# sed -n '7p' /data/3307/mysql
mysql_pwd=""
[root@localhost ~]# sed -i '7 s#""#"123123"#' /data/3307/mysql
[root@localhost ~]# sed -n '7p' /data/3307/mysql
mysql_pwd="123123"
```

**多实例下正常停止数据库的命令如下：**

```
/data/3306/mysql stop
```

> 由于选择了mysqladmin shutdown的停止方式，所以停止数据库时需要在启动文件里配置数据库的密码

```
/data/3306/mysql start
```

> **重点提示：**
> 禁止使用pkill，kill -9，killall -9等命令强制杀死数据库，这会引起数据库无法启动等故障的发生。

### 5.5 如何再增加一个MySQL的实例

> 若再3306和3307实例的基础上，再增加一个MySQL实例，该怎么办？下面给出增加一个MySQL3308端口实例的命令集合：

```
mkdir -p /data/3308/data
\cp /data/3306/my.cnf /data/3308/
\cp /data/3306/mysql /data/3308/
sed -i 's#3306#3308#g' /data/3308/my.cnf
sed -i 's#server-id = 1#server-id = 8#g' /data/3308/my.cnf
sed -i 's#3306#3308#g' /data/3308/mysql
chown -R mysql:mysql /data/3308
chmod 700 /data/3308/mysql
cd /usr/local/mysql/scripts
./mysql_install_db --datadir=/data/3308/data --basedir=/usr/local/mysql --user=mysql
chown -R mysql:mysql /data/3308
egrep "server-id|log-bin" /data/3308/my.cnf
/data/3308/mysql start

netstat -antup | grep 3308
#提示：最好把server-id按照IP地址最后一个小数点的数字设置
#成功标志：多了一个启动的端口3308
```

> 如果配置以后，服务启动后却没有运行起来，别忘了一定要看MySQL错误日志，在/data/3308/my.cnf最下面有错误日志路径地址。

### 5.6 多实例MySQL登陆问题分析

（1）多实例本地登录MySQL

> 多实例本地登录一般通过socket文件来指定具体登陆到哪个实例，此文件的具体位置是在MySQL编译过程或my.cnf文件中指定的。在本地登陆数据库时，登陆程序会通过socket文件来判断登陆的是哪个数据库实例。
> 例如：通过mysql -uroot -p '123123' -S /data/3307/mysql.sock可知，登陆的是3307这个实例。
> mysql.sock文件是MySQL服务器端与本地MySQL客户端进行通信的UNIX套接字文件。

（2）远程连接登陆MySQL多实例

> 远程登陆MySQL多实例中的一个实例时，通过TCP端口（port）来指定说要登陆的MySQL实例，此端口的配置是在MySQL配置文件my.cnf中指定的。
> 例如：在mysql -uyunjisuan -p '123123' -h 192.168.200.101 -P  3307中，-P为端口参数，后面接具体的实例端口，端口是一种“逻辑连接位置”，是客户端程序被分派到计算机上特殊服务程序的一种方式，强调提前在192.168.200.101上对yunjisuan用户做了授权。

