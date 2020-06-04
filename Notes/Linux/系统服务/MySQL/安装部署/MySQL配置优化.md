[TOC]

# 前言

在安装完MySQL之后，肯定是需要对MySQL的各种参数选项进行一些优化调整的。虽然MySQL系统的伸缩性很强，既可以在有很充足的硬件资源环境下高效的运行，也可以在极少资源环境下很好的运行，但不管怎样，尽可能充足的硬件资源对MySQL的性能提升总是有帮助的。在这一节我们主要分析一下MySQL的日志（主要是Binlog）对系统性能的影响，并根据日志的相关特性得出相应的优化思路。

# 日志产生的性能影响

由于日志的记录带来的直接性能损耗就是数据库系统中最为昂贵的IO资源。

在之前介绍MySQL物理架构的章节中，我们已经了解到了MySQL的日志包括错误日志（ErrorLog），更新日志（UpdateLog），二进制日志（Binlog），查询日志（QueryLog），慢查询日志（SlowQueryLog）等。当然，更新日志是老版本的MySQL才有的，目前已经被二进制日志替代。

在默认情况下，系统仅仅打开错误日志，关闭了其他所有日志，以达到尽可能减少IO损耗提高系统性能的目的。但是在一般稍微重要一点的实际应用场景中，都至少需要打开二进制日志，因为这是MySQL很多存储引擎进行增量备份的基础，也是MySQL实现复制的基本条件。有时候为了进一步的性能优化，定位执行较慢的SQL语句，很多系统也会打开慢查询日志来记录执行时间超过特定数值（由我们自行设置）的SQL语句。

一般情况下，在生产系统中很少有系统会打开查询日志。因为查询日志打开之后会将MySQL中执行的每一条Query都记录到日志中，会该系统带来比较大的IO负担，而带来的实际效益却并不是非常大。一般只有在开发测试环境中，为了定位某些功能具体使用了哪些SQL语句的时候，才会在短时间段内打开该日志来做相应的分析。所以，在MySQL系统中，会对性能产生影响的MySQL日志（不包括各存储引擎自己的日志）主要就是Binlog了。

# Binlog 相关参数及优化策略

我们首先看看Binlog的相关参数，通过执行如下命令可以获得关于Binlog的相关参数。当然，其中也显示出了“innodb_locks_unsafe_for_binlog”这个Innodb存储引擎特有的与Binlog相关的参数：

```
mysql> show variables like '%binlog%'; 
+--------------------------------+------------+ | Variable_name | Value | +--------------------------------+------------+ 
| binlog_cache_size | 1048576 | 
| innodb_locks_unsafe_for_binlog | OFF |
| max_binlog_cache_size| 4294967295 |
| max_binlog_size| 1073741824 |
| sync_binlog| 0|
+--------------------------------+------------+
```

“binlog_cache_size"：在事务过程中容纳二进制日志SQL语句的缓存大小。二进制日志缓存是服务器支持事务存储引擎并且服务器启用了二进制日志(—log-bin选项)的前提下为每个客户端分配的内存，注意，是每个Client都可以分配设置大小的binlogcache空间。如果读者朋友的系统中经常会出现多语句事务的华，可以尝试增加该值的大小，以获得更有的性能。当然，我们可以通过MySQL的以下两个状态变量来判断当前的binlog_cache_size的状况：Binlog_cache_use和Binlog_cache_disk_use。

“max_binlog_cache_size”：和"binlog_cache_size"相对应，但是所代表的是binlog能够使用的最大cache内存大小。当我们执行多语句事务的时候，max_binlog_cache_size如果不够大的话，系统可能会报出“Multi-statementtransactionrequiredmorethan'max_binlog_cache_size'bytesofstorage”的错误。

