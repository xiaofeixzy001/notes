[TOC]

## 一，mysql-mha环境准备

### 1.1 实验环境：

| 主机名     | IP地址（NAT）     | 描述                                        |
| ---------- | ----------------- | ------------------------------------------- |
| mysql-db01 | eth0:192.168.0.51 | 系统：CentOS6.5（6.x都可以） 安装：mysql5.6 |
| mysql-db02 | eth0:192.168.0.52 | 系统：CentOS6.5（6.x都可以） 安装：mysql5.6 |
| mysql-db03 | eth0:192.168.0.53 | 系统：CentOS6.5（6.x都可以） 安装：mysql5.6 |

### 1.2 软件包

**1） mha管理节点安装包：**

mha4mysql-manager-0.56-0.el6.noarch.rpm

mha4mysql-manager-0.56.tar.gz

**2） mha node节点安装包：**

mha4mysql-node-0.56-0.el6.noarch.rpm

mha4mysql-node-0.56.tar.gz

**3） mysql中间件：**

Atlas-2.2.1.el6.x86_64.rpm

**4） mysql源码安装包**

mysql-5.6.17-linux-glibc2.5-x86_64.tar

### 1.3 主机名映射

```
[root@mysql-db01 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.0.51 mysql-db01
192.168.0.52 mysql-db02
192.168.0.53 mysql-db03
```

### 1.4 关闭selinux和iptables

```
[root@mysql-db01 ~]# vim /etc/sysconfig/selinux 
[root@mysql-db01 ~]# cat /etc/sysconfig/selinux | grep -v "#"

SELINUX=disabled
SELINUXTYPE=targeted 

[root@mysql-db01 ~]# setenforce 0
[root@mysql-db01 ~]# service iptables stop
iptables: Setting chains to policy ACCEPT: filter          [  OK  ]
iptables: Flushing firewall rules:                         [  OK  ]
iptables: Unloading modules:                               [  OK  ]
[root@mysql-db01 ~]# chkconfig iptables off
```

## 二，简介

### 2.1 作者简介

