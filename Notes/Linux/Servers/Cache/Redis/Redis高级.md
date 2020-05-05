[TOC]

redis的高级用法

实现认证方法

1,编辑redis.conf,重启服务

requirepass PASSWORD  # 设置认证密码

2,REDIS-CLI

auth password  # 进入客户端命令行,验证密码

清空数据库

flushdb: 清空当前库

flushall: 清空所有库

事务功能

通过multi,exec,watch等命令来实现事务功能

redis的事务是指将多个命令打包,多个命令按顺序执行,并一次执行完成,并将执行结果一次性全部返回给客户端.

redis事务不支持回滚操作,在事务中应避免发生错误(如命令写错等)事务了也将会执行失败.

 

multi: 启动一个事务

exec: 执行事务,一次性将事务中的所有操作执行完成后,返回给客户端

watch: 乐观锁机制,在EXEC命令执行之前,用于监视指定数据键,如果监视中的某任意键数据被修改,服务器拒绝执行事务

示例:

 

```
127.0.0.1:6379> multi # 进入事务
OK
127.0.0.1:6379> set ip 192.168.1.1  # 进入队列
QUEUED
127.0.0.1:6379> get ip  # 进入队列
QUEUED
127.0.0.1:6379> set port 8080  # 进入队列
QUEUED
127.0.0.1:6379> get port  # 进入队列
QUEUED
127.0.0.1:6379> exec  # 最后一次性返回结果
1) OK
2) "192.168.1.1"
3) OK
4) "8080"
# 继续
127.0.0.1:6379> get ip  # 前面定义好的
"192.168.1.1"
127.0.0.1:6379> watch ip # 监控ip这个键
OK
127.0.0.1:6379> multi  # 开启事务
OK
127.0.0.1:6379> set ip 10.0.0.1  # 设置ip为一个新键
QUEUED
127.0.0.1:6379> get ip # 显示ip的值
QUEUED
# 这里注意,执行exec之前,我们再另外一个终端上连接redis
127.0.0.1:6379> get ip  # 此时显示的ip的值还是最初的那个
"192.168.1.1"
127.0.0.1:6379> set ip 172.16.1.100  # 这里设置一个新的值
OK
127.0.0.1:6379> get ip  # 验证设置成功
"172.16.1.100"
# 然后回到开始的终端上,执行exec,查看事务执行结果
127.0.0.1:6379> exec
(nil)
```

publish/subscribe

发布和订阅功能被广泛用于构建即时通信应用，比如：网络聊天室和实时广播、实时提醒等.

订阅发布功能就能帮你很轻松地实现通知、监控程序.

订阅相关的命令:

- subsciribe CHANNEL_NAME : 订阅一个或多个频道(队列)
- publish CHANNEL_NAME : 向频道发送一个消息
- unsubscribe CHANNEL_NAME : 退订频道
- psubscribe CHANNEL_NAME_PATTERN : 基于正则表达式模式定义多个频道

示例:

 

```
# 开启一个频道news
> SUBSCRIBE news
"""
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "news"
3) (integer) 1
"""
# 向这个频道news发送一条消息
> PUBLISH news hello,world
# 查看news频道信息
"""
1) "subscribe"
2) "news"
3) (integer) 1
1) "message"
2) "news"
3) "hello,world"  # 这里显示的是刚刚发过的信息
"""
```

模式订阅频道的建立

 

```
# 开启一个频道news
> PSUBSCRIBE news.i[to]
"""
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "news.i[to]"
3) (integer) 1
"""
# 向news.io频道发送一条消息
> PUBLISH news.io hello,io
# 向news.it频道发送一条消息
> PUBLISH news.it hello,it
# 查看各频道收到的信息
"""
1) "pmessage"
2) "news.i[to]"
3) "news.io"
4) "hello,io"
1) "pmessage"
2) "news.i[to]"
3) "news.it"
4) "hello,it"
"""
```

持久化功能

redis是内存数据库,它把数据存储在内存中,这样在加快读取速度的同时也对数据安全性产生了新的问题,即当redis所在服务器发生宕机后,redis数据库里的所有数据将会全部丢失.

为了解决这个问题,redis提供了持久化功能——RDB和AOF.通俗的讲就是将内存中的数据写入硬盘中.

