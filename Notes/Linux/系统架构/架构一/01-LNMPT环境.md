[TOC]

# 需求

Nginx作反代服务器，根据域名转发至后端不同的动态服务器。

MySQL单机多实例，

# 架构图

![image-20200520110238917](01-LNMP%E7%8E%AF%E5%A2%83.assets/image-20200520110238917.png)



# 主机信息表

| *主机名* | *资源配置*   | *操作系统*                    | *角色*        | *IP*           |
| -------- | ------------ | ----------------------------- | ------------- | -------------- |
| nginx    | 2/cpu+2G/mem | CentOS Linux release 7.5.1804 | Web-Server    | 192.168.100.11 |
| php      | 2/cpu+2G/mem | CentOS Linux release 7.5.1804 | PHP-Server    | 192.168.100.21 |
| tomcat   | 2/cpu+2G/mem | CentOS Linux release 7.5.1804 | Tomcat-Server | 192.168.100.31 |
| MariaDB  | 2/cpu+2G/mem | CentOS Linux release 7.5.1804 | DB-Server     | 192.168.100.41 |



# 软件版本

| **名称** | **版本**                                    |
| -------- | ------------------------------------------- |
| Nginx    | nginx-1.16.1.tar.gz                         |
| PHP      | php-7.4.6.tar.gz                            |
| JDK      | jdk-8u241-linux-x64.tar.gz                  |
| Tomcat   | apache-tomcat-8.5.51.tar.gz                 |
| MariaDB  | mariadb-10.4.13-linux-systemd-x86_64.tar.gz |



# 1,安装Nginx

安装方式：源码编译安装

主程序安装目录：/apps/nginx

配置文件目录：/apps/nginx/conf/nginx.conf

日志文件目录：/apps/nginx/logs/

PID文件目录：/apps/nginx/run/nginx.pid

LOCK文件目录：/apps/nginx/run/nginx.lock

## 1.1，依赖环境

```shell
yum -y install pcre pcre-devel openssl openssl-devel zlib-devel gcc
rpm -q pcre pcre-devel openssl openssl-devel
groupadd -r nginx
useradd -r -g nginx -s /bin/false -M nginx
mkdir -pv /apps/nginx/{logs,run,client,proxy,fcgi,uwsgi,scgi}
```



## 1.2，编译安装

```shell
tar xf nginx-1.16.1.tar.gz
cd nginx-1.16.1/

./configure \
--prefix=/apps/nginx \
--conf-path=/apps/nginx/conf/nginx.conf \
--sbin-path=/apps/nginx/sbin/nginx \
--error-log-path=/apps/nginx/logs/error.log \
--http-log-path=/apps/nginx/logs/access.log \
--pid-path=/apps/nginx/run/nginx.pid  \
--lock-path=/apps/nginx/run/nginx.lock \
--user=nginx \
--group=nginx \
--with-http_ssl_module \
--with-http_flv_module \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--http-client-body-temp-path=/apps/nginx/client/ \
--http-proxy-temp-path=/apps/nginx/proxy/ \
--http-fastcgi-temp-path=/apps/nginx/fcgi/ \
--http-uwsgi-temp-path=/apps/nginx/uwsgi \
--http-scgi-temp-path=/apps/nginx/scgi \
--with-pcre

make && make install
echo $?
```



## 1.3，服务启动脚本

```shell
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /apps/nginx/conf/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /apps/nginx/run/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/apps/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/apps/nginx/conf/nginx.conf"

[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx

lockfile=/apps/nginx/run/nginx.lock

make_dirs() {
   # make required directories
   user=`nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   options=`$nginx -V 2>&1 | grep 'configure arguments:'`
   for opt in $options; do
       if [ `echo $opt | grep '.*-temp-path'` ]; then
           value=`echo $opt | cut -d "=" -f 2`
           if [ ! -d "$value" ]; then
               # echo "creating" $value
               mkdir -p $value && chown -R $user $value
           fi
       fi
   done
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```



## 1.4，启动服务

```shell
chmod +x /etc/rc.d/init.d/nginx 
chkconfig --add nginx
chkconfig --level 3 nginx on
service nginx start

# centos7
cat /usr/lib/systemd/system/nginx.service
"""
[Unit]
Description=nginx - high performance web server
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/apps/nginx/sbin/nginx
ExecReload=/apps/nginx/sbin/nginx -s reload
ExecStop=/apps/nginx/sbin/nginx -s stop

[Install]
WantedBy=multi-user.target
"""

