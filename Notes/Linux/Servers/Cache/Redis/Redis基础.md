[TOC]

# 简介

redis单线程模型，支持主从模式，提高可用性，是一个开源项目，经常用来当一个数据结构服务器；其是内存级别的缓存服务器并可实现持久化功能. 据称一百万的变量存储（字串）占用100M内存空间，单台redis服务器可达到5万并发的能力。

## redis与memcache的对比

### redis的优势

- 支持丰富的操作(String,Lists,Hashs,Sets,Sorted Set,Bitmap,HyperLogLogs)
- 內建replication及cluster
- 就地更新(in-place update)操作
- 支持持久化(磁盘),避免雪崩效应

### memcache优势

- 多线程,善用多核CPU,更少的阻塞操作
- 更少的内存开销
- 更少的内存分配压力
- 可能有更少的内存碎片

## 存储系统

RDBMS,关系型数据库系统,包括Oracle,BD2,PostgreSQL,MySQL,SQL Server...

NoSQL,非关系型数据库系统,包括Cassandra,HBase,Memcached,MongoDB,Redis,...

NewSQL,支持分布式的关系型数据库系统,包括Aerospike,FoundationDB,RethinkDB,...

## redis的持久化方式:

Snapshotting(快照方式) 

异步传输磁盘保存

AOF(Append Only File)

## redis的组件

- redis-server 服务端
- redis-cli 客户端,命令行接口
- redis-benchmark 性能压测工具
- redis-check-dump & redis-check-aof 实现redis检测工具

## redis端口

- 6379/TCP(默认)

# redis安装

## For Windows

1,下载地址：https://github.com/dmajkic/redis/downloads

2,将下载的软件内容cp到自定义盘符安装目录取名redis,如 C:\reids

3,打开一个cmd窗口,使用cd命令切换目录到C:\redis, 运行

redis-server.exe redis.conf ,启动服务端.

4,如果想方便的话,可以把redis的路径加到系统的环境变量里,这样就省得再输路径了,后面的那个redis.conf可以省略,如果省略,会启用默认的.

5,另起一个cmd窗口,来当做客户端,服务端不要关闭,否则就无法访问服务端了.客户端连接: 切换到redis目录下运行

redis-cli.exe -h 127.0.0.1 -p 6379 ,连接成功后就可以操作了.

## For Linux

 

```shell
# epel源yum安装
yum info redis
yum install redis -y

# 源码安装
yum install jemalloc gcc gcc++ -y
wget http://download.redis.io/releases/redis-4.0.9.tar.gz
tar xf redis-redis-4.0.9.tar.gz
cd redis-4.0.9
make || make MALLOC=libc

# make完后 redis-3.0.6目录下的src文件夹下会出现编译后的redis服务程序,包括redis-server,还有用于测试的客户端程序redis-cli等
cd src/
ls
"""
redis-benchmark # 性能测试工具
redis-check-dump # 用于修复有问题的dump.rdb文件
redis-cli # 客户端
redis-server # 服务端
redis-check-aof # 用于修复有问题的AOF文件
redis-sentinel # 用于集群管理
"""

cp redis.conf /etc/redis.conf
./redis-server /etc/redis.conf # 启动服务,使用指定配置文件,不指定则为默认

# 此时redis服务会进入监听状态,另开一个终端,通过redis-cli连接
./redis-cli
"""
> set foo hello
OK
> get foo
"hello"
"""
```

后附redis服务启动脚本

# 配置文件说明

开头部分首先明确了一些单位:

\# 1k => 1000 bytes

\# 1kb => 1024 bytes

\# 1m => 1000000 bytes

\# 1mb => 1024*1024 bytes

\# 1g => 1000000000 bytes

\# 1gb => 1024*1024*1024 bytes

可引入外部配置文件

\# include /path/to/local.conf

\# include /path/to/other.conf

redis的配置文件分为了几大块区域

general(通用)