“max_binlog_size”：Binlog日志最大值，一般来说设置为512M或者1G，但不能超过1G。该大小并不能非常严格控制Binlog大小，尤其是当到达Binlog比较靠近尾部而又遇到一个较大事务的时候，系统为了保证事务的完整性，不可能做切换日志的动作，只能将该事务的所有SQL都记录进入当前日志，直到该事务结束。这一点和Oracle的Redo日志有点不一样，因为Oracle的Redo日志所记录的是数据文件的物理位置的变化，而且里面同时记录了Redo和Undo相关的信息，所以同一个事务是否在一个日志中对Oracle来说并不关键。而MySQL在Binlog中所记录的是数据库逻辑变化信息，MySQL称之为Event，实际上就是带来数据库变化的DML之类的Query语句。

“sync_binlog”：这个参数是对于MySQL系统来说是至关重要的，他不仅影响到Binlog对MySQL所带来的性能损耗，而且还影响到MySQL中数据的完整性。对于“sync_binlog”参数的各种设置的说明如下：

sync_binlog=0，当事务提交之后，MySQL不做fsync之类的磁盘同步指令刷新binlog_cache中的信息到磁盘，而让Filesystem自行决定什么时候来做同步，或者cache满了之后才同步到磁盘。

sync_binlog=n，当每进行n次事务提交之后，MySQL将进行一次fsync之类的磁盘同步指令来将binlog_cache中的数据强制写入磁盘。

在MySQL中系统默认的设置是sync_binlog=0，也就是不做任何强制性的磁盘刷新指令，这时候的性能是最好的，但是风险也是最大的。因为一旦系统Crash，在binlog_cache中的所有binlog信息都会被丢失。而当设置为“1”的时候，是最安全但是性能损耗最大的设置。因为当设置为1的时候，即使系统Crash，也最多丢失binlog_cache中未完成的一个事务，对实际数据没有任何实质性影响。从以往经验和相关测试来看，对于高并发事务的系统来说，“sync_binlog”设置为0和设置为1的系统写入性能差距可能高达5倍甚至更多。

大家都知道，MySQL的复制（Replication），实际上就是通过将Master端的Binlog通过利用IO线程通过网络复制到Slave端，然后再通过SQL线程解析Binlog中的日志再应用到数据库中来实现的。所以，Binlog量的大小对IO线程以及Msater和Slave端之间的网络都会产生直接的影响。

MySQL中Binlog的产生量是没办法改变的，只要我们的Query改变了数据库中的数据，那么就必须将该Query所对应的Event记录到Binlog中。那我们是不是就没有办法优化复制了呢？当然不是，在MySQL复制环境中，实际上是是有8个参数可以让我们控制需要复制或者需要忽略而不进行复制的DB或者Table的，分别为：

Binlog_Do_DB：设定哪些数据库（Schema）需要记录Binlog；

Binlog_Ignore_DB：设定哪些数据库（Schema）不要记录Binlog；

Replicate_Do_DB：设定需要复制的数据库（Schema），多个DB用逗号（“,”）分隔；

Replicate_Ignore_DB：设定可以忽略的数据库（Schema）；

Replicate_Do_Table：设定需要复制的Table；

Replicate_Ignore_Table：设定可以忽略的Table；

Replicate_Wild_Do_Table：功能同Replicate_Do_Table，但可以带通配符来进行设置；

Replicate_Wild_Ignore_Table：功能同Replicate_Ignore_Table，可带通配符设置；

通过上面这八个参数，我们就可以非常方便按照实际需求，控制从Master端到Slave端的Binlog量尽可能的少，从而减小Master端到Slave端的网络流量，减少IO线程的IO量，还能减少SQL线程的解析与应用SQL的数量，最终达到改善Slave上的数据延时问题。

实际上，上面这八个参数中的前面两个是设置在Master端的，而后面六个参数则是设置在Slave端的。虽然前面两个参数和后面六个参数在功能上并没有非常直接的关系，但是对于优化MySQL的Replication来说都可以启到相似的功能。当然也有一定的区别，其主要区别如下：