systemctl daemon-reload
systemctl start nginx.service
systemctl enable nginx.service
systemctl status nginx.service
systemctl list-units --type=service
```

测试访问：http://192.168.100.11

# 2,安装PHP

安装方式：源码编译安装

主程序安装目录：/apps/php

配置文件目录：/apps/php/etc

## 2.1，依赖环境

```shell
yum install -y gcc gcc-c++ make libxml2-devel sqlite-devel oniguruma-devel zlib-devel libcurl-devel mhash openssl-devel bzip2-devel libcurl-devel

groupadd -r nginx
useradd -r -g nginx -s /bin/false -M nginx
```



## 2.2，编译安装

```shell
tar zxf php-7.4.6.tar.gz
cd php-7.4.6

./configure \
--prefix=/apps/php \
--with-config-file-path=/apps/php/etc \
--with-config-file-scan-dir=/apps/php/etc/php.d \
--enable-fpm \
--with-pdo-mysql=mysqlnd \
--with-mysqli=mysqlnd \
--with-iconv-dir \
--with-zlib-dir \
--with-curl \
--with-openssl \
--with-mhash \
--with-bz2

make
make install

# 如果编译失败
make clean
```



## 2.3，服务启动脚本

```shell
# 配置服务启动脚本
cp sapi/fpm/init.d.php-fpm /etc/rc.d/init.d/php-fpm
chmod +x /etc/rc.d/init.d/php-fpm
chkconfig --add php-fpm
chkconfig php-fpm on

# centos7
cp sapi/fpm/php-fpm.service /usr/lib/systemd/system/
```



## 2.4，配置文件

```shell
cp php-7.4.6/php.ini-production /apps/php/etc/php.ini
cd /apps/php/etc
cp php-fpm.conf.default php-fpm.conf
vim php-fpm.conf
"""
pid = /apps/php/var/run/php-fpm.pid
error_log = /apps/php/var/log/php-fpm.log
"""

cd php-fpm.d/
cp www.conf.default www.conf
vim www.conf
"""
# 确保本地用户和组存在
user = nginx
group = nginx
listen = 192.168.100.21:9000
pm.max_children = 150
pm.start_servers = 8
pm.min_spare_servers = 5
pm.max_spare_servers = 10
"""

service php-fpm start
ps aux | grep php-fpm
netstat -tnl | grep 9000
```



# 3,整合Nginx和PHP

nginx作为反代服务器，将php的请求反代至php服务器。

本地静态文件目录：/data/webroot

PHP服务端项目目录：/data/phpapps

## 3.1，Nginx配置

```shell
mkdir -pv /data/webroot
chown -R nginx.nginx /data/webroot
cd /apps/nginx/conf/
vim nginx.conf
"""
user  nginx;
worker_processes  auto;

error_log   /apps/nginx/logs/error.log  notice;
pid       /apps/nginx/run/nginx.pid;

events {
	worker_connections   10240;
	multi_accept       on;
	use             epoll;
}

http {
	include         mime.types;
	default_type    application/octet-stream;

	log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                 '$status $body_bytes_sent "$http_referer" '
                 '"$http_user_agent" "$http_x_forwarded_for"';

	access_log  /apps/nginx/logs/access.log  main;

	sendfile        on;
	#tcp_nopush     on;
	keepalive_timeout  65;
	server_tokens off;
	gzip  on;
	gzip_min_length 1k;
	gzip_comp_level 3;
	gzip_types text/plain application/javascript application/x-javascript text/javascript text/xml text/css;
	gzip_disable "MSIE[1-6]";
	gzip_vary on;
	
	include extra/*.conf;
}
"""

mkdir extra
vim extra/vhosts.conf
"""
server {
        listen 80;
        server_name test01.com;
		location / {
                root /data/webroot;
                index index.php index.html index.htm;
        }
        location ~* \.php$ {
        		# 此处root是PHP服务端所在位置,确保其存在
                root            /data/phpapps;
                fastcgi_pass    192.168.100.21:9000;
                fastcgi_index   index.php;
                fastcgi_param   SCRIPT_FILENAME /scripts$fastcgi_script_name;
                include         fastcgi_params;
        }
}
"""

