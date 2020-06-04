[TOC]

# 多实例

mysql多实例，简单理解就是在一台服务器上，mysql服务开启多个不同的端口运行多个服务进程。这些 mysql服务进程通过不同的socket来监听不同的数据端口，进而互不干涉的提供各自的服务。

在同一台服务器上，mysql多实例会去共用一套mysql应用程序，因此你在部署mysql的时候只需要部署一次mysql程序即可,无需多次部署。但是,mysql多实例之间会各自使用不同的 my.cnf 配置文件、启动程序和数据文件。在提供服务方面，mysql多实例在逻辑上看起来是各自独立，互不干涉的，并且多个实例之间是根据配置文件的设定值，来获取相关服务器的硬件资源。

下面用一个比喻,来帮助大家理解mysql多实例的本质。

mysql多实例相当于合租房，合租房里面有多个租客，每个租客都租有一个卧室，这个卧室就相当于我们的mysql的一个实例。整个合租房就相当于一台服务器。合租房里面的洗衣机、卫生间、阳台就相当于我们服务器上的各种硬件资源，比如CPU、MEM、DISK等，这些东西都是公共资源，大家共用的。

另外,多实例并不仅仅是mysql才有，其实我们日常运维中碰到的很多服务都可以部署使用多实例，并且在生产环境中也非常热衷去使用，甚至在门户网站应用也很广泛，例如nginx多实例、apache多实例、redis多实例等等。

# 多实例的优缺点

## 优点

1,有效利用服务器资源

当单个服务器资源过剩时，可以充分利用剩余的资源来提供更多的服务

2,节约服务器资源

当公司资金紧张，但数据库又需要数据库之间各自提供服务时，并且还想使用主从同步等技术，此时多实例就再好不过了

3,方便后期架构扩展

当公司的某个项目才启动时，启动初期并不一定有很大的用户量，因此可以先用一组物理数据库服务器，在上面部署多个实例，方便后续架构扩展、迁移.

## 缺点

1,资源互相抢占问题

当某个服务实例并发很高或者有慢查询时，整个实例会消耗更多的内存、CPU和IO资源，这将导致服务器上的其它实例提供服务的质量下降。这就比如说合租房的各个租客，每当早晨上班时，都会洗漱，此时卫生间的占用率就大，各个租客总会发生等待。

# 应用场景

1,当一个公司业务访问量不太大，又想节俭成本，并且还希望不同业务的数据库服务能够各自尽量独立，提供服务能够互相不受影响。另外还需要应用主从同步等技术来提供数据库备份或读写分离服务，以及方便后期业务量增大时，数据库架构的扩展和迁移。此时，Mysql多实例就再好不过了。比如，我们可以通过在 3 台服务器部署 6-9个实例，然后交叉做主从同步备份及读写分离，来实现6-9台服务器才能够达到的效果.

2,公司业务访问量不是太大的时候，服务器的资源基本都是过剩状态。此时就很适合 mysql 多实例的应用。如果对 SQL语句 优化做的比较好，mysql 多实例 是一个很值得去使用的技术。即使后期业务并发很大，只要合理分配好系统资源，也不会有太大的问题.

3,为了规避 mysql 对 SMP 架构不支持的缺陷，我们可以使用 mysql 多实例绑定处理器的办法（NUMA处理器必须支持，不过现在大部分处理器都支持的）将不同的数据库分配到不同的实例上提供数据服务.

4,传统游戏行业的 MMO/MMORPG以及Web Game，会将每个服都对应一个数据库，而且可能经常要做很多数据查询和数据订正工作。此时,为了减少维护而出错的概率，我们也可以采用多实例的部署方式，按区的概念来分配数据库。

# 多实例配置方式

mysql多实例常规来讲，有三种方案可以实现，这三种方案各有利弊，如下：

1,基于多配置文件

通过使用多个配置文件来启动不同的进程，以此来实现多实例。

优点：逻辑简单，配置简单

缺点：管理起来不方便

2,基于mysqld_multi

通过官方自带的mysqld_multi工具，使用单独配置文件来实现多实例。

优点：便于集中管理管理

缺点：不方便针对每个实例配置进行定制

3,基于IM

使用MySQL实例管理器(MYSQLMANAGER)这个方法好像比较好,不过也有点复杂。

优点：便于集中管理

