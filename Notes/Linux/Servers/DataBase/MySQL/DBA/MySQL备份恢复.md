[TOC]

备份介绍

备份工具

备份

为什么要备份?

灾难恢复：硬件故障，软件故障，自然灾害，黑客攻击，误操作等

测试环境：备份还原测试

需要注意的要点：

可容忍丢失的数据多少

恢复数据所要求的时间

明确需要恢复什么

备份类型：

完全备份：备份所有库所有表所有数据;

部分备份：仅备份其中的一张表或多张表； 

增量备份：仅备份从上次完全备份或增量备份之后，变化的数据的部分；

差异备份：以第一次备份时间节点为标准，备份到当前时刻发生的所有变化数据；

备份方式:

热备份：在线备份，读写操作不受影响；(InnoDB基于快照进行备份)

温备份：在线备份，可读不可写；(MyISAM)

冷备份：离线备份，服务器在备份期间不再提供读写服务；

物理备份：直接复制数据文件；

逻辑备份：从数据库中‘导出’数据另存而进行的备份；

规划备份时需要考虑的因素：

-持锁的时长；

-备份过程时长；

-备份负载；

-恢复过程时长；

备份工具:

mysqldump,逻辑备份工具,适用于所有的存储引擎.可实现完全备份,部分备份,属于温备;对于InnoDB存储引擎可实现热备;

cp,tar等系统工具,适用于所有存储引擎,可实现完全备份和部分备份,属于冷备;

lvm2的快照,几乎相当于热备,借助于系统工具实现物理备份;

设计备份方案:

1,mysqldump + binlog：mysqldump完全备份,通过备份二进制日志实现增量备份；

2,lvm2快照 + binlog：几乎热备,物理备份

3,xtrabackup：InnoDB(支持热备,完全备份和增量备份)；MyISAM存储引擎(支持温备)

mysqldump

格式:

mysqldump [OPTIONS] db_name [tb_name]: 备份单个库或单个或多个指定表

mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...]：备份一个或多个库

mysqldump [OPTIONS] --all-databases [OPTIONS]：备份所有数据库

OPTIONS:

--all-databases , -A

导出全部数据库 

 

```
mysqldump -uroot -p --all-databases
```

--all-tablespaces , -Y

导出全部表空间

 

```
mysqldump -uroot -p --all-databases --all-tablespaces
```

--no-tablespaces , -y

不导出任何表空间信息

 

```
mysqldump -uroot -p --all-databases --no-tablespaces
```

--add-drop-database

每个数据库创建之前添加drop数据库语句

 

```
mysqldump -uroot -p --all-databases --add-drop-database
```

--add-drop-table

每个数据表创建之前添加drop数据表语句

默认为打开状态,使用 --skip-add-drop-table 取消选项 

 

```
# 默认添加drop语句
mysqldump -uroot -p --all-databases

# 取消drop语句
mysqldump -uroot -p --all-databases –skip-add-drop-table
```

--add-locks

在每个表导出之前增加LOCK TABLES并且之后UNLOCK TABLE 

默认为打开状态，使用 --skip-add-locks 取消选项

 

```
# 默认添加LOCK语句
mysqldump -uroot -p --all-databases

# 取消LOCK语句
mysqldump -uroot -p --all-databases –skip-add-locks
```

--allow-keywords

允许创建是关键词的列名字,这由表名前缀于每个列名做到 

 

```
mysqldump -uroot -p --all-databases --allow-keywords
```

--apply-slave-statements

在'CHANGE MASTER'前添加'STOP SLAVE'，并且在导出的最后添加'START SLAVE' 

 

```
mysqldump -uroot -p --all-databases --apply-slave-statements
```

--character-sets-dir

字符集文件的目录

 

```
mysqldump -uroot -p --all-databases --character-sets-dir=/usr/local/mysql/share/mysql/charsets
```

--comments

附加注释信息。默认为打开，可以用 --skip-comments 取消 

 

```
# 默认记录注释
mysqldump -uroot -p --all-databases

# 取消注释
mysqldump -uroot -p --all-databases --skip-comments
```

--compatible

导出的数据将和其它数据库或旧版本的MySQL相兼容.值可以为ansi、mysql323、mysql40、postgresql、oracle、mssql、db2、maxdb、no_key_options、no_tables_options、no_field_options等,要使用几个值,用逗号将它们隔开,它并不保证能完全兼容,而是尽量兼容. 

 

```
mysqldump -uroot -p --all-databases --compatible=ansi
```

--compact

导出更少的输出信息(用于调试),去掉注释和头尾等结构

可以使用选项：--skip-add-drop-table  --skip-add-locks --skip-comments --skip-disable-keys

 

```
mysqldump -uroot -p --all-databases --compact
```

--complete-insert, -c

使用完整的insert语句(包含列名称),这么做能提高插入效率,但是可能会受到max_allowed_packet参数的影响而导致插入失败

 

```
mysqldump -uroot -p --all-databases --complete-insert
```

--compress, -C

在客户端和服务器之间启用压缩传递所有信息 

 

```
mysqldump -uroot -p --all-databases --compress
```

--create-options,  -a

在CREATE TABLE语句中包括所有MySQL特性选项(默认为打开状态) 

 

```
mysqldump -uroot -p --all-databases
```

--databases,  -B

导出几个数据库。参数后面所有名字参量都被看作数据库名

 

```
mysqldump -uroot -p --databases test mysql
```

--debug

输出debug信息,用于调试,默认值为：d:t,/tmp/mysqldump.trace 

 

```
mysqldump -uroot -p --all-databases --debug
mysqldump -uroot -p --all-databases --debug="d:t,/tmp/debug.trace"
```

--debug-check

