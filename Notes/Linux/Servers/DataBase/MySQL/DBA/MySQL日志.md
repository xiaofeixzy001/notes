[TOC]

MySQL日志

SET SESSION ...=    ###设定变量

\#mysqlbinlog+日志文件  ###查看日志文件内容

  -j，--start-position=#     ###从指定的事件位置查看

  --stop-position=#        ###只显示到指定的事件位置

  --start-datetime=name    ###从指定的事件时间查看

  --stop-datetime=name    ###之显示到指定的时间位置，时间格式：YYYY-MM-DD hh：mm：ss

mysql日志的配置

mysql的数据文件：文件和日志

文件：数据文件和索引文件

日志：事务日志，二进制日志，查询日志，慢查询日志，错误日志，中继日志

MySQL日志

binlog日志文件

Mysql的binlog日志作用是用来记录mysql内部增删改查等对mysql数据库有更新的内容的记录（对数据库的改动），对数据库的查询select或show等不会被binlog日志记录;主要用于数据库的主从复制以及增量恢复。

mysql的binlog日志必须打开log-bin功能才能生成binlog日志;

打开方法在mysql的配置文件my.cnf中[mysqld]配置段下;

binlog日志文件格式如下:

mysql-bin.000003

mysql-bin.000004

mysql-bin.000005

 

```
# 编辑my.cnf配置文件
~]# grep log-bin my.cnf
log-bin = /data/mysql/mysql-bin

# 查看是否启用了日志
mysql>show variables like 'log_bin';
```

Mysqlbinlog功能是将Mysql的binlog日志转换成Mysql语句，默认情况下binlog日志是二进制文件，无法直接查看。

mysqlbinlog binlog_file

-d 	指定库的binlog

-r 	相当于重定向到指定文件

--start-position=#

--stop-position=#

按照指定位置精确解析binlog日志（精确），如不接--stop-positiion则一直到binlog日志结尾 ;

\#为binlog文件中"at #"的数字

--start-datetime=#

--stop-datetime=#

按照指定时间解析binlog日志（模糊，不准确），如不接--stop-datetime则一直到binlog日志结尾;

\#为binlog文件中的时间,时间格式: YYYY-MM-DD hh:mm:ss

binlog的三种工作模式

Row level

日志中会记录每一行数据被修改的情况，然后在slave端对相同的数据进行修改。

优点：能清楚的记录每一行数据修改的细节

缺点：数据量太大

Statement level（默认）

每一条被修改数据的sql都会记录到master的bin-log中，slave在复制的时候sql进程会解析成和原来master端执行过的相同的sql再次执行

优点：解决了 Row level下的缺点，不需要记录每一行的数据变化，减少bin-log日志量，节约磁盘IO，提高新能

缺点：容易出现主从复制不一致

Mixed（混合模式）

结合了Row level和Statement level的优点

企业binlog模式的选择

互联网公司使用MySQL的功能较少（不用存储过程、触发器、函数），选择默认的Statement level

用到MySQL的特殊功能（存储过程、触发器、函数）则选择Mixed模式

用到MySQL的特殊功能（存储过程、触发器、函数），又希望数据最大化一直则选择Row模式

 

```
# 查看MySQLbinlog模式
> SHOW GLOBAL VARIABLES LIKE 'binlog%';

# 设置MySQL binlog模式
> SET GLOBAL binlog_format='ROW';

# 配置文件中设置binlog模式
~]# vim my.cnf
"""
[mysqld]
binlog_format='ROW'          #放在mysqld模块下面
...
"""
```

ROW模式下解析binlog日志

 

```
mysqlbinlog --base64-output='decode-rows' -v mysql-bin.000001
```

日志类型

查询日志：记录查询操作，一般不开启

log_output = {TABLE | FILE | NONE}   ###表示日志记录格式是表、文件或不记录，逗号隔开同时记录两种方式；

log_output = TABLE,FILE      ###表示同时记录为TABLE格式和FILE格式

FILE：genernel_log

general_log = {ON | OFF}    ###是否启用查询日志

general_log = www.log　　　　　  ###当log_output有FILE类型时，日志信息的记录位置

慢查询日志：执行查询操作时，所需要的时长超过了指定时长，记录此类操作的日志，可通过此日志来查看定位语句执行速度慢的原因所在。

  \>SELECT @@GLOBAL.long_query_time；    ###查看指定时长，如果超过指定时长则会被记录到慢查询日志中

​     slow_query_log = {ON | OFF}    ###是否启用慢查询日志

​     log_output = FILE       ###当慢查询日志类型定义为FILE类型时

​     slow_query_log_file = www-slow.log ###慢查询日志类型存储位置

