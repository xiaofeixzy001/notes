[TOC]

# 二进制包安装脚本

说明:

脚本后面跟参数:解压后的软件包名

```shell
#!/bin/bash
# Filename:    install_mysql.sh
# Revision:    1
# Date:        2018/05/31
# Author:      xiaofei
# Email:       1023668666@qq.com
# system:      CentOS 6
# Description: Auto install mysql

. /etc/init.d/functions

appname=$1
appuser="mysql"
basedir="/apps"
datadir="/data/mysql"
passwd="123.com"

# 安装依赖环境
/usr/bin/yum install libaio expect expect-devel -y 

# 判断数据存储路径是否存在
[ ! -d ${datadir} ] && /bin/mkdir -p ${datadir}

# 判断安装路径是否存在
[ ! -d ${basedir} ] && /bin/mkdir -p ${basedir}

# 判断mysql用户和组是否存在
if ! $(id ${appuser} &>/dev/null) ; then
   /usr/sbin/useradd  -s /sbin/nologin -M ${appuser}
   echo "user $appuser created."
fi

/usr/bin/ln -sv ${appname} ${basedir}/mysql
/bin/chown -R ${appuser}:${appuser} ${basedir}/mysql
/bin/chown -R ${appuser}:${appuser} ${datadir}
cd ${basedir}/mysql

ins_log_file=${basedir}/mysql.install.txt

# 初始化mysql
./bin/mysqld --user=${appuser} --basedir=${basedir}/mysql --datadir=${datadir} --initialize $> ${ins_log_file}

cat ${ins_log_file}

[ -f /etc/my.cnf ] && mv /etc/my.cnf{,.back.$(date +"%Y%d%m%H%M%S")} &> /dev/null 
/usr/bin/cp support-files/my-default.cnf /etc/my.cnf
/usr/bin/cp support-files/mysql.server  /etc/rc.d/init.d/mysqld

# 编辑配置文件my.cnf
cat << EOF >/etc/my.cnf

[client]
port        = 3306
socket      = ${basedir}/mysql/mysql.sock
default-character-set=utf8mb4

[mysqld]
port        = 3306
socket      = ${basedir}/mysql/mysql.sock
basedir =  /apps/mysql
datadir = /data/mysql
skip-external-locking
skip_name_resolve=1
key_buffer_size = 16M
max_allowed_packet = 1M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
thread_cache_size = 8
query_cache_size = 8M
tmp_table_size = 16M
performance_schema_max_table_instances = 500

explicit_defaults_for_timestamp = true
#skip-networking
max_connections = 500
max_connect_errors = 100
open_files_limit = 65535

log-bin=mysql-bin
binlog_format=mixed
server-id   = 1
expire_logs_days = 10
early-plugin-load = ""

#loose-innodb-trx=0
#loose-innodb-locks=0
#loose-innodb-lock-waits=0
#loose-innodb-cmp=0
#loose-innodb-cmp-per-index=0
#loose-innodb-cmp-per-index-reset=0
#loose-innodb-cmp-reset=0
#loose-innodb-cmpmem=0
#loose-innodb-cmpmem-reset=0
#loose-innodb-buffer-page=0
#loose-innodb-buffer-page-lru=0
#loose-innodb-buffer-pool-stats=0
#loose-innodb-metrics=0
#loose-innodb-ft-default-stopword=0
#loose-innodb-ft-inserted=0
#loose-innodb-ft-deleted=0
#loose-innodb-ft-being-deleted=0
#loose-innodb-ft-config=0
#loose-innodb-ft-index-cache=0
#loose-innodb-ft-index-table=0
#loose-innodb-sys-tables=0
#loose-innodb-sys-tablestats=0
#loose-innodb-sys-indexes=0
#loose-innodb-sys-columns=0
#loose-innodb-sys-fields=0
#loose-innodb-sys-foreign=0
#loose-innodb-sys-foreign-cols=0

default_storage_engine = InnoDB
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci 
innodb_file_per_table = 1

#innodb_data_home_dir = /application/mysql/data
#innodb_data_file_path = ibdata1:10M:autoextend
#innodb_log_group_home_dir = /application/mysql/data
#innodb_buffer_pool_size = 16M
#innodb_log_file_size = 5M
#innodb_log_buffer_size = 8M
#innodb_flush_log_at_trx_commit = 1
#innodb_lock_wait_timeout = 50

[mysqldump]
quick
max_allowed_packet = 16M

[mysql]
no-auto-rehash

[myisamchk]
key_buffer_size = 20M
sort_buffer_size = 20M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout

[mysqld_safe]
pid-file=${basedir}/mysql/mysqld.pid

EOF

#添加环境变量
echo "export PATH="'$PATH'":${basedir}/mysql/bin" >/etc/profile.d/${basedir}.sh
/bin/bash /etc/profile.d/${basedir}.sh && . /etc/profile.d/${basedir}.sh

#加入开机自启动
/sbin/chkconfig --add mysqld
/sbin/chkconfig mysqld on

# 输出mysql的头文件至系统头文件路径/usr/include
/bin/ln -sv ${basedir}/mysql/include /usr/include/mysql

# 输出mysql的库文件给系统库查找路径
/bin/echo "${basedir}/mysql/lib" > /etc/ld.so.conf.d/mysql.conf
/sbin/ldconfig

/sbin/service mysqld start


# 设置管理员密码
function change_mysql_passwd(){
       /bin/expect -c "
       set time 30
       spawn ${basedir}/mysql/bin/mysqladmin -u root  -p  password \"root\"
       expect {
          \"*yes/no\" { send \"yes\r\"; exp_continue }
          \"*password:\" { send \"$passwd\r\" }
       }  
       interact
       expect eof " >/dev/null 2>&1 ;

      if [ $? -eq 0 ];then
           action "mysql password changes succeeded"   /bin/true
         else
           action "mysql password changes fail"    /bin/false
      fi
}

change_mysql_passwd
```

