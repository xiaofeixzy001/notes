[TOC]



# 一，yum安装





# 二，二进制安装

## 2.1 部署数据盘为lvm

新增一块硬盘作为数据盘，并将其做成lvm格式

```shell
fdisk /dev/sdb
:p
:n
:1
:30G

kpartx -af /dev/sdb  # 让Linux内核读取一个设备上的分区表，然后生成代表相应分区的设备
partx -a /dev/sdb
fdisk /dev/sdb
:t
:8e
:w

partx -a /dev/sdb
pvcreate /dev/sdb1
vgcreate myvg /dev/sdb1
lvcreate -L 20G -n mydata myvg
mke2fs -t ext4 -L MYDATA -b 4096 -m 3 /dev/myvg/mydata

mkdir /data
vim /etc/fstab
"""
/dev/myvg/mydata /data ext4 defaults 0 0
"""

mount -a
mount
```

通常为了方便扩容，将数据盘做成lvm逻辑卷

## 2.2 创建用户和组

```shell
groupadd -r mysql
useradd -g mysql -r -s /sbin/nologin -M -d /data/mysql/ mysql
mkdir /data/mysql
chown -R mysql:mysql /data/mysql
```



## 2.3 各版本二进制安装

### 2.3.1 mysql-5.6版本

5.5和5.6安装类似，以下以5.6版本为例。

```shell
# 安装依赖
# yum -y install libaio numactl
# cd /usr/local/src
# wget https://cdn.mysql.com//Downloads/MySQL-5.6/mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz
# tar xf mysql-5.6.42-linux-glibc2.12-x86_64.tar.gz -C /apps/
# cd /apps
# ln -sv mysql-5.6.42-linux-glibc2.12-x86_64 mysql
# chown -R mysql:mysql mysql
# ll
# cd mysql
# scripts/mysql_install_db --user=mysql --datadir=/data/mysql/
# ll
# cp support-files/my-default.cnf /etc/my.cnf

# vim /etc/my.cnf
"""
basedir = /apps/mysql
datadir = /data/mysql
port = 3306
server_id = 11  # 唯一
socket = /data/mysql/mysql.sock
user = mysql
log-error = /data/mysql/mysql.err
pid-file = /data/mysql/mysqld.pid
"""

# cp support-files/mysql.server /etc/rc.d/init.d/mysqld  # 服务启动脚本
# chmod +x /etc/rc.d/init.d/mysqld
# chkconfig --add mysqld
# chkconfig mysqld on
# vim /etc/rc.d/init.d/mysqld
"""
basedir=/apps/mysql
datadir=/data/mysql
mysqld_pid_file_path=/data/mysql/mysql.pid
"""

# vim /etc/man.config
"""
#MySQL
MANPATH /apps/mysql/man  # mysql的man手册至man命令的查找路径
"""

# 输出mysql的头文件至系统头文件路径/usr/include
# ln -sv /apps/mysql/include /usr/include/mysql

# 输出mysql的库文件给系统库查找路径
# echo "/apps/mysql/lib" > /etc/ld.so.conf.d/mysql.conf

# ldconfig
# ldconfig -p | grep mysql

# 导入mysql至系统环境变量
# echo "export PATH=$PATH:/apps/mysql/bin" >> /etc/profile
# . /etc/profile
# echo $PATH

# service mysqld start
# cat /data/mysql/mysql.err
# ln -sv /data/mysql/mysql.sock /tmp/mysql.sock
```



### 2.3.2 mysql-5.7版本

```shell
vim /etc/my.cnf
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
./bin/mysqld --initialize --user=mysql --basedir=/apps/mysql --datadir=/data/mysql
cp support-files/mysql.server /etc/rc.d/init.d/mysqld
# 如果改了默认位置,需要修改服务启动脚本

vim /etc/rc.d/init.d/mysqld
"""
basedir=/apps/mysql
datadir=/data/mysql
lockdir='/data'
"""

chmod +x /etc/rc.d/init.d/mysqld
chkconfig --add mysqld
chkconfig mysqld on
chkconfig --list mysqld
service mysqld start
echo "export PATH=$PATH:/apps/mysql/bin" >> /etc/profile
. /etc/profile
echo $PATH
mysql -uroot -p"gSwp-V5tN(ua" -S /data/mysql/mysql.sock
mysql> alter user 'root'@'localhost' identified by '123.com';  # 重置密码
mysql> SELECT user,host,authentication_string FROM mysql.user;
mysql> update user set authentication_string=password('123.com') where user='root' and Host='%';
mysql> update user set host = "%" where user = "root";

ln -sv /data/mysql/mysql.sock /tmp/mysql.sock
mysql -uroot -p
```



### 2.3.3 mysql-8版本

