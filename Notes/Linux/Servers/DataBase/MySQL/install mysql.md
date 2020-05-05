[TOC]

# 通用版本

mysql下载地址：https://dev.mysql.com/downloads/mysql/

mariadb下载地址：https://downloads.mariadb.org/

## 创建逻辑卷

新增一块硬盘作为数据盘，并将其做成lvm格式

```shell
[root@mysql ~]# fdisk /dev/sdb
:p
:n
:1
:30G

[root@mysql ~]# kpartx -af /dev/sdb  # 让Linux内核读取一个设备上的分区表，然后生成代表相应分区的设备
[root@mysql ~]# partx -a /dev/sdb
[root@mysql ~]# fdisk /dev/sdb
:t
:8e
:w

[root@mysql ~]# partx -a /dev/sdb
[root@mysql ~]# pvcreate /dev/sdb1
[root@mysql ~]# vgcreate myvg /dev/sdb1
[root@mysql ~]# lvcreate -L 20G -n mydata myvg
[root@mysql ~]# mke2fs -t ext4 -L MYDATA -b 4096 -m 3 /dev/myvg/mydata

[root@mysql ~]# mkdir /data
[root@mysql ~]# vim /etc/fstab
"""
/dev/myvg/mydata /data ext4 defaults 0 0
"""

[root@mysql ~]# mount -a
[root@mysql ~]# mount
```

通常为了方便扩容，将数据盘做成lvm逻辑卷。



## 创建用户和组

```shell
[root@mysql ~]# groupadd -r mysql
[root@mysql ~]# mkdir -pv /data/mysql
[root@mysql ~]# useradd -g mysql -r -s /sbin/nologin -M -d /data/mysql/ mysql
[root@mysql ~]# chown -R mysql:mysql /data/mysql
```



## 安装步骤

### 6版本

```shell
# 安装依赖
[root@mysql ~]# yum -y install libaio numactl
[root@mysql ~]# cd /usr/local/src
[root@mysql ~]# wget https://cdn.mysql.com/archives/mysql-5.6/mysql-5.6.46-linux-glibc2.12-x86_64.tar.gz
[root@mysql ~]# tar xf mysql-5.6.46-linux-glibc2.12-x86_64.tar.gz -C /usr/local/
[root@mysql ~]# cd /usr/local/
[root@mysql ~]# ln -sv mysql-5.6.46-linux-glibc2.12-x86_64 mysql
[root@mysql ~]# chown -R mysql:mysql mysql
[root@mysql ~]# scripts/mysql_install_db --user=mysql --datadir=/data/mysql/
[root@mysql ~]# echo $?
[root@mysql ~]# cp support-files/my-default.cnf /etc/my.cnf
[root@mysql ~]# vim /etc/my.cnf
"""
[client]
port            = 3306
socket          = /tmp/mysql.sock
default-character-set = utf8

[mysqld]
port            = 3306
socket          = /data/mysql/mysql.sock
pid-file = /data/mysql/mysql-server.pid
skip-external-locking
key_buffer_size = 256M
max_allowed_packet = 1M
table_open_cache = 256
sort_buffer_size = 1M
read_buffer_size = 1M
read_rnd_buffer_size = 4M
myisam_sort_buffer_size = 64M
thread_cache_size = 8
query_cache_size= 16M
thread_concurrency = 4
basedir = /apps/mysql
datadir = /data/mysql
log-error = /data/mysql/error.log
default-storage-engine = INNODB
character-set-server = utf8
collation-server = utf8_general_ci
log-bin=mysql-bin
binlog_format=mixed
server-id       = 1

[mysqldump]
quick
max_allowed_packet = 16M

[mysql]
no-auto-rehash

[myisamchk]
key_buffer_size = 128M
sort_buffer_size = 128M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout
"""

[root@mysql ~]# cp support-files/mysql.server /etc/rc.d/init.d/mysqld
[root@mysql ~]# chmod +x /etc/rc.d/init.d/mysqld
[root@mysql ~]# chkconfig --add mysqld
[root@mysql ~]# chkconfig mysqld on

[root@mysql ~]# vim /etc/man.config
"""
#MySQL
MANPATH /apps/mysql/man  # mysql的man手册至man命令的查找路径
"""

# 输出mysql的头文件至系统头文件路径/usr/include
[root@mysql ~]# ln -sv /apps/mysql/include /usr/include/mysql

# 输出mysql的库文件给系统库查找路径
[root@mysql ~]# echo "/apps/mysql/lib" > /etc/ld.so.conf.d/mysql.conf
[root@mysql ~]# ldconfig
[root@mysql ~]# ldconfig -p | grep mysql
[root@mysql ~]# echo "export PATH=$PATH:/apps/mysql/bin" >> /etc/profile
[root@mysql ~]# . /etc/profile
[root@mysql ~]# echo $PATH
[root@mysql ~]# ln -sv /data/mysql/mysql.sock /tmp/mysql.sock
[root@mysql ~]# service mysqld start
[root@mysql ~]# cat /data/mysql/mydb.err
```