典型的需要持久化数据的场景如下:

将Redis作为数据库使用;

将Redis作为缓存服务器使用,但是缓存miss后会对性能造成很大影响,所有缓存同时失效时会造成服务雪崩,无法响应;

注:配置文件修改需要重启redis服务,我们还可以在命令行里进行配置,即时生效,服务器重启后需重新配置.

持久化的两种方式

1,RDB（redisDB）

工作原理

save指令指定的时间,redis主进程将fork一个子进程,负责内存中的内容快照并保存到磁盘中.Linux系统有写时复制机制,父进程与子进程会共享相同的物理页面,当父进程处理写请求时,操作系统为写的数据创建一个副本,因此子进程保存的数据一定是与时间点一致的数据.当子进程将快照写入临时文件后,会使用临时文件替换旧的文件,然后子进程完成退出.

RDB为snapshot(快照)存储机制,其也是redis默认的存储机制,按照事先定制的策略,周期性地将数据从内存中读取出来保存至磁盘,数据文件默认为dump.rdb,如果在SAVE周期之前停电了,会造成部分数据丢失.

 

RDB数据保存(持久化)方式

save : 阻塞式的RDB持久化,当执行这个命令时,redis的主进程把内存里的数据库状态写入到RDB文件(即上面的dump.rdb)中,保存时会阻塞所有客户端请求;每次将完整数据写至dump.rdb文件中,都会带来大量的IO压力.

bgsave : 异步保存机制,属于非阻塞式的持久化,它会创建一个子进程专门去把内存中的数据库状态写入RDB文件里,同时主进程还可以处理来自客户端的命令请求,但子进程基本是复制的父进程,这等于两个相同大小的redis进程在系统上运行,会造成内存使用率的大幅增加.

配置参数(redis.conf)

 

```
stop-writes-on-bgsave-error yes # 在基于快照备份时,一旦发生错误,是否停止写操作
rdbcompression yes # rdb文件是否压缩来节约空间
rdbchecksum yes # 是否对rdb的镜像文件做校验码检测,当redis启动时会根据事先保存好的校验码进行对比,保证数据的完整性
dbfilename dump.rdb # 保存的文件名
dir /data/dbs/redis/6380 # dump.rdb文件保存目录
save "" # 空表示关闭rdb功能
```

2,AOF(append only file)

AOF是通过保存对redis服务端的写命令(如set、sadd、rpush等)来记录数据库状态的，即保存的是一些操作命令.

工作原理

redis以顺序IO的方式附加在文件的尾部,将每一次的写命令操作都通过write函数追加到文件后面,其是比RDB更好的持久化方案,但文件会变得越来越大.当redis重启时,可通过执行文件中的命令在内存中重建数据库

AOF数据保存(持久化)方式

bgrewriteaof : AOF文件重写,它不会读取正在使用的AOF文件,而是通过将内存中的数据以命令的方式保存至临时文件中,完成之后替换原来的AOF文件,这样可以减少AOF的大小



AOF重写过程

AOF的持久化是通过命令追加,文件写入和文件同步三个步骤实现的:

redis主进程通过fork机制创建子线程;
子进程根据redis内存中现有的数据通过重建命令创建数据库于临时文件中;
父进程继续接收客户请求,并会把这些请求中的写操作继续追加到原来的AOF文件中,额外地,也将新的写请求放置于一个缓冲队列中;
子进程重写完成,会通知父进程,父进程把缓冲中的队列命令写到临时文件中;
父进程用临时文件替换老的AOF文件;

当reids开启AOF后,服务端每执行一次写操作(如set、sadd、rpush)就会把该条命令追加到一个单独的AOF缓冲区的末尾,这就是命令追加;然后把AOF缓冲区的内容写入AOF文件里.

看上去第二步就已经完成AOF持久化了那第三步是干什么的呢？这就需要从系统的文件写入机制说起：

一般我们现在所使用的操作系统，为了提高文件的写入效率，都会有一个写入策略，即当你往硬盘写入数据时，操作系统不是实时的将数据写入硬盘，而是先把数据暂时的保存在一个内存缓冲区里，等到这个内存缓冲区的空间被填满或者是超过了设定的时限后才会真正的把缓冲区内的数据写入硬盘中。也就是说当redis进行到第二步文件写入的时候，从用户的角度看是已经把AOF缓冲区里的数据写入到AOF文件了，但对系统而言只不过是把AOF缓冲区的内容放到了另一个内存缓冲区里而已，之后redis还需要进行文件同步把该内存缓冲区里的数据真正写入硬盘上才算是完成了一次持久化。