```shell
# 初始化后会生成一个root的随机密码
./bin/mysqld --initialize --user=mysql --basedir=/apps/mysql --datadir=/data/mysql
cp support-files/mysql.server /etc/rc.d/init.d/mysqld
# 如果改了默认位置,需要修改服务启动脚本

vim /etc/rc.d/init.d/mysqld
"""
basedir=/apps/mysql
datadir=/data/mysql
lockdir='/data'
"""

chmod +x /etc/rc.d/init.d/mysqld
chkconfig --add mysqld
chkconfig mysqld on
chkconfig --list mysqld
service mysqld start
echo "export PATH=$PATH:/apps/mysql/bin" >> /etc/profile
. /etc/profile
echo $PATH

# 查看用户信息
select host,user,authentication_string,plugin from mysql.user;

# 更新远程连接
mysql> update user set host = "%" where user = "root";

# 更新密码验证方式为mysql_native_password
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'Beijing$123';

mysql> flush privileges;
```



## 2.4 优化数据库root用户

服务正常启动后，优化一下用户

```shell
mysql -uroot -h"127.0.0.1"

> SELECT user,host,password FROM mysql.user;
> DELETE FROM mysql.user WHERE user='';
> DELETE FROM mysql.user WHERE host='::1';
> USE mysql;
> GRANT ALL PRIVILEGES ON *.* TO root@"localhost" IDENTIFIED BY "123.com";
> GRANT ALL PRIVILEGES ON *.* TO root@"%" IDENTIFIED BY "123.com";
> GRANT ALL PRIVILEGES ON *.* TO root@"127.0.0.1" IDENTIFIED BY "123.com";
> FLUSH PRIVILEGES;
> \q

mysql -u root -p

# 或删除全部用户,重新授权管理员
mysql -u root -p

> DELETE FROM mysql.user;
> GRANT ALL PRIVILEGES ON *.* TO admin@'localhost' IDENTIFIED BY 'oldboy123' WITH GRANT OPTION;
```

修改mysql命令提示符

```shell
# 临时修改
mysql> prompt \u@mysql \r:\m:\s> 

# 永久修改
vim /etc/mysql/my.cnf
"""
[mysql]  # 注意是mysql而非mysqld
prompt = \\u@\\h \\d \\r:\\m:\\s>
"""
```

1

# 三，源码安装

## 3.1 常规编译安装方式

常规编译安装,适用于5.1及以前版本，延续编译安装3部曲方式:

./configure && make && make install

在5.4及之后的版本需要使用cmake工具编译安装

```shell
./configure \
--prefix=/apps/mysql5.1.72 \
--with-unix-socket-path=/apps/mysql5.1.72/tmp/mysql.sock \
--localstatedir=/apps/mysql5.1.72/data \
--enable-assembler \
--enable-thread-safe-client \
--with-mysqld-user=mysql \
--with-big-tables \
--with-pthread \
--without-debug \
--with-extra-charsets=complex \
--with-readline \
--with-ssl \
--with-embedded-server \
--enable-local-infile \
--with-plugind=psrtition,innobase \
--with-mysqld-ldflags=-all-static \
--with-client-ldflags=-all-static \

make && make install
```



## 3.2 cmake

cmake的重要特性之一是其独立于源码(out-of-source)的编译功能，即编译工作可以在另一个指定的目录中而非源码目录中进行，这可以保证源码目录不受任何一次编译的影响，因此在同一个源码树上可以进行多次不同的编译，如针对于不同平台编译。

cmake指定编译选项的方式不同于make，其实现方式对比如下：

```shell
./configure cmake .
./configure --help cmake . -LH or ccmake .
```

cmake常用的选项:

-DCMAKE_INSTALL_PREFIX=/usr/local/mysql # 指定安装路径

-DMYSQL_DATADIR=/data/mysql # 数据安装路径

-DSYSCONFDIR=/etc # 配置文件的安装路径

默认编译的存储引擎包括：csv、myisam、myisammrg和heap,若要安装其它存储引擎，可以使用类似如下编译选项:

-DWITH_INNOBASE_STORAGE_ENGINE=1 # 安装innobase存储引擎 

-DWITH_ARCHIVE_STORAGE_ENGINE=1 # 安装archive存储引擎 

-DWITH_BLACKHOLE_STORAGE_ENGINE=1 # 安装blackhole存储引擎

-DWITH_FEDERATED_STORAGE_ENGINE=1 # 安装federated存储引擎 

若要明确指定不编译某存储引擎，可以使用类似如下的选项：

-DWITHOUT_<ENGINE>_STORAGE_ENGINE=1

比如:

-DWITHOUT_EXAMPLE_STORAGE_ENGINE=1 # 不启用example存储引擎 