![QQ截图20170903134026.png-65.9kB](http://static.zybuluo.com/chensiqi/76yuakera3f8cvqaikdxwthv/QQ%E6%88%AA%E5%9B%BE20170903134026.png)

姓名：松信嘉范
 MySQL/Linux专家
 2001年索尼公司入职
 2001年开始使用oracle
 2004年开始使用MySQL
 2006年9月-2010年8月MySQL从事顾问
 2010年-2012年DeNA
 2012年至今Facebook

### 2.2 软件简介

> - MHA（Master High  Availability）目前在MySQL高可用方面是一个相对成熟的解决方案，是一套优秀的作为MySQL高可用性环境下故障切换和主从提升的高可用软件。在MySQL故障切换过程中，MHA能做到0~30秒之内自动完成数据库的故障切换操作，并且在进行故障切换过程中，MHA能最大程度上保证数据库的一致性，以达到真正意义上的高可用。
> - MHA由两部分组成：MHA Manager（管理节点）和MHA Node（数据节点）。MHA Manager可以独立部署在一台独立的机器上管理多个Master-Slave集群，也可以部署在一台Slave上。**当Master出现故障时，它可以自动将最新数据的Slave提升为新的Master，然后将所有其他的Slave重新指向新的Master。**整个故障转移过程对应程序是完全透明的。

### 2.3 工作流程

- 从宕机崩溃的master保存二进制日志事件（binlog events）；
- 识别含有最新更新的slave；
- 应用差异的中继日志（relay log）到其他的slave；
- 应用从master保存的二进制日志事件（binlog events）；
- 提升一个slave为新的master；
- 使其他的slave连接新的master进行复制；

### 2.4 MHA架构图

![QQ截图20170903190825.png-18.7kB](http://static.zybuluo.com/chensiqi/1rcy63r21ssvl3cd2dkm2ii8/QQ%E6%88%AA%E5%9B%BE20170903190825.png)

### 2.5 MHA工具介绍

MHA软件由两部分组成，Manager工具包和Node工具包，具体的说明如下：

```
#Manager工具包主要包括以下几个工具：

masterha_check_ssh          #检查MHA的SSH配置状况
masterha_check_repl         #检查MySQL复制状况
masterha_check_status       #检测当前MHA运行状态
masterha_master_monitor     #检测master是否宕机
masterha_manger             #启动MHA
masterha_master_switch      #控制故障转移（自动或者手动）
masterha_conf_host          #添加或删除配置的server信息
masterha_secondary_check    #试图建立TCP连接从远程服务器
masterha_stop               #停止MHA

#Node工具包主要包括以下几个工具：

save_binary_logs            #保存和复制master的二进制日志
apply_diff_relay_logs       #识别差异的中继日志事件
filter_mysqlbinlog          #去除不必要的ROLLBACK事件
purge_relay_logs            #清除中继日志
```

## 三，mysql环境准备

### 3.1 环境检查

- mysql-db01

```
#系统版本
[root@mysql-db01 bin]# cat /etc/redhat-release 
CentOS release 6.5 (Final)
[root@mysql-db01 bin]# uname -r
2.6.32-431.el6.x86_64
[root@mysql-db01 bin]# hostname -I
192.168.0.51 
```

- mysql-db02

```
#系统版本
[root@mysql-db02 ~]# cat /etc/redhat-release 
CentOS release 6.5 (Final)
[root@mysql-db02 ~]# uname -r
2.6.32-431.el6.x86_64
[root@mysql-db02 ~]# hostname -I
192.168.0.52 
```

- mysql-db03

```
#系统版本
[root@mysql-db03 ~]# cat /etc/redhat-release 
CentOS release 6.5 (Final)
[root@mysql-db03 ~]# uname -r
2.6.32-431.el6.x86_64
[root@mysql-db03 ~]# hostname -I
192.168.0.53 
```

### 3.2 安装mysql

#### 3.2.1 安装包准备

```
[root@mysql-db01 ~]# ls
anaconda-ks.cfg     mha4mysql-manager-0.56.tar.gz              rpm
install.log         mha4mysql-node-0.56.tar.gz
install.log.syslog  mysql-5.6.17-linux-glibc2.5-x86_64.tar.gz
[root@mysql-db01 ~]# ll mysql-5.6.17-linux-glibc2.5-x86_64.tar.gz 
-rw-r--r--. 1 root root 305102088 Sep  3 21:33 mysql-5.6.17-linux-glibc2.5-x86_64.tar.gz
```

#### 3.2.2 安装（3台都装）

```
[root@mysql-db01 ~]# yum -y install ncurses-devel
[root@mysql-db01 ~]# yum -y install libaio
[root@mysql-db01 ~]# tar xf mysql-5.6.17-linux-glibc2.5-x86_64.tar.gz -C /usr/local/
[root@mysql-db01 ~]# ln -s /usr/local/mysql-5.6.17-linux-glibc2.5-x86_64 /usr/local/mysql
[root@mysql-db01 ~]# useradd mysql -s /sbin/nologin -M
[root@mysql-db01 ~]# /usr/local/mysql/scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data/
[root@mysql-db01 ~]# /bin/cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf
[root@mysql-db01 ~]# /bin/cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
[root@mysql-db01 ~]# ln -s /usr/local/mysql/bin/* /usr/local/bin/
[root@mysql-db01 ~]# which mysqladmin
/usr/local/bin/mysqladmin
```

#### 3.2.3 加入开机自启动并启动mysql

```
[root@mysql-db01 ~]# chkconfig mysqld on
[root@mysql-db01 ~]# chkconfig mysqld --list
mysqld          0:off   1:off   2:on    3:on    4:on    5:on    6:off
[root@mysql-db01 ~]# /etc/init.d/mysqld start
Starting MySQL. SUCCESS! 
```

#### 3.2.4 配置密码

```
[root@mysql-db01 ~]# mysqladmin -uroot password '123123'
```

## 四，配置基于GTID的主从复制

### 4.1 先决条件

- 主库和从库都要开启binlog
- 主库和从库server-id不同
- 要有主从复制用户

### 4.2 主库操作（mysql-db01）

#### 4.2.1 修改配置文件

```
#修改主库配置文件/etc/my.cnf
[root@mysql-db01 mysql]# cat /etc/my.cnf 
[client]
socket          = /usr/local/mysql/data/mysql.sock
[mysqld]
lower_case_table_names  = 1
default-storage-engine  = InnoDB
port            = 3306
datadir         = /usr/local/mysql/data
character-set-server    = utf8
socket          = /usr/local/mysql/data/mysql.sock

log_bin         = mysql-bin     #开启binlog日志
server_id       = 1             #设置server_id

innodb_buffer_pool_size = 200M
slave-parallel-workers  = 8
thread_cache_size   = 600
back_log        = 600
slave_net_timeout   = 60
max_binlog_size     = 512M
key_buffer_size     = 8M
query_cache_size    = 64M
join_buffer_size    = 2M
sort_buffer_size    = 2M
query_cache_type    = 1
thread_stack        = 192K

#重启动MySQL服务
[root@mysql-db01 mysql]# /etc/init.d/mysqld restart
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS! 
```

#### 4.2.2 登陆MySQL删除不必要的用户并创建主从复制用户

**1）删除不必要的用户**

```
 mysql> select user,host from mysql.user;
+------+------------+
| user | host       |
+------+------------+
| root | 127.0.0.1  |
| root | ::1        |
|      | localhost  |
| root | localhost  |
|      | mysql-db01 |
| root | mysql-db01 |
+------+------------+
6 rows in set (0.00 sec)

mysql> drop user root@'127.0.0.1';
Query OK, 0 rows affected (0.00 sec)

mysql> drop user root@'::1';
Query OK, 0 rows affected (0.00 sec)

mysql> drop user ' '@'localhost';
Query OK, 0 rows affected (0.00 sec)

mysql> drop user ' '@'mysql-db01';
Query OK, 0 rows affected (0.00 sec)

mysql> select user,host from mysql.user;
+------+------------+
| user | host       |
+------+------------+
| root | localhost  |
| root | mysql-db01 |
+------+------------+
2 rows in set (0.00 sec)
```

**2）创建主从复制用户**

```
mysql> grant replication slave on *.* to rep@'192.168.0.%' identified by '123123';
Query OK, 0 rows affected (0.00 sec)

mysql> select user,host from mysql.user;
+------+-------------+
| user | host        |
+------+-------------+
| rep  | 192.168.0.% |
| root | localhost   |
| root | mysql-db01  |
+------+-------------+
3 rows in set (0.00 sec)

mysql> show grants for rep@'192.168.0.%';
+--------------------------------------------------------------------------------------------------------------------------+
| Grants for rep@192.168.0.%                                                                                               |
+--------------------------------------------------------------------------------------------------------------------------+
| GRANT REPLICATION SLAVE ON *.* TO 'rep'@'192.168.0.%' IDENTIFIED BY PASSWORD '*E56A114692FE0DE073F9A1DD68A00EEB9703F3F1' |
+--------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

### 4.3 从库操作(mysql-db02和mysql-db03)

#### 4.3.1 修改配置文件

```
#修改mysql-db02配置文件(和mysql-db01配置文件一致)
#只需要修改server-id = 5选项
[root@mysql-db02 ~]# cat /etc/my.cnf 
[client]
socket          = /usr/local/mysql/data/mysql.sock
[mysqld]
lower_case_table_names  = 1
default-storage-engine  = InnoDB
port            = 3306
datadir         = /usr/local/mysql/data
character-set-server    = utf8
socket          = /usr/local/mysql/data/mysql.sock

log_bin         = mysql-bin         #从binlog也要打开
server_id       = 5                 #仅需修改此项

innodb_buffer_pool_size = 200M
slave-parallel-workers  = 8
thread_cache_size   = 600
back_log        = 600
slave_net_timeout   = 60
max_binlog_size     = 512M
key_buffer_size     = 8M
query_cache_size    = 64M
join_buffer_size    = 2M
sort_buffer_size    = 2M
query_cache_type    = 1
thread_stack        = 192K
[root@mysql-db02 ~]# /etc/init.d/mysqld restart #重启mysql
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS!


#修改mysql-db03配置文件(和mysql-db01配置文件一致)
#只需要修改server-id = 10选项
[root@mysql-db03 ~]# cat /etc/my.cnf
[client]
socket          = /usr/local/mysql/data/mysql.sock
[mysqld]
lower_case_table_names  = 1
default-storage-engine  = InnoDB
port            = 3306
datadir         = /usr/local/mysql/data
character-set-server    = utf8
socket          = /usr/local/mysql/data/mysql.sock

log_bin         = mysql-bin     #从binlog也要打开
server_id       = 10            #只需修改此项

innodb_buffer_pool_size = 200M
slave-parallel-workers  = 8
thread_cache_size   = 600
back_log        = 600
slave_net_timeout   = 60
max_binlog_size     = 512M
key_buffer_size     = 8M
query_cache_size    = 64M
join_buffer_size    = 2M
sort_buffer_size    = 2M
query_cache_type    = 1
thread_stack        = 192K
[root@mysql-db03 ~]# /etc/init.d/mysqld restart #重启mysql
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS! 
```

> **特别提示：**
>  在以往如果是基于binlog日志的主从复制，则必须要记住主库的master状态信息。

![QQ截图20170905003920.png-12.7kB](http://static.zybuluo.com/chensiqi/eovoatv6gtt2iyhvc6t32q03/QQ%E6%88%AA%E5%9B%BE20170905003920.png)

> 但是在MySQL5.6版本里多了一个Gtid的功能，可以自动记录主从复制位置点的信息，并在日志中输出出来。

### 4.4 开启GTID

```
#没开启之前先看一下GTID状态
mysql> show global variables like '%gtid%';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| enforce_gtid_consistency | OFF   |
| gtid_executed            |       |
| gtid_mode                | OFF   |
| gtid_owned               |       |
| gtid_purged              |       |
+--------------------------+-------+
5 rows in set (0.00 sec)
```

**编辑mysql配置文件（主库从库都需要修改）**

![QQ截图20170905005846.png-49.9kB](http://static.zybuluo.com/chensiqi/v5zdfooo0lksyxtul1lf3yqv/QQ%E6%88%AA%E5%9B%BE20170905005846.png)

> mysql-db01,mysql-db02,mysql-db03都需要加入上图的上行代码

**修改完配置文件以后重启动数据库**

```repl
[root@mysql-db01 ~]# /etc/init.d/mysqld restart
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS! 
[root@mysql-db02 ~]# /etc/init.d/mysqld restart
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS!
[root@mysql-db03 ~]# /etc/init.d/mysqld restart
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS!
```

**再次查看GTID状态**

```
[root@mysql-db01 ~]# mysql -uroot -p123123
Warning: Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.6.17-log MySQL Community Server (GPL)

Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show global variables like '%gtid%';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| enforce_gtid_consistency | ON    |    #执行GTID一致
| gtid_executed            |       |    
| gtid_mode                | ON    |    #开启GTID模块
| gtid_owned               |       |
| gtid_purged              |       |
+--------------------------+-------+
5 rows in set (0.00 sec)

mysql> 
```

> **再次提示：**
>  主库从库都必须要开启GTID，否则在做主从复制的时候就会报错.

### 4.5 配置主从复制(mysql-db02,mysql-db03)

```repl
mysql> change master to \
    -> master_host='192.168.0.51',\     #主库IP
    -> master_user='rep',\              #主库复制用户
    -> master_password='123123',\       #主库复制用密码
    -> master_auto_position=1;          #GTID位置点（自动追踪需要同步的position）
Query OK, 0 rows affected, 2 warnings (0.00 sec)
```

### 4.6 开启从库的主从复制功能（mysql-db02,mysql-db03）

```
mysql> start slave;         #开启主从同步功能
Query OK, 0 rows affected, 1 warning (0.01 sec)
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.0.51
                  Master_User: rep
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 151
               Relay_Log_File: mysql-db02-relay-bin.000002
                Relay_Log_Pos: 361
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes      #此项yes代表成功
            Slave_SQL_Running: Yes      #此项yes代表成功
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
        #....以下省略若干行....
```

> 两个从库mysql-db02和mysql-db03都执行以上步骤。

### 4.7 什么是GTID

> - GTID（Global  Transaction）全局事务标识符：是一个唯一的标识符，它创建并与源服务器（主）上提交的每个事务相关联。此标识符不仅对其发起的服务器是唯一的，而且在给定复制设置中的所有服务器上都是唯一的。所有交易和所有GTID之间都有1对1的映射。
> - GTID实际上是由UUID+TID组成的。其中UUID是一个MySQL实例的唯一标识。TID代表了该实例上已经提交的事务数量，并且随着事务提交单调递增。
> - 下面是一个GTID的具体形式：

```
3E11FA47-71CA-11E1-9E33-C80AA9429562:23
```

### 4.8 GTID的新特性

（1）支持多线程复制：事实上是针对每个database开启相应的独立线程，即每个库有一个单独的（sql thread）

（2）支持启用GTID，在配置主从复制，传统的方式里，你需要找到binlog和POS点，然后change master to  指向。在mysql5.6里，无须再知道binlog和POS点，只需要知道master的IP/端口/账号密码即可，因为同步复制是自动的，MySQL通过内部机制GTID自动找点同步。

（3）基于Row复制只保存改变的列，大大节省磁盘空间，网络，内存等

（4）支持把Master和Slave的相关信息记录在Table中；原来是记录在文件里，现在则记录在表里，增强可用性

（5）支持延迟复制

### 4.9 开启方法

```
#mysql配置文件：
[mysqld]
gtid_mode=ON
enforce_gtid_consistency

#查看
show global variables like ‘%gtid%’；
```

### 4.10 从库设置（mysql-db02,mysql-db03）

```
#登陆从库
[root@mysql-db02 ~]# mysql -uroot -p123123

#临时禁用自动删除relay log功能
mysql> set global relay_log_purge = 0;
Query OK, 0 rows affected (0.00 sec)

#设置只读
mysql> set global read_only=1;
Query OK, 0 rows affected (0.00 sec)
```

**编辑配置文件/etc/my.cnf**

![QQ截图20170905021335.png-49.3kB](http://static.zybuluo.com/chensiqi/7z2irfyxi8xcauc43hox0cbg/QQ%E6%88%AA%E5%9B%BE20170905021335.png)

**修改完配置文件，别忘了重启动mysql服务**

```repl
root@mysql-db02 ~]# /etc/init.d/mysqld restart
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS! 
root@mysql-db03 ~]# /etc/init.d/mysqld restart
Shutting down MySQL.. SUCCESS! 
Starting MySQL. SUCCESS! 
```

## 五，部署MHA

### 5.1 环境准备（所有节点mysql-db01,mysql-db02,mysql-db03）

```
#光盘安装依赖包
[root@mysql-db01 ~]# yum -y install perl-DBD-MySQL