错误日志：记录错误信息的日志(重要）

  mysqld启动和关闭过程中输出的信息；

  mysqld运行中产生的错误信息;

  event scheduler运行一个事件时产生的日志信息

  在主从复制架构中的从服务器上，启动从服务器线程时产生的日志信息；

​    log_error = /mydata/data/log.err  ###错误日志位置 

​    log_warnings = {ON | OFF}    ###是否记录警告信息

二进制日志：记录引起元数据改变的操作，也称重做日志(重要)

  \>SHOW BINARY LOGS；    ###查看服务器二进制日志信息

  \>SHOW BINLOG EVENTS [IN 'log_name'] [FROM pos] [LIMIT [offset，] row_count]；  ###显示指定二进制日志文件log_name中，相关的事件

 例如：>SHOW BINLOG EVENTS IN 'mysql-bin.000001' FROM 351 LIMIT 2；

日志记录格式：

​    binlog_format =  

​    基于‘语句’记录：statement

​    基于‘行’记录：row

   ‘混合’记录，mixed

mysql-bin.index：索引文件，记录了二进制日志文件列表

log_bin = /path/to/somefile  ###定义二进制日志文件位置和名字

sql_log_bin = ON | OFF   #是否启用二进制日志格式

max_binlog_size = 1073741824  ###二进制日志文件单个大小，字节

max_binlog_cache_size = 18446744073709547520  ###二进制缓存空间大小

max_binlog_stmt_cache_size = 18446744073709547520  ###语句缓存大小

sync_binlog = 0   ###设定多久同步一次二进制日志文件，0表示不同步

中继日志：一般是从服务器上的，复制于主服务器二进制日志文件，以便从服务器在空闲时慢慢执行同步

事务日志：保证一个事务满足ACID标准，可将随机I/O转换成顺序I/O

  一般是innodb存储引擎

日志相关的服务器参数详解：

expire_logs_days={0..99}

设定二进制日志的过期天数，超出此天数的二进制日志文件将被自动删除。默认为0，表示不启用过期自动删除功能。如果启用此功能，自动删除工作通常发生在MySQL启动时或FLUSH日志时。作用范围为全局，可用于配置文件，属动态变量。

general_log={ON|OFF}

设定是否启用查询日志，默认值为取决于在启动mysqld时是否使用了--general_log选项。如若启用此项，其输出位置则由--log_output选项进行定义，如果log_output的值设定为NONE，即使用启用查询日志，其也不会记录任何日志信息。作用范围为全局，可用于配置文件，属动态变量。

 

general_log_file=FILE_NAME

查询日志的日志文件名称，默认为“hostname.log"。作用范围为全局，可用于配置文件，属动态变量。

binlog-format={ROW|STATEMENT|MIXED}

指定二进制日志的类型，默认为STATEMENT。如果设定了二进制日志的格式，却没有启用二进制日志，则MySQL启动时会产生警告日志信息并记录于错误日志中。作用范围为全局或会话，可用于配置文件，且属于动态变量。

log={YES|NO}

是否启用记录所有语句的日志信息于一般查询日志(general query log)中，默认通常为OFF。MySQL 5.6已经弃用此选项。

 

log-bin={YES|NO}

是否启用二进制日志，如果为mysqld设定了--log-bin选项，则其值为ON，否则则为OFF。其仅用于显示是否启用了二进制日志，并不反应log-bin的设定值。作用范围为全局级别，属非动态变量。

 

log_bin_trust_function_creators={TRUE|FALSE}

此参数仅在启用二进制日志时有效，用于控制创建存储函数时如果会导致不安全的事件记录二进制日志条件下是否禁止创建存储函数。默认值为0，表示除非用户除了CREATE ROUTING或ALTER ROUTINE权限外还有SUPER权限，否则将禁止创建或修改存储函数，同时，还要求在创建函数时必需为之使用DETERMINISTIC属性，再不然就是附带READS SQL DATA或NO SQL属性。设置其值为1时则不启用这些限制。作用范围为全局级别，可用于配置文件，属动态变量。

 

log_error=/PATH/TO/ERROR_LOG_FILENAME

定义错误日志文件。作用范围为全局或会话级别，可用于配置文件，属非动态变量。

 

log_output={TABLE|FILE|NONE}

定义一般查询日志和慢查询日志的保存方式，可以是TABLE、FILE、NONE，也可以是TABLE及FILE的组合(用逗号隔开)，默认为TABLE。如果组合中出现了NONE，那么其它设定都将失效，同时，无论是否启用日志功能，也不会记录任何相关的日志信息。作用范围为全局级别，可用于配置文件，属动态变量。

 

log_query_not_using_indexes={ON|OFF}

设定是否将没有使用索引的查询操作记录到慢查询日志。作用范围为全局级别，可用于配置文件，属动态变量。

 

log_slave_updates

用于设定复制场景中的从服务器是否将从主服务器收到的更新操作记录进本机的二进制日志中。本参数设定的生效需要在从服务器上启用二进制日志功能。

 

log_slow_queries={YES|NO}

是否记录慢查询日志。慢查询是指查询的执行时间超出long_query_time参数所设定时长的事件。MySQL 5.6将此参数修改为了slow_query_log。作用范围为全局级别，可用于配置文件，属动态变量。

 

log_warnings=#

设定是否将警告信息记录进错误日志。默认设定为1，表示启用；可以将其设置为0以禁用；而其值为大于1的数值时表示将新发起连接时产生的“失败的连接”和“拒绝访问”类的错误信息也记录进错误日志。

long_query_time=#

设定区别慢查询与一般查询的语句执行时间长度。这里的语句执行时长为实际的执行时间，而非在CPU上的执行时长，因此，负载较重的服务器上更容易产生慢查询。其最小值为0，默认值为10，单位是秒钟。它也支持毫秒级的解析度。作用范围为全局或会话级别，可用于配置文件，属动态变量。

max_binlog_cache_size{4096 .. 18446744073709547520}

二进定日志缓存空间大小，5.5.9及以后的版本仅应用于事务缓存，其上限由max_binlog_stmt_cache_size决定。作用范围为全局级别，可用于配置文件，属动态变量。

max_binlog_size={4096 .. 1073741824}

设定二进制日志文件上限，单位为字节，最小值为4K，最大值为1G，默认为1G。某事务所产生的日志信息只能写入一个二进制日志文件，因此，实际上的二进制日志文件可能大于这个指定的上限。作用范围为全局级别，可用于配置文件，属动态变量。

max_relay_log_size={4096..1073741824}

设定从服务器上中继日志的体积上限，到达此限度时其会自动进行中继日志滚动。此参数值为0时，mysqld将使用max_binlog_size参数同时为二进制日志和中继日志设定日志文件体积上限。作用范围为全局级别，可用于配置文件，属动态变量。

innodb_log_buffer_size={262144 .. 4294967295}

设定InnoDB用于辅助完成日志文件写操作的日志缓冲区大小，单位是字节，默认为8MB。较大的事务可以借助于更大的日志缓冲区来避免在事务完成之前将日志缓冲区的数据写入日志文件，以减少I/O操作进而提升系统性能。因此，在有着较大事务的应用场景中，建议为此变量设定一个更大的值。作用范围为全局级别，可用于选项文件，属非动态变量。

 

innodb_log_file_size={108576 .. 4294967295}

设定日志组中每个日志文件的大小，单位是字节，默认值是5MB。较为明智的取值范围是从1MB到缓存池体积的1/n，其中n表示日志组中日志文件的个数。日志文件越大，在缓存池中需要执行的检查点刷写操作就越少，这意味着所需的I/O操作也就越少，然而这也会导致较慢的故障恢复速度。作用范围为全局级别，可用于选项文件，属非动态变量。

 

innodb_log_files_in_group={2 .. 100}

设定日志组中日志文件的个数。InnoDB以循环的方式使用这些日志文件。默认值为2。作用范围为全局级别，可用于选项文件，属非动态变量。

 

innodb_log_group_home_dir=/PATH/TO/DIR

设定InnoDB重做日志文件的存储目录。在缺省使用InnoDB日志相关的所有变量时，其默认会在数据目录中创建两个大小为5MB的名为ib_logfile0和ib_logfile1的日志文件。作用范围为全局级别，可用于选项文件，属非动态变量。

relay_log=file_name

设定中继日志的文件名称，默认为host_name-relay-bin。也可以使用绝对路径，以指定非数据目录来存储中继日志。作用范围为全局级别，可用于选项文件，属非动态变量。

relay_log_index=file_name

设定中继日志的索引文件名，默认为为数据目录中的host_name-relay-bin.index。作用范围为全局级别，可用于选项文件，属非动态变量。

relay-log-info-file=file_name

设定中继服务用于记录中继信息的文件，默认为数据目录中的relay-log.info。作用范围为全局级别，可用于选项文件，属非动态变量。

relay_log_purge={ON|OFF}

设定对不再需要的中继日志是否自动进行清理。默认值为ON。作用范围为全局级别，可用于选项文件，属动态变量。

relay_log_space_limit=#

设定用于存储所有中继日志文件的可用空间大小。默认为0，表示不限定。最大值取决于系统平台位数。作用范围为全局级别，可用于选项文件，属非动态变量。

slow_query_log={ON|OFF}

设定是否启用慢查询日志。0或OFF表示禁用，1或ON表示启用。日志信息的输出位置取决于log_output变量的定义，如果其值为NONE，则即便slow_query_log为ON，也不会记录任何慢查询信息。作用范围为全局级别，可用于选项文件，属动态变量。

slow_query_log_file=/PATH/TO/SOMEFILE

设定慢查询日志文件的名称。默认为hostname-slow.log，但可以通过--slow_query_log_file选项修改。作用范围为全局级别，可用于选项文件，属动态变量。

sql_log_bin={ON|OFF}

用于控制二进制日志信息是否记录进日志文件。默认为ON，表示启用记录功能。用户可以在会话级别修改此变量的值，但其必须具有SUPER权限。作用范围为全局和会话级别，属动态变量。

sql_log_off={ON|OFF}

用于控制是否禁止将一般查询日志类信息记录进查询日志文件。默认为OFF，表示不禁止记录功能。用户可以在会话级别修改此变量的值，但其必须具有SUPER权限。作用范围为全局和会话级别，属动态变量。

sync_binlog=#

设定多久同步一次二进制日志至磁盘文件中，0表示不同步，任何正数值都表示对二进制每多少次写操作之后同步一次。当autocommit的值为1时，每条语句的执行都会引起二进制日志同步，否则，每个事务的提交会引起二进制日志同步。