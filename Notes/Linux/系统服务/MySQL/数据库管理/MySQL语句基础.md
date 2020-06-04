[TOC]

# MySQL命令

mysql程序的类别：

服务器端程序：启动并监听于套接字上，如：mysqld，mysqld_safe，mysqld_multi等；

客户端程序：可通过mysql协议连入服务器并发出请求的程序，如：mysql，mysqlbinlog，mysqladmin，mysqldump等；

工具程序：运行于服务器程序所在的主机，实现一些管理或维护操作，如myisamchk

客户端程序通用选项：

-u，--user=

-h，--host=

-p，--password=

--protocol=tcp --port # ：使用tcp协议的#端口连接mysql

--protocol=socket

或--socket         ：unix或linux系统专用于本机进行socket间通信        

--protocol=pipe      ：windows专用

--protocol=memory   ：共享内存，windows专用

-D db_name：连接mysql数据库后使用db_name数据库为默认

--database=db_name

mysql客户端程序：mysql

  运行方式有两类：

  交互式模式：

  批模式：mysql < /path/from/somefile.sql

交互式模式命令有两类：

客户端命令：help可列出所有命令

  clear，\c：表示中止一个命令语句

  ego，\G：竖排显示一个行中的内容

  go，\g：可作为语句结束符

  delimiter，\d：定义命令结束符

  exit，quit，\q：退出

  source，\. /path/to/somefile.sql ：执行一个脚本

  system，\! COMMAND：在不退出mysql的前提下，运行系统的shell命令

  use，\u db_name：将指定的库设为默认库

\#mysql -e ‘mysql-command；’   不需连接至mysql直接执行mysql语句

服务器端命令：

  DDL，DML

  help COMMAND：查看COMMAND的帮助信息

mysqladmin

\#mysqladmin [options] command [ard] [command [arg]] ...

mysqladmin [options] create db_name    不需要连接mysql，直接创建一个数据库

mysqladmin [options] drop db_name     不需要连接mysql，直接删除一个数据库

mysqladmin 

  status   显示mysql服务器的简要状态信息

​     --sleep # ：间隔#秒显示一次

​     --count # ：显示#次

  extend-status：显示mysql的所有服务器的状态变量

  flush-privileges：刷新授权表

  flush-hosts：清除DNS缓存和被拒绝的客户端列表缓存

  flush-logs：滚动日志，中继日志和二进制日志

  flush-status：重置各状态变量

  flush-tables：关闭当前打开的所有的表文件句柄，如果有正在打开的表，需要等待表关闭

  flush-treads：重置线程

  ping：测试mysql服务器是否在线

  processlist：查看服务器线程列表

  refresh：相当于同时执行flush-hosts和flush-logs

  start-slave，stop-slave：启动，关闭从服务器线程

  variables：显示服务器变量

# SQL语句

## 书写规范

1 SQL关键字及函数名不区分字符大小写；

数据库、表、索引和视图的名称是否区分大小写取决于底层的os和fs；

存储过程、存储函数和事件调度器不区分字符大小写；但触发器区分大小写；

表的别名不区分大小写；

字段中的字符数据，类型为binary，blog，varbinary类型时区分大小写；其他的不区分；

建议命令大写，库表名小写；

SQL语句可单行或多行书写，以分号';'结尾,关键词不能跨多行或简写；

用空格和缩进来提高语句的可读性,子句通常位于独立行,便于编辑,提高可读性；

 

```
SELECT * FROM tb_table
    WHERE NAME="YUAN";
```

2 注释

单行注释：--

多行注释：/*......*/

3 常用快捷键

ctrl + a：快速移动光标至行首；

ctrl + e：移动至行尾；

ctrl + w：删除光标前的单词；

ctrl + u：删除行首至光标所有内容；

ctrl + y：粘贴ctrl+w或ctrl+u删除的内容

4 提示符

->: 续行

'>, ">, `>: 表示缺少后半部分符号

## SQL分类

1 数据定义语言(DDL)

数据定义语言Data Defination Language,用于创建、修改、和删除数据库内的数据结构，如CREATE, ALTER, DROP；

2 数据操作语言（DML）

数据操作语言Data Manapulace Language,修改数据库中的数据，如: CRUD增删改查；

3 数据控制语言(DCL)

数据控制语言Data Control Language,负责数据库权限访问控制,如GRANT, REVOKE；

4 事务控制语言(TCL)

负责处理ACID事务，支持commit、rollback指令；

# 用户管理

增删用户

 

```
# 查看已存在用户
> SELECT user,host,password FROM mysql.user;

# 创建用户user1
> CREATE USER xiaofei@'%' IDENTIFIED BY '123.com';

# 改名
RENAME USER old_user TO new_user;

# 删除用户, 两种方法
> DROP mysql.user 'xiaofei'@'%';
> DELETE FROM mysql.user WHERE user='xiaofei' AND host='%';
```

密码更改

 

```
# 更改当前登录用户密码
> SET PASSWORD = PASSWORD('123.com');

# 更改指定用户密码
> SET PASSWORD FOR 'xiaofei'@'%' = PASSWORD('123.com');

# 更改现有用户密码
> UPDATE USER SET PASSWORD=PASSWORD('123.com') WHERE user='xiaofei';
```

忘记密码

 