# 编译安装脚本

注:如果是5.5+版本,需要事先安装cmake

```SHELL
#!/bin/bash

# Notes: install mysql5.6 on centos
#
mysql_install_dir=/applications/mysql
mysql_data_dir=/data/mysql
mysql_6_version=5.6.26
dbrootpwd=root
 
Mem=`free -m | awk '/Mem:/{print $2}'`
Swap=`free -m | awk '/Swap:/{print $2}'`
 
Install_MySQL-5-6()
{
yum -y install make gcc-c++ cmake bison-devel  ncurses-devel
wget http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-${mysql_6_version}.tar.gz

id -u mysql >/dev/null 2>&1
[ $? -ne 0 ] && useradd -M -s /sbin/nologin mysql

mkdir -p $mysql_data_dir;chown mysql.mysql -R $mysql_data_dir
tar zxf mysql-${mysql_6_version}.tar.gz
cd mysql-$mysql_6_version
make clean
[ ! -d "$mysql_install_dir" ] && mkdir -p $mysql_install_dir

cmake . -DCMAKE_INSTALL_PREFIX=$mysql_install_dir \
-DMYSQL_DATADIR=$mysql_data_dir \
-DSYSCONFDIR=/etc \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DENABLE_DTRACE=0 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8mb4 \
-DDEFAULT_COLLATION=utf8mb4_general_ci \
-DWITH_EMBEDDED_SERVER=1 \
 
make -j `grep processor /proc/cpuinfo | wc -l`
make install
 
if [ -d "$mysql_install_dir/support-files" ];then
    echo "${CSUCCESS}MySQL install successfully! ${CEND}"
    cd ..
    rm -rf mysql-$mysql_6_version
else
    rm -rf $mysql_install_dir
    echo "${CFAILURE}MySQL install failed, Please contact the author! ${CEND}"
    kill -9 $$
fi
 
/bin/cp $mysql_install_dir/support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
chkconfig mysqld on
cd ..
 
# my.cf
[ -d "/etc/mysql" ] && /bin/mv /etc/mysql{,_bk}
cat > /etc/my.cnf << EOF
[client]
port = 3306
socket = /tmp/mysql.sock
default-character-set = utf8mb4
 
[mysqld]
port = 3306
socket = /tmp/mysql.sock
 
basedir = $mysql_install_dir
datadir = $mysql_data_dir
pid-file = $mysql_data_dir/mysql.pid
user = mysql
bind-address = 0.0.0.0
server-id = 1
 
init-connect = 'SET NAMES utf8mb4'
character-set-server = utf8mb4
 
skip-name-resolve
skip-external-locking
#skip-networking
back_log = 300
 
max_connections = 1000
max_connect_errors = 6000
open_files_limit = 65535
table_open_cache = 128
max_allowed_packet = 4M
binlog_cache_size = 1M
max_heap_table_size = 8M
tmp_table_size = 16M
 
read_buffer_size = 2M
read_rnd_buffer_size = 8M
sort_buffer_size = 8M
join_buffer_size = 8M
key_buffer_size = 4M
thread_cache_size = 8
query_cache_type = 1
query_cache_size = 8M
query_cache_limit = 2M
ft_min_word_len = 4
log_bin = mysql-bin
binlog_format = mixed
expire_logs_days = 10
log_error = $mysql_data_dir/mysql-error.log
slow_query_log = 1
long_query_time = 1
#slow_query_log_file = $mysql_data_dir/mysql-slow.log
performance_schema = 0
explicit_defaults_for_timestamp
 
#lower_case_table_names = 1
default_storage_engine = InnoDB
#default-storage-engine = MyISAM
innodb_file_per_table = 1
innodb_open_files = 500
innodb_buffer_pool_size = 64M
innodb_write_io_threads = 4
innodb_read_io_threads = 4
innodb_thread_concurrency = 0
innodb_purge_threads = 1
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 2M
innodb_log_file_size = 32M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout = 120
 
bulk_insert_buffer_size = 8M
myisam_sort_buffer_size = 8M
myisam_max_sort_file_size = 10G
myisam_repair_threads = 1
 
interactive_timeout = 28800
wait_timeout = 28800
 
[mysqldump]
quick
max_allowed_packet = 16M
 
[myisamchk]
key_buffer_size = 8M
sort_buffer_size = 8M
read_buffer = 4M
write_buffer = 4M
 
EOF
 
if [ $Mem -gt 1500 -a $Mem -le 2500 ];then
    sed -i 's@^thread_cache_size.*@thread_cache_size = 16@' /etc/my.cnf
    sed -i 's@^query_cache_size.*@query_cache_size = 16M@' /etc/my.cnf
    sed -i 's@^myisam_sort_buffer_size.*@myisam_sort_buffer_size = 16M@' /etc/my.cnf
    sed -i 's@^key_buffer_size.*@key_buffer_size = 16M@' /etc/my.cnf
    sed -i 's@^innodb_buffer_pool_size.*@innodb_buffer_pool_size = 128M@' /etc/my.cnf
    sed -i 's@^tmp_table_size.*@tmp_table_size = 32M@' /etc/my.cnf
    sed -i 's@^table_open_cache.*@table_open_cache = 256@' /etc/my.cnf
elif [ $Mem -gt 2500 -a $Mem -le 3500 ];then
    sed -i 's@^thread_cache_size.*@thread_cache_size = 32@' /etc/my.cnf
    sed -i 's@^query_cache_size.*@query_cache_size = 32M@' /etc/my.cnf
    sed -i 's@^myisam_sort_buffer_size.*@myisam_sort_buffer_size = 32M@' /etc/my.cnf
    sed -i 's@^key_buffer_size.*@key_buffer_size = 64M@' /etc/my.cnf
    sed -i 's@^innodb_buffer_pool_size.*@innodb_buffer_pool_size = 512M@' /etc/my.cnf
    sed -i 's@^tmp_table_size.*@tmp_table_size = 64M@' /etc/my.cnf
    sed -i 's@^table_open_cache.*@table_open_cache = 512@' /etc/my.cnf
elif [ $Mem -gt 3500 ];then
    sed -i 's@^thread_cache_size.*@thread_cache_size = 64@' /etc/my.cnf
    sed -i 's@^query_cache_size.*@query_cache_size = 64M@' /etc/my.cnf
    sed -i 's@^myisam_sort_buffer_size.*@myisam_sort_buffer_size = 64M@' /etc/my.cnf
    sed -i 's@^key_buffer_size.*@key_buffer_size = 256M@' /etc/my.cnf
    sed -i 's@^innodb_buffer_pool_size.*@innodb_buffer_pool_size = 1024M@' /etc/my.cnf
    sed -i 's@^tmp_table_size.*@tmp_table_size = 128M@' /etc/my.cnf
    sed -i 's@^table_open_cache.*@table_open_cache = 1024@' /etc/my.cnf
fi
 
$mysql_install_dir/scripts/mysql_install_db --user=mysql --basedir=$mysql_install_dir --datadir=$mysql_data_dir
 
chown mysql.mysql -R $mysql_data_dir
service mysqld start
[ -z "`grep ^'export PATH=' /etc/profile`" ] && echo "export PATH=$mysql_install_dir/bin:\$PATH" >> /etc/profile
[ -n "`grep ^'export PATH=' /etc/profile`" -a -z "`grep $mysql_install_dir /etc/profile`" ] && sed -i "s@^export PATH=\(.*\)@export PATH=$mysql_install_dir/bin:\1@" /etc/profile
 
. /etc/profile
 
$mysql_install_dir/bin/mysql -e "grant all privileges on *.* to root@'127.0.0.1' identified by \"$dbrootpwd\" with grant option;"
$mysql_install_dir/bin/mysql -e "grant all privileges on *.* to root@'localhost' identified by \"$dbrootpwd\" with grant option;"
$mysql_install_dir/bin/mysql -uroot -p$dbrootpwd -e "delete from mysql.user where Password='';"
$mysql_install_dir/bin/mysql -uroot -p$dbrootpwd -e "delete from mysql.db where User='';"
$mysql_install_dir/bin/mysql -uroot -p$dbrootpwd -e "delete from mysql.proxies_priv where Host!='localhost';"
$mysql_install_dir/bin/mysql -uroot -p$dbrootpwd -e "drop database test;"
$mysql_install_dir/bin/mysql -uroot -p$dbrootpwd -e "reset master;"
rm -rf /etc/ld.so.conf.d/{mysql,mariadb,percona}*.conf
echo "$mysql_install_dir/lib" > mysql.conf
/sbin/ldconfig
service mysqld stop
}
Install_MySQL-5-6
```