```shell
daemonize no # 默认是非daemon形式运行,改为yes表示以daemon方式运行

pidfile /var/run/redis.pid  # 当以daemon方式运行时,默认的pid文件路径

port 6379 # 监听端口

bind 127.0.0.1 # 绑定的主机地址,可多个,空格分开

timeout 0 # 客户端空闲超时时长,即客户端多久没有向server端发送请求,server端就关闭连接,0表示永远不关闭

tcp-keepalive 0 # TCP连接保活策略,单位为秒,假如设置为60秒,则server端会每60秒向连接空闲的客户端发起一次ACK请求,以检查客户端是否已经挂掉,对于无响应的客户端则会关闭其连接.如果设置为0，则不会进行保活检测

loglevel notice # 日志记录级别(debug,verbose,notice,warning)

logfile "" # 日志记录位置,如果为空,则输出至标准输出,如果同时redis以daemon方式运行,则输出至/dev/null中

syslog-enabled no # 控制日志打印到syslog中

syslog-ident redis # 指定日志标志

syslog-facility local0 # 指定syslog设备,其值可以是USER,LOCAL0-7,具体可参考syslog服务

database 16 # 设置redis的数据库数量
```

注:

1,端口如果修改为0,则表示不在监听端口,此时可以通过unix socket方式来接受请求:

unixsocket /tmp/redis.sock

unixsocketperm 755

2,这16个数据库的编号是0-15,默认数据库为0号,可以使用SELECT DB_ID命令在连接上指定数据库id

## snapshotting(快照)

主要涉及的是redis的RDB持久化相关的配置

```shell
# save <seconds> <changes>表示让数据保存到磁盘上,即控制RDB快照功能
save "" # 禁用
save 900 1 # 表示每15分钟且至少有1个key改变,就触发一次持久化
save 300 10 # 表示每5分钟且至少有10个key改变,就触发一次持久化
save 60 10000 # 表示每60秒至少有10000个key改变,就触发一次持久化

stop-writes-on-bgsave-error no # 当快照写入失败时,继续接收新的写请求

rdbcompression yes # 快照是否进行压缩存储,如果是的话,redis会采用LZF算法进行压缩;如果你不想消耗CPU来进行压缩的话,可以设置为关闭此功能,但是存储在磁盘上的快照会比较大

rdbchecksum no # 在存储快照后,还可以让redis使用CRC64算法来进行数据校验,但是这样做会增加大约10%的性能消耗,如果你希望获取到最大的性能提升,可以关闭此功能

dbfilename dump.rdb # 设置快照文件名称

dir /var/lib/redis/ # 快照文件存放路径
```

注:

如果用户开启了RDB快照功能,那么在redis持久化数据到磁盘时如果出现失败,默认情况下,redis会停止接受所有的写请求.

这样做的好处在于可以让用户很明确的知道内存中的数据和磁盘上的数据已经存在不一致了.如果redis不顾这种不一致,一意孤行的继续接收写请求,就可能会引起一些灾难性的后果,如果下一次RDB持久化成功，redis会自动恢复接受写请求.

## replication(复制)

主要控制主从同步功能

```shell
slaveof <masterip> <masterport> # 设置本机作为目标的从服务器,一般情况下,我们会建议用户为从redis设置一个不同频率的快照持久化的周期,或者为从redis配置一个不同的服务端口等

masterauth <master-password> # 主redis服务器设置了密码验证的话,则在此填写校验密码

# 当从redis失去了与主redis的连接,或者主从同步正在进行中时,redis该如何处理外部发来的访问请求呢?这里,从redis可以有两种选择:
slave-serve-stale-data yes # 从redis仍会继续响应客户端读写请求
slave-serve-stale-data no  # 从redis会返回"SYNC with master in progress",例外:当客户端发来INFO或SLAVEOF请求时,仍会进行处理

slave-read-only yes # 从redis是否可以接受写请求,默认只读

repl-diskless-sync no

repl-diskless-sync-delay 5

repl-ping-slave-period 10 # 从redis会周期性的向主redis发出PING包,你可以通过repl_ping_slave_period指令来控制其周期,默认是10秒

repl-timeout 60  # 设定主从同步超时时长

repl-disable-tcp-nodelay no  # 我们可以控制在主从同步时是否禁用TCP_NODELAY,如果开启TCP_NODELAY,那么主redis会使用更少的TCP包和更少的带宽来向从redis传输数据.但是这可能会增加一些同步的延迟,大概会达到40毫秒左右.如果你关闭了TCP_NODELAY,那么数据同步的延迟时间会降低,但是会消耗更多的带宽

repl-backlog-size 1mb # 设置队列长度,队列长度(backlog)是主redis中的一个缓冲区,在与从redis断开连接期间,主redis会用这个缓冲区来缓存应该发给从redis的数据.这样的话,当从redis重新连接上之后,就不必重新全量同步数据,只需要同步这部分增量数据即可

repl-backlog-ttl 3600 # 如果主redis等了一段时间之后,还是无法连接到从redis,那么缓冲队列中的数据将被清理掉.我们可以设置主redis要等待的时间长度,如果设置为0,则表示永远不清理,默认是1个小时
 
slave-priority 100 # 设置优先级,当主redis工作不正常时,优先级高的从redis将会升级为主redis,编号越小,优先级越高,0表示永不被选中

min-slaves-to-write 3
min-slaves-max-lag 10
# 假如有大于等于3个从redis的连接延迟大于10秒,那么主redis就不再接受外部的写请求,上述两个配置中有一个被置为0,则这个特性将被关闭.默认情况下min-slaves-to-write为0,而min-slaves-max-lag为10
```