缺点：耦合度高。IM一挂，实例全挂

# 配置示例

## 同版本多实例

### 方案

/data/

├── 3306

│  ├── data  --> 3306实例的数据文件

│  ├── my.cnf  --> 3306实例的配置文件

│  └── mysql  --> 3306实例的启动文件

└── 3307

  ├── data  --> 3307实例的数据文件

  ├── my.cnf  --> 3307实例的配置文件

  └── mysql  --> 3307实例的启动文件

### 安装配置

生产硬件配置参考:MEM(32G),cpux2,硬盘sas 15k(600G*6),跑2-3个实例. 

1, 安装mysql-5.5.57

安装过程略

2, 创建多实例目录

 

```
mkdir -pv /data/{3306,3307}/data
chown -R mysql.mysql /data/
tree /data/
/data/
├── 3306
│   └── data
└── 3307
    └── data
```

3,配置文件

/data/3306/mysql/my.cnf

 

```
[client]
port            = 3306
socket          = /data/3306/mysql.sock

[mysql]
no-auto-rehash

[mysqld]
user    = mysql
port    = 3306
socket  = /data/3306/mysql.sock
basedir = /usr/local/mysql/
datadir = /data/3306/data
open_files_limit    = 1024
back_log = 600
max_connections = 800
max_connect_errors = 3000
table_cache = 614
external-locking = FALSE
max_allowed_packet =8M
sort_buffer_size = 1M
join_buffer_size = 1M
thread_cache_size = 100
thread_concurrency = 2
query_cache_size = 2M
query_cache_limit = 1M
query_cache_min_res_unit = 2k
#default_table_type = InnoDB
thread_stack = 192K
#transaction_isolation = READ-COMMITTED
tmp_table_size = 2M
max_heap_table_size = 2M
long_query_time = 1
#log_long_format
#log-error = /data/3306/error.log
#log-slow-queries = /data/3306/slow.log
pid-file = /data/3306/mysql.pid
log-bin = /data/3306/mysql-bin
relay-log = /data/3306/relay-bin
relay-log-info-file = /data/3306/relay-log.info
binlog_cache_size = 1M
max_binlog_cache_size = 1M
max_binlog_size = 2M
expire_logs_days = 7
key_buffer_size = 16M
read_buffer_size = 1M
read_rnd_buffer_size = 1M
bulk_insert_buffer_size = 1M
#myisam_sort_buffer_size = 1M
#myisam_max_sort_file_size = 10G
#myisam_max_extra_sort_file_size = 10G
#myisam_repair_threads = 1
#myisam_recover

lower_case_table_names = 1
skip-name-resolve
slave-skip-errors = 1032,1062
replicate-ignore-db=mysql

server-id = 1  # 注意这里,id唯一

innodb_additional_mem_pool_size = 4M
innodb_buffer_pool_size = 32M
innodb_data_file_path = ibdata1:128M:autoextend
innodb_file_io_threads = 4
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

[mysqld_safe]
log-error=/data/3306/mysql3306.err
pid-file=/data/3306/mysqld.pid
```

启动脚本

/data/3306/mysql

 

```
#!/bin/sh
#
#init
port=3306
mysql_user="root"
mysql_pwd="3306.com"
CmdPath="/usr/local/mysql/bin"
mysql_sock="/data/${port}/mysql.sock"
#startup function
function_start_mysql()
{
    if [ ! -e "$mysql_sock" ];then
      printf "Starting MySQL...\n"
      /bin/sh ${CmdPath}/mysqld_safe --defaults-file=/data/${port}/my.cnf 2>&1 > /dev/null &
    else
      printf "MySQL is running...\n"
      exit
    fi
}

#stop function
function_stop_mysql()
{
    if [ ! -e "$mysql_sock" ];then
       printf "MySQL is stopped...\n"
       exit
    else
       printf "Stoping MySQL...\n"
       ${CmdPath}/mysqladmin -u ${mysql_user} -p${mysql_pwd} -S /data/${port}/mysql.sock shutdown
   fi
}

#restart function
function_restart_mysql()
{
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
    printf "Usage: /data/${port}/mysql {start|stop|restart}\n"
esac
```

6, 修改属主数组,服务启动脚本添加执行权限

 