#安装mha4mysql-node-0.56-0.el6.noarch.rpm
[root@mysql-db01 rpm]# rpm -ivh mha4mysql-node-0.56-0.el6.noarch.rpm 
Preparing...                ########################################### [100%]
   1:mha4mysql-node         ########################################### [100%]

[root@mysql-db01 ~]# mysql -uroot -p123123

mysql> grant all privileges on *.* to mha@'192.168.0.%' identified by '123123';
Query OK, 0 rows affected (0.00 sec)

mysql> select user,host from mysql.user where user='mha';
+------+-------------+
| user | host        |
+------+-------------+
| mha  | 192.168.0.% |          #主库上创建从库会自动复制
+------+-------------+
1 row in set (0.00 sec)

#特别提示：3台MySQL都需要安装mha4mysql-node-0.56-0.el6.noarch.rpm
```

### 5.2 部署管理节点（mha-manager）

#### 5.2.1 在mysql-db03上部署管理节点

```
#使用阿里云源+epel源
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
wget -O /etc/yum.repos.d/epel-6.repo http://mirrors.aliyun.com/repo/epel-6.repo

#安装manager依赖包（需要公网源）
[root@mysql-db03 ~]# yum -y install perl-Config-Tiny epel-release perl-Log-Dispatch perl-Parallel-ForkManager perl-Time-HiRes