而何时进行文件同步则是根据配置的appendfsync来进行：

appendfsync有三个策略选项:

always:每次收到写命令,立即写入磁盘.服务器会在每执行一个事件就把AOF缓冲区的内容强制性的写入硬盘上的AOF文件里,可以看成你每执行一个redis写入命令就往AOF文件里记录这条命令,这保证了数据持久化的完整性,但效率是最慢的,却也是最安全的;
everysec:每秒钟写一次[推荐操作].服务端每执行一次写操作（如set、sadd、rpush）也会把该条命令追加到一个单独的AOF缓冲区的末尾，并将AOF缓冲区写入AOF文件，然后每隔一秒才会进行一次文件同步把内存缓冲区里的AOF缓存数据真正写入AOF文件里，这个模式兼顾了效率的同时也保证了数据的完整性，即使在服务器宕机也只会丢失一秒内对redis数据库做的修改;
no:redis数据库里的数据就算丢失你也可以接受，它也会把每条写命令追加到AOF缓冲区的末尾，然后写入文件，但什么时候进行文件同步真正把数据写入AOF文件里则由系统自身决定，即当内存缓冲区的空间被填满或者是超过了设定的时限后系统自动同步。这种模式下效率是最快的，但对数据来说也是最不安全的，如果redis里的数据都是从后台数据库如mysql中取出来的，属于随时可以找回或者不重要的数据，那么可以考虑设置成这种模式

相比RDB每次持久化都会内存翻倍，AOF持久化除了在第一次启用时会新开一个子进程创建AOF文件会大幅度消耗内存外，之后的每次持久化对内存使用都很小。但AOF也有一个不可忽视的问题：AOF文件过大。你对redis数据库的每一次写操作都会让AOF文件里增加一条数据，久而久之这个文件会形成一个庞然大物。还好的是redis提出了AOF重写的机制，即我们上面配置的auto-aof-rewrite-percentage和auto-aof-rewrite-min-size。AOF重写机制这里暂不细述，之后本人会另开博文对此解释，有兴趣的同学可以看看。我们只要知道AOF重写既是重新创建一个精简化的AOF文件，里面去掉了多余的冗余命令，并对原AOF文件进行覆盖。这保证了AOF文件大小处于让人可以接受的地步。而上面的auto-aof-rewrite-percentage和auto-aof-rewrite-min-size配置触发AOF重写的条件。       

Redis 会记录上次重写后AOF文件的文件大小，而当前AOF文件大小跟上次重写后AOF文件大小的百分比超过auto-aof-rewrite-percentage设置的值，同时当前AOF文件大小也超过auto-aof-rewrite-min-size设置的最小值，则会触发AOF文件重写。以上面的配置为例，当现在的AOF文件大于64mb同时也大于上次重写AOF后的文件大小，则该文件就会被AOF重写

配置参数

(redis.conf)

 

```
dir "/data/dbs/redis/6381" #AOF文件存放目录
appendonly yes # 开启AOF持久化,默认关闭 
appendfilename "appendonly.aof" # AOF文件名称(默认)
appendfsync no # AOF持久化策略
no-appendfsync-no-rewrite no # 在重写的过程中是否调用fsync
auto-aof-rewrite-percentage 100 # 触发AOF文件重写的条件(默认),在aof文件已经是上次重写时的2倍大小,将自动启动重写操作
auto-aof-rewrite-min-size 64mb # 触发AOF文件重写的条件,当文件达到64MB才执行重写操作
```

注:

RDB与AOF同时启用时:

1,bgsave和bgrewriteaof不会同时执行;

2,在redis服务器启动用于恢复数据时,会优先使用AOF.

主从复制

- 1、redis的主从复制特点

  

  - 一个master可以有多个slave
  - 支持链式复制
  - master以非阻塞方式同步数据至slave,意味着可同时与多个slave同步