检查内存和打开文件使用说明并退出

 

```
mysqldump -uroot -p --all-databases --debug-check
```

--debug-info

输出调试信息并退出 

 

```
mysqldump -uroot -p --all-databases --debug-info
```

--default-character-set

设置默认字符集,默认值为utf8

 

```
mysqldump -uroot -p --all-databases --default-character-set=utf8
```

--delayed-insert

采用延时插入方式（INSERT DELAYED）导出数据 

 

```
mysqldump -uroot -p --all-databases --delayed-insert
```

--delete-master-logs

master备份后删除日志,这个参数将自动激活 --master-data

 

```
mysqldump -uroot -p --all-databases --delete-master-logs
```

--disable-keys

对于每个表,用/*!40000 ALTER TABLE tbl_name DISABLE KEYS */;和/*!40000 ALTER TABLE tbl_name ENABLE KEYS */;语句引用INSERT语句.这样可以更快地导入dump出来的文件，因为它是在插入所有行后创建索引的。该选项只适合MyISAM表，默认为打开状态。 

 

```
mysqldump -uroot -p --all-databases
```

--dump-slave

该选项将主的binlog位置和文件名追加到导出数据的文件中(show slave status)

设置为1时，将会以CHANGE MASTER命令输出到数据文件；设置为2时，会在change前加上注释。该选项将会打开--lock-all-tables，除非--single-transaction被指定。该选项会自动关闭--lock-tables选项。默认值为0

 

```
mysqldump -uroot -p --all-databases --dump-slave=1 
mysqldump -uroot -p --all-databases --dump-slave=2
```

--master-data

该选项将当前服务器的binlog的位置和文件名追加到输出文件中(show master status).如果为1，将会输出CHANGE MASTER 命令；如果为2，输出的CHANGE  MASTER命令前添加注释信息.

该选项将打开--lock-all-tables 选项，除非--single-transaction也被指定（在这种情况下，全局读锁在开始导出时获得很短的时间；其他内容参考下面的--single-transaction选项）。该选项自动关闭--lock-tables选项。 

 

```
mysqldump -uroot -p --host=localhost --all-databases --master-data=1; 
mysqldump -uroot -p --host=localhost --all-databases --master-data=2;
```

--events, -E  导出事件。 

 

```
mysqldump -uroot -p --all-databases --events
```

--extended-insert,  -e

使用具有多个VALUES列的INSERT语法。这样使导出文件更小，并加速导入时的速度。默认为打开状态，使用--skip-extended-insert取消选项

 

```
mysqldump -uroot -p --all-databases
mysqldump -uroot -p --all-databases--skip-extended-insert  # 取消选项
```

--fields-terminated-by

导出文件中忽略给定字段,与--tab选项一起使用,不能用于--databases和--all-databases选项

 

```
mysqldump -uroot -p test test --tab=”/home/mysql” --fields-terminated-by=”#”
```

--fields-enclosed-by

输出文件中的各个字段用给定字符包裹。与--tab选项一起使用，不能用于--databases和--all-databases选项

 

```
mysqldump -uroot -p test test --tab=”/home/mysql” --fields-enclosed-by=”#”-c
```

--fields-optionally-enclosed-by

输出文件中的各个字段用给定字符选择性包裹。与--tab选项一起使用，不能用于--databases和--all-databases选项

 

```
mysqldump -uroot -p test test --tab=”/home/mysql” --fields-enclosed-by=”#” --fields-optionally-enclosed-by =”#”
```

--fields-escaped-by

输出文件中的各个字段忽略给定字符。与--tab选项一起使用，不能用于--databases和--all-databases选项

 

```
mysqldump -uroot -p mysql user --tab=”/home/mysql” --fields-escaped-by=”#”
```

--flush-logs

开始导出之前刷新日志

请注意：假如一次导出多个数据库(使用选项--databases或者--all-databases)，将会逐个数据库刷新日志。除使用--lock-all-tables或者--master-data外。在这种情况下，日志将会被刷新一次，相应的所以表同时被锁定。因此，如果打算同时导出和刷新日志应该使用--lock-all-tables 或者--master-data 和--flush-logs

 

```
mysqldump -uroot -p --all-databases --flush-logs
```

--flush-privileges

在导出mysql数据库之后，发出一条FLUSH  PRIVILEGES 语句。为了正确恢复，该选项应该用于导出mysql数据库和依赖mysql数据库数据的任何时候

 

```
mysqldump -uroot -p --all-databases --flush-privileges
```

--force

在导出过程中忽略出现的SQL错误

 

```
mysqldump -uroot -p --all-databases --force
```

--help

显示帮助信息并退出

 

```
mysqldump --help
```

--hex-blob

使用十六进制格式导出二进制字符串字段。如果有二进制数据就必须使用该选项。影响到的字段类型有BINARY、VARBINARY、BLOB。 

 

```
mysqldump -uroot -p --all-databases --hex-blob
```

--host, -h

需要导出的主机信息

 

```
mysqldump -uroot -p --host=localhost --all-databases
```

--ignore-table

不导出指定表。指定忽略多个表时，需要重复多次，每次一个表。每个表必须同时指定数据库和表名。例如：--ignore-table=database.table1 --ignore-table=database.table2 …… 

 

```
mysqldump -uroot -p --host=localhost --all-databases --ignore-table=mysql.user
```

--include-master-host-port

在--dump-slave产生的'CHANGE  MASTER TO..'语句中增加'MASTER_HOST=<host>，MASTER_PORT=<port>'  

 

```
mysqldump -uroot -p --host=localhost --all-databases --include-master-host-port
```

--insert-ignore

在插入行时使用INSERT IGNORE语句. 

 

