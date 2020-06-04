[TOC]

# 需求

通过一些中间件来实现主从复制架构中主节点的高可用,当主节点down掉以后,会根据一定规则算法提升一个从节点为新的主节点,然后让其他从节点自动切换为新的主节点进行复制.

# 架构

常见的架构有:

MMM[Multi Master Mysql]

MHA[Master HA]

# MMM简介

MMM即Master-Master Replication Manager for MySQL[mysql主主复制管理器]

它是perl语言开发的一套用于管理mysql主主同步架构的一种工具集,主要作用是,监控和管理mysql的主主复制拓扑,并在当前的主服务器失效时,进行主和主备服务器之间的主从切换和故障转移工作.

MMM提供了自动和手动两种方式移除一组服务器中复制延迟较高的服务器的虚拟ip,同时它还可以备份数据,实现两节点之间的数据同步等.

由于MMM无法完全的保证数据一致性,所以MMM适用于对数据的一致性要求不是很高,但是又想最大程度的保证业务可用性的场景,对于那些对数据的一致性要求很高的业务,非常不建议采用MMM这种高可用架构.

MMM项目来自Google：http://code.google.com/p/mysql-master-master

官方网站为：http://mysql-mmm.org

# MHA简介

MHA[Master HA]是一款开源的mysql高可用软件,为mysql主从复制架构提供了automating master failover功能.

MHA在监控到master节点故障时,会提升其中拥有最新数据的slave节点成为新的master,在此期间,MHA会通过于其他从节点获取额外信息来避免一致性方面的问题.

MHA还提供了master节点的在线切换功能,即按需切换master/slave节点.

MHA是一位日本MySQL大牛用Perl写一套MySQL故障切换方案,来保证数据库系统的高可用,在宕机的事件内(通常10-30秒),完成故障转意,部署MHA,可避免主从一致性问题,节约购买新服务器的费用,不影响服务器性能,易安装,不改变现有部署.

MHA服务有两种角色:

MHA Manager[管理节点]

通常单独部署在一台独立主机上管理多个master/slave集群,每个master/slave集群称作一个application.

MHA Node[数据节点]

运行在每台mysql节点上[master/slave/manager],它通过监控具备解析和清理logs功能的脚本来加快故障转移.

![img](MHA%E9%AB%98%E5%8F%AF%E7%94%A8.assets/f618fc3d-6a3e-49f2-b8b2-3e775ac3021f.jpg)

mysql复制集群中master故障时,MHA按如下步骤进行故障转移:

![img](MHA%E9%AB%98%E5%8F%AF%E7%94%A8.assets/1354456c-6c35-4b0a-a75c-4ed875b54acc.jpg)

在MHA自动故障切换过程中,MHA试图从宕机的主服务器上保存二进制日志,最大程度的保证数据的不丢失,但这并不总是可行的.例如,如果主服务器硬件故障或无法通过ssh访问,MHA没法保存二进制日志,只进行故障转移而丢失了最新的数据.使用MySQL5.5的半同步复制,可以大大降低数据丢失的风险.

MHA可以与半同步复制结合起来,如果只有一个slave已经收到了最新的二进制日志,MHA可以将最新的二进制日志应用于其他所有的slave服务器上,因此可以保证所有节点的数据一致性.

目前MHA主要支持一主多从的架构,要搭建MHA,要求一个复制集群中必须最少有三台数据库服务器,一主二从,即一台充当master,一台充当备用master,另外一台充当从库,因为至少需要三台服务器,出于机器成本的考虑,淘宝也在该基础上进行了改造，目前淘宝TMHA已经支持一主一从.

## 优点

同样是由Perl语言开发的开源工具,可以支持基于GTID的复制模式,MHA在进行故障转移时更不易产生数据丢失,同一个监控节点可以监控多个集群.

## 缺点

需要编写脚本或利用第三方工具来实现vip的配置,MHA启动后只会对数据库进行监控,需要基于ssh免认证配置,存在一定的安全隐患,没有提供从服务器的读负载均很的功能.

# MHA工具栈

## Manager节点

\- masterha_check_ssh:MHA依赖的ssh环境监测工具

\- masterha_check_repl:mysql复制环境监测工具