#安装manager包
[root@mysql-db03 rpm]# rpm -ivh mha4mysql-manager-0.56-0.el6.noarch.rpm 
Preparing...                ########################################### [100%]
   1:mha4mysql-manager      ########################################### [100%]
```

#### 5.2.2 编辑配置文件

```
#创建配置文件目录
[root@mysql-db03 ~]# mkdir -p /etc/mha

#创建日志目录
[root@mysql-db03 ~]# mkdir -p /var/log/mha/mha1

#创建配置文件（默认没有）
[root@mysql-db03 ~]# cd /etc/mha/
[root@mysql-db03 mha]# ls
[root@mysql-db03 mha]# vim /etc/mha/mha1.cnf
[root@mysql-db03 mha]# cat /etc/mha/mha1.cnf
[server default]
manager_log=/var/log/mha/mha1/manager               #manager管理日志存放路径
manager_workdir=/var/log/mha/mha1                   #manager管理日志的目录路径
master_binlog_dir=/usr/local/mysql/data             #binlog日志的存放路径
user=mha                                            #管理账户
password=123123                                     #管理账户密码
ping_interval=2                                     #存活检查的间隔时间
repl_user=rep                                       #主从复制的授权账户
repl_password=123123                                #主从复制的授权账户密码
ssh_user=root                                       #用于ssh连接的账户