```
mysqldump -uroot -p --host=localhost --all-databases --insert-ignore
```

--lines-terminated-by

输出文件的每行用给定字符串划分。与--tab选项一起使用，不能用于--databases和--all-databases选项。 

 

```
mysqldump -uroot -p --host=localhost test test --tab=”/tmp/mysql” --lines-terminated-by=”##”
```

--lock-all-tables,  -x

提交请求锁定所有数据库中的所有表，以保证数据的一致性。这是一个全局读锁，并且自动关闭--single-transaction 和--lock-tables 选项

 

```
mysqldump -uroot -p --host=localhost --all-databases --lock-all-tables
```

--lock-tables,  -l

开始导出前，锁定所有表。用READ  LOCAL锁定表以允许MyISAM表并行插入。对于支持事务的表例如InnoDB和BDB，--single-transaction是一个更好的选择，因为它根本不需要锁定表

请注意当导出多个数据库时，--lock-tables分别为每个数据库锁定表。因此，该选项不能保证导出文件中的表在数据库之间的逻辑一致性。不同数据库表的导出状态可以完全不同.

 

```
mysqldump -uroot -p --host=localhost --all-databases --lock-tables
```

--log-error

附加警告和错误信息到给定文件 

 

```
mysqldump -uroot -p --host=localhost --all-databases --log-error=/tmp/mysqldump_error_log.err
```

--max_allowed_packet

服务器发送和接受的最大包长度

 

```
mysqldump -uroot -p --host=localhost --all-databases --max_allowed_packet=10240
```

--net_buffer_length

TCP/IP和socket连接的缓存大小

 

```
mysqldump -uroot -p --host=localhost --all-databases --net_buffer_length=1024
```

--no-autocommit

使用autocommit/commit 语句包裹表

 

```
mysqldump -uroot -p --host=localhost --all-databases --no-autocommit
```

--no-create-db,  -n

只导出数据，而不添加CREATE DATABASE 语句

 

```
mysqldump -uroot -p --host=localhost --all-databases --no-create-db
```

--no-create-info,  -t

只导出数据，而不添加CREATE TABLE 语句

 

```
mysqldump -uroot -p --host=localhost --all-databases --no-create-info
```

--no-data, -d

不导出任何数据，只导出数据库表结构

 

```
mysqldump -uroot -p --host=localhost --all-databases --no-data
```

--no-set-names,  -N

等同于--skip-set-charset 

 

```
mysqldump -uroot -p --host=localhost --all-databases --no-set-names
```

--opt

等同于--add-drop-table,  --add-locks, --create-options, --quick, --extended-insert, --lock-tables,  --set-charset, --disable-keys

该选项默认开启,可以用--skip-opt禁用

 

```
mysqldump -uroot -p --host=localhost --all-databases --opt
```

--order-by-primary

如果存在主键，或者第一个唯一键，对每个表的记录进行排序。在导出MyISAM表到InnoDB表时有效，但会使得导出工作花费很长时间

 

```
mysqldump -uroot -p --host=localhost --all-databases --order-by-primary
```

--password, -p

连接数据库密码

--pipe(windows系统可用)

使用命名管道连接mysql 

 

```
mysqldump -uroot -p --host=localhost --all-databases --pipe
```

--port, -P

连接数据库端口号

--protocol

使用的连接协议，包括：tcp, socket, pipe, memory. 

 

```
mysqldump -uroot -p --host=localhost --all-databases --protocol=tcp
```

--quick, -q

不缓冲查询，直接导出到标准输出。默认为打开状态，使用--skip-quick取消该选项。 

 

```
mysqldump -uroot -p --host=localhost --all-databases
mysqldump -uroot -p --host=localhost --all-databases --skip-quick
```

--quote-names,-Q