- 2、复制的工作原理

  

  - 1、主库会自己基于ping check机制来检查从库是否在线
  - 2、如果从库在线就同步数据文件至从服务器端
  - 3、从服务器也可以发送请求同步的请求，主库将启动一个线程，把内存中的数据同步给从库，从库将数据保存至文件中
  - 4、从库再把文件装载到内存中，从而完成复制功能

- 3、主从相关配置（/etc/redis.conf）

  

  - slave-serve-stale-data yes : 当主服务器连接不上了，从服务器是否可以使用过期数据响应
  - repl-diskless-sync no : 是否基于diskless机制同步
  - slave-priority 100 : 指定从服务器优先级
  - min-slave-to-write 3 : 如果从节点小于三个，主服务器将拒绝写操作
  - min-slave-max-lag 10 : 从服务器不能晚于主服务器10秒钟，是否将停止复制

注:

如果master使用了requirepass开启了认证功能，从服务器要使用masterauth <password>来连接，使用指定的密码进行认证

示例:同一台主机按照不同端口

master: 172.16.1.101:6379

slave1: 172.16.1.101:6380

slave2: 172.16.1.101:6381

 

```
# 配置文件目录
mkdir /etc/redis/ -pv
cp redis-4.0.9/redis.conf /etc/redis/
cd /etc/redis
cp redis.conf redis.conf1
cp redis.conf redis.conf2
# 数据目录
mkdir -pv /redis/db{1,2,3}
chown -R redis.redis /redis/db*
# 修改各配置文件
vim /etc/redis/redis.conf  # 主
"""
bind 0.0.0.0
daemonize yes
dir /redis/db1
pidfile /var/run/redis.pid
logfile /var/log/redis/redis.log
port 6379
# 其他参数默认
"""
vim /etc/redis/redis1.conf  # 从1
"""
bind 0.0.0.0
daemonize yes
dir /redis/db2
pidfile /var/run/redis1.pid
logfile /var/log/redis/redis1.log
port 6380
slaveof 172.16.1.101 6379
# 其他参数默认
"""
vim /etc/redis/redis1.conf  # 从2
"""
bind 0.0.0.0
daemonize yes
dir /redis/db3
pidfile /var/run/redis2.pid
logfile /var/log/redis/redis2.log
port 6381
slaveof 172.16.1.101 6379  # slaveof 172.16.1.101 6379 也可通过redis-cli命令行的slaveof命令修改
# 其他参数默认
"""
# 启动服务
killall redis-server
/usr/local/redis-6379/src/redis-server /etc/redis/redis.conf
/usr/local/redis-6379/src/redis-server /etc/redis/redis1.conf
/usr/local/redis-6379/src/redis-server /etc/redis/redis2.conf
ss -tnl
# 登录测试
# 主
/usr/local/redis-6379/src/redis-cli -h 172.16.1.101 -p 6379
> info replication
> keys *
> set test hello,world
# 从1
/usr/local/redis-6379/src/redis-cli -p 6380
> keys *
> get test
> set test1 hey,girl
# 从2
/usr/local/redis-6379/src/redis-cli -p 6380
> keys *  # 发现从1上设定的数据是看不到的,只有主才能看到
```

高可用

由于redis目前只支持主从复制备份(不支持主主复制),当主redis挂了,从redis只能提供读服务,无法提供写服务.所以,还得想办法,当主redis挂了,让从redis升级成为主redis. 这就需要自动故障转移,redis的sentinel带有这个功能,当一个主redis不能提供服务时,redis的sentinel可以将一个从redis升级为主redis,并对其他从redis进行配置,让它们使用新的主redis进行复制备份.
sentinel机制

工作原理

找一台专用的监控主机,即能提供监控又可以提供配置功能,如果发现master离线了,sentinel服务器会在从节点选择一个作为新的主节点.

为了不误判,setinel至少有奇数个节点,同时监控;如果主节点不在线,多个setinel会协调一个新的主节点,以免发生误判,所有setinel每秒一次向所有服务器发送ping请求,判断节点是否在线.

setinel是一个分布式系统,使用'流言'协议和'投票'协议来决定故障迁移,可以监控多组redis实例,类似heartbeat.



注意:当主节点下线后在重新上线,不会再成为主节点.

s

entinel的功用

用于管理多个redis服务,实现HA

监控
通知
自动故障转移

工作过程