[server1]
hostname=192.168.0.51                               
port=3306                                           

[server2]
#candidate_master=1                                 #此条暂时注释掉（后面解释）
#check_repl_delay=0                                 #此条暂时注释掉（后面解释）
hostname=192.168.0.52
port=3306

[server3]
hostname=192.168.0.53
port=3306

#**特别提示：**
#以上配置文件内容里每行的最后不要留有空格，因此，不能复制的呦
```

> **特别说明：**
>  参数：candidate_master=1
>  解释：设置为候选master，如果设置该参数以后，发生主从切换以后会将此从库提升为主库，即使这个主库不是集群中事件最新的slave
>  参数：check_repl_delay=0
>  解释：默认情况下如果一个slave落后master 100M的relay logs  的话，MHA将不会选择该slave作为一个新的master，因为对于这个slave的恢复需要花费很长时间，通过设置check_repl_delay=0,MHA触发切换在选择一个新的master的时候将会忽略复制延时，这个参数对于设置了candidate_master=1的主机非常有用，因为这个候选主在切换的过程中一定是新的master

### 5.3 配置ssh信任（所有节点mysql-db01,mysql-db02,mysql-db03）

```
#创建密钥对
[root@mysql-db03 ~]# ssh-keygen -t dsa -P "" -f ~/.ssh/id_dsa >/dev/null 2>&1

