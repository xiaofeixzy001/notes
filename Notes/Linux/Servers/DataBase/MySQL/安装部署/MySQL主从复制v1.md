[TOC]

# 主从复制示例

系统环境:CentOS 6.9 x86_64

mysql: mysql-5.5.59-linux-glibc2.12-x86_64

Master: 172.16.1.6

Slave1: 172.16.1.7

Slave1: 172.16.1.8

![img](MySQL%E5%A4%8D%E5%88%B6%E5%AE%9E%E4%BE%8B.assets/e9ce6891-7f48-4d28-8064-d452e88c9102.png)

## 假设M/S都没数据

master:

\- 启用二进制日志[log-bin]

\- server-id全局唯一

\- 创建一个拥有复制权限的用户账号[replication slave,replication client]

```
# 修改mysql配置文件,-和_没有区别,建议使用统一风格
vim /etc/my.cnf
"""
[mysqld]
read-only  # 只读方式启动服务
log-bin = mysql-bin  # 启用二进制日志功能,log-bin=master-bin一样
server-id = 6  # 设置服务器唯一的id,默认是1,这里按ip最后位定义
innodb_file_per_table = ON
skip_name_resolve = ON  # 忽略名称解析
#expire_logs_days=15 # 自动删除binlog
#innodb_flush_log_at_trx_commit=1
"""

# 检查mysql环境
mysql> show global variables like '%log%'; # 查看二进制日志是否开启
mysql> show variables like 'log_bin';
mysql> show global variables like '%server%';  # 查看server-id
mysql> show master logs;  # 查看二进制日志文件


# 创建用户repluser,并授复制权限
mysql -uroot -p
mysql> grant replication slave,replication client on *.* to 'repluser'@'172.16.1.%' identified by 'replpass';
mysql> select * from mysql.user where user='repluser'\G
mysql> show grants for 'repluser'@'172.16.100.%';
mysql> flush privileges;

# 查看master状态,记录下 FILE 及 Position 的值，在后面进行从服务器操作的时候需要用到
mysql> show master status;
mysql> show master logs;

# 如果以上修改未生效,可重启mysql服务尝试
```

slave：

\- 启用中继日志[relay-log]

\- server-id全局唯一

\- 使用拥有复制权限的账号连接至master,并启动复制线程[change master ...]

\- 启用slave服务[start slave]

```
# 从节点mysql-2
vim /etc/mysql/my.cnf
"""
[mysqld]
#log-bin=mysql-bin  # 如果slave还要作为级联的master,则需要开启bin-log
server-id = 7
read_only = ON  # 设置该节点为只读,不能写.但对拥有super权限的用户无效
relay_log = relay_log
relay_log_index = relay_log.index
innodb_file_per_table = ON
skip_name_resolve = ON
skip_slave_start  # 复制进程就不会随着数据库的启动而启动
"""

# 检查环境
mysql> show global variables like '%log%';
mysql> show global variables like '%server%';

# 执行同步sql语句
mysql> change master to master_host='172.16.1.6',master_user='repluser',master_password='replpass',master_log_file='mysql-bin.000012',master_log_pos=107;

mysql> start slave
mysql> show slave status\G
"""
# 保证以下2项状态即可
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
"""
```

从节点mysql-3除了id不要一样,其他配置一样.

至此,主从复制已实现.

## 假设M中有数据

如果主服务器已经存在应用数据

通过备份master数据,恢复数据至slave节点

```
# 主服务器
# 主数据库进行锁表操作,不让数据再进行写入动作
mysql> flush tables with read lock;  # 5.1版本
mysql> flush table with read lock;  # 5.5版本

# 锁表超时时长受以下2个参数影响,超时后自动解锁
interactive_timeout        28800
wait_timeout               28800 


# 查看主数据库状态
mysql> show master status;

# 记录下 FILE 及 Position 的值,然后将主服务器的现有数据文件复制到从服务器,建议进行压缩全备再传
mysqldump -uroot -p'123.com' -A -B --events --master-data=2 | gzip > /backup/rep.sql.gz
scp /backup/rep.sql.gz root@172.16.1.6:/backup/

# 导入主服务器上的现有的数据,然后进行同步
gunzip /backup/rep.sql.gz
mysql -uroot -p'123.com' < /backup/rep.sql

# 最后取消主数据库锁定
mysql> unlock tables;
```