使用（`）引起表和列名,默认为打开状态，使用--skip-quote-names取消该选项。 

 

```
mysqldump -uroot -p --host=localhost --all-databases
mysqldump -uroot -p --host=localhost --all-databases --skip-quote-names
```

--replace

使用REPLACE INTO 取代INSERT INTO. 

 

```
mysqldump -uroot -p --host=localhost --all-databases --replace
```

--result-file,  -r

直接输出到指定文件中。该选项应该用在使用回车换行对（\\r\\n）换行的系统上（例如：DOS，Windows）。该选项确保只有一行被使用

 

```
mysqldump -uroot -p --host=localhost --all-databases --result-file=/tmp/mysqldump_result_file.txt
```

--routines, -R  导出存储过程以及自定义函数。 

 

```
mysqldump -uroot -p --host=localhost --all-databases --routines
```

--set-charset

添加'SET NAMES  default_character_set'到输出文件。默认为打开状态，使用--skip-set-charset关闭选项

 

```
mysqldump -uroot -p --host=localhost --all-databases
mysqldump -uroot -p --host=localhost --all-databases --skip-set-charset
```

--single-transaction

该选项在导出数据之前提交一个BEGIN SQL语句，BEGIN 不会阻塞任何应用程序且能保证导出时数据库的一致性状态。它只适用于多版本存储引擎，仅InnoDB。本选项和--lock-tables 选项是互斥的，因为LOCK  TABLES 会使任何挂起的事务隐含提交。要想导出大表的话，应结合使用--quick 选项。 

 

```
mysqldump -uroot -p --host=localhost --all-databases --single-transaction
```

--dump-date

将导出时间添加到输出文件中。默认为打开状态，使用--skip-dump-date关闭选项

 

```
mysqldump -uroot -p --host=localhost --all-databases
mysqldump -uroot -p --host=localhost --all-databases --skip-dump-date
```

--skip-opt

禁用–opt选项

 

```
mysqldump -uroot -p --host=localhost --all-databases --skip-opt
```

--socket,-S

指定连接mysql的socket文件位置，默认路径/tmp/mysql.sock 

 

```
mysqldump -uroot -p --host=localhost --all-databases --socket=/tmp/mysqld.sock
```

--tab,-T  为每个表在给定路径创建tab分割的文本文件

注意：仅仅用于mysqldump和mysqld服务器运行在相同机器上

注意使用--tab不能指定--databases参数 

 

```
mysqldump -uroot -p --host=localhost test test --tab="/home/mysql" 
```

--tables

覆盖--databases (-B)参数，指定需要导出的表名，在后面的版本会使用table取代tables

 

```
mysqldump -uroot -p --host=localhost --databases test --tables test
```

--triggers

导出触发器,该选项默认启用，用--skip-triggers禁用它

 

```
mysqldump -uroot -p --host=localhost --all-databases --triggers
```

--tz-utc

在导出顶部设置时区TIME_ZONE='+00:00',以保证在不同时区导出的 TIMESTAMP 数据或者数据被移动其他时区时的正确性

 

```
mysqldump -uroot -p --host=localhost --all-databases --tz-utc
```

--user, -u

指定连接的用户名

--verbose, --v

输出多种平台信息

--version, -V

输出mysqldump版本信息并退出

--where, -w

只转储给定的WHERE条件选择的记录。请注意如果条件包含命令解释符专用空格或字符，一定要将条件引用起来

 

```
mysqldump -uroot -p --host=localhost --all-databases --where=” user=’root’”
```

--xml, -X

导出XML格式

 

```
mysqldump -uroot -p --host=localhost --all-databases --xml
```

--plugin_dir

客户端插件的目录，用于兼容不同的插件版本。 

 

```
mysqldump -uroot -p --host=localhost --all-databases --plugin_dir=”/usr/local/lib/plugin”
```

--default_auth

客户端插件默认使用权限

 

```
mysqldump -uroot -p --host=localhost --all-databases --default-auth=”/usr/local/lib/plugin/<PLUGIN>”
```

mysqldump做备份还原时，因会产生二进制日志文件，所以需要先关闭日志记录功能

 

```
> SET SESSION sql_log_bin=0;
> SOURCE /path/from/somefile.sql;
> SET SESSION sql_log_bin=1;
```

物理备份：

注意保证数据文件的时间一致性

冷备：主从

类热备：lvm2快照

示例1:mysqldump逻辑备份工具

1,建立测试数据

 

```
> CREATE DATABASE db1 DEFAULT CHARSET utf8;
> USE db1;
> CREATE TABLE a1(id int);
> INSERT INTO a1() VALUES(1),(2);
> CREATE TABLE a2(id int);
> INSERT INTO a2() VALUES(2);
> CREATE INTO a3(id int);
> CREATE INTO a3() VALUES(3);

> CREATE DATABASE db2 DEFAULT CHARSET utf8;
> USE db2;
> CREATE TABLE b1(id int);
> INSERT INTO b1() VALUES(1);
> CREATE TABLE b2(id int);
> INSERT INTO b2() VALUE(2);
```

2,导出所有数据库

 

```
~]# mysqldump -uroot -p'123.com' --all-databases > db_bak/all.sql
```

3,导出db1,db2两个数据库中的所有数据

 

```
~]# mysqldump -uroot -p'123.com' --databases db1 db2 > db_bak/all_data.sql
```

说明: 默认不带参数的导出，导出文本内容大概如下：创建数据库判断语句-删除表-创建表-锁表-禁用索引-插入数据-启用索引-解锁表。

4,导出db1库中的a1,a2表

 

```
~]# mysqldump -uroot -p'123.com' --databases db1 --tables a1 a2 > db_bak/db1.sql
```

注意:导出指定表只能针对一个数据库进行导出，且导出的内容中和导出数据库也不一样，导出指定表的导出文本中没有创建数据库的判断语句，只有删除表-创建表-导入数据

5,条件导出,导出db1表中a1中id=1的数据,如果多个表的条件相同可以一次性导出多个表

 

```
# 字段是整型
~]# mysqldump -uroot -p'123.com' --databases db1 --tables a1 --where='id=1' > db_bak/a1-1.sql

# 字段是字符串,并且导出的sql中不包含drop table,create table
~]# mysqldump -uroot -p'123.com' --no-create-info --databases db1 --tables a1 --where="id='a'" > db_bak/a1-2.sql
```

6,生成新的binlog文件,有时候会希望导出数据之后生成一个新的binlog文件,只需要加上-F参数即可

 

```
~]# mysqldump -uroot -p'123.com' --databases db1 -F > db_bak/db1.sql
```

7,只导出表结构不导出数据

 

```
~]# mysqldump -uroot -p'123.com' --no-data --databases db1 > db_bak/db1.sql
```

8,跨服务器导入导出数据,将node1中的db1库中所有数据导入到node2中的db2库中

 

```
~]# mysqldump --host=node1 -uroot -p'123.com' --databases db1 | mysql --host=node2 -uroot -p'123.com' db2