```
chown -R mysql.mysql /data
ls -ld /data

find /data -type f -name "mysql" -exec chmod 700 {} \;
find /data -type f -name "mysql" -exec chown root.root {} \;
find /data -type f -name "mysql" -exec ls -l {} \;

echo 'export PATH=/usr/local/mysql/bin:$PATH' >> /etc/profile
```

6, 初始化并启动多实例的服务

 

```
# 初始化数据库
cd /usr/local/mysql
scripts/mysql_install_db --basedir=/usr/local/mysql/ --datadir=/data/3306/data --user=mysql
scripts/mysql_install_db --basedir=/usr/local/mysql/ --datadir=/data/3307/data --user=mysql

# 启动服务
/data/3306/mysql start
/data/3307/mysql start
ss -tnlp | grep 330[6-7]

# 登录
mysql -S /data/3306/mysql.sock
> CREATE DATABASE d3306;
> SHOW DATABASES;

# 在不退出sql的情况下,连接另外的mysql服务,退出后会回到原来的mysql服务中
> SYSTEM mysql -S /data/3306/mysql.sock

# 为mysql登录设置密码
mysqladmin -u root -S /data/3306/mysql.sock password '3306.com'
mysqladmin -u root -S /data/3307/mysql.sock password '3307.com'
```

需要注意的是,mysql_install_db初始化脚本的路径,不同版本路径不同.

服务器起不来,排查办法: 

检查/data目录权限, 是否是mysql;

查看错误日志, 错误日志的路径: grep log-error my.cnf | tail -l

遇到的问题:

mysql多实例,启动不起来,查看错误日志如下:

 

```
180305 16:14:06 [ERROR] InnoDB: auto-extending data file ./ibdata1 is of a different size 768 pages (rounded down to MB) than specified in the .cnf file: initial 8192 pages, max 0 (relevant if non-zero) pages!
180305 16:14:06 [ERROR] InnoDB: Could not open or create the system tablespace. If you tried to add new data files to the system tablespace, and it failed here, you should now edit innodb_data_file_path in my.cnf back to what it was, and remove the new ibdata files InnoDB created in this failed attempt. InnoDB only wrote those files full of zeros, but did not yet use them in any way. But be careful: do not remove old data files which contain your precious data!
180305 16:14:06 [ERROR] Plugin 'InnoDB' init function returned error.
180305 16:14:06 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
180305 16:14:06 [Note] Plugin 'FEEDBACK' is disabled.
180305 16:14:06 [ERROR] Unknown/unsupported storage engine: InnoDB
180305 16:14:06 [ERROR] Aborting
```

数据库我用的mariadb-10.x版本,二进制安装,通过google,查找到解决办法:

 

```
# 进入到初始化数据库的数据存储目录
# 删除这2个文件：ib_logfile0和ib_logfile1

```

### 启动与关闭

#### 单实例

 

```
# 启动服务方式1
/etc/init.d/mysqld start
ss -tnlp | grep 3306

# 启动服务方式2
mysqld_safe --user=mysql

# 关闭方式1
/etc/init.d/mysqld stop

# 关闭方式2
mysqladmin -uroot -p shutdown

# 登录
# 无密码
mysql

# 有密码
mysql -uroot -p -h172.16.100.7 -P3306
```

服务自带停启脚本文件。

#### **多实例**

 

```
# 启动1
/data/3306/mysql start
/data/3307/mysql start
ss -tnlp | grep 330[6-7]
# 启动2
mysqld_safe --defaults-file=/data/3306/my.cnf --user=mysql &
mysqld_safe --defaults-file=/data/3307/my.cnf --user=mysql &

# 关闭方式1
/data/3306/mysql stop
/data/3307/mysql stop

# 关闭方式2
mysqladmin -uroot -p -S /tmp/3306/mysql.sock shutdown
mysqladmin -uroot -p -S /tmp/3307/mysql.sock shutdown


# 多实例登录方式:
# 本地登录
mysql -uroot -p -h172.16.100.7 -S /data/3306/mysql.sock
mysql -uroot -p -h172.16.100.7 -S /data/3307/mysql.sock
# 远程登录
mysql -uroot -p -h172.16.100.7 -P3306
mysql -uroot -p -h172.16.100.7 -P3307

# 在不退出mysql命令行的情况下,连接另外的mysql服务,退出后会回到原来的mysql服务中
> SYSTEM mysql -S /data/3306/mysql.sock

# 为mysql登录设置密码
mysqladmin -u root password '3306.com' -S /data/3306/mysql.sock
mysqladmin -u root password '3307.com' -S /data/3307/mysql.sock

# 修改密码方法1
mysqladmin -uroot -p3306.com password '3306.cn'
mysqladmin -uroot -p3307.com password '3307.cn'

# 修改密码方法2,此方法可在数据库密码丢失或忘记的情况下,可以以--skip-grant-tables参数来启动数据库重置密码
mysql
> UPDATE mysql.user SET password=PASSWORD("3306.cn") WHERE user='root' AND host='localhost'; # 如不指定where条件,则会修改所有用户,PASSWORD()密码加密
> FLUSH PRIVILEGES;
```