\- masterha_manager:MHA服务主程序

\- masterha_check_status:MHA运行状态探测工具

\- masterha_master_monitor:mysql的master节点可用性监测工具

\- masterha_master_switch:master节点切换工具

\- masterha_conf_host:添加或删除配置的节点

\- masterha_stop:关闭MHA服务的工具

## Node节点

\- save_binary_logs:保存和复制master的二进制日志

\- apply_diff_relay_logs:识别差异的中继日志事件,并应用于其他slave

\- filter_mysqlbinlog:去除不必要的ROLLBACK事件[MHA已不再使用此工具]

\- purge_relay_logs:清除中继日志[不会阻塞SQL线程]

## 自定义扩展

\- secondary_check_script:通过多条网络路由检测master的可用性

\- master_ip_failover_script:更新application使用的masterip

\- shutdown_script:强制关闭master节点

\- report_script:发送报告

\- init_conf_load_script:加载初始配置参数

\- master_ip_online_change_script:更新master节点ip地址

# 示例

环境准备:

系统环境:CentOS 6.9 x86_64

mysql: mysql-5.5.59-linux-glibc2.12-x86_64.tar.gz

各节点IP及主机名:

172.16.1.5  mysql: MHA Manager

172.16.1.6  mysql-1: Master

172.16.1.7  mysql-2: slave1

172.16.1.8  mysql-3: slave2

注意:MHA对mysql复制环境有特殊要求,例如各节点都要开启二进制日志及中继日志,各从节点必须显式启用其read_only属性,并关闭relay_log_purge功能等.

保证各节点/etc/hosts:

172.16.1.5   mysql.com    mysql

172.16.1.6   mysql-1.com   mysql-1

172.16.1.7   mysql-2.com   mysql-2

172.16.1.8   mysql-3.com   mysql-3

master配置:

 

```
serve_id=6  # 以ip地址为id
relay-log=relay-bin
log-bin=mysql-bin
```

slave1节点配置:

 

```
server_id=7
relay-log=relay-bin
log-bin=master-bin
relay_log_purge=0
read_only=1
```

slave2节点配置:

 

```
server_id=8
relay-log=relay-bin
log-bin=master-bin
relay_log_purge=0
read_only=1
```

按上述要求分别配置好主从节点,并实现主从复制;

然后,在所有节点授权拥有管理权限的用户可在本地网络中有其他节点上的远程访问权限;

最后在主节点master上额外执行如下sql语句:

mysql> grant all on *.* to 'mhaadmin'@'%' identified by 'mhapass';

具体配置过程:

1,master节点

 

```
vim /etc/mysql/my.cnf
"""
server-id = 6  # id以ip主机位命名
relay-log = relay-bin
skip_name_resolve = 1
log-bin = master-bin
innodb_file_per_table = 1
"""

# 重启服务,查看log-bin是否开启
mysql> show variables like 'log_bin';

# 授权用于复制的用户
mysql> grant replication slave,replication client on *.* to 'repluser'@'172.16.1.%' identified by 'replpass';
mysql> flush privileges;
mysql> show master status;  # 查看并记录bin-log和pos值
+-------------------+----------+--------------+------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+-------------------+----------+--------------+------------------+
| master-bin.000001 |      466 |              |                  |
+-------------------+----------+--------------+------------------+

```

2,slave节点

 

```
# mysql-2配置文件:
server-id = 7  # id唯一
innodb_file_per_table = 1
skip_name_resolve = 1
log-bin = master-bin
relay-log = relay-bin
read_only = 1  # 开启只读
relay_log_purge = 0  # 关闭中继日志清除功能


# mysql-3配置文件:
server-id = 8  # id唯一
innodb_file_per_table = 1
skip_name_resolve = 1
log-bin = mysql-bin
relay-log = relay-log
read_only = 1
relay_log_purge = 0


# 重启服务
```

3,分别在slave上指定master信息

 

```
mysql> change master to master_host='172.16.1.6',master_user='repluser',master_password='replpass',master_log_file='master-bin.000001',master_log_pos=466;
mysql> start slave;
mysql> show slave status\G
```

注:如果提示:

ERROR 1201 (HY000): Could not initialize master info structure; more error messages can be found in the MySQL error log