注:

1,将数据直接写入从redis,一般只适用于那些生命周期非常短的数据,因为在主从同步时,这些临时数据就会被清理掉.自从redis2.6版本之后,默认从redis为只读;

只读的从redis并不适合直接暴露给不可信的客户端,为了尽量降低风险,可以使用rename-command指令来将一些可能有破坏力的命令重命名,避免外部直接调用: rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52

2,在主从同步时,可能在这些情况下会有超时发生：

以从redis的角度来看,当有大规模IO传输时

以从redis的角度来看,当数据传输或PING时,主redis超时

以主redis的角度来看,在回复从redis的PING时,从redis超时

用户可以设置上述超时的时限,不过要确保这个时限比repl-ping-slave-period的值要大,否则每次主redis都会认为从redis超时.

3,假如主redis发现有超过M个从redis的连接延时大于N秒,那么主redis就停止接受外来的写请求.这是因为从redis一般会每秒钟都向主redis发出PING,而主redis会记录每一个从redis最近一次发来PING的时间点,所以主redis能够了解每一个从redis的运行情况

## security(安全)

 

```shell
requirepass password  # 连接加密

rename-command CONFIG "" # redis允许我们对redis指令进行更名,比如将一些比较危险的命令改个名字,避免被误执行.比如可以把CONFIG命令改成一个很复杂的名字,这样可以避免外部的调用,同时还可以满足内部调用的需要,如果设置为空串,则表示禁用此命令

```

我们可以要求redis客户端在向redis-server发送请求之前,先进行密码验证.当你的redis-server处于一个不太可信的网络环境中时,相信你会用上这个功能.由于redis性能非常高,所以每秒钟可以完成多达15万次的密码尝试,所以你最好设置一个足够复杂的密码,否则很容易被黑客破解.

## limits(限制)



```shell
maxclients 10000 # 设置redis同时可以与多少个客户端进行连接,默认情况下为10000个客户端

maxmemory <bytes> # 设置redis可以使用的内存量,一旦到达内存使用上限,redis将会试图移除内部数据,移除规则可以通过maxmemory-policy来指定

maxmemory-policy noeviction  # 设定内存中数据移除规则

maxmemory-samples 5  # LRU算法和最小TTL算法都并非是精确的算法,而是估算值,所以你可以设置样本的大小,假如redis默认会检查5个key并选择其中LRU的那个,那么你可以改变这个key样本的数量
```

注:

1,当你无法设置进程文件句柄限制时,redis会设置为当前的文件句柄限制值减去32,因为redis会为自身内部处理逻辑留一些句柄出来,如果达到了此限制,redis则会拒绝新的连接请求,并且向这些连接请求方发出"max number of clients reached"以作回应.

2,如果redis无法根据移除规则来移除内存中的数据,或者我们设置了"不允许移除",那么redis则会针对那些需要申请内存的指令返回错误信息,比如SET、LPUSH等.但是对于无内存申请的指令,仍然会正常响应,比如GET等,如果你的redis是主redis(说明你的redis有从redis),那么在设置内存使用上限时,需要在系统中留出一些内存空间给同步队列缓存,只有在你设置的是"不移除"的情况下,才不用考虑这个因素

3,对于内存移除规则来说,redis提供了多达6种的移除规则,他们是：

volatile-lru:使用LRU算法移除过期集合中的key

allkeys-lru:使用LRU算法移除key