使用的是自定义脚本停启服务。

#### 密码找回

 

```
# 方法1
vi /etc/my.cnf
"""
 [mysqld] 
 datadir=/var/lib/mysql 
 socket=/var/lib/mysql/mysql.sock 
 skip-grant-tables  # [mysqld]的段中加上一句：skip-grant-tables
"""

/etc/init.d/mysqld restart 
# 然后可直接免密进入mysql,使用update更改密码后,在删除配置文件中添加的那条数据,重启服务即可

# 方法2
# 单实例
/etc/init.d/mysqld stop
mysqld_safe --skip-grant-tables --user=mysql
mysql

> UPDATE mysql.user SET password=PASSWORD("3306.com") WHERE user='root' AND host='localhost';
> FLUSH PRIVILEGES;
> \q

mysqladmin -uroot -p shutdown
password:  # 3306.com

/etc/init.d/mysqld start

mysqld -uroot -p
password: # 3306.com

# 多实例
/data/3306/mysql stop # 若停服务也需要密码,则可killall
ss -tnlp
mysqld_safe --defaults-file=/data/3306/my.cnf --skip-grant-tables
ss -tnlp
mysql -uroot -p -S /data/3306/mysql.sock
password: # 免密登录

> UPDATE mysql.user SET Password=PASSWORD("3306.com") WHERE user='root' AND host='localhost';
> FLUSH PRIVILEGES;
> \q

mysqladmin -uroot -p -S /data/3306/mysql.sock shutdown
ss -tnlp
/data/3306/mysql start
ss -tnlp
mysql -uroot -p -S /data/3306/mysql.sock
```

# 

总结：修改配置文件；mysqld_safe

## **不同版本多实例**

有时候可能会需要多版本共存，这里以5.5和5.6两个版本为例

### 方案

/data/

├── 3306

│ ├── data  --> 3306实例的数据文件

│ ├── my.cnf  --> 3306实例的配置文件

│ └── mysql  --> 3306实例的启动文件

└── 3307

  ├── data  --> 3307实例的数据文件

  ├── my.cnf  --> 3307实例的配置文件

  └── mysql  --> 3307实例的启动文件

其中3306用于5.5，3307用于5.6

### 安装配置

安装和配置差不多，只需修改一下配置文件和启动脚本文件

 

```
# /data/3306/my.cnf
"""
basedir = /usr/local/mysql5/
"""

# /data/3307/my.cnf
"""
basedir = /usr/local/mysql6/
"""


# /data/3306/mysql5.cnf
"""
CmdPath="/usr/local/mysql/bin"
"""

# /data/3307/mysql6.cnf
"""
CmdPath="/usr/local/mysql6/bin"
"""
```

### 启动和关闭

 

```
# 启动
/usr/local/mysql5/bin/mysqld_safe --defaults-file=/data/3306/my.cnf --user=mysql &

/usr/local/mysql6/bin/mysqld_safe --defaults-file=/data/3307/my.cnf --user=mysql &

# 关闭
mysqladmin -uroot -p -S /tmp/3306/mysql.sock shutdown
mysqladmin -uroot -p -S /tmp/3307/mysql.sock shutdown


# 本地登录
/usr/local/mysql5/bin/mysql -uroot -p -h172.16.100.7 -S /data/3306/mysql.sock
/usr/local/mysql6/bin/mysql -uroot -p -h172.16.100.7 -S /data/3307/mysql.sock

# 远程登录
mysql -uroot -p -h172.16.100.7 -P3306
mysql -uroot -p -h172.16.100.7 -P3307
```