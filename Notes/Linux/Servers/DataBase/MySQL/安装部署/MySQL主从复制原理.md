[TOC]

# 架构

1,单向主从复制[M -> S]

![img](MySQL%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6%E5%8E%9F%E7%90%86.assets/1be9060f-64bc-4768-aa60-1293db33b910.png)

2,双向,互为主从[M/S <-> S/M]

![img](MySQL%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6%E5%8E%9F%E7%90%86.assets/88751e0a-5ef0-457d-8ed8-38a9f8e81e5c.png)

双主/Dual Master架构适用于写压力比较大的场景,或者DBA做维护需要主从切换的场景,通过双主/Dual master架构避免了重复搭建从库的麻烦.

最大问题就是更新冲突.

可以采用MySQL Cluster,以及将Cluster和Replication结合起来,可以建立强大的高性能的数据库平台.

3,链式级联一主多从[A -> B -> C -> D]

![img](MySQL%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6%E5%8E%9F%E7%90%86.assets/67825831-2288-47fc-9439-798ace174597.png)

在主库读取请求压力非常大的场景下,可以通过配置一主多从复制架构实现读写分离,把大量对实时性要求不是特别高的读请求通过负载均衡到多个从库上,降低主库的读取压力.在主库出现异常宕机的情况下,可以把一个从库切换为主库继续提供服务.

当Slave增加到一定数量时,Slave对Master的负载以及网络带宽都会成为一个严重的问题;

不同的Slave扮演不同的作用(例如使用不同的索引,或者不同的存储引擎);

用一个Slave作为备用Master,只进行复制;

用一个远程的Slave,用于灾难恢复.

# 主从复制原理

MySQL主从复制的基础是二进制日志文件(binary log file).

一台MySQL数据库作为master,需启用其二进制日志功能,它的数据库中所有操作都会以“事件”的方式记录在二进制日志中;

slave通过自己的一个I/O线程与主服务器保持通信,并监控master的二进制日志文件的变化,如果发现master二进制日志文件发生变化,则会把变化复制到自己的中继日志(relay-log)中,然后slave的另一个SQL线程会把相关的"事件"(也就是检测到的变化)在自己的数据库中执行,以此实现从数据库和主数据库的一致性,也就实现了主从复制.

![img](MySQL%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6%E5%8E%9F%E7%90%86.assets/55f46d90-6ccb-4931-b098-816b90aff38b.jpg)

dump thread:主服务器用于接收从节点的I/O线程请求

从服务器的IO线程通过master.info文件中的信息,来监控主服务器的二进制文件,有变化则会拉取到本地,存到本地的relay-log中继日志文件中,然后另外一个线程SQL,则是监控本地的中继日志文件,一旦有了数据,则会在本地执行sql语句来更新自己的数据库信息,更新完了会将当前状态记录到relay-log.info中.

类似老师讲课,老师讲到哪了?(relay-log)

我学到哪里了?(relay-log.info)

# 主从复制优缺点

## 优点

如果主服务器出现问题,可以快速切换到从服务器提供服务[高可用故障切换];

可以在从服务器上执行查询,降低主服务器的压力[读写分离];

可以在从服务器上执行备份,以避免备份期间影响主服务器的性能[冗余备份];

可复制一份用于升级测试;

## 缺点

由于MySQL实现的是异步复制,所以主从服务器之间的数据存在一定差异,对实时性要求高的数据仍然需要从主服务器上获得;

所有对数据库内容的更新就必须在主服务器上进行;

# 主从复制类型

## 基于语句的复制

在主服务器上执行的 SQL 语句,在从服务器上执行同样的语句.否则,你必须要小心,以避免用户对主服务器上的表进行的更新与对从服务器上的表所进行的更新之间的冲突.

配置：binlog_format = 'STATEMENT'

## 基于行的复制

把改变的内容复制过去,而不是把命令在从服务器上执行一遍,从 MySQL 5.0开始支持

配置：binlog_format = 'ROW'

## 混合类型的复制

默认采用基于语句的复制,一旦发现基于语句的无法精确的复制时,就会采用基于行的复制

配置：binlog_format = 'MIXED'

# 主从复制原则

\- 每个 Slave 只能有一个 Master

\- 每个 Slave 只能有一个唯一的服务器ID

\- 每个 Master 可以有很多 Slave

\- 如果你在slave上设置了log-bin和log_slave_updates,Slave可以是其他Slave的Master,从而扩散 Master的更新

\- MySQL 不支持多主服务器复制---即一个 Slave 可以有多个 Master,但是,通过一些简单的组合,我们却可以建立灵活而强大的复制体系结构

# 半同步复制

从MySQL5.5开始，MySQL以插件的形式支持半同步复制。如何理解半同步呢？首先我们来看看异步、全同步的概念

## 异步复制(Asynchronous replication)

MySQL默认的复制即是异步的，主库在执行完客户端提交的事务后会立即将结果返给给客户端，并不关心从库是否已经接收并处理，这样就会有一个问题，主如果crash掉了，此时主上已经提交的事务可能并没有传到从上，如果此时，强行将从提升为主，可能导致新主上的数据不完整。

## 全同步复制(Fully synchronous replication)

指当主库执行完一个事务，所有的从库都执行了该事务才返回给客户端。因为需要等待所有从库执行完该事务才能返回，所以全同步复制的性能必然会收到严重的影响。