#发送mysql-db03公钥，包括自己
[root@mysql-db03 ~]# ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.51
[root@mysql-db03 ~]# ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.52
[root@mysql-db03 ~]# ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.53

#发送mysql-db02公钥，包括自己
[root@mysql-db02 ~]# ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.51
[root@mysql-db02 ~]# ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.52
[root@mysql-db02 ~]# ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.53

#发送mysql-db01公钥，包括自己
[root@mysql-db01 ~]# ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.51
[root@mysql-db01 ~]# ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.52
[root@mysql-db01 ~]# ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.0.53
```

### 5.4 启动测试

#### 5.4.1 ssh检查检测

```
[root@mysql-db03 ~]# masterha_check_ssh --conf=/etc/mha/mha1.cnf    #ssh检查命令
Tue Sep  5 03:01:38 2017 - [warning] Global configuration file /etc/masterha_default.cnf not found. Skipping.
Tue Sep  5 03:01:38 2017 - [info] Reading application default configuration from /etc/mha/mha1.cnf..
Tue Sep  5 03:01:38 2017 - [info] Reading server configuration from /etc/mha/mha1.cnf..
Tue Sep  5 03:01:38 2017 - [info] Starting SSH connection tests..
..中间省略若干行..
Tue Sep  5 03:01:40 2017 - [debug]  Connecting via SSH from root@192.168.0.53(192.168.0.53:22) to root@192.168.0.52(192.168.0.52:22)..
Tue Sep  5 03:01:40 2017 - [debug]   ok.
Tue Sep  5 03:01:40 2017 - [info] All SSH connection tests passed successfully. #出现这个就表示成功
```

#### 5.4.2 主从复制检测

（1）错误的主从复制检测

```
[root@mysql-db03 ~]# masterha_check_repl --conf=/etc/mha/mha1.cnf
```

> 如果不出意外，同学们的检测结果都会是下面的样子

![QQ截图20170906002542.png-56.7kB](http://static.zybuluo.com/chensiqi/aorxkk67v0ru8465byctyz2n/QQ%E6%88%AA%E5%9B%BE20170906002542.png)

> 因此在mysql-db02和mysql-db03上添加主从复制的用户即可。
>  `grant replication slave on *.* to rep@'192.168.0.%' identified by '123123';`
>  再次检查如下图所示：

![QQ截图20170906003531.png-23.9kB](http://static.zybuluo.com/chensiqi/ga2ast740c7qz3pxb27ooaby/QQ%E6%88%AA%E5%9B%BE20170906003531.png)

### 5.5 启动MHA

```
#启动
[root@mysql-db03 ~]# nohup masterha_manager --conf=/etc/mha/mha1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/mha/mha1/manager.log 2>&1 &
[root@mysql-db03 ~]# ps -ef | grep perl | grep -v grep
root       4961   4690  0 06:33 pts/2    00:00:00 perl /usr/bin/masterha_manager --conf=/etc/mha/mha1.cnf --remove_dead_master_conf --ignore_last_failover

