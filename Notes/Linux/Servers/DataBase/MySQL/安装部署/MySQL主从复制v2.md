[TOC]



# 一， 主从复制介绍

MySQL数据库的主从复制方案，与使用scp/rsync等命令进行的文件级别复制类似，都是数据的远程传输，只不过MySQL的主从复制是其自带的功能，无需借助第三方工具，而且，MySQL的主从复制并不是数据库磁盘上的文件直接拷贝，而是通过逻辑的binlog日志复制到要同步的服务器本地，然后由本地的线程读取日志里面的SQL语句，重新应用到MySQL数据库中。

## 1.1 概述

MySQL数据库支持单向，双向，链式级联，环状等不同业务场景的复制。在复制过程中，一台服务器充当主服务器（Master），接收来自用户的内容更新，而一个或多个其他的服务器充当从服务器（Slave），接收来自主服务器binlog文件的日志内容，解析出SQL，重新更新到从服务器，使得主从服务器数据达到一致。

如果设置了链式级联复制，那么，从服务器（Slave）本身除了充当从服务器外，也会同时充当其下面从服务器的主服务器。链式级联复制类似A-->B-->C的复制形式。

**下图为单向主从复制架构逻辑图，此架构只能在Master端进行数据写入**

![QQ截图20170721180412.png-30.1kB](http://static.zybuluo.com/chensiqi/nm0ohwspkgghlknl5qsj389n/QQ%E6%88%AA%E5%9B%BE20170721180412.png)

![QQ截图20170721180604.png-58.6kB](http://static.zybuluo.com/chensiqi/s8jd916b8651ucgfn8smldw9/QQ%E6%88%AA%E5%9B%BE20170721180604.png)

**下图为双向主主复制逻辑架构图，此架构可以在Master1端或Master2端进行数据写入，或者两端同时写入数据（需要特殊设置）**

![QQ截图20170721180728.png-32.8kB](http://static.zybuluo.com/chensiqi/3izzhd19jetkys86oiqjb56a/QQ%E6%88%AA%E5%9B%BE20170721180728.png)

**下图为线性级联单向双主复制逻辑架构图，此架构只能在Master1端进行数据写入，工作场景中，Master1和master2作为主主互备，Slave1作为从库，中间的Master2需要做特殊的设置。**

![QQ截图20170721181032.png-47.1kB](http://static.zybuluo.com/chensiqi/qy7heqpfy5iub3sl8m4doxng/QQ%E6%88%AA%E5%9B%BE20170721181032.png)

**下图为环状级联单向多主同步逻辑架构图，任意一个点都可以写入数据，此架构比较复杂，属于极端环境下的“成品”，一般场景慎用**

![QQ截图20170721190430.png-60.6kB](http://static.zybuluo.com/chensiqi/ccd4xqja3tkb84or04wptjt2/QQ%E6%88%AA%E5%9B%BE20170721190430.png)

> 在当前的生产工作中，MySQL主从复制都是异步的复制方式，既不是严格实时的数据同步，但是正常情况下给用户的体验是真实的。

## 1.2 企业应用场景

MySQL主从复制集群功能使得MySQL数据库支持大规模高并发读写成为可能，同时有效地保护了物理服务器宕机场景的数据备份。

**应用场景1：从服务器作为主服务器的实时数据备份**

- 主从服务器架构的设置可以大大加强MySQL数据库架构的健壮性。例如：当主服务器出现问题时，我们可以人工或设置自动切换到从服务器继续提供服务，此时从服务器的数据与宕机时的主数据库几乎是一致的。

- 这类似NFS存储数据通过inotify + rsync同步到备份的NFS服务器，只不过MySQL的复制方案是其自带的工具。

- 利用MySQL的复制功能进行数据备份时，在硬件故障，软件故障的场景下，该数据备份是有效的，但对于人为地执行drop，delete等语句删除数据的情况，从库的备份功能就没用了，因为从服务器也会执行删除的语句。

**应用场景2：主从服务器实现读写分离，从服务器实现负载均衡**

- 主从服务器架构可通过程序（PHP，java等）或代理软件（mysql-proxy，Amoeba）实现对用户（客户端）的请求读写分离，即让从服务器仅仅处理用户的select查询请求，降低用户查询响应时间，以及同时读写在主服务器上带来的访问压力。对于更新的数据（例如：update，insert，delete语句），则仍然交给主服务器处理，确保主服务器和从服务器保持实时同步。

- 百度，淘宝，新浪等绝大多数的网站都是用户浏览页面多于用户发布内容，因此通过在从服务器上接收只读请求，就可以很好地减轻主库的读压力，且从服务器可以很容易地扩展为多台，使用LVS做负载均衡效果就非常棒了，这就是传说中的数据库读写分离架构。逻辑架构图如下所示：

![QQ截图20170721192135.png-136.6kB](http://static.zybuluo.com/chensiqi/g881eb4yypxkfqm8vjuo09aj/QQ%E6%88%AA%E5%9B%BE20170721192135.png)

**应用场景3：把多个从服务器根据业务重要性进行拆分访问**

可以把几个不同的从服务器，根据公司的业务进行拆分。例如：有为外部用户提供查询服务的从服务器，有内部DBA用来数据备份的从服务器，还有为公司内部人员提供访问的后台，脚本，日志分析及供开发人员查询使用的从服务器。这样的拆分除了减轻主服务器的压力外，还可以使数据库对外部用户浏览，内部用户业务处理及DBA人员的备份等互不影响。

## 1.3 实现主从读写分离的方案

（1）通过程序实现读写分离（性能和效率最佳，推荐）

PHP和Java程序都可以通过设置多个连接文件轻松地实现对数据库的读写分离，即当语句关键字为select时，就去连接读库的连接文件，若为update，insert，delete时，则连接写库的连接文件。
通过程序实现读写分离的缺点就是需要开发人员对程序进行改造，使其对下层不透明，但这种方式更容易开发和实现，适合互联网业务场景。

**根据业务重要性拆分从库方案**
 ![QQ截图20170721194745.png-158.4kB](http://static.zybuluo.com/chensiqi/0ehfth40dem0jmgfbme795ve/QQ%E6%88%AA%E5%9B%BE20170721194745.png)

（2）通过开源的软件实现读写分离

MySQL-proxy，Amoeba等代理软件也可以实现读写分离功能，这些软件的稳定性和功能一般，不建议生产使用。绝大多数公司常用的还是在应用端发程序实现读写分离。

（3）大型门户独立开发DAL层综合软件

百度，阿里等大型门户都有开发牛人，会花大力气开发适合自己业务的读写分离，负载均衡，监控报警，自动扩容，自动收缩等一系列功能的DAL层软件。

![QQ截图20170721195448.png-105.8kB](http://static.zybuluo.com/chensiqi/ypg6silomt2a89mmcpupojfr/QQ%E6%88%AA%E5%9B%BE20170721195448.png)

## 1.4 主从复制原理介绍

- MySQL的主从复制是一个异步的复制过程（虽然一般情况下感觉是实时的），数据将从一个MySQL数据库（我们称之为Master）复制到另一个MySQL数据库（我们称之为Slave），在Master与Slave之间实现整个主从复制的过程是由三个线程参与完成的。其中有两个线程（SQL线程和I/O线程）在Slave端，另外一个线程（I/O线程）在Master端。

- 要实现MySQL的主从复制，首先必须打开Master端的binlog记录功能，否则就无法实现。因为整个复制过程实际上就是Slave从Master端获取binlog日志，然后再在Slave上以相同顺序执行获取的binlog日志中所记录的各种SQL操作。

- 要打开MySQL的binlog记录功能，可通过在MySQL的配置文件my.cnf中的mysqld模块（[mysqld]标识后的参数部分）增加“log-bin”参数选项来实现，具体信息如下。

```shell
[mysqld]
log-bin=/data/3306/mysql-bin
```

**提示：**
有些同学把log-bin放在了配置文件结尾，而不是[mysqld]标识后，从而导致配置复制不成功。

## 1.5 主从复制原理过程

**下面简单描述MySQL Replication的复制原理过程**

- 在Slave服务器上执行start slave命令开启主从复制开关，开始进行主从复制
- 此时，Slave服务器的I/O线程会通过在Master上已经授权的复制用户权限请求连接Master服务器，并请求从指定binlog日志文件的指定位置（日志文件名和位置就是在配置主从复制服务时执行change master命令指定的）之后开始发送binlog日志内容。
- Master服务器接收到来自Slave服务器的I/O线程的请求后，其上负责复制的I/O线程会根据Slave服务器的I/O线程请求的信息分批读取指定binlog日志文件指定位置之后的binlog日志信息，然后返回给Slave端的I/O线程。返回的信息中除了binlog日志内容外，还有在Master服务器端记录的新的binlog文件名称，以及在新的binlog中的下一个指定更新位置。
- 当Slave服务器的I/O线程获取到Master服务器上I/O线程发送的日志内容，日志文件及位置点后，会将binlog日志内容依次写到Slave端自身的Relay  Log（即中继日志）文件（MySQL-relay-bin.xxxx）的最末端，并将新的binlog文件名和位置记录到master-info文件中，以便下一次读取Master端新binlog日志时能够告诉Master服务器从新binlog日志的指定文件及位置开始请求新的binlog日志内容。
- Slave服务器端的SQL线程会实时检测本地Relay Log中I/O线程新增加的日志内容，然后及时地把Relay  Log文件中的内容解析成SQL语句，并在自身Slave服务器上按解析SQL语句的位置顺序执行应用这些SQL语句，并在relay-log.info中记录当前应用中继日志的文件名及位置点。

经过了上面的过程，就可以确保在Master端和Slave端执行了同样的SQL语句。当复制状态正常时，Master端和Slave端的数据是完全一样的。当然，MySQL的复制机制也有一些特殊情况，具体请参考官方的说明，大多数情况下，同学们不用担心。

**MySQL Replication的复制原理逻辑图**

![QQ截图20170721204858.png-100.1kB](http://static.zybuluo.com/chensiqi/l7j7ej39llknjrgkms668ere/QQ%E6%88%AA%E5%9B%BE20170721204858.png)

**特别说明：**
当企业面试MySQL主从复制原理时，不管是面试还是笔试，都要尽量画图表达，而不是口头讲或文字描述，面试时可以找黑板或拿出纸来给面试官详细讲解。

**下面针对MySQL主从复制原理的重点进行小结：**

- 主从复制是异步的逻辑的SQL语句级的复制
- 复制时，主库有一个I/O线程，从库有两个线程，即I/O和SQL线程
- 实现主从复制的必要条件是主库要开启记录binlog功能
- 作为复制的所有MySQL节点的server-id都不能相同。
- binlog文件只记录对数据库有更改的SQL语句（来自主数据库内容的变更），不记录任何查询（如select，show）语句。

# 二，MySQL主从复制实践

## 2.1 环境准备

mysql版本：5.6

安装方式：二进制安装，过程略

主机IP：192.168.100.5

单机多实例方式：3306端口为主，3307端口为从

**提示：**

MySQL主从复制实践对环境的要求比较简单，可以是单机单数据库多实例的环境，也可以是两台服务器，每个机器一个独立数据库的环境。一般情况下，小企业在做常规的主从复制时，主从服务器多数在不同的机器上，并且监听的端口均为默认的3306.虽然不在同一个机器上，但是步骤和过程却是一样的。

## 2.2 Master操作配置

设置server-id值并开启binlog功能参数

根据之前介绍的MySQL主从复制原理我们知道，要实现主从复制，关键是要开启binlog日志功能，所以，首先来打开主库的binlog日志参数。

修改主库的配置文件。执行vi /data/3306/my.cnf,编辑多实例3006的my.cnf配置文件，按如下内容修改两个参数：

```
[mysqld]
server-id = 1               #用于同步的每台机器或实例server-id都不能相同
log-bin = /data/3306/mysql-bin     #binlog日志的位置 
```

**提示：**

上面的两个参数要放在my.cnf中的[mysqld]模块下，否则会出错。
不同实例间server-id的值不可以重复
要先在my.cnf配置文件中查找相关参数，并按要求修改。若发现不存在，再添加参数，切记，参数不能重复。
修改my.cnf配置后，需要重启动数据库，命令为：/data/3306/mysql restart ,注意要确认真正重启了。



检查配置参数后的结果，如下：

```
root@localhost ~]# egrep "server-id|log-bin" /data/3306/my.cnf
server-id = 1
log-bin=/data/3306/mysql-bin        #log-bin后面也可以不带等号内容，MySQL会使用默认日志
```



重启主库MySQL服务，命令如下：

```
[root@localhost ~]# /data/3306/mysql restart
Restarting MySQL...
Stoping MySQL...
Starting MySQL....
```



登陆数据库，检查参数的更改情况，如下：

```
[root@localhost ~]# mysql -uroot -p123123 -S /data/3306/mysql.sock       #登陆3306实例
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.5.22-log Source distribution

Copyright (c) 2000, 2011, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show variables like 'server_id';         #查看MySQL的系统变量（like类似于grep过滤）
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 1     |           #配置的server_id为1
+---------------+-------+
1 row in set (0.00 sec)

mysql> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | ON    |           #binlog功能已开启
+---------------+-------+
1 row in set (0.00 sec)

mysql> 

#这样，binlog功能就开启了。
```



在主库上建立用于主从复制的账号

根据主从复制的原理，从库要想和主库同步，必须有一个可以连接主库的账号，并且这个账号是主库上创建的，权限是允许主库的从库连接并同步数据。

登陆MySQL3306实例主数据库，命令如下：

```
[root@localhost ~]# mysql -uroot -p123123 -S /data/3306/mysql.sock
```

建立用于从库复制的账号yunjisuan，命令如下：

```
mysql> grant replication slave on *.* to 'yunjisuan'@'192.168.0.%' identified by 'yunjisuan123';
Query OK, 0 rows affected (0.00 sec)

#语句说明：
1）replication slave为mysql同步的必须权限，此处不要授权all权限
2）*.* 表示所有库所有表，也可以指定具体的库和表进行复制。例如yunjisuan.test中，yunjisuan为库名，test为表名
3）'yunjisuan'@'192.168.0.%' yunjisuan为同步账号。192.168.0.%为授权主机网段，使用了%表示允许整个192.168.0.0网段可以用yunjisuan这个用户访问数据库
4）identified by 'yunjisuan123';  yunjisuan123为密码，实际环境下设置的复杂些为好
```

**创建完账号并授权后，需要刷新权限，使授权的权限生效**

```
mysql> flush privileges;            #刷新权限
Query OK, 0 rows affected (0.00 sec)
```

检查主库创建的yunjisuan复制账号命令及结果如下：

```
mysql> select user,host from mysql.user;
+-----------+-------------+
| user      | host        |
+-----------+-------------+
| root      | 127.0.0.1   |
| yunjisuan | 192.168.0.% |         #出现这行表示复制账号已经配置好了
| root      | ::1         |
|           | localhost   |
| root      | localhost   |
+-----------+-------------+
5 rows in set (0.00 sec)

#说明：
MySQL里的授权用户是以数据表格的形式存储在mysql这个库的user表里。
mysql> select user,host from mysql.user where user='yunjisuan';  #where是SQL查询语句的条件
+-----------+-------------+
| user      | host        |
+-----------+-------------+
| yunjisuan | 192.168.0.% |
+-----------+-------------+
1 row in set (0.00 sec)

mysql> show grants for yunjisuan@'192.168.0.%';         #查看账号的授权情况
+--------------------------------------------------------------------------------------------------------------------------------+
| Grants for yunjisuan@192.168.0.%                                                                                               |
+--------------------------------------------------------------------------------------------------------------------------------+
| GRANT REPLICATION SLAVE ON *.* TO 'yunjisuan'@'192.168.0.%' IDENTIFIED BY PASSWORD '*A2CC7FA422EF5A7CB098FEA7732C1F78CDC32F67' |               #结果显示授权正确
+--------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```



实现对主数据库锁表只读

1）对主数据库锁表只读（当前窗口不要关掉）的命令如下：

```
mysql> flush table with read lock;
Query OK, 0 rows affected (0.00 sec)
```

> **提示：**
> 在引擎不同的情况下，这个锁表命令的时间会受下面参数的控制。锁表时，如果超过设置时间不操作会自动解锁。
> 默认情况下自动解锁的时长参数值如下：

```
mysql> show variables like '%timeout%';
+----------------------------+----------+
| Variable_name              | Value    |
+----------------------------+----------+
| connect_timeout            | 10       |
| delayed_insert_timeout     | 300      |
| innodb_lock_wait_timeout   | 120      |
| innodb_rollback_on_timeout | OFF      |
| interactive_timeout        | 28800    |       #自动解锁时间受本参数影响
| lock_wait_timeout          | 31536000 |
| net_read_timeout           | 30       |
| net_write_timeout          | 60       |
| slave_net_timeout          | 3600     |
| wait_timeout               | 28800    |       #自动解锁时间受本参数影响
+----------------------------+----------+
10 rows in set (0.00 sec)

#提示：有关这两个参数，请同学们自行测试
```

2）锁表后查看主库状态。可通过当前binlog日志文件名和二进制binlog日志偏移量来查看，结果如下：

注意，show master status;命令显示的信息要记录在案，后面的从库导入全备后，继续和主库复制时就是要从这个位置开始。

```
mysql> show master status;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      345 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```

或者新开一个命令行窗口，用如下命令查看锁表后的主库binlog位置点信息：

```
[root@localhost ~]# mysql -uroot -p123123 -S /data/3306/mysql.sock -e "show master status"
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      533 |              |                  |
+------------------+----------+--------------+------------------+
```

3）锁表后，一定要单开一个新的SSH窗口，导出数据库的所有数据，如果数据量很大（50GB以上），并且允许停机，可以停库直接打包数据文件进行迁移，那样更快。

```
[root@localhost ~]# mkdir -p /server/backup
[root@localhost ~]# mysqldump -uroot -p123123 -S /data/3306/mysql.sock --events -A -B | gzip >/server/backup/mysql_bak.$(date +%F).sql.gz

#注意：-A表示备份所有库；-B表示增加use DB和 drop 等（导库时会直接覆盖原有的）

[root@localhost ~]# ll /server/backup/mysql_bak.2017-07-21.sql.gz 
-rw-r--r--. 1 root root 137344 Jul 21 10:17 /server/backup/mysql_bak.2017-07-21.sql.gz

#为了确保导出数据期间，数据库没有数据插入，导库完毕可以再次检查主库状态信息，结果如下：

[root@localhost ~]# mysql -uroot -p123123 -S /data/3306/mysql.sock -e "show master status"
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      533 |              |                  |
+------------------+----------+--------------+------------------+

提示：若无特殊情况，binlog文件及位置点和锁表后导出数据前是一致的，即没有变化。
#导出数据完毕后，解锁主库，恢复可写，命令如下.因为主库还要对外提供服务，不能一直锁定不让用户访问。

mysql> unlock tables;
Query OK, 0 rows affected (0.00 sec)

#可能会有同学因为锁表后的binlog位置问题犯迷糊，实际上做从库前，无论主库更新了多少数据，最后从库都可以从上面show master status的位置很快赶上主库的进度。
```



把主库导出的MySQL数据迁移到从库

> 下面主要讲解单数据库多实例的主从配置，也就是说，mysqldump备份的3306实例的数据和要恢复的3307实例在一台机器上，因此无需异地复制拷贝。想查看主库导出的数据，如下：

```
[root@localhost ~]# ll /server/backup/mysql_bak.2017-07-21.sql.gz 
-rw-r--r--. 1 root root 137344 Jul 21 10:17 /server/backup/mysql_bak.2017-07-21.sql.gz
```

## 2.3 Slave操作配置

设置server-id值并关闭binlog功能参数

数据库的server-id一般在一套主从复制体系内是唯一的，这里从库的server-id要和主库及其他从库的不同，并且要注释掉从库的binlog参数配置，如果从库不做级联复制，并且不作为备份用，就不要开启binlog，开启了反而会增加从库磁盘I/O等的压力。

但是，有以下两种情况需要打开从库的binlog记录功能，记录数据库更新的SQL语句：

级联同步A-->B-->C中间的B时，就要开启binlog记录功能。

在从库做数据库备份，数据库备份必须要有全备和binlog日志，才是完整的备份。

（1）修改配置文件，配置从库1的相关参数

执行vi /data/3307/my.cnf,编辑my.cnf配置文件，按如下内容修改两个参数：

```
[mysqld]
server-id = 3       #调整等号后的数值，和任何一个数据库实例都不同
```

**提示：**
上面两参数要放在my.cnf中的[mysqld]模块下，否则会出错。
server-id的值不能和任何MySQL实例重复。
要先在文件中查找相关参数按要求修改。若发现不存在，再添加参数，切记，参数不能重复。
修改my.cnf配置后需要重启数据库，命令为：/data/3307/mysql restart,注意要确认真正重启了。

（2）检查配置参数后的结果
 **命令如下：**

```
[root@localhost ~]# egrep "server-id|log-bin" /data/3307/my.cnf
server-id = 3
```

（3）重启3307的从数据库
 **命令如下：**

```
[root@localhost ~]# /data/3307/mysql restart
Restarting MySQL...
Stoping MySQL...
Starting MySQL....
[root@localhost ~]# ss -antup | grep 3307
tcp    LISTEN     0      128                    *:3307                  *:*      users:(("mysqld",5659,11))
```

（4）登陆数据库检查参数的改变情况
 **命令如下：**

```
[root@localhost ~]# mysql -uroot -p123123 -S /data/3307/mysql.sock 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.5.22 Source distribution

Copyright (c) 2000, 2011, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+
1 row in set (0.00 sec)

mysql> show variables like 'server_id';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| server_id     | 3     |
+---------------+-------+
1 row in set (0.00 sec)

mysql> 
```

把从主库mysqldump导出的数据恢复到从库

**操作命令如下：**

```
[root@localhost ~]# cd /server/backup/
[root@localhost backup]# ls -l
total 136
-rw-r--r--. 1 root root 137344 Jul 21 10:17 mysql_bak.2017-07-21.sql.gz
[root@localhost backup]# gzip -d mysql_bak.2017-07-21.sql.gz 
[root@localhost backup]# ll
total 496
-rw-r--r--. 1 root root 506730 Jul 21 10:17 mysql_bak.2017-07-21.sql
[root@localhost backup]# mysql -uroot -p123123 -S /data/3307/mysql.sock <mysql_bak.2017-07-21.sql           #这是把数据还原到3307实例的命令

#提示：
如果备份时使用了-A参数，则在还原数据到3307实例时，登陆3307实例的密码也会和3306主库的一致，因为3307实例的授权表MySQL也被覆盖了。
```

登陆3307从库，配置复制参数

（1）MySQL从库连接主库的配置信息如下：

```
CHANGE MASTER TO
MASTER_HOST='192.168.0.200',         #这里是主库的IP
MASTER_PORT=3306,                    #这里是主库的端口，从库端口可以和主库不同
MASTER_USER='yunjisuan',             #这里是主库上建立的用于复制的用户yunjisuan
MASTER_PASSWORD='yunjisuan123',      #这里是yunjisuan用户的密码
MASTER_LOG_FILE='mysql-bin.000001',  #这里是show master status时查看到的二进制日志文件名称，注意不能多空格
MASTER_LOG_POS=533;                  #这里是show master status时查看到的二进制日志偏移量，注意不能多空格

#提示：字符串用单引号括起来，数值不用引号，注意内容前后不能有空格。
```

（2）登陆数据库后，去掉上述语句中的注释，执行如下：

```
mysql> CHANGE MASTER TO MASTER_HOST='192.168.0.200',MASTER_PORT=3306,MASTER_USER='yunjisuan',MASTER_PASSWORD='yunjisuan123',MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=533;

#提示：这个步骤的参数一定不能错，否则，数据库复制配置会失败
```

**上述操作的原理实际上是把用户密码等信息写入从库新的master.info文件中**

```
[root@localhost backup]# ll /data/3307/data/master.info 
-rw-rw----. 1 mysql mysql 90 Jul 21 11:31 /data/3307/data/master.info
[root@localhost backup]# cat /data/3307/data/master.info
18
mysql-bin.000001                    #这里是show master status时查看的二进制日志文件名称
533                                 #这里是show master status时查看的二进制日志偏移量
192.168.0.200                       #这里是主库的IP
yunjisuan                           #这里是主库上建立的用于复制的用户yunjisuan
yunjisuan123                        #这里是yunjisuan用户的密码
3306                                #这里是主库的端口
60

0...以下省略若干...
```

## 2.4 开始同步并测试

（1）启动从库主从复制开关，并查看复制状态
 **相关语句如下：**

```
[root@localhost backup]# mysql -uroot -p123123 -S /data/3307/mysql.sock -e "start slave"
[root@localhost backup]# mysql -uroot -p123123 -S /data/3307/mysql.sock -e "show slave status\G"
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.0.200
                  Master_User: yunjisuan
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 533
               Relay_Log_File: relay-bin.000002
                Relay_Log_Pos: 253
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 533
              Relay_Log_Space: 403
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
```

**主从同步是否成功，最关键的为下面的3项状态参数：**

```
[root@localhost backup]# mysql -uroot -p123123 -S /data/3307/mysql.sock -e "show slave status\G" | egrep "IO_Running|SQL_Running|Seconds_Behind_Master"
        Slave_IO_Running: Yes
        Slave_SQL_Running: Yes
        Seconds_Behind_Master: 0
```

- [x] :Slave_IO_Running: Yes,这个时I/O线程状态，I/O线程负责从从库到主库读取binlog日志，并写入从库的中继日志，状态为Yes表示I/O线程工作正常。
- [x] :Slave_SQL_Running: Yes,这个是SQL线程状态，SQL线程负责读取中继日志（relay-log）中的数据并转换为SQL语句应用到从数据库中，状态为Yes表示I/O线程工作正常。
- [x] :Seconds_Behind_Master:0,这个是复制过程中从库比主库延迟的秒数，这个参数极度重要，但企业里更准确地判断主从延迟的方法为：在主库写时间戳，然后从库读取时间戳，和当前数据库时间进行比较，从而认定是否延迟。

（2）测试主从复制结果
 在主库上写入数据，然后观察从库的数据状况。

```
[root@localhost backup]# mysql -uroot -p123123 -S /data/3306/mysql.sock -e "create database benet"
[root@localhost backup]# mysql -uroot -p123123 -S /data/3307/mysql.sock -e "show databases"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| benet              |
| mysql              |
| performance_schema |
| test               |
+--------------------+
[root@localhost backup]# mysql -uroot -p123123 -S /data/3306/mysql.sock -e "create database yunjisuan"
[root@localhost backup]# mysql -uroot -p123123 -S /data/3307/mysql.sock -e "show databases"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| benet              |
| mysql              |
| performance_schema |
| test               |
| yunjisuan          |
+--------------------+
[root@localhost backup]# mysql -uroot -p123123 -S /data/3306/mysql.sock -e "drop database yunjisuan"
[root@localhost backup]# mysql -uroot -p123123 -S /data/3307/mysql.sock -e "show databases"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| benet              |
| mysql              |
| performance_schema |
| test               |
+--------------------+

#根据测试可以判断，主从库是同步的。
```

## 2.5 配置小结

**MySQL主从复制配置完整步骤如下：**

1. 准备两台数据库环境或单台多实例环境，确定能正常启动和登陆
2. 配置my.cnf文件：主库配置log-bin和server-id参数；从库配置server-id，该值不能和主库及其他从库一样，一般不开启从库log-bin功能。注意，配置参数后要重启才能生效。
3. 登陆主库，增加从库连接主库同步的账户，例如：yunjisuan，并授权replication slave同步的权限。
4. 登陆主库，整库锁表flush table with read lock（窗口关闭后即失效，超时参数设置的时间到了，锁表也失效），然后show master status查看binlog的位置状态。
5. 新开窗口，在Linux命令行备份导出原有的数据库数据，并拷贝到从库所在的服务器目录。如果数据库数据量很大，并且允许停机，可以停机打包，而不用mysqldump。
6. 导出主库数据后，执行unlock tables解锁主库。
7. 把主库导出的数据恢复到从库
8. 根据主库的show master status查看到的binlog的位置状态，在从库执行change master to....语句。
9. 从库开启复制开关，即执行start slave；。
10. 从库show slave status\G,检查同步状态，并在主库进行更新测试。

# 三， 主从复制线程状态说明及用途

## 3.1 主库I/O线程状态说明

（1）登陆主数据库查看MySQL线程的同步状态

```
mysql> show processlist\G
*************************** 1. row ***************************
     Id: 1
   User: root
   Host: localhost
     db: NULL
Command: Query
   Time: 0
  State: NULL
   Info: show processlist
*************************** 2. row ***************************
     Id: 5
   User: yunjisuan
   Host: 192.168.0.200:42008
     db: NULL
Command: Binlog Dump
   Time: 267
  State: Master has sent all binlog to slave; waiting for binlog to be updated
   Info: NULL
2 rows in set (0.00 sec)

#提示：上述状态的意思是线程已经从binlog日志读取所有更新，并已经发送到了从数据库服务器。线程目前为空闲状态，等待由主服务器上二进制日志中的新事件更新。
```

**下图中列出了主服务器binlog Dump线程中State列的最常见状态。如果你没有在主服务器上看见任何binlog Dump线程，则说明复制没有运行，二进制binlog日志由各种事件组成，事件通常会为更新添加信息。**

![QQ截图20170722220305.png-80kB](http://static.zybuluo.com/chensiqi/4kpegx0nz0j0sar5b9b59826/QQ%E6%88%AA%E5%9B%BE20170722220305.png)

（2）登陆从数据库查看MySQL线程工作状态
 从库有两个线程，即I/O和SQL线程。从库I/O线程的状态如下：

```
mysql> show processlist\G
*************************** 1. row ***************************
     Id: 3
   User: root
   Host: localhost
     db: NULL
Command: Query
   Time: 0
  State: NULL
   Info: show processlist
*************************** 2. row ***************************
     Id: 8
   User: system user
   Host: 
     db: NULL
Command: Connect
   Time: 659
  State: Waiting for master to send event
   Info: NULL
*************************** 3. row ***************************
     Id: 9
   User: system user
   Host: 
     db: NULL
Command: Connect
   Time: 511
  State: Slave has read all relay log; waiting for the slave I/O thread to update it
   Info: NULL
3 rows in set (0.00 sec)
```

**下图列出了从服务器的I/O线程的State列的最常见的状态。该状态也出现在Slave_IO_State列，由SHOW SLAVE STATUS显示。**

![QQ截图20170722220305.png-109.4kB](http://static.zybuluo.com/chensiqi/q1aj232151oderxgmowdbk86/QQ%E6%88%AA%E5%9B%BE20170722220305.png)

**下图列出了从服务器的SQL线程的State列的最常见状态**

![QQ截图20170722220305.png-51.4kB](http://static.zybuluo.com/chensiqi/u3fusxx6unkotrv6uqnlf6fe/QQ%E6%88%AA%E5%9B%BE20170722220305.png)



## 3.2 查看MySQL线程同步状态的用途

通过MySQL线程同步状态可以看到同步是否正常进行，故障的位置是什么，另外还可查看数据库同步是否完成，可用于主库宕机切换数据库或人工数据库主从切换迁移等。

例如：主库宕机，要选择最快的从库将其提升为主库，就需要查看主从库的线程状态，如果主从复制在正常情况下进行角色切换，也需要查看主从库的线程状态，根据复制状态确定更新是否完成。

# 四，应用技巧实践

## 4.1 工作中MySQL从库停止复制故障案例

模拟重现故障的能力是运维人员最重要的能力。下面就来次模拟操作。先在从库创建一个库，然后去主库创建同名的库来模拟数据冲突，命令如下：

```
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.0.200
                  Master_User: yunjisuan
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 544
               Relay_Log_File: relay-bin.000010
                Relay_Log_Pos: 336
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 1007
                   Last_Error: Error 'Can't create database 'yunjisuan'; database exists' on query. Default database: 'yunjisuan'. Query: 'create database yunjisuan'
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 451
              Relay_Log_Space: 810
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 1007
               Last_SQL_Error: Error 'Can't create database 'yunjisuan'; database exists' on query. Default database: 'yunjisuan'. Query: 'create database yunjisuan'
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
1 row in set (0.00 sec)
```

**对于该冲突，解决办法如下**

**办法一：**关闭从同步，调动sql_slave指针

```
mysql> stop slave;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> set global sql_slave_skip_counter=1;   #将sql线程同步指针向下移动一个，如果多次不同步，可以重复操作
Query OK, 0 rows affected (0.00 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```

对于普通的互联网业务，上述的移动指针的操作带来的问题不是很大。当然，要在确认不影响公司业务的前提下。
若是在企业场景下，对当前业务来说，解决主从同步比主从不一致更重要，如果主从数据一致也是很重要的，那就再找个时间恢复这个从库。
是主从数据不一致更重要，还是保持主从同步持续状态更重要，要根据业务选择。这样Slave就会与Master同步了，主要关键点如下：

```
            Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
            Seconds_Behind_Master: 0   #0表示已经同步状态

#提示：set global sql_slave_skip_counter=n;     #n取值>0,忽略执行N个更新。
```

**办法二：**根据可以忽略的错误号事先在配置文件中配置，跳过指定的不影响业务数据的错误，例如：

```
[root@localhost ~]# grep slave-skip /data/3306/my.cnf
slave-skip-errors = 1032,1062

#提示：类似由于入库重复导致的失败可以忽略，其他情况是不是可以忽略需要根据不同公司的具体业务来评估。
```

**其他可能引起复制故障的原因：**

- MySQL自身的原因及人为重复插入数据。
- 不同的数据库版本会引起不同步，低版本到高版本可以，但是高版本不能往低版本同步。
- MySQL的运行错误或程序bug
- binlog记录模式，例如：row level模式就比默认的语句模式要好。

## 4.2 让MySQL从库记录binlog日志的方法

从库需要记录binlog的应用场景：当前的从库还要作为其他从库的主库，例如级联复制或双主互为主从场景的情况下。下面介绍从库记录binlog日志的方法。
在从库的my.cnf中加入如下参数，然后重启服务生效即可。

```
    log-slave-updates       #必须要有这个参数
    log-bin = /data/3307/mysql-bin
    expire_logs_days = 7    #相当于find /data/3307/ -type f -name "mysql-bin.000*" -mtime +7 | xargs rm -f
```

## 4.3 MySQL主从复制集群架构的数据备份策略

- 有主从复制了，还需要做定时全量加增量备份么？答案是肯定的！
  因为，如果主库有语句级误操作（例如：drop database yunjisuan；），从库也会执行drop database yunjisuan；，这样MySQL主从库就都删除了该数据。
- 把从库作为数据库备份服务器时，备份策略如下：
  高并发业务场景备份时，可以选择在一台从库上备份（Slave5），把从库作为数据库备份服务器时需要在从库开启binlog功能，其逻辑图如下所示：

![QQ截图20170722220305.png-132.7kB](http://static.zybuluo.com/chensiqi/l6wmj1txak157js9iyid774u/QQ%E6%88%AA%E5%9B%BE20170722220305.png)

**步骤如下：**
 1）选择一个不对外提供服务的从库，这样可以确保和主库更新最接近，专门用于做数据备份。
 2）开启从库的binlog功能

备份时可以选择只停止SQL线程，停止应用SQL语句到数据库，I/O线程保留工作状态，执行命令为stop slave  sql_thread;,备份方式可以采取mysqldump逻辑备份或直接物理备份，例如：使用cp，tar（针对/data目录）工具或xtrabackup（第三方的物理备份软件）进行备份，则逻辑备份和物理备份的选择，一般是根据总的备份数据量的多少进行选择的，数据量低于30G，建议选择mysqldump逻辑备份方法，安全稳定，最后把全备和binlog数据发送到备份服务器上留存。

## 4.4 MySQL主从复制延迟问题的原因及解决方案

**问题一：**主库的从库太多，导致复制延迟

从库数量以3~5个为宜，要复制的从节点数量过多，会导致复制延迟。

**问题二：**从库硬件比主库差，导致复制延迟。

查看Master和Slave的系统配置，可能会因为机器配置不当，包括磁盘I/O，CPU，内存等各方面因素造成复制的延迟。这一般发生在高并发大数据量写入场景中。

**问题三：**慢SQL语句太多

假如一条SQL语句执行时间是20秒，那么从执行完毕到从库上能查到数据至少需要20秒，这样就延迟20秒了。
一般要把SQL语句的优化作为常规工作，不断的进行监控和优化，如果单个SQL的写入时间长，可以修改后分多次写入。通过查看慢查询日志或show full processlist命令，找出执行时间长的查询语句或大的事务。

**问题四：**主从复制的设计问题

例如，主从复制单线程，如果主库写并发太大，来不及传送到从库，就会导致延迟。
更高版本的MySQL可以支持多线程复制，门户网站则会自己开发多线程同步功能。

**问题五：**主从库之间的网络延迟

主从库的网卡，网线，连接的交换机等网络设备都可能成为复制的瓶颈，导致复制延迟，另外，跨公网主从复制很容易导致主从复制延迟。

**问题六：**主库读写压力大，导致复制延迟。

主库硬件要搞好一点，架构的前端要加buffer及缓存层。

## 4.5 通过read-only参数让从库只读访问

read-only参数选项可以让从服务器只允许来自从服务器线程或具有SUPER权限的数据库用户进行更新，确保从服务器不接受来自用户端的非法用户更新。
read-only参数允许数据库更新的条件为：

- 具有SUPER权限的用户可以更新，不受read-only参数影响，例如：管理员root。
- 来自从服务器线程可以更新，不受read-only参数影响，例如：前文的yunjisuan用户。
- 再生产环境中，可以在从库Slave中使用read-only参数，确保从库数据不被非法更新。

**read-only参数的配置方法如下：**

**方法一：**直接带 --read-only参数启动或重启数据库，
 使用`killall mysqld`
 或`mysqladmin -uroot -p123123 -S /data/3307/mysql.sock shutdown`
 `mysqld_safe --defaults-file=/data/3307/my.cnf --read-only &`

**方法二：**在my.cnf里[mysqld]模块下加read-only参数重启数据库，配置如下：

```
[mysqld]
read-only
```