# -C 压缩传输,提高传输速度
~]# mysqldump --host=172.16.100.10 -uroot -p'123.com' -C --databases db1 |mysql --host=172.16.100.20 -uroot -p'123.com' db2 
```

注意:node2中的db2库必须存在,否则报错

9,将主库的binlog位置和文件名追加到导出数据的文件中

 

```
~]# mysqldump -uroot -proot --dump-slave=1 --databases db1 > db_bak/db1.sql
# --dump-slave 该参数在在从服务器上执行，相当于执行show slave status;
# 当设置为1时，将会以CHANGE MASTER命令输出到数据文件;
# 设置为2时，会在change前加上注释;该选项将会打开--lock-all-tables，除非--single-transaction被指定;在执行完后会自动关闭--lock-tables选项;
# --dump-slave默认是1
```

注意：--dump-slave命令如果当前服务器是从服务器那么使用该命令会执行stop slave来获取master binlog的文件和位置，等备份完后会自动执行start slave启动从服务器。但是如果是大的数据量备份会给从和主的延时变的更大，使用--dump-slave获取到的只是当前的从服务器的数据执行到的主的binglog的位置是（relay_mater_log_file,exec_master_log_pos),而不是主服务器当前的binlog执行的位置，主要是取决于主从的数据延时.

10,将当前服务器的binlog的位置和文件名追加到输出文件

--master-data

该参数和--dump-slave方法一样，只是它是记录的是当前服务器的binlog，相当于执行show master status，状态（file,position)的值;

注意:--master-data不会停止当前服务器的主从服务.

11,保证导出的一致性状态

--single-transaction

该选项在导出数据之前提交一个BEGIN SQL语句，BEGIN 不会阻塞任何应用程序且能保证导出时数据库的一致性状态。它只适用于多版本存储引擎（它不显示加锁通过判断版本来对比数据），仅InnoDB。本选项和--lock-tables 选项是互斥的，因为LOCK TABLES 会使任何挂起的事务隐含提交。要想导出大表的话，应结合使用--quick 选项。

--quick, -q

不缓冲查询，直接导出到标准输出。默认为打开状态，使用--skip-quick取消该选项。

12,--opt

等同于 --add-drop-table,--add-locks,--create-options,--quick,--extended-insert,--lock-tables,--set-charset,--disable-keys

可用--skip-opt禁用

 

```
~]# mysqldump -uroot -p --host=localhost --all-databases --opt
```

13,导出存储过程和自定义函数

~]# mysqldump -uroot -p --host=localhost --all-databases --routines

14,压缩备份

 

```
# 压缩备份
~]# mysqldump -uroot -p --databases a 2>/dev/null | gzip > db_bak/a.sql.gz

# 还原
~]# mysqldump -c a.sql.gz | mysql -uroot -p
```

15,分库备份

分库备份实际上就是执行一个备份语句,可以备份多库. 比如数据库里面有多个库,想要单独备份一个库,需要执行多条命令,目的是每个库都有以自己库名命名的备份文件.

 

```
# mysql -uroot -p'123.com' -e 'SHOW DATABASES;' | grep -Evi 'database|info|perfor'| sed -r 's#^([a-z].*$)#mysqldump -uroot -p'123.com' -B \1 | gzip > /opt/bak/\1.sql.gz#g' | bash
```

意义: 有时候一个数据库里面会有多个库,但是出问题时可能是某一个库,如果在备份时将所有库备份成一个数据文件,恢复时就比较麻烦了,所以备份时分开来备份,方便日后恢复.

关键参数总结:

--compact 去掉注释,适合调试输出,生产不用

--master-data 增加binlog日志文件名及对应的位置点

-B 指定多个库,增加建库语句和use语句

-A 备份所有库

-D 只备份表结构

-F 刷新binlog日志

-t 只备份数据

-x --lock-all-tables 锁定表

--single-transaction 适合备份innodb事务数据库备份

myisam备份:

mysqldump -uroot -p'123.com' -A -B --master-data=2 --lock-all-tables --events | gzip > /backup/all.`date +%F`.sql.gz

innodb备份:

mysqldump -uroot -p'123.com' -A -B --master-data=2 --single-transaction --events | gzip > /backup/all.`date +%F`.sql.gz

示例2:实际场景备份策略

备份策略:备份mydb数据库，每周完全，每日增量，备份存放位置注意不要与数据库放在同一硬盘中。

 

```
# 完全备份
~]# mysqldump -B mydb -x --master-data=2 | gzip > /backup/mydb.`date +%F`.sql
~]# mysql
> CREATE TABLE db2 (ID INT);
> INSERT INTO db2 VALUES (1),(2),(33);

# 增量备份
~]# mysqlbinlog --start-datetime '2016-12-13 15:00:00' --stop-datetime '2016-12-13 15:30:00' /mydata/data/mysql-bin.0000* > /backup/incre-`date +%F`.sql

~]# mysql
# 在db2中插入新的数据
>INSERT INTO db2 VALUES (44),(555);

#模拟灾难，删除数据库mydb
>DROP DATABASE mydb;
```

数据恢复:

示例:

 

```
# 读取最近的二进制日志文件并保存到一个临时文件中123.sql
~]# mysqlbinlog /mydata/data/mysql-bin.000014 > /tmp/123.sql
~]# vim /tmp/123.sql
# 删除DROP DATABASE mydb 此行，保存退出
# 也可以先查看二进制日志文件，找出问题所在，然后定位到问题发生的前一刻，即at 853，然后把此事件的之前的信息保存到临时文件中

# 备份时间点文件
~]# mysqlbinlog --stop-position=853 /mydata/data/mysql-bin.000014 > /tmp/123.sql

# 接下来进行还原:
~]# mysql < /backup/mydb.`date +%F`.sql # 先导入完全备份
~]# mysql
> SHOW DATABASES;
> USE mydb;
> SHOW TABLES # 可查看mydb数据库依然恢复，但数据库中的表还没恢复