vim fastcgi_params
""" 
# 增加如下行
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
"""

echo "Welcome to Nginx...." > /data/webroot/index.html

# 检测配置语法
../sbin/nginx -t
../sbin/nginx -s reload
```



## 3.2，PHP配置

```shell
mkdir -pv /data/phpapps
chown -R nginx.nginx /data/webroot
vim /apps/php/etc/php-fpm.conf
"""
user = nginx
group = nginx
listen = 192.168.100.21:9000
"""

vim /data/phpapps/index.php
"""
<h3>Welcome to PHP!!</h3>

<?php
$servername = "192.168.100.41";
$username = "phpuser";
$password = "123.com";

try {
    $conn = new PDO("mysql:host=$servername;", $username, $password);
    echo "Succeed...";
}
catch(PDOException $e)
{
    echo $e->getMessage();
}
?>
"""
```

重载nginx和php服务，在客户端测试。

http://192.168.100.11

http://192.168.100.11/index.php

由于数据还未配置，所以无法显示连接状态信息。



# 4,安装MariaDB

各版本说明

源代码包，编译用的

mariadb-10.4.13.tar.gz



搞mariadb集群用的，单机不需要

Galera 25.3.22



Windows包

mariadb-10.4.13-winx64.msi
mariadb-10.4.13-winx64.zip
mariadb-10.4.13-win32.zip
mariadb-10.4.13-win32.msi   



下面这个包是包含glibc的二进制包
mariadb-10.4.13-linux-glibc_214-x86_64.tar.gz (requires GLIBC_2.14+)  



各linux发行版二进制通用包，比如centos6
mariadb-10.4.13-linux-x86_64.tar.gz



支持systemd的二进制包， 比如centos7 systemd
mariadb-10.4.13-linux-systemd-x86_64.tar.gz (for systems with systemd)



下面这几个是32位linux的包
mariadb-10.4.13-linux-i686.tar.gz
mariadb-10.4.13-linux-systemd-i686.tar.gz (for systems with systemd)
mariadb-10.4.13-linux-glibc_214-i686.tar.gz (requires GLIBC_2.14+)



下面这两个是rpm包
Debian and Ubuntu Packages
Red Hat, Fedora, and CentOS Packages

## 4.1，LVM卷作为数据存储目录

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

## 4.2，创建用户和组

```shell
groupadd -r -g 306 mysql
useradd -r -m -g 306 -u 306 -d /data/mysql mysql
mkdir /data/mysql
chown -R mysql:mysql /data/mysql
```



## 4.3，安装mariadb

程序包位置：/apps/mysql

数据目录：/data/mydata

```shell
# 创建程序目录
mkdir /apps

# 安装依赖包
yum -y install libaio numactl

# 解压二进制文件
tar xf mariadb-10.4.13-linux-systemd-x86_64.tar.gz -C /apps/
cd /apps
ln -sv mariadb-10.4.13-linux-systemd-x86_64 mysql

# 修改属主和属组
chown -R root:mysql mysql

# 创建配置文件目录
mkdir /etc/mysql
cp /etc/my.cnf /etc/mysql/my.cnf
vim /etc/mysql/my.cnf
"""
[mysqld]
datadir=/data/mysql
socket=/var/lib/mysql/mysql.sock
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mariadb according to the
# instructions in http://fedoraproject.org/wiki/Systemd
innodb_file_per_table=on  # 一张表一个文件
skip_name_resolve=on  # 禁止主机名解析

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
"""

# 服务启动脚本
# centos6
cp support-files/mysql.server /etc/rc.d/init.d/mysqld
chkconfig --add mysqld
chkconfig mysqld on
vim /etc/rc.d/init.d/mysqld
"""
basedir=/apps/mysql
datadir=/data/mysql
"""

# centos7
cp support-files/systemd/mariadb.service /usr/lib/systemd/system/
vim /usr/lib/systemd/system/mariadb.service
"""
ExecStartPre=/bin/sh -c "[ ! -e /apps/mysql/bin/galera_recovery ] && VAR= || \
 VAR=`cd /apps/mysql/bin/..; /apps/mysql/bin/galera_recovery`; [ $? -eq 0 ] \
 && systemctl set-environment _WSREP_START_POSITION=$VAR || exit 1"

ExecStart=/apps/mysql/bin/mysqld $MYSQLD_OPTS $_WSREP_NEW_CLUSTER $_WSREP_START_POSITION
"""

