[TOC]

配置文件说明

my.cnf

```
[client]    
port = 3306 
socket = /apps/mysql/mysql.sock

[mysqld] # 此配置段设置启动mysql服务的条件
user = mysql
port = 3306
socket = /apps/mysql/mysql.sock
pid-file = /apps/mysql/mysql.pid
basedir = /apps/mysql/
datadir = /data/mysql/
open_files_limit = 10240

back_log = 600
# 在MYSQL暂时停止响应新请求之前,短时间内的多少个请求可以被存在堆栈中,如果系统在短时间内有很多连接,则需要增大该参数的值,该参数值指定到来的TCP/IP连接的监听队列的大小.默认值80.

max_connections = 3000
# MySQL允许最大的进程连接数,如果经常出现Too Many Connections的错误提示,则需要增大此值,默认151

max_connect_errors = 6000
# 设置每个主机的连接请求异常中断的最大次数,当超过该次数,MYSQL服务器将禁止host的连接请求,直到mysql服务器重启或通过flush hosts命令清空此host的相关信息,默认100

external-locking = FALSE
# 使用–skip-external-locking MySQL选项以避免外部锁定,该选项默认开启

max_allowed_packet = 32M
# 设置在网络传输中一次消息传输量的最大值,系统默认值 为4MB,最大值是1GB,必须设置1024的倍数.

tmpdir = /home/mysql/mysql/tmp/    
slave-load-tmpdir = /home/mysql/mysql/tmp/    
#当slave 执行 load data infile 时用    
#language = /home/mysql/mysql/share/mysql/english/    
character-sets-dir = /home/mysql/mysql/share/mysql/charsets/

sort_buffer_size = 2M  
# Sort_Buffer_Size 是一个connection级参数,在每个connection（session）第一次需要使用这个buffer的时候,一次性分配设置的内存.
# Sort_Buffer_Size 并不是越大越好,由于是connection级的参数,过大的设置+高并发可能会耗尽系统内存资源,例如:500个连接将会消耗 500*sort_buffer_size(8M)=4G内存
# Sort_Buffer_Size 超过2KB的时候,就会使用mmap() 而不是 malloc() 来进行内存分配,导致效率降低.系统默认2M,使用默认值即可

join_buffer_size = 2M  
# 用于表间关联缓存的大小,和sort_buffer_size一样,该参数对应的分配内存也是每个连接独享.系统默认2M,使用默认值即可

thread_cache_size = 300  
# 默认38
# 服务器线程缓存这个值表示可以重新利用保存在缓存中线程的数量,当断开连接时如果缓存中还有空间,那么客户端的线程将被放到缓存中,如果线程重新被请求，那么请求将从缓存中读取,如果缓存中是空的或者是新的请求，那么这个线程将被重新创建,如果有很多新的线程，增加这个值可以改善系统性能.通过比较 Connections 和 Threads_created 状态的变量，可以看到这个变量的作用。设置规则如下：1GB 内存配置为8，2GB配置为16，3GB配置为32，4GB或更高内存，可配置更大。

thread_concurrency = 8  
# 系统默认为10,使用10先观察
# 设置thread_concurrency的值的正确与否, 对mysql的性能影响很大, 在多个cpu(或多核)的情况下，错误设置了thread_concurrency的值, 会导致mysql不能充分利用多cpu(或多核), 出现同一时刻只能一个cpu(或核)在工作的情况。thread_concurrency应设为CPU核数的2倍. 比如有一个双核的CPU, 那么thread_concurrency的应该为4; 2个双核的cpu, thread_concurrency的值应为8

query_cache_size = 64M  
#在MyISAM引擎优化中，这个参数也是一个重要的优化参数。但也爆露出来一些问题。机器的内存越来越大，习惯性把参数分配的值越来越大。这个参数加大后也引发了一系列问题。我们首先分析一下 query_cache_size的工作原理：一个SELECT查询在DB中工作后，DB会把该语句缓存下来，当同样的一个SQL再次来到DB里调用时，DB在该表没发生变化的情况下把结果从缓存中返回给Client。这里有一个关建点，就是DB在利用Query_cache工作时，要求该语句涉及的表在这段时间内没有发生变更。那如果该表在发生变更时，Query_cache里的数据又怎么处理呢？首先要把Query_cache和该表相关的语句全部置为失效，然后在写入更新。那么如果Query_cache非常大，该表的查询结构又比较多，查询语句失效也慢，一个更新或是Insert就会很慢，这样看到的就是Update或是Insert怎么这么慢了。所以在数据库写入量或是更新量也比较大的系统，该参数不适合分配过大。而且在高并发，写入量大的系统，建议把该功能禁掉。

query_cache_limit = 4M  
#指定单个查询能够使用的缓冲区大小,缺省为1M

query_cache_min_res_unit = 2k  
#默认是4KB，设置值大对大数据查询有好处，但如果你的查询都是小数据查询，就容易造成内存碎片和浪费
#查询缓存碎片率 = Qcache_free_blocks / Qcache_total_blocks * 100%
#如果查询缓存碎片率超过20%，可以用FLUSH QUERY CACHE整理缓存碎片，或者试试减小query_cache_min_res_unit，如果你的查询都是小数据量的话。
#查询缓存利用率 = (query_cache_size – Qcache_free_memory) / query_cache_size * 100%
#查询缓存利用率在25%以下的话说明query_cache_size设置的过大，可适当减小;查询缓存利用率在80%以上而且Qcache_lowmem_prunes > 50的话说明query_cache_size可能有点小，要不就是碎片太多。
#查询缓存命中率 = (Qcache_hits – Qcache_inserts) / Qcache_hits * 100%

# skip options
skip-name-resolve # grant 时,必须使用ip不能使用主机名
skip-symbolic-links # 不能使用连接文件
skip-external-locking # 不使用系统锁定,要使用myisamchk,必须关闭服务器
skip-slave-start # 启动mysql,不启动复制

#sysdate-is-now    
# res settings    
back_log = 50 #接受队列，对于没建立tcp连接的请求队列放入缓存中，队列大小为back_log，受限制与OS参数    
max_connections = 1000 #最大并发连接数 ，增大该值需要相应增加允许打开的文件描述符数    
max_connect_errors = 10000 #如果某个用户发起的连接error超过该数值，则该用户的下次连接将被阻塞，直到管理员执行flush hosts ; 命令；防止黑客    
#open_files_limit = 10240   
connect-timeout = 10 #连接超时之前的最大秒数,在Linux平台上，该超时也用作等待服务器首次回应的时间    
wait-timeout = 28800 #等待关闭连接的时间    
interactive-timeout = 28800 #关闭连接之前，允许interactive_timeout（取代了wait_timeout）秒的不活动时间。客户端的会话wait_timeout变量被设为会话interactive_timeout变量的值。    
slave-net-timeout = 600 #从服务器也能够处理网络连接中断。但是，只有从服务器超过slave_net_timeout秒没有从主服务器收到数据才通知网络中断    
net_read_timeout = 30 #从服务器读取信息的超时    
net_write_timeout = 60 #从服务器写入信息的超时    
net_retry_count = 10 #如果某个通信端口的读操作中断了，在放弃前重试多次    
net_buffer_length = 16384 #包消息缓冲区初始化为net_buffer_length字节，但需要时可以增长到max_allowed_packet字节    
max_allowed_packet = 64M #    
#table_cache = 512 #所有线程打开的表的数目。增大该值可以增加mysqld需要的文件描述符的数量    
thread_stack = 192K #每个线程的堆栈大小
thread_cache_size = 20 #线程缓存
thread_concurrency = 8 #同时运行的线程的数据 此处最好为CPU个数两倍。本机配置为CPU的个数
# qcache settings
query_cache_size = 256M #查询缓存大小
query_cache_limit = 2M #不缓存查询大于该值的结果
query_cache_min_res_unit = 2K #查询缓存分配的最小块大小
# default settings
# time zone
default-time-zone = system #服务器时区
character-set-server = utf8 #server级别字符集
default-storage-engine = InnoDB #默认存储
# tmp & heap    
tmp_table_size = 512M #临时表大小，如果超过该值，则结果放到磁盘中
max_heap_table_size = 512M #该变量设置MEMORY (HEAP)表可以增长到的最大空间大小
log-bin = mysql-bin #这些路径相对于datadir
log-bin-index = mysql-bin.index
relayrelay-log = relay-log
relayrelay_log_index = relay-log.index
# warning & error log
log-warnings = 1
log-error = /home/mysql/mysql/log/mysql.err
log_output = FILE #参数log_output指定了慢查询输出的格式，默认为FILE，你可以将它设为TABLE，然后就可以查询mysql架构下的slow_log表了    
# slow query log
slow_query_log = 1
long-query-time = 1 #慢查询时间 超过1秒则为慢查询
slow_query_log_file = /home/mysql/mysql/log/slow.log
#log-queries-not-using-indexes
#log-slow-slave-statements
general_log = 1
general_log_file = /home/mysql/mysql/log/mysql.log
max_binlog_size = 1G
max_relay_log_size = 1G
# if use auto-ex, set to 0
relay-log-purge = 1 #当不用中继日志时，删除他们。这个操作有SQL线程完成    
# max binlog keeps days
expire_logs_days = 30 #超过30天的binlog删除
binlog_cache_size = 1M #session级别
# replication
replicate-wild-ignore-table = mysql.% #复制时忽略数据库及表    
replicate-wild-ignore-table = test.% #复制时忽略数据库及表    
# slave_skip_errors=all
key_buffer_size = 256M #myisam索引buffer,只有key没有data    
sort_buffer_size = 2M #排序buffer大小；线程级别    
read_buffer_size = 2M #以全表扫描(Sequential Scan)方式扫描数据的buffer大小 ；线程级别    
join_buffer_size = 8M # join buffer 大小;线程级别    
read_rnd_buffer_size = 8M #MyISAM以索引扫描(Random Scan)方式扫描数据的buffer大小 ；线程级别    
bulk_insert_buffer_size = 64M #MyISAM 用在块插入优化中的树缓冲区的大小。注释：这是一个per thread的限制    
myisam_sort_buffer_size = 64M #MyISAM 设置恢复表之时使用的缓冲区的尺寸,当在REPAIR TABLE或用CREATE INDEX创建索引或ALTER TABLE过程中排序 MyISAM索引分配的缓冲区    
myisam_max_sort_file_size = 10G #MyISAM 如果临时文件会变得超过索引，不要使用快速排序索引方法来创建一个索引。注释：这个参数以字节的形式给出.重建MyISAM索引(在REPAIR TABLE、ALTER TABLE或LOAD DATA INFILE过程中)时，允许MySQL使用的临时文件的最大空间大小。如果文件的大小超过该值，则使用键值缓存创建索引，要慢得多。该值的单位为字节    
myisam_repair_threads = 1 #如果该值大于1，在Repair by sorting过程中并行创建MyISAM表索引(每个索引在自己的线程内)    
myisam_recover = 64K#允许的GROUP_CONCAT()函数结果的最大长度    
transaction_isolation = REPEATABLE-READ
innodb_file_per_table
#innodb_status_file = 1
#innodb_open_files = 2048
innodb_additional_mem_pool_size = 100M #帧缓存的控制对象需要从此处申请缓存，所以该值与innodb_buffer_pool对应    
innodb_buffer_pool_size = 2G #包括数据页、索引页、插入缓存、锁信息、自适应哈希所以、数据字典信息    
innodb_data_home_dir = /longxibendi/mysql/mysql/var/
#innodb_data_file_path = ibdata1:1G:autoextend
innodb_data_file_path = ibdata1:500M;ibdata2:2210M:autoextend #表空间    
innodb_file_io_threads = 4 #io线程数
innodb_thread_concurrency = 16 #InnoDB试着在InnoDB内保持操作系统线程的数量少于或等于这个参数给出的限制    
innodb_flush_log_at_trx_commit = 1 #每次commit 日志缓存中的数据刷到磁盘中    
innodb_log_buffer_size = 8M #事物日志缓存    
innodb_log_file_size = 500M #事物日志大小    
#innodb_log_file_size =100M
innodb_log_files_in_group = 2 #两组事物日志    
innodb_log_group_home_dir = /longxibendi/mysql/mysql/var/#日志组    
innodb_max_dirty_pages_pct = 90 #innodb主线程刷新缓存池中的数据，使脏数据比例小于90%    
innodb_lock_wait_timeout = 50 #InnoDB事务在被回滚之前可以等待一个锁定的超时秒数。InnoDB在它自己的 锁定表中自动检测事务死锁并且回滚事务。InnoDB用LOCK TABLES语句注意到锁定设置。默认值是50秒    
#innodb_flush_method = O_DSYNC
[mysqldump]
quick
max_allowed_packet = 64M
[mysql]
disable-auto-rehash #允许通过TAB键提示    
default-character-set = utf8
connect-timeout = 3
```