volatile-random:在过期集合中移除随机的key

allkeys-random:移除随机的key

volatile-ttl:移除那些TTL值最小的key,即那些最近才过期的key

noeviction:不进行移除,针对写操作,只是返回错误信息

无论使用上述哪一种移除规则,如果没有合适的key可以移除的话,redis都会针对写请求返回错误信息

## append only mode(追加模式)

默认情况下,redis会异步的将数据持久化到磁盘.这种模式在大部分应用程序中已被验证是很有效的,但是在一些问题发生时,比如断电,则这种机制可能会导致数分钟的写请求丢失,追加文件（Append Only File）是一种更好的保持数据一致性的方式.即使当服务器断电时,也仅会有1秒钟的写请求丢失,当redis进程出现问题且操作系统运行正常时,甚至只会丢失一条写请求,AOF机制和RDB机制可以同时使用,不会有任何冲突

```shell
appendonly no # 是否开启追加模式

appendfilename "appendonly.aof"  # 设置aof文件的名称
```

fsync()调用,用来告诉操作系统立即将缓存的指令写入磁盘,一些操作系统会“立即”进行,而另外一些操作系统则会“尽快”进行

### redis支持appendfsync三种不同的模式： 

no：不调用fsync()。而是让操作系统自行决定sync的时间。这种模式下，redis的性能会最快。

always：在每次写请求后都调用fsync()。这种模式下，redis会相对较慢，但数据最安全。

everysec：每秒钟调用一次fsync().这是性能和安全的折衷,默认为everysec

```shell
appendfsync everysec
```

当fsync方式设置为always或everysec时，如果后台持久化进程需要执行一个很大的磁盘IO操作，那么redis可能会在fsync()调用时卡住。目前尚未修复这个问题，这是因为即使我们在另一个新的线程中去执行fsync()，也会阻塞住同步写调用。

为了缓解这个问题，我们可以使用下面的配置项，这样的话，当BGSAVE或BGWRITEAOF运行时，fsync()在主进程中的调用会被阻止。这意味着当另一路进程正在对AOF文件进行重构时，redis的持久化功能就失效了，就好像我们设置了“appendsync none”一样。如果你的redis有时延问题，那么请将下面的选项设置为yes。否则请保持no，因为这是保证数据完整性的最安全的选择

```shell
no-appendfsync-on-rewrite no
```

我们允许redis自动重写aof,当aof增长到一定规模时,redis会隐式调用BGREWRITEAOF来重写log文件,以缩减文件体积.

redis是这样工作的:redis会记录上次重写时的aof大小.假如redis自启动至今还没有进行过重写,那么启动时aof文件的大小会被作为基准值.这个基准值会和当前的aof大小进行比较.如果当前aof大小超出所设置的增长比例,则会触发重写.另外,你还需要设置一个最小大小,是为了防止在aof很小时就触发重写,如果设置auto-aof-rewrite-percentage为0,则会关闭此重写功能

```shell
auto-aof-rewrite-percentage 10
auto-aof-rewrite-min-size 64mb
```

## LUA scripting(LUA脚本) 

lua脚本的最大运行时间是需要被严格限制的,要注意单位是毫秒,如果此值设置为0或负数,则既不会有报错也不会有时间限制

```shell
lua-time-limit 5000
```

## redis cluster(集群)

## slow log(慢日志)

redis慢日志是指一个系统进行日志查询超过了指定的时长.这个时长不包括IO操作,比如与客户端的交互、发送响应内容等,而仅包括实际执行查询命令的时间.

针对慢日志,你可以设置两个参数,一个是执行时长,单位是微秒,另一个是慢日志的长度.当一个新的命令被写入日志时,最老的一条会从命令日志队列中被移除.

单位是微秒,即1000000表示一秒.负数则会禁用慢日志功能,而0则表示强制记录每一个命令

```shell
slowlog-log-slower-than 10000 # 慢日志执行时长

slowlog-max-len 128 # 慢日志最大长度,可以随便填写数值,没有上限,但要注意它会消耗内存.你可以使用SLOWLOG RESET来重设这个值
```

## latency monitor(监控)

## event notification(事件通知) 

## advanced config(高级设置) 

有关哈希数据结构的一些配置项：

 

```shell
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
```

有关列表数据结构的一些配置项：

 