可能的原因是由于新的slave改变了服务端口和文件路径，分析应该是由于mysql-relay-bin.index中仍然保存着旧relay日志文件的路径，而这些路径下又找不到合适的文件，因此报错.

解决办法:重置slave参数即可:

 

```
mysql> reset slave;
```

4,在主节点上创建一个有管理且远程连接权限的mysql账号

 

```
mysql> grant all on *.* to 'mhauser'@'172.16.1.%' identified by 'mhapass';
mysql> flush privileges;
```

5,配置实现各主机基于ssh免密登录

分析:由于要求各节点间需要实现免密互访,那么可以考虑所有节点都用一个密钥对.

注意hosts文件

 

```
# node1:MHA Manager
ssh-keygen -t rsa -P ''  # 生成密钥对

# 将自己的私钥写入本地的authorized_keys文件中,注意根据实际情况选择覆盖写入还是追加写入
cat .ssh/id_rsa.pub > .ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys

# 测试连接本机是否免密
ssh -p 2201 node1

scp -p -P 2201 .ssh/id_rsa .ssh/authorized_keys mysql-1:/root/.ssh  # -p保持权限,-P指定ssh端口
scp -p -P 2201 .ssh/id_rsa .ssh/authorized_keys mysql-2:/root/.ssh
scp -p -P 2201 .ssh/id_rsa .ssh/authorized_keys mysql-3:/root/.ssh

# 测试
ssh -p 2201 mysql-1 'ifconfig eth0'
...
```

6,安装配置MHA

除了源码包,MHA官网也提供了rpm格式的程序包,下载地址为:

https://downloads.mariadb.com/MHA/

https://code.google.com/archive/p/mysql-master-ha/downloads

CentOS7可直接使用适用于el6的程序包.另外,MHA Manager和MHA Node的版本并不强制要求一致.

这里安装下载的是:

mha4mysql-manager-0.55-0.el6.noarch.rpm

mha4mysql-node-0.54-0.el6.noarch.rpm

 

```
# manager管理节点安装manager包和node包
yum install -y mha4mysql-manager-0.55-0.el6.noarch.rpm
rpm -ql mha4mysql-manager

# 其它节点分别安装node包
yum install -y mha4mysql-node-0.54-0.el6.noarch.rpm
rpm -ql mha4mysql-node
```

Manager节点需要为每个监控的master/slave集群提供一个专用的配置文件,而所有的master/slave集群也可共享全局配置.

/etc/masterha_default.cnf

全局配置文件,为可选配置,如果仅监控一组master/slave集群,也可直接通过application的配置来提供服务器的默认配置信息.每个application的配置文件路径为自定义.

这里使用/etc/masterha/app1.cnf

 

```
mkdir /etc/masterha
vim /etc/masterha/app1.cnf
"""

[server default]
user=mhauser  # 用于管理节点的mysql管理账号,默认root
password=mhapass  # 密码
manager_workdir=/data/masterha/app1  # mha manger节点生成的相关状态文件的绝对路径,默认/var/tmp
manager_log=/data/masterha/app1/manager.log  # 日志路径
remote_workdir=/data/masterha/app1  # 每一个MHA节点生成日志文件的工作路径,不存在则会自动创建,注意访问权限,默认/var/tmp
ssh_user=root  # 访问节点所使用的系统账号
repl_user=repluser  # 用于做主从同步的数据库的账号,在所有slave节点上执行change master的账号,建议在master上拥有replication slave权限
repl_password=replpass
ping_interval=1  # ping检测状态次数

[server1]
hostname=172.16.1.6  # master节点,只能配置在app配置文件的[server_xx]段下
port=3306  # 默认端口
#ssh_host=  # 默认和hostname参数相同,当使用了vlan隔离时,比如为了安全,吧ssh网段和数据库集群分开使用时,需单独配置
ssh_port=2201
#ssh_connection_timeout=5  # 默认5s
#ssh_options=  # ssh命令的额外选项
#candidate_master=1  # 根据优先级,从不同的slave节点中,提升一个可靠的机器为新master,不一定会成为master
#no_master=1 # 禁止成为master
#ignore_fail # 忽略故障,继续转移
#skip_init_ssh_check # 跳过SSH连接检测

[server2]
hostname=172.16.1.7  # slave节点
candidate_master=1
ssh_port=2201

[server3]
hostname=172.16.1.8  # slave节点
ssh_port=2201
#no_master=1
"""

# 检测各节点间ssh互信通信配置是否OK
masterha_check_ssh --conf=/etc/masterha/app1.cnf

# 检测主从复制的连接配置参数是否OK
masterha_check_repl --conf=/etc/masterha/app1.cnf

# 前台启动MHA服务
masterha_manager --conf=/etc/masterha/app1.cnf
```