# mysql < /backup/incre-`date +%F`.sql  # 导入增量备份
> SHOW TABLES;   # 表已恢复，但表内的数据依旧未恢复
# mysql < /tmp/456.sql  # 时间点还原
> SELECT * FROM db2;  # 数据依然全部恢复
```

需要注意的是：

在时间点还原的时候，需要记得把DROP的那条记录删除掉，否则还原数据后到后来又会执行删除操作，如果导入时提示某个数据库或表已经存在，也可以把创建的那条语句删除，在导入。

使用lvm2快照进行备份

1，请求锁定所有表

 

```
> FLUSH TABLES WITH READ LOCK;
```

2，记录二进制日志文件及事件位置

 

```
> SHOW MASTER STATUS;
```

3，创建快照

 

```
# lvcreate -L SIZE -s -p r -n NAME /dev/VG_NAME/LV_NAME
```

4，施放锁

 

```
>UNLOCK TABLES;
```

5，挂在快照所在的卷，复制数据进行备份

使用cp，rsync，tar等命令复制数据

6，备份完成后，删除快照卷

使用Xtrabackup工具备份

1,简介

Xtrabackup是由percona提供的mysql数据库备份工具，据官方介绍，这也是世界上惟一一款开源的能够对innodb和xtradb数据库进行热备的工具。特点：

(1)备份过程快速、可靠；

(2)备份过程不会打断正在执行的事务；

(3)能够基于压缩等功能节约磁盘空间和流量；

(4)自动实现备份检验；

(5)还原速度快；

2,下载安装Xtrabackup-2.2.3-4982.el6.x86_64.rpm

官网地址：www.percona.com

依赖的包：libaio.x86_64,libaio-devel.x86_64,perl-DBD-MySQL,perl-Time-HiRes.x86_64

 

```
# rpm -ql percona-xtrabackup
/usr/bin/innobackupex
/usr/bin/xbcrypt
/usr/bin/xbstream
/usr/bin/xtrabackup
/usr/share/doc/percona-xtrabackup-2.2.3
/usr/share/doc/percona-xtrabackup-2.2.3/COPYING
```

1、完全备份

 

```
# innobackupex --user=DBUSER --password=DBUSERPASS  /path/to/BACKUP-DIR/
```

如果要使用一个最小权限的用户进行备份，则可基于如下命令创建此类用户：

 

```
mysql> CREATE USER ’bkpuser’@’localhost’ IDENTIFIED BY ’s3cret’;
mysql> REVOKE ALL PRIVILEGES, GRANT OPTION FROM ’bkpuser’;
mysql> GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO ’bkpuser’@’localhost’;
mysql> FLUSH PRIVILEGES;
```

使用innobakupex备份时，其会调用xtrabackup备份所有的InnoDB表，复制所有关于表结构定义的相关文件(.frm)、以及MyISAM、MERGE、CSV和ARCHIVE表的相关文件，同时还会备份触发器和数据库配置信息相关的文件。这些文件会被保存至一个以时间命名的目录中。

在备份的同时，innobackupex还会在备份目录中创建如下文件：

(1)xtrabackup_checkpoints —— 备份类型（如完全或增量）、备份状态（如是否已经为prepared状态）和LSN(日志序列号)范围信息；每个InnoDB页(通常为16k大小)都会包含一个日志序列号，即LSN。LSN是整个数据库系统的系统版本号，每个页面相关的LSN能够表明此页面最近是如何发生改变的。

(2)xtrabackup_binlog_info —— mysql服务器当前正在使用的二进制日志文件及至备份这一刻为止二进制日志事件的位置。

(3)xtrabackup_binlog_pos_innodb —— 二进制日志文件及用于InnoDB或XtraDB表的二进制日志文件的当前position。

(4)xtrabackup_binary —— 备份中用到的xtrabackup的可执行文件；

(5)backup-my.cnf —— 备份命令用到的配置选项信息；

在使用innobackupex进行备份时，还可以使用--no-timestamp选项来阻止命令自动创建一个以时间命名的目录；如此一来，innobackupex命令将会创建一个BACKUP-DIR目录来存储备份数据。

2、准备(prepare)一个完全备份

一般情况下，在备份完成后，数据尚且不能用于恢复操作，因为备份的数据中可能会包含尚未提交的事务或已经提交但尚未同步至数据文件中的事务。因此，此时数据文件仍处理不一致状态。“准备”的主要作用正是通过回滚未提交的事务及同步已经提交的事务至数据文件也使得数据文件处于一致性状态。

innobakupex命令的--apply-log选项可用于实现上述功能。如下面的命令：

\# innobackupex --apply-log  /path/to/BACKUP-DIR

如果执行正确，其最后输出的几行信息通常如下：

xtrabackup: starting shutdown with innodb_fast_shutdown = 1

120407  9:01:36  InnoDB: Starting shutdown...

120407  9:01:40  InnoDB: Shutdown completed; log sequence number 92036620

120407 09:01:40  innobackupex: completed OK!

在实现“准备”的过程中，innobackupex通常还可以使用--use-memory选项来指定其可以使用的内存的大小，默认通常为100M。如果有足够的内存可用，可以多划分一些内存给prepare的过程，以提高其完成速度。

3、从一个完全备份中恢复数据 

注意：恢复不用启动MySQL

innobackupex命令的--copy-back选项用于执行恢复操作，其通过复制所有数据相关的文件至mysql服务器DATADIR目录中来执行恢复过程。innobackupex通过backup-my.cnf来获取DATADIR目录的相关信息。

\# innobackupex --copy-back  /path/to/BACKUP-DIR

如果执行正确，其输出信息的最后几行通常如下：

innobackupex: Starting to copy InnoDB log files

innobackupex: in '/backup/2012-04-07_08-17-03'

innobackupex: back to original InnoDB log directory '/mydata/data'

innobackupex: Finished copying back files.

120407 09:36:10  innobackupex: completed OK!

请确保如上信息的最行一行出现“innobackupex: completed OK!”。

当数据恢复至DATADIR目录以后，还需要确保所有数据文件的属主和属组均为正确的用户，如mysql，否则，在启动mysqld之前还需要事先修改数据文件的属主和属组。如：

\# chown -R  mysql:mysql  /mydata/data

4、使用innobackupex进行增量备份

每个InnoDB的页面都会包含一个LSN信息，每当相关的数据发生改变，相关的页面的LSN就会自动增长。这正是InnoDB表可以进行增量备份的基础，即innobackupex通过备份上次完全备份之后发生改变的页面来实现。

要实现第一次增量备份，可以使用下面的命令进行：

 

```
# innobackupex --incremental /backup --incremental-basedir=BASEDIR
```

其中，BASEDIR指的是完全备份所在的目录，此命令执行结束后，innobackupex命令会在/backup目录中创建一个新的以时间命名的目录以存放所有的增量备份数据。另外，在执行过增量备份之后再一次进行增量备份时，其--incremental-basedir应该指向上一次的增量备份所在的目录。

需要注意的是，增量备份仅能应用于InnoDB或XtraDB表，对于MyISAM表而言，执行增量备份时其实进行的是完全备份。

“准备”(prepare)增量备份与整理完全备份有着一些不同，尤其要注意的是：

(1)需要在每个备份(包括完全和各个增量备份)上，将已经提交的事务进行“重放”。“重放”之后，所有的备份数据将合并到完全备份上。

(2)基于所有的备份将未提交的事务进行“回滚”。

于是，操作就变成了：

 

```
# innobackupex --apply-log --redo-only BASE-DIR
接着执行：
# innobackupex --apply-log --redo-only BASE-DIR --incremental-dir=INCREMENTAL-DIR-1
而后是第二个增量：
# innobackupex --apply-log --redo-only BASE-DIR --incremental-dir=INCREMENTAL-DIR-2
```

其中BASE-DIR指的是完全备份所在的目录，而INCREMENTAL-DIR-1指的是第一次增量备份的目录，INCREMENTAL-DIR-2指的是第二次增量备份的目录，其它依次类推，即如果有多次增量备份，每一次都要执行如上操作；

5、Xtrabackup的“流”及“备份压缩”功能

Xtrabackup对备份的数据文件支持“流”功能，即可以将备份的数据通过STDOUT传输给tar程序进行归档，而不是默认的直接保存至某备份目录中。要使用此功能，仅需要使用--stream选项即可。如：

 

```
# innobackupex --stream=tar  /backup | gzip > /backup/`date +%F_%H-%M-%S`.tar.gz