### 7版本

```shell
[root@mysql ~]# vim /etc/my.cnf
"""
[mysqld]
datadir=/data/mysql
socket=/data/mysql/mysql.sock
user=mysql
basedir=/apps/mysql
symbolic-links=0

[mysqld_safe]
log-error=/data/mysql/mysqld.log
pid-file=/data/mysql/mysqld.pid
"""

# 初始化后会生成一个root的随机密码
[root@mysql ~]# ./bin/mysqld --initialize --user=mysql --basedir=/apps/mysql --datadir=/data/mysql
[root@mysql ~]# cp support-files/mysql.server /etc/rc.d/init.d/mysqld
# 如果改了默认位置,需要修改服务启动脚本

[root@mysql ~]# vim /etc/rc.d/init.d/mysqld
"""
basedir=/apps/mysql
datadir=/data/mysql
lockdir='/data'
"""

[root@mysql ~]# chmod +x /etc/rc.d/init.d/mysqld
[root@mysql ~]# chkconfig --add mysqld
[root@mysql ~]# chkconfig mysqld on
[root@mysql ~]# chkconfig --list mysqld
[root@mysql ~]# service mysqld start
[root@mysql ~]# echo "export PATH=$PATH:/apps/mysql/bin" >> /etc/profile
[root@mysql ~]# . /etc/profile
[root@mysql ~]# echo $PATH
[root@mysql ~]# mysql -uroot -p"gSwp-V5tN(ua" -S /data/mysql/mysql.sock

mysql> alter user 'root'@'localhost' identified by '123.com';  # 重置密码
mysql> SELECT user,host,authentication_string FROM mysql.user;
mysql> update user set authentication_string=password('123.com') where user='root' and Host='%';
mysql> update user set host = "%" where user = "root";

[root@mysql ~]# ln -sv /data/mysql/mysql.sock /tmp/mysql.sock
[root@mysql ~]# mysql -uroot -p
```



### 8版本

```shell
# 初始化后会生成一个root的随机密码
[root@mysql ~]# ./bin/mysqld --initialize --user=mysql --basedir=/apps/mysql --datadir=/data/mysql
[root@mysql ~]# cp support-files/mysql.server /etc/rc.d/init.d/mysqld
# 如果改了默认位置,需要修改服务启动脚本

[root@mysql ~]# vim /etc/rc.d/init.d/mysqld
"""
basedir=/apps/mysql
datadir=/data/mysql
lockdir='/data'
"""

[root@mysql ~]# chmod +x /etc/rc.d/init.d/mysqld
[root@mysql ~]# chkconfig --add mysqld
[root@mysql ~]# chkconfig mysqld on
[root@mysql ~]# chkconfig --list mysqld
[root@mysql ~]# service mysqld start
[root@mysql ~]# echo "export PATH=$PATH:/apps/mysql/bin" >> /etc/profile
[root@mysql ~]# . /etc/profile
[root@mysql ~]# echo $PATH

# 查看用户信息
mysql> select host,user,authentication_string,plugin from mysql.user;

# 更新远程连接
mysql> update user set host = "%" where user = "root";

# 更新密码验证方式为mysql_native_password
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'Beijing$123';

mysql> flush privileges;
```



## 设置root

服务正常启动后，优化一下用户

```shell
[root@mysql ~]# mysql -uroot -p

> SELECT user,host,password FROM mysql.user;
> DELETE FROM mysql.user WHERE user='';
> DELETE FROM mysql.user WHERE host='::1';
> USE mysql;
> GRANT ALL PRIVILEGES ON *.* TO root@"localhost" IDENTIFIED BY "123.com";
> GRANT ALL PRIVILEGES ON *.* TO root@"172.16.100.%" IDENTIFIED BY "123.com";
> GRANT ALL PRIVILEGES ON *.* TO root@"127.0.0.1" IDENTIFIED BY "123.com";
> FLUSH PRIVILEGES;
> \q

[root@mysql ~]# mysql -u root -p

# 或删除全部用户,重新授权管理员
[root@mysql ~]# mysql -u root -p

> DELETE FROM mysql.user;
> GRANT ALL PRIVILEGES ON *.* TO admin@'localhost' IDENTIFIED BY 'oldboy123' WITH GRANT OPTION;
```



## 修改mysql命令提示符

```shell
# 临时修改
mysql> prompt \u@mysql \r:\m:\s> 

# 永久修改
[root@mysql ~]# vim /etc/mysql/my.cnf
"""
[mysql]  # 注意是mysql而非mysqld
prompt = \\u@\\h \\d \\r:\\m:\\s>
"""
```