```shell
list-max-ziplist-entries 512
list-max-ziplist-value 64
```

有关集合数据结构的配置项：

 

```shell
set-max-intset-entries 512
```

有关有序集合数据结构的配置项：

 

```shell
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
```

关于是否需要再哈希的配置项：

 

```shell
activerehashing yes
```

关于客户端输出缓冲的控制项：

 

```shell
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
```

有关频率的配置项：

 

```shell
hz 10
```

有关重写aof的配置项

 

```shell
aof-rewrite-incremental-fsync yes
```

# 命令 

## redis-cli

OPTIONS:

- -h HOST : 连接的主机地址或主机名
- -p PORT :连接的端口
- -s socket : 指定套接字
- -a password : 指定连接密码
- -r <repeat> : 指定命令运行多次

- connection相关的命令

  - auth PASS : 认证
  - ping : 测试服务器是否在线
  - echo "string" : 显示string
  - quit : 退出
  - select # : 挑选指定的名称空间（即数据库）
  - help @connection : 获取与连接相关的命令帮助

- 与服务器端支持的命令

  - help @server : 获取与服务器端相关的命令帮助

  - client getname : 获取当前客户端的连接名

  - client kill IP:PORT : 指定IP:PORT可关闭相关的连接信息

  - client list : 查看客户端的连接信息

    172.16.36.70:6379> client list id=5 addr=172.16.36.70:57606 fd=8 name= age=904 idle=879 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get

  - client setname CONNECTION-NAME: 设定当前连接的名称

  - info : 查看当前服务器的状态信息

  - info memory : 只显示memory段的相关信息

  - config resetstart : 重置info中所统计的数据

  - config set PARAMETER value : 运行时修改设定指定参数的值，只保存在内存中

  - config rewrite : 将修改在内存中的参数值同步到配置文件中

  - config get dir : 查看redis的文件保存目录

  - dbsize : 显示数据库中所有键的数量

  - bgsave : 实现异步将数据集同步到磁盘上

  - lastsave : 用来获取最新一次save执行的时间戳

  - save : 保存数据到磁盘

  - monitor : 实时监控所接收到的请求

  - shutdown : 将所有数据从内存同步到磁盘，并安全关闭

  - shutdown [nosave][save] : 关闭程序并选择是否将数据同步到磁盘上

  - salveof HOST PORT :配置主从，当前节点将变成从节点

  - slowlog : 查看慢查询日志，需要开启慢查询日志功能

  - sync : 复制功能的内建命令

- 与订阅相关的命令

  - help @pubsub : 获取与订阅相关的命令
  - psubscribe : 基于模式进行订阅
  - publish : 向频道发送消息
  - subscribe CHANNEL : 订阅一个频道

# Redis数据类型

Redis支持五种数据类型：strings（字符串），hashes（哈希），lists（列表），sets（集合）及zset(sorted set：有序集合),

可以使用 help + 数据类型 来查看所有该数据类型的操作命令.

关于key设定的建议

1,key不要太长,尽量不要超过1024字节,否则不仅消耗内存,还会降低查找的效率;

2,key也不要太短,否则可读性降低;

3,在一个项目中,key最好使用统一的命名模式,如user:10000:passwd

## String（字符串） 

string是redis最基本的类型,你可以理解成与Memcached一模一样的类型,一个key对应一个value;

string类型是二进制安全的.意思是redis的string可以包含任何数据.比如jpg图片或者序列化的对象;

string类型是Redis最基本的数据类型,一个键最大能存储512MB;

格式

set KEY VAL

支持的命令:

GET

INCR

DECR

EXIST

示例:

 

```shell
127.0.0.1:6379> set num "2"
OK
127.0.0.1:6379> get num
"2"
127.0.0.1:6379> incr num
(integer) 3
127.0.0.1:6379> get num
"3"
```

注意:一个键最大能存储512MB

应用:

访问量统计,每次访问博客和文章使用incr命令进行递增;

将数据以二进制序列化的方式进行存储;

总结:

![img](Redis%E5%9F%BA%E7%A1%80.assets/5b38cc0d-a31a-4e6d-9bc2-e6995823c041.png)

## Lists（列表） 

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）.

注:redis的lists在底层实现上并不是数组,而是链表,比如用LPUSH在10个元素的lists头部插入新元素，和在上千万元素的lists头部插入新元素的速度应该是相同的,链表型lists的元素定位较慢,数组型lists的元素定位快