#说明：
nohup:启动命令
--conf：指定配置文件位置
--remove_dead_master_conf：如果有master down了，就去掉配置文件里该master的部分。
```

### 5.6 进行mha自动切换master的测试

**初始状态：**

![QQ截图20170906233818.png-10.5kB](http://static.zybuluo.com/chensiqi/kaiuv2nwx5wsihhr9jegouin/QQ%E6%88%AA%E5%9B%BE20170906233818.png)

（1）登陆mysql-db02(192.168.0.53)查看信息状态

```
#登陆数据库mysql-db02(192.168.0.53)
[root@mysql-db03 ~]#  mysql -uroot -p123123
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.0.51         #这是主库IP地址
                  Master_User: rep
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 1656
               Relay_Log_File: mysql-db02-relay-bin.000004
                Relay_Log_Pos: 1796
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
            ..以下省略若干内容..
```

（2）停掉mysql-db01(192.168.0.51)上的MySQL服务

```
[root@mysql-db01 ~]# /etc/init.d/mysqld stop
Shutting down MySQL..... SUCCESS!
```

（3）查看mysql-db03上的MySQL从库同步状态

```
[root@mysql-db03 ~]# mysql -uroot -p123123 -e 'show slave status\G'
Warning: Using a password on the command line interface can be insecure.
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.0.52         #现在的主库IP
                  Master_User: rep
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000006     #binlog日志
          Read_Master_Log_Pos: 777                  #binlog日志位置
               Relay_Log_File: mysql-db03-relay-bin.000002
                Relay_Log_Pos: 408
        Relay_Master_Log_File: mysql-bin.000006
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
             ..以下省略若干内容..
```

（4）查看mysql-db02上的MySQL，主库同步状态。

![QQ截图20170906201709.png-18.6kB](http://static.zybuluo.com/chensiqi/t5f4umvlpf6mrnwkdadq3uxw/QQ%E6%88%AA%E5%9B%BE20170906201709.png)

（5）查看mysql-db03上的mha进程状态

```
[root@mysql-db03 ~]# ps -ef | grep perl | grep -v grep      #查询发现mha进程已经没了
[root@mysql-db03 ~]# 
```

（6）查看mha配置文件信息

![QQ截图20170906213038.png-26.7kB](http://static.zybuluo.com/chensiqi/8zboi0vav5qxqeb6fvjkkv4m/QQ%E6%88%AA%E5%9B%BE20170906213038.png)

> **说明：**
>  当作为主库的mysql-db01上的MySQL宕机以后，mha通过检测发现mysql-db01宕机，那么会将binlog日志最全的从库立刻提升为主库，而其他的从库会指向新的主库进行再次同步。

**此处需要进行简单的mha日志记录的讲解：/var/log/mha/mha1/manager**

### 5.7 进行mha的故障还原测试

> 由于mysql-db01的MySQL服务宕机，因此mha将mysql-db02提升为了主库。因此，我们需要将宕机的mysql-db01的MySQL服务启动，然后作为主库mysql-db02的从库。

**初始状态：**

![QQ截图20170906233927.png-10.4kB](http://static.zybuluo.com/chensiqi/kjgtcz7p8q85nehc725w2p0w/QQ%E6%88%AA%E5%9B%BE20170906233927.png)

（1）将故障宕机的mysql-db01的MySQL服务启动并授权进行从同步

```
[root@mysql-db01 ~]# /etc/init.d/mysqld start
Starting MySQL. SUCCESS! 
[root@mysql-db01 ~]# mysql -uroot -p123123
mysql> CHANGE MASTER TO MASTER_HOST='192.168.0.52', MASTER_PORT=3306, MASTER_AUTO_POSITION=1, MASTER_USER='rep', MASTER_PASSWORD='123123';
mysql> start slave;
mysql> show slave status\G          #查看从同步状态
```

（2）将mha配置文件里缺失的部分补全

```
[root@mysql-db03 ~]# cat /etc/mha/mha1.cnf 
[server default]
manager_log=/var/log/mha/mha1/manager
manager_workdir=/var/log/mha/mha1
master_binlog_dir=/usr/local/mysql/data
password=123123
ping_interval=2
repl_password=123123
repl_user=rep
ssh_user=root
user=mha

[server1]
hostname=192.168.0.51
port=3306

[server2]
hostname=12.168.0.52
port=3306

