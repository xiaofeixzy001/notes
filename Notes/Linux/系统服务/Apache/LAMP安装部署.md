[TOC]

# 架构方式
LAMP有两种部署方式

1,在一台主机上同时部署httpd，php，mysql，这也是web服务器最初的选择；

2,随着业务的增多，一台主机肯定不够用，这时可以将httpd，php，mysql分开部署；


# 部署在同一主机
## 环境要求：
1，均采用源码编译安装方式

2，建立2个虚拟主机v1.com，v2.com

3，添加xcache为php提供缓存加速页面访问速度

## 资源信息

| 主机名 | 系统版本                             | IP             | 角色 | 备注 |
| ------ | ------------------------------------ | -------------- | ---- | ---- |
| node01 | CentOS Linux release 7.5.1804 (Core) | 192.168.100.11 | LAMP |      |

# 配置过程
## mysql安装
创建mysql用户和组
```
groupadd -r mysql
mkdir -pv /mydata/data
useradd -g mysql -r -s /sbin/nologin -M -d /mydata/data mysql
id mysql 
chown -R mysql.mysql /mydata/data
```
安装mysql
```
yum install -y libaio
tar xf mariadb-10.0.29-linux-x86_64.tar.gz -C /usr/local/
cd /usr/local
ln -sv mariadb-5.5.36-linux-x86_64 mysql
cd /mysql
chown -R root.mysql ./*
```

添加并配置配置文件
```
cp /usr/local/mysql/support-files/my-large.cnf /etc/mysql/my.cnf
cat /etc/mysql/my.cnf
'''
[client]
port            = 3306
socket          = /data/3306/mysql.sock

[mysql]
no-auto-rehash

[mysqld]
user    = mysql
port    = 3306
socket  = /data/3306/mysql.sock
basedir = /usr/local/mysql/
datadir = /data/3306/data  # 添加
open_files_limit    = 1024
back_log = 600
max_connections = 800
max_connect_errors = 3000
table_cache = 614
external-locking = FALSE
max_allowed_packet =8M
sort_buffer_size = 1M
join_buffer_size = 1M
thread_cache_size = 100
thread_concurrency = 2
query_cache_size = 2M
query_cache_limit = 1M
query_cache_min_res_unit = 2k
#default_table_type = InnoDB
thread_stack = 192K
#transaction_isolation = READ-COMMITTED
tmp_table_size = 2M
max_heap_table_size = 2M
long_query_time = 1
#log_long_format
#log-error = /data/3306/error.log
#log-slow-queries = /data/3306/slow.log
pid-file = /data/3306/mysql.pid
log-bin = /data/3306/mysql-bin
relay-log = /data/3306/relay-bin
relay-log-info-file = /data/3306/relay-log.info
binlog_cache_size = 1M
max_binlog_cache_size = 1M
max_binlog_size = 2M
expire_logs_days = 7
key_buffer_size = 16M
read_buffer_size = 1M
read_rnd_buffer_size = 1M
bulk_insert_buffer_size = 1M
#myisam_sort_buffer_size = 1M
#myisam_max_sort_file_size = 10G
#myisam_max_extra_sort_file_size = 10G
#myisam_repair_threads = 1
#myisam_recover

lower_case_table_names = 1
skip-name-resolve
slave-skip-errors = 1032,1062
replicate-ignore-db=mysql

server-id = 1  # 注意这里,id唯一

innodb_additional_mem_pool_size = 4M
innodb_buffer_pool_size = 32M
innodb_data_file_path = ibdata1:128M:autoextend
innodb_file_io_threads = 4
innodb_thread_concurrency = 8
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 2M
innodb_log_file_size = 4M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout = 120
innodb_file_per_table = 0
[mysqldump]
quick
max_allowed_packet = 2M

[mysqld_safe]
log-error=/data/3306/mysql3306.err
pid-file=/data/3306/mysqld.pid
'''
```
初始化数据库
```
scripts/mysql_install_db --help # 用于初始化mysql脚本
scripts/mysql_install_db --user=mysql --datadir=/mydata/data/
ls /mydata/data/
```

配置服务启动脚本
```
ls support-files/
cp support-files/mysql.server /etc/rc.d/init.d/mysqld \\注意执行权限
chkconfig --add mysqld
chkconfig --list mysqld
service mysqld start
ss -tnl
```

配置系统环境变量
```
echo "export PATH=$PATH:/usr/local/mysql/bin" >> /etc/profile
. /etc/profile
cat /etc/profile
```

配置mysql的库文件
```
# 导出头文件
ln -sv /usr/local/mysql/include /usr/include/mysql

# 重载并列出当前系统已加载的所有库文件
ldconfig -v

vim /etc/ld.so.conf.d/mysql.conf
'''
/usr/local/mysql/  # 添加
'''

# 更新为新版的客户端库
ldconfig -v | grep mysql
```

更改mysql的root登录密码，添加一个用于远程连接的用户sqluser
```
mysql -uroot -p

MariaDB [(none)]> SELECT user,host,password FROM mysql.user;
MariaDB [(none)]> DELETE FROM mysql.user WHERE user='';
MariaDB [(none)]> DELETE FROM mysql.user WHERE host='::1';
MariaDB [(none)]> USE mysql;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO root@"localhost" IDENTIFIED BY "123.com";
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO root@"172.16.100.%" IDENTIFIED BY "123.com";
MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* TO root@"127.0.0.1" IDENTIFIED BY "123.com";
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> \q

mysql -u root -p
```
数据库连接测试

## PHP安装