支持的命令:

LPUSH

RPUSH

LPOP

RPOP

LINDEX

RINDEX

LSET

示例

 

```shell
# 创建列表mylist, 从头部插入元素1
127.0.0.1:6379> lpush mylist "1"
(integer) 1

# 从尾部插入2
127.0.0.1:6379> rpush mylist "2"
(integer) 2

# 再从头部插入0
127.0.0.1:6379> lpush mylist "0"
(integer) 3

# 显示list中索引为0到1的元素(注意是范围)
127.0.0.1:6379> LRANGE mylist 0 1
1) "0"
2) "1"

# 显示list中索引为0到倒数第一个元素
127.0.0.1:6379> LRANGE mylist 0 -1
1) "0"
2) "1"
3) "2"
127.0.0.1:6379> 
```

列表最多可存储 232 - 1 元素 (4294967295, 每个列表可存储40多亿)

应用:

我们可以利用lists来实现一个消息队列,而且可以确保先后顺序,不必像MySQL那样还需要通过ORDER BY来进行排序;

利用lrange还可以很方便的实现分页的功能;

在博客系统中,每片博文的评论也可以存入一个单独的list中;

显示社交网站的新鲜事,热门评论和新闻等;

记录日志;

总结:

![img](Redis%E5%9F%BA%E7%A1%80.assets/0ffd35bf-5619-4156-8d1c-1737a39c9b83.png)

## Sets（集合） 

Redis的Sets是string类型的无序集合.

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1).

支持的命令:

SADD

SINTER

SUNION

SPOP

SISMEMBER

添加一个string元素到集合中,key对应的set集合中，成功返回1,如果元素已经在集合中返回0,key对应的set不存在返回错误。

示例:

 

```shell
127.0.0.1:6379> SADD myset "one"  # 新建集合并添加集合元素
(integer) 1
127.0.0.1:6379> sadd myset "two"  # 添加集合元素
(integer) 1
127.0.0.1:6379> smembers myset  # 显示集合所有元素
1) "two"
2) "one"
127.0.0.1:6379> sismember myset "one"  # 判断某个元素是否存在于指定集合中,1是,0否
(integer) 1
127.0.0.1:6379> sismember myset "three"
(integer) 0

127.0.0.1:6379> sadd youset "1"  # 建立第二个集合
(integer) 1
127.0.0.1:6379> sadd youset "2"
(integer) 1
127.0.0.1:6379> smembers youset
1) "1"
2) "2"
127.0.0.1:6379> sunion myset youset  # 2个集合的并集
1) "1"
2) "one"
3) "two"
4) "2"
127.0.0.1:6379> sadd youset "three"
(integer) 1
127.0.0.1:6379> sadd myset "three"
(integer) 1
127.0.0.1:6379> sinter myset youset  # 2个集合的交集
1) "three"

```

注意：以上实例中 rabitmq 添加了两次，但根据集合内元素的唯一性，第二次插入的元素将被忽略。

集合中最大的成员数为 232 - 1(4294967295, 每个集合可存储40多亿个成员)。

应用:

QQ有一个社交功能叫做“好友标签”，大家可以给你的好友贴标签，比如“大美女”、“土豪”、“欧巴”等等，这时就可以使用redis的集合来实现，把每一个用户的标签都存储在一个集合之中。

总结:

![img](Redis%E5%9F%BA%E7%A1%80.assets/443cc290-9866-4c87-8718-55a7439c9ad9.png)

## Sorted Sets(有序集合) 

和sets一样也是string类型元素的集合,且不允许重复的成员;

每个元素都会关联一个double类型的序号(score),redis正是通过这个序号来为集合中的成员进行排序;

因有序集合的相关操作命令都是以z开头的,所以通常我们习惯称有序集合为zset,zset的成员是唯一的,但序号(score)却可以重复;

支持的命令:

ZADD

ZRANGE

ZCARD

ZRANK

示例:

 