# 初始化数据库数据
./scripts/mysql_install_db --user=mysql --basedir=/apps/mysql --datadir=/data/mysql/

# 添加环境边境
echo "PATH=/apps/mysql/bin:$PATH" > /etc/profile.d/mysql.sh
. /etc/profile.d/mysql.sh

# 启动服务
systemctl start mariadb.service
systemctl status mariadb.service
ss -tnlp

# 初始化安装配置
ln -sv /var/lib/mysql/mysql.sock /tmp/
mysql_secure_installation --basedir=/apps/mysql
mysql -uroot -p
```

配置文件路径：后面覆盖前面的配置文件。

/etc/my.cnf Global选项
/etc/mysql/my.cnf Global选项
SYSCONFDIR/my.cnf Global选项
$MYSQL_HOME/my.cnf Server-specific 选项
--defaults-extra-file=path
~/.my.cnf User-specific 选项

## 4.4，修改mariadb命令提示符

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



## 4.5，创建php连接用户

php数据库：phpapps

php用户：phpuser

```shell
mysql
> SHOW DATABASES;
mysql> create database phpapps character set utf8 collate utf8_bin;
Query OK, 1 row affected (0.07 sec)

mysql> grant all on phpapps.* to "phpuser"@"%" identified by "123.com";
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
> \q

# 测试
mysql -uphpuser -p
```

测试访问：http://192.168.100.11/index.php

# 5,安装tomcat

JDK：/apps/jdk

程序目录：/apps/tomcat

项目目录：/data/webapps

## 5.1，部署java环境jdk

```shell
tar xf jdk-8u241-linux-x64.tar.gz -C /apps/
cd /apps/
ln -sv jdk1.8.0_241 jdk
vim /etc/profile.d/jdk.sh
'''
export JAVA_HOME=/apps/jdk
export PATH=$PATH:$JAVA_HOME/bin:$JAVA_HOME/jre/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
'''

source  /etc/profile.d/jdk.sh && java –version
```



## 5.2，安装Tomcat

```shell
tar xf apache-tomcat-8.5.51.tar.gz -C /apps/
ln -sv apache-tomcat-8.5.51 tomcat
cd tomcat
vim /etc/profile.d/tomcat.sh
"""
export CATALINA_HOME=/usr/local/tomcat
export PATH=$CATALINA_HOME/bin:$PATH
"""

source /etc/profile.d/tomcat.sh
catalina.sh version
catalina.sh start  # 启动服务
ss -tnlp
```

测试：http://192.168.100.31:8080

# 6,整合nginx和tomcat

## nginx配置

修改nginx虚拟主机配置文件/apps/nginx/conf/extra/vhosts.conf

```shell
server {
        listen 80;
        server_name test01.com;

        location / {
                root /data/webroot;
                index index.php index.jsp index.html index.htm;
        }

        location ~* \.php$ {
                root            /data/phpapps;
                fastcgi_pass    192.168.100.21:9000;
                fastcgi_index   index.php;
                fastcgi_param   SCRIPT_FILENAME /scripts$fastcgi_script_name;
                include         fastcgi_params;
        }
        location ~* \.jsp$ {
                root            /data/webapps;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host:$server_port;
                proxy_set_header X-Forwarded-Proto https;
                proxy_redirect off;
                proxy_connect_timeout      240;
                proxy_send_timeout         240;
                proxy_read_timeout         240;
                proxy_pass http://192.168.100.31:8080;  # 注意后面不能带根
        }
}
```

重载nginx配置nginx -s reload

## tomcat配置

修改tomcat配置文件，修改其默认项目目录。

```shell
vim /apps/tomcat/conf/server.xml
"""
<Host name="localhost"  appBase="/data/webapps"
            unpackWARs="true" autoDeploy="true">
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log" suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
        <Context path="" docBase="/data/webapps" reloadable="false" crossContext="true" />
</Host>
"""

mkdir -pv /data/webapps
vim /data/webapps/index.jsp
"""
<html>
        <head>
                <title>test page</title>
        </head>
        <body>
                <% out.println("Hellow World"); %>
        </body>
</html>
"""
```

修改了配置文件，重启tomcat服务。

测试访问：http://192.168.100.11/index.jsp

# End