```
yum -y groupinstall "Desktop Platform Development" 

yum install gcc gcc-c++ libmcrypt libmcrypt-devel mhash mhash-devel libxml2-devel php-gd bzip2-devel openssl openssl-devel bzip2 bzip2-devel curl-devel readline-devel -y

tar xf php-5.4.26.tar.bz2
cd php-5.4.26
./configure \
--prefix=/usr/local/php \
--with-config-file-path=/etc \
--with-config-file-scan-dir=/etc/php.d \
--with-mysql=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-mysqli=mysqlnd \
--enable-mbstring \
--with-openssl \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--with-zlib \
--with-libxml-dir=/usr \
--enable-xml  \
--enable-sockets \
--enable-fpm \
--with-mcrypt  \
--with-bz2

make && make install

# 为php提供配置文件：
cp php.ini-production /etc/php.ini

# 配置php-fpm启动脚本
cp sapi/fpm/init.d.php-fpm /etc/rc.d/init.d/php-fpm
chmod +x /etc/rc.d/init.d/php-fpm
chkconfig --add php-fpm
chkconfig --level 3 php-fpm on

# 提供配置文件
cd /usr/local/php/etc/
cp php-fpm.conf.default php-fpm.conf

vim php-fpm.conf
"""
listen = 0.0.0.0:9000  # 监听在哪个主机的套接字上,如果web服务和php-fpm同机则无需修改,默认127.0.0.1
pm.max_children = 50 # 最大并发响应数
pm.start_servers = 5 # 服务刚启动时启动几个空闲进程
pm.min_spare_servers = 2 # 最少空闲进程数
pm.max_spare_servers = 8 # 最大空闲进程数
pid = /usr/local/php/var/run/php-fpm.pid  # 新增
"""

service php-fpm start
ss -tnlp
ps aux | grep php-fpm
```
默认情况下，php-fpm监听在127.0.0.1的9000端口，如果要监听其他主机上的端口，需要修改php配置文件php-fpm.conf，也可以使用如下命令验正其是否已经监听在相应的套接字。

## 可能出现的错误:
启动php-fpm,提示错误,但端口已被监听:
```
PHP Warning: PHP Startup: Unable to load dynamic library '/usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/curl.so' - /usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/curl.so: cannot open shared object file: No such file or directory in Unknown on line 0
PHP Warning: PHP Startup: Unable to load dynamic library '/usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/fileinfo.so' - /usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/fileinfo.so: cannot open shared object file: No such file or directory in Unknown on line 0
PHP Warning: PHP Startup: Unable to load dynamic library '/usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/gd.so' - /usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/gd.so: cannot open shared object file: No such file or directory in Unknown on line 0
PHP Warning: PHP Startup: Unable to load dynamic library '/usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/json.so' - /usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/json.so: cannot open shared object file: No such file or directory in Unknown on line 0
PHP Warning: PHP Startup: Unable to load dynamic library '/usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/phar.so' - /usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/phar.so: cannot open shared object file: No such file or directory in Unknown on line 0
PHP Warning: PHP Startup: Unable to load dynamic library '/usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/zip.so' - /usr/local/php/lib/php/extensions/no-debug-non-zts-20131226/zip.so: cannot open shared object file: No such file or directory in Unknown on line 0
Could not open input file: restart
```
解决办法：
```
mkdir /etc/php.d/bak
mv /etc/php.d/*.ini /etc/php.d/bak/
```

## apache配置
启用httpd的相关模块
在httpd 2.4版本以后已经专门有一个模块针对FastCGI的实现，此模块为:
```
mod_proxy_fcgi.so
```
它其实是作为 mod_proxy.so 模块的扩充，因此，这两个模块都要加载:
```
[root@linux-study ~]# vim /etc/httpd/httpd.conf
"""
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
"""
```
## 配置使用fcgi
```
vim /etc/httpd/extra/httpd-vhosts.conf
"""
<VirtualHost 172.16.100.10>
        ServerName v1.com
        DocumentRoot "/data/vhosts/v1"
        ProxyRequests Off  # 关闭正向代理
        ProxyPassMatch ^/(.*\.php)$ fcgi://172.16.100.9:9000/data/vhosts/v1/$1
        # 反代,以.php结尾的任意请求资源通过fcgi协议反代至本地的url路径，php-fpm至少需要知道运行的目录和URI，所以这里直接在 fcgi://127.0.0.1:9000 后指明了这两个参数，其它的参数的传递已经被mod_proxy_fcgi.so 进行了封装，不需要手动指定, $1表示正则匹配到url路径
        ErrorLog "logs/v1_errors_log"
        CustomLog "logs/v1_access_log" combind

        <Directory "/data/vhosts/v1">
                Options none
                AllowOverride None
                Require all granted
        </Directory>
</VirtualHost>
"""
```
在相应的虚拟主机中添加类似如下两行，如果没有启用虚拟主机，则在全局配置段内添加即可.
```
# 关闭正向代理功能
ProxyRequests Off
ProxyPassMatch ^/(.*\.php)$

# 将以.php结尾的任何请求文件以fcgi协议模式全部交给后端127.0.0.1:9000/PATH/TO/DOCUMENT_ROOT/$1处理 
```
其中127.0.0.1:9000指的是php所在服务器地址，也可以是另外一台主机, PATH/TO/DOCUMENT_ROOT 指的是网页文件根目录，$1后相引用，指引用前面括号()中的内容
fcgi://127.0.0.1:9000/PATH/TO/DOCUMENT_ROOT/$1 

## 配置Apache可识别php请求
```
vim /etc/httpd/httpd.conf 
# 添加
AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps

DirectoryIndex index.php index.html  # 添加index.php
```
测试

浏览器访问:http://v1.com/admin/index.php

压力测试:
```
ab -c 200 -n 10000
172.16.100.10/admin/index.php
```
# End