测试

手动kill掉主节点的mysql,由于masterha工作在前台,当检测到集群中主节点down掉以后,它会在完成切换后,自动退出.

 

```
# mysql-1
killall mysqld mysqld_safe

# mysql-2
show slave status\G

# mysql-3
show slave status\G
```

注意：假如原主节点的mysql又重新启动了.不会在成为主,而是要手动配置为从节点.

遇到的错误

1,*[error][/usr/share/perl5/vendor_perl/MHA/Server.pm, ln382] 172.16.1.7(172.16.1.7:3306): User repluser does not exist or does not have REPLICATION SLAVE privilege! Other slaves can not start replication from this host.*

查看1.7这个从节点,发现用于复制的用户没有同步过来.

解决办法:

清除所有从节点的slave信息, 重新复制

2,*Failed to save binary log: Binlog not found from /var/lib/mysql,/var/log/mysql! If you got this error at MHA Manager, please set "master_binlog_dir=/path/to/binlog_directory_of_the_master" correctly in the MHA Manager's configuration file and try again.*

是说在MHA管理节点上,没有指定master_binlog_dir,在配置文件里面添加此项,指向master节点的binlog路径即可.

3,*Can't exec "mysqlbinlog": No such file or directory at /usr/share/perl5/vendor_perl/MHA/BinlogManager.pm line 99.*

*mysqlbinlog version not found!*

解决办法:

 

```
# 在集群的每个节点上都做链接
which mysqlbinlog
ln -sv /applications/mysql/bin/mysqlbinlog /usr/bin/mysqlbinlog
```

4,*Testing mysql connection and privileges..sh: mysql: command not found**
*

*mysql command failed with rc 127:0!*

 

```
# 在集群的每个节点上都做链接
ln -sv /applications/mysql/bin/mysql /usr/bin/mysql
```

# MHA配置文件参数详解

 