启用sentinel时,首先服务器做自身初始化,运行redis-server中专用于sentinel功能中的代码;
初始化sentinel状态,根据给定的配置文件,初始化监控的master服务列表(可以根据master的配置,获取从服务器节点);
创建连向Master的连接;



sentinel下线机制

主观下线: 一个sentinel实例判断出某节点下线
客观下线: 多个sentinel节点协商后判断出某节点下线



setinel程序

redis-sentinel /path/to/redis-sentinel.conf
redis-server /path/to/sentinel --sentinel



端口
26379/TCP



专用的配置文件
redis-sentinel.conf

配置参数

prot 26379
dir /tmp
sentinel monitor mymaster 127.0.0.1 6379 2
mymaster: sentinel要监控的实例名称,此名称可以随意取;

127.0.0.1: 主节点的IP地址;

6379: 主节点的端口;

2: sentinel节点的票数,此值要大于sentinel节点数量的半数.



sentinel down-after-milliseconds mymaster 30000 : 判断主节点不在线的默认超时时长，默认30秒(其单位是毫秒)
sentinel parallel-syncs mymaster 1 : 故障转移时最多能有多少个从服务器向主服务器发起同步请求
sentinel failover-timeout mymaster 180000 : 提升主节点的超时时长,表示提升新的主节点在3分钟内未完成，操作将失败

sentinel专用命令

sentinel masters : 列出所有主服务器
sentinel slaves <master name> : 获取所有当前redis实例中的从节点信息



sentinel get-master-addr-by-name <master name> :直接获取当前redis实例主节点的IP地址及端口
例如:sentinel get-master-addr-by-name mymaster



sentinel reset : 重置服务器所有状态
sentinel failover <master name> : 手动实现故障转移

示例:

接上面的例子继续操作

 

```
cp redis-4.0.9/sentinel.conf /etc/redis/
cd /etc/redis/
vim sentinel.conf
"""
sentinel monitor mymaster 172.16.1.101 6379 1
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
"""
redis-sentinel /etc/redis/sentinel.conf
redis-cli -h 172.16.1.101 -p 26379
> info
> sentinel masters
```

Clustering

在redis3.0版本中引入clustering功能,是去中心化的分布式数据库,通过分片机制进行数据分布,clustering内的每个节点仅有数据库一部分数据,每个节点都有全局元数据,通过查找元数据,即可查询到数据存放在哪台服务器.

分布式解决方案

Twemproxy(Twitter研发)

代理分片机制



优点: 

非常稳定,企业级方案;



缺点:

单点故障;

需要依赖第三方软件,如Keeplived;

无法平滑地横向扩展;

没有后台界面;

代理分片机制引入更多的来回次数并提高延迟;

单核模式,无法充分利用多核,除非多实例;
Twitter官方肉串不再继续使用;



Codis(豌豆荚研发)

代理分片机制

2014年11月开源
基于GO以及C语言开发
分布式redis推荐使用

优点:

非常稳定,企业级方案;
数据自动平衡;
高性能;
简单的测试显示较Twemproxy快一倍;
善用多核CPU;

简单(

没有paxos类的协调机制,没有主从复制);
有后台界面;

缺点:

代理分片机制引入更多的来回次数并提高延迟;

需要第三方软件支持协调机制(

目前支持zookeeper及Etcd);
不支持主从复制,需要另外实现;

Codis采用proxy方案,所以必然会带来单机性能的损失(

经测试,在不开Pipeline的情况下,大概会损失40%左右的性能)



Redis Cluster(官方)

官方实现
需要3.0或更高版本

成本高

优点:

无中心的P2P Gossip分散式模式;
更少的来回次数并降低延迟;
自动于多个redis节点进行分片;
不需要第三方软件支持协调机制;

缺点:

依赖于redis 3.0或更高版本;

需要时间验证其稳定性;

没有后台界面;
需要智能客户端;
redis客户端必须支持redis cluster架构;
较Codis有更多的维护升级版本;

Cerberus(芒果TV研发)

优点:

数据自动平衡;

本身实现了Redis的Smart Client;

支持读写分离;

缺点:

依赖Redis3.0或更高版本;

代理分片机制引入更多的来回次数并增大延迟;

需要时间验证其稳定性;

没有后台界面;