```shell
# 创建一个有序集合myzset,并新增一个元素baidu.com,其scores为1
127.0.0.1:6379> zadd myzset 1 baidu.com
(integer) 1

# 新增360.com,scores为3
127.0.0.1:6379> zadd myzset 3 360.com
(integer) 1

# 新增qq.com,scores为2
127.0.0.1:6379> zadd myzset 2 qq.com
(integer) 1

# 显示所有元素,默认以scores大小排序
127.0.0.1:6379> zrange myzset 0 -1
1) "baidu.com"
2) "qq.com"
3) "360.com"

# 显示所有元素,同时列出其序号
127.0.0.1:6379> zrange myzset 0 -1 withscores
1) "baidu.com"
2) "1"
3) "qq.com"
4) "2"
5) "360.com"
6) "3"
```

应用:

点击量排序;

总结:

![img](Redis%E5%9F%BA%E7%A1%80.assets/7a5275d9-55a2-43e9-b182-c2837ccf2672.png)

## Hash（散列） 

Redis的hashes是一个键值对(key=>value)的集合

hashes存的是字符串和字符串值之间的映射,比如一个用户要存储其全名、姓氏、年龄等等,就很适合使用哈希,特别适合用于存储对象.

散列类型适合存储对象.可以采用这样的命名方式:对象类别和ID构成键名,使用字段表示对象的属性,而字段值则存储属性值.

如下图:存储ID为2的汽车对象.

![img](Redis%E5%9F%BA%E7%A1%80.assets/02d33655-3ca5-40f8-bf2f-b60714fcc86b.jpg)

支持的命令:

HSET

HMSET

HSETNX

HGET

HKEYS

HVALS

HDEL

示例:

 

```shell
# 创建一个hash集合myhash,内容如下
127.0.0.1:6379> hmset myhash name flight age 18 passwd 123.com
OK

# 显示所有元素
127.0.0.1:6379> hgetall myhash
1) "name"
2) "flight"
3) "age"
4) "18"
5) "passwd"
6) "123.com"

# 根据key显示value
127.0.0.1:6379> hget myhash name
"flight"
127.0.0.1:6379> hget myhash passwd
"123.com"

# 根据key修改value
127.0.0.1:6379> hset myhash passwd 456.cn
(integer) 0
127.0.0.1:6379> hget myhash passwd
"456.cn"

```

更多:https://redis.io/commands#hash

应用:

文章内容存储,如下所示:

![img](Redis%E5%9F%BA%E7%A1%80.assets/7b28ff47-c046-4f71-8e31-06da62d95c69.jpg)

总结:

![img](Redis%E5%9F%BA%E7%A1%80.assets/c6845d4a-6c8d-43e7-9f69-9cfe23972ee8.png)

# 安装时遇到的报错:

 

```shell
zmalloc.h:50:31: error: jemalloc/jemalloc.h: No such file or directory
zmalloc.h:55:2: error: #error "Newer version of jemalloc required"
make[1]: *** [adlist.o] Error 1
make[1]: Leaving directory `/data0/src/redis-2.6.2/src'
make: *** [all] Error 2
```

解决办法:

在README 有这个一段话。

 

```shell
Allocator  
---------  
Selecting a non-default memory allocator when building Redis is done by setting  
the `MALLOC` environment variable. Redis is compiled and linked against libc  
malloc by default, with the exception of jemalloc being the default on Linux  
systems. This default was picked because jemalloc has proven to have fewer  
fragmentation problems than libc malloc.  
To force compiling against libc malloc, use:  
% make MALLOC=libc  
To compile against jemalloc on Mac OS X systems, use:  
% make MALLOC=jemalloc
```

没有 jemalloc 而只有 libc 当然 make 出错,所以加这么一个参数:

 

```shell
make MALLOC=libc
```

# 其他

keys pattern

pattern支持golb风格通配符:

? 单个字符

\* 任意个字符

[] 分组,括号内任一字符,"-"表示范围

\x 特殊符号需要转义

判断一个键是否存在

exists key

删除键,可删除多个,返回值是删除的键的个数

del key [key ...]

获得键值的数据类型

type key

# redis服务启动脚本

 

```shell
#!/bin/sh
# chkconfig: 2345 80 90
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

REDISPORT=6379
EXEC=/usr/local/redis-${REDISPORT}/src/redis-server
CLIEXEC=/usr/local/redis-${REDISPORT}/src/redis-cli

PIDFILE=/usr/local/redis-${REDISPORT}/run/redis_${REDISPORT}.pid
CONF="/usr/local/redis-${REDISPORT}/redis.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```