## 半同步复制(Semisynchronous replication)

介于异步复制和全同步复制之间，主库在执行完客户端提交的事务后不是立刻返回给客户端，而是等待至少一个从库接收到并写到relay log中才返回给客户端。相对于异步复制，半同步复制提高了数据的安全性，同时它也造成了一定程度的延迟，这个延迟最少是一个TCP/IP往返的时间。所以，半同步复制最好在低延时的网络中使用.

# master参数说明

log-bin=mysql-bin

1,控制master是否开启binlog记录功能;

2,二进制文件最好放在单独的目录下,这不但方便优化,更方便维护;

3,重新命名二进制日志很简单,只需要修改[mysqld]里的log_bin选项,

如下例子:要重新调整logbin的路径为"/home/mysql/binlog"

 

```
ll /home/mysql/binlog
-rw-rw---- 1 mysql mysql 98 Mar 7 17:24 binlog.000001
-rw-rw---- 1 mysql mysql 33 Mar 7 17:24 binlog.index

# my.cnf
[mysqld]
log_bin=/home/mysql/binlog/binlog.log

# 需要注意:指定目录时候一定要以*.log结尾,即不能仅仅指定到文件夹的级别,否则在重启mysql时会报错.
```

innodb_flush_log_at_trx_commit=1

此参数表示在事务提交时,处理重做日志的方式.此变量有三个可选值0，1，2:

0:当事务提交时,并不将事务的重做日志写入日志文件,而是等待每秒刷新一次;

1:当事务提交时,将重做日志缓存的内容同步写到磁盘日志文件,为了保证数据一致性,在replication环境中使用此值;

2:当事务提交时,将重做日志缓存的内容异步写到磁盘日志文件(写到文件系统缓存中);

建议设置innodb_flush_log_at_trx_commit=1 

sync_binlog=1

此参数表示每写缓冲多少次就同步到磁盘;

sync_binlog=1表示同步写缓冲和磁盘二进制日志文件,不使用文件系统缓存;

在使用innodb事务引擎时,在复制环境中,为了保证最大的可用性,都设置为1,但会对影响io的性能;

即使设置为1,也会有问题发生:

假如当二进制日志写入磁盘,但事务还没有commit,这个时候宕机,当服务再次起来的恢复的时候,无法回滚以及记录到二进制日志的未提交的内容,这个时候就会造成master和slave数据不一致;

解决方案:

需要参数innodb_support_xa=1来保证,建议必须设置.

innodb_support_xa=1

此参数与XA事务有关,它保证了二进制日志和innodb数据文件的同步,保证复制环境中数据一致性,建议设置.

binlog_do_db=skate_db

只记录指定数据库的更新到二进制日志中

binlog_do_table=skate_tab

只记录指定表的更新到二进制日志中

binlog-ignore-db=skate_db

忽略指定数据库的更新到二进制日志中

log_slave_updates=1

此参数控制slave数据库是否把从master接受到的log并在本slave执行的内容记录到slave的二进制日志中

在级联复制环境中(包括双master环境),这个参数是必须的.

binlog_format=statement|row|mixed

控制以什么格式记录二进制日志的内容,默认是mixed

max_binlog_size

master的每个二进制日志文件的大小,默认1G

binlog_cache_size

所有未提交的事务都会被记录到一个缓存或临时文件中,待提交时,统一同步到二进制日志中;

此变量是基于session的,每个会话开启一个binlog_cache_size大小的缓存;

通过变量"Binlog_cache_disk_use"和"Binlog_cache_use"来设置binlog_cache_size的大小;

说明:

Binlog_cache_disk_use:使用临时文件写二进制日志的次数

Binlog_cache_use:使用缓冲记写二进制的次数

auto_increment_increment=2 //增长的步长

auto_increment_offset=1 //起始位置

在双master环境下可以防止键值冲突

# slave参数说明

server-id=2

和master的含义一样,服务标识,唯一

log-bin=mysql-bin

和master的含义一样,开启二进制

relay-log=relay-bin

中继日志文件的路径名称

relay-log-index=relay-bin

中继日志索引文件的路径名称

log_slave_updates=1

和master的含义一样,如上

read_only=1

使数据库只读,此参数在slave的复制环境和具有super权限的用户不起作用;

对于复制环境设置read_only=1非常有用;它可以保证slave只接受master的更新;而不接受client的更新;

客户端设置:

mysq> set global read_only=1

skip_slave_start

使slave在mysql启动时不启动复制进程,mysql起来之后使用start slave启动,建议必须.

replicate-do-db=

只复制指定db,逗号隔开多个

replicate-do-table=

只复制指定表,逗号隔开多个

replicate-ingore-table=

忽略指定表,逗号隔开多个

replicate_wild_do_table=skatedb.%

模糊匹配复制指定db

auto_increment_increment=2

auto_increment_offset=1

和master含义一样,参考如上

log_slow_slave_statements

在slave上开启慢查询日志,在query的时间大于long_query_time时,记录在慢查询日志里

max_relay_log_size

slave上的relay-log的大小,默认是1G

relay_log_info_file

中继日志状态信息文件的路径名称

relay_log_purge

当relay-log不被需要时就删除,默认是on

SET GLOBAL relay_log_purge=1

replicate-rewrite-db=from_name->to_name

数据库的重定向,可以把分库汇总到主库便于统计分析