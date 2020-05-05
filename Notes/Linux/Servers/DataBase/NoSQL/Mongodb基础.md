[TOC]

# MongoDB简介

MongoDB是一个基于分布式文件存储的数据库，属于文档型的，虽然也是NoSQL数据库的一种，但是与redis、memcached等数据库有些区别。MongoDB由C++语言编写。旨在为WEB应用提供可扩展的高性能数据存储解决方案。

MongoDB是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。他支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型。Mongo最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

2007年10月，MongoDB由10gen团队所发展。2009年2月首度推出。

MongoDB 可以将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档、数组及文档数组。

# 主要特点

1,MongoDB是一个面向文档存储的数据库,操作起来比较简单和容易;

2,你可以在MongoDB记录中设置任何属性的索引(如:FirstName="Sameer",Address="8 Gandhi Road")来实现更快的排序;

3,你可以通过本地或者网络创建数据镜像,这使得MongoDB有更强的扩展性;

4,如果负载的增加(需要更多的存储空间和更强的处理能力),它可以分布在计算机网络中的其他节点上这就是所谓的分片;

5,Mongo支持丰富的查询表达式,查询指令使用JSON形式的标记,可轻易查询文档中内嵌的对象及数组;

6,MongoDb使用update()命令可以实现替换完成的文档(数据)或者一些指定的数据字段;

7,Mongodb中的Map/reduce主要是用来对数据进行批量处理和聚合操作;

8,Map函数调用emit(key,value)遍历集合中所有的记录,将key与value传给Reduce函数进行处理;

9,Map函数和Reduce函数是使用Javascript编写的,并可以通过db.runCommand或mapreduce命令来执行MapReduce操作;

10,GridFS是MongoDB中的一个内置功能,可以用于存放大量小文件;

11,MongoDB允许在服务端执行脚本,可以用Javascript编写某个函数,直接在服务端执行,也可以把函数的定义存储在服务端,下次直接调用即可;

12,MongoDB支持各种编程语言:RUBY,PYTHON,JAVA,C++,PHP,C#等多种语言;

13,MongoDB安装简单;

# 历史

2007年10月,MongoDB由10gen团队所发展,2009年2月首度推出;

2012年05月23日,MongoDB2.1开发分支发布了!该版本采用全新架构,包含诸多增强;

2012年06月06日,MongoDB2.0.6发布,分布式文档数据库;

2013年04月23日,MongoDB2.4.3发布,此版本包括了一些性能优化,功能增强以及bug修复;

2013年08月20日,MongoDB2.4.6发布;

2013年11月01日,MongoDB2.4.8发布;

# 下载

MonggoDB支持以下平台:

OS X 32-bit

OS X 64-bit

Linux 32-bit

Linux 64-bit

Windows 32-bit

Windows 64-bit

Solaris i86pc

Solaris 64

MongoDB官网地址:

https://www.mongodb.com/

阿里镜像源:

https://mirrors.aliyun.com/mongodb/

# 语言支持

MongoDB有官方的驱动如下:

C

C++

C# / .NET

Erlang

Haskell

Java

JavaScript

Lisp

node.JS

Perl

PHP

Python

Ruby

Scala

# 工具

有几种可用于MongoDB的管理工具

## 监控

MongoDB提供了网络和系统监控工具Munin,它作为一个插件应用于MongoDB中;

Gangila是MongoDB高性能的系统监视的工具,它作为一个插件应用于MongoDB中;

基于图形界面的开源工具Cacti,用于查看CPU负载,网络带宽利用率,它也提供了一个应用于监控MongoDB的插件;

## GUI

Fang of Mongo: 网页式,由Django和jQuery所构成

Futon4Mongo: 一个CouchDB Futon web的mongodb山寨版

Mongo3: Ruby写成

MongoHub: 适用于OSX的应用程序

Opricot: 一个基于浏览器的MongoDB控制台,由PHP撰写而成

Database Master: Windows的mongodb管理工具

RockMongo: 最好的PHP语言的MongoDB管理工具,轻量级,支持多国语言

## 应用

下面列举一些公司MongoDB的实际应用:

1, Craiglist上使用MongoDB的存档数十亿条记录;

2, FourSquare,基于位置的社交网站,在Amazon EC2的服务器上使用MongoDB分享数据;

3, Shutterfly,以互联网为基础的社会和个人出版服务,使用MongoDB的各种持久性数据存储的要求;

4, bit.ly,一个基于Web的网址缩短服务,使用MongoDB的存储自己的数据;

5, spike.com,一个MTV网络的联营公司,spike.com使用MongoDB的;

6, Intuit公司,一个为小企业和个人的软件和服务提供商,为小型企业使用MongoDB的跟踪用户的数据;

7, sourceforge.net,资源网站查找,创建和发布开源软件免费,使用MongoDB的后端存储;

8, etsy.com,一个购买和出售手工制作物品网站,使用MongoDB;

9, 纽约时报,领先的在线新闻门户网站之一,使用MongoDB;

10, CERN,著名的粒子物理研究所,欧洲核子研究中心大型强子对撞机的数据使用MongoDB;