亦可以在备份时跟上-x,自动锁表,可以省去备份前后的2步加锁解锁操作.

剩下操作同第一种情况

```
# 从服务器
# 导入主服务器上的现有的数据,然后进行同步
gunzip /backup/rep.sql.gz
mysql -uroot -p'123.com' < /backup/rep.sql
```

测试

主服务器上的操作

```
# 在主服务器上创建数据库first_db
mysql> create database first_db;

# 在主服务器上创建表first_tb
mysql> create table first_tb(id int(3),name char(10));

# 在主服务器上的表first_tb中插入记录
mysql> insert into first_tb values (001,’myself’);
```

在从服务器上查看

```
mysql> show databases;
mysql> use first_db
mysql> show tables;
mysql> select * from first_tb;
```

至此,mysql的主从复制,一主多从已完成.

## 遇到的问题

1,主库执行show master status 没有结果,主库的binlog功能没有开启或没生效.

egrep "server-id | log-bin" /etc/mysql/my.cnf

\> show variables like "server-id"

\> show variables like "log_bin"

2,查看slave的状态出现的是:

Slave_IO_State: Connecting to master

查看你change的时候host（这里最好使用ip）、port、user、password、fileN、pos是否正确,有没有真的创建了用户zyh。如果还不行在查看一下两台服务器能不能ping通,主节点主机能ping通从节点，反过来不行

复制架构中应该注意的问题

1,限制从服务器为只读

read_only=ON # 配置文件添加此项,但对拥有super权限的用户无效

flush tables with read lock;  # 为所有表加读锁

2,如何保证主从复制的事务安全

在master节点启用参数:

sync_binlog=ON

sync_master_info

如果用到的存储引擎为InnoDB,还应启用参数:

innodb_flush_logs_at_trx_commit=ON 

innodb_support_xa=ON

在slave节点上启用参数:

skip_slave_start=ON

sync_relay_log

sync_relay_log_info

# 主主同步示例

在企业中,数据库高可用一直是企业的重中之重,中小企业很多都是使用mysql主从方案,一主多从,读写分离等,但是单主存在单点故障,从库切换成主库需要作改动.因此,如果是双主或者多主,就会增加mysql入口,增加高可用.不过多主需要考虑自增长ID问题,这个需要特别设置配置文件,比如双主,可以使用奇偶,总之,主之间设置自增长ID相互不冲突就能完美解决自增长ID冲突问题.

## 需要注意的一些问题

1,数据不一致,因此慎用;

2,自增id

配置一个节点使用奇数id:

auto_increment_offset=1

auto_increment_increment=2

另一个节点使用偶数id:

auto_increment_offset=1

auto_increment_increment=2

## MySQL双主架构方案思路

1,两台mysql都可读写,互为主备,默认只使用一台(masterA)负责数据的写入,另一台(masterB)备用;

2,masterA是masterB的主库,masterB又是masterA的主库,它们互为主从;

3,两台主库之间做高可用,可以采用keepalived等方案(使用VIP对外提供服务);

4,所有提供服务的从服务器与masterB进行主从同步(双主多从);

5,建议采用高可用策略的时候,masterA或masterB 均不因宕机恢复后而抢占VIP(非抢占模式),这样做可以在一定程度上保证主库的高可用,在一台主库down掉之后,可以在极短的时间内切换到另一台主库上(尽可能减少主库宕机对业务造成的影响),减少了主从同步给线上主库带来的压力.

但是也有几个不足的地方:

1,masterB可能会一直处于空闲状态(可以用它当从库,负责部分查询);

2,主库后面提供服务的从库要等masterB先同步完了数据后才能去masterB上去同步数据,这样可能会造成一定程度的同步延时.

## 配置步骤

1,各节点使用唯一server-id

2,都启用二进制日志[log-bin]和中继日志[relay-log]

3,创建拥有复制权限的用户账号

4,定义自增id的数值范围为奇偶

5,互为主备,启用复制线程

## 配置示例

masterA:172.16.1.6

masterB:172.16.1.7

简略架构图:

