[TOC]

# 资源信息

| 主机名 | 系统版本                             | IP             | 角色        | 备注 |
| ------ | ------------------------------------ | -------------- | ----------- | ---- |
| node01 | CentOS Linux release 7.5.1804 (Core) | 192.168.100.11 | http-server |      |


# yum安装
epel源版本：2.4.6-93.el7.centos
```
rpm -qa httpd
yum install -y httpd
rpm -ql httpd
systemctl start httpd
ss -tnlp
```

http生成的文件:

配置文件：

/etc/httpd/conf/httpd.conf

/etc/httpd/conf.d/*.conf

服务脚本：/etc/rc.d/init.d/httpd

脚本配置文件：/etc/sysconfig/httpd


模块目录：/usr/lib64/httpd/modules


主程序：

/usr/sbin/httpd: prefork

/usr/sbin/httpd.event

/usr/sbin/httpd.worker


日志文件目录：/var/log/httpd/
- access_log：访问日志
- error_log：错误日志

站点文档根目录：/var/www/html/

# 编译安装

## 下载地址：

httpd：

https://mirrors.tuna.tsinghua.edu.cn/apache/httpd/httpd-2.4.37.tar.gz

APR:

https://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-1.6.5.tar.gz

APR-UTIL：

https://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-util-1.6.1.tar.gz

## APR

Apache portable Run-time libraries，Apache可移植运行库。

主要为上层的应用程序提供一个可以跨越多操作系统平台使用的底层支持接口库。

目前，完整的APR实际上包含了三个开发包：apr、apr-util以及apr-iconv，每一个开发包分别独立开发，并拥有自己的版本。

apr中包含了一些通用的开发组件，包括mmap，DSO等等
apr-util该目录中也是包含了一些常用的开发组件。这些组件与apr目录下的相比，它们与apache的关系更加密切一些。比如存储段和存储段组，加密等等。

## apr-iconv
包中的文件主要用于实现iconv编码。目前的大部分编码转换过程都是与本地编码相关的。在进行转换之前必须能够正确地设置本地编码。因此假如两个非本地编码A和B需要转换，则转换过程大致为: A->Local以及Local->B或者B->Local以及Local->A

## 配置
```
cd /usr/local/src
wget https://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-1.6.5.tar.gz
wget https://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-util-1.6.1.tar.gz
wget https://mirrors.tuna.tsinghua.edu.cn/apache/httpd/httpd-2.4.37.tar.gz

for i in `ls ./`; do tar xf $i ; done

# 安装依赖环境
yum groupinstall "Development Tools" "Compatibility Libraries" "Server Platform Development"

# 或
yum -y install pcre-devel ntp make openssl openssl-devel pcre pcre-devel libpng libpng-devel libtiff-devel freetype freetype-devel gd gd-devel fontconfig-devel zlib zlib-devel libevent-devel gcc gcc-c++ flex bison bzip2-devel libXpm libXpm-devel ncurses ncurses-devel libmcrypt libmcrypt-devel libxml2 libxml2-devel imake autoconf automake screen sysstat compat-libstdc++-33 curl curl-devel ncurses-devel libtool ncurses-devel libtool

# 安装apr和apr-util
cd apr-1.5.2
./configure --prefix=/usr/local/apr
make && make install

cd apr-util-1.5.4
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr/
make && make install

# 编译安装httpd
tar xf httpd-2.4.37.tar.gz
cd httpd-2.4.37
./configure --prefix=/usr/local/apache --sysconfdir=/etc/httpd --enable-so --enable-ssl --enable-cgi --enable-rewrite --with-zlib --with-pcre --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util/ -enable-modules=most --enable-mpms-shared=all --with-mpm=event
make && make install
```

## 编译参数说明

--prefix=/usr/local/apache：表示指定apache的安装路径，默认安装路径为/usr/local/apache

--enable-rewrite：提供URL规则的重写更嫩那个，即根据已知的URL地址，转换为其它想要的URL地址

--enable-so：激活apache服务的DSO（Dynamic Shared Objects动态共享目标），即在以后可以以DSO的方式编译安装共享模块，这个模块本身不能以DSO方式编译。

--enable-headers：提供允许对HTTP请求头的控制。

--enable-expires：激活荀彧通过配置文件控制HTTP的“Expires:”和“Cache-Control:”头内容，即对网站图片、js、css等内容，提供客户端浏览器缓存的设置。这个是apache调优的一个重要选项之一。

--with-mpm=worker：选择apache mpm的模式为worker模式。为worker模式原理是更多的使用线程来处理请求，所以可以处理更多的并发请求。而系统 资源的开销小玉基于进程的MPM prefork。如果不指定此参数，默认的模式是prefork进程模式。这个是apache调优的一个重要选项之一。

--enable-deflate：提供对内容的压缩传输编码支持，一般是html、js、css等内容的站点。使用此参数会打打提高传输速度，提升访问者访问的体验。在生产环境中，这是apache调优的一个重要选项之一。


# 安装后操作
```
# 添加启动脚本
vim /etc/init.d/httpd
'''
# 最后附服务启动脚本
'''
chmod +x /etc/profile.d/httpd.sh
chkconfig --add httpd
chkconfig --list

# 添加环境变量
vim /etc/profile.d/httpd.sh
"""
export PATH=/usr/local/apache/bin:$PATH
"""
source /etc/profile.d/httpd.sh

#修改配置文件
vim /etc/httpd/httpd.conf
"""
PidFile "/var/run/httpd.pid"  # 
ServerName xiaofei.com:80
"""
# 添加man帮助文档
im /etc/man.conf
"""
MANPATH /usr/local/apache/man  # 添加
"""

# 启动服务
service httpd start
netstat -lntp | grep 80
lsof -i :80
测试访问：http://192.168.100.20/
```

# httpd.conf配置文件
```
ServerRoot "/usr/local/apache2"
# 表示apache根目录，该目录应只有root用户具有访问，一般不需要修改。

Listen 80
# 表示apache监听端口，默认为80。如果同时监控81端口，可以加一行: Listen 81。

AddType application/x-httpd-php .php
LoadModule php5_module modules/libphp5.so
# 用于apache与php进行集成时使用。

User daemon
Group daemon
# 表示apache运行时的用户及组，默认为daemon,建议修改，如apache。

DocumentRoot "/usr/local/apache2/htdocs"
# 表示apache默认的web站点目录，路径结尾不要添加斜线。

ServerAdmin you@example.com
# 表示系统管理员的邮箱，此项为非重要选项。当网站出现问题时，页面会显示此页面地址。

DirectoryIndex index.php index.html
# 配置默认的apache首页。如果虚拟主机未配置，默认应用这里的配置。

ErrorLog "logs/error_log"
# 错误日志路径,相对路径

LogLevel warn
# 错误日志级别。

ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
# 配置cgi别名。
```

# 虚拟主机
建立2个虚拟主机,不同域名,显示不同页面。

环境如下：

虚拟主机1: v1.xiaofei.com

虚拟主机2: v2.xiaofei.com

## 开启虚拟主机功能
虚拟主机和物理主机不能同时启用，若要启用虚拟主机，需要关闭中心主机。
```
vim /etc/httpd/httpd.conf
"""
# 注释掉此行
# DocumentRoot "/usr/local/apache/htdocs"

# 启用此行    
Include /etc/httpd/extra/httpd-vhosts.conf

# 确保此行是被启用状态
LoadModule log_config_module modules/mod_log_config.so
"""
```
## 编辑虚拟主机配置文件
```
mkdir -pv /data/vhosts/v{1,2}
cp /etc/httpd/extra/httpd-vhosts.conf{,.bak}
vim /etc/httpd/extra/httpd-vhosts.conf
'''
<VirtualHost *:80>
        ServerName v1.com
        DocumentRoot "/data/vhosts/v1"
        ErrorLog "logs/v1_errors_log"
        CustomLog "logs/v1_access_log" combind

        <Directory "/data/vhosts/v1">
                Options Indexes MultiViews FollowSymLinks
                AllowOverride None
                Order allow,deny
                Allow from all
                Require all granted
        </Directory>
</VirtualHost>

<VirtualHost *:80>
        ServerName v2.com
        DocumentRoot "/data/vhosts/v2"
        ErrorLog "logs/v2_errors_log"
        CustomLog "logs/v2_access_log" combind

        <Directory "/data/vhosts/v2">
                Options Indexes MultiViews FollowSymLinks
                AllowOverride None
                Order allow,deny
                Allow from all
                Require all granted
        </Directory>
</VirtualHost>
'''
```
测试访问(注意hosts文件)：
v1.com和v2.com

## 虚拟主机配置文件详解
```
grep "Section" httpd.conf
"""
### Section 1: Global Environment # 全局配置
### Section 2: 'Main' server configuration # 主服务器配置 
### Section 3: Virtual Hosts # 虚拟主机配置
"""
```
配置文件内的指令和参数：不区分大小写，但其值区分大小写.

# httpd启动脚本
```
附启动脚本
#!/bin/bash
#
# httpd Startup script for the Apache HTTP Server
#
# chkconfig: - 85 15
# description: Apache is a World Wide Web server.  It is used to serve \
#        HTML files and CGI.
# processname: httpd
# config: /etc/httpd/conf/httpd.conf
# config: /etc/sysconfig/httpd
# pidfile: /var/run/httpd.pid

# Source function library.
. /etc/rc.d/init.d/functions

if [ -f /etc/sysconfig/httpd ]; then
        . /etc/sysconfig/httpd
fi

# Start httpd in the C locale by default.
HTTPD_LANG=${HTTPD_LANG-"C"}

# This will prevent initlog from swallowing up a pass-phrase prompt if
# mod_ssl needs a pass-phrase from the user.
INITLOG_ARGS=""

# Set HTTPD=/usr/sbin/httpd.worker in /etc/sysconfig/httpd to use a server
# with the thread-based "worker" MPM; BE WARNED that some modules may not
# work correctly with a thread-based MPM; notably PHP will refuse to start.

# Path to the apachectl script, server binary, and short-form for messages.
apachectl=/usr/local/apache/bin/apachectl
httpd=${HTTPD-/usr/local/apache/bin/httpd}
prog=httpd
pidfile=${PIDFILE-/var/run/httpd.pid}
lockfile=${LOCKFILE-/var/lock/subsys/httpd}
RETVAL=0

start() {
        echo -n $"Starting $prog: "
        LANG=$HTTPD_LANG daemon --pidfile=${pidfile} $httpd $OPTIONS
        RETVAL=$?
        echo
        [ $RETVAL = 0 ] && touch ${lockfile}
        return $RETVAL
}

stop() {
  echo -n $"Stopping $prog: "
  killproc -p ${pidfile} -d 10 $httpd
  RETVAL=$?
  echo
  [ $RETVAL = 0 ] && rm -f ${lockfile} ${pidfile}
}
reload() {
    echo -n $"Reloading $prog: "
    if ! LANG=$HTTPD_LANG $httpd $OPTIONS -t >&/dev/null; then
        RETVAL=$?
        echo $"not reloading due to configuration syntax error"
        failure $"not reloading $httpd due to configuration syntax error"
    else
        killproc -p ${pidfile} $httpd -HUP
        RETVAL=$?
    fi
    echo
}

# See how we were called.
case "$1" in
  start)
  start
  ;;
  stop)
  stop
  ;;
  status)
        status -p ${pidfile} $httpd
  RETVAL=$?
  ;;
  restart)
  stop
  start
  ;;
  condrestart)
  if [ -f ${pidfile} ] ; then
    stop
    start
  fi
  ;;
  reload)
        reload
  ;;
  graceful|help|configtest|fullstatus)
  $apachectl $@
  RETVAL=$?
  ;;
  *)
  echo $"Usage: $prog {start|stop|restart|condrestart|reload|status|fullstatus|graceful|help|configtest}"
  exit 1
esac

exit $RETVAL
```



# End