# 甚至也可以使用类似如下命令将数据备份至其它服务器：
# innobackupex --stream=tar  /backup | ssh user@www.magedu.com  "cat -  > /backups/`date +%F_%H-%M-%S`.tar" 
```

此外，在执行本地备份时，还可以使用--parallel选项对多个文件进行并行复制。此选项用于指定在复制时启动的线程数目。当然，在实际进行备份时要利用此功能的便利性，也需要启用innodb_file_per_table选项或共享的表空间通过innodb_data_file_path选项存储在多个ibdata文件中。对某一数据库的多个文件的复制无法利用到此功能。其简单使用方法如下：

 

```
# innobackupex --parallel  /path/to/backup

# 同时，innobackupex备份的数据文件也可以存储至远程主机，这可以使用--remote-host选项来实现：
# innobackupex --remote-host=root@www.magedu.com  /path/IN/REMOTE/HOST/to/backup
```

6、导入或导出单张表

默认情况下，InnoDB表不能通过直接复制表文件的方式在mysql服务器之间进行移植，即便使用了innodb_file_per_table选项。而使用Xtrabackup工具可以实现此种功能，不过，此时需要“导出”表的mysql服务器启用了innodb_file_per_table选项（严格来说，是要“导出”的表在其创建之前，mysql服务器就启用了innodb_file_per_table选项），并且“导入”表的服务器同时启用了innodb_file_per_table和innodb_expand_import选项。

(1)“导出”表

导出表是在备份的prepare阶段进行的，因此，一旦完全备份完成，就可以在prepare过程中通过--export选项将某表导出了：

\# innobackupex --apply-log --export /path/to/backup

此命令会为每个innodb表的表空间创建一个以.exp结尾的文件，这些以.exp结尾的文件则可以用于导入至其它服务器。

(2)“导入”表

要在mysql服务器上导入来自于其它服务器的某innodb表，需要先在当前服务器上创建一个跟原表表结构一致的表，而后才能实现将表导入：

mysql> CREATE TABLE mytable (...)  ENGINE=InnoDB;

然后将此表的表空间删除：

mysql> ALTER TABLE mydatabase.mytable  DISCARD TABLESPACE;

接下来，将来自于“导出”表的服务器的mytable表的mytable.ibd和mytable.exp文件复制到当前服务器的数据目录，然后使用如下命令将其“导入”：

mysql> ALTER TABLE mydatabase.mytable  IMPORT TABLESPACE;

7、使用Xtrabackup对数据库进行部分备份

Xtrabackup也可以实现部分备份，即只备份某个或某些指定的数据库或某数据库中的某个或某些表。但要使用此功能，必须启用innodb_file_per_table选项，即每张表保存为一个独立的文件。同时，其也不支持--stream选项，即不支持将数据通过管道传输给其它程序进行处理。

此外，还原部分备份跟还原全部数据的备份也有所不同，即你不能通过简单地将prepared的部分备份使用--copy-back选项直接复制回数据目录，而是要通过导入表的方向来实现还原。当然，有些情况下，部分备份也可以直接通过--copy-back进行还原，但这种方式还原而来的数据多数会产生数据不一致的问题，因此，无论如何不推荐使用这种方式。

(1)创建部分备份

创建部分备份的方式有三种：正则表达式(--include), 枚举表文件(--tables-file)和列出要备份的数据库(--databases)。

(a)使用--include

使用--include时，要求为其指定要备份的表的完整名称，即形如databasename.tablename，如：

\# innobackupex --include='^mageedu[.]tb1'  /path/to/backup

(b)使用--tables-file

此选项的参数需要是一个文件名，此文件中每行包含一个要备份的表的完整名称；如：

\# echo -e 'mageedu.tb1\nmageedu.tb2' > /tmp/tables.txt

\# innobackupex --tables-file=/tmp/tables.txt  /path/to/backup

(c)使用--databases

此选项接受的参数为数据名，如果要指定多个数据库，彼此间需要以空格隔开；同时，在指定某数据库时，也可以只指定其中的某张表。此外，此选项也可以接受一个文件为参数，文件中每一行为一个要备份的对象。如：

\# innobackupex --databases="mageedu testdb"  /path/to/backup

(2)整理(preparing)部分备份

prepare部分备份的过程类似于导出表的过程，要使用--export选项进行：

\# innobackupex --apply-log --export  /pat/to/partial/backup

此命令执行过程中，innobackupex会调用xtrabackup命令从数据字典中移除缺失的表，因此，会显示出许多关于“表不存在”类的警告信息。同时，也会显示出为备份文件中存在的表创建.exp文件的相关信息。

(3)还原部分备份

还原部分备份的过程跟导入表的过程相同。当然，也可以通过直接复制prepared状态的备份直接至数据目录中实现还原，不要此时要求数据目录处于一致状态。

注意：

1，将数据和二进制文件放置不同的设备，二进制日志也应该周期性的备份；

2，将数据和备份分开存放，建议不再同一设备，同一主机，同一机房，同一地域；

3，每次灾难恢复后都应该立即做一次完全备份；

4，备份后的数据应该周期性的做还原测试；

从备份中恢复应该遵循的步骤：

1，停止mysql服务器；

2，记录服务器配置和文件权限；

3，将备份恢复到mysql数据目录，此步骤依赖具体的备份工具；

4，改变配置和文件权限；

5，以限制方式启动mysql，进行测试；

  限制方式：如禁止通过网络访问等，可以在[mysql]配置段中添加以下两项：

  skip-networking

  socket=/tmp/mysql-recovery.sock

6，载入额外的逻辑备份，而检查和重放二进制日志；

7，以完全访问模式重启服务器；

实例：

1，先做一次完全备份，保存到/backup/目录下

```
# innobackupex --user=root --password=123.com /backup/
# ls /backup/
   2016-12-14_13-22-55