[server3]
hostname=192.168.0.53
port=3306
```

（3）启动mha进程

```
[root@mysql-db03 ~]# nohup masterha_manager --conf=/etc/mha/mha1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/mha/mha1/manager.log 2>&1 &
[root@mysql-db03 ~]# ps -ef | grep perl | grep -v grep
root       5226   4690  0 09:42 pts/2    00:00:00 perl /usr/bin/masterha_manager --conf=/etc/mha/mha1.cnf --remove_dead_master_conf --ignore_last_failover
```

（4）停掉mysql-db02上的MySQL服务

```
[root@mysql-db02 ~]# /etc/init.d/mysqld stop
Shutting down MySQL..... SUCCESS! 
```

（5）查看mysql-db03上的主从同步状态：

```
[root@mysql-db03 ~]# mysql -uroot -p123123 -e 'show slave status\G'
Warning: Using a password on the command line interface can be insecure.
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.0.51             #此时的主库IP切换回了mysql-db01
                  Master_User: rep
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000005
          Read_Master_Log_Pos: 231
               Relay_Log_File: mysql-db03-relay-bin.000002
                Relay_Log_Pos: 361
        Relay_Master_Log_File: mysql-bin.000005
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
            ..以下省略若干行..
```

（6）启动mysql-db02上的MySQL服务

```
[root@mysql-db02 ~]# /etc/init.d/mysqld start
Starting MySQL. SUCCESS! 
[root@mysql-db02 ~]# mysql -uroot -p123123
mysql> CHANGE MASTER TO MASTER_HOST='192.168.0.51', MASTER_PORT=3306, MASTER_AUTO_POSITION=1, MASTER_USER='rep', MASTER_PASSWORD='123123';
mysql> start slave;
mysql> show slave status\G          #查看从同步状态
```

（7）再次补全mha配置文件后，启动mha进程

```
[root@mysql-db03 ~]# cat /etc/mha/mha1.cnf 
[server default]
manager_log=/var/log/mha/mha1/manager
manager_workdir=/var/log/mha/mha1
master_binlog_dir=/usr/local/mysql/data
password=123123
ping_interval=2
repl_password=123123
repl_user=rep
ssh_user=root
user=mha

[server1]
hostname=192.168.0.51
port=3306

[server2]
hostname=192.168.0.52
port=3306

[server3]
#andidate_master=1
#check_repl_delay=0
hostname=192.168.0.53
port=3306
[root@mysql-db03 ~]# nohup masterha_manager --conf=/etc/mha/mha1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/mha/mha1/manager.log 2>&1 &
[root@mysql-db03 ~]# ps -ef | grep perl | grep -v grep
root       5226   4690  0 09:42 pts/2    00:00:01 perl /usr/bin/masterha_manager --conf=/etc/mha/mha1.cnf --remove_dead_master_conf --ignore_last_failover
```

**此时的初始状态还原为下图：**

![QQ截图20170906234956.png-10.4kB](http://static.zybuluo.com/chensiqi/co9ncxxtr5jsuxh0rwqi436m/QQ%E6%88%AA%E5%9B%BE20170906234956.png)

## 六，MHA参数验证实践（同学们操作）

![QQ截图20170907001005.png-30.9kB](http://static.zybuluo.com/chensiqi/s868cf3uj0w072kvq5gvegk9/QQ%E6%88%AA%E5%9B%BE20170907001005.png)

**mha配置文件内容如下：**

![QQ截图20170907001142.png-31.8kB](http://static.zybuluo.com/chensiqi/kacrdzna0b0q81v4ebnhehsx/QQ%E6%88%AA%E5%9B%BE20170907001142.png)

## 附录：源码安装mha的方法

### node节点的源码安装方法：

```
[root@mysql-db01 ~]# yum -y install perl-DBD-MySQL perl-Config-Tiny perl-Params-Validate perl-CPAN perl-devel perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker

[root@mysql-db01 ~]# tar xf mha4mysql-node-0.56.tar.gz -C /usr/src/

[root@mysql-db01 ~]# cd /usr/src/mha4mysql-node-0.56/

[root@mysql-db01 mha4mysql-node-0.56]# perl Makefile.PL

[root@mysql-db01 mha4mysql-node-0.56]# make && make install
```

### manager节点的源码安装方法：

```
[root@mysql-db01 ~]# yum -y install perl-DBD-MySQL perl-Config-Tiny perl-Params-Validate perl-CPAN perl-devel perl-ExtUtils-CBuilder perl-ExtUtils-MakeMaker

[root@mysql-db01 ~]# tar xf mha4mysql-manager-0.56.tar.gz -C /usr/src/

[root@mysql-db01 ~]# cd /usr/src/mha4mysql-manager-0.56/

[root@mysql-db01 mha4mysql-manager-0.56]# perl Makefile.PL

[root@mysql-db01 mha4mysql-manager-0.56]# make && make install
```