![img](MySQL%E5%A4%8D%E5%88%B6%E5%AE%9E%E4%BE%8B.assets/a9fd7d14-3adf-456e-a941-9d4b3deffe1a.png)

配置步骤

1,分别修改master节点的配置文件

```
# masterA my.cnf
[mysqld]
log-bin=master-bin
relay_log=relay-log
server-id = 6
innodb_file_pre_table=ON
skip_name_reslove=ON

auto_increment_offset = 1  # 起始值
auto_increment_increment = 2  # 步进值


# masterB my.cnf
[mysqld]
log-bin=master-bin
relay_log=relay-log
server-id = 7
innodb_file_pre_table=ON
skip_name_reslove=ON

auto_increment_offset = 2  # 起始值
auto_increment_increment = 2  # 步进值

 
# 重启mysql服务并验证是否生效
service mysqld restart
mysql -uroot -p
mysql> show global variables like '%log%';

```

在各master节点添加用于复制的账户

```
# masterA
mysql> grant replication slave,replication client on *.* to 'repluser'@'172.16.1.%' identified by 'replpass';
mysql> flush privileges;

mysql> show master status;
+------------------+----------+--------------+--------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB         |
+------------------+----------+--------------+--------------------------+
| mysql-bin.000008 |      191 |              | mysql,information_schema |
+------------------+----------+--------------+--------------------------+
1 row in set (0.00 sec)



mysql> change master to master_host='172.16.1.7',master_port=3306,master_user='repluser',master_password='replpass',master_log_file='mysql-bin.000008',master_log_pos=348;
mysql> start slave;
mysql> show slave status\G
"""
            Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
"""


# masterB
mysql> grant replication slave,replication client on *.* to 'repluser'@'172.16.1.%' identified by 'replpass';
mysql> flush privileges;

mysql> show master status;
+------------------+----------+--------------+--------------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB         |
+------------------+----------+--------------+--------------------------+
| mysql-bin.000008 |      348 |              | mysql,information_schema |
+------------------+----------+--------------+--------------------------+
1 row in set (0.00 sec)

mysql> change master to master_host='172.16.1.6',master_port=3306,master_user='repluser',master_password='replpass',master_log_file='mysql-bin.000008',master_log_pos=191;
mysql> start slave;
mysql> show slave status\G
"""
            Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
"""
```

测试

```
# masterA上:
mysql> create database db01;
mysql> use db01;
mysql> create table tb01 (id int unsigned not null auto_increment primary key, name char(30));
mysql> desc tb01;
mysql> show master status;


# masterB上:
mysql> show master status;
mysql> show databases;
mysql> show tables from db01;


mysql> use db01;
mysql> insert into tb01 (name) values ('Yang Kang'),('Yang Guo'),('Yang Yang');
mysql> select * from tb01;


# masterA上:
mysql> show master status;
mysql> select * from tb01;


mysql> insert into tb01 (name) values ('Zhu Yuanzhang'),('Zhu Di'),('Zhu Yue');
mysql> select * from tb01;
```

注意:

1,主主复制配置文件中auto_increment_increment和auto_increment_offset只能保证主键不重复,却不能保证主键有序;

2,当配置完成Slave_IO_Running,Slave_SQL_Running不全为YES时,show slave status\G信息中有错误提示,可根据错误提示进行更正;

3,Slave_IO_Running,Slave_SQL_Running不全为YES时,大多数问题都是数据不统一导致.

## 可能遇到的错误

两台数据库都存在db数据库,而第一台MySQL db中有tab1,第二台MySQL db中没有tab1,那肯定不能成功;已经获取了数据的二进制日志名和位置,又进行了数据操作,导致POS发生变更.在配置CHANGE MASTER时还是用到之前的POS;

```
# vim my.cnf
slave-skip-errors = 1032,1062,1007  # 忽略指定的错误更新

# 或
mysql> stop slave;
mysql> set global sql_slave_skip_counter = 1;  # 忽略执行N个更新
mysql> start slave;
```

终极更正法

重新执行一遍CHANGE MASTER就好了;

# 半同步复制示例

插件:

目录:mysql/lib/plugin/

semisync_master.so

semisync_slave.so

还是以上面主复制的示例,我们将mysql-2作为半同步复制.