# cat 2016-12-14_13-22-55/xtrabackup_checkpoints 
   backup_type = full-backuped
   from_lsn = 0
   to_lsn = 1823859
   last_lsn = 1823859
   compact = 0
```

2，修改一下mysql数据，并做第一次增量备份

```
#mysql
>CREATE TABLE db3 (ID INT);
>INSERT INTO db3 VALUES (11111),(22222),(33333);
>\q
# innobackupex --incremental /backup/ --incremental-basedir=/backup/2016-12-14_13-22-55/
# ls /backup/
  2016-12-14_13-22-55  2016-12-14_13-33-50
# cat 2016-12-14_13-33-50/xtrabackup_checkpoints 
  backup_type = incremental
  from_lsn = 1823859
  to_lsn = 1829325
  last_lsn = 1829325
  compact = 0
```

3,再次修改mysql数据，做第二次增量备份

```
# mysql
>DROP TABLE db2；
>INSERT INTO db3 VALUES (88888);
>\q
# innobackupex --incremental /backup/ --incremental-basedir=/backup/2016-12-14_13-33-50/
# ls
  2016-12-14_13-22-55  2016-12-14_13-33-50  2016-12-14_13-41-10
# cat 2016-12-14_13-41-10/xtrabackup_checkpoints 
backup_type = incremental
from_lsn = 1829325
to_lsn = 1831977
last_lsn = 1831977
compact = 0
```

4,最后在修改一次mysql数据，但不要做备份，然后模拟mysql数据丢失损坏来进行恢复

```
# mysql
> INSERT INTO mydb.db3 VALUES (99999);
>\q
# service mysql stop
# cat /backup/2016-12-14_13-41-10/xtrabackup_binlog_info 
  mysql-bin.000015  1269
# mysqlbinlog --start-position=1269 /mydata/data/mysql-bin.000015 > /tmp/timepoint.sql
# rm -rf /mydata/data/*
```

5,还原数据方式：第一次完全备份+第一次增量备份+第二次增量备份+最后一次二进制日志文件

```
# innobackupex --apply-log --redo-only /backup/2016-12-14_13-22-55/
# innobackupex --apply-log --redo-only /backup/2016-12-14_13-22-55/ --incremental-dir=/backup/2016-12-14_13-33-50/
# innobackupex --apply-log --redo-only /backup/2016-12-14_13-22-55/ --incremental-dir=/backup/2016-12-14_13-41-10/
# innobackupex --copy-back /backup/2016-12-14_13-22-55/
# ls /mydata/data/
  ibdata1  mydb  mysql  performance_schema  xtrabackup_info
# chown -R mysql.mysql ./*
# service mysqld start
  Starting MySQL... SUCCESS!
# mysql
>USE mydb;
>SHOW TABLES;
>SET SESSION sql_log_bin=0;
>SOURCE /tmp/timepoint.sql;
>SET SESSION sql_log_bin=1;
>SHOW TABLES;
>SELECT * FROM db3;
```

6,至此数据已完全恢复，注意恢复完成以后，应先做一次完全备份。

```
# innobackupex --user=root --password=123.com /backup/
```

总结:

按天备

按周备