```
以下是配置文件中具体每个配置参数的解释：

hostname：目标实例的主机名或者IP地址，这个参数是强制的，只能配置在app配置文件的[server_xxx]段落标记下

ip：目标实例的ip地址，默认从hostname获取，MHA内部管理节点与数据节点之间的mysql和ssh通过这个IP连接，正常情况下你不需要配置这个参数，因为MHA会自动解析Hostname获得

port：目标实例的端口，默认是3306，MHA使用mysql server的IP和port连接

ssh_host：从0.53版本开始支持，默认情况下和hostname参数相同，当你使用了VLAN隔离的时候，比如为了安全，把ssh网段和访问数据库实例的网段分开使用两个不同的IP，那么就需要单独配置这个参数

ssh_ip：从0.53版本开始支持，默认情况下与ssh_host相同

ssh_port：从0.53版本开始支持，目标数据库的ssh端口，默认是22

ssh_connection_timeout：从0.54版本开始支持，默认是5秒，增加这个参数之前这个超时时间是写死在代码里的

ssh_options：从0.53版本开始支持，额外的ssh命令行选项

candidate_master：从不同的从库服务器中，提升一个可靠的机器为新主库，（比如：RAID 10比RAID0的从库更可靠），可以通过在配置文件中对应的从库服务器的配置段下添加candidate_master=1来提升这个从库被提升为新主库的优先级（这个从库要开始binlog，以及没有显著的复制延迟，如果不满足这两个条件，也并不会在主库挂掉的时候成为新主库，所以，这个参数只是提升了优先级，并不是说指定了这个参数就一定会成为新主库）

no_master：通过在目标服务器的配置段落设置no_master=1，它永远也不会变为新主库，用于配置在不想用于接管主库故障的从库上，如：只是一个binlog server，或者硬件配置比较差的从库上，或者这个从库只是一个远程灾备的从库，但是，要注意，不能把所有的从库都配置这个参数，这样MHA检测到没有可用备主的时候，会中断故障转移操作

ignore_fail：默认情况下，MHA在做故障转移的时候，会去检测所有的server，如果发现有任何的从库有问题（比如不能通过SSH检测主机或者mysql的存活，或者说有SQL线程复制错误等）就不会进行故障转移操作，但是，有时候可能需要在特定从库有故障的情况下，仍然继续进行故障转移，就可以使用这个参数指定它。默认是0，需要设置时在对应服务器的配置段下添加ignore_fail=1

skip_init_ssh_check：监控程序启动的时候，跳过SSH连接检测
skip_reset_slave：0.56开始支持，master故障转移之后，跳过执行reset slave all语句
user：目标mysql实例的管理帐号，尽量是root用户，因为运行所有的管理命令（如：stop slave,change master,reset slave）需要使用，默认是root

password：user参数指定的用户的密码，默认为空

repl_user：在所有slave上执行change master的复制用户名，这个用户最好是在主库上拥有replication slave权限

repl_password：repl_user参数指定的用户的密码，默认情况下，是当前复制帐号的密码，如果你运行了online master switch脚本且使用了–orig_master_is_new_slave做在线切换，当前主库作为了从库加入新主库，此时，如果你没有设置repl_password来指定复制帐号的密码，当前主库作为从库加入新主库的时候会在change master的时候失败，因为默认情况下这个密码是空的，MHA做切换的时候，其他所有作为从库加入新主库时，都需要使用这个密码

disable_log_bin：如果设置这个参数，当应用差异日志到从库时，slave不生成binlog写入，内部通过mysqlbinlog命令的–disable-log-bin选项实现

master_pid_file：设置master的pid，在运行多实例的环境下，你可能需要用到这个参数来指定master的pid，参考shutdown_script参数详解

ssh_user：访问MHA manger和MHA mysql节点的OS系统帐号，需要使用它来远程执行各种各样的管理mysql节点的命令（从manager到mysql），从latest slave拷贝差异日志到其他从库上（从mysql到mysql），该用户必须要能够没有任何交互地连接到服务器，通常使用ssh key认证，默认情况下，这个用户是manager所在的OS系统用户。

remote_workdir：每一个MHA node（指的是mysql server节点）生成日志文件的工作路径，这个路径是绝对路径，如果该路径目录不存在，则会自动创建，如果没有权限访问这个路径，那么MHA将终止后续过程，另外，你需要关心一下这个路径下的文件系统是否有足够的磁盘空间，默认值是/var/tmp

master_binlog_dir：在master上生成binlog的绝对路径，这个参数用在master挂了，但是ssh还可达的时候，从主库的这个路径下读取和复制必须的binlog events，这个参数是必须的，因为master的mysqld挂掉之后，没有办法自动识别master的binlog存放目录。默认情况下，master_binlog_dir的值是/var/lib/mysql,/var/log/mysql，/var/lib/mysql目录是大多数mysql分支默认的binlog输出目录，而 /var/log/mysql是ubuntu包的默认binlog输出目录，这个参数可以设置多个值，用逗号隔开

log_level：manager记录日志的等级，默认以及大多数场景下都是info级别，可以设置的值为debug/info/warning/error

manager_workdir：mha manager生成的相关状态文件的绝对路径，如果没有设置，则默认使用/var/tmp

client_bindir: 如果mysql命令工具是安装在非标准路径下，那么使用这个参数指定绝对路径

client_libdir：如果mysql的库文件是安装在非标准路径下，那么使用这个参数指定据对路径

manager_log：mha manager生成的日志据对路径，如果没有设置，mha manager将打印在标准输出，标准错误输出上，如：当mha manager执行故障转移的时候，这些信息就会打印

check_repl_delay：默认情况下，如果从库落后主库100M的relay logs，MHA不会选择这个从库作为新主库，因为它会增加恢复的时间，设置这个参数为0，MHA在选择新主库的时候，则忽略复制延迟，这个选项用在你使用candidate_master=1 明确指定需要哪个从库作为新主库的时候使用。

check_repl_filter：默认情况下，如果master和slave相互之间有任何不同的binlog复制过滤规则，MHA将打印错误日志并拒绝启动监控程序或者拒绝故障转移操作，这是为了避免意外出现类似Table not exists这样的错误，如果你100%确定不同的复制过滤规则不会出现恢复问题，那么就设置check_repl_filter = 0，此时MHA在应用差异日志的时候就不会去检测复制过滤规则，但是你可能会碰到Table not exists这样的错误，使用这个参数要非常小心

latest_priority：默认情况下，latest slave(收到最新的binlog的slave)，优先作为新主库，如果你想控制这个优先级顺序，可以通过 latest_priority=0，此时，就会按照配置文件中配置的顺序来选择新主库（如 host2->host3->host4）。

multi_tier_slave：从0.52版本开始，支持多个主库的复制架构，默认情况下，在配置文件中不支持三个或更多层级的复制架构，例如：host2从host1复制，host3从host2复制，默认情况下，是不允许host1,2,3同时可写的。因为这个是一个三层复制，MHA manager会报错并停止，设置这个参数之后，MHA manager不会中断三层复制架构的监控，只是忽略了第三层，即host3，如果host1挂了，host2将接管host1，被选择为新主库，host3继续从host3复制。

ping_interval：这个参数表示mha manager多久ping（执行select ping sql语句）一次master，连续三个丢失ping连接，mha master就判定mater死了，因此，通过4次ping间隔的最大时间的机制来发现故障，默认是3，表示间隔是3秒

ping_type：从0.53版本开始支持，默认情况下，mha master基于一个到master的已经存在的连接执行select 1(ping_type=select)语句来检查master的可用性，但是在有些场景下，最好是每次通过新建/断开连接的方式来检测，因为这能够更严格以及更快速地发现TCP连接级别的故障，把ping_type设置为connect就可以实现。从0.56开始新增了ping_type=INSERT类型。

secondary_check_script：通常，推荐通过两个或者更多的网络路径来检测master的可用性，默认情况下，MHA只是通过单个网络路径来检测，即从mha master到master的路线，但是不推荐这么做，mha可以通过secondary_check_script 参数来调用一个外部的脚本来实现两个或更多个网络路径来检测master的可用性，配置示例如下： 
secondary_check_script = masterha_secondary_check -s remote_host1 -s remote_host2  masterha_secondary_check 脚本包含在mha manager的安装包里，这个脚本适合大多数的场景，但是你可以调用任何你自定义的脚本。  在上述的示例中，mha manager检测master的存活的网络路径是：mha manager–>remote_host1–>master_host和mha manager–>remote_host2–>master_host，如果两条线路中，mha master到远程主机的连接是通的，但是远程主机到master不通，这个脚本返回状态0，那么mha就判定master死了。 将开始执行故障转移。如果mha manager到远程主机都不通，那么这个脚本返回状态2，mha manager就猜测网络可能发生了问题了，拒绝执行故障转移操作。如果两条线路从头到位都走通了，那么这个脚本返回状态3，此时mha master知道主库是或者的，就不会进行故障转移操作。  通常来说，remote_host1和remote_host2需要处于本地的两个不同的网段  mha调用secondary_check_script 脚本自动地传递如下参数（不需要你在配置文件中的secondary_check_script 参数中指定这些参数），如果你需要更多的功能，你可以自定义这个脚本：  –user：远程主机ssh用户名，如果指定了这个，那么ssh_user参数就会被忽略  –master_host：指定master的主机名（实例）  –master_ip：指定master的IP（实例）  –master_port：指定master的端口号（实例）  PS：运行这个脚本需要依赖perl的 IO::Socket::INET包，这个包包含在perl的5.6.0里，masterha_secondary_check 脚本仍然通过免密钥的ssh来连接远程服务器，这个脚本尝试从远程服务器建立到master的TCP连接，这个意思是my.cnf中的max_connections 参数对这个么有 影响，如果这个TCP连接成功了，在mysql的Aborted_connects 变量中会增加1

master_ip_failover_script：通常在MHA环境下，通常需要分配一个VIP供master用于对外提供读写服务，如果master挂了，HA软件指引备用服务器接管VIP，另外一个通用的方法，是在数据库中创建一个全局的应用程序名称与读写IP地址之间的全局类目映射（ {app1_master1, 192.168.0.1}, {app_master2, 192.168.0.2}, …）来代替VIP地址的使用，在这种情况下，如果你的master挂了，你需要更新这个数据库中的全局类目映射。第二种方法这里个人觉得跟智能DNS解析类似。 
两种方法各有优缺点，MHA不强制使用哪一种方法，但是允许用户使用任何IP地址做故障转移的方案，master_ip_failover_script 参数可以用于实现这个目的，换句话说，你需要自己编写一个脚本来透明地把应用程序连接到新主库上。需要在配置文件中定义这个参数，示例如下：  master_ip_failover_script= /usr/local/sample/bin/master_ip_failover  mha manager安装包中（tar包和githup分支上才有，rpm包没有）的samples/scripts/master_ip_failover路径下有一个简单的脚本  mha manager会调用master_ip_failover_script 这个参数指定的脚本三次，第一次是在进入主库监控之前（为了检测脚本的有效性），第二次是将要调用shutdown_script参数的脚本之前，第三次是新主库应用完成了差异relay log之后，mha manager调用这个脚本会用到如下参数（注意：你不需要在配置文件中这个参数上单独设置这些）  检测阶段参数：  –command=status  –ssh_user=(current master's ssh username)  –orig_master_host=(current master's hostname)  –orig_master_ip=(current master's ip address)  –orig_master_port=(current master's port number)  关闭当前master的阶段参数（注意，到调用shutdown_script的时候，主库已经被判定为死了）：  –command=stop or stopssh  –ssh_user=(dead master's ssh username, if reachable via ssh)  –orig_master_host=(current(dead) master's hostname)  –orig_master_ip=(current(dead) master's ip address)  –orig_master_port=(current(dead) master's port number)  新的master激活阶段参数：  –command=start  –ssh_user=(new master's ssh username)  –orig_master_host=(dead master's hostname)  –orig_master_ip=(dead master's ip address)  –orig_master_port=(dead master's port number)  –new_master_host=(new master's hostname)  –new_master_ip=(new master's ip address)  –new_master_port(new master's port number)  –new_master_user=(new master's user)  –new_master_password(new master's password)  如果你在master上使用的是共享VIP，那么在master的shutdown阶段就不需要做任何事情，只要在shutdown_script脚本关闭机器的电源之后，在新的master的激活阶段，在新的master上分配这个VIP即可，如果你使用的是一个全局类目映射的方案，你可能需要在shutdown死掉的master阶段之后，删除或者更新对应的dead master的记录。在新的master的激活阶段，你可以新增或者更新对应的新的master的记录，然后，你可以做一些事情（如：在新主库上执行set global read_only=0;以及创建数据库用户的写权限），以便应用程序就可以在新的主库上写入数据了  mha manager会检测脚本执行退出的代号，如果是0或者10，mha manager则继续操作，如果返回的是0和10以外的代号，mha manager停止并中断故障转移，默认情况下，这个参数为空，所以，默认情况下mha不会调用这个脚本做任何事情。

master_ip_online_change_script：这是一个与master_ip_failover_script 参数很接近的参数，但是这个参数不使用故障转移命令，而是master的在线change命令（masterha_master_switch –master_state=alive），使用如下参数（注意：你不需要在配置文件中这个参数上单独设置这些）： 
冻结当前主库写操作阶段的参数：  –command=stop or stopssh  –orig_master_host=(current master's hostname)  –orig_master_ip=(current master's ip address)  –orig_master_port=(current master's port number)  –orig_master_user=(current master's user)  –orig_master_password=(current master's password)  –orig_master_ssh_user=(from 0.56, current master's ssh user)  –orig_master_is_new_slave=(from 0.56, notifying whether the orig master will be new slave or not)  允许新的主库写阶段的参数：  –command=start  –orig_master_host=(orig master's hostname)  –orig_master_ip=(orig master's ip address)  –orig_master_port=(orig master's port number)  –new_master_host=(new master's hostname)  –new_master_ip=(new master's ip address)  –new_master_port(new master's port number)  –new_master_user=(new master's user)  –new_master_password=(new master's password)  –new_master_ssh_user=(from 0.56, new master's ssh user)  在冻结当前主库写的阶段，mha manager在当前主库上执行FLUSH TABLES WITH READ LOCK，你可以写任何的逻辑来优雅地实现主库的切换（参考http://www.slideshare.net/matsunobu/automated-master-failover/44），在新的master允许写的阶段，你可以做和master_ip_failover_script脚本的第三阶段几乎一样的事情，例如：在新的主库上创建写权限用户和执行set global read_only=0语句，或者更新全局类目映射记录，如果这个脚本执行返回状态吗是0或者10以外的值，mha manager将终止并停止master在线切换操作。默认这个参数为空，mha不会调用这个参数做任何事情，在mha manager的tar包和githup分支下的samples/scripts/master_ip_online_change路径下有一个简单的脚本。

shutdown_script：你可能想要强制关闭master节点来避免master节点重新启动服务，这对于避免脑裂来说是很重要的，示例： 
shutdown_script= /usr/local/sample/bin/power_manager  在mha manager的tar包和github的分支的samples/scripts/power_manager路径下有一个简单的脚本，在调用这个脚本之前，mha manager内部会通过ssh去检测master是否可达，如果ssh可达（主机OS活着但是mysqld挂了），mha manager使用如下参数：  –command=stopssh  –ssh_user=(ssh username so that you can connect to the master)  –host=(master's hostname)  –ip=(master's ip address)  –port=(master's port number)  –pid_file=(master's pid file)  如果master通过ssh不可达，那么mha manager就会使用如下参数：  –command=stop  –host=(master's hostname)  –ip=(master's ip address)  这个脚本工作流程如下：  如果通过–command=stopssh参数，这个脚本通过ssh过去master上kill -9杀掉mysqld和mysqld_safe进程，如果同时也通过–pid_file参数，这个脚本通过ssh过去主库上只kill -9杀掉指定的进程号，不会杀掉所有的进程，这个对在master上运行多实例的时候有帮助，如果通过ssh停止成功，这个脚本退出状态是10，如果退出状态是10，mha manager随后就通过ssh连接到master保存必要的binlog，如果这个脚本通过ssh连接master失败，或者mha manager是使用的参数 –command=stop，这个脚本就会尝试关闭机器的电源，关闭机器的命令依赖硬件（For HP(iLO), 通常是ipmitool or SSL，For Dell(DRAC), 通常是dracadm），如果关闭电源成功，这个脚本的退出状态是0，否则退出的状态是1，如果这个脚本的退出状态是0，则mha manager开始执行故障转移过程，如果这个状态是0和10以外的其他状态，则mha终止故障转移，默认这个参数为空，所以mha不会调用这个参数做任何事情。 另外，mha manager开始监控的时候会调用shutdown_script 参数，在这次调用会使用以下参数，在这里可以检测脚本的设置，控制电源需要高度依赖硬件，所以建议要检测电源的状态，如果发现什么问题，你可以在启动监控程序之前去解决。  –command=status  –host=(master's hostname)  –ip=(master's ip address)

report_script：在故障转移完成或者说因为错误而终止的时候，你可能希望发送一个报告出来，这就是使用report_script 参数的目的，mha manager通过如下参数使用这个脚本： 
–orig_master_host=(dead master's hostname)  –new_master_host=(new master's hostname)  –new_slave_hosts=(new slaves' hostnames, delimited by commas)  –subject=(mail subject)  –body=(body)  默认情况下，这个参数是空的，mha manager不会调用这个参数做任何事情，在mha manager的tar包和github分支的samples/scripts/send_report路径下有一个简单的脚本。

init_conf_load_script：这个脚本在你不想在配置文件中设置纯文本的信息的时候使用（比如：password和repl_password），从这个脚本返回name=value的键值对，它可以作为全局配置文件参数，示例脚本内容如下：

#!/usr/bin/perl

print "password=$ROOT_PASS\n";

print "repl_password=$REPL_PASS\n";

这个参数默认为空，所以mha manager不会调用这个参数做任何事情
```