```
# 先停掉mysql-2节点的slave线程
mysql> stop slave;

# master节点上装载插件semisync_master.so
mysql> install plugin rpl_semi_sync_master soname 'semisync_master.so';
mysql> show plugins;


# 在从节点mysql-2上装载插件semisync_slave.so
mysql> install plugin rpl_semi_sync_slave soname 'semisync_slave.so';
mysql> show plugins;
```

安装好插件以后,主节点和目标从节点均多了一些变量,用于控制半同步复制.

```
# 主节点mysql-1
mysql> show global variables like '%semi%';
"""
+------------------------------------+-------+
| Variable_name                      | Value |
+------------------------------------+-------+
| rpl_semi_sync_master_enabled       | OFF   |  # 是否启用半同步复制
| rpl_semi_sync_master_timeout       | 10000 |  # 等待从节点响应超时时间
| rpl_semi_sync_master_trace_level   | 32    |  # 跟踪节点
| rpl_semi_sync_master_wait_no_slave | ON    |  # 没有slave时是否等待
+------------------------------------+-------+
"""

mysql> show global status like '%semi%';
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 0     |
| Rpl_semi_sync_master_net_avg_wait_time     | 0     |
| Rpl_semi_sync_master_net_wait_time         | 0     |
| Rpl_semi_sync_master_net_waits             | 0     |
| Rpl_semi_sync_master_no_times              | 0     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | OFF   |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 0     |
| Rpl_semi_sync_master_tx_wait_time          | 0     |
| Rpl_semi_sync_master_tx_waits              | 0     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 0     |
+--------------------------------------------+-------+

mysql> set global rpl_semi_sync_master_enabled=1;  # 开启半同步复制
mysql> show global status like '%semi%';




# 从节点mysql-2
mysql> show global variables like '%semi%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| rpl_semi_sync_slave_enabled     | OFF   |
| rpl_semi_sync_slave_trace_level | 32    |
+---------------------------------+-------+

mysql> show global status like '%semi%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| Rpl_semi_sync_slave_status | OFF   |
+----------------------------+-------+

mysql> set global rpl_semi_sync_slave_enabled=1;  # 开启半同步复制
mysql> start slave;
mysql> show slave status\G
```

下面进行一下测试

```
# 主节点上测试
mysql> show databases;
mysql> create database db02;
mysql> use db02;
mysql> create table tb02 (id int, name char(30));
mysql> show global status like '%semi%';
```

## 复制过滤器

让从节点仅复制指定的库,或指定库中的指定表.

有2种实现方式

1,主服务器仅向二进制日志中记录与特定数据库(表)相关的事件;

可能出现的问题:

时间还原无法实现,不建议使用.

相关参数:

binlog_do_db=db1,db2,..  # 白名单

binlog_ignore_db=db1,db2,..  # 黑名单

binlog_do_table=tb1,tb2,..  # 白名单

binlog_ignore_table=tb1,tb2,..  # 黑名单

2,从服务器SQL_THREAD在replay(中继日志)中的事件时,仅读取与特定数据库(表)相关的事件并应用于本地.

可能出现的问题:

会造成网络及磁盘IO的浪费

相关参数:

replicate_do_db=db1,db2,..  # 白名单

replicate_ignore_db=db1,db2,..  # 黑名单

replicate_do_table=tb1,tb2,..  # 白名单

replicate_ignore_table=tb1,tb2,..  # 黑名单

## 基于ssl复制

```
mysql> help change;
mysql> show global variables like '%ssl%';
```

需要在编译时支持ssl

1,master配置证书和私钥,并且创建一个要求必须使用SSL连接的复制账号;

2,slave端使用change master to 命令时指明ssl相关选项;



跟复制功能相关的文件:

master.info:用于保存slave连接至master时的相关信息,例如账号,密码,服务器地址等;

relay-log.info:保存在当前slave节点上已经复制的当前二进制日志和本地replay log日志的对应关系;

复制的监控和维护:

1,清理日志

建议使用purge

mysql> help purge;可查看使用方式

2,复制监控

show master status;

show binlog events;

show binary logs;

show slave status;

show processlist;

3,从是否落后于主

show slave status;

查看此项:

seconds_behind_master: 0

4,如何确定主从节点数据一致

percona-tools工具

5,数据不一致如何解决

放弃从节点,重新复制