-DWITHOUT_FEDERATED_STORAGE_ENGINE=1 # 不启用federated

-DWITHOUT_PARTITION_STORAGE_ENGINE=1 # 不启用partition

如若要编译进其它功能，如SSL等，则可使用类似如下选项来实现编译时使用某库或不使用某库:

-DWITH_READLINE=1 

-DWITH_SSL=system # 表示使用系统上的自带的SSL库 

-DWITH_ZLIB=system 

-DWITH_LIBWRAP=0

其它常用的选项：

-DMYSQL_TCP_PORT=3306 # 设置默认端口的 

-DMYSQL_UNIX_ADDR=/tmp/mysql.sock # MySQL进程间通信的套接字的位置 

-DENABLED_LOCAL_INFILE=1 # 是否启动本地的local_infile

-DEXTRA_CHARSETS=all # 支持哪些额外的字符集 

-DDEFAULT_CHARSET=utf8 # 默认字符集 

-DDEFAULT_COLLATION=utf8_general_ci # 默认的字符集排序规则 

-DWITH_DEBUG=0 # 是否启用DEBUG功能 

-DENABLE_PROFILING=1 # 是否启用性能分析功能

如果想清理此前的编译所生成的文件，则需要使用如下命令：

make clean

rm -rf CMakeCache.txt

## 3.3 cmake安装MySQL

cmake方式，适用于5.4及之后版本

安装依赖包

```shell
yum -y install yum-fastestmirror patch make flex bison tar libtool libtool-libs kernel-devel libjpeg libjpeg-devel libpng libpng-devel libtiff libtiff-devel gettext gettext-devel libxml2 libxml2-devel zlib-devel  net-snmp file glib2 glib2-devel bzip2 diff* openldap-devel bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel vim-minimal unzip
```

说明：yum-fastestmirror:自动搜索最快镜像插件

```shell
yum -y install gcc gcc-c++ ncurses-devel git
tar xf cmake-2.8.8.tar.gz
cd cmake-2.8.8
./bootstrap
make && make install
```

cmake编译安装

```shell
tar xf mysql-5.5.57.tar.gz
cd mysql-5.5.57
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/data/mysqldb \
-DSYSCONFDIR=/etc \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DWITH_SSL=system \
-DWITH_ZLIB=system \
-DWITH_LIBWRAP=0 \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_COLLATION=utf8_general_ci

make && make install

# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mariadb-5.5.44 -DMYSQL_DATADIR=/mydata/data -DSYSCONFDIR=/etc -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITH_READLINE=1 -DWITH_SSL=system -DWITH_ZLIB=system -DWITH_LIBWRAP=0 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci

cd /usr/local/mysql
chown -R mysql:mysql ./*
chown -R mysql:mysql /data/mysqldb
vim /etc/profile
"""
export PATH=/usr/local/mysql/bin:/usr/local/mysql/lib:$PATH
"""

source /etc/profile
echo $PATH

cd /usr/local/mysql
cp support-files/my-large.cnf /etc/my.cnf
scripts/mysql_install_db --user=mysql --datadir=/data/mysqldb

cd /usr/local/mysql
cp support-files/mysql.server /etc/init.d/mysqld
chkconfig –add mysqld
chkconfig mysqld on
chkconfig –list mysql
chmod +x /etc/init.d/mysqld
service mysql start
netstat -tnulp | grep 3306
```

参数说明：

-DCMAKE_INSTALL_PREFIX=/usr/local/mysql # 安装目录

-DINSTALL_DATADIR=/usr/local/mysql/data # 数据库存放目录

-DDEFAULT_CHARSET=utf8 # 使用utf8字符

-DDEFAULT_COLLATION=utf8_general_ci # 校验字符，设置服务器的排序规则。

-DEXTRA_CHARSETS=all # 安装所有扩展字符集

-DENABLED_LOCAL_INFILE=1 # 允许从本地导入数据

-DMYSQL_UNIX_ADDR=file_name # 设置监听套接字路径，这必须是一个绝对路径名。默认为/tmp/mysql.sock

-DDEFAULT_CHARSET=utf8 缺省情况下，MySQL使用latin1的（CP1252西欧）字符集。cmake/character_sets.cmake文件包含允许的字符集名称列表。

存储引擎参数

-DWITH_INNOBASE_STORAGE_ENGINE=1

-DWITH_ARCHIVE_STORAGE_ENGINE=1

-DWITH_BLACKHOLE_STORAGE_ENGINE=1

-DWITH_PERFSCHEMA_STORAGE_ENGINE=1

MyISAM，MERGE，MEMORY，和CSV引擎是默认编译到服务器中，并不需要明确地安装。