如果在Master端设置前面两个参数，不仅仅会让Master端的Binlog记录所带来的IO量减少，还会让Master端的IO线程就可以减少Binlog的读取量，传递给Slave端的IO线程的Binlog量自然就会较少。这样做的好处是可以减少网络IO，减少Slave端IO线程的IO量，减少Slave端的SQL线程的工作量，从而最大幅度的优化复制性能。当然，在Master端设置也存在一定的弊端，因为MySQL的判断是否需要复制某个Event不是根据产生该Event的Query所更改的数据

所在的DB，而是根据执行Query时刻所在的默认Schema，也就是我们登录时候指定的DB或者运行“USEDATABASE”中所指定的DB。只有当前默认DB和配置中所设定的DB完全吻合的时候IO线程才会将该Event读取给Slave的IO线程。所以如果在系统中出现在默认DB和设定需要复制的DB不一样的情况下改变了需要复制的DB中某个Table的数据的时候，该Event是不会被复制到Slave中去的，这样就会造成Slave端的数据和Master的数据不一致的情况出现。同样，如果在默认Schema下更改了不需要复制的Schema中的数据，则会被复制到Slave端，当Slave端并没有该Schema的时候，则会造成复制出错而停止。

而如果是在Slave端设置后面的六个参数，在性能优化方面可能比在Master端要稍微逊色一点，因为不管是需要还是不需要复制的Event都被会被IO线程读取到Slave端，这样不仅仅增加了网络IO量，也给Slave端的IO线程增加了RelayLog的写入量。但是仍然可以减少Slave的SQL线程在Slave端的日志应用量。虽然性能方面稍有逊色，但是在Slave端设置复制过滤机制，可以保证不会出现因为默认Schema的问题而造成Slave和Master数据不一致或者复制出错的问题。

# Slow Query Log 相关参数及使用建议

再来看看SlowQueryLog的相关参数配置。有些时候，我们为了定位系统中效率比较地下的Query语句，则需要打开慢查询日志，也就是SlowQueryLog。我们可以如下查看系统慢查询日志的相关设置：

```
mysql> show variables like 'log_slow%';
+------------------+-------+
| Variable_name | Value |
+------------------+-------+
| log_slow_queries | ON |
+------------------+-------+
row in set (0.00 sec)

mysql> show variables like 'long_query%';
+-----------------+-------+
| Variable_name | Value |
+-----------------+-------+
| long_query_time | 1 |
+-----------------+-------+
row in set (0.01 sec)
```

“log_slow_queries”参数显示了系统是否已经打开SlowQueryLog功能，而“long_query_time”参数则告诉我们当前系统设置的SlowQuery记录执行时间超过多长的Query。在MySQLAB发行的MySQL版本中SlowQueryLog可以设置的最短慢查询时间为1秒，这在有些时候可能没办法完全满足我们的要求，如果希望能够进一步缩短慢查询的时间限制，可以使用Percona提供的microslow-patch（件成为mslPatch）来突破该限制。mslpatch不仅仅能将慢查询时间减小到毫秒级别，同时还能通过一些特定的规则来过滤记录的SQL，如仅记录涉及到某个表的SlowQuery等等附加功能。考虑到篇幅问题，这里就不介绍mslpatch给我们带来的更为详细的功能和使用，大家请参考官方介绍（http://www.mysqlperformanceblog.com/2008/04/20/updated-msl-microslow-patch-installation-walk-through/）

打开SlowQueryLog功能对系统性能的整体影响没有Binlog那么大，毕竟SlowQueryLog的数据量比较小，带来的IO损耗也就较小，但是，系统需要计算每一条Query的执行时间，所以消耗总是会有一些的，主要是CPU方面的消耗。如果大家的系统在CPU资源足够丰富的时候，可以不必在乎这一点点损耗，毕竟他可能会给我们带来更大性能优化的收获。但如果我们的CPU资源也比较紧张的时候，也完全可以在大部分时候关闭该功能，而只需要间断性的打开SlowQueryLog功能来定位可能存在的慢查询。