## Redis,Memcache,MongoDB各自特点与区别

## Redis优点

1.支持多种数据结构，如 string（字符串）、 list(双向链表)、dict(hash表)、set(集合）、zset(排序set)、hyperloglog（基数估算）

2.支持持久化操作，可以进行aof及rdb数据持久化到磁盘，从而进行数据备份或数据恢复等操作，较好的防止数据丢失　　的手段。

3.支持通过Replication进行数据复制，通过master-slave机制，可以实时进行数据的同步复制，支持多级复制和增量复制，master-slave机制是Redis进行HA的重要手段。

4.单线程请求，所有命令串行执行，并发情况下不需要考虑数据一致性问题。

5.支持pub/sub消息订阅机制，可以用来进行消息订阅与通知。

6.支持简单的事务需求，但业界使用场景很少，并不成熟。

## Redis缺点

1.Redis只能使用单线程，性能受限于CPU性能，故单实例CPU最高才可能达到5-6wQPS每秒（取决于数据结构，数据大小以及服务器硬件性能，日常环境中QPS高峰大约在1-2w左右）。

2.支持简单的事务需求，但业界使用场景很少，并不成熟，既是优点也是缺点。

3.Redis在string类型上会消耗较多内存，可以使用dict（hash表）压缩存储以降低内存耗用。

## Memcache优点

1.Memcached可以利用多核优势，单实例吞吐量极高，可以达到几十万QPS（取决于key、value的字节大小以及服务器硬件性能，日常环境中QPS高峰大约在4-6w左右）。适用于最大程度扛量。

2.支持直接配置为session handle。

## Memcache缺点

1只支持简单的key/value数据结构，不像Redis可以支持丰富的数据类型。

2.无法进行持久化，数据不能备份，只能用于缓存使用，且重启后数据全部丢失。

3.无法进行数据同步，不能将MC中的数据迁移到其他MC实例中。

4.Memcached内存分配采用Slab Allocation机制管理内存，value大小分布差异较大时会造成内存利用率降低，并引发低利用率时依然出现踢出等问题。需要用户注重value设计。

## MongoDB优点

1.更高的写负载，MongoDB拥有更高的插入速度。

2.处理很大的规模的单表，当数据表太大的时候可以很容易的分割表。

3.高可用性，设置M-S不仅方便而且很快，MongoDB还可以快速、安全及自动化的实现节点（数据中心）故障转移。

4.快速的查询，MongoDB支持二维空间索引，比如管道，因此可以快速及精确的从指定位置获取数据。MongoDB在启动后会将数据库中的数据以文件映射的方式加载到内存中。如果内存资源相当丰富的话，这将极大地提高数据库的查询速度。

5.非结构化数据的爆发增长，增加列在有些情况下可能锁定整个数据库，或者增加负载从而导致性能下降，由于MongoDB的弱数据结构模式，添加1个新字段不会对旧表格有任何影响，整个过程会非常快速。

## MongoDB缺点

1.不支持事务。

2.MongoDB占用空间过大 。

3.MongoDB没有成熟的维护工具。

## 区别

1.性能

三者的性能都比较高，总的来讲：Memcache和Redis差不多，要高于MongoDB。

2.便利性

memcache数据结构单一。

redis丰富一些，数据操作方面，redis更好一些，较少的网络IO次数。

mongodb支持丰富的数据表达，索引，最类似关系型数据库，支持的查询语言非常丰富。

3,存储空间

redis在2.0版本后增加了自己的VM特性，突破物理内存的限制；可以对key value设置过期时间（类似memcache）。

memcache可以修改最大可用内存,采用LRU算法。

mongoDB适合大数据量的存储，依赖操作系统VM做内存管理，吃内存也比较厉害，服务不要和别的服务在一起。

4.可用性

redis，依赖客户端来实现分布式读写；主从复制时，每次从节点重新连接主节点都要依赖整个快照,无增量复制，因性能和效率问题，所以单点问题比较复杂；不支持自动sharding,需要依赖程序设定一致hash 机制。一种替代方案是，不用redis本身的复制机制，采用自己做主动复制（多份存储），或者改成增量复制的方式（需要自己实现），一致性问题和性能的权衡。

Memcache本身没有数据冗余机制，也没必要；对于故障预防，采用依赖成熟的hash或者环状的算法，解决单点故障引起的抖动问题。

mongoDB支持master-slave,replicaset（内部采用paxos选举算法，自动故障恢复）,auto sharding机制，对客户端屏蔽了故障转移和切分机制。

5.可靠性

redis支持（快照、AOF）：依赖快照进行持久化，aof增强了可靠性的同时，对性能有所影响。

memcache不支持，通常用在做缓存,提升性能。

MongoDB从1.8版本开始采用binlog方式支持持久化的可靠性。

6.一致性

Memcache 在并发场景下，用cas保证一致性。

redis事务支持比较弱，只能保证事务中的每个操作连续执行。

mongoDB不支持事务。

7.数据分析

mongoDB内置了数据分析的功能(mapreduce),其他两者不支持。

8.应用场景

redis：数据量较小的更性能操作和运算上。

memcache：用于在动态系统中减少数据库负载，提升性能;做缓存，提高性能（适合读多写少，对于数据量比较大，可以采用sharding）。

MongoDB:主要解决海量数据的访问效率问题。

# MongoDB安装

官网安装文档:

https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/

说明:

MongoDB provides officially supported packages in their own repository. This repository contains the following packages:

[MongoDB在自己的存储库中提供了官方支持的软件包。这个存储库包含以下软件包:]

mongodb-org: A metapackage that will automatically install the four component packages listed below.[自动安装以下包]

mongodb-org-server: Contains the mongod daemon and associated configuration and init scripts.

[包含mongodb守护进程和相关配置和初始化脚本]

mongodb-org-mongos: Contains the mongos daemon.

[包含守护进程]

mongodb-org-shell: Contains the mongo shell.

[包含命令接口]

mongodb-org-tools: Contains the following MongoDB tools: mongoimport bsondump, mongodump, mongoexport, mongofiles, mongorestore, mongostat, and mongotop.

[包含了mongodb的一些工具]

 

```
# 安装依赖
yum install -y libcurl openssl

# yum安装
cd /etc/yum.repos.d
vim mongodb-org-4.0.repo
"""
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://mirrors.aliyun.com/mongodb/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
"""

yum list | grep mongodb
yum install -y mongodb-org*

# 卸载
yum erase ${rpm -qa | grep mongodb-org}
rm -r /var/log/mongodb
rm -r /var/lib/mongo

# 检查并关闭selinux
getenfore
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

# 关闭防火墙或者开放端口
systemctl stop firewalld

# 开放端口号
firewall-cmd --zone=public --add-port=27017/tcp   # mongodb默认端口号
firewall-cmd --reload  # 重新加载防火墙

systemctl start mongod.service
chkconfig mongod on
ps aux | grep mongod
```

 

```
# 安装依赖
yum install -y libcurl openssl
# yum安装
cd /etc/yum.repos.d
vim mongodb-org-4.0.repo
"""
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://mirrors.aliyun.com/mongodb/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
"""
yum list | grep mongodb
yum install -y mongodb-org*
# 卸载
yum erase ${rpm -qa | grep mongodb-org}
rm -r /var/log/mongodb
rm -r /var/lib/mongo
# 检查并关闭selinux
getenfore
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
# 关闭防火墙或者开放端口
systemctl stop firewalld
# 开放端口号
firewall-cmd --zone=public --add-port=27017/tcp   # mongodb默认端口号
firewall-cmd --reload  # 重新加载防火墙
systemctl start mongod.service
chkconfig mongod on
ps aux | grep mongod
```

连接mongodb

cat /etc/mongod.conf

 

```
# mongod.conf

# for documentation of all options, see:
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# Where and how to store data.
storage:
  dbPath: /data/mongodb
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:


# how the process runs
processManagement:
  fork: true  # fork and run in background
  pidFilePath: /var/run/mongodb/mongod.pid  # location of pidfile
  timeZoneInfo: /usr/share/zoneinfo

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1,172.16.2.15  # Enter 0.0.0.0,:: to bind to all IPv4 and IPv6 addresses or, alternatively, use the net.bindIpAll setting.

#security:

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options

#auditLog:

#snmp:
```

 

```
# mongod.conf
# for documentation of all options, see:
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log
# Where and how to store data.
storage:
  dbPath: /data/mongodb
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:
# how the process runs
processManagement:
  fork: true  # fork and run in background
  pidFilePath: /var/run/mongodb/mongod.pid  # location of pidfile
  timeZoneInfo: /usr/share/zoneinfo
# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1,172.16.2.15  # Enter 0.0.0.0,:: to bind to all IPv4 and IPv6 addresses or, alternatively, use the net.bindIpAll setting.
#security:
#operationProfiling:
#replication:
#sharding:
## Enterprise-Only Options
#auditLog:
#snmp:
```

如果出现错误信息提示没有找到/root/.dbshell,这个没关系,等你里面输入了命令后就会自动生成,这个文件是用来记录命令历史的.其他的警告信息不用管,不影响使用.

 

```
# 连接远程的mongodb,需要加--host来指定目标IP,--port指定端口.例如:
mongo --host 192.168.77.130 --port 27017
mongo -uUSER_NAME -pPASSWORD --authenticationDatabase db

# 进入到mongodb的shell
mongo
```

 

```
# 连接远程的mongodb,需要加--host来指定目标IP,--port指定端口.例如:
mongo --host 192.168.77.130 --port 27017
mongo -uUSER_NAME -pPASSWORD --authenticationDatabase db
# 进入到mongodb的shell
mongo
```

.mongorc.js

Shell开始运行时,mongo将在用户HOME目录下检查名为. mongorc.js的JavaScript文件,如果找到,mongo将在首次命令行显示之前解释执行.mongorc.js的内容.如果你使用shell执行一个JavaScript或表达式,无论是通过命令行使用--eval选项,或通过指定a.js file to mongo,mongo都将在JavaScript处理完成之后读取.mongorc.js``文件.你可以通过使用 :option:`--norc` 选项禁止加载.mongorc.js``.