静态编译一个存储引擎到服务器，使用-DWITH_engine_STORAGE_ENGINE= 1

可用的存储引擎值有：ARCHIVE, BLACKHOLE, EXAMPLE, FEDERATED, INNOBASE (InnoDB), PARTITION (partitioning support), 和PERFSCHEMA (Performance Schema)

-DMYSQL_TCP_PORT=port_num

设置mysql服务器监听端口，默认为3306

-DENABLE_DOWNLOADS=bool

是否要下载可选的文件。例如，启用此选项（设置为1），cmake将下载谷歌所使用的测试套件运行单元测试

## 3.4 可能出现的错误

1， Could NOT find Curses (missing:  CURSES_LIBRARY CURSES_INCLUDE_PATH) 

CMake Error at cmake/readline.cmake:83 (MESSAGE):

Curses library not found.  Please install appropriate package,

remove CMakeCache.txt and rerun cmake.On Debian/Ubuntu, 

package name is libncurses5-dev, on Redhat and derivates it is ncurses-devel.

错误说的很明显，找不到Curse需要安装ncurses-devel但是需要先清空前面的安装信息。重新编译时，需要清除旧的对象文件和缓存信息。

```shell
make clean
rm -f CMakeCache.txt
rm -rf /etc/my.cnf
yum install ncurses-devel
```

2，启动时出现

Starting MySQL..The server quit without updating PID file ([FAILED]/mysql/Server03.mylinux.com.pid).   

解决：

修改/etc/my.cnf 中datadir,指向正确的mysql数据库文件目录

3，启动时出现

ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)

解决：

新建一个链接或在mysql中加入-S参数，直接指出mysql.sock位置。

ln -s /usr/local/mysql/data/mysql.sock /tmp/mysql.sock,

/usr/local/mysql/bin/mysql -u root -S /usr/local/mysql/data/mysql.sock

4，mysql命令找不到,

MySQL问题解决：-bash:mysql:command not found

因为mysql命令的路径在/usr/local/mysql/bin下面,所以你直接使用mysql命令时,

系统在/usr/bin下面查此命令,所以找不到了

解决办法是：

ln -s /usr/local/mysql/bin/mysql /usr/bin　做个链接即可

5，configure 与 cmake 参数对照指南：

http://forge.mysql.com/wiki/Autotools_to_CMake_Transition_Guide,

## 3.5 cmake常用的选项

-DCMAKE_INSTALL_PREFIX=/usr/local/mysql # 指定安装路径

-DMYSQL_DATADIR=/data/mysql # 数据安装路径

-DSYSCONFDIR=/etc # 配置文件的安装路径

默认编译的存储引擎包括：csv、myisam、myisammrg和heap,若要安装其它存储引擎，可以使用类似如下编译选项:

-DWITH_INNOBASE_STORAGE_ENGINE=1 # 安装innobase存储引擎

-DWITH_ARCHIVE_STORAGE_ENGINE=1 # 安装archive存储引擎

-DWITH_BLACKHOLE_STORAGE_ENGINE=1 # 安装blackhole存储引擎

-DWITH_FEDERATED_STORAGE_ENGINE=1 # 安装federated存储引擎 

若要明确指定不编译某存储引擎，可以使用类似如下的选项：

-DWITHOUT_<ENGINE>_STORAGE_ENGINE=1

比如:

-DWITHOUT_EXAMPLE_STORAGE_ENGINE=1 # 不启用example存储引擎

-DWITHOUT_FEDERATED_STORAGE_ENGINE=1 # 不启用federated

-DWITHOUT_PARTITION_STORAGE_ENGINE=1 # 不启用partition

如若要编译进其它功能，如SSL等，则可使用类似如下选项来实现编译时使用某库或不使用某库:

-DWITH_READLINE=1

-DWITH_SSL=system # 表示使用系统上的自带的SSL库

-DWITH_ZLIB=system

-DWITH_LIBWRAP=0

其它常用的选项：

-DMYSQL_TCP_PORT=3306 # 设置默认端口的

-DMYSQL_UNIX_ADDR=/tmp/mysql.sock # MySQL进程间通信的套接字的位置

-DENABLED_LOCAL_INFILE=1 # 是否启动本地的local_infile

-DEXTRA_CHARSETS=all # 支持哪些额外的字符集

-DDEFAULT_CHARSET=utf8 # 默认字符集

-DDEFAULT_COLLATION=utf8_general_ci # 默认的字符集排序规则

-DWITH_DEBUG=0 # 是否启用DEBUG功能

-DENABLE_PROFILING=1 # 是否启用性能分析功能

如果想清理此前的编译所生成的文件，则需要使用如下命令：

make clean

rm /源码目录/CMakeCache.txt