```
# 方法1
~]# vi /etc/my.cnf
"""
 [mysqld] 
 datadir=/var/lib/mysql 
 socket=/var/lib/mysql/mysql.sock 
 skip-grant-tables  # [mysqld]的段中加上一句：skip-grant-tables
"""
~]# /etc/init.d/mysqld restart 
# 然后可直接免密进入mysql,使用update更改密码后,在删除配置文件中添加的那条数据,重启服务即可


# 方法2
~]# /etc/init.d/mysqld stop
~]# mysqld_safe --skip-grant-tables --user=mysql
~]# mysql
> UPDATE mysql.user SET Password=PASSWORD("3306.com") WHERE user='root' AND host='localhost';
> FLUSH PRIVILEGES;
> \q

~]# mysqladmin -uroot -p shutdown
~]# /etc/init.d/mysqld start
~]# mysqld -uroot -p
```

mysql初始安全策略

1,设置或修改root密码

2,删除无用的mysql库内的用户账号

3,删除默认存在的test数据库

4,更严格的做法, 创建新的管理员账户,如:

 

```
mysql> GRANT ALL PRIVILEGES ON *.* TO system@'localhost' IDENTIFIED BY 'system.com' with grant option;  # 赋予管理员权限
```

添加完成后可以删掉或改名root用户

# 授权管理

GRANT授权, REVOKE撤销

 

```
# 授权用户xiaofei对所有库拥有所有权限(慎用)
> GRANT ALL PRIVILEGES ON *.* TO 'user2'@'%' IDENTIFIED BY 'user2pass';

# 授权指定权限
> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP ON blog.* TO 'user3'@'%' IDENTIFIED BY 'user3pass';

# 授权一个用户可给其他用户授权
> GRANT ALL PRIVILEGES ON d3306.* TO 'user2'@'%' IDENTIFIED BY 'user2pass' WITH GRANT OPTION;

# 撤销权限
> REVOKE ALL PRIVILEGES ON d3306.* FROM 'user2'@'%';

> FLUSH PRIVILEGES;
> SELECT user,host,password FROM mysql.user;

# 查看已分配权限
> SHOW GRANTS FOR 'user2'@'%';
```

ALL PRIVILEGES 可选择的权限如下：

1 管理类：

CREATE TEMPORARY TABLES   创建临时表

CREATE USER       创建用户

FILE         把数据保存在文件中或从一个文件中装载数据

SUPER         不便归类的管理类权限

SHOW DATABASES      查看数据库权限

RELOAD         重置

SHUTDOWN        关闭服务器权限

REPLICATION SLAVE     复制主从架构中的的从节点

REPLICATION CLIENT     在主从架构中可向主服务器发起请求的权限

LOCK TABLES       对表施加锁权限

PROCESS         线程权限

storage routine: 存储例程

​    storage procedure存储过程

​    storage function存储函数

2 库和表级别(DDL)：

ALTER: 修改表和索引。

CREATE: 创建数据库和表。

DELETE: 删除表中已有的记录。

DROP: 抛弃(删除)数据库和表。

INDEX: 创建或抛弃索引。

INSERT: 向表中插入新行。

REFERENCE: 未用。

SELECT: 检索表中的记录。

UPDATE: 修改现存表记录。

FILE: 读或写服务器上的文件。

PROCESS: 查看服务器中执行的线程信息或杀死线程。

RELOAD: 重载授权表或清空日志、主机缓存或表缓存。

SHUTDOWN: 关闭服务器。

ALL: 所有权限，ALL PRIVILEGES同义词。

USAGE: 特殊的 "无权限" 权限。

3 数据操作（表级别）：

SELECT

INSERT

UPDATE

DELETE

4 字段级别：

SELECT(col1,...)

UPDATE(col1,...)

INSERT(col1,...)

总结:

\- 用户账户包括 "username" 和 "host" 两部分;

\- 后者表示该用户被允许从何地接入,tom@'%' 表示任何地址，默认可以省略。还可以是 "tom@192.168.1.%"、"tom@%.abc.com" 等。

\- 数据库格式为 db@table，可以是 "test.*" 或 "*.*"，前者表示 test 数据库的所有表，后者表示所有数据库的所有表。

\- 子句"WITH GRANT OPTION"表示该用户可以为其他用户分配权限。 

@HOST: 表示允许连接本地的客户端范围

%:任意多个字符

_:任意单个字符

如：172.16.0.0/16, 172.16.%.%

# 配置文件



mysql的配置文件是分段式的，如[client]，[mysqld]，[mysql]，[mysqladmin]，[mysql_safe]等；

mysql的程序通常仅读取与它同名的分段部分，如服务器mysqld通常会读取[mysqld]分段下的相关配置项，client会读取[client]配置段中的配置项；

配置文件读取顺序：/etc/my.cnf，/etc/mysql/my.cnf，~/.my.cnf

可通过获取帮助查看：

\#mysqld --verbose --help

​       --defaults-file=#                ##让mysql仅读取此处指定的配置文件

​       --defaults-extra-file=#           ##额外读取的配置文件

查看mysql的一些全局变量设置

\#mysqladmin variables -p

或

\>SHOW GLOBAL variables；

~/.my.cnf配置文件

```
[client]user = roothost = localhostpassword = 123.com
